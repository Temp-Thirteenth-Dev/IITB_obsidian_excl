# CS 699 — Lecture 4 Study Notes
## Regular Expressions, Shell Scripting (`sed` / `awk`) and Python

**Source:** CS 699 – Lec 4, Om Damani, CSE IIT Bombay  
**Designed for:** Lecture revision • Quiz preparation • Lab-test preparation

---

# 0. Lecture Map

## Part I — Shell / Text Processing
- Shell functions
- Regular expressions
- `grep`, `sed`, `awk`
- `sed` substitution, printing, deletion, append, quit
- `sed` capture groups and backreferences
- AWK fields and patterns
- AWK built-in variables
- AWK associative arrays
- Practical pipelines
- Duplicate-file detection

## Part II — Python
- Why Python / scripting
- Running Python
- Modules and `pip`
- Functions
- Conditionals and loops
- Lists
- Dictionaries
- Tuples
- Nested dictionaries
- JSON
- Dictionary-based Tic-Tac-Toe
- Rewriting shell scripts in Python

---

# 1. Shell Functions

## Why functions?

Functions provide **abstraction and reuse**.

Instead of repeating:

```bash
ping -c 1 "$server"
```

define it once:

```bash
myping() {
    target=$1

    if ping -c 1 "$target" &> /dev/null; then
        echo "Server $target is reachable"
    else
        echo "Server $target is unreachable"
    fi
}
```

Then:

```bash
myping "$server"
```

## Function arguments

Shell function arguments use positional parameters:

| Parameter | Meaning |
|---|---|
| `$1` | first argument |
| `$2` | second argument |
| `$3` | third argument |
| ... | ... |

Example:

```bash
myping cse.iitb.ac.in
```

Inside the function:

```bash
target=$1
```

so `target` becomes `cse.iitb.ac.in`.

## Function syntax details

```bash
name() {
    command
}
```

Important:
- At least one whitespace must separate `{` from the first command.
- If `{`, the command and `}` are on one line, `;` separates the last command from `}`.

Useful:

```bash
type myping
```

This tells you what `myping` is.

---

# 2. Regular Expressions (RegEx)

A **regular expression is a pattern used to find, match, or describe text**.

Instead of searching for one exact string, we describe a pattern.

Typical uses:

- Find lines beginning with `ERROR`
- Find filenames ending in `.log`
- Identify email addresses
- Validate input formats
- Search and transform text

## Why learn RegEx?

Real-world data can be messy:
- logs
- system files
- reports
- text data

RegEx helps identify patterns.

Together:

```text
grep → find / filter
sed  → transform / clean
awk  → extract / filter / analyse
```

Unix pipes allow them to form text-processing workflows.

---

# 3. RegEx Cheat Sheet

| Symbol | Meaning | Example |
|---|---|---|
| `a` | exact character | matches `a` |
| `.` | any single character | `c.t` → `cat`, `cut` |
| `^` | beginning of line | `^ERROR` |
| `$` | end of line | `ERROR$` |
| `*` | zero or more occurrences | `a*` |
| `+` | one or more occurrences | `a+` |
| `?` | zero or one occurrence | `a\?` |
| `\char` | make a character literal | `\.` |

## Escape character

