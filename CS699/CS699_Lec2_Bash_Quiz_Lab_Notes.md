# CS 699 - Lecture 2: Shell Scripting (BASH)
## Lecture Notes + Quiz Revision + Lab-Test Prep

**Source:** CS 699 - Lec 2, Shell Scripting - BASH, Om Damani, CSE IIT Bombay.

> This note is organized for quick revision before a quiz or lab test. The concepts, commands, examples, and practice tasks are based on the lecture slides. Explanatory comments are added to make the material easier to revise.

---

# 1. Big Picture

## What is a shell script?

A **shell script** is a text file containing one or more shell commands executed sequentially by a shell interpreter.

Key idea:

- Shell scripts are **interpreted**, not compiled like Java or C++.
- Earlier instructions can execute before the shell reaches an erroneous instruction.
- A script lets us collect a sequence of commands in one place and execute them repeatedly.

### Why use shell scripting?

Shell scripts are useful for automating tasks such as:

- daily computer tasks
- research/data management
- code management
- automatic processing
- repetitive filesystem tasks

Example problem from the lecture: identify duplicate files in a directory.

---

# 2. Creating a Bash Script

Basic structure:

```bash
#!/bin/bash

command1
command2
command3
```

## Shebang

```bash
#!/bin/bash
```

This indicates the Bash interpreter to use for the script.

## Example

```bash
#!/bin/bash
cd ~/cs699

echo "Number of files:"
ls | wc -l

echo "Total disk usage:"
du -hs

echo "Detailed file listing:"
ls -rtl | tail -n +2
```

A `.sh` extension is commonly used, although the lecture treats it as optional.

---

# 3. Running a Shell Script and Permissions

A newly created script is normally a regular text file and does not automatically have execute permission.

Trying to execute it directly can produce:

```text
bash: permission denied: ./test_cs699.sh
```

## Give execute permission

```bash
chmod +x filename.sh
```

Example:

```bash
chmod +x ./test_cs699.sh
```

You can also use an explicit permission mode:

```bash
chmod 744 ./test_cs699.sh
```

The lecture shows the permission string:

```text
-rwxr-xr-x
```

`+x` means: add execute permission.

## Why `./`?

`.` represents the **current directory**.

So:

```bash
./script_name.sh
```

means: execute `script_name.sh` from the current directory.

### Lab-test memory pair

```text
chmod +x file.sh    -> make it executable
./file.sh           -> execute it from current directory
```

---

# 4. Special Shell Parameters

These are very important for quiz/lab questions.

| Parameter | Meaning |
|---|---|
| `$0` | Name of the current script |
| `$1`, `$2`, `$3`, ... | Positional command-line arguments |
| `$#` | Number of command-line arguments |
| `$*` | All command-line arguments as a single string |
| `$@` | All command-line arguments as separate strings |
| `$$` | PID of the current shell/script |
| `$?` | Exit status of the previously executed command |
| `!!` | Repeat the previous command; not a parameter/variable |

## Example

Suppose:

```bash
./test.sh abc 25 xyz
```

Then conceptually:

```text
$0 -> ./test.sh
$1 -> abc
$2 -> 25
$3 -> xyz
$# -> 3
```

---

# 5. Command-Line Arguments

Parameters allow values to be supplied when running the script instead of hard-coding them.

Example:

```bash
./directory_info.sh ~/Downloads
./directory_info.sh ~/Documents
```

Another example:

```bash
./ping_server.sh google.com
./ping_server.sh cse.iitb.ac.in
```

## Basic pattern

```bash
#!/bin/bash

directory="$1"
echo "Directory: $directory"

ls "$directory" | wc -l
du -sh "$directory"
ls -rtl "$directory" | tail -n +2
```

Run with:

```bash
./test_cs699.sh backup/
```

### Lab-test pattern

When the question says:

> "Write a script that accepts X as input"

look for:

```bash
value="$1"
```

