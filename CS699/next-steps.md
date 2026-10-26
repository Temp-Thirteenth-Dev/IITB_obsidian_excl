Good — you've already got a solid foundation. Based on your PROJECT_STATUS.md, here's what you need to complete the full local build. I'll give you every file in order.

---

## What you already have ✅
- `users` table, signup/login endpoints (no password hashing yet)
- Next.js frontend with signup/login pages
- Basic FastAPI + psycopg pool setup

## What we're building now
1. Fix auth (bcrypt hashing + JWT tokens)
2. Cycles table + endpoints
3. Bookings state machine + OTP
4. Condition photo ledger (local file storage for now)

---

## Step 1 — Update dependencies

**`api/requirements.txt`**
```
fastapi>=0.115,<1.0
uvicorn[standard]>=0.30
psycopg[binary,pool]>=3.2,<4
python-dotenv>=1.0
bcrypt>=4.0
pyjwt>=2.8
pytest>=8.0
httpx>=0.27
python-multipart>=0.0.9
```

---

## Step 2 — Database migrations

**`db/migrations/002_add_password_hash.sql`**
```sql
-- Replace plain password with hashed version
ALTER TABLE users RENAME COLUMN password TO password_hash;
```

**`db/migrations/003_cycles.sql`**
```sql
CREATE TABLE cycles (
    id          SERIAL PRIMARY KEY,
    owner_id    INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title       TEXT NOT NULL CHECK (char_length(title) BETWEEN 3 AND 80),
    description TEXT,
    price_per_hour INTEGER NOT NULL CHECK (price_per_hour >= 0),  -- in rupees
    hostel      TEXT,          -- e.g. "H4", "H10"
    lat         DOUBLE PRECISION,
    lng         DOUBLE PRECISION,
    is_available BOOLEAN NOT NULL DEFAULT TRUE,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**`db/migrations/004_bookings.sql`**
```sql
CREATE TYPE booking_status AS ENUM (
    'requested', 'accepted', 'active', 'completed', 'cancelled'
);

CREATE TABLE bookings (
    id           SERIAL PRIMARY KEY,
    cycle_id     INTEGER NOT NULL REFERENCES cycles(id) ON DELETE CASCADE,
    renter_id    INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    start_time   TIMESTAMPTZ NOT NULL,
    end_time     TIMESTAMPTZ NOT NULL,
    status       booking_status NOT NULL DEFAULT 'requested',
    start_otp    CHAR(6),       -- hashed, set when accepted
    end_otp      CHAR(6),       -- hashed, set when active
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CHECK (end_time > start_time)
);

-- Prevent overlapping bookings for same cycle
CREATE UNIQUE INDEX no_overlap ON bookings (cycle_id, status)
    WHERE status IN ('accepted', 'active');
```

**`db/migrations/005_condition_ledger.sql`**
```sql
CREATE TABLE condition_ledger (
    id          SERIAL PRIMARY KEY,
    booking_id  INTEGER NOT NULL REFERENCES bookings(id) ON DELETE CASCADE,
    stage       TEXT NOT NULL CHECK (stage IN ('pre', 'post')),
    photo_path  TEXT NOT NULL,   -- local path for now, S3 URL later
    notes       TEXT,
    recorded_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (booking_id, stage)
);
```

Run all migrations:
```bash
psql -U postgres -d pedalpeer -f db/migrations/002_add_password_hash.sql
psql -U postgres -d pedalpeer -f db/migrations/003_cycles.sql
psql -U postgres -d pedalpeer -f db/migrations/004_bookings.sql
psql -U postgres -d pedalpeer -f db/migrations/005_condition_ledger.sql
```

---

## Step 3 — Backend files

**`api/auth.py`** — JWT-based auth (replaces any previous version)
```python
import os, bcrypt, jwt
from datetime import datetime, timedelta, timezone
from fastapi import Header, HTTPException

SECRET = os.environ["JWT_SECRET"]
ALGO   = "HS256"
TTL    = 60 * 24  # minutes

def hash_password(plain: str) -> str:
    return bcrypt.hashpw(plain.encode(), bcrypt.gensalt()).decode()

def verify_password(plain: str, hashed: str) -> bool:
    return bcrypt.checkpw(plain.encode(), hashed.encode())

def make_token(user_id: int, email: str) -> str:
    payload = {
        "sub": str(user_id),
        "email": email,
        "exp": datetime.now(timezone.utc) + timedelta(minutes=TTL),
    }
    return jwt.encode(payload, SECRET, algorithm=ALGO)