`\` is the escape character.

Examples:

```text
\.    literal .
\+    literal +
\?    literal ?
\*    literal *
\^    literal ^
\$    literal $
```

### Quiz trap

`^` and `$` normally have positional meanings:

```text
^ERROR     → line begins with ERROR
ERROR$     → line ends with ERROR
```

But:

```text
\^
\$
```

treat them as literal characters.

---

# 4. Character Classes `[ ]`

A character class matches **one character** from a specified set.

| Pattern | Meaning |
|---|---|
| `[a-z]` | any character from a through z |
| `[0-9]` | any digit |
| `[abc]` | one of `a`, `b`, `c` |
| `[^0-9]` | any character that is not a digit |
| `[^ABC]` | any character except A, B or C |
| `[a-zA-Z]` | an English letter |

Inside `[ ]`, characters have special rules.

For example:

```text
[?*()+]
```

matches the literal characters `?`, `*`, `(`, `)` or `+`.

`-` can specify a range:

```text
[a-z]
```

---

# 5. Repetition and Grouping

| Pattern | Meaning |
|---|---|
| `\{i\}` | exactly `i` occurrences |
| `\{i,j\}` | `i` through `j` occurrences |
| `\{i,\}` | at least `i` occurrences |
| `[list]` | one character from the list |
| `[^list]` | one character not from the list |
| `\(regexp\)` | group a pattern |
| `regexp1\|regexp2` | either pattern |
| `regexp1regexp2` | patterns in sequence |

Examples:

```text
[0-9]\{3\}
```

matches exactly three digits.

```text
[0-9]\{2,4\}
```

matches 2 to 4 digits.

```text
\(abc\)*
```

groups `abc` and applies `*`.

> **Lab note:** The lecture examples use forms such as `\+`, `\?`, `\{...\}`, `\(...\)` and `\|`. Remember the regex mode/tool being used.

---

# 6. `grep`, `sed`, `awk`

A useful mental model:

```text
grep → FIND / FILTER
sed  → TRANSFORM
awk  → EXTRACT / FILTER / ANALYSE
```

Example:

```bash
ps aux | grep -E 'python|java|ssh'
```

Find processes whose command contains `python`, `java`, or `ssh`.

---

# 7. `sed`

`sed` processes text according to commands and patterns.

## Basic substitution

General form:

```bash
sed 's|pattern|replacement|'
```

Meaning:

```text
s            substitute
pattern      text/pattern to find
replacement  replacement text
```

Example:

```bash
sed -E 's|root|ADMIN|' processes.txt
```

replaces `root` with `ADMIN`.

## Why use `|` as separator?

The separator does not have to be `/`.

This can make path-related substitutions easier:

```bash
sed -E 's|.*/||' processes.txt
```

This removes the directory path, leaving the final command/file name.

---

# 8. Important `sed` Commands

## Print matching lines

```bash
sed -n '/^text/p' book.txt
```

Meaning:

- `-n` → suppress automatic printing
- `/^text/` → select lines beginning with `text`
- `p` → print selected lines

## Delete a range

```bash
sed '1,5d' book.txt
```

`d` = delete.

This deletes lines 1 through 5.

## Substitute globally

```bash
sed 's/ext1/text2/g' book.txt
```

`g` means replace all matching occurrences on each selected line.

Without `g`, the usual substitution replaces only the first matching occurrence on each selected line.

## Multiple commands

```bash
sed '/^foo/d ; s/text1/text2/' book.txt
```

Interpretation:

1. Delete lines beginning with `foo`.
2. On remaining lines, replace the first occurrence of `text1` with `text2`.

## Quit

```bash
sed '/^text/q' book.txt
```

`q` = quit.

The command processes/prints lines until a line beginning with `text` is encountered, then quits.

## Append

```bash
sed '/^text/a\--- BEGIN CHAPTER ---' book.txt
```

Adds the chapter marker after every line beginning with `text`.

---

# 9. `sed` Capture Groups and Backreferences

`sed` can capture parts of a line and rearrange them.

Example:

```bash
du -ac d* | sed 's/\(.*\)\t\(.*\)/\2 \1/'
```

The pattern contains:

```text
\(.*\)    capture first part
\t        tab
\(.*\)    capture second part
```

The replacement:

```text
\2 \1
```

prints the second captured part first.

## Remember

```text
\( ... \)  → capture group
\1          → first captured group
\2          → second captured group
```

This is a common lab-test pattern.

---

# 10. `sed` + Command Substitution

Example:

```bash
ping "$(echo "cse" | sed 's/$/.iitb.ac.in/')"
```

Step-by-step:

```text
echo "cse"
    ↓
cse
    ↓
sed 's/$/.iitb.ac.in/'
    ↓
cse.iitb.ac.in
    ↓
ping "cse.iitb.ac.in"
```

Here `$` means **end of line**.

Therefore:

```bash
s/$/.iitb.ac.in/
```

appends `.iitb.ac.in` to the end of the input.

---

# 11. `sed` Inside a Shell Function

Example:

```bash
servers=("cse" "asc")

myping() {
    target=$1

    if ping -c 1 "$target" &> /dev/null; then
        echo "Server $target is reachable"
    else
        echo "Server $target is unreachable"
    fi
}

for server in "${servers[@]}"; do
    myping "$(echo "$server" | sed 's/$/.iitb.ac.in/')"
done
```

The transformation:

```bash
echo "$server" | sed 's/$/.iitb.ac.in/'
```

turns:

```text
cse → cse.iitb.ac.in
asc → asc.iitb.ac.in
```

---

# 12. `awk`

AWK is especially useful for **structured text**.

Basic structure:

```bash
awk 'pattern { action }' file
```

Example:

```bash
awk '$3 > 2 {print $1, $2, $3, $11}' processes.txt
```

Meaning:

```text
if field 3 > 2:
    print fields 1, 2, 3 and 11