or, if multiple inputs are needed, `$1`, `$2`, etc.

---

# 6. Variables

Shell variables store values for reuse.

Example:

```bash
project="cs699"
datafile="data.csv"
```

Use them with `$`:

```bash
echo "project directory: $project"
echo "data file: $datafile"
```

Example from the lecture:

```bash
find "$project" -type f | wc -l
du -sh "$project"
ls -lrt "$project" | tail -n +2
```

## Important syntax

Assignment:

```bash
name="value"
```

Use:

```bash
$name
```

Do **not** put spaces around `=` in a variable assignment.

---

# 7. Quoting Variables

The lecture examples frequently use:

```bash
"$directory"
"$target"
"$log_file"
```

This is especially useful when a filename or path may contain spaces.

For lab work, a good habit is:

```bash
command "$variable"
```

instead of leaving the variable unquoted when it represents a path.

---

# 8. Command Substitution: `$()`

Very important concept:

```bash
$(command)
```

means:

1. Execute the command inside `$(...)`.
2. Replace `$(...)` with the command's output.

Example:

```bash
timestamp=$(date +"%Y%m%d%H%M%S")
```

Here `date` is executed and its output is stored in `timestamp`.

The lecture uses it in a backup example:

```bash
#!/bin/bash

bck_dir="/Users/sakina/Desktop/cs_699/backup"
src_dir="/Users/sakina/Desktop/cs_699/source"
timestamp=$(date +"%Y%m%d%H%M%S")

tar -czf "$bck_dir/backup_$timestamp.tar.gz" "$src_dir"
```

### Quiz question to expect

**What does `$()` do?**

> It executes the command inside the parentheses and substitutes its output.

---

# 9. Shell Functions

A function is a named block of commands that performs a specific task.

It can be called multiple times.

## Syntax

```bash
function_name() {
    Commands
}
```

## Example

```bash
#!/bin/bash

greet() {
    echo "Welcome to CS699"
}

greet
greet
greet
```

## Function argument

The lecture shows function parameters using `$1` inside the function:

```bash
#!/bin/bash

greet() {
    message="$1"
    echo "Message: $message"
    echo "Length: ${#message}"
}

greet "Good day!"
```

Here:

- `greet` is the function.
- `"Good day!"` is passed to the function.
- Inside the function, `$1` refers to that function argument.
- `${#message}` gives the length of the string stored in `message`.

### Important distinction

```bash
./script.sh hello
```

Here `hello` is a **script argument**.

Inside:

```bash
greet "hello"
```

`hello` is a **function argument**.

---

# 10. `$n` Inside Other Commands

Sometimes `$n` may be interpreted by another program rather than the shell.

The lecture specifically shows `awk`:

```bash
echo a b c | awk '{print $3 $1}'
```

To insert a space between the outputs:

```bash
echo a b c | awk '{print $3 " " $1}'
```

### Key idea

Be careful when `$n` belongs to the command being run inside the script rather than to the outer shell script.

---

# 11. `if`, `then`, `else`, `elif`, `fi`

Basic syntax:

```bash
if [ condition ]
then
    commands
else
    commands
fi
```

For multiple conditions:

```bash
if [ condition1 ]
then
    ...
elif [ condition2 ]
then
    ...
else
    ...
fi
```

## Example: marks

```bash
#!/bin/bash

marks=75

if [ "$marks" -ge 40 ]
then
    echo "Pass"
else
    echo "Fail"
fi
```

Here:

```text
-ge -> greater than or equal to
```

---

# 12. Test Conditions `[ ]` and `[[ ]]`

The lecture notes that both can be used for conditions in an `if` statement.

### `[ ]`

Basic `test` command form and POSIX compliant.

Example:

```bash
if [ "$marks" -ge 40 ]
```

### `[[ ]]`

Extended Bash test with additional features.

For this lecture, remember the distinction and be comfortable reading both forms.