def current_user(authorization: str = Header(default="")):
    token = authorization.removeprefix("Bearer ").strip()
    if not token:
        raise HTTPException(401, "missing token")
    try:
        payload = jwt.decode(token, SECRET, algorithms=[ALGO])
    except jwt.ExpiredSignatureError:
        raise HTTPException(401, "token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(401, "bad token")
    return {"id": int(payload["sub"]), "email": payload["email"]}
```

**`api/database.py`** — connection pool
```python
import os
from psycopg_pool import ConnectionPool
from psycopg.rows import dict_row

pool = ConnectionPool(
    os.environ["DATABASE_URL"],
    kwargs={"row_factory": dict_row},
    open=True,
)
```

**`api/otp.py`** — secure OTP generation
```python
import secrets, hashlib, os

def new_code() -> str:
    return f"{secrets.randbelow(1_000_000):06d}"

def hash_code(code: str, booking_id: int) -> str:
    pepper = os.environ.get("OTP_PEPPER", "dev-pepper")
    return hashlib.sha256(f"{booking_id}:{code}:{pepper}".encode()).hexdigest()

def verify_code(submitted: str, stored_hash: str, booking_id: int) -> bool:
    return secrets.compare_digest(hash_code(submitted, booking_id), stored_hash)
```

**`api/main.py`** — all routes
```python
from dotenv import load_dotenv
load_dotenv()

import os, shutil
from pathlib import Path
from fastapi import FastAPI, Depends, HTTPException, UploadFile, File
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field, EmailStr
from typing import Optional
from database import pool
from auth import hash_password, verify_password, make_token, current_user
from otp import new_code, hash_code, verify_code

app = FastAPI(title="PedalPeer API")
app.add_middleware(
    CORSMiddleware,
    allow_origins=[os.environ.get("WEB_ORIGIN", "http://localhost:3000")],
    allow_methods=["*"],
    allow_headers=["*"],
)

UPLOAD_DIR = Path("uploads")
UPLOAD_DIR.mkdir(exist_ok=True)

# ── Pydantic models ────────────────────────────────────────────────────────────

class UserSignup(BaseModel):
    username: str = Field(min_length=3, max_length=30)
    email:    str
    password: str = Field(min_length=6)

class UserLogin(BaseModel):
    email:    str
    password: str

class NewCycle(BaseModel):
    title:          str = Field(min_length=3, max_length=80)
    description:    Optional[str] = None
    price_per_hour: int = Field(ge=0)
    hostel:         Optional[str] = None
    lat:            Optional[float] = None
    lng:            Optional[float] = None

class NewBooking(BaseModel):
    cycle_id:   int
    start_time: str   # ISO format: "2024-11-01T10:00:00"
    end_time:   str

class OTPSubmit(BaseModel):
    code: str = Field(min_length=6, max_length=6)

# ── Health ─────────────────────────────────────────────────────────────────────

@app.get("/health")
def health():
    return {"ok": True}

@app.get("/test-db")
def test_db():
    with pool.connection() as conn:
        row = conn.execute("SELECT version();").fetchone()
        return {"version": row["version"]}

# ── Auth ───────────────────────────────────────────────────────────────────────

@app.post("/signup", status_code=201)
def signup(body: UserSignup):
    with pool.connection() as conn:
        existing = conn.execute(
            "SELECT id FROM users WHERE email = %s", (body.email,)
        ).fetchone()
        if existing:
            raise HTTPException(409, "email already registered")
        row = conn.execute(
            """INSERT INTO users (username, email, password_hash)
               VALUES (%s, %s, %s) RETURNING id, username, email""",
            (body.username, body.email, hash_password(body.password))
        ).fetchone()
        return {"user": row, "token": make_token(row["id"], row["email"])}

@app.post("/login")
def login(body: UserLogin):
    with pool.connection() as conn:
        row = conn.execute(
            "SELECT id, username, email, password_hash FROM users WHERE email = %s",
            (body.email,)
        ).fetchone()
        if not row or not verify_password(body.password, row["password_hash"]):
            raise HTTPException(401, "invalid credentials")
        return {
            "user":  {"id": row["id"], "username": row["username"], "email": row["email"]},
            "token": make_token(row["id"], row["email"])
        }

@app.get("/me")
def me(user=Depends(current_user)):
    with pool.connection() as conn:
        return conn.execute(
            "SELECT id, username, email, created_at FROM users WHERE id = %s",
            (user["id"],)
        ).fetchone()

# ── Cycles ─────────────────────────────────────────────────────────────────────

@app.get("/cycles")
def list_cycles(hostel: Optional[str] = None):
    with pool.connection() as conn:
        if hostel:
            rows = conn.execute(
                """SELECT c.*, u.username as owner_name
                   FROM cycles c JOIN users u ON c.owner_id = u.id
                   WHERE c.is_available = TRUE AND c.hostel = %s
                   ORDER BY c.created_at DESC""",
                (hostel,)
            ).fetchall()
        else:
            rows = conn.execute(
                """SELECT c.*, u.username as owner_name
                   FROM cycles c JOIN users u ON c.owner_id = u.id
                   WHERE c.is_available = TRUE
                   ORDER BY c.created_at DESC"""
            ).fetchall()
        return rows

@app.post("/cycles", status_code=201)
def create_cycle(body: NewCycle, user=Depends(current_user)):
    with pool.connection() as conn:
        row = conn.execute(
            """INSERT INTO cycles (owner_id, title, description, price_per_hour, hostel, lat, lng)
               VALUES (%s, %s, %s, %s, %s, %s, %s) RETURNING *""",
            (user["id"], body.title, body.description,
             body.price_per_hour, body.hostel, body.lat, body.lng)
        ).fetchone()
        return row

@app.delete("/cycles/{cycle_id}")
def delete_cycle(cycle_id: int, user=Depends(current_user)):
    with pool.connection() as conn:
        result = conn.execute(
            "DELETE FROM cycles WHERE id = %s AND owner_id = %s",
            (cycle_id, user["id"])
        )
        if result.rowcount == 0:
            raise HTTPException(404, "not found or not your cycle")
        return {"ok": True}

# ── Bookings ───────────────────────────────────────────────────────────────────

@app.post("/bookings", status_code=201)
def request_booking(body: NewBooking, user=Depends(current_user)):
    with pool.connection() as conn:
        cycle = conn.execute(
            "SELECT * FROM cycles WHERE id = %s AND is_available = TRUE",
            (body.cycle_id,)
        ).fetchone()
        if not cycle:
            raise HTTPException(404, "cycle not available")
        if cycle["owner_id"] == user["id"]:
            raise HTTPException(400, "cannot book your own cycle")
        row = conn.execute(
            """INSERT INTO bookings (cycle_id, renter_id, start_time, end_time)
               VALUES (%s, %s, %s, %s) RETURNING *""",
            (body.cycle_id, user["id"], body.start_time, body.end_time)
        ).fetchone()
        return row

@app.get("/bookings/mine")
def my_bookings(user=Depends(current_user)):
    with pool.connection() as conn:
        return conn.execute(
            """SELECT b.*, c.title as cycle_title, c.hostel
               FROM bookings b JOIN cycles c ON b.cycle_id = c.id
               WHERE b.renter_id = %s ORDER BY b.created_at DESC""",
            (user["id"],)
        ).fetchall()

@app.get("/bookings/incoming")
def incoming_bookings(user=Depends(current_user)):
    """Bookings on my cycles that I need to accept/reject."""
    with pool.connection() as conn:
        return conn.execute(
            """SELECT b.*, c.title as cycle_title, u.username as renter_name
               FROM bookings b
               JOIN cycles c ON b.cycle_id = c.id
               JOIN users u ON b.renter_id = u.id
               WHERE c.owner_id = %s AND b.status = 'requested'
               ORDER BY b.created_at DESC""",
            (user["id"],)
        ).fetchall()

@app.post("/bookings/{booking_id}/accept")
def accept_booking(booking_id: int, user=Depends(current_user)):
    code = new_code()
    with pool.connection() as conn:
        result = conn.execute(
            """UPDATE bookings b SET status = 'accepted', start_otp = %s
               FROM cycles c
               WHERE b.id = %s AND b.cycle_id = c.id
                 AND c.owner_id = %s AND b.status = 'requested'""",
            (hash_code(code, booking_id), booking_id, user["id"])
        )
        if result.rowcount == 0:
            raise HTTPException(409, "not allowed")
        # In production send OTP via SMS/email. For now return it.
        return {"ok": True, "start_otp": code}

@app.post("/bookings/{booking_id}/start")
def start_rental(booking_id: int, body: OTPSubmit, user=Depends(current_user)):
    """Renter submits OTP to start the ride."""
    with pool.connection() as conn:
        booking = conn.execute(
            "SELECT * FROM bookings WHERE id = %s AND renter_id = %s AND status = 'accepted'",
            (booking_id, user["id"])
        ).fetchone()
        if not booking:
            raise HTTPException(404, "booking not found")
        if not verify_code(body.code, booking["start_otp"], booking_id):
            raise HTTPException(403, "wrong OTP")
        end_code = new_code()
        conn.execute(
            "UPDATE bookings SET status = 'active', end_otp = %s WHERE id = %s",
            (hash_code(end_code, booking_id), booking_id)
        )
        conn.execute(
            "UPDATE cycles SET is_available = FALSE WHERE id = %s",
            (booking["cycle_id"],)
        )
        return {"ok": True, "end_otp": end_code}

@app.post("/bookings/{booking_id}/complete")
def complete_rental(booking_id: int, body: OTPSubmit, user=Depends(current_user)):
    """Owner submits end OTP to close the ride."""
    with pool.connection() as conn:
        booking = conn.execute(
            """SELECT b.* FROM bookings b
               JOIN cycles c ON b.cycle_id = c.id
               WHERE b.id = %s AND c.owner_id = %s AND b.status = 'active'""",
            (booking_id, user["id"])
        ).fetchone()
        if not booking:
            raise HTTPException(404, "booking not found")
        if not verify_code(body.code, booking["end_otp"], booking_id):
            raise HTTPException(403, "wrong OTP")
        conn.execute(
            "UPDATE bookings SET status = 'completed' WHERE id = %s", (booking_id,)
        )
        conn.execute(
            "UPDATE cycles SET is_available = TRUE WHERE id = %s",
            (booking["cycle_id"],)
        )
        return {"ok": True}

@app.post("/bookings/{booking_id}/cancel")
def cancel_booking(booking_id: int, user=Depends(current_user)):
    with pool.connection() as conn:
        result = conn.execute(
            """UPDATE bookings SET status = 'cancelled'
               WHERE id = %s AND renter_id = %s
                 AND status IN ('requested', 'accepted')""",
            (booking_id, user["id"])
        )
        if result.rowcount == 0:
            raise HTTPException(409, "cannot cancel")
        return {"ok": True}

# ── Condition ledger ───────────────────────────────────────────────────────────

@app.post("/bookings/{booking_id}/photos/{stage}")
def upload_photo(
    booking_id: int,
    stage: str,
    file: UploadFile = File(...),
    user=Depends(current_user)
):
    if stage not in ("pre", "post"):
        raise HTTPException(400, "stage must be pre or post")
    dest = UPLOAD_DIR / f"{booking_id}_{stage}_{file.filename}"
    with dest.open("wb") as f:
        shutil.copyfileobj(file.file, f)
    with pool.connection() as conn:
        row = conn.execute(
            """INSERT INTO condition_ledger (booking_id, stage, photo_path)
               VALUES (%s, %s, %s) RETURNING *""",
            (booking_id, stage, str(dest))
        ).fetchone()
        return row

@app.get("/bookings/{booking_id}/photos")
def get_photos(booking_id: int, user=Depends(current_user)):
    with pool.connection() as conn:
        return conn.execute(
            "SELECT * FROM condition_ledger WHERE booking_id = %s ORDER BY stage",
            (booking_id,)
        ).fetchall()
```

---

## Step 4 — Environment file

**`api/.env`**
```
DATABASE_URL=postgresql://postgres:yourpassword@localhost:5432/pedalpeer
JWT_SECRET=change-this-to-a-long-random-string
OTP_PEPPER=another-random-secret
WEB_ORIGIN=http://localhost:3000
```

Generate good secrets:
```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

---

## Step 5 — Frontend additions

**`web/.env.local`**
```
NEXT_PUBLIC_API_URL=http://localhost:8000
```

**`web/lib/api.ts`** — central API client
```ts
const BASE = process.env.NEXT_PUBLIC_API_URL ?? "http://localhost:8000";

function getToken() {
  return typeof window !== "undefined" ? localStorage.getItem("token") : null;
}

async function req(path: string, options: RequestInit = {}) {
  const token = getToken();
  const res = await fetch(`${BASE}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });
  if (!res.ok) {
    const err = await res.json().catch(() => ({}));
    throw new Error(err.detail ?? `HTTP ${res.status}`);
  }
  return res.json();
}

