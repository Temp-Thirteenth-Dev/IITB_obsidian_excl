FRONTEND

web/.env.local

NEXT_PUBLIC_API_URL=http://localhost:8000

web/lib/api.ts
```

ts
const BASE = process.env.NEXT_PUBLIC_API_URL ?? "http://localhost:8000";

function token() {
  return typeof window !== "undefined" ? localStorage.getItem("token") : null;
}

async function req(path: string, options: RequestInit = {}) {
  const t = token();
  const res = await fetch(`${BASE}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(t ? { Authorization: `Bearer ${t}` } : {}),
      ...options.headers,
    },
  });
  if (!res.ok) {
    const err = await res.json().catch(() => ({}));
    throw new Error(err.detail ?? `HTTP ${res.status}`);
  }
  return res.json();
}

async function upload(path: string, file: File, notes?: string) {
  const t = token();
  const fd = new FormData();
  fd.append("file", file);
  if (notes) fd.append("notes", notes);
  const res = await fetch(`${BASE}${path}`, {
    method: "POST",
    headers: t ? { Authorization: `Bearer ${t}` } : {},
    body: fd,
  });
  if (!res.ok) {
    const err = await res.json().catch(() => ({}));
    throw new Error(err.detail ?? `HTTP ${res.status}`);
  }
  return res.json();
}