```

---

# 13. AWK Fields

Given:

```text
Alice 25 Mumbai
```

AWK fields are:

```text
$1 = Alice
$2 = 25
$3 = Mumbai
```

and:

```text
$0 = entire current line
```

So:

```bash
awk '{print $1, $3}' file
```

prints the first and third fields.

---

# 14. AWK Built-in Variables

| Variable | Meaning |
|---|---|
| `$1`, `$2`, ... | individual fields |
| `$0` | entire current record/line |
| `NF` | number of fields in current record |
| `NR` | current record/line number |
| `FS` | input field separator |
| `OFS` | output field separator |
| `RS` | input record separator |
| `ORS` | output record separator |

Example input:

```text
manohar 25 Mumbai
pooja 30 Pune
```

Command:

```bash
awk '{print "NR =", NR, "NF =", NF, "$0 =", $0}' data.txt
```

For the first line:

```text
NR = 1
NF = 3
$0 = manohar 25 Mumbai
```

---

# 15. AWK + Regular Expressions

AWK can use a regex directly as a pattern.

## Select directories

```bash
ls -l | awk '/^d/ {print $1, $9}'
```

`/^d/` means lines beginning with `d`.

## Negate a pattern

```bash
ls -l | awk '!/^d/ {print $1, $9}'
```

This selects lines that do not begin with `d`.

## Multiple patterns

```bash
ls -l | awk '
/^-/ {print "file", $9}
/^d/ {print "dir", $9}
/^l/ {print "link", $9}'
```

The first character of the permissions field is used to classify:

```text
- → regular file
d → directory
l → symbolic link
```

---

# 16. AWK `BEGIN`, Main Rules, `END`

A useful AWK structure:

```awk
BEGIN {
    # initialization
}

pattern {
    # process records
}

END {
    # final processing
}
```

Example:

```awk
BEGIN {
    file = "file"
    dir = "directory"
    link = "link"
    print "Type"
}

/^-/ {
    print file, $9
    count[file]++
}

/^d/ {
    print dir, $9
    count[dir]++
}

/^l/ {
    print link, $9
    count[link]++
}

END {
    print count[file], file
    print count[dir], dir
    print count[link], link
}
```

## What happens?

### `BEGIN`

Runs before input records are processed.

Used here to initialize labels.

### Main rules

Each matching input record triggers its rule.

### `END`

Runs after input processing.

Used here to print the summary.

---

# 17. AWK Associative Arrays

AWK arrays can use strings as indices.

Example:

```awk
count[file]++
```

If:

```text
file = "file"
```

then:

```text
count["file"]
```

stores the count.

Another example:

```awk
count[$1]++
```

counts occurrences based on field 1.

---

# 18. AWK Script Files

Instead of writing a program on the command line:

```bash
ls -l | awk -f filename.awk
```

The AWK program can be stored in:

```text
filename.awk
```

Example executable AWK script:

```awk
#!/usr/bin/awk -f

BEGIN {
    print "Duplicate files"
}

{
    count[$1]++
    names[$1] = names[$1] " " $2
}

END {
    for (key in count)
        if (count[key] > 1)
            print names[key]
}
```

Then:

```bash
md5sum * 2>/dev/null | ./duplicatefile.awk
```

---

# 19. Detecting Duplicate Files

Two files can have different names but identical contents.

A useful approach:

```bash
md5sum *
```

produces hashes.

Then:

```bash
md5sum * 2>/dev/null | awk '{print $1}' | sort | uniq -c
```

This counts how many times each hash occurs.

But to obtain the filenames belonging to each hash:

```bash
md5sum * 2>/dev/null |
awk '
{
    count[$1]++
    names[$1] = names[$1] " " $2
}
END {
    for (key in count)
        if (count[key] > 1)
            print names[key]
}'
```

## Logic

For every input line:

```text
$1 = hash
$2 = filename
```

Store:

```awk
count[$1]++
```

and:

```awk
names[$1] = names[$1] " " $2
```

At the end:

```awk
if (count[key] > 1)
```

means that hash appeared more than once → duplicate contents.

---

# 20. AWK Custom Field Separator

By default, AWK splits input into fields using its input field separator.

For `/etc/passwd`, fields are separated by `:`.

Use:

```bash
awk -F: '{print $1}' /etc/passwd
```

`-F:` means:

```text
FS = :
```

Example:

```text
root:...:0:1:...
```

then:

```text
$1 = root
$2 = ...
$3 = 0
$4 = 1
...
```

The lecture also shows:

```bash
awk -F: '$2 == ""' /etc/passwd
```

This selects records where field 2 is empty.

---

# 21. Shell Text-Processing Patterns to Memorize

## Filter with grep

```bash
command | grep -E 'pattern1|pattern2'
```

## Print matching lines with sed

```bash
sed -n '/pattern/p' file
```

## Delete lines

```bash
sed '/pattern/d' file
```

## Substitute

```bash
sed 's|old|new|'
```

## Substitute globally

```bash
sed 's|old|new|g'
```

## AWK field extraction

```bash
awk '{print $1, $3}' file
```

## AWK condition

```bash
awk '$3 > 2 {print $1, $2}' file
```

## AWK regex condition

```bash
awk '/^d/ {print $1, $9}' file
```

## AWK custom separator

```bash
awk -F: '{print $1}' file
```

---

# 22. Python — Why Learn It?

The lecture describes Python as a scripting language where many tasks can be done in relatively few lines.

Python has libraries for:

- Web scraping
- Graphics
- Data analytics
- Document manipulation
- Web-app development
- AI/ML
- Game development

---

# 23. Running Python

## Run a script

```bash
python3 test.py
```

## Interactive interpreter

```bash
python3
```

Then:

```python
>>> print('hello world')
hello world
>>> 2 + 4
6
```

The interpreter evaluates expressions immediately.

Variables can be used without declaration.

---

# 24. Python Modules

A module can provide functionality that is not built into your script directly.

Example:

```python
import webbrowser as wb

with open('links.txt') as linkfile:
    for link in linkfile:
        wb.open(link.strip())
```

Here:

```python
import webbrowser as wb
```

imports the `webbrowser` module and gives it the shorter name `wb`.

---

# 25. Installing a Library

The lecture uses:

```bash
pip install pyperclip
```

Then:

```python
import pyperclip
```

Example workflow:

```python
text = pyperclip.paste()
```

reads clipboard text.

Then the program can process it and:

```python
pyperclip.copy(bulleted)
```

copies the result back.

---

# 26. Clipboard Example

The lecture's idea:

```text
Clipboard text
      ↓
Read
      ↓
Process each line
      ↓
Add "* "
      ↓
Copy back
```

Code:

```python
import pyperclip

text = pyperclip.paste()
lines = text.split('\n')

bulleted = ""

for i in range(len(lines)):
    print(lines[i])
    lines[i] = '* ' + lines[i]
    bulleted += lines[i] + '\n'

pyperclip.copy(bulleted)
```

### Concepts tested

- importing modules
- strings
- `split`
- lists
- loops
- indexing
- string concatenation

---

# 27. Reading a PDF in Python

The lecture demonstrates:

```python
import PyPDF2
import pyttsx3

speaker = pyttsx3.init()

readpdf = PyPDF2.PdfReader(open('story.pdf', 'rb'))

for pagenumber in range(len(readpdf.pages)):
    page = readpdf.pages[pagenumber]
    text = page.extract_text()
    speaker.save_to_file(text, 'story.mp3')
    speaker.runAndWait()

speaker.stop()
```

Conceptual workflow:

```text
initialize
    ↓
configure
    ↓
read PDF
    ↓
extract text
    ↓
generate audio
    ↓
cleanup
```

The lecture also demonstrates changing TTS settings such as:

```python
speaker.getProperty('rate')
speaker.setProperty('rate', 200)
speaker.setProperty('volume', 1)
speaker.getProperty('voices')
speaker.setProperty('voice', voice.id)
```

---

# 28. Python Version

The lecture emphasizes checking the Python version.

Useful commands:

```bash
which python
which python3
```

and:

```bash
python3 --version
```

The lecture recommends Python 3.9 or higher.

Also remember:

```bash
echo $?
```

reports the exit status of the previous command.

---

# 29. Python Indentation

Python uses **indentation to group statements**.

It replaces curly-brace block syntax used in languages such as C/C++/Java.

Indentation defines the body of:

- `if`
- `for`
- `while`
- functions

Example:

```python
if x > 0:
    print("positive")
```

### Important lab rule

Do **not** mix tabs and spaces.

---

# 30. Python `if`, `for`, `while`, `break`

Example structure:

```python
while True:
    guess = input('> ')

    if guess.isdecimal():
        return int(guess)
```

For loop:

```python
for i in range(10):
    print(i)
```

Break:

```python
if guess == secretNumber:
    break
```

### Constructs demonstrated by the lecture

| Construct | Example |
|---|---|
| function | `def askForGuess():` |
| while | `while True:` |
| if | `if guess < secretNumber:` |
| for | `for i in range(10):` |
| break | `break` |
| input | `input()` |
| conversion | `int(guess)` |

---

# 31. Python Functions

## Function without return value

```python
def greet_user(username):
    print(f"Hello, welcome to today's session {username.title()}!")

greet_user('sakina')
```

## Function with return value

```python
def get_formatted_name(first_name, last_name):
    full_name = f"{first_name} {last_name}"
    return full_name.title()

full_name = get_formatted_name('sakina', 'hashmi')
print(full_name)
```

### Important distinction

A function may:

```text
perform an action
```

without returning a value, or:

```text
calculate/produce something
```

and return it with `return`.

---

# 32. Function Arguments

## Positional arguments

Arguments are assigned according to order:

```python
def describe_pet(animal_type, pet_name):
    print(f"My pet {animal_type}' name is {pet_name}")

describe_pet('dog', 'Tutto')
```

So:

```text
animal_type = 'dog'
pet_name    = 'Tutto'
```

## Keyword arguments

Parameters can be explicitly named:

```python
describe_pet(animal_type='Dog', pet_name='Tuttu')
```

Order can then be changed:

```python
describe_pet(pet_name='Tuttu', animal_type='Dog')
```

## Default argument

```python
def describe_pet1(pet_name, animal_type="dog"):
    print(f"My {animal_type}'s name is {pet_name.title()}")
```

Now:

```python
describe_pet1('tutto')
```

uses:

```text
animal_type = "dog"
```

But:

```python
describe_pet1('kali', 'cat')
```

overrides the default.

---

# 33. Lists

Lists are ordered and mutable collections.

Create:

```python
fruits = ['apple', 'banana', 'orange', 'strawberry']
```

Traverse:

```python
for fruit in fruits:
    print(fruit)
```

Index:

```python
flowers = ['jasmine', 'hibiscus', 'rose']

print(flowers[0])
```

Python list indexing starts at `0`.

For a list of length `n`, valid indices are:

```text
0 ... n-1
```

## List methods shown

```python
flowers.sort()
fruits.reverse()
```

## Membership / non-membership

```python
blocked_users = ['sakina', 'swapnil', 'maya']
user = 'anshul'

if user not in blocked_users:
    print(f"{user.title()}, not blocked")
```

---

# 34. List vs Dictionary vs Tuple

This comparison is important for quizzes/labs.

| Structure | Syntax | Main idea | Mutable? |
|---|---|---|---|
| List | `[...]` | ordered collection | Yes |
| Dictionary | `{key: value}` | key → value mapping | Yes |
| Tuple | `(...)` | ordered collection | No |

Examples:

```python
servers = ["server1", "server2", "db1"]
```

```python
server_ip = {
    "server1": "192.168.1.10",
    "server2": "192.168.1.11",
    "db1": "192.168.1.20"
}
```

```python
db_connection = ("db1", 5432, "production")
```

---

# 35. Dictionary

A dictionary stores **key:value pairs**.

Example:

```python
likes = {
    "color": "blue",
    "fruit": "apple",
    "pet": "dog"
}
```

Access:

```python
likes["color"]
```

## Traverse keys

```python
for key in likes:
    print(key, "->", likes[key])
```

## Traverse keys and values

```python
for key, value in likes.items():
    print(key, "->", value)
```

## Update a value

```python
fruits = {
    "apple": 0.40,
    "orange": 0.35,
    "banana": 0.25
}

for fruit, price in fruits.items():
    fruits[fruit] = round(price * 0.9, 2)
```

## Add a key-value pair

```python
fruits["mango"] = 0.42
```

---

# 36. Tuple

Tuple example:

```python
db_connection = ("db1", 5432, "production")
```

Access:

```python
print(db_connection[0])
print(db_connection[1])
```

But:

```python
db_connection[1] = 3306
```

causes an error because tuples are **immutable**.

---

# 37. Nested Dictionaries

A nested dictionary contains dictionaries as values.

Example:

```python
users = {
    'einstein': {
        'first': 'albert',
        'last': 'einstein',
        'location': 'princeton'
    },
    'mcurie': {
        'first': 'marie',
        'last': 'curie',
        'location': 'paris'
    }
}
```

Conceptually:

```text
users
 ├── einstein
 │    ├── first
 │    ├── last
 │    └── location
 │
 └── mcurie
      ├── first
      ├── last
      └── location
```

Access can be nested:

```python
users['einstein']['first']
```

---

# 38. JSON

JSON = **JavaScript Object Notation**.

It is a lightweight, text-based format for representing structured data.

JSON can represent:

- key-value pairs
- arrays
- nested objects

It is widely used for transferring structured data between clients and servers in web applications.

Example:

```json
{
    "first_name": "Rohan",
    "last_name": "Pandey",
    "websites": [
        {
            "description": "work",
            "URL": "www.google.com"
        },
        {
            "description": "personal",
            "URL": "www.youtube.com"
        }
    ],
    "social_media": [
        {
            "description": "facebook",
            "URL": "www.facebook.com"
        }
    ]
}
```

### Key idea

JSON is particularly useful when data has nested structure.

---

# 39. Dictionary with Integer Indices — Tic-Tac-Toe

The lecture uses nested dictionaries to represent a 2D game board.

Conceptually:

```python
theBoard[row][column]
```

The board is initialized with:

```python
ROWS = 3
COLS = 3
turns = ['X', 'O']

theBoard = dict()

for i in range(ROWS):
    theBoard[i + 1] = dict()

    for j in range(COLS):
        theBoard[i + 1][j + 1] = ' '
```

So positions look like:

```text
theBoard[1][1]
theBoard[1][2]
...
theBoard[3][3]
```

---

# 40. Tic-Tac-Toe: Turns

```python
move_no = 0

while move_no < ROWS * COLS:
    turn = turns[move_no % len(turns)]
```

Because:

```text
turns = ['X', 'O']
```

the expression:

```python
move_no % len(turns)
```

alternates between:

```text
0, 1, 0, 1, ...
```

giving:

```text
X, O, X, O, ...
```

---

# 41. Tic-Tac-Toe: Reading a Move

```python
move = input()

move_row, move_col = move.split(',')

turn_row, turn_col = int(move_row), int(move_col)
```

Example input:

```text
1,3
```

After `split(',')`:

```text
move_row = "1"
move_col = "3"
```

After `int()`:

```text
turn_row = 1
turn_col = 3
```

---

# 42. Tic-Tac-Toe: Validating a Move

The lecture checks:

```python
if (
    0 < turn_row <= ROWS and
    0 < turn_col <= COLS and
    theBoard[turn_row][turn_col] == ' '
):
```

Three conditions:

1. Row is inside the board.
2. Column is inside the board.
3. The selected position is empty.

Then:

```python
theBoard[turn_row][turn_col] = turn
move_no += 1
```

Otherwise:

```python
print('NOT A VALID MOVE!')
```

---

# 43. Printing the Board

Function:

```python
def printBoard(board):
    for each_row in board:
        print('|'.join(board[each_row].values()))
        print('+'.join(['-' for _ in range(len(board[each_row]))]))
```

Important concepts:

- function
- loop
- dictionary values
- `join`
- list comprehension
- board representation

---

# 44. Better Program Structure

The lecture gives a cleaner high-level design:

```text
initialize()
createBoard()

while notOver():
    askMove
    row, col = getMove()

    if validMove(row, col):
        makeMove

    printBoard

endGame
```

The key software-engineering idea is to divide a large task into small functions with clear responsibilities.

---

# 45. Debugging a Small Python Script

The lecture gives:

```python
import webbrowser

with open('links.txt') as file:
    links = file.readlines()

for link in links:
    webbrowser.open('link')
```

There is a bug.

The loop variable is:

```python
link
```

but the code passes the literal string:

```python
'link'
```

Instead, the variable should be used.

The intended form is:

```python
webbrowser.open(link)
```

This is a useful lab-test lesson:

> **A variable name in quotes becomes a string literal.**

Compare:

```python
webbrowser.open(link)
```

vs.

```python
webbrowser.open('link')
```

---

# 46. Shell → Python: What the Assignment Wants

The lecture asks students to:

- Read selected chapters from *Automate the Boring Stuff with Python*
- Study:
  - Chapter 8 — Manipulating Strings
  - Chapter 9 — Pattern Matching with Regular Expressions
  - Chapter 10 — Reading and Writing Files
  - Chapter 11 — Organizing Files
- Redo shell-script homework in Python.
- If something cannot be done directly, note why.

The broader point is to understand how similar automation tasks can be expressed in Python.

---

# 47. Lab-Test: Commands You Should Be Able to Write

## Shell functions

```bash
f() {
    x=$1
    ...
}
```

Call:

```bash
f argument
```

## Regex filtering

```bash
grep -E 'python|java|ssh' file
```

## `sed` print

```bash
sed -n '/pattern/p' file
```

## `sed` delete

```bash
sed '/pattern/d' file
```

## `sed` substitute

```bash
sed 's|old|new|g' file
```

## `sed` capture/rearrange

```bash
sed 's/\(first\)\(second\)/\2\1/'
```

## AWK fields

```bash
awk '{print $1, $3}' file
```

## AWK condition

```bash
awk '$3 > 2 {print $1, $2, $3}' file
```

## AWK regex

```bash
awk '/^d/ {print $1, $9}' file
```

## AWK custom separator

```bash
awk -F: '{print $1}' file
```

## AWK counting

```bash
awk '{count[$1]++} END {for (x in count) print x, count[x]}' file
```

---

# 48. Quiz Traps

## Trap 1 — `$1` vs `$0`

```text
$1 → first field
$0 → entire current record
```

## Trap 2 — `NF` vs `NR`

```text
NF → number of fields
NR → record/line number
```

## Trap 3 — `FS` vs `OFS`

```text
FS  → input field separator
OFS → output field separator
```

## Trap 4 — `^` vs `$`

```text
^ → beginning of line
$ → end of line
```

## Trap 5 — `d` vs `p`

```text
d → delete
p → print
```

## Trap 6 — `g` in `sed`

```text
g → global substitution on the selected line
```

## Trap 7 — `\1`, `\2`

These refer to captured groups:

```text
\( ... \) → group
\1 → first group
\2 → second group
```

## Trap 8 — list vs tuple

```text
list  → mutable
tuple → immutable
```

## Trap 9 — dictionary lookup

```python
server_ip["server1"]
```

uses a **key**, not a positional index.

## Trap 10 — indentation

Python uses indentation to define blocks.

---

# 49. Quick Concept Comparison

| Tool / Concept | Main purpose |
|---|---|
| RegEx | describe text patterns |
| `grep` | search/filter text |
| `sed` | transform text |
| `awk` | process structured text |
| `$1` in AWK | first field |
| `$0` in AWK | complete record |
| `NF` | number of fields |
| `NR` | record number |
| `FS` | input separator |
| `OFS` | output separator |
| `BEGIN` | before input processing |
| `END` | after input processing |
| List | ordered, mutable collection |
| Dictionary | key-value mapping |
| Tuple | ordered, immutable collection |
| JSON | structured text/data interchange format |

---

# 50. Mini Lab Practice

Try writing these **without looking at the answers first**.

### Q1
Print only lines beginning with `ERROR` from `log.txt`.

### Q2
Delete lines 1 through 5 from `file.txt`.

### Q3
Replace every `foo` with `bar`.

### Q4
From:

```text
Alice 25 Mumbai
Bob 30 Pune
```

print:

```text
Alice Mumbai
Bob Pune
```

### Q5
Print only records whose third field is greater than 50.

### Q6
For `/etc/passwd`, print the first field using `:` as the separator.

### Q7
Count how many times each first-field value occurs.

### Q8
Write a Python function that receives a name and returns its title-cased form.

### Q9
Create a list of three servers and print each server.

### Q10
Create a dictionary mapping server names to IP addresses and print all key-value pairs.

### Q11
Create a tuple containing database name, port and environment. Try modifying the port.

### Q12
Read comma-separated row/column coordinates and convert them to integers.

---

# 51. Mini Lab Practice — Answers

## Q1

```bash
sed -n '/^ERROR/p' log.txt
```

or, conceptually, use a grep pattern that matches the beginning of the line.

## Q2

```bash
sed '1,5d' file.txt
```

## Q3

```bash
sed 's|foo|bar|g' file.txt
```

## Q4

```bash
awk '{print $1, $3}' file.txt
```

## Q5

```bash
awk '$3 > 50 {print $0}' file.txt
```

## Q6

```bash
awk -F: '{print $1}' /etc/passwd
```

## Q7

```bash
awk '{count[$1]++} END {for (x in count) print x, count[x]}' file.txt
```

## Q8

```python
def format_name(name):
    return name.title()
```

## Q9

```python
servers = ["server1", "server2", "db1"]

for server in servers:
    print(server)
```

## Q10

```python
server_ip = {
    "server1": "192.168.1.10",
    "server2": "192.168.1.11"
}

for name, ip in server_ip.items():
    print(name, ip)
```

## Q11

```python
db_connection = ("db1", 5432, "production")
```

Trying:

```python
db_connection[1] = 3306
```

raises an error because tuples are immutable.

## Q12

```python
move = input()

move_row, move_col = move.split(',')
turn_row, turn_col = int(move_row), int(move_col)
```

---

# 52. High-Priority Lab Checklist

Before the lab test, make sure you can write these **from memory**:

### Shell
- [ ] Function definition
- [ ] `$1`, `$2`, etc.
- [ ] Arrays and `"${array[@]}"`
- [ ] `if ... then ... else`
- [ ] Exit status / `$?`
- [ ] Pipes `|`
- [ ] Redirection
- [ ] Command substitution `$(...)`

### RegEx
- [ ] `.`
- [ ] `^`
- [ ] `$`
- [ ] `*`
- [ ] `+`
- [ ] `?`
- [ ] `[ ]`
- [ ] `[^ ]`
- [ ] `{i,j}` form used by the lecture
- [ ] grouping
- [ ] alternation
- [ ] escaping

### `sed`
- [ ] `s`
- [ ] `p`
- [ ] `d`
- [ ] `q`
- [ ] `a`
- [ ] `g`
- [ ] ranges such as `1,5`
- [ ] capture groups
- [ ] backreferences

### `awk`
- [ ] `$0`
- [ ] `$1`, `$2`, ...
- [ ] `NF`
- [ ] `NR`
- [ ] `FS`
- [ ] `OFS`
- [ ] `BEGIN`
- [ ] `END`
- [ ] regex patterns
- [ ] conditions
- [ ] associative arrays
- [ ] `-F`

### Python
- [ ] `import`
- [ ] `input()`
- [ ] `int()`
- [ ] `if`
- [ ] `for`
- [ ] `while`
- [ ] `break`
- [ ] functions
- [ ] `return`
- [ ] positional arguments
- [ ] keyword arguments
- [ ] default arguments
- [ ] lists
- [ ] dictionaries
- [ ] tuples
- [ ] nested dictionaries
- [ ] `.items()`
- [ ] `.split()`
- [ ] `.join()`
- [ ] indentation

---

# 53. One-Page Memory Sheet

```text
REGEX
^       beginning
$       end
.       any one character
*       0 or more
+       1 or more
?       0 or 1
[abc]   one of a,b,c
[^abc]  not a,b,c
[a-z]   range
\( \)   capture group
\1      first captured group

SED
s       substitute
p       print
d       delete
q       quit
a       append
g       global substitution

sed -n '/pattern/p' file
sed '/pattern/d' file
sed 's|old|new|g' file

AWK
$0      whole line
$1      first field
$2      second field
NF      number of fields
NR      record/line number
FS      input separator
OFS     output separator

awk '{print $1, $3}' file
awk '$3 > 2 {print $1, $2}' file
awk '/^d/ {print $1, $9}' file
awk -F: '{print $1}' file

BEGIN   before records
END     after records
count[$1]++   associative-array counting

PYTHON
list       []      mutable
dict       {}      key:value
tuple      ()      immutable

def f(x):
    return x

for x in items:
    ...

if condition:
    ...

while condition:
    ...

for k, v in d.items():
    ...

x, y = s.split(',')
```

---

# 54. What to Understand, Not Just Memorize

For the quiz/lab, don't only memorize commands. Be able to explain:

1. **Why** a regex matches a particular line.
2. **Why** `sed -n` is used with `p`.
3. Difference between **printing** and **deleting** in `sed`.
4. How a `sed` capture group becomes `\1`, `\2`, etc.
5. How AWK converts a line into fields.
6. Difference between `$0`, `$1`, `NF`, and `NR`.
7. Why `-F:` is needed for colon-separated data.
8. How AWK associative arrays can count occurrences.
9. What `BEGIN` and `END` do.
10. Why a tuple cannot be modified.
11. Difference between positional and keyword arguments.
12. Why Python indentation matters.
13. How nested dictionaries represent structured data.
14. How a shell pipeline can be translated into Python.

---

# 55. Lecture Assignment / Further Practice

The lecture specifically points to:

- `sed` exercises:
  - Numbering lines
  - Numbering non-blank lines
  - Counting characters
  - Counting words
  - Counting lines
  - Printing first/last lines
  - Making duplicate lines unique
  - Printing duplicated lines
  - Removing duplicated lines
  - Squeezing blank lines

- Python reading:
  - Automate the Boring Stuff — Chapter 8: Manipulating Strings
  - Chapter 9: Pattern Matching with Regular Expressions
  - Chapter 10: Reading and Writing Files
  - Chapter 11: Organizing Files

- Redo earlier shell-script homework in Python.

---

# 56. Final Revision Strategy

## First pass — Concepts

Be able to explain:

```text
RegEx → pattern
grep  → search/filter
sed   → transform
awk   → fields/filter/analysis
Python → scripting/automation
```

## Second pass — Syntax

Write from memory:

```bash
sed -n '/pattern/p' file
sed '/pattern/d' file
sed 's|old|new|g' file
awk '{print $1, $2}' file
awk '$3 > 2 {print $1}' file
awk -F: '{print $1}' file
```

## Third pass — Code

Without notes, implement:

1. A shell function.
2. A `sed` transformation.
3. An AWK field filter.
4. An AWK counting program.
5. A Python function.
6. A Python list traversal.
7. A Python dictionary traversal.
8. A small nested-dictionary program.

## Fourth pass — Debugging

Look at short code and ask:

- What does each variable contain?
- What is `$1` here?
- What does the regex match?
- How many fields does AWK see?
- Does `sed` print, delete or substitute?
- Is this Python object mutable?
- Is the function returning a value?
- Is indentation correct?
- Is a variable accidentally written as a string, e.g. `'link'` instead of `link`?

---

# 57. Ultra-Short Exam Recall

If you have only **2 minutes**:

```text
REGEX:
^ start, $ end, . any, * 0+, + 1+, ? 0/1
[abc] one, [^abc] except
\( \) group, \1 backreference

SED:
s substitute
p print
d delete
q quit
a append
g global
-n suppress automatic printing

AWK:
$0 whole line
$1... fields
NF number fields
NR line number
FS input separator
OFS output separator
BEGIN before input
END after input
-F custom separator
count[key]++ counting

PYTHON:
[] list → mutable
{} dict → key:value
() tuple → immutable
def → function
return → return value
.items() → key/value traversal
split() → split string
join() → combine strings
indentation → blocks
```

---

# 58. Source Coverage

These notes are based on the uploaded **CS 699 Lecture 4** slides and preserve the lecture's main terminology, examples and code-oriented emphasis.

For exact lecture-slide wording/examples, refer back to the original PDF.