export const api = {
  signup: (data: object) =>
    req("/signup", { method: "POST", body: JSON.stringify(data) }),

  login: (data: object) =>
    req("/login", { method: "POST", body: JSON.stringify(data) }),

  me: () => req("/me"),

  cycles: {
    list: (hostel?: string) =>
      req(`/cycles${hostel ? `?hostel=${hostel}` : ""}`),
    create: (data: object) =>
      req("/cycles", { method: "POST", body: JSON.stringify(data) }),
    delete: (id: number) =>
      req(`/cycles/${id}`, { method: "DELETE" }),
  },

  bookings: {
    request: (data: object) =>
      req("/bookings", { method: "POST", body: JSON.stringify(data) }),
    mine: () => req("/bookings/mine"),
    incoming: () => req("/bookings/incoming"),
    accept: (id: number) =>
      req(`/bookings/${id}/accept`, { method: "POST" }),
    start: (id: number, code: string) =>
      req(`/bookings/${id}/start`, { method: "POST", body: JSON.stringify({ code }) }),
    complete: (id: number, code: string) =>
      req(`/bookings/${id}/complete`, { method: "POST", body: JSON.stringify({ code }) }),
    cancel: (id: number) =>
      req(`/bookings/${id}/cancel`, { method: "POST" }),
  },
};
```

**`web/lib/auth.tsx`** — auth context
```tsx
"use client";
import { createContext, useContext, useEffect, useState, ReactNode } from "react";
import { api } from "./api";