export const api = {
  signup: (d: object) => req("/signup", { method: "POST", body: JSON.stringify(d) }),
  login:  (d: object) => req("/login",  { method: "POST", body: JSON.stringify(d) }),
  me: () => req("/me"),

  cycles: {
    list:   (hostel?: string) => req(`/cycles${hostel ? `?hostel=${hostel}` : ""}`),
    mine:   () => req("/cycles/mine"),
    get:    (id: number) => req(`/cycles/${id}`),
    create: (d: object) => req("/cycles", { method: "POST", body: JSON.stringify(d) }),
    toggle: (id: number) => req(`/cycles/${id}/toggle`, { method: "PATCH" }),
    delete: (id: number) => req(`/cycles/${id}`, { method: "DELETE" }),
  },

  bookings: {
    request:  (d: object) => req("/bookings", { method: "POST", body: JSON.stringify(d) }),
    mine:     () => req("/bookings/mine"),
    incoming: () => req("/bookings/incoming"),
    active:   () => req("/bookings/active-on-my-cycles"),
    get:      (id: number) => req(`/bookings/${id}`),
    accept:   (id: number) => req(`/bookings/${id}/accept`, { method: "POST" }),
    start:    (id: number, code: string) =>
      req(`/bookings/${id}/start`,    { method: "POST", body: JSON.stringify({ code }) }),
    complete: (id: number, code: string) =>
      req(`/bookings/${id}/complete`, { method: "POST", body: JSON.stringify({ code }) }),
    cancel:   (id: number) => req(`/bookings/${id}/cancel`, { method: "POST" }),
    photos:   (id: number) => req(`/bookings/${id}/photos`),
    uploadPhoto: (id: number, stage: "pre" | "post", file: File, notes?: string) =>
      upload(`/bookings/${id}/photos/${stage}`, file, notes),
  }
};
```

web/lib/auth.tsx

tsx
"use client";
import { createContext, useContext, useEffect, useState, ReactNode } from "react";
import { api } from "./api";

type User = { id: number; username: string; email: string } | null;

const AuthCtx = createContext<{
  user: User;
  loading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}>({ user: null, loading: true, login: async () => {}, logout: () => {} });

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const t = localStorage.getItem("token");
    if (t) {
      api.me().then(setUser).catch(() => localStorage.clear()).finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);

  async function login(email: string, password: string) {
    const data = await api.login({ email, password });
    localStorage.setItem("token", data.token);
    setUser(data.user);
  }

  function logout() {
    localStorage.clear();
    setUser(null);
  }

  return <AuthCtx.Provider value={{ user, loading, login, logout }}>{children}</AuthCtx.Provider>;
}

export const useAuth = () => useContext(AuthCtx);

web/app/layout.tsx

tsx
import type { Metadata } from "next";
import { Geist } from "next/font/google";
import "./globals.css";
import { AuthProvider } from "@/lib/auth";

const geist = Geist({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "PedalPeer",
  description: "Peer-to-peer cycle rentals at IIT Bombay",
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={geist.className}>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  );
}

web/app/page.tsx

tsx
import Link from "next/link";

export default function Home() {
  return (
    <main className="min-h-screen bg-gradient-to-br from-blue-600 to-blue-800 flex items-center justify-center px-4">
      <div className="text-center text-white">
        <p className="text-6xl mb-4">🚲</p>
        <h1 className="text-4xl font-bold mb-3">PedalPeer</h1>
        <p className="text-blue-100 mb-8 text-lg">Peer-to-peer cycle rentals at IIT Bombay</p>
        <div className="flex gap-4 justify-center">
          <Link href="/login"
            className="bg-white text-blue-700 font-semibold px-6 py-2.5 rounded-lg hover:bg-blue-50 transition">
            Log in
          </Link>
          <Link href="/signup"
            className="border-2 border-white text-white font-semibold px-6 py-2.5 rounded-lg hover:bg-white/10 transition">
            Sign up
          </Link>
        </div>
      </div>
    </main>
  );
}

web/app/signup/page.tsx

tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { api } from "@/lib/api";

export default function SignupPage() {
  const router = useRouter();
  const [form, setForm] = useState({ username: "", email: "", password: "" });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError(""); setLoading(true);
    try {
      const data = await api.signup(form);
      localStorage.setItem("token", data.token);
      router.push("/dashboard");
    } catch (err: any) {
      setError(err.message ?? "Signup failed");
    } finally { setLoading(false); }
  }

  return (
    <main className="min-h-screen flex items-center justify-center bg-gray-50 px-4">
      <div className="w-full max-w-sm bg-white rounded-2xl shadow p-8">
        <h1 className="text-2xl font-bold mb-1">Create account</h1>
        <p className="text-sm text-gray-500 mb-6">PedalPeer · IITB</p>
        {error && <div className="mb-4 px-3 py-2 rounded-lg bg-red-50 text-red-600 text-sm">{error}</div>}
        <form onSubmit={handleSubmit} className="space-y-4">
          {[
            { label: "Username", key: "username", type: "text",     placeholder: "e.g. debasish",        min: 3 },
            { label: "Email",    key: "email",    type: "email",    placeholder: "rollno@iitb.ac.in" },
            { label: "Password", key: "password", type: "password", placeholder: "min 6 characters",     min: 6 },
          ].map(f => (
            <div key={f.key}>
              <label className="block text-sm font-medium text-gray-700 mb-1">{f.label}</label>
              <input type={f.type} required minLength={f.min} placeholder={f.placeholder}
                value={(form as any)[f.key]}
                onChange={e => setForm(p => ({ ...p, [f.key]: e.target.value }))}
                className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
            </div>
          ))}
          <button type="submit" disabled={loading}
            className="w-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
            {loading ? "Creating…" : "Sign up"}
          </button>
        </form>
        <p className="mt-5 text-center text-sm text-gray-500">
          Already have an account?{" "}
          <Link href="/login" className="text-blue-600 hover:underline font-medium">Log in</Link>
        </p>
      </div>
    </main>
  );
}

web/app/login/page.tsx

tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { useAuth } from "@/lib/auth";

export default function LoginPage() {
  const { login } = useAuth();
  const router = useRouter();
  const [form, setForm] = useState({ email: "", password: "" });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError(""); setLoading(true);
    try {
      await login(form.email, form.password);
      router.push("/dashboard");
    } catch (err: any) {
      setError(err.message ?? "Login failed");
    } finally { setLoading(false); }
  }

  return (
    <main className="min-h-screen flex items-center justify-center bg-gray-50 px-4">
      <div className="w-full max-w-sm bg-white rounded-2xl shadow p-8">
        <h1 className="text-2xl font-bold mb-1">Welcome back</h1>
        <p className="text-sm text-gray-500 mb-6">PedalPeer · IITB</p>
        {error && <div className="mb-4 px-3 py-2 rounded-lg bg-red-50 text-red-600 text-sm">{error}</div>}
        <form onSubmit={handleSubmit} className="space-y-4">
          {[
            { label: "Email",    key: "email",    type: "email",    placeholder: "rollno@iitb.ac.in" },
            { label: "Password", key: "password", type: "password", placeholder: "your password" },
          ].map(f => (
            <div key={f.key}>
              <label className="block text-sm font-medium text-gray-700 mb-1">{f.label}</label>
              <input type={f.type} required placeholder={f.placeholder}
                value={(form as any)[f.key]}
                onChange={e => setForm(p => ({ ...p, [f.key]: e.target.value }))}
                className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
            </div>
          ))}
          <button type="submit" disabled={loading}
            className="w-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
            {loading ? "Logging in…" : "Log in"}
          </button>
        </form>
        <p className="mt-5 text-center text-sm text-gray-500">
          No account?{" "}
          <Link href="/signup" className="text-blue-600 hover:underline font-medium">Sign up</Link>
        </p>
      </div>
    </main>
  );
}

web/app/dashboard/page.tsx

tsx
"use client";
import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { useAuth } from "@/lib/auth";
import { api } from "@/lib/api";

const STATUS_STYLE: Record<string, string> = {
  requested: "bg-yellow-100 text-yellow-700",
  accepted:  "bg-blue-100 text-blue-700",
  active:    "bg-green-100 text-green-700",
  completed: "bg-gray-100 text-gray-500",
  cancelled: "bg-red-100 text-red-400",
};

export default function Dashboard() {
  const { user, loading: authLoading, logout } = useAuth();
  const router = useRouter();
  const [cycles, setCycles]     = useState<any[]>([]);
  const [bookings, setBookings] = useState<any[]>([]);
  const [incoming, setIncoming] = useState<any[]>([]);
  const [active, setActive]     = useState<any[]>([]);
  const [ready, setReady]       = useState(false);

  useEffect(() => {
    if (authLoading) return;
    if (!user) { router.push("/login"); return; }
    Promise.all([
      api.cycles.list(),
      api.bookings.mine(),
      api.bookings.incoming(),
      api.bookings.active(),
    ]).then(([c, b, i, a]) => {
      setCycles(c); setBookings(b); setIncoming(i); setActive(a);
    }).finally(() => setReady(true));
  }, [user, authLoading]);

  if (!ready) return <div className="min-h-screen flex items-center justify-center text-gray-400">Loading…</div>;

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white border-b px-6 py-3 flex items-center justify-between sticky top-0 z-10">
        <span className="font-bold text-blue-600 text-lg">🚲 PedalPeer</span>
        <div className="flex items-center gap-3">
          <span className="text-sm text-gray-500 hidden sm:block">{user?.username}</span>
          <Link href="/cycles/new"
            className="bg-blue-600 text-white text-sm px-3 py-1.5 rounded-lg hover:bg-blue-700">
            + List cycle
          </Link>
          <Link href="/my-cycles"
            className="text-sm text-gray-600 hover:text-blue-600 border px-3 py-1.5 rounded-lg">
            My cycles
          </Link>
          <button onClick={() => { logout(); router.push("/"); }}
            className="text-sm text-gray-400 hover:text-red-500">
            Logout
          </button>
        </div>
      </nav>

      <div className="max-w-4xl mx-auto px-4 py-8 space-y-10">

        {/* Incoming requests */}
        {incoming.length > 0 && (
          <section>
            <h2 className="font-semibold text-lg mb-3 text-orange-600">⏳ Booking requests on your cycles</h2>
            <div className="space-y-3">
              {incoming.map((b: any) => (
                <div key={b.id} className="bg-white rounded-xl border p-4 flex items-center justify-between">
                  <div>
                    <p className="font-medium">{b.cycle_title}</p>
                    <p className="text-sm text-gray-500">
                      {b.renter_name} · {new Date(b.start_time).toLocaleDateString()} – {new Date(b.end_time).toLocaleDateString()}
                    </p>
                  </div>
                  <Link href={`/bookings/${b.id}/accept`}
                    className="bg-green-600 text-white text-sm px-3 py-1.5 rounded-lg hover:bg-green-700">
                    Review
                  </Link>
                </div>
              ))}
            </div>
          </section>
        )}

        {/* Active rides on my cycles — owner can complete */}
        {active.length > 0 && (
          <section>
            <h2 className="font-semibold text-lg mb-3 text-green-700">🟢 Active rides on your cycles</h2>
            <div className="space-y-3">
              {active.map((b: any) => (
                <div key={b.id} className="bg-white rounded-xl border p-4 flex items-center justify-between">
                  <div>
                    <p className="font-medium">{b.cycle_title}</p>
                    <p className="text-sm text-gray-500">Rented by {b.renter_name}</p>
                  </div>
                  <Link href={`/bookings/${b.id}/complete`}
                    className="bg-blue-600 text-white text-sm px-3 py-1.5 rounded-lg hover:bg-blue-700">
                    Complete
                  </Link>
                </div>
              ))}
            </div>
          </section>
        )}

        {/* My bookings */}
        <section>
          <h2 className="font-semibold text-lg mb-3">My Rentals</h2>
          {bookings.length === 0
            ? <p className="text-gray-400 text-sm">No bookings yet. <Link href="/cycles" className="text-blue-500 underline">Browse cycles →</Link></p>
            : <div className="space-y-3">
                {bookings.map((b: any) => (
                  <div key={b.id} className="bg-white rounded-xl border p-4 flex items-center justify-between">
                    <div>
                      <p className="font-medium">{b.cycle_title}</p>
                      <p className="text-sm text-gray-500">
                        {b.hostel} · {new Date(b.start_time).toLocaleDateString()}
                      </p>
                      <span className={`text-xs px-2 py-0.5 rounded-full font-medium mt-1 inline-block ${STATUS_STYLE[b.status]}`}>
                        {b.status}
                      </span>
                    </div>
                    {(b.status === "accepted" || b.status === "active") && (
                      <Link href={`/bookings/${b.id}`}
                        className="text-sm text-blue-600 border border-blue-300 px-3 py-1.5 rounded-lg hover:bg-blue-50">
                        Manage
                      </Link>
                    )}
                  </div>
                ))}
              </div>
          }
        </section>

        {/* Browse cycles */}
        <section>
          <div className="flex items-center justify-between mb-3">
            <h2 className="font-semibold text-lg">Available Cycles</h2>
            <Link href="/cycles" className="text-sm text-blue-600 hover:underline">See all →</Link>
          </div>
          {cycles.length === 0
            ? <p className="text-gray-400 text-sm">No cycles listed yet.</p>
            : <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                {cycles.slice(0, 4).map((c: any) => (
                  <Link key={c.id} href={`/cycles/${c.id}`}
                    className="bg-white rounded-xl border p-4 hover:shadow-md transition block">
                    <div className="flex justify-between">
                      <div>
                        <p className="font-medium">{c.title}</p>
                        <p className="text-sm text-gray-500">{c.hostel ?? "—"}</p>
                      </div>
                      <span className="text-blue-600 font-semibold text-sm">₹{c.price_per_hour}/hr</span>
                    </div>
                    <p className="text-xs text-gray-400 mt-3">by {c.owner_name}</p>
                  </Link>
                ))}
              </div>
          }
        </section>
      </div>
    </main>
  );
}

web/app/cycles/page.tsx

tsx
"use client";
import { useEffect, useState } from "react";
import Link from "next/link";
import { api } from "@/lib/api";

const HOSTELS = ["H1","H2","H3","H4","H5","H6","H7","H8","H9","H10","H11","H12","H13","H14","H15","H16","Tansa","Qip"];

export default function CyclesPage() {
  const [cycles, setCycles] = useState<any[]>([]);
  const [hostel, setHostel] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    setLoading(true);
    api.cycles.list(hostel || undefined).then(setCycles).finally(() => setLoading(false));
  }, [hostel]);

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white border-b px-6 py-3 flex items-center gap-4">
        <Link href="/dashboard" className="text-blue-600 font-bold text-lg">🚲 PedalPeer</Link>
        <span className="text-gray-300">|</span>
        <span className="text-gray-600 text-sm">Browse Cycles</span>
      </nav>
      <div className="max-w-4xl mx-auto px-4 py-8">
        <div className="flex flex-wrap gap-2 mb-6">
          <button onClick={() => setHostel("")}
            className={`px-3 py-1 rounded-full text-sm border transition ${!hostel ? "bg-blue-600 text-white border-blue-600" : "bg-white text-gray-600 hover:border-blue-400"}`}>
            All
          </button>
          {HOSTELS.map(h => (
            <button key={h} onClick={() => setHostel(h)}
              className={`px-3 py-1 rounded-full text-sm border transition ${hostel === h ? "bg-blue-600 text-white border-blue-600" : "bg-white text-gray-600 hover:border-blue-400"}`}>
              {h}
            </button>
          ))}
        </div>
        {loading
          ? <p className="text-gray-400 text-sm">Loading…</p>
          : cycles.length === 0
            ? <p className="text-gray-400 text-sm">No cycles available{hostel ? ` in ${hostel}` : ""}.</p>
            : <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                {cycles.map(c => (
                  <Link key={c.id} href={`/cycles/${c.id}`}
                    className="bg-white rounded-xl border p-4 hover:shadow-md transition block">
                    <div className="flex justify-between">
                      <div>
                        <p className="font-medium">{c.title}</p>
                        <p className="text-sm text-gray-500">{c.hostel ?? "—"}</p>
                        {c.description && <p className="text-xs text-gray-400 mt-1 line-clamp-2">{c.description}</p>}
                      </div>
                      <span className="text-blue-600 font-semibold text-sm whitespace-nowrap">₹{c.price_per_hour}/hr</span>
                    </div>
                    <p className="text-xs text-gray-400 mt-3">by {c.owner_name}</p>
                  </Link>
                ))}
              </div>
        }
      </div>
    </main>
  );
}

web/app/cycles/new/page.tsx

tsx
"use client";
import { useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { api } from "@/lib/api";

const HOSTELS = ["H1","H2","H3","H4","H5","H6","H7","H8","H9","H10","H11","H12","H13","H14","H15","H16","Tansa","Qip"];
const cls = "w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500";

export default function NewCyclePage() {
  const router = useRouter();
  const [form, setForm] = useState({ title: "", description: "", price_per_hour: "", hostel: "" });
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  const set = (k: string) => (e: any) => setForm(f => ({ ...f, [k]: e.target.value }));

  async function submit(e: React.FormEvent) {
    e.preventDefault(); setError(""); setLoading(true);
    try {
      await api.cycles.create({ ...form, price_per_hour: Number(form.price_per_hour) });
      router.push("/dashboard");
    } catch (err: any) { setError(err.message ?? "Failed"); }
    finally { setLoading(false); }
  }

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white border-b px-6 py-3 flex items-center gap-4">
        <Link href="/dashboard" className="text-blue-600 font-bold text-lg">🚲 PedalPeer</Link>
        <span className="text-gray-300">|</span>
        <span className="text-gray-600 text-sm">List a Cycle</span>
      </nav>
      <div className="max-w-lg mx-auto px-4 py-10">
        <div className="bg-white rounded-2xl shadow p-8">
          <h1 className="text-xl font-bold mb-6">List your cycle</h1>
          {error && <div className="mb-4 px-3 py-2 rounded-lg bg-red-50 text-red-600 text-sm">{error}</div>}
          <form onSubmit={submit} className="space-y-4">
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Title <span className="text-red-500">*</span></label>
              <input type="text" required minLength={3} value={form.title} onChange={set("title")}
                placeholder="e.g. Firefox MTB, Hero Sprint" className={cls} />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Description</label>
              <textarea rows={3} value={form.description} onChange={set("description")}
                placeholder="Condition, gear count, any notes…" className={cls} />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Price per hour (₹) <span className="text-red-500">*</span></label>
              <input type="number" required min={0} value={form.price_per_hour} onChange={set("price_per_hour")}
                placeholder="0 for free" className={cls} />
            </div>
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Hostel</label>
              <select value={form.hostel} onChange={set("hostel")} className={cls}>
                <option value="">Select hostel</option>
                {HOSTELS.map(h => <option key={h}>{h}</option>)}
              </select>
            </div>
            <button type="submit" disabled={loading}
              className="w-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
              {loading ? "Listing…" : "List Cycle"}
            </button>
          </form>
        </div>
      </div>
    </main>
  );
}

web/app/cycles/[id]/page.tsx

tsx
"use client";
import { useEffect, useState } from "react";
import { useParams, useRouter } from "next/navigation";
import Link from "next/link";
import { useAuth } from "@/lib/auth";
import { api } from "@/lib/api";

const cls = "w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500";

export default function CycleDetail() {
  const { id } = useParams();
  const router = useRouter();
  const { user } = useAuth();
  const [cycle, setCycle] = useState<any>(null);
  const [form, setForm] = useState({ start_time: "", end_time: "" });
  const [msg, setMsg] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    api.cycles.get(Number(id)).then(setCycle).catch(() => router.push("/cycles"));
  }, [id]);

  async function book(e: React.FormEvent) {
    e.preventDefault(); setMsg(""); setLoading(true);
    try {
      await api.bookings.request({ cycle_id: Number(id), ...form });
      setMsg("✅ Booking requested! Owner will accept soon.");
    } catch (err: any) { setMsg("❌ " + (err.message ?? "Failed")); }
    finally { setLoading(false); }
  }

  if (!cycle) return <div className="min-h-screen flex items-center justify-center text-gray-400">Loading…</div>;

  const isOwner = user?.id === cycle.owner_id;

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white border-b px-6 py-3 flex items-center gap-4">
        <Link href="/dashboard" className="text-blue-600 font-bold text-lg">🚲 PedalPeer</Link>
        <span className="text-gray-300">|</span>
        <Link href="/cycles" className="text-gray-500 text-sm hover:text-blue-600">Browse</Link>
      </nav>
      <div className="max-w-2xl mx-auto px-4 py-10 space-y-6">
        <div className="bg-white rounded-2xl shadow p-6">
          <div className="flex items-start justify-between">
            <div>
              <h1 className="text-2xl font-bold">{cycle.title}</h1>
              <p className="text-gray-500 mt-1">{cycle.hostel ?? "No hostel set"} · by {cycle.owner_name}</p>
            </div>
            <span className="text-2xl font-bold text-blue-600">
              ₹{cycle.price_per_hour}<span className="text-sm font-normal text-gray-400">/hr</span>
            </span>
          </div>
          {cycle.description && <p className="mt-4 text-gray-600 text-sm leading-relaxed">{cycle.description}</p>}
          <div className="mt-4">
            <span className={`text-xs px-2 py-1 rounded-full font-medium ${cycle.is_available ? "bg-green-100 text-green-700" : "bg-red-100 text-red-500"}`}>
              {cycle.is_available ? "Available" : "Not available"}
            </span>
          </div>
        </div>

        {isOwner
          ? <div className="bg-yellow-50 border border-yellow-200 rounded-xl p-4 text-sm text-yellow-700">
              This is your cycle.{" "}
              <Link href="/my-cycles" className="underline">Manage your cycles →</Link>
            </div>
          : cycle.is_available
            ? <div className="bg-white rounded-2xl shadow p-6">
                <h2 className="font-semibold mb-4">Request a booking</h2>
                <form onSubmit={book} className="space-y-4">
                  <div className="grid grid-cols-2 gap-4">
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">Start</label>
                      <input type="datetime-local" required value={form.start_time}
                        onChange={e => setForm(f => ({ ...f, start_time: e.target.value }))} className={cls} />
                    </div>
                    <div>
                      <label className="block text-sm font-medium text-gray-700 mb-1">End</label>
                      <input type="datetime-local" required value={form.end_time}
                        onChange={e => setForm(f => ({ ...f, end_time: e.target.value }))} className={cls} />
                    </div>
                  </div>
                  {msg && <p className="text-sm">{msg}</p>}
                  <button type="submit" disabled={loading}
                    className="w-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
                    {loading ? "Requesting…" : "Request Booking"}
                  </button>
                </form>
              </div>
            : <div className="bg-gray-50 border rounded-xl p-4 text-sm text-gray-500">
                This cycle is currently unavailable.
              </div>
        }
      </div>
    </main>
  );
}

web/app/my-cycles/page.tsx

tsx
"use client";
import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { useAuth } from "@/lib/auth";
import { api } from "@/lib/api";

export default function MyCycles() {
  const { user, loading: authLoading } = useAuth();
  const router = useRouter();
  const [cycles, setCycles] = useState<any[]>([]);
  const [ready, setReady] = useState(false);

  useEffect(() => {
    if (authLoading) return;
    if (!user) { router.push("/login"); return; }
    api.cycles.mine().then(setCycles).finally(() => setReady(true));
  }, [user, authLoading]);

  async function toggle(id: number) {
    const updated = await api.cycles.toggle(id);
    setCycles(cs => cs.map(c => c.id === id ? { ...c, is_available: updated.is_available } : c));
  }

  async function remove(id: number) {
    if (!confirm("Delete this cycle?")) return;
    await api.cycles.delete(id);
    setCycles(cs => cs.filter(c => c.id !== id));
  }

  if (!ready) return <div className="min-h-screen flex items-center justify-center text-gray-400">Loading…</div>;

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white border-b px-6 py-3 flex items-center gap-4">
        <Link href="/dashboard" className="text-blue-600 font-bold text-lg">🚲 PedalPeer</Link>
        <span className="text-gray-300">|</span>
        <span className="text-gray-600 text-sm">My Cycles</span>
        <div className="ml-auto">
          <Link href="/cycles/new" className="bg-blue-600 text-white text-sm px-3 py-1.5 rounded-lg hover:bg-blue-700">
            + List new
          </Link>
        </div>
      </nav>
      <div className="max-w-3xl mx-auto px-4 py-8">
        {cycles.length === 0
          ? <div className="text-center py-16">
              <p className="text-4xl mb-3">🚲</p>
              <p className="text-gray-500 mb-4">You haven't listed any cycles yet.</p>
              <Link href="/cycles/new" className="bg-blue-600 text-white px-4 py-2 rounded-lg text-sm hover:bg-blue-700">
                List your first cycle
              </Link>
            </div>
          : <div className="space-y-4">
              {cycles.map(c => (
                <div key={c.id} className="bg-white rounded-xl border p-5 flex items-start justify-between">
                  <div className="flex-1">
                    <div className="flex items-center gap-2">
                      <p className="font-semibold">{c.title}</p>
                      <span className={`text-xs px-2 py-0.5 rounded-full font-medium ${c.is_available ? "bg-green-100 text-green-700" : "bg-red-100 text-red-500"}`}>
                        {c.is_available ? "Available" : "Unavailable"}
                      </span>
                    </div>
                    <p className="text-sm text-gray-500 mt-0.5">{c.hostel ?? "No hostel"} · ₹{c.price_per_hour}/hr</p>
                    {c.description && <p className="text-xs text-gray-400 mt-1 line-clamp-2">{c.description}</p>}
                  </div>
                  <div className="flex gap-2 ml-4 shrink-0">
                    <button onClick={() => toggle(c.id)}
                      className="text-xs border px-2 py-1 rounded-lg text-gray-600 hover:border-blue-400 hover:text-blue-600">
                      {c.is_available ? "Mark unavailable" : "Mark available"}
                    </button>
                    <button onClick={() => remove(c.id)}
                      className="text-xs border border-red-200 px-2 py-1 rounded-lg text-red-500 hover:bg-red-50">
                      Delete
                    </button>
                  </div>
                </div>
              ))}
            </div>
        }
      </div>
    </main>
  );
}

web/app/bookings/[id]/accept/page.tsx

tsx
"use client";
import { useEffect, useState } from "react";
import { useParams, useRouter } from "next/navigation";
import Link from "next/link";
import { api } from "@/lib/api";

export default function AcceptBooking() {
  const { id } = useParams();
  const router = useRouter();
  const [booking, setBooking] = useState<any>(null);
  const [otp, setOtp] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    api.bookings.get(Number(id)).then(setBooking).catch(() => router.push("/dashboard"));
  }, [id]);

  async function accept() {
    setLoading(true);
    try {
      const data = await api.bookings.accept(Number(id));
      setOtp(data.start_otp);
    } catch (err: any) { alert(err.message); }
    finally { setLoading(false); }
  }

  async function decline() {
    if (!confirm("Decline this request?")) return;
    await api.bookings.cancel(Number(id));
    router.push("/dashboard");
  }

  if (!booking) return <div className="min-h-screen flex items-center justify-center text-gray-400">Loading…</div>;

  return (
    <main className="min-h-screen bg-gray-50 flex items-center justify-center px-4">
      <div className="w-full max-w-sm bg-white rounded-2xl shadow p-8">
        {otp
          ? <>
              <h1 className="text-xl font-bold mb-2">Accepted ✅</h1>
              <p className="text-gray-500 text-sm mb-6">Share this start OTP with the renter in person.</p>
              <div className="bg-blue-50 border border-blue-200 rounded-xl p-6 text-center mb-4">
                <p className="text-4xl font-mono font-bold tracking-widest text-blue-700">{otp}</p>
                <p className="text-xs text-blue-400 mt-2">Start OTP — note this down, it won't show again</p>
              </div>
              <Link href="/dashboard"
                className="block text-center bg-blue-600 text-white rounded-lg py-2 text-sm font-semibold hover:bg-blue-700">
                Back to Dashboard
              </Link>
            </>
          : <>
              <h1 className="text-xl font-bold mb-2">Booking Request</h1>
              <div className="bg-gray-50 rounded-xl p-4 mb-6 space-y-2 text-sm">
                <p><span className="text-gray-500">Cycle:</span> <strong>{booking.cycle_title}</strong></p>
                <p><span className="text-gray-500">Renter:</span> {booking.renter_name}</p>
                <p><span className="text-gray-500">From:</span> {new Date(booking.start_time).toLocaleString()}</p>
                <p><span className="text-gray-500">To:</span> {new Date(booking.end_time).toLocaleString()}</p>
              </div>
              <div className="flex gap-3">
                <button onClick={decline}
                  className="flex-1 border border-red-300 text-red-600 rounded-lg py-2 text-sm font-semibold hover:bg-red-50">
                  Decline
                </button>
                <button onClick={accept} disabled={loading}
                  className="flex-1 bg-green-600 text-white rounded-lg py-2 text-sm font-semibold hover:bg-green-700 disabled:opacity-50">
                  {loading ? "Accepting…" : "Accept"}
                </button>
              </div>
            </>
        }
      </div>
    </main>
  );
}

web/app/bookings/[id]/page.tsx — renter OTP entry

tsx
"use client";
import { useEffect, useState } from "react";
import { useParams, useRouter } from "next/navigation";
import Link from "next/link";
import { api } from "@/lib/api";

const STATUS_MSG: Record<string, string> = {
  requested: "Waiting for the owner to accept your request…",
  completed: "This ride is complete. ✅",
  cancelled: "This booking was cancelled.",
};

export default function ManageBooking() {
  const { id } = useParams();
  const router = useRouter();
  const [booking, setBooking] = useState<any>(null);
  const [code, setCode] = useState("");
  const [msg, setMsg] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    api.bookings.get(Number(id)).then(setBooking).catch(() => router.push("/dashboard"));
  }, [id]);

  async function submitOtp() {
    if (code.length !== 6) { setMsg("Enter the 6-digit OTP"); return; }
    setMsg(""); setLoading(true);
    const isAccepted = booking.status === "accepted";
    try {
      if (isAccepted) {
        const data = await api.bookings.start(Number(id), code);
        setMsg(`✅ Ride started! End OTP: ${data.end_otp} — show this to the owner when done.`);
        setBooking((b: any) => ({ ...b, status: "active" }));
      } else {
        await api.bookings.complete(Number(id), code);
        setMsg("✅ Ride complete!");
        setTimeout(() => router.push("/dashboard"), 1500);
      }
    } catch (err: any) { setMsg("❌ " + (err.message ?? "Wrong OTP")); }
    finally { setLoading(false); setCode(""); }
  }

  if (!booking) return <div className="min-h-screen flex items-center justify-center text-gray-400">Loading…</div>;

  const isAccepted = booking.status === "accepted";
  const isActive   = booking.status === "active";
  const needsOtp   = isAccepted || isActive;

  return (
    <main className="min-h-screen bg-gray-50 flex items-center justify-center px-4">
      <div className="w-full max-w-sm bg-white rounded-2xl shadow p-8">
        <Link href="/dashboard" className="text-sm text-blue-600 hover:underline">← Dashboard</Link>
        <h1 className="text-xl font-bold mt-4 mb-1">{booking.cycle_title}</h1>
        <p className="text-gray-500 text-sm mb-4">{booking.hostel}</p>
        <span className={`text-xs px-2 py-1 rounded-full font-medium ${
          isActive ? "bg-green-100 text-green-700" :
          isAccepted ? "bg-blue-100 text-blue-700" : "bg-gray-100 text-gray-500"
        }`}>{booking.status}</span>

        {needsOtp && (
          <div className="mt-6">
            <p className="text-sm text-gray-600 mb-3">
              {isAccepted
                ? "Enter the start OTP from the owner to begin your ride."
                : "Enter the end OTP from the owner to complete your ride."}
            </p>
            <input
              type="text" inputMode="numeric" maxLength={6} value={code}
              onChange={e => setCode(e.target.value.replace(/\D/g, ""))}
              placeholder="000000"
              className="w-full border border-gray-300 rounded-xl px-4 py-4 text-center text-3xl font-mono tracking-[0.5em] focus:outline-none focus:ring-2 focus:ring-blue-500 mb-4"
            />
            {msg && <p className="text-sm mb-3 break-words">{msg}</p>}
            <button onClick={submitOtp} disabled={loading || code.length !== 6}
              className="w-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
              {loading ? "Verifying…" : isAccepted ? "Start Ride" : "Complete Ride"}
            </button>
            {isAccepted && (
              <button onClick={() => api.bookings.cancel(Number(id)).then(() => router.push("/dashboard"))}
                className="w-full mt-3 text-sm text-red-500 hover:underline">
                Cancel booking
              </button>
            )}
          </div>
        )}

        {!needsOtp && (
          <p className="mt-6 text-sm text-gray-500">{STATUS_MSG[booking.status] ?? ""}</p>
        )}
      </div>
    </main>
  );
}

web/app/bookings/[id]/complete/page.tsx — owner enters end OTP

tsx
"use client";
import { useState } from "react";
import { useParams, useRouter } from "next/navigation";
import Link from "next/link";
import { api } from "@/lib/api";

export default function CompleteBooking() {
  const { id } = useParams();
  const router = useRouter();
  const [code, setCode] = useState("");
  const [msg, setMsg] = useState("");
  const [loading, setLoading] = useState(false);

  async function submit(e: React.FormEvent) {
    e.preventDefault(); setMsg(""); setLoading(true);
    try {
      await api.bookings.complete(Number(id), code);
      setMsg("✅ Booking completed!");
      setTimeout(() => router.push("/dashboard"), 1500);
    } catch (err: any) { setMsg("❌ " + (err.message ?? "Wrong OTP")); }
    finally { setLoading(false); }
  }

  return (
    <main className="min-h-screen bg-gray-50 flex items-center justify-center px-4">
      <div className="w-full max-w-sm bg-white rounded-2xl shadow p-8">
        <Link href="/dashboard" className="text-sm text-blue-600 hover:underline">← Dashboard</Link>
        <h1 className="text-xl font-bold mt-4 mb-2">Complete Ride</h1>
        <p className="text-sm text-gray-500 mb-6">The renter will show you a 6-digit end OTP. Enter it to close the booking.</p>
        <form onSubmit={submit} className="space-y-4">
          <input
            type="text" inputMode="numeric" maxLength={6} value={code}
            onChange={e => setCode(e.target.value.replace(/\D/g, ""))}
            placeholder="000000"
            className="w-full border border-gray-300 rounded-xl px-4 py-4 text-center text-3xl font-mono tracking-[0.5em] focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          {msg && <p className="text-sm">{msg}</p>}
          <button type="submit" disabled={loading || code.length !== 6}
            className="w-full bg-green-600 hover:bg-green-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
            {loading ? "Verifying…" : "Mark Complete"}
          </button>
        </form>
      </div>
    </main>
  );
}

web/app/bookings/[id]/photos/page.tsx — condition photo upload

tsx
"use client";
import { useEffect, useRef, useState } from "react";
import { useParams, useRouter } from "next/navigation";
import Link from "next/link";
import { api } from "@/lib/api";

export default function PhotoUpload() {
  const { id } = useParams();
  const router = useRouter();
  const [photos, setPhotos] = useState<any[]>([]);
  const [stage, setStage] = useState<"pre" | "post">("pre");
  const [notes, setNotes] = useState("");
  const [preview, setPreview] = useState<string | null>(null);
  const [file, setFile] = useState<File | null>(null);
  const [msg, setMsg] = useState("");
  const [loading, setLoading] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    api.bookings.photos(Number(id)).then(setPhotos).catch(() => router.push("/dashboard"));
  }, [id]);

  function pickFile(e: React.ChangeEvent<HTMLInputElement>) {
    const f = e.target.files?.[0];
    if (!f) return;
    setFile(f);
    setPreview(URL.createObjectURL(f));
  }

  async function upload(e: React.FormEvent) {
    e.preventDefault();
    if (!file) { setMsg("Choose a photo first"); return; }
    setMsg(""); setLoading(true);
    try {
      const row = await api.bookings.uploadPhoto(Number(id), stage, file, notes || undefined);
      setPhotos(ps => {
        const rest = ps.filter(p => p.stage !== stage);
        return [...rest, row].sort((a, b) => a.stage.localeCompare(b.stage));
      });
      setMsg("✅ Photo saved!");
      setFile(null); setPreview(null); setNotes("");
      if (inputRef.current) inputRef.current.value = "";
    } catch (err: any) { setMsg("❌ " + (err.message ?? "Upload failed")); }
    finally { setLoading(false); }
  }

  const BASE = process.env.NEXT_PUBLIC_API_URL ?? "http://localhost:8000";

  return (
    <main className="min-h-screen bg-gray-50">
      <nav className="bg-white border-b px-6 py-3 flex items-center gap-4">
        <Link href="/dashboard" className="text-blue-600 font-bold text-lg">🚲 PedalPeer</Link>
        <span className="text-gray-300">|</span>
        <span className="text-gray-600 text-sm">Condition Photos</span>
      </nav>

      <div className="max-w-xl mx-auto px-4 py-8 space-y-6">

        {/* Existing photos */}
        {photos.length > 0 && (
          <div className="bg-white rounded-2xl shadow p-6">
            <h2 className="font-semibold mb-4">Saved photos</h2>
            <div className="grid grid-cols-2 gap-4">
              {photos.map(p => (
                <div key={p.id} className="border rounded-xl overflow-hidden">
                  <img src={`${BASE}${p.photo_path}`} alt={p.stage}
                    className="w-full h-36 object-cover" />
                  <div className="p-2">
                    <span className={`text-xs font-medium px-2 py-0.5 rounded-full ${p.stage === "pre" ? "bg-blue-100 text-blue-700" : "bg-orange-100 text-orange-700"}`}>
                      {p.stage === "pre" ? "Pre-ride" : "Post-ride"}
                    </span>
                    {p.notes && <p className="text-xs text-gray-400 mt-1">{p.notes}</p>}
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* Upload form */}
        <div className="bg-white rounded-2xl shadow p-6">
          <h2 className="font-semibold mb-4">Upload photo</h2>
          <form onSubmit={upload} className="space-y-4">
            <div className="flex gap-3">
              {(["pre", "post"] as const).map(s => (
                <button key={s} type="button" onClick={() => setStage(s)}
                  className={`flex-1 py-2 rounded-lg text-sm font-medium border transition ${
                    stage === s ? "bg-blue-600 text-white border-blue-600" : "bg-white text-gray-600 hover:border-blue-400"
                  }`}>
                  {s === "pre" ? "Pre-ride" : "Post-ride"}
                </button>
              ))}
            </div>

            <div
              onClick={() => inputRef.current?.click()}
              className="border-2 border-dashed border-gray-300 rounded-xl p-6 text-center cursor-pointer hover:border-blue-400 transition">
              {preview
                ? <img src={preview} alt="preview" className="max-h-48 mx-auto rounded-lg object-cover" />
                : <div className="text-gray-400">
                    <p className="text-3xl mb-2">📷</p>
                    <p className="text-sm">Click to choose a photo</p>
                  </div>
              }
              <input ref={inputRef} type="file" accept="image/*" onChange={pickFile} className="hidden" />
            </div>

            <div>
              <label className="block text-sm font-medium text-gray-700 mb-1">Notes (optional)</label>
              <textarea rows={2} value={notes} onChange={e => setNotes(e.target.value)}
                placeholder="e.g. scratch on left pedal, tyre pressure low"
                className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500" />
            </div>

            {msg && <p className="text-sm">{msg}</p>}

            <button type="submit" disabled={loading || !file}
              className="w-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 text-white font-semibold rounded-lg py-2 text-sm transition">
              {loading ? "Uploading…" : "Save photo"}
            </button>
          </form>
        </div>

        <Link href="/dashboard" className="block text-center text-sm text-gray-500 hover:text-blue-600">
          ← Back to dashboard
        </Link>
      </div>
    </main>
  );
}
Final file tree
pedalpeer/
├── api/
│   ├── main.py
│   ├── auth.py
│   ├── database.py
│   ├── otp.py
│   ├── requirements.txt
│   └── .env
├── db/migrations/
│   ├── 001_init.sql
│   ├── 002_cycles.sql
│   ├── 003_bookings.sql
│   └── 004_condition_ledger.sql
└── web/
    ├── .env.local
    ├── lib/
    │   ├── api.ts
    │   └── auth.tsx
    └── app/
        ├── layout.tsx
        ├── page.tsx
        ├── login/page.tsx
        ├── signup/page.tsx
        ├── dashboard/page.tsx
        ├── cycles/
        │   ├── page.tsx
        │   ├── new/page.tsx
        │   └── [id]/page.tsx
        ├── my-cycles/page.tsx
        └── bookings/[id]/
            ├── page.tsx
            ├── accept/page.tsx
            ├── complete/page.tsx
            └── photos/page.tsx