---

# 13. File Tests

The lecture uses:

```bash
[ -e "$log_file" ]
```

to test whether a path exists.

Example pattern:

```bash
if [ ! -e "$log_file" ]
then
    echo "$log_file does not exist."
fi
```

Important pieces:

```text
-e       -> test existence
!        -> negate the test
```

Another example from the lecture:

```bash
if [ -f "$FILE" ]
then
    echo "$FILE found."
else
    echo "$FILE not found."
    exit 1
fi
```

Here `-f` is used to test for a regular file.

---

# 14. Log Rotation Example

This combines variables, arguments, `if`, command substitution, and file operations.

```bash
#!/bin/bash

log_file=$1
max_size=1000000

if [ ! -e "$log_file" ]
then
    echo "$log_file does not exist."
elif [ "$(wc -c < "$log_file")" -gt "$max_size" ]
then
    mv "$log_file" "$log_file.old"
    touch "$log_file"
    echo "log rotated successfully."
else
    echo "$log_file is not large enough to rotate."
fi
```

### What this script does

1. Takes the log filename from `$1`.
2. Checks whether it exists.
3. Computes its size using `wc -c`.
4. Compares the size with `1000000` bytes.
5. If large enough, renames the old log using `mv` and creates a new empty file using `touch`.
6. Otherwise, it says rotation is not required.

### Test-file commands shown in the lecture

```bash
head --bytes 300000 /dev/zero > file3.txt
```

and

```bash
truncate -s 10485760 file.txt
```

The slides also caution that some commands found online may not work as expected on the system being used.

---

# 15. `for` Loop

A `for` loop repeats a task for every item in a list.

Basic pattern from the lecture:

```bash
for item in list
 do
    commands
 done
```

Example from server monitoring:

```bash
#!/bin/bash

servers=("cse.iitb.ac.in" "iitb.ac.in" "asc.iitb.ac.in")

for server in "${servers[@]}"
do
    if ping -c 1 "$server" &> /dev/null
    then
        echo "Server $server is reachable"
    else
        echo "Server $server is unreachable"
    fi
done
```

## Array expansion

The lecture notes:

```bash
"${servers[@]}"
```

represents all elements of the array.

### Lab-test pattern

When asked:

> "Repeat this operation for each directory/server/department/file..."

think:

```bash
for item in ...
do
    ...
done
```

---

# 16. `while` Loop

A `while` loop repeats a block of commands as long as its condition remains true.

## Counter example

```bash
#!/bin/bash

x=1
while [ $x -le 3 ]
do
    echo "Line $x"
    x=$((x + 1))
done
```

Output conceptually:

```text
Line 1
Line 2
Line 3
```

## Repeating command every 10 seconds

The lecture gives:

```bash
#!/bin/bash

while sleep 10
do
    df -h | grep "/dev/disk3s2" | awk '{print $5}' | tr -d '%'
done
```

The idea is to repeatedly check disk usage at 10-second intervals.

---

# 17. Arithmetic Expansion

The lecture uses:

```bash
x=$((x + 1))
```

to update a counter.

Remember the pattern:

```bash
$(( arithmetic expression ))
```

---

# 18. `read` - Getting User Input

The lecture shows:

```bash
echo -n "Please type in your name: "
read name

echo "Your name is ${name}"
```

So:

```bash
read variable
```

stores typed input in the variable.

## `-n`

The lecture uses:

```bash
read -n 1 gender
```

`-n 1` means read only one character.

The slides also explain that `echo -n` means do not print a newline.

---

# 19. `case` Statement

Useful when a single value can take several choices.

Example from the lecture:

```bash
#!/bin/bash

echo -n "Please type in your name: "
read name
echo "Your name is ${name}"

echo -n "Please type in your gender [M/F/T]: "
read -n 1 gender

case "$gender" in
    [mM])
        echo -e "\nYour answer: Male";;
    [fF])
        echo -e "\nYour answer: Female";;
    [tT])
        echo -e "\nYour answer: Transgender";;
    *)
        echo -e "\nInvalid choice: $gender";;
esac
```

## Structure to memorize

```bash
case "$value" in
    pattern1)
        commands;;
    pattern2)
        commands;;
    *)
        default commands;;
esac
```

### Quiz trap

The closing keyword is:

```bash
esac
```

not `endcase`.

---

# 20. Useful Commands Used in the Lecture

| Command | Role in lecture |
|---|---|
| `echo` | Print messages/values |
| `cd` | Change directory |
| `ls` | List files |
| `wc -l` | Count lines |
| `wc -c` | Count bytes |
| `du -sh` / `du -hs` | Disk usage, human-readable summary |
| `find` | Search/find filesystem entries |
| `tail -n +2` | Skip the first line in shown listing examples |
| `grep` | Search/filter matching text |
| `awk` | Process/extract fields from text |
| `tr -d '%'` | Remove `%` characters in the shown pipeline |
| `ping -c 1` | Send one ping request in the server example |
| `mv` | Move/rename files |
| `touch` | Create/update a file |
| `tar -czf` | Create a gzip-compressed tar archive |
| `date` | Produce current date/time output |
| `chmod` | Change permissions |
| `scp` | Copy files to/from a remote server |
| `exit 1` | Terminate script with a nonzero status in the example |
| `df -h` | Show filesystem disk-space information |
| `ps` / login-related commands | Used in the assignment-style system/process tasks |

---

# 21. Pipelines

The lecture uses the pipe operator repeatedly:

```bash
command1 | command2 | command3
```

The output of one command becomes the input to the next.

Example:

```bash
ls | wc -l
```

Conceptually:

```text
ls output -> wc -l -> number of lines
```

Another example:

```bash
df -h | grep "/dev/disk3s2" | awk '{print $5}' | tr -d '%'
```

This is a very important lab-test pattern: **combine simple Linux commands into a useful pipeline**.

---

# 22. `scp` - Copying a File to a Remote Server

The lecture gives:

```text
scp <source> <destination>
```

The remote home directory is written as:

```text
$SERVER:~/
```

`~/` means the home directory of the remote user.

Example idea:

```bash
scp notes.txt "$SERVER:~/"
```

The source file needs to exist in the current directory unless a full path is provided.

---

# 23. System / Login Information Practice

The lecture asks scripts to report information such as:

- current date
- current logged-in user
- current working directory
- all logged-in users
- total number of logged-in users
- hostname
- available disk space
- available memory
- system uptime

The important lab skill is not just memorizing one command; it is combining shell commands into a small reusable report script.

---

# 24. Assignment Question Patterns = Lab-Test Patterns

The final part of the lecture contains nine questions. These are especially useful as lab-test preparation because they combine the lecture topics.

## Q1. Project summary

Write `project_summary.sh` that:

- stores project directory name in a variable
- checks whether it exists
- counts files
- shows total disk usage
- lists files sorted by modification time
- prints `Analysis Completed`

**Concepts combined:** variables + file tests + `if` + `find/ls` + `wc` + `du`.

---

## Q2. Student data analysis

Write `analyze.sh` that:

- checks whether `students.csv` exists
- prints an error and terminates if absent
- otherwise prints total lines
- counts CSE students
- counts EE students
- prints `Analysis Completed`

**Concepts combined:** `-f` + `if/else` + `wc` + `grep`.

---

## Q3. Find largest directories

Write a script that:

- looks inside the home directory
- finds the 10 largest directories
- sorts largest to smallest
- shows human-readable sizes
- ignores permission-denied errors
- prints only the largest 10 entries

**Concepts combined:** `find`/filesystem traversal + sorting/filtering + redirection/error handling.

---

## Q4. Duplicate file names

Given thousands of files in subdirectories, write commands to display filenames occurring more than once regardless of location.

**Main idea:** produce a list of names, sort/group them, and identify repeated names.

---

## Q5. Directory health report

Accept multiple directory names as input.

For each directory:

- verify existence
- count files
- display disk usage
- classify it as:
  - Small: `<100 MB`
  - Medium: `100 MB-1 GB`
  - Large: `>1 GB`

Must use:

- functions
- loops
- if-else

**Very important lab pattern:** this question combines almost the entire lecture.

---

## Q6. System information report

Generate a report containing:

- date and time
- logged-in user
- hostname
- current working directory
- available disk space
- available memory
- system uptime

Save the report in a text file.

**Concepts:** commands + variables/redirection + report generation.

---

## Q7/Q8. User login report

The lecture repeats essentially the same task:

- all users currently logged in
- login time of each user
- total number logged in
- check whether a specified user is logged in
- last 10 user logins
- number of unique users who logged in
- most recent logins
- save the report to a file

**Concepts:** command pipelines + arguments + filtering + counting + output redirection.

---

## Q9. Process status and management

Accept a process name and:

- check whether it is running
- display PID
- display CPU usage
- display memory usage
- print a suitable message when not found
- save the report in a text file

**Concepts:** arguments + process commands + conditionals + formatted output.

---

# 25. High-Yield Syntax Sheet

## Script

```bash
#!/bin/bash
```

## Variable

```bash
x="value"
echo "$x"
```

## Script argument

```bash
x="$1"
```

## Number of arguments

```bash
$#
```

## Function

```bash
func() {
    echo "$1"
}
func "hello"
```

## If

```bash
if [ condition ]
then
    ...
elif [ condition ]
then
    ...
else
    ...
fi
```

## File exists

```bash
[ -e "$file" ]
```

## Regular file

```bash
[ -f "$file" ]
```

## Negation

```bash
[ ! -e "$file" ]
```

## For

```bash
for x in ...
do
    ...
done
```

## While

```bash
while [ condition ]
do
    ...
done
```

## Case

```bash
case "$x" in
    pattern)
        ...;;
    *)
        ...;;
esac
```

## User input

```bash
read name
```

## One character

```bash
read -n 1 gender
```

## Command substitution

```bash
result=$(command)
```

## Arithmetic

```bash
x=$((x + 1))
```

## Make executable

```bash
chmod +x file.sh
```

## Run current-directory script

```bash
./file.sh
```

## Pipeline

```bash
cmd1 | cmd2 | cmd3
```

## Remote copy

```bash
scp source destination
```

---

# 26. Quiz Revision - Questions You Should Be Able to Answer Quickly

Try answering these without looking at the answers.

### Q1
What is a shell script?

### Q2
Are Bash scripts compiled or interpreted?

### Q3
Why can an error in a shell script occur after earlier commands have already executed?

### Q4
What does `#!/bin/bash` indicate?

### Q5
Why can `./script.sh` give `permission denied` for a newly created script?

### Q6
What does `chmod +x script.sh` do?

### Q7
What does `./` represent?

### Q8
What does `$0` represent?

### Q9
What does `$1` represent?

### Q10
What does `$#` represent?

### Q11
What is the difference between `$*` and `$@` according to the lecture?

### Q12
What does `$$` represent?

### Q13
What does `$?` represent?

### Q14
What does `!!` do?

### Q15
How do you pass a directory to a script?

### Q16
How do you store a value in a shell variable?

### Q17
What does `$(date ...)` do?

### Q18
What is the syntax of a shell function?

### Q19
Inside a function, what does `$1` refer to?

### Q20
What does `[ -e "$file" ]` test?

### Q21
What does `[ -f "$file" ]` test?

### Q22
What does `-ge` mean?

### Q23
What is the difference between `[ ]` and `[[ ]]` as described in the lecture?

### Q24
What does a `for` loop do?

### Q25
Why is `"${servers[@]}"` used in the server example?

### Q26
What condition controls a `while` loop?

### Q27
What does `read -n 1 gender` do?

### Q28
What closes a `case` statement?

### Q29
What does `echo -n` do?

### Q30
What is the purpose of the pipe `|`?

---

# 27. Quiz Answer Key

1. A text file containing one or more shell commands executed sequentially by a shell interpreter.
2. Interpreted.
3. The shell executes earlier commands before reaching the erroneous instruction.
4. It specifies the Bash interpreter path.
5. The file does not have execute permission by default.
6. Adds execute permission.
7. The current directory.
8. Name of the current script.
9. First command-line argument.
10. Number of command-line arguments.
11. The lecture describes `$*` as all arguments as a single string and `$@` as all arguments as separate strings.
12. PID of the current shell/script.
13. Exit status of the previously executed command.
14. Repeats the previous command; it is not a parameter/variable.
15. Put it after the script name: `./script.sh directory`.
16. `x="value"`.
17. Executes the command and substitutes its output.
18. `name() { commands; }`.
19. The first argument passed to that function.
20. Whether the path exists.
21. Whether the path is a regular file.
22. Greater than or equal to.
23. `[ ]` is the basic test command form; `[[ ]]` is the extended Bash test form.
24. Repeats commands for items in a list.
25. To expand all elements of the array.
26. The specified condition must remain true.
27. Reads one character into `gender`.
28. `esac`.
29. Prints without a newline.
30. Sends the output of one command to the input of the next command.

---

# 28. Lab-Test Practice Set

Do these in a terminal without copying the examples from the lecture.

## Practice 1 - Argument + file check

Write `checkdir.sh` that accepts a directory name.

Requirements:

- if it does not exist, print an error and exit
- otherwise print the directory name
- print its disk usage
- print the number of entries

Skills: `$1`, `if`, `-e`, `du`, `ls`, `wc`.

---

## Practice 2 - Function

Write a script with a function:

```text
report_dir()
```

The function receives a directory as its first argument and prints:

- directory name
- disk usage
- number of files

Call the function for two directories.

Skills: functions + function arguments + commands.

---

## Practice 3 - Loop over directories

Store several directory names in a list/array and, using a `for` loop, print whether each exists.

Skills: array + `for` + `if` + `-e`.

---

## Practice 4 - While counter

Write a script that prints:

```text
Line 1
Line 2
Line 3
Line 4
Line 5
```

using a `while` loop.

Skills: `while`, `-le`, arithmetic expansion.

---

## Practice 5 - `case`

Read one character from the user and handle:

```text
A -> option A
B -> option B
C -> option C
anything else -> invalid
```

Skills: `read -n 1`, `case`, `esac`, patterns.

---

## Practice 6 - Command substitution

Store the current date/time in a variable and print:

```text
Current time: <value>
```

Skills: `date`, `$(...)`, variables.

---

## Practice 7 - Combined lab question

Write a script that accepts multiple directory names.

For each directory:

1. Check whether it exists.
2. If not, print a message and continue.
3. Count files.
4. Display disk usage.
5. Print a size classification.
6. Use a function for the report.

This closely follows the structure of the lecture's directory-health assignment.

---

# 29. Common Lab-Test Mistakes

## Mistake 1 - Forgetting execute permission

```bash
chmod +x file.sh
```

## Mistake 2 - Forgetting `./`

```bash
./file.sh
```

## Mistake 3 - Spaces around assignment

Use:

```bash
x=10
```

not:

```bash
x = 10
```

## Mistake 4 - Forgetting `$` when reading a variable

Assignment:

```bash
x=10
```

Use:

```bash
echo "$x"
```

## Mistake 5 - Wrong `if` closing keyword

Correct:

```bash
fi
```

## Mistake 6 - Wrong `case` closing keyword

Correct:

```bash
esac
```

## Mistake 7 - Forgetting `;;` in `case`

Each case arm in the lecture examples ends with:

```bash
;;
```

## Mistake 8 - Forgetting the function argument rule

Inside:

```bash
func() {
    echo "$1"
}
```

`$1` is the first argument supplied to `func`, not automatically the script's `$1` in the sense used by the function example.

## Mistake 9 - Forgetting to update a `while` counter

For example:

```bash
x=$((x + 1))
```

## Mistake 10 - Forgetting to quote paths

Prefer the lecture's pattern:

```bash
ls -rtl "$directory"
```

---

# 30. Last-Minute Revision Order

When you have only a short time before the quiz/lab test, revise in this order:

### Tier 1 - Must know

1. `#!/bin/bash`
2. `chmod +x`
3. `./script.sh`
4. `$0`, `$1`, `$#`, `$@`, `$*`, `$$`, `$?`
5. variables
6. command substitution `$()`
7. `if/elif/else/fi`
8. `-e`, `-f`, `-ge`
9. `for`
10. `while`
11. functions and function `$1`
12. `read`
13. `case/esac`
14. pipelines `|`

### Tier 2 - Practice

- `ls | wc -l`
- `du -sh "$dir"`
- `grep -c ...`
- `find ... | wc -l`
- `df -h | grep ... | awk ... | tr ...`
- `scp source destination`
- `mv`, `touch`, `tar -czf`

### Tier 3 - Full lab combinations

Practice the lecture's questions on:

- project summary
- CSV analysis
- largest directories
- duplicate filenames
- directory health report
- system report
- login report
- process report

---

# 31. One-Page Mental Model

Think of a Bash lab problem as building blocks:

```text
INPUT
  |
  +--> command-line arguments ($1, $2, ...)
  |
  +--> read user input (read)
  |
  v
STORE
  |
  +--> variables
  +--> arrays
  |
  v
DECIDE
  |
  +--> if / elif / else
  +--> test conditions: [ ] / [[ ]]
  +--> case
  |
  v
REPEAT
  |
  +--> for
  +--> while
  |
  v
PROCESS
  |
  +--> ls / find / grep / wc / du / df / awk / tr / etc.
  +--> pipelines: |
  +--> command substitution: $(...)
  |
  v
OUTPUT
  |
  +--> echo
  +--> file/report output
  |
  v
REUSE
  |
  +--> functions
```

This is the core structure behind most of the lecture's assignment questions.

---

# 32. Source Coverage

This note covers the lecture material on:

- shell scripting basics
- motivation for scripting
- script creation
- Bash interpreter line
- permissions and execution
- special shell parameters
- command-line arguments
- functions and function arguments
- command substitution
- variables
- `if/then/else`
- file tests
- `for` loops and arrays
- `while` loops
- `read`
- `case`
- system/login information tasks
- `scp`
- the nine assignment/lab-style questions

**Source basis:** CS 699 Lecture 2 PDF, pages 1-34.

---

# 33. Ultra-Short Cheat Sheet

```bash
#!/bin/bash

# variable
x="hello"

# script argument
name="$1"

# number of arguments
$#

# current script
$0

# previous exit status
$?

# command substitution
t=$(date)

# function
f() {
    echo "$1"
}

# if
if [ -f "$file" ]
then
    echo "found"
else
    echo "not found"
fi

# for
for x in a b c
do
    echo "$x"
done

# while
x=1
while [ $x -le 3 ]
do
    echo "$x"
    x=$((x + 1))
done

# read
read name

# case
case "$x" in
    a) echo "A";;
    *) echo "other";;
esac

# permissions
chmod +x script.sh

# run current-directory script
./script.sh

# pipeline
ls | wc -l
```

---

## Final self-check before the test

You are ready to attempt a basic lab question when you can write a script from scratch that uses:

```text
$1 + variable + if + loop + function + one Linux command pipeline
```

without needing to look up the basic syntax.