type User = { id: number; username: string; email: string } | null;
const AuthCtx = createContext<{
  user: User;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}>({ user: null, login: async () => {}, logout: () => {} });

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User>(null);

  useEffect(() => {
    const token = localStorage.getItem("token");
    if (token) api.me().then(setUser).catch(() => localStorage.removeItem("token"));
  }, []);

  async function login(email: string, password: string) {
    const data = await api.login({ email, password });
    localStorage.setItem("token", data.token);
    setUser(data.user);
  }

  function logout() {
    localStorage.removeItem("token");
    setUser(null);
  }

  return <AuthCtx.Provider value={{ user, login, logout }}>{children}</AuthCtx.Provider>;
}

export const useAuth = () => useContext(AuthCtx);
```

Add the provider to **`web/app/layout.tsx`**:
```tsx
import { AuthProvider } from "@/lib/auth";
// wrap children:
<AuthProvider>{children}</AuthProvider>
```

---

## Step 6 — Tests

**`api/tests/test_auth.py`**
```python
from fastapi.testclient import TestClient
import sys, os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))

from main import app

client = TestClient(app)

def test_health():
    assert client.get("/health").json() == {"ok": True}

def test_signup_and_login():
    # signup
    r = client.post("/signup", json={
        "username": "testuser",
        "email": "test@iitb.ac.in",
        "password": "password123"
    })
    assert r.status_code == 201
    token = r.json()["token"]

    # login
    r2 = client.post("/login", json={
        "email": "test@iitb.ac.in",
        "password": "password123"
    })
    assert r2.status_code == 200
    assert "token" in r2.json()

    # protected route
    r3 = client.get("/me", headers={"Authorization": f"Bearer {token}"})
    assert r3.status_code == 200
    assert r3.json()["email"] == "test@iitb.ac.in"
```

Run with:
```bash
cd api && pytest tests/ -v
```

---

## File structure after all this

```
pedalpeer/
├── api/
│   ├── main.py          ← all routes
│   ├── auth.py          ← JWT + bcrypt
│   ├── database.py      ← connection pool
│   ├── otp.py           ← OTP generation
│   ├── requirements.txt
│   ├── .env             ← never commit this
│   ├── uploads/         ← local photo storage
│   └── tests/
│       └── test_auth.py
├── db/
│   └── migrations/
│       ├── 001_init.sql
│       ├── 002_add_password_hash.sql
│       ├── 003_cycles.sql
│       ├── 004_bookings.sql
│       └── 005_condition_ledger.sql
└── web/
    ├── lib/
    │   ├── api.ts        ← API client
    │   └── auth.tsx      ← auth context
    ├── app/
    │   ├── layout.tsx    ← wrap with AuthProvider
    │   ├── signup/
    │   └── login/
    └── .env.local
```

Start everything:
```bash
# Terminal 1 — API
cd api && source .venv/bin/activate && uvicorn main:app --reload --port 8000

# Terminal 2 — Frontend  
cd web && npm run dev
```

Then open `http://localhost:8000/docs` to test all API endpoints interactively before touching the frontend.