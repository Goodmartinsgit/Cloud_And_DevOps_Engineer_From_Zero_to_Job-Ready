

# Bash Scripting & Automation
## A Complete Learning Book for Cloud & DevOps Engineers
### Beginner to Advanced — With Real-World Tasks

---

> **Who this book is for:** Cloud and DevOps engineering students who want to go from zero shell knowledge to writing production-grade automation scripts used by real teams at real companies. No prior Bash knowledge is assumed.

---

## Table of Contents

1. [Introduction — Why Bash Still Rules the Server](#introduction)
2. [Chapter 1 — Shebang, Script Execution, chmod +x, PATH Best Practices](#chapter-1)
3. [Chapter 2 — Variables: Strings, Integers, Arrays, Associative Arrays](#chapter-2)
4. [Chapter 3 — Special Variables: $0, $1–$9, $#, $@, $?, $$, $!](#chapter-3)
5. [Chapter 4 — User Input: read, getopts, Positional Parameters](#chapter-4)
6. [Chapter 5 — Conditionals: if/elif/else, case, test, [ ] vs [[ ]] vs (( ))](#chapter-5)
7. [Chapter 6 — Loops: for, while, until, C-style for, break, continue](#chapter-6)
8. [Chapter 7 — Functions: Definition, Local Variables, Return Values, Libraries](#chapter-7)
9. [Chapter 8 — Exit Codes and Error Handling: set -e, set -u, set -o pipefail, trap](#chapter-8)
10. [Chapter 9 — Text Processing: grep, sed, awk, cut, sort, uniq, tr, xargs, tee](#chapter-9)
11. [Chapter 10 — Regular Expressions: BRE, ERE, Character Classes, Anchors, Groups](#chapter-10)
12. [Chapter 11 — File Operations: Reading Line by Line, Heredocs, Process Substitution](#chapter-11)
13. [Chapter 12 — Cron and Crontab: Syntax, Special Strings, cron.d, logrotate](#chapter-12)
14. [Chapter 13 — Signals: SIGTERM, SIGKILL, SIGHUP, trap for Cleanup](#chapter-13)
15. [Chapter 14 — Script Best Practices: ShellCheck, Modularity, Documentation, Idempotency](#chapter-14)
16. [Final Chapter — How It All Connects: A Real-World DevOps Workflow](#final-chapter)

---

## Introduction — Why Bash Still Rules the Server {#introduction}

Imagine you're a DevOps engineer at a company that runs 200 servers. Every morning, you need to:

- Check the disk space on all 200 servers and send an alert if any is above 80%
- Archive last week's application logs and delete them from the servers
- Create new user accounts for three new developers who just joined
- Run database backups, verify they succeeded, and log the results

If you did each of these tasks manually, clicking around in a terminal — one server at a time — you'd spend your entire day doing repetitive, error-prone busywork. And what happens the next morning? You do it all again.

**This is exactly the problem Bash scripting solves.**

A Bash script is a plain text file containing a list of commands. When you run that file, the computer executes every command in order — automatically, consistently, at whatever speed your machine can handle. The same script that takes you 10 seconds to run can process all 200 servers in the time it would take you to open a single terminal window.

### What is Bash?

Bash stands for **B**ourne **A**gain **SH**ell. It is the default command-line interpreter on the vast majority of Linux systems (and available on macOS and Windows via WSL). When you open a terminal and type a command like `ls` or `cd`, you are already using Bash. A script is simply a file full of those same commands, saved so you can run them again and again.

### Why DevOps Engineers Need Bash

Modern DevOps is built on automation. Cloud infrastructure (AWS, Azure, GCP) provides powerful APIs, but the glue that holds automated workflows together — startup scripts, deployment pipelines, health checks, log rotation, user management — is almost always Bash. Even when tools like Ansible, Terraform, or Kubernetes are in use, Bash scripts appear at every layer of the stack.

Hiring managers and senior engineers universally expect DevOps candidates to be comfortable writing and reading Bash. It is, without exaggeration, the single most transferable technical skill you can have as a systems or cloud engineer.

### What You Will Learn

This book covers 14 topics in the depth required to work as a professional. By the end, you will be able to:

- Write scripts from scratch that are safe, readable, and maintainable
- Handle real-world scenarios: backups, deployments, log parsing, user provisioning
- Debug scripts confidently and avoid the mistakes that trip up most beginners
- Schedule automated jobs that run without any human involvement
- Build scripts that survive unexpected failures gracefully

Each chapter ends with a hands-on task modelled on real job scenarios. Complete every one. Reading without practice is how people stay beginners.

Let's begin.

---

## Chapter 1 — Shebang, Script Execution, chmod +x, PATH Best Practices {#chapter-1}

### The Problem We're Solving

Before you can write automation scripts, you need to understand how the computer knows *what to do* with your script file, *who is allowed to run it*, and *how to find it* without typing its full location every time. This chapter answers all three questions.

### The Shebang Line — Telling the System What to Use

Every Bash script should begin with a special line called a **shebang** (also written as *hashbang*). It looks like this:

```bash
#!/usr/bin/env bash
```

Let's break this down character by character:

- `#!` — These two characters together are the shebang. The operating system recognises this pattern at the very start of a file as meaning "use what follows to interpret this file."
- `/usr/bin/env` — This is a program called `env`. Its job is to find another program in your current environment. We use it here rather than hardcoding a path.
- `bash` — This tells `env` to find `bash` and use it as the interpreter.

**Why not just write `#!/bin/bash`?**

Many scripts on the internet use `#!/bin/bash`. This works on most Linux systems, where Bash lives at `/bin/bash`. However, on some systems (macOS, some Linux distributions), Bash may be at `/usr/local/bin/bash`. Using `#!/usr/bin/env bash` tells the system to search for Bash in the PATH — wherever it is installed — making your script portable across environments.

**Analogy:** Imagine you write a letter in French and put it in a mailbox. The shebang is like writing "Please find a French speaker to translate this" on the envelope. Without that instruction, the postman might try to read it himself in English and get confused.

### Creating Your First Script

Open a terminal and follow these steps:

```bash
# Step 1: Create a new file called hello.sh
# The .sh extension is conventional but not required
nano hello.sh
```

Inside the file, type:

```bash
#!/usr/bin/env bash

# This is a comment. Bash ignores any line starting with #
# Comments are for humans reading the script, not for the computer

echo "Hello, DevOps World!"
echo "Today's date is: $(date)"
```

Line by line explanation:

- `#!/usr/bin/env bash` — Shebang: use Bash to run this file
- `# This is a comment...` — Comments: ignored by Bash, vital for documentation
- `echo "Hello, DevOps World!"` — The `echo` command prints text to the terminal
- `echo "Today's date is: $(date)"` — `$(date)` is called **command substitution**: it runs the `date` command and inserts its output inline

Save the file (in nano: `Ctrl+O`, then `Enter`, then `Ctrl+X`).

### Making Your Script Executable — chmod +x

If you try to run your script right now, you'll get an error:

```bash
./hello.sh
# bash: ./hello.sh: Permission denied
```

This is because Linux has a permission system. Every file has three sets of permissions:

- **r** = read (can you look at the file's contents?)
- **w** = write (can you modify the file?)
- **x** = execute (can you run the file as a program?)

And three groups of people: the file's **owner**, the **group** it belongs to, and **everyone else**.

By default, new text files are not executable. You need to explicitly grant execute permission using `chmod`:

```bash
chmod +x hello.sh
```

- `chmod` — **Ch**ange file **mod**e (permissions)
- `+x` — **Add** e**x**ecute permission
- `hello.sh` — The file to change

You can verify the permissions changed:

```bash
ls -l hello.sh
# -rwxr-xr-x 1 ubuntu ubuntu 72 Jan 15 09:00 hello.sh
#  ^^^ these three characters show owner permissions: r=read, w=write, x=execute
```

Now run it:

```bash
./hello.sh
# Hello, DevOps World!
# Today's date is: Tue Jan 15 09:00:00 UTC 2025
```

The `./` means "in the current directory." You're explicitly saying "run the file called hello.sh that is right here."

### PATH Best Practices — Running Scripts From Anywhere

You may have noticed you always need `./` before the script name. That's because the shell only automatically finds programs that are stored in directories listed in a special variable called `$PATH`.

Check your current PATH:

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/ubuntu/bin
```

Each directory is separated by a colon. When you type a command like `ls`, the shell searches through each of these directories in order until it finds a program named `ls`. It doesn't look in your current directory (for security reasons).

**Where to store your personal scripts:**

The conventional location is `~/bin` (a `bin` folder inside your home directory):

```bash
# Create the directory if it doesn't exist
mkdir -p ~/bin

# Add it to your PATH permanently (add this line to ~/.bashrc)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc

# Reload ~/.bashrc so the change takes effect now
source ~/.bashrc

# Copy your script there
cp hello.sh ~/bin/hello

# Now you can run it from anywhere, without ./
hello
# Hello, DevOps World!
```

For system-wide scripts that all users should access, use `/usr/local/bin/`.

### Common Mistakes Beginners Make

**Mistake 1: Forgetting the shebang**
Without `#!/usr/bin/env bash`, the script may be interpreted by a different shell (`sh`, `dash`, etc.) that behaves differently. Always include the shebang.

**Mistake 2: Windows line endings**
If you create a script on Windows and transfer it to Linux, it may have Windows-style line endings (`\r\n` instead of `\n`). This causes cryptic errors. Fix with: `sed -i 's/\r//' script.sh`

**Mistake 3: Forgetting chmod +x**
After writing a new script, always run `chmod +x scriptname.sh` before trying to execute it.

**Mistake 4: Spaces around the shebang path**
`#! /usr/bin/env bash` (with a space after `#!`) may not work on all systems. Keep it together: `#!/usr/bin/env bash`.

### How This Works in the Real World

In professional environments, scripts are usually:

1. Stored in a Git repository so changes are tracked and reversible
2. Named descriptively: `backup-postgres.sh`, `provision-user.sh`, not `script1.sh`
3. Made executable in the repository (via `git update-index --chmod=+x`)
4. Deployed to servers via CI/CD pipelines, placed in `/usr/local/bin/` or a dedicated `/opt/scripts/` directory

### Key Takeaways

- The shebang (`#!/usr/bin/env bash`) tells the OS which interpreter to use
- `chmod +x script.sh` grants execute permission — required before running
- Use `~/bin` for personal scripts, add it to `$PATH` for convenient access
- Scripts are just text files; any editor works
- Comments (`#`) are essential for maintainability

### Chapter 1 Task

**Task:** Write a script that checks if a file exists, is readable, and prints its size — handle all edge cases.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: file_inspector.sh
# Purpose: Check if a file exists, is readable, and report its size
# Usage: ./file_inspector.sh <filepath>
# =============================================================================

# --- Configuration ---
# We use a variable for the script name so error messages are consistent
SCRIPT_NAME="$(basename "$0")"

# --- Input Validation ---
# Check that the user supplied exactly one argument
if [[ $# -ne 1 ]]; then
    # $# is the number of arguments passed to the script
    # We print the error to stderr (standard error), not stdout
    echo "ERROR: Expected exactly 1 argument, got $#." >&2
    echo "Usage: $SCRIPT_NAME <filepath>" >&2
    exit 1   # Exit code 1 signals an error
fi

# Assign the first argument to a named variable for clarity
TARGET_FILE="$1"

# --- Existence Check ---
# The -e flag tests whether a path exists (file, directory, or symlink)
if [[ ! -e "$TARGET_FILE" ]]; then
    echo "ERROR: '$TARGET_FILE' does not exist." >&2
    exit 2   # Different exit codes help callers identify what went wrong
fi

# --- Type Check ---
# The -f flag tests whether the path is a regular file (not a directory or device)
if [[ ! -f "$TARGET_FILE" ]]; then
    echo "ERROR: '$TARGET_FILE' exists but is not a regular file." >&2
    echo "       (It might be a directory, symlink, or device file.)" >&2
    exit 3
fi

# --- Readability Check ---
# The -r flag tests whether the current user has read permission
if [[ ! -r "$TARGET_FILE" ]]; then
    echo "ERROR: '$TARGET_FILE' exists but is not readable by this user." >&2
    echo "       Try: chmod +r '$TARGET_FILE'" >&2
    exit 4
fi

# --- Size Retrieval ---
# wc -c counts bytes in a file
# We use command substitution $(...) to capture the output
FILE_SIZE_BYTES=$(wc -c < "$TARGET_FILE")
# Note: "wc -c < file" redirects the file as input; this avoids wc printing the filename

# Convert to a human-readable format using du
# du -sh gives a human-readable size; awk extracts just the first field
FILE_SIZE_HUMAN=$(du -sh "$TARGET_FILE" | awk '{print $1}')

# --- Success Output ---
echo "================================================"
echo "  File Inspector Report"
echo "================================================"
echo "  Path     : $TARGET_FILE"
echo "  Exists   : Yes"
echo "  Readable : Yes"
echo "  Size     : $FILE_SIZE_BYTES bytes ($FILE_SIZE_HUMAN)"
echo "  Modified : $(stat -c '%y' "$TARGET_FILE" | cut -d'.' -f1)"
echo "================================================"

exit 0   # Exit code 0 means success
```

**How to test it:**

```bash
chmod +x file_inspector.sh

# Test 1: Normal file
echo "Hello World" > testfile.txt
./file_inspector.sh testfile.txt

# Test 2: File doesn't exist
./file_inspector.sh /nonexistent/path.txt

# Test 3: No arguments
./file_inspector.sh

# Test 4: A directory instead of file
./file_inspector.sh /etc

# Test 5: File with no read permission
touch secret.txt
chmod 000 secret.txt
./file_inspector.sh secret.txt
```

Each test exercises a different edge case and a different exit code. This is exactly how production scripts are written and tested.

---

## Chapter 2 — Variables: Strings, Integers, Arrays, Associative Arrays {#chapter-2}

### What is a Variable?

A variable is a named container that holds a value. Think of it like a labelled box: you put something in the box, give the box a name, and later you can retrieve the contents by referencing the name.

In Bash, you create a variable by writing: `VARIABLE_NAME=value`

**Critical rule: No spaces around the equals sign.** This is the most common beginner mistake. `NAME = "Alice"` causes an error; `NAME="Alice"` works.

### String Variables

```bash
#!/usr/bin/env bash

# Creating string variables
FIRST_NAME="Alice"
LAST_NAME="Okafor"
CITY="Lagos"

# Accessing a variable — prefix with $
echo "Name: $FIRST_NAME $LAST_NAME"
echo "City: $CITY"

# Using curly braces — clearer and required in some contexts
echo "Name: ${FIRST_NAME} ${LAST_NAME}"

# When you NEED curly braces
FRUIT="apple"
echo "${FRUIT}s are delicious"  # Prints: apples are delicious
# Without braces: echo "$FRUITs" — Bash looks for variable named "FRUITs" (undefined)

# String concatenation — just place variables side by side
FULL_NAME="${FIRST_NAME} ${LAST_NAME}"
echo "Full name: $FULL_NAME"

# String length
echo "Name has ${#FULL_NAME} characters"   # ${#var} gives string length

# Default values — if a variable is empty or unset, use a fallback
USER_INPUT=""
echo "Value: ${USER_INPUT:-"no input provided"}"   # Prints: no input provided

# Substrings: ${variable:start:length}
TEXT="Hello, DevOps!"
echo "${TEXT:0:5}"    # Prints: Hello  (start at 0, take 5 chars)
echo "${TEXT:7:6}"    # Prints: DevOps (start at 7, take 6 chars)

# String replacement
PATH_NAME="/home/alice/documents/file.txt"
echo "${PATH_NAME/alice/bob}"         # Replace first occurrence
echo "${PATH_NAME//l/L}"              # Replace ALL occurrences (double slash)
echo "${PATH_NAME%.txt}.bak"          # Remove suffix, add new one: file.bak
echo "${PATH_NAME##*/}"               # Remove everything up to last /: file.txt
```

### Integer Variables

Bash treats everything as a string by default. For arithmetic, you need special syntax:

```bash
#!/usr/bin/env bash

# Simple arithmetic with $(( ))
NUM_A=10
NUM_B=3

echo "Addition:       $((NUM_A + NUM_B))"   # 13
echo "Subtraction:    $((NUM_A - NUM_B))"   # 7
echo "Multiplication: $((NUM_A * NUM_B))"   # 30
echo "Division:       $((NUM_A / NUM_B))"   # 3 (integer division — no decimals!)
echo "Modulo:         $((NUM_A % NUM_B))"   # 1 (remainder of 10 ÷ 3)
echo "Exponent:       $((NUM_A ** NUM_B))"  # 1000 (10 to the power of 3)

# Incrementing a variable
COUNTER=0
COUNTER=$((COUNTER + 1))    # COUNTER is now 1
((COUNTER++))               # Shorthand — COUNTER is now 2
((COUNTER+=5))              # COUNTER is now 7
echo "Counter: $COUNTER"

# declare -i marks a variable as integer
declare -i SCORE=0
SCORE=SCORE+10              # With -i, you can do this without $(( ))
echo "Score: $SCORE"        # 10

# For floating point arithmetic, use bc (basic calculator)
RESULT=$(echo "scale=2; 10 / 3" | bc)
echo "Float result: $RESULT"   # 3.33
```

### Arrays (Indexed Arrays)

An array holds multiple values in a single variable, accessed by position number (index), starting at 0.

```bash
#!/usr/bin/env bash

# Creating an array
SERVERS=("web01" "web02" "web03" "db01" "db02")

# Accessing elements
echo "First server:  ${SERVERS[0]}"    # web01
echo "Third server:  ${SERVERS[2]}"    # web03
echo "Last server:   ${SERVERS[-1]}"   # db02 (negative index from end)

# Get all elements
echo "All servers: ${SERVERS[@]}"      # web01 web02 web03 db01 db02
echo "All servers: ${SERVERS[*]}"      # Same, but behaves differently in quotes

# Get number of elements
echo "Total servers: ${#SERVERS[@]}"   # 5

# Get all indices
echo "Indices: ${!SERVERS[@]}"         # 0 1 2 3 4

# Adding elements
SERVERS+=("cache01")                   # Append to end
echo "Updated: ${SERVERS[@]}"

# Modifying an element
SERVERS[1]="web02-upgraded"
echo "Modified: ${SERVERS[@]}"

# Removing an element
unset 'SERVERS[3]'                     # Removes db01; index 3 becomes empty
echo "After removal: ${SERVERS[@]}"   # Note the gap in indices

# Slicing an array: ${array[@]:start:count}
echo "Slice: ${SERVERS[@]:1:3}"       # Elements at index 1, 2, 3

# Iterating over an array (covered more in Chapter 6)
for SERVER in "${SERVERS[@]}"; do
    echo "Processing: $SERVER"
done

# Creating array from command output
RUNNING_PROCESSES=($(ps aux | awk 'NR>1 {print $1}' | sort -u))
echo "Users with processes: ${RUNNING_PROCESSES[@]}"
```

### Associative Arrays (Key-Value Maps)

Associative arrays use named keys instead of numbers — like a dictionary or hash map in other languages.

```bash
#!/usr/bin/env bash

# MUST declare before use with -A
declare -A SERVER_PORTS

# Assigning key-value pairs
SERVER_PORTS["nginx"]=80
SERVER_PORTS["apache"]=8080
SERVER_PORTS["postgres"]=5432
SERVER_PORTS["redis"]=6379

# Accessing values
echo "nginx port:    ${SERVER_PORTS["nginx"]}"
echo "postgres port: ${SERVER_PORTS["postgres"]}"

# Alternative declaration syntax
declare -A DB_CONFIG=(
    ["host"]="db.internal.example.com"
    ["port"]="5432"
    ["name"]="production_db"
    ["user"]="app_user"
)

echo "DB Host: ${DB_CONFIG["host"]}"
echo "DB Port: ${DB_CONFIG["port"]}"

# Get all keys
echo "Keys: ${!DB_CONFIG[@]}"

# Get all values
echo "Values: ${DB_CONFIG[@]}"

# Iterate over key-value pairs
for KEY in "${!DB_CONFIG[@]}"; do
    echo "  $KEY = ${DB_CONFIG[$KEY]}"
done

# Check if key exists
if [[ -v DB_CONFIG["host"] ]]; then
    echo "host key exists"
fi

# Deleting a key
unset 'DB_CONFIG["user"]'
```

### Variable Scope and the export Command

By default, variables are local to the current shell session. Child processes (programs started from your script) cannot see them. Use `export` to make them available to child processes:

```bash
#!/usr/bin/env bash

LOCAL_VAR="only in this shell"
export SHARED_VAR="visible to child processes"

# Start a child process and try to access both variables
bash -c 'echo "Local: $LOCAL_VAR"'     # Prints nothing — not exported
bash -c 'echo "Shared: $SHARED_VAR"'  # Prints: Shared: visible to child processes
```

### Common Mistakes Beginners Make

**Mistake 1: Spaces around =**
`NAME = "Alice"` fails. `NAME="Alice"` works. Bash is strict.

**Mistake 2: Not quoting variables**
`echo $FILENAME` breaks if the filename has spaces. Always use `echo "$FILENAME"`.

**Mistake 3: Forgetting declare -A for associative arrays**
Without `declare -A`, Bash treats the variable as a plain string and gives confusing errors.

**Mistake 4: Using $ when assigning**
`$MY_VAR="hello"` is wrong. Only use `$` when *reading* a variable, not when *setting* it.

### How This Works in the Real World

Variables are the backbone of real scripts. Professionals use them to:

- Store configuration at the top of scripts (server names, file paths, thresholds)
- Make scripts reusable by parameterising values rather than hardcoding them
- Build connection strings and commands dynamically
- Store output of commands for later processing

A script with well-named variables at the top reads like a configuration file — you can hand it to a colleague and they immediately understand what it does.

### Key Takeaways

- Variables: `NAME=value` (no spaces). Access with `$NAME` or `${NAME}`
- String operations: length `${#var}`, replace `${var/old/new}`, substring `${var:start:len}`
- Arithmetic: use `$(( ))` for integers, `bc` for floats
- Arrays: indexed (`ARRAY=(a b c)`) and associative (`declare -A MAP`)
- Quote your variables: always `"$VAR"` not `$VAR`

### Chapter 2 Task

**Task:** Build a CSV processor — reads CSV, validates fields, transforms data, outputs new CSV with summary.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: csv_processor.sh
# Purpose: Read a CSV, validate fields, transform data, output a new CSV
# Usage: ./csv_processor.sh <input.csv> <output.csv>
# =============================================================================

set -euo pipefail   # Covered in detail in Chapter 8 — fail on errors

SCRIPT_NAME="$(basename "$0")"

# --- Argument Validation ---
if [[ $# -ne 2 ]]; then
    echo "Usage: $SCRIPT_NAME <input.csv> <output.csv>" >&2
    exit 1
fi

INPUT_FILE="$1"
OUTPUT_FILE="$2"

if [[ ! -f "$INPUT_FILE" ]]; then
    echo "ERROR: Input file '$INPUT_FILE' not found." >&2
    exit 2
fi

# --- Counters (using integer variables) ---
declare -i TOTAL_ROWS=0
declare -i VALID_ROWS=0
declare -i INVALID_ROWS=0
declare -i SKIPPED_HEADER=0

# --- Associative array for department summary ---
declare -A DEPT_COUNT

# --- Write output header ---
echo "name,email,department,salary_usd,salary_ngn" > "$OUTPUT_FILE"

# Exchange rate (for transformation)
declare -i USD_TO_NGN=1600

# --- Process the CSV ---
# IFS=',' sets comma as the field separator for read
# read -r prevents backslash interpretation
FIRST_LINE=true
while IFS=',' read -r NAME EMAIL DEPARTMENT SALARY REST; do

    # Skip header row
    if [[ "$FIRST_LINE" == "true" ]]; then
        FIRST_LINE=false
        SKIPPED_HEADER=1
        continue
    fi

    ((TOTAL_ROWS++))

    # Trim whitespace from all fields
    NAME="${NAME// /}"
    EMAIL="${EMAIL// /}"
    DEPARTMENT="${DEPARTMENT// /}"
    SALARY="${SALARY// /}"

    # --- Field Validation ---
    VALID=true

    # Validate name (non-empty, only letters, spaces, hyphens)
    if [[ -z "$NAME" ]] || [[ ! "$NAME" =~ ^[A-Za-z[:space:]-]+$ ]]; then
        echo "WARNING: Row $TOTAL_ROWS — Invalid name: '$NAME'" >&2
        VALID=false
    fi

    # Validate email (simple check: contains @ and .)
    if [[ ! "$EMAIL" =~ ^[^@]+@[^@]+\.[^@]+$ ]]; then
        echo "WARNING: Row $TOTAL_ROWS — Invalid email: '$EMAIL'" >&2
        VALID=false
    fi

    # Validate salary (must be a positive integer)
    if [[ ! "$SALARY" =~ ^[0-9]+$ ]]; then
        echo "WARNING: Row $TOTAL_ROWS — Invalid salary: '$SALARY'" >&2
        VALID=false
    fi

    if [[ "$VALID" == "false" ]]; then
        ((INVALID_ROWS++))
        continue
    fi

    # --- Transformation: Convert salary to NGN ---
    SALARY_NGN=$((SALARY * USD_TO_NGN))

    # --- Department count tracking ---
    DEPT_COUNT["$DEPARTMENT"]=$(( ${DEPT_COUNT["$DEPARTMENT"]:-0} + 1 ))

    # --- Write to output CSV ---
    echo "$NAME,$EMAIL,$DEPARTMENT,$SALARY,$SALARY_NGN" >> "$OUTPUT_FILE"

    ((VALID_ROWS++))

done < "$INPUT_FILE"   # Redirect file as input to the while loop

# --- Summary Report ---
echo ""
echo "============================================="
echo "  CSV Processing Summary"
echo "============================================="
echo "  Input file  : $INPUT_FILE"
echo "  Output file : $OUTPUT_FILE"
echo "  Total rows  : $TOTAL_ROWS"
echo "  Valid rows  : $VALID_ROWS"
echo "  Invalid rows: $INVALID_ROWS"
echo "  Exchange rate used: 1 USD = $USD_TO_NGN NGN"
echo ""
echo "  Department Breakdown:"
for DEPT in "${!DEPT_COUNT[@]}"; do
    printf "    %-20s : %d employees\n" "$DEPT" "${DEPT_COUNT[$DEPT]}"
done
echo "============================================="
```

**Sample input CSV (`employees.csv`):**
```
name,email,department,salary
Alice Okafor,alice@example.com,Engineering,75000
Bob Smith,bob@example.com,Marketing,55000
Invalid,,Sales,abc
Charlie Brown,charlie@example.com,Engineering,80000
Diana Prince,diana@example.com,HR,60000
```

**Run it:**
```bash
chmod +x csv_processor.sh
./csv_processor.sh employees.csv employees_transformed.csv
```

---

## Chapter 3 — Special Variables: $0, $1–$9, $#, $@, $?, $$, $! {#chapter-3}

### What Are Special Variables?

Bash has a set of variables that it sets automatically. You cannot assign to most of them — they are read-only, always reflecting the current state of your script's execution. These are called **special variables** or **positional parameters**.

Think of them as your script's built-in dashboard: they tell you what arguments were passed, what happened last, and who's running.

### Positional Parameters: $0, $1–$9, and Beyond

When you run a script and pass arguments after its name, Bash stores those arguments in numbered variables:

```bash
./deploy.sh myapp staging 3
#   $0        $1     $2   $3
```

| Variable | Contains |
|----------|----------|
| `$0` | The name of the script itself |
| `$1` | The first argument |
| `$2` | The second argument |
| `$3` | The third argument |
| ... | ... |
| `$9` | The ninth argument |
| `${10}` | The tenth and beyond (use braces!) |

```bash
#!/usr/bin/env bash
# Script: show_args.sh

echo "Script name: $0"
echo "First arg:   $1"
echo "Second arg:  $2"
echo "Third arg:   $3"
```

```bash
./show_args.sh hello world foo
# Script name: ./show_args.sh
# First arg:   hello
# Second arg:  world
# Third arg:   foo
```

**The `shift` command** — moves arguments down by one position, making it easy to process many arguments in a loop:

```bash
#!/usr/bin/env bash
# Process every argument one by one
while [[ $# -gt 0 ]]; do
    echo "Processing: $1"
    shift   # $2 becomes $1, $3 becomes $2, etc.
done
```

### $# — The Count of Arguments

`$#` tells you how many arguments were passed. Use it for validation:

```bash
#!/usr/bin/env bash
if [[ $# -lt 2 ]]; then
    echo "Error: Need at least 2 arguments, got $#" >&2
    exit 1
fi
```

### $@ and $* — All Arguments

Both `$@` and `$*` refer to all positional parameters. The difference matters when they're inside double quotes:

- `"$@"` — expands to each argument as a separate, properly quoted word
- `"$*"` — joins all arguments into a single string with the first character of `IFS` (usually a space)

In practice, **always use `"$@"`** when you want to pass all arguments along correctly:

```bash
#!/usr/bin/env bash
# This script forwards all its arguments to another command

backup_file() {
    cp "$@"   # Passes all arguments individually — handles spaces in filenames
}

backup_file "my document.txt" /backup/
# Correctly treats "my document.txt" as one filename
```

```bash
#!/usr/bin/env bash
# Demonstrate the difference
ARGS=("hello world" "foo" "bar baz")

echo "Using \$*:"
for ITEM in "${ARGS[*]}"; do
    echo "  [$ITEM]"   # Prints one giant concatenated string
done

echo "Using \$@:"
for ITEM in "${ARGS[@]}"; do
    echo "  [$ITEM]"   # Prints each element correctly
done
```

### $? — The Exit Status of the Last Command

`$?` holds the exit code of the most recently executed command. An exit code of `0` means success; anything else means failure. This is the primary way Bash scripts check whether commands succeeded:

```bash
#!/usr/bin/env bash

# Try to ping a server
ping -c 1 google.com > /dev/null 2>&1

# $? now holds ping's exit code
if [[ $? -eq 0 ]]; then
    echo "Network is reachable"
else
    echo "Network is DOWN! Exit code: $?"
fi

# Better practice: check immediately in the if statement
if ping -c 1 google.com > /dev/null 2>&1; then
    echo "Network is reachable"
else
    echo "Network is DOWN!"
fi
```

**Important:** Every command overwrites `$?`. Save it immediately if you need it later:

```bash
some_command
EXIT_CODE=$?   # Save it right away
echo "Did some other stuff"
echo "The earlier command exited with: $EXIT_CODE"   # Still accurate
```

### $$ — The Current Process ID (PID)

`$$` gives you the Process ID of the current shell running your script. This is extremely useful for creating unique temporary files:

```bash
#!/usr/bin/env bash

# Create a unique temp file using the PID
TEMP_FILE="/tmp/myapp_$$.tmp"
echo "Using temp file: $TEMP_FILE"

# Do work with the temp file
echo "some data" > "$TEMP_FILE"

# Clean up when done (also see: trap in Chapter 13)
rm -f "$TEMP_FILE"
```

Without `$$`, if two instances of your script ran at the same time, they'd both try to use the same temp file and corrupt each other's data.

### $! — The PID of the Last Background Process

When you run a command in the background with `&`, `$!` captures its PID. This lets you wait for it or kill it:

```bash
#!/usr/bin/env bash

# Start a long-running process in the background
sleep 60 &
BACKGROUND_PID=$!   # Capture the PID immediately

echo "Background process started with PID: $BACKGROUND_PID"

# Wait for it to finish
wait $BACKGROUND_PID
echo "Background process completed. Exit code: $?"

# Or kill it if it runs too long
# kill $BACKGROUND_PID
```

### A Complete Reference Script

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: special_vars_demo.sh
# Demonstrates all special variables in context
# =============================================================================

echo "======== Special Variables Demo ========"
echo ""
echo "Script identity:"
echo "  \$0 (script name)     : $0"
echo "  \$\$ (PID)              : $$"
echo ""
echo "Arguments received:"
echo "  \$# (count)           : $#"
echo "  \$@ (all args)        : $@"
echo "  \$1 (first)           : ${1:-<not provided>}"
echo "  \$2 (second)          : ${2:-<not provided>}"
echo ""

# Run a command and check exit status
echo "Running 'true' (always succeeds):"
true
echo "  \$? after 'true'      : $?"

echo "Running 'false' (always fails):"
false
echo "  \$? after 'false'     : $?"

# Background process
sleep 2 &
BG_PID=$!
echo ""
echo "Background process:"
echo "  \$! (bg PID)          : $BG_PID"
wait $BG_PID
echo "  Background done, \$?  : $?"
```

### Common Mistakes Beginners Make

**Mistake 1: Using `$10` instead of `${10}`**
`$10` is parsed as `$1` followed by the character `0`. Use `${10}` for the tenth argument.

**Mistake 2: Not saving $? immediately**
If you check `$?` after running another command, it reflects that other command, not the one you were interested in.

**Mistake 3: Using $* instead of $@**
In almost all cases, `"$@"` is what you want. `"$*"` is rarely the right choice.

### How This Works in the Real World

Special variables are used constantly in:

- **Argument validation** — every production script checks `$#` before proceeding
- **Error handling** — `$?` after every critical command tells you whether to abort
- **Unique temp files** — `$$` prevents collisions between concurrent script instances
- **Background job management** — `$!` lets scripts launch parallel tasks and track them

### Key Takeaways

- `$0` = script name, `$1`–`$9` = positional arguments (use `${10}` and beyond)
- `$#` = number of arguments
- `"$@"` = all arguments individually (prefer this over `"$*"`)
- `$?` = last command's exit code (0 = success, non-zero = failure)
- `$$` = current script's PID — use for unique temp file names
- `$!` = PID of last background process

---

## Chapter 4 — User Input: read, getopts, Positional Parameters {#chapter-4}

### Why Scripts Need Input

A script that only does one hardcoded thing isn't very useful. Real scripts need to accept different values each time they run — which server to deploy to, which user to provision, which date range to query. This input can come from three sources:

1. **Positional parameters** — arguments typed after the script name (covered in Chapter 3)
2. **The `read` command** — interactive prompts during script execution
3. **`getopts`** — structured option parsing (like `-f filename -v -n 5`)

### The read Command — Interactive Input

The `read` command waits for the user to type something and press Enter, then stores what they typed in a variable:

```bash
#!/usr/bin/env bash

# Basic read
echo "What is your name?"
read USERNAME
echo "Hello, $USERNAME!"

# Read with a prompt on the same line using -p
read -p "Enter server name: " SERVER_NAME
echo "You chose: $SERVER_NAME"

# Read silently (for passwords — nothing appears as user types)
read -s -p "Enter password: " PASSWORD
echo ""   # read -s doesn't add a newline, so we add one
echo "Password captured (length: ${#PASSWORD} chars)"

# Read with a timeout — don't wait forever
if read -t 10 -p "Continue? (y/n): " ANSWER; then
    echo "You chose: $ANSWER"
else
    echo ""
    echo "Timeout — using default: yes"
    ANSWER="y"
fi

# Read multiple variables from one line
echo "1.2.3.4 webserver production"
echo "1.2.3.4 webserver production" | read IP HOSTNAME ENVIRONMENT
# Note: read from a pipe runs in a subshell in bash, so variables may not persist
# Use process substitution or herestrings instead:
read IP HOSTNAME ENVIRONMENT <<< "1.2.3.4 webserver production"
echo "IP: $IP, Host: $HOSTNAME, Env: $ENVIRONMENT"

# Read a full line, preserving spaces (IFS= prevents word splitting)
read -r -p "Enter a sentence: " SENTENCE
echo "You said: $SENTENCE"
```

### Reading From Files

`read` is also the standard way to process a file line by line (covered more in Chapter 11):

```bash
#!/usr/bin/env bash

SERVERS_FILE="servers.txt"

while IFS= read -r LINE; do
    echo "Processing server: $LINE"
done < "$SERVERS_FILE"
```

### Validating User Input

Never trust user input. Always validate:

```bash
#!/usr/bin/env bash

# Function to get a valid port number
get_port() {
    local PORT
    while true; do
        read -p "Enter port number (1-65535): " PORT
        # Check if PORT is a number and within valid range
        if [[ "$PORT" =~ ^[0-9]+$ ]] && (( PORT >= 1 && PORT <= 65535 )); then
            echo "$PORT"
            return 0
        else
            echo "Invalid port. Please enter a number between 1 and 65535." >&2
        fi
    done
}

CHOSEN_PORT=$(get_port)
echo "Will use port: $CHOSEN_PORT"
```

### getopts — Structured Option Parsing

For scripts that need to accept flags and options (like `-f filename`, `-v` for verbose, `-n 5` for count), use `getopts`. It's the standard, POSIX-compliant way to parse options.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: deploy.sh — demonstrates getopts
# Usage: ./deploy.sh -e <environment> -a <app> [-v] [-d]
# =============================================================================

# Default values
ENVIRONMENT=""
APP_NAME=""
VERBOSE=false
DRY_RUN=false

# Usage function — always include one
usage() {
    echo "Usage: $(basename "$0") -e <environment> -a <appname> [-v] [-d]"
    echo ""
    echo "Options:"
    echo "  -e <env>   Environment to deploy to (staging|production)"
    echo "  -a <app>   Application name to deploy"
    echo "  -v         Enable verbose output"
    echo "  -d         Dry run — show what would happen without doing it"
    echo "  -h         Show this help message"
    exit 1
}

# getopts loop
# The string "e:a:vdh" defines valid options:
# - Letters followed by : require an argument (e: means -e requires a value)
# - Letters without : are flags (no argument needed)
while getopts "e:a:vdh" OPTION; do
    case "$OPTION" in
        e)
            ENVIRONMENT="$OPTARG"   # OPTARG holds the value after the flag
            ;;
        a)
            APP_NAME="$OPTARG"
            ;;
        v)
            VERBOSE=true
            ;;
        d)
            DRY_RUN=true
            ;;
        h)
            usage
            ;;
        ?)
            # getopts sets ? for unknown options
            echo "Unknown option. Use -h for help." >&2
            usage
            ;;
    esac
done

# Shift past the options so $1, $2, etc. now refer to non-option arguments
shift $((OPTIND - 1))

# Validate required options
if [[ -z "$ENVIRONMENT" ]] || [[ -z "$APP_NAME" ]]; then
    echo "ERROR: -e and -a are required." >&2
    usage
fi

# Validate environment value
if [[ "$ENVIRONMENT" != "staging" && "$ENVIRONMENT" != "production" ]]; then
    echo "ERROR: Environment must be 'staging' or 'production'." >&2
    exit 1
fi

# Main logic
[[ "$VERBOSE" == true ]] && echo "Verbose mode enabled."
[[ "$DRY_RUN" == true ]] && echo "DRY RUN — no changes will be made."

echo "Deploying '$APP_NAME' to '$ENVIRONMENT'..."

if [[ "$DRY_RUN" == false ]]; then
    echo "  (actual deployment would happen here)"
fi

echo "Done."
```

**Usage examples:**

```bash
chmod +x deploy.sh

./deploy.sh -e staging -a myapi          # Deploy myapi to staging
./deploy.sh -e production -a frontend -v # Deploy with verbose output
./deploy.sh -d -e staging -a myapi      # Dry run
./deploy.sh -h                           # Show help
```

### Combining getopts With a Menu

For complex tools, a menu-driven interface may be more user-friendly than flags:

```bash
#!/usr/bin/env bash
# Simple confirmation prompt — reusable pattern

confirm() {
    local PROMPT="${1:-Are you sure?}"
    local ANSWER
    read -r -p "$PROMPT [y/N]: " ANSWER
    case "$ANSWER" in
        [yY][eE][sS]|[yY]) return 0 ;;   # Yes
        *) return 1 ;;                     # No (default)
    esac
}

if confirm "This will delete all logs. Proceed?"; then
    echo "Deleting logs..."
else
    echo "Aborted."
fi
```

### Common Mistakes Beginners Make

**Mistake 1: Using read inside a pipe**
```bash
# WRONG — read runs in a subshell; USERNAME won't be set outside
echo "Alice" | read USERNAME
echo "$USERNAME"   # Empty!

# RIGHT — use herestring or process substitution
read USERNAME <<< "Alice"
echo "$USERNAME"   # Alice
```

**Mistake 2: Not using -r with read**
Without `-r`, backslashes are treated as escape characters. Always use `read -r`.

**Mistake 3: Forgetting that getopts stops at the first non-option argument**
`./script.sh file.txt -e staging` — getopts won't see `-e staging` if `file.txt` comes first.

### How This Works in the Real World

Professional scripts typically use getopts for CI/CD pipelines and cron jobs (where flags are set programmatically), and `read` for interactive admin tools. Most cloud tooling follows the same pattern: flags for automation, prompts for human-operated tools.

### Key Takeaways

- `read -p "Prompt: " VARNAME` — interactive user input
- `read -s` — silent input (passwords)
- `read -t SECONDS` — timeout
- Always use `read -r` to prevent backslash interpretation
- `getopts "a:b:c"` — parse flags; `:` after a letter means it requires a value
- `$OPTARG` — holds the value for the current option in getopts

---

## Chapter 5 — Conditionals: if/elif/else, case, test, [ ] vs [[ ]] vs (( )) {#chapter-5}

### Making Decisions in Scripts

Every non-trivial script needs to make decisions: "if this file exists, do X; otherwise do Y." Bash has multiple ways to express conditions, and understanding the differences between them is essential to writing correct, safe scripts.

### The if / elif / else Structure

```bash
#!/usr/bin/env bash

DISK_USAGE=87   # Percentage

if [[ $DISK_USAGE -ge 90 ]]; then
    echo "CRITICAL: Disk is almost full!"
elif [[ $DISK_USAGE -ge 75 ]]; then
    echo "WARNING: Disk usage is high: ${DISK_USAGE}%"
else
    echo "OK: Disk usage is normal: ${DISK_USAGE}%"
fi

# General structure:
# if <condition>; then
#     <commands>
# elif <condition>; then
#     <commands>
# else
#     <commands>
# fi
```

The `then` must be on the same line as `if` (separated by `;`) or on the next line. The block always ends with `fi` (if backwards).

### Understanding [ ], [[ ]], and (( ))

This is where many beginners get confused. Here's the definitive breakdown:

#### Single brackets: `[ ]` (the test command)

`[ ]` is actually a program called `test`. It's POSIX-compliant (works in any shell) but has some sharp edges:

```bash
#!/usr/bin/env bash

# String tests with [ ]
NAME="Alice"
[ "$NAME" = "Alice" ] && echo "Name matches"    # Note: = not == for strings in [ ]
[ -z "$NAME" ] && echo "Name is empty"           # -z tests for zero length
[ -n "$NAME" ] && echo "Name is non-empty"       # -n tests for non-zero length

# File tests with [ ]
[ -f /etc/passwd ] && echo "File exists"
[ -d /tmp ] && echo "Directory exists"
[ -r /etc/passwd ] && echo "File is readable"

# ALWAYS quote variables inside [ ] — unquoted spaces cause syntax errors
FILENAME="my file.txt"
[ -f $FILENAME ]    # BROKEN — treated as: [ -f my file.txt ] — "too many args"
[ -f "$FILENAME" ]  # CORRECT
```

#### Double brackets: `[[ ]]` (Bash built-in)

`[[ ]]` is a Bash keyword (not a program), smarter and safer:

```bash
#!/usr/bin/env bash

NAME="Alice Okafor"
ENV="production"

# Pattern matching (doesn't work in [ ])
[[ "$ENV" == prod* ]] && echo "This is a production environment"

# Regex matching with =~
[[ "$NAME" =~ ^[A-Z] ]] && echo "Name starts with uppercase"

# Logical operators: && and || (no need for -a and -o like in [ ])
[[ -f /etc/hosts && -r /etc/hosts ]] && echo "hosts file is readable"

# No word splitting — unquoted variables are safe
[[ -f $NAME ]]   # Won't break even without quotes (though quoting is still good practice)

# No need to quote empty strings
EMPTY=""
[[ -z $EMPTY ]] && echo "Empty variable works fine without quotes here"
```

**Rule of thumb: Use `[[ ]]` in Bash scripts. Use `[ ]` only if you need to write POSIX sh for maximum portability.**

#### Double parentheses: `(( ))` — arithmetic evaluation

```bash
#!/usr/bin/env bash

COUNT=5

# Pure arithmetic — no $ needed inside (( ))
if (( COUNT > 3 )); then
    echo "Count is greater than 3"
fi

if (( COUNT % 2 == 0 )); then
    echo "Count is even"
else
    echo "Count is odd"
fi

# Combining with variables
MAX=100
CURRENT=87
if (( CURRENT >= MAX * 0.9 )); then
    echo "Over 90% of maximum"
fi

# (( )) returns 0 (success) if expression is non-zero, 1 (failure) if zero
(( 1 )) && echo "True"    # Prints
(( 0 )) && echo "True"    # Does not print
```

### File Test Operators

These are critical for DevOps scripts:

| Operator | Meaning |
|----------|---------|
| `-e FILE` | File/directory exists |
| `-f FILE` | Is a regular file |
| `-d FILE` | Is a directory |
| `-r FILE` | Is readable |
| `-w FILE` | Is writable |
| `-x FILE` | Is executable |
| `-s FILE` | File exists and has size > 0 |
| `-L FILE` | Is a symbolic link |
| `FILE1 -nt FILE2` | FILE1 is newer than FILE2 |
| `FILE1 -ot FILE2` | FILE1 is older than FILE2 |

### String Comparison Operators

| Operator | Meaning |
|----------|---------|
| `=` or `==` | Equal (use `==` in `[[ ]]`) |
| `!=` | Not equal |
| `<` | Less than (alphabetically) |
| `>` | Greater than (alphabetically) |
| `-z STR` | String is empty (zero length) |
| `-n STR` | String is non-empty |

### Numeric Comparison Operators (for `[ ]` and `[[ ]]`)

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-lt` | Less than |
| `-le` | Less than or equal |
| `-gt` | Greater than |
| `-ge` | Greater than or equal |

*Inside `(( ))`, you can use `==`, `!=`, `<`, `>`, `<=`, `>=` directly.*

### The case Statement

`case` is like a multi-branch if/elif but much cleaner for matching a variable against multiple patterns:

```bash
#!/usr/bin/env bash

read -p "Enter environment (dev/staging/production): " ENV

case "$ENV" in
    dev|development)
        # Matches either "dev" or "development"
        echo "Using development configuration"
        DEBUG=true
        LOG_LEVEL="debug"
        ;;
    staging)
        echo "Using staging configuration"
        DEBUG=false
        LOG_LEVEL="info"
        ;;
    prod|production)
        echo "Using production configuration — be careful!"
        DEBUG=false
        LOG_LEVEL="warn"
        ;;
    *)
        # * is the wildcard — matches anything not caught above
        echo "ERROR: Unknown environment '$ENV'" >&2
        exit 1
        ;;
esac   # End of case (case backwards)

echo "Debug: $DEBUG, Log level: $LOG_LEVEL"
```

Each case is:
- A **pattern** (which can use wildcards like `*`, `?`, or alternatives with `|`)
- Followed by `)`
- Commands
- `;;` to end that case (like a `break`)
- `esac` closes the whole thing

### Practical Conditional Examples

```bash
#!/usr/bin/env bash

# Check if running as root
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root." >&2
    exit 1
fi

# Check if a command exists
if ! command -v docker &>/dev/null; then
    echo "Docker is not installed. Installing..." 
    apt-get install -y docker.io
fi

# Check if a service is running
if systemctl is-active --quiet nginx; then
    echo "nginx is running"
else
    echo "nginx is not running — starting..."
    systemctl start nginx
fi

# Ternary-style one-liner
STATUS=$( [[ -f /tmp/lockfile ]] && echo "locked" || echo "free" )
echo "Status: $STATUS"
```

### Common Mistakes Beginners Make

**Mistake 1: Using = inside (( ))**
`(( a = 5 ))` is assignment, not comparison. Use `(( a == 5 ))` for comparison.

**Mistake 2: Missing spaces inside brackets**
`[-f file]` fails; `[ -f file ]` works. Brackets need spaces inside.

**Mistake 3: Using == for numeric comparison**
`[[ "10" == "9" ]]` is a *string* comparison (and it's false alphabetically). Use `-eq` or `(( ))` for numbers.

**Mistake 4: Forgetting ;; in case**
Missing `;;` causes cases to fall through.

### Key Takeaways

- `[[ ]]` is the Bash-preferred test — safer, more features, use it by default
- `(( ))` for arithmetic conditions — supports standard math operators
- `[ ]` for POSIX portability — requires more quoting care
- `case` is cleaner than long if/elif chains for pattern matching
- File tests (`-f`, `-d`, `-r`, `-w`, `-x`) are essential for defensive scripting

---

## Chapter 6 — Loops: for, while, until, C-style for, break, continue {#chapter-6}

### What Are Loops?

A loop is a way to repeat a block of commands multiple times. Without loops, if you wanted to process 100 servers, you'd need to write 100 identical blocks of code. With a loop, you write it once.

### The for Loop — Iterating Over a List

```bash
#!/usr/bin/env bash

# Loop over a literal list
for FRUIT in apple banana mango pineapple; do
    echo "Processing: $FRUIT"
done

# Loop over an array (always use "${ARRAY[@]}")
SERVERS=("web01" "web02" "db01" "cache01")
for SERVER in "${SERVERS[@]}"; do
    echo "Checking status of: $SERVER"
    ping -c 1 -W 1 "$SERVER" &>/dev/null && echo "  UP" || echo "  DOWN"
done

# Loop over files in a directory
for LOG_FILE in /var/log/*.log; do
    echo "Log file: $LOG_FILE (size: $(du -sh "$LOG_FILE" | cut -f1))"
done

# Loop over command output
for USER in $(cut -d: -f1 /etc/passwd); do
    echo "User: $USER"
done

# Loop over a number range
for NUM in {1..5}; do
    echo "Iteration $NUM"
done

# Loop with step value
for NUM in {0..20..5}; do   # 0, 5, 10, 15, 20
    echo "Value: $NUM"
done
```

### C-style for Loop

For when you need classic counter-based iteration:

```bash
#!/usr/bin/env bash

# C-style: (( init; condition; increment ))
for (( i=0; i<5; i++ )); do
    echo "Counter: $i"
done

# Count down
for (( i=10; i>=0; i-- )); do
    printf "\rCountdown: %d  " "$i"
    sleep 1
done
echo ""
echo "Launch!"

# Nested loops
for (( row=1; row<=3; row++ )); do
    for (( col=1; col<=3; col++ )); do
        printf "(%d,%d) " "$row" "$col"
    done
    echo ""
done
```

### The while Loop — Loop While Condition Is True

`while` repeats as long as its condition evaluates to true:

```bash
#!/usr/bin/env bash

# Basic while loop
COUNTER=1
while [[ $COUNTER -le 5 ]]; do
    echo "Iteration $COUNTER"
    (( COUNTER++ ))
done

# Reading a file line by line (canonical pattern)
while IFS= read -r LINE; do
    echo "Line: $LINE"
done < /etc/hosts

# Wait for a service to become available
MAX_WAIT=30
ELAPSED=0
while ! pg_isready -h localhost -p 5432 &>/dev/null; do
    if (( ELAPSED >= MAX_WAIT )); then
        echo "ERROR: Database did not start within ${MAX_WAIT}s" >&2
        exit 1
    fi
    echo "Waiting for database... (${ELAPSED}s elapsed)"
    sleep 2
    (( ELAPSED += 2 ))
done
echo "Database is ready!"

# Infinite loop — runs until explicitly broken
while true; do
    echo "Running health check..."
    # ... check something ...
    sleep 60
done
```

### The until Loop — Loop Until Condition Becomes True

`until` is the opposite of `while` — it loops while the condition is *false*:

```bash
#!/usr/bin/env bash

# Keep trying until a command succeeds
ATTEMPTS=0
until curl -s http://localhost:8080/health | grep -q '"status":"ok"'; do
    (( ATTEMPTS++ ))
    echo "Attempt $ATTEMPTS: Service not ready, waiting..."
    sleep 5
    if (( ATTEMPTS >= 12 )); then
        echo "ERROR: Service failed to become healthy after 60s" >&2
        exit 1
    fi
done
echo "Service is healthy after $ATTEMPTS attempts!"
```

### break and continue — Controlling Loop Flow

```bash
#!/usr/bin/env bash

# break — exit the loop immediately
for SERVER in web01 web02 CRITICAL_DOWN web03 web04; do
    if [[ "$SERVER" == "CRITICAL_DOWN" ]]; then
        echo "Critical server found — stopping processing"
        break   # Exits the for loop entirely
    fi
    echo "Processing: $SERVER"
done

echo "Loop ended (break was hit)"

# continue — skip the rest of this iteration, go to next
for FILE in file1.txt file2.bak file3.txt file4.log file5.txt; do
    # Skip backup files
    if [[ "$FILE" == *.bak ]]; then
        echo "Skipping backup: $FILE"
        continue   # Jump to next iteration
    fi
    echo "Processing: $FILE"
done

# break with nested loops — break N exits N levels of nesting
for (( i=1; i<=3; i++ )); do
    for (( j=1; j<=3; j++ )); do
        if (( i == 2 && j == 2 )); then
            echo "Breaking out of both loops at i=$i, j=$j"
            break 2   # Exit 2 levels of loops
        fi
        echo "i=$i, j=$j"
    done
done
```

### Parallel Processing with Loops

Running tasks in parallel dramatically speeds up scripts that process many items:

```bash
#!/usr/bin/env bash

SERVERS=("web01" "web02" "web03" "db01" "db02" "cache01")
MAX_PARALLEL=3

run_with_limit() {
    local JOB_COUNT
    while true; do
        JOB_COUNT=$(jobs -r | wc -l)   # Count running background jobs
        if (( JOB_COUNT < MAX_PARALLEL )); then
            break
        fi
        sleep 0.1
    done
}

for SERVER in "${SERVERS[@]}"; do
    run_with_limit
    # Run each server check in background with &
    (
        echo "Checking $SERVER..."
        sleep $(( RANDOM % 3 + 1 ))   # Simulate variable-length operation
        echo "$SERVER: OK"
    ) &
done

# Wait for all background jobs to finish
wait
echo "All servers checked."
```

### Common Mistakes Beginners Make

**Mistake 1: Forgetting `"${ARRAY[@]}"` in for loops**
```bash
SERVERS=("web01" "db server" "cache01")
for S in ${SERVERS[@]}; do    # WRONG — "db server" becomes two items
for S in "${SERVERS[@]}"; do  # RIGHT
```

**Mistake 2: Infinite loops without an exit condition**
Always ensure there's a way out — a `break`, a maximum iteration count, or a guaranteed-changing condition.

**Mistake 3: Modifying a file while looping over it**
Reading and writing to the same file in a loop can cause strange behaviour. Write to a temporary file first, then replace.

### Key Takeaways

- `for ITEM in LIST; do ... done` — iterate over known items
- `for (( i=0; i<N; i++ )); do ... done` — counter-based loops
- `while CONDITION; do ... done` — repeat while true
- `until CONDITION; do ... done` — repeat while false (less common)
- `break` exits a loop; `continue` skips to the next iteration
- `break 2` exits two levels of nested loops
- Background `&` + `wait` for parallel processing

---

## Chapter 7 — Functions: Definition, Local Variables, Return Values, Libraries {#chapter-7}

### Why Functions?

Imagine you need to check if a service is running in 10 different places in your script. Without functions, you'd copy and paste the same 5 lines of code 10 times. When you find a bug, you'd need to fix it in 10 places. With a function, you write the logic once and call it by name.

**Functions in Bash are named, reusable blocks of code.** They reduce repetition, make scripts readable, and are the foundation of maintainable automation.

### Defining and Calling Functions

```bash
#!/usr/bin/env bash

# Method 1: function keyword (Bash-specific)
function greet() {
    echo "Hello, $1!"
}

# Method 2: POSIX-compatible syntax (preferred)
greet() {
    echo "Hello, $1!"
}

# Calling a function — just use its name
greet "Alice"        # Hello, Alice!
greet "Bob"          # Hello, Bob!
greet "DevOps team"  # Hello, DevOps team!
```

**Important:** Functions must be defined *before* they are called. Bash reads scripts top to bottom. Define all functions at the top (or in a sourced library), then call them below.

### Positional Parameters Inside Functions

Inside a function, `$1`, `$2`, etc. refer to *the function's arguments*, not the script's arguments. `$0` still refers to the script name, not the function name:

```bash
#!/usr/bin/env bash

create_backup() {
    local SOURCE_DIR="$1"    # First argument to the function
    local DEST_DIR="$2"      # Second argument
    local FILENAME="$3"      # Third argument

    echo "Backing up $SOURCE_DIR to $DEST_DIR/$FILENAME"
    cp -r "$SOURCE_DIR" "$DEST_DIR/$FILENAME"
}

create_backup /var/www/html /backups "html_$(date +%Y%m%d)"
```

### Local Variables — Keeping Functions Clean

By default, variables inside functions are *global* — they affect the whole script. Use `local` to make variables only exist within the function:

```bash
#!/usr/bin/env bash

# WITHOUT local — dangerous global leak
calculate_bad() {
    RESULT=$((5 * 10))   # Global RESULT is set!
}

RESULT=99
calculate_bad
echo "$RESULT"   # Prints 50, not 99 — the function overwrote it!

# WITH local — clean and safe
calculate_good() {
    local RESULT=$((5 * 10))   # Local to this function
    echo "Inside function: $RESULT"
}

RESULT=99
calculate_good
echo "Outside function: $RESULT"   # Still 99 — untouched

# Always declare loop variables local inside functions
process_servers() {
    local SERVER
    local STATUS
    for SERVER in "$@"; do
        STATUS=$(check_server "$SERVER")
        echo "$SERVER: $STATUS"
    done
}
```

### Return Values

Bash functions communicate results in two ways:

#### Method 1: Exit codes (for success/failure)

`return N` sets the function's exit code (stored in `$?`). Use 0 for success, non-zero for failure:

```bash
#!/usr/bin/env bash

is_port_open() {
    local HOST="$1"
    local PORT="$2"
    # timeout 2: give up after 2 seconds
    # /dev/tcp is a bash special file for TCP connections
    if timeout 2 bash -c ">/dev/tcp/$HOST/$PORT" 2>/dev/null; then
        return 0   # Success — port is open
    else
        return 1   # Failure — port is closed or unreachable
    fi
}

if is_port_open "localhost" "80"; then
    echo "Web server is listening on port 80"
else
    echo "Port 80 is not open"
fi
```

#### Method 2: Echo output (for data)

For returning strings, numbers, or any data, `echo` the value and capture it with `$()`:

```bash
#!/usr/bin/env bash

get_disk_usage() {
    local MOUNT_POINT="${1:-/}"
    # df outputs a table; awk extracts the usage percentage column from line 2
    df -h "$MOUNT_POINT" | awk 'NR==2 {gsub(/%/,""); print $5}'
}

get_timestamp() {
    date "+%Y-%m-%d_%H:%M:%S"
}

USAGE=$(get_disk_usage "/")
TS=$(get_timestamp)

echo "[$TS] Disk usage on /: ${USAGE}%"
```

#### Method 3: Nameref (for complex data — Bash 4.3+)

```bash
#!/usr/bin/env bash

# Return an array via nameref (a reference to the caller's variable)
get_running_services() {
    local -n RESULT_ARRAY="$1"   # -n creates a nameref to the caller's variable
    mapfile -t RESULT_ARRAY < <(systemctl list-units --type=service --state=running \
        --no-heading --no-legend | awk '{print $1}')
}

declare -a MY_SERVICES
get_running_services MY_SERVICES
echo "Running services: ${MY_SERVICES[@]}"
```

### Reusable Library Scripts

For large automation projects, split functions into library files and source them:

```bash
# === File: lib/logging.sh ===
#!/usr/bin/env bash

# Logging library — source this from other scripts

# Log levels
readonly LOG_DEBUG=0
readonly LOG_INFO=1
readonly LOG_WARN=2
readonly LOG_ERROR=3

# Default log level — can be overridden before sourcing
LOG_LEVEL=${LOG_LEVEL:-$LOG_INFO}

# Timestamp format
_timestamp() {
    date "+%Y-%m-%d %H:%M:%S"
}

log_debug() {
    (( LOG_LEVEL <= LOG_DEBUG )) || return 0
    echo "[DEBUG] $(_timestamp) $*" >&2
}

log_info() {
    (( LOG_LEVEL <= LOG_INFO )) || return 0
    echo "[INFO]  $(_timestamp) $*"
}

log_warn() {
    (( LOG_LEVEL <= LOG_WARN )) || return 0
    echo "[WARN]  $(_timestamp) $*" >&2
}

log_error() {
    echo "[ERROR] $(_timestamp) $*" >&2
}
```

```bash
# === File: deploy.sh ===
#!/usr/bin/env bash

# Source the logging library
# ${BASH_SOURCE[0]} is the path to the current script
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "${SCRIPT_DIR}/lib/logging.sh"

log_info "Starting deployment..."
log_warn "Production environment — proceeding with caution"
log_error "This would log an error (for demo purposes)"
```

### Common Mistakes Beginners Make

**Mistake 1: Not using local**
Forgetting `local` makes all function variables global, causing hard-to-debug side effects.

**Mistake 2: Confusing return with echo**
`return` only passes exit codes (0-255). To return a string, `echo` it and capture with `$()`.

**Mistake 3: Calling functions before defining them**
Bash is not like Python or JavaScript — you cannot call a function before it appears in the file.

**Mistake 4: Forgetting to source library files**
`./lib/logging.sh` *executes* the library as a separate process. `source lib/logging.sh` (or `. lib/logging.sh`) loads it into the current shell — what you want.

### Key Takeaways

- Define functions before calling them
- Use `local` for all variables inside functions — always
- Return exit codes with `return 0`/`return 1` for success/failure
- Return data by echoing and capturing with `$(function_name arg1 arg2)`
- Source library files with `source` or `.` to share functions across scripts

---

## Chapter 8 — Exit Codes and Error Handling: set -e, set -u, set -o pipefail, trap {#chapter-8}

### Why Error Handling Matters

Imagine a deployment script that:
1. Pulls new code from Git ✓
2. Builds the application — **fails silently**
3. Replaces the running production app with a broken build ✗

Without proper error handling, the script merrily continues through failure. By the time you notice something is wrong, you've already caused an outage. Error handling is what separates hobby scripts from production-grade automation.

### Exit Codes — The Universal Language of Success and Failure

Every command in Linux returns an exit code when it finishes:
- **0** = success
- **1-255** = failure (different numbers mean different types of failure)

```bash
# See the exit code of the last command
ls /tmp
echo $?   # 0 — success

ls /nonexistent
echo $?   # 2 — file not found (ls-specific code)

grep "nothere" /etc/hosts
echo $?   # 1 — pattern not found
```

Common conventions:
- `1` — General error
- `2` — Misuse of shell built-ins or wrong arguments
- `126` — Command found but not executable
- `127` — Command not found
- `128+N` — Died from signal N (e.g., `130` = killed by Ctrl+C / SIGINT)

### set -e — Exit on Error

```bash
#!/usr/bin/env bash
set -e   # Exit immediately if any command returns non-zero

echo "Step 1: Pulling code"
git pull origin main      # If this fails, script stops here

echo "Step 2: Building"
make build                # If this fails, script stops here

echo "Step 3: Deploying"
./deploy.sh               # Only reaches here if both previous steps succeeded
```

Without `set -e`, a script continues even after failures. With it, the first failure stops everything.

**Exception patterns — when you want to allow failure:**

```bash
#!/usr/bin/env bash
set -e

# This is fine — grep returns 1 if not found, which would abort the script
# Use || true to ignore the exit code intentionally
grep "pattern" file.txt || true   # "|| true" means "if this fails, that's OK"

# Or use if to check explicitly
if grep -q "pattern" file.txt; then
    echo "Found"
fi   # grep's exit code is consumed by if — doesn't trigger set -e

# Or disable set -e temporarily
set +e    # Turn OFF exit-on-error
some_optional_command
RESULT=$?
set -e    # Turn it back ON
```

### set -u — Treat Unset Variables as Errors

```bash
#!/usr/bin/env bash
set -u   # Exit if you use a variable that was never set

echo "Server: $SERVER_NAME"   # Error: SERVER_NAME is unset

# Provide defaults for variables that may not be set
ENVIRONMENT="${ENVIRONMENT:-development}"   # Use "development" if ENVIRONMENT is unset
```

Without `set -u`, `$UNSET_VAR` silently expands to an empty string. This causes bugs like `rm -rf "$BASE_DIR/"` becoming `rm -rf /` when `BASE_DIR` is unset.

### set -o pipefail — Catch Failures in Pipelines

```bash
#!/usr/bin/env bash
set -e
set -o pipefail   # CRITICAL for pipelines

# Without pipefail, this appears to succeed:
cat /nonexistent_file | sort | uniq
# cat fails (exit 1), but sort and uniq succeed (exit 0)
# The pipeline's exit code is the LAST command's code (uniq: 0)
# So set -e doesn't trigger!

# With pipefail, the pipeline returns the exit code of the first failed command
# cat's failure (1) becomes the pipeline's exit code
```

### The Safety Header — Using All Three Together

Every professional Bash script should start with this:

```bash
#!/usr/bin/env bash
set -euo pipefail

# -e: exit on error
# -u: exit on unset variable
# -o pipefail: catch pipeline failures
# All three combined with one set command
```

This trio is called the "strict mode" of Bash and is considered industry best practice.

### trap — Cleanup on Exit and Signal Handling

`trap` lets you run commands automatically when the script exits or receives a signal. This is how you ensure cleanup happens even if the script fails:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Create a temp file
TEMP_FILE=$(mktemp)
TEMP_DIR=$(mktemp -d)

# Register a cleanup function to run on EXIT
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f "$TEMP_FILE"
    rm -rf "$TEMP_DIR"
    echo "Cleanup complete."
}

# trap runs cleanup() whenever the script exits — for ANY reason
# This includes normal exit, errors (set -e), and signals (Ctrl+C)
trap cleanup EXIT

# Now do your work — cleanup will happen no matter what
echo "Working with temp file: $TEMP_FILE"
echo "Data" > "$TEMP_FILE"

echo "Working with temp dir: $TEMP_DIR"
cp /etc/hosts "$TEMP_DIR/"

# Even if we exit here due to an error, cleanup runs
echo "Processing complete."
# Script exits normally here — cleanup still runs
```

### A Comprehensive Error Handling Template

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: production_template.sh
# A robust template for production scripts with full error handling
# =============================================================================

set -euo pipefail

# --- Script metadata ---
readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly TIMESTAMP="$(date +%Y%m%d_%H%M%S)"
readonly LOG_FILE="/var/log/myapp/${SCRIPT_NAME%.sh}_${TIMESTAMP}.log"

# --- Logging ---
log() {
    local LEVEL="$1"; shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$LEVEL] $*" | tee -a "$LOG_FILE"
}
info()  { log "INFO " "$@"; }
warn()  { log "WARN " "$@" >&2; }
error() { log "ERROR" "$@" >&2; }

# --- Cleanup ---
TEMP_FILES=()
cleanup() {
    local EXIT_CODE=$?
    [[ ${#TEMP_FILES[@]} -gt 0 ]] && rm -f "${TEMP_FILES[@]}" 2>/dev/null || true
    if [[ $EXIT_CODE -ne 0 ]]; then
        error "Script failed with exit code $EXIT_CODE"
    else
        info "Script completed successfully"
    fi
}
trap cleanup EXIT

# --- Error handler with line number ---
error_handler() {
    error "Command failed at line $1: $BASH_COMMAND"
}
trap 'error_handler $LINENO' ERR

# --- Create log directory ---
mkdir -p "$(dirname "$LOG_FILE")"

# --- Main logic ---
main() {
    info "Starting $SCRIPT_NAME"

    # Create a tracked temp file
    local TMP
    TMP=$(mktemp)
    TEMP_FILES+=("$TMP")   # Register for cleanup

    info "Doing work..."
    echo "output" > "$TMP"

    info "Done."
}

main "$@"
```

### Building a Retry Function

```bash
#!/usr/bin/env bash

retry() {
    local MAX_ATTEMPTS="${1}"; shift
    local DELAY="${1}"; shift
    local COMMAND=("$@")
    local ATTEMPT=1

    while true; do
        if "${COMMAND[@]}"; then
            return 0
        fi

        if (( ATTEMPT >= MAX_ATTEMPTS )); then
            echo "ERROR: Command failed after $MAX_ATTEMPTS attempts: ${COMMAND[*]}" >&2
            return 1
        fi

        echo "Attempt $ATTEMPT failed. Retrying in ${DELAY}s..."
        sleep "$DELAY"
        (( ATTEMPT++ ))
    done
}

# Usage: retry <max_attempts> <delay_seconds> <command> [args...]
retry 5 10 curl -f http://api.example.com/health
retry 3 2  pg_isready -h localhost -p 5432
```

### Chapter 8 Task — Retry Wrapper with Exponential Backoff

**Task:** Build a retry wrapper function that re-runs a failing command with exponential backoff.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: retry_wrapper.sh
# Purpose: Retry a failing command with exponential backoff
# Usage: ./retry_wrapper.sh [--max-attempts N] [--initial-delay N] -- COMMAND [ARGS]
# =============================================================================

set -euo pipefail

# --- Defaults ---
MAX_ATTEMPTS=5
INITIAL_DELAY=1
MAX_DELAY=60
MULTIPLIER=2

# --- Usage ---
usage() {
    cat <<EOF
Usage: $(basename "$0") [OPTIONS] -- COMMAND [ARGS...]

Options:
  --max-attempts N    Maximum number of attempts (default: $MAX_ATTEMPTS)
  --initial-delay N   Initial delay in seconds (default: $INITIAL_DELAY)
  --max-delay N       Maximum delay in seconds (default: $MAX_DELAY)
  -h, --help          Show this help

Examples:
  $(basename "$0") -- curl -f https://api.example.com/health
  $(basename "$0") --max-attempts 10 --initial-delay 2 -- ping -c1 google.com
EOF
    exit 0
}

# --- Parse arguments ---
while [[ $# -gt 0 ]]; do
    case "$1" in
        --max-attempts) MAX_ATTEMPTS="$2"; shift 2 ;;
        --initial-delay) INITIAL_DELAY="$2"; shift 2 ;;
        --max-delay) MAX_DELAY="$2"; shift 2 ;;
        -h|--help) usage ;;
        --) shift; break ;;   # Everything after -- is the command
        *) echo "Unknown option: $1" >&2; usage ;;
    esac
done

if [[ $# -eq 0 ]]; then
    echo "ERROR: No command specified after --" >&2
    usage
fi

# --- Retry with exponential backoff ---
retry_with_backoff() {
    local COMMAND=("$@")
    local ATTEMPT=1
    local DELAY=$INITIAL_DELAY

    echo "Executing: ${COMMAND[*]}"
    echo "Max attempts: $MAX_ATTEMPTS, Initial delay: ${INITIAL_DELAY}s"
    echo "---"

    while true; do
        echo "[$(date '+%H:%M:%S')] Attempt $ATTEMPT of $MAX_ATTEMPTS..."

        # Run the command; if it succeeds, we're done
        if "${COMMAND[@]}"; then
            echo "[$(date '+%H:%M:%S')] SUCCESS on attempt $ATTEMPT!"
            return 0
        fi

        local EXIT_CODE=$?
        echo "[$(date '+%H:%M:%S')] Command failed (exit code: $EXIT_CODE)"

        # Check if we've exhausted our attempts
        if (( ATTEMPT >= MAX_ATTEMPTS )); then
            echo "[$(date '+%H:%M:%S')] ERROR: All $MAX_ATTEMPTS attempts failed." >&2
            return $EXIT_CODE
        fi

        # Add jitter to prevent thundering herd (multiple scripts retrying simultaneously)
        local JITTER=$(( RANDOM % 3 ))   # 0-2 seconds of random jitter
        local SLEEP_TIME=$(( DELAY + JITTER ))

        echo "[$(date '+%H:%M:%S')] Waiting ${SLEEP_TIME}s before retry..."
        sleep "$SLEEP_TIME"

        # Exponential backoff: double the delay, but cap at MAX_DELAY
        DELAY=$(( DELAY * MULTIPLIER ))
        if (( DELAY > MAX_DELAY )); then
            DELAY=$MAX_DELAY
        fi

        (( ATTEMPT++ ))
    done
}

# --- Run ---
retry_with_backoff "$@"
```

**Test it:**
```bash
chmod +x retry_wrapper.sh

# Test with a command that will succeed
./retry_wrapper.sh -- echo "Hello"

# Test with a command that always fails
./retry_wrapper.sh --max-attempts 4 --initial-delay 1 -- false

# Test with a realistic scenario (will retry if curl fails)
./retry_wrapper.sh --max-attempts 5 --initial-delay 2 -- curl -f http://localhost:8080/health
```

### Key Takeaways

- `set -euo pipefail` is your safety net — use it in every script
- `$?` holds the last command's exit code (0=success, non-zero=failure)
- `trap cleanup EXIT` ensures cleanup runs no matter how the script exits
- `trap 'handler $LINENO' ERR` gives you the line number of failed commands
- Exponential backoff prevents overwhelming services during retry storms

---

## Chapter 9 — Text Processing: grep, sed, awk, cut, sort, uniq, tr, xargs, tee {#chapter-9}

### The Unix Philosophy in Action

One of Linux's most powerful design principles is: "Do one thing and do it well." Text processing tools each do one job, but the magic comes from **chaining them together with pipes** (`|`). The output of one command becomes the input of the next.

This chapter covers the essential toolkit every DevOps engineer uses daily.

### grep — Find Lines Matching a Pattern

`grep` (**G**lobal **R**egular **E**xpression **P**rint) searches for lines matching a pattern:

```bash
# Basic usage: grep PATTERN FILE
grep "ERROR" /var/log/app.log               # Lines containing "ERROR"
grep -i "error" /var/log/app.log            # Case-insensitive
grep -n "error" /var/log/app.log            # Show line numbers
grep -c "error" /var/log/app.log            # Count matching lines
grep -v "DEBUG" /var/log/app.log            # Lines NOT matching (invert)
grep -r "secret_key" /etc/                  # Recursive search in directory
grep -l "error" /var/log/*.log              # List FILES containing the pattern
grep -A 3 "ERROR" /var/log/app.log          # 3 lines After match
grep -B 2 "ERROR" /var/log/app.log          # 2 lines Before match
grep -C 2 "ERROR" /var/log/app.log          # 2 lines Context (before AND after)

# Extended regex (-E or use egrep)
grep -E "ERROR|WARN|CRIT" /var/log/app.log  # Multiple patterns

# Fixed string (-F) — don't treat pattern as regex
grep -F "10.0.0.1" /var/log/access.log      # Literal search, faster for fixed strings

# Practical examples
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head -10
# ^ Find top 10 IPs with failed SSH login attempts
```

### sed — Stream Editor for Substitutions and Transformations

`sed` processes text line by line, applying edits. Its most common use is substitution:

```bash
# Syntax: sed 's/FIND/REPLACE/FLAGS' file

# Replace first occurrence per line
sed 's/http:/https:/' urls.txt

# Replace ALL occurrences per line (g = global flag)
sed 's/old_server/new_server/g' config.txt

# Case-insensitive replacement
sed 's/error/ERROR/gi' log.txt

# Edit file in place (modifies the file directly)
sed -i 's/localhost/db.internal/g' app.conf

# Delete lines matching a pattern
sed '/^#/d' config.txt          # Delete comment lines
sed '/^$/d' config.txt          # Delete empty lines

# Print only matching lines (like grep)
sed -n '/ERROR/p' app.log

# Print line range
sed -n '10,20p' file.txt        # Print lines 10 to 20

# Insert a line after a match
sed '/\[database\]/a host = db.internal' config.ini

# Replace a specific line number
sed '5s/.*/new content/' file.txt   # Replace line 5 entirely

# Multiple commands with -e
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Using a delimiter other than / (useful when paths are involved)
sed 's|/old/path|/new/path|g' file.txt
```

### awk — Powerful Column-Based Processing

`awk` is a full programming language for processing structured text. It's unparalleled for processing columns and records:

```bash
# awk '{action}' file
# awk processes one line at a time; $1, $2, etc. refer to columns (fields)

# Print the second column of a file (space-separated)
awk '{print $2}' data.txt

# Print multiple columns with custom formatting
awk '{printf "%-20s %s\n", $1, $3}' data.txt

# Set custom field separator with -F
awk -F: '{print $1}' /etc/passwd          # Print usernames (colon-separated)
awk -F, '{print $1, $3}' data.csv         # Print columns 1 and 3 of a CSV

# Conditional actions
awk '$3 > 1000 {print $1, $3}' data.txt   # Print if column 3 > 1000

# Built-in variables
# NR = current line number
# NF = number of fields in current line
# FS = field separator (default: whitespace)
# OFS = output field separator

awk 'NR==1 {print "Header: " $0; next} {print NR": "$0}' file.txt

# Skip header and process data
awk 'NR > 1 {print $2, $4}' report.csv

# Sum a column
awk -F, '{sum += $4} END {print "Total:", sum}' sales.csv

# Multiple-file processing / statistics
awk '
BEGIN {
    print "Starting analysis..."
    total = 0
    count = 0
}
/ERROR/ {
    count++
    total += $5   # Assume column 5 has error codes
}
END {
    print "Total errors: " count
    if (count > 0) print "Average code: " total/count
}' /var/log/app.log
```

### cut — Extract Fields by Position

```bash
# Cut by delimiter
cut -d: -f1 /etc/passwd                    # Field 1, colon-delimited (usernames)
cut -d, -f1,3 data.csv                     # Fields 1 and 3, comma-delimited

# Cut by character position
cut -c1-10 file.txt                        # Characters 1 through 10
cut -c5- file.txt                          # From character 5 to end of line
```

### sort — Sort Lines

```bash
sort file.txt                              # Alphabetical sort
sort -r file.txt                           # Reverse order
sort -n numbers.txt                        # Numeric sort (not alphabetical)
sort -rn numbers.txt                       # Numeric, reversed (largest first)
sort -k2 file.txt                          # Sort by second field
sort -t, -k3 -n data.csv                  # Sort CSV by 3rd column numerically
sort -u file.txt                           # Sort and remove duplicates
```

### uniq — Remove or Count Duplicate Lines

`uniq` only removes *adjacent* duplicates — almost always use it after `sort`:

```bash
sort access.log | uniq                     # Remove duplicate lines
sort access.log | uniq -c                 # Count occurrences of each line
sort access.log | uniq -d                 # Show only duplicated lines
sort access.log | uniq -u                 # Show only unique (non-duplicated) lines
sort ips.txt | uniq -c | sort -rn | head  # Top 10 most frequent IPs
```

### tr — Translate or Delete Characters

```bash
# tr 'FROM' 'TO' — translates characters one-for-one
echo "Hello World" | tr 'a-z' 'A-Z'      # Uppercase: HELLO WORLD
echo "Hello World" | tr 'A-Z' 'a-z'      # Lowercase: hello world
echo "a:b:c:d"     | tr ':' ','          # Replace colons with commas: a,b,c,d
echo "Hello   World" | tr -s ' '         # Squeeze repeated spaces: Hello World
echo "Hello, World!" | tr -d '[:punct:]' # Delete punctuation: Hello World
cat file.txt | tr -d '\r'                # Remove Windows carriage returns
```

### xargs — Pass Input as Arguments

```bash
# xargs takes stdin and converts it to command arguments
find /tmp -name "*.log" -mtime +7 | xargs rm      # Delete old log files
cat server_list.txt | xargs -I{} ssh {} 'uptime'  # SSH to each server
                                                   # {} is replaced by each line

# Parallel execution with xargs
cat urls.txt | xargs -P 4 -I{} curl -o /dev/null -s {}   # Fetch 4 URLs at a time
```

### tee — Split Output to File and Stdout

```bash
# tee writes to both stdout AND a file — great for logging
some_command | tee output.log              # See output AND save it
some_command | tee -a output.log          # Append instead of overwrite

# Real-world: log deployment output while still seeing it in the terminal
./deploy.sh 2>&1 | tee -a "/var/log/deploy_$(date +%Y%m%d).log"
```

### Putting It All Together — Pipeline Examples

```bash
#!/usr/bin/env bash
# Real-world pipeline examples

# 1. Top 10 IPs attacking via SSH
echo "=== Top SSH Attack Sources ==="
grep "Failed password" /var/log/auth.log \
    | awk '{print $11}' \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -10

# 2. Find the biggest files in /var and their owners
echo "=== Largest Files in /var ==="
find /var -type f -printf '%s %p\n' 2>/dev/null \
    | sort -rn \
    | head -20 \
    | awk '{printf "%.1f MB  %s\n", $1/1024/1024, $2}'

# 3. Extract all unique HTTP status codes from nginx log
echo "=== HTTP Status Code Breakdown ==="
awk '{print $9}' /var/log/nginx/access.log \
    | sort \
    | uniq -c \
    | sort -rn

# 4. Parse a CSV and calculate average salary
echo "=== Average Salary ==="
tail -n +2 employees.csv \
    | cut -d, -f4 \
    | awk '{sum+=$1; count++} END {printf "Average: $%.2f\n", sum/count}'
```

### Chapter 9 Task — Log Parser

**Task:** Build a log parser that reads a log file, extracts ERRORs and WARNINGs, emails a summary.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: log_parser.sh
# Purpose: Parse a log file, extract ERRORs/WARNINGs, email a report
# Usage: ./log_parser.sh --log /var/log/app.log --email admin@example.com
# =============================================================================

set -euo pipefail

# --- Defaults ---
LOG_FILE=""
RECIPIENT=""
REPORT_DIR="/tmp/log_reports"
HOURS_BACK=24   # Analyse last N hours of logs
SUBJECT_PREFIX="[Log Report]"

# --- Parse args ---
while [[ $# -gt 0 ]]; do
    case "$1" in
        --log)     LOG_FILE="$2"; shift 2 ;;
        --email)   RECIPIENT="$2"; shift 2 ;;
        --hours)   HOURS_BACK="$2"; shift 2 ;;
        *) echo "Unknown option: $1" >&2; exit 1 ;;
    esac
done

[[ -z "$LOG_FILE" ]] && { echo "ERROR: --log is required" >&2; exit 1; }
[[ -z "$RECIPIENT" ]] && { echo "ERROR: --email is required" >&2; exit 1; }
[[ ! -f "$LOG_FILE" ]] && { echo "ERROR: Log file not found: $LOG_FILE" >&2; exit 1; }

# --- Setup ---
mkdir -p "$REPORT_DIR"
REPORT_FILE="$REPORT_DIR/report_$(date +%Y%m%d_%H%M%S).txt"
HOSTNAME_=$(hostname)

generate_report() {
    local LOG="$1"

    echo "============================================="
    echo " Log Analysis Report"
    echo " Host     : $HOSTNAME_"
    echo " Log file : $LOG"
    echo " Period   : Last ${HOURS_BACK} hours"
    echo " Generated: $(date '+%Y-%m-%d %H:%M:%S')"
    echo "============================================="
    echo ""

    # Total line count
    local TOTAL_LINES
    TOTAL_LINES=$(wc -l < "$LOG")
    echo "Total log lines analysed: $TOTAL_LINES"
    echo ""

    # Error extraction
    local ERROR_COUNT
    ERROR_COUNT=$(grep -c "ERROR" "$LOG" 2>/dev/null || echo 0)
    local WARN_COUNT
    WARN_COUNT=$(grep -c "WARN" "$LOG" 2>/dev/null || echo 0)

    echo "Error summary:"
    echo "  ERROR count : $ERROR_COUNT"
    echo "  WARN count  : $WARN_COUNT"
    echo ""

    # Most frequent error messages
    if [[ $ERROR_COUNT -gt 0 ]]; then
        echo "--- Top 10 ERROR Messages ---"
        grep "ERROR" "$LOG" \
            | sed 's/^[0-9T:. -]* //' \
            | sort \
            | uniq -c \
            | sort -rn \
            | head -10 \
            | awk '{printf "  (%d times) %s\n", $1, substr($0, index($0,$2))}'
        echo ""
    fi

    if [[ $WARN_COUNT -gt 0 ]]; then
        echo "--- Top 10 WARNING Messages ---"
        grep "WARN" "$LOG" \
            | sed 's/^[0-9T:. -]* //' \
            | sort \
            | uniq -c \
            | sort -rn \
            | head -10 \
            | awk '{printf "  (%d times) %s\n", $1, substr($0, index($0,$2))}'
        echo ""
    fi

    echo "--- Recent Errors (last 20) ---"
    grep "ERROR\|WARN" "$LOG" | tail -20 | sed 's/^/  /'
    echo ""
    echo "============================================="
    echo "End of report"
}

# Generate the report
generate_report "$LOG_FILE" > "$REPORT_FILE"

# Display it
cat "$REPORT_FILE"

# Email it (requires mail/sendmail to be configured on the system)
if command -v mail &>/dev/null; then
    mail -s "${SUBJECT_PREFIX} $(hostname) - $(date '+%Y-%m-%d')" \
         "$RECIPIENT" < "$REPORT_FILE"
    echo "Report sent to: $RECIPIENT"
else
    echo "Note: 'mail' command not found. Report saved to: $REPORT_FILE"
    echo "Install mailutils: apt-get install mailutils"
fi
```

### Key Takeaways

- `grep` finds matching lines; `-v` inverts, `-i` is case-insensitive, `-E` enables extended regex
- `sed 's/old/new/g'` replaces text; `-i` edits files in place
- `awk` processes columns: `$1`, `$2`...; `NR` is line number; `NF` is field count
- `cut -d, -f2` extracts a specific field by delimiter
- `sort | uniq -c | sort -rn` is the canonical "count and rank" pattern
- `tee` lets you see output AND save it simultaneously
- Pipes (`|`) chain tools together — this is the heart of Unix power

---

## Chapter 10 — Regular Expressions: BRE, ERE, Character Classes, Anchors, Groups {#chapter-10}

### What Are Regular Expressions?

A regular expression (regex) is a pattern for matching text. Instead of searching for an exact string like "error", you can search for patterns like "any line starting with a date, followed by the word ERROR."

Regex is the language that powers `grep`, `sed`, `awk`, and many other tools. Once you understand it, your text processing abilities multiply enormously.

**Analogy:** Think of regex like a very precise search query. Google's search accepts plain text. Regex is like adding wildcards, AND/OR logic, and position anchors — but for any text, not just the web.

### BRE vs ERE — Two Flavours of Regex

There are two main regex dialects you'll encounter in Bash:

**BRE (Basic Regular Expressions)** — used by default in `grep` and `sed`

**ERE (Extended Regular Expressions)** — more powerful syntax; enabled with:
- `grep -E` or `egrep`
- `sed -E`
- `awk` uses ERE by default

The main difference is which characters need to be escaped:

| Feature | BRE | ERE |
|---------|-----|-----|
| Grouping | `\(…\)` | `(…)` |
| Alternation | `\|` | `\|` or `\|` |
| One or more | `\+` | `+` |
| Zero or one | `\?` | `?` |

In modern scripting, **ERE is recommended** — it's cleaner and more readable.

### The Building Blocks of Regular Expressions

#### Literal Characters

Most characters match themselves:
```
grep "nginx" /var/log/syslog    # Matches lines containing "nginx" literally
```

#### The Dot — Match Any Character

`.` matches any single character (except newline):
```
grep "err.r" log.txt    # Matches "error", "errer", "err7r", "err r", etc.
```

#### Anchors — Match Position

```bash
^    # Beginning of line
$    # End of line

grep "^ERROR" log.txt       # Lines that START with "ERROR"
grep "\.log$" filelist.txt  # Lines that END with ".log"
grep "^$" file.txt          # Empty lines (nothing between start and end)
grep "^.\{80,\}$" file.txt  # Lines with 80 or more characters
```

#### Character Classes — `[...]`

```bash
[abc]       # Match a, b, or c
[a-z]       # Match any lowercase letter
[A-Z]       # Match any uppercase letter
[0-9]       # Match any digit
[a-zA-Z0-9] # Match any alphanumeric character
[^abc]      # Match anything EXCEPT a, b, or c (negation with ^)

# POSIX classes (work in [ ]):
[:alpha:]   # Letters
[:digit:]   # Digits
[:alnum:]   # Letters and digits
[:space:]   # Whitespace (space, tab, newline, etc.)
[:upper:]   # Uppercase letters
[:lower:]   # Lowercase letters
[:punct:]   # Punctuation

grep "[0-9]\{3\}-[0-9]\{4\}" phones.txt  # Match phone numbers: 555-1234 (BRE)
grep -E "[0-9]{3}-[0-9]{4}" phones.txt   # Same in ERE — cleaner
```

#### Quantifiers — How Many Times?

```bash
*       # Zero or more of the preceding
+       # One or more (ERE only; BRE: \+)
?       # Zero or one (ERE only; BRE: \?)
{n}     # Exactly n times
{n,}    # At least n times
{n,m}   # Between n and m times

grep -E "col{2,4}our" text.txt    # Matches: collour, colllour, etc.
grep -E "https?" urls.txt         # Matches http and https
grep -E "[0-9]+" data.txt         # Lines with one or more digits
```

#### Alternation — OR Logic

```bash
grep -E "ERROR|WARNING|CRITICAL" log.txt    # Match any of these words
grep -E "nginx|apache|httpd" process.txt    # Match any web server
```

#### Grouping — `(...)` in ERE

```bash
grep -E "(ERROR|WARN): " log.txt           # Group for alternation
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" text.txt  # Match IPv4 addresses
# Breakdown: ([0-9]{1,3}\.){3} = one-to-three digits followed by a dot, three times
#            [0-9]{1,3} = one-to-three digits (the final octet)
```

#### Regex in Bash `[[ ]]` with =~

```bash
#!/usr/bin/env bash

validate_ip() {
    local IP="$1"
    local REGEX='^([0-9]{1,3}\.){3}[0-9]{1,3}$'

    if [[ "$IP" =~ $REGEX ]]; then
        echo "$IP looks like a valid IPv4 address"
        return 0
    else
        echo "$IP is NOT a valid IPv4 address" >&2
        return 1
    fi
}

validate_ip "192.168.1.100"    # Valid
validate_ip "256.1.2.3"        # Passes format check (deeper validation requires awk/python)
validate_ip "not-an-ip"        # Invalid

# Capturing groups with BASH_REMATCH
LOG_LINE="2025-01-15 09:30:45 ERROR Database connection failed"
REGEX='^([0-9]{4}-[0-9]{2}-[0-9]{2}) ([0-9]{2}:[0-9]{2}:[0-9]{2}) ([A-Z]+) (.+)$'

if [[ "$LOG_LINE" =~ $REGEX ]]; then
    echo "Date   : ${BASH_REMATCH[1]}"   # 2025-01-15
    echo "Time   : ${BASH_REMATCH[2]}"   # 09:30:45
    echo "Level  : ${BASH_REMATCH[3]}"   # ERROR
    echo "Message: ${BASH_REMATCH[4]}"   # Database connection failed
fi
```

### sed with Regex

```bash
#!/usr/bin/env bash

# Replace all IP addresses with [REDACTED]
sed -E 's/[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}/[REDACTED]/g' access.log

# Extract just email addresses from a file
grep -Eo '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' contacts.txt

# Remove HTML tags
sed -E 's/<[^>]+>//g' page.html

# Extract version numbers
grep -Eo 'v[0-9]+\.[0-9]+\.[0-9]+' releases.txt

# Replace config value
# Before: db_host = old.db.server.com
# After:  db_host = new.db.server.com
sed -E 's/(db_host\s*=\s*).*/\1new.db.server.com/' config.ini
# \1 refers to the first capture group — preserves "db_host = "
```

### Common Regex Patterns for DevOps

```bash
# IPv4 address
^([0-9]{1,3}\.){3}[0-9]{1,3}$

# Email address
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

# URL
^https?://[a-zA-Z0-9.-]+(/[a-zA-Z0-9._~:/?#\[\]@!$&'()*+,;=-]*)?$

# ISO date (YYYY-MM-DD)
^[0-9]{4}-[0-9]{2}-[0-9]{2}$

# Time (HH:MM:SS)
^([0-1][0-9]|2[0-3]):[0-5][0-9]:[0-5][0-9]$

# Semantic version (v1.2.3)
^v?[0-9]+\.[0-9]+\.[0-9]+$

# Username (letters, numbers, underscores, hyphens, 3-20 chars)
^[a-zA-Z0-9_-]{3,20}$

# AWS ARN
^arn:[a-z0-9-]+:[a-z0-9-]+:[a-z0-9-]*:[0-9]{12}:[a-zA-Z0-9:/_-]+$
```

### Key Takeaways

- `.` matches any character; `^` anchors to start of line; `$` anchors to end
- `[abc]`, `[a-z]`, `[^xyz]` — character classes
- `*` zero-or-more; `+` one-or-more; `?` zero-or-one; `{n,m}` range
- Use `-E` with grep/sed for ERE (cleaner syntax)
- `=~` in `[[ ]]` for in-script regex matching; capture groups in `BASH_REMATCH`
- Store regex patterns in variables (no quotes around the variable when used with `=~`)

---

## Chapter 11 — File Operations: Reading Line by Line, Heredocs, Process Substitution {#chapter-11}

### Reading Files Line by Line

The canonical pattern for reading a file line by line in Bash:

```bash
#!/usr/bin/env bash

# The safe, correct way to read a file line by line
while IFS= read -r LINE; do
    echo "Processing: $LINE"
done < /etc/hosts

# Breaking down the magic:
# IFS=              — Set Internal Field Separator to empty; preserves leading/trailing whitespace
# read -r           — Don't interpret backslashes as escape sequences
# done < /etc/hosts — Redirect the file as stdin to the while loop

# Also handle the last line even if it has no trailing newline:
while IFS= read -r LINE || [[ -n "$LINE" ]]; do
    echo "$LINE"
done < file.txt
```

### Reading Files With Specific Delimiters

```bash
#!/usr/bin/env bash

# Read CSV: each line becomes NAME, EMAIL, ROLE
while IFS=',' read -r NAME EMAIL ROLE; do
    echo "User: $NAME ($ROLE) — $EMAIL"
done < users.csv

# Read /etc/passwd fields (colon-separated)
while IFS=: read -r USERNAME PASSWD UID GID COMMENT HOME SHELL; do
    if [[ "$SHELL" == "/bin/bash" ]]; then
        echo "Bash user: $USERNAME (home: $HOME)"
    fi
done < /etc/passwd
```

### Heredocs — Embedding Multi-Line Text in Scripts

A heredoc lets you write multi-line text directly in your script without external files:

```bash
#!/usr/bin/env bash

# Basic heredoc syntax
cat << EOF
This is line 1
This is line 2
Variables work here: $HOME
Command substitution: $(date)
EOF

# The word after << (here EOF, but can be anything) marks the end
# The closing word must be on its own line with no indentation (unless using <<-)

# Indented heredoc with <<-
# (strips leading TABS — not spaces — from both body and closing word)
if true; then
    cat <<- EOF
        This line has leading tabs stripped
        So does this one
        Closing word can be indented too
    EOF
fi

# Heredoc to a file
cat > /etc/nginx/sites-available/myapp.conf << 'EOF'
server {
    listen 80;
    server_name myapp.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
    }
}
EOF
# Note: using 'EOF' (quoted) prevents variable expansion — $ is literal

# Heredoc with variable expansion (default behaviour)
APP_PORT=3000
cat > /tmp/app.conf << EOF
port=$APP_PORT
host=$(hostname)
EOF

# Heredoc to a command
grep -E "ERROR|WARN" << EOF
2025-01-15 09:00:00 INFO Starting
2025-01-15 09:01:00 ERROR Database down
2025-01-15 09:01:05 WARN Retrying
2025-01-15 09:02:00 INFO Connected
EOF

# Heredoc to a variable (herestring alternative for multi-line)
read -r -d '' TEMPLATE << 'EOF' || true
Dear ${NAME},

Your account has been provisioned.
Server: ${SERVER}

Regards,
DevOps Team
EOF
```

### Herestrings — Single-Line Input

A herestring (`<<<`) passes a single string as stdin — shorter and faster than a pipe:

```bash
#!/usr/bin/env bash

# Instead of: echo "hello world" | read -r A B
read -r A B <<< "hello world"
echo "A=$A B=$B"   # A=hello B=world

# Instead of: echo "192.168.1.1" | grep -o '[0-9]*$'
LAST_OCTET=$(grep -o '[0-9]*$' <<< "192.168.1.1")
echo "Last octet: $LAST_OCTET"   # 1

# Useful for parsing strings
VERSION="2.15.3"
IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"
echo "Major: $MAJOR, Minor: $MINOR, Patch: $PATCH"
```

### Process Substitution — Treat Command Output as a File

Process substitution `<(command)` makes the output of a command appear as if it were a file. It solves the classic "variables set inside a pipe subshell are lost" problem:

```bash
#!/usr/bin/env bash

# PROBLEM: Variable doesn't persist after pipe
grep "ERROR" /var/log/app.log | while IFS= read -r LINE; do
    COUNT=$(( COUNT + 1 ))   # This runs in a subshell!
done
echo "Count: $COUNT"   # 0 — variable was lost!

# SOLUTION: Process substitution — while loop runs in current shell
COUNT=0
while IFS= read -r LINE; do
    (( COUNT++ ))
done < <(grep "ERROR" /var/log/app.log)
echo "Count: $COUNT"   # Correct!

# Compare two sorted outputs without temp files
diff <(sort file1.txt) <(sort file2.txt)

# Feed command output to a command that requires a filename
wc -l <(find /var/log -name "*.log")

# Multiple process substitutions
paste <(cut -d: -f1 /etc/passwd | sort) \
      <(cut -d: -f7 /etc/passwd | sort) \
      | column -t
```

### Writing to Files — Output Redirection

```bash
#!/usr/bin/env bash

# Overwrite a file (creates if doesn't exist)
echo "New content" > output.txt

# Append to a file
echo "Additional line" >> output.txt

# Redirect stderr to a file
command 2> errors.txt

# Redirect both stdout and stderr to the same file
command > all_output.txt 2>&1
command &> all_output.txt    # Bash shorthand

# Redirect stdout to file, keep stderr on screen
command > output.txt

# Discard output (send to /dev/null — the Linux "trash")
command > /dev/null 2>&1

# Write to multiple files simultaneously with tee
command | tee file1.txt file2.txt   # Writes to both files and stdout
command | tee -a logfile.txt        # Append mode
```

### Chapter 11 Task — System Health Report (HTML Output)

**Task:** Create a full system health report script: CPU, memory, disk, network, top processes — HTML output.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: health_report.sh
# Purpose: Generate a system health report in HTML format
# Usage: ./health_report.sh [--output /path/to/report.html]
# =============================================================================

set -euo pipefail

OUTPUT_FILE="${1:-/tmp/health_report_$(date +%Y%m%d_%H%M%S).html}"
HOSTNAME_=$(hostname)
GENERATED_AT=$(date '+%Y-%m-%d %H:%M:%S')

# Helper to get a coloured status badge
status_badge() {
    local VALUE="$1"
    local WARN_THRESHOLD="$2"
    local CRIT_THRESHOLD="$3"
    local UNIT="${4:-%}"

    if (( $(echo "$VALUE >= $CRIT_THRESHOLD" | bc -l) )); then
        echo "<span class='badge critical'>${VALUE}${UNIT}</span>"
    elif (( $(echo "$VALUE >= $WARN_THRESHOLD" | bc -l) )); then
        echo "<span class='badge warning'>${VALUE}${UNIT}</span>"
    else
        echo "<span class='badge ok'>${VALUE}${UNIT}</span>"
    fi
}

# Gather metrics
CPU_IDLE=$(top -bn1 | grep "Cpu(s)" | awk '{print $8}' | tr -d '%')
CPU_USED=$(echo "100 - $CPU_IDLE" | bc)
MEM_TOTAL=$(free -m | awk 'NR==2{print $2}')
MEM_USED=$(free -m  | awk 'NR==2{print $3}')
MEM_PCT=$(echo "scale=1; $MEM_USED * 100 / $MEM_TOTAL" | bc)
DISK_USED=$(df -h / | awk 'NR==2{print $5}' | tr -d %)
LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | sed 's/,//g' | awk '{print $1}')
UPTIME=$(uptime -p)
CPU_BADGE=$(status_badge "$CPU_USED" 70 90)
MEM_BADGE=$(status_badge "$MEM_PCT" 75 90)
DISK_BADGE=$(status_badge "$DISK_USED" 80 95)

generate_html() {
cat << HTML
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Health Report — ${HOSTNAME_}</title>
<style>
  body { font-family: monospace; background: #1a1a2e; color: #e0e0e0; margin: 0; padding: 20px; }
  h1   { color: #00d4ff; border-bottom: 2px solid #00d4ff; padding-bottom: 10px; }
  h2   { color: #7ec8e3; margin-top: 30px; }
  .meta { color: #888; font-size: 0.9em; }
  table { width: 100%; border-collapse: collapse; margin: 10px 0; }
  th    { background: #16213e; color: #00d4ff; padding: 8px; text-align: left; }
  td    { padding: 6px 8px; border-bottom: 1px solid #333; }
  tr:hover td { background: #1f2a4a; }
  .badge { padding: 3px 10px; border-radius: 4px; font-weight: bold; }
  .ok       { background: #1a5c3a; color: #4cff91; }
  .warning  { background: #5c3a00; color: #ffb84d; }
  .critical { background: #5c0000; color: #ff4d4d; }
  .metric-card { display: inline-block; background: #16213e; border: 1px solid #333;
                 border-radius: 8px; padding: 15px 25px; margin: 10px; text-align: center; min-width: 150px; }
  .metric-value { font-size: 2em; font-weight: bold; }
  pre { background: #111; padding: 15px; border-radius: 6px; overflow-x: auto; font-size: 0.85em; }
</style>
</head>
<body>
<h1>🖥️ System Health Report</h1>
<p class="meta">Host: <strong>${HOSTNAME_}</strong> | Generated: <strong>${GENERATED_AT}</strong> | Uptime: <strong>${UPTIME}</strong></p>

<h2>Overview</h2>
<div>
  <div class="metric-card">
    <div class="metric-value">${CPU_BADGE}</div>
    <div>CPU Usage</div>
  </div>
  <div class="metric-card">
    <div class="metric-value">${MEM_BADGE}</div>
    <div>Memory Usage</div>
  </div>
  <div class="metric-card">
    <div class="metric-value">${DISK_BADGE}</div>
    <div>Disk Usage (/)</div>
  </div>
  <div class="metric-card">
    <div class="metric-value">${LOAD_AVG}</div>
    <div>Load Average (1m)</div>
  </div>
</div>

<h2>Memory Details</h2>
<pre>$(free -h)</pre>

<h2>Disk Usage</h2>
<pre>$(df -h | grep -v tmpfs)</pre>

<h2>Top 10 Processes by CPU</h2>
<pre>$(ps aux --sort=-%cpu | head -11)</pre>

<h2>Top 10 Processes by Memory</h2>
<pre>$(ps aux --sort=-%mem | head -11)</pre>

<h2>Network Interfaces</h2>
<pre>$(ip -brief addr show)</pre>

<h2>Last 20 System Log Entries</h2>
<pre>$(journalctl -n 20 --no-pager 2>/dev/null || tail -20 /var/log/syslog 2>/dev/null || echo "Log unavailable")</pre>

<p class="meta" style="margin-top:40px">Generated by health_report.sh on ${HOSTNAME_}</p>
</body>
</html>
HTML
}

generate_html > "$OUTPUT_FILE"
echo "Health report written to: $OUTPUT_FILE"
```

### Key Takeaways

- `while IFS= read -r LINE; do ... done < file` is the canonical file-reading pattern
- Heredocs (`<< EOF`) embed multi-line text; quoted `'EOF'` prevents variable expansion
- Herestrings (`<<< "string"`) pass single strings as stdin
- Process substitution `< <(command)` solves the subshell-variable-scope problem
- `>` overwrites; `>>` appends; `2>&1` merges stderr into stdout

---

## Chapter 12 — Cron and Crontab: Syntax, Special Strings, cron.d, logrotate {#chapter-12}

### What is Cron?

Cron is the Linux scheduler. It runs commands automatically at specified times — no human needed. A backup at 2 AM, a log cleanup every Sunday, a health check every 5 minutes — cron handles all of it.

Think of cron as an alarm clock that runs a program instead of making noise.

### Crontab Syntax

Every cron job is defined by one line in a **crontab** (cron table). The format is:

```
MIN  HOUR  DOM  MON  DOW  COMMAND
 │    │     │    │    │
 │    │     │    │    └── Day of Week (0-7, 0 and 7 are Sunday)
 │    │     │    └─────── Month (1-12)
 │    │     └──────────── Day of Month (1-31)
 │    └────────────────── Hour (0-23)
 └─────────────────────── Minute (0-59)
```

Each field accepts:
- A specific value: `5`
- A range: `1-5` (Monday through Friday)
- A list: `1,3,5` (Monday, Wednesday, Friday)
- A step: `*/15` (every 15 units)
- A wildcard: `*` (any value)

### Examples

```cron
# Run every minute
* * * * * /path/to/script.sh

# Run at 2:30 AM every day
30 2 * * * /path/to/backup.sh

# Run at midnight on the first of every month
0 0 1 * * /path/to/monthly_report.sh

# Run every 15 minutes
*/15 * * * * /path/to/health_check.sh

# Run at 9 AM on weekdays (Mon-Fri)
0 9 * * 1-5 /path/to/morning_job.sh

# Run at 2 AM and 2 PM
0 2,14 * * * /path/to/twice_daily.sh

# Run every hour between 8 AM and 6 PM on weekdays
0 8-18 * * 1-5 /path/to/work_hours_job.sh

# Run every Sunday at 3 AM
0 3 * * 0 /path/to/weekly_cleanup.sh

# Run on the 1st and 15th of each month at 6 AM
0 6 1,15 * * /path/to/bimonthly_job.sh
```

### Managing Crontabs

```bash
# Edit YOUR crontab (opens in $EDITOR)
crontab -e

# List your current crontab
crontab -l

# Remove your crontab entirely
crontab -r

# Edit another user's crontab (as root)
crontab -u alice -e

# List another user's crontab
crontab -u alice -l
```

### Special Cron Strings

Instead of the five-field syntax, cron accepts special strings:

| String | Equivalent | Meaning |
|--------|-----------|---------|
| `@reboot` | — | Run once at startup |
| `@yearly` | `0 0 1 1 *` | Once a year, January 1st at midnight |
| `@annually` | `0 0 1 1 *` | Same as @yearly |
| `@monthly` | `0 0 1 * *` | Once a month, 1st at midnight |
| `@weekly` | `0 0 * * 0` | Once a week, Sunday at midnight |
| `@daily` | `0 0 * * *` | Once a day, midnight |
| `@midnight` | `0 0 * * *` | Same as @daily |
| `@hourly` | `0 * * * *` | Once an hour, at the top of the hour |

```cron
@reboot   /usr/local/bin/startup_check.sh
@daily    /usr/local/bin/backup.sh
@weekly   /usr/local/bin/cleanup.sh
@monthly  /usr/local/bin/monthly_report.sh
```

### System-Wide Cron: /etc/cron.d

The `crontab -e` command manages per-user crontabs. For system-wide jobs, use `/etc/cron.d/`:

```bash
# Files in /etc/cron.d have an extra field: the user to run as
# Format: MIN HOUR DOM MON DOW USER COMMAND

# /etc/cron.d/my-application
# Backup at 2 AM daily, run as the 'backup' user
0 2 * * * backup /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Health check every 5 minutes as www-data
*/5 * * * * www-data /opt/scripts/healthcheck.sh
```

You can also use the drop-in directories for common intervals:
- `/etc/cron.hourly/` — scripts run every hour
- `/etc/cron.daily/` — scripts run daily
- `/etc/cron.weekly/` — scripts run weekly
- `/etc/cron.monthly/` — scripts run monthly

Just drop executable scripts into these directories — no crontab editing needed.

### Cron Best Practices

**1. Always use full paths:**
Cron runs with a minimal environment. `$PATH` is not what you expect. Always use `/usr/bin/python3`, not `python3`.

```cron
# WRONG — cron can't find 'backup.sh' without a full path
0 2 * * * backup.sh

# RIGHT
0 2 * * * /usr/local/bin/backup.sh

# RIGHT — use a wrapper that sets the environment
0 2 * * * /bin/bash -l /usr/local/bin/backup.sh
```

**2. Redirect output to a log file:**
Cron emails output by default (if mail is configured). Better to redirect to a file:
```cron
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

**3. Set MAILTO="" to suppress emails:**
```cron
MAILTO=""
0 2 * * * /usr/local/bin/backup.sh
```

**4. Add a lock to prevent overlapping runs:**
```bash
#!/usr/bin/env bash
# Use flock to ensure only one instance runs at a time
exec 200>/var/run/myjob.lock
flock -n 200 || { echo "Job already running, exiting." ; exit 1; }

# Your job code here
echo "Running job at $(date)"
```

### logrotate — Automatic Log Management

`logrotate` is a companion to cron that automatically rotates, compresses, and deletes old log files:

```
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily                    # Rotate daily
    missingok                # Don't error if log file is missing
    rotate 14                # Keep 14 rotated copies
    compress                 # Compress old log files with gzip
    delaycompress            # Don't compress the most recent rotated file
    notifempty               # Don't rotate empty files
    create 0640 www-data adm # Create new log file with these permissions
    sharedscripts            # Run postrotate script only once (not per file)
    postrotate
        # Tell nginx to reopen its log files after rotation
        nginx -s reopen 2>/dev/null || true
    endscript
}
```

Test your logrotate config:
```bash
logrotate -d /etc/logrotate.d/myapp    # Debug — show what would happen (dry run)
logrotate -f /etc/logrotate.d/myapp    # Force rotation now
```

### Chapter 12 Task — Schedule 5 Cron Jobs

**Task:** Schedule 5 different cron jobs: backup at 2am, log cleanup weekly, health check every 5 mins.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: setup_cron_jobs.sh
# Purpose: Install all required cron jobs for the system
# Usage: sudo ./setup_cron_jobs.sh
# =============================================================================

set -euo pipefail

[[ $EUID -ne 0 ]] && { echo "Run as root." >&2; exit 1; }

SCRIPTS_DIR="/usr/local/bin"
LOG_DIR="/var/log/scheduled_jobs"

mkdir -p "$LOG_DIR"
chmod 755 "$LOG_DIR"

echo "Installing cron jobs..."

# --- Install the crontab for root ---
cat > /etc/cron.d/sysadmin-jobs << 'CRONTAB'
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=""

# Job 1: Database/file backup at 2:00 AM every day
0 2 * * *  root  /usr/local/bin/backup.sh >> /var/log/scheduled_jobs/backup.log 2>&1

# Job 2: Log cleanup every Sunday at 3:00 AM (weekly)
0 3 * * 0  root  /usr/local/bin/log_cleanup.sh >> /var/log/scheduled_jobs/cleanup.log 2>&1

# Job 3: System health check every 5 minutes
*/5 * * * * root /usr/local/bin/healthcheck.sh >> /var/log/scheduled_jobs/healthcheck.log 2>&1

# Job 4: Monthly usage report on the 1st at 6:00 AM
0 6 1 * *  root  /usr/local/bin/monthly_report.sh >> /var/log/scheduled_jobs/monthly_report.log 2>&1

# Job 5: SSL certificate check every day at 8:00 AM
0 8 * * *  root  /usr/local/bin/ssl_check.sh >> /var/log/scheduled_jobs/ssl_check.log 2>&1
CRONTAB

chmod 644 /etc/cron.d/sysadmin-jobs

echo "Creating script files..."

# --- Script 1: backup.sh ---
cat > "$SCRIPTS_DIR/backup.sh" << 'EOF'
#!/usr/bin/env bash
set -euo pipefail
BACKUP_DIR="/backups/$(date +%Y/%m/%d)"
mkdir -p "$BACKUP_DIR"
echo "[$(date)] Starting backup..."
tar -czf "$BACKUP_DIR/etc_$(date +%H%M).tar.gz" /etc 2>/dev/null
tar -czf "$BACKUP_DIR/var_www_$(date +%H%M).tar.gz" /var/www 2>/dev/null
echo "[$(date)] Backup complete: $BACKUP_DIR"
# Keep only last 30 days of backups
find /backups -type d -mtime +30 -exec rm -rf {} + 2>/dev/null || true
EOF

# --- Script 2: log_cleanup.sh ---
cat > "$SCRIPTS_DIR/log_cleanup.sh" << 'EOF'
#!/usr/bin/env bash
set -euo pipefail
echo "[$(date)] Starting weekly log cleanup..."
# Delete compressed logs older than 90 days
find /var/log -name "*.gz" -mtime +90 -delete
# Delete empty log files
find /var/log -empty -name "*.log" -delete
echo "[$(date)] Log cleanup complete."
df -h / | awk 'NR==2{print "  Disk usage: " $5}'
EOF

# --- Script 3: healthcheck.sh ---
cat > "$SCRIPTS_DIR/healthcheck.sh" << 'EOF'
#!/usr/bin/env bash
ISSUES=()
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print int(100-$8)}')
MEM=$(free | awk '/Mem:/{printf "%d", $3/$2*100}')
DISK=$(df / | awk 'NR==2{print int($5)}')

(( CPU > 90 )) && ISSUES+=("HIGH_CPU:${CPU}%")
(( MEM > 90 )) && ISSUES+=("HIGH_MEM:${MEM}%")
(( DISK > 90 )) && ISSUES+=("HIGH_DISK:${DISK}%")

if [[ ${#ISSUES[@]} -gt 0 ]]; then
    echo "[$(date)] ALERT: ${ISSUES[*]}"
else
    echo "[$(date)] OK cpu=${CPU}% mem=${MEM}% disk=${DISK}%"
fi
EOF

# --- Script 4: monthly_report.sh ---
cat > "$SCRIPTS_DIR/monthly_report.sh" << 'EOF'
#!/usr/bin/env bash
echo "[$(date)] Monthly Usage Report"
echo "============================="
echo "Disk usage:"
df -h
echo "Memory:"
free -h
echo "Top 10 users by process count:"
ps aux | awk 'NR>1{print $1}' | sort | uniq -c | sort -rn | head -10
EOF

# --- Script 5: ssl_check.sh ---
cat > "$SCRIPTS_DIR/ssl_check.sh" << 'EOF'
#!/usr/bin/env bash
DOMAINS=("example.com" "api.example.com" "admin.example.com")
WARN_DAYS=30
for DOMAIN in "${DOMAINS[@]}"; do
    EXPIRY=$(echo | openssl s_client -servername "$DOMAIN" -connect "$DOMAIN:443" 2>/dev/null \
        | openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
    EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s 2>/dev/null || echo 0)
    NOW_EPOCH=$(date +%s)
    DAYS_LEFT=$(( (EXPIRY_EPOCH - NOW_EPOCH) / 86400 ))
    if (( DAYS_LEFT < WARN_DAYS )); then
        echo "[$(date)] WARNING: $DOMAIN SSL cert expires in ${DAYS_LEFT} days!"
    else
        echo "[$(date)] OK: $DOMAIN SSL cert valid for ${DAYS_LEFT} days"
    fi
done
EOF

# Make all scripts executable
chmod +x "$SCRIPTS_DIR/backup.sh" \
         "$SCRIPTS_DIR/log_cleanup.sh" \
         "$SCRIPTS_DIR/healthcheck.sh" \
         "$SCRIPTS_DIR/monthly_report.sh" \
         "$SCRIPTS_DIR/ssl_check.sh"

echo ""
echo "✅ Cron jobs installed successfully!"
echo ""
echo "Installed cron jobs:"
cat /etc/cron.d/sysadmin-jobs
echo ""
echo "Scripts in: $SCRIPTS_DIR"
echo "Logs in: $LOG_DIR"
```

### Key Takeaways

- Cron syntax: `MIN HOUR DOM MON DOW COMMAND` — five fields then the command
- `*` = any, `*/N` = every N, ranges with `-`, lists with `,`
- Special strings: `@daily`, `@weekly`, `@reboot`, etc.
- `/etc/cron.d/` for system-wide jobs — includes username field
- Always use absolute paths in cron jobs
- Redirect output (`>> logfile 2>&1`) to capture cron job output
- Use `flock` to prevent overlapping job runs

---

## Chapter 13 — Signals: SIGTERM, SIGKILL, SIGHUP, trap for Cleanup {#chapter-13}

### What Are Signals?

A signal is a notification sent to a process by the operating system or another process. It's the mechanism by which you "tell" a running program something happened: "please shut down," "someone pressed Ctrl+C," "reload your config."

Think of signals like a set of standardised text messages a process can receive: each signal has a fixed meaning, and programs can choose how to respond.

### Common Signals

| Signal | Number | Description | Default Action |
|--------|--------|-------------|----------------|
| `SIGHUP` | 1 | Hang up (terminal closed, or "reload config") | Terminate |
| `SIGINT` | 2 | Interrupt (user pressed Ctrl+C) | Terminate |
| `SIGQUIT` | 3 | Quit (Ctrl+\\) — also creates core dump | Terminate+Core |
| `SIGTERM` | 15 | Termination request — graceful shutdown | Terminate |
| `SIGKILL` | 9 | Kill immediately — cannot be caught or ignored | Terminate (forced) |
| `SIGUSR1` | 10 | User-defined signal 1 | Terminate |
| `SIGUSR2` | 12 | User-defined signal 2 | Terminate |
| `SIGCHLD` | 17 | Child process stopped or exited | Ignore |
| `SIGSTOP` | 19 | Stop (pause) process — cannot be caught | Pause |
| `SIGCONT` | 18 | Continue stopped process | Continue |

### Sending Signals — The kill Command

Despite its name, `kill` sends *any* signal — not just kill:

```bash
# Send SIGTERM (polite request to stop)
kill 12345          # By PID
kill -15 12345      # Explicit SIGTERM (same)
kill -SIGTERM 12345 # By name (same)
kill -TERM 12345    # Short form (same)

# Send SIGKILL (force kill — no cleanup possible)
kill -9 12345
kill -SIGKILL 12345

# Send SIGHUP (typically means "reload config")
kill -HUP 12345

# Kill all processes with a name
pkill nginx          # Send SIGTERM to all nginx processes
pkill -9 nginx       # Force kill
killall nginx        # Similar to pkill

# Send signal to all processes in a process group
kill -TERM -12345    # Negative PID = process group
```

### SIGTERM vs SIGKILL

This is critical knowledge:

**SIGTERM (15)** — The polite way to stop a process. The process receives it, can perform cleanup (close files, finish transactions, save state), then exits gracefully. Almost all production software handles SIGTERM properly.

**SIGKILL (9)** — The nuclear option. The kernel immediately destroys the process with no notice. No cleanup is possible. Use only when SIGTERM fails.

The correct shutdown sequence:
```bash
PID=12345

# 1. Ask nicely
kill -TERM "$PID"

# 2. Give it time to clean up
sleep 5

# 3. If still running, force it
if kill -0 "$PID" 2>/dev/null; then
    echo "Process still alive — sending SIGKILL"
    kill -KILL "$PID"
fi
```

### SIGHUP — Reload Without Restart

By convention, many daemon processes reload their configuration when they receive SIGHUP:

```bash
# Reload nginx config (graceful — no dropped connections)
kill -HUP $(cat /var/run/nginx.pid)
# Or:
nginx -s reload    # Which internally sends SIGHUP
systemctl reload nginx

# Reload sshd config
kill -HUP $(cat /var/run/sshd.pid)

# Reload logrotate on your application
kill -HUP $(cat /var/run/myapp.pid)
```

### trap — Catching Signals in Scripts

The `trap` command lets your script intercept signals and run custom code:

```bash
# Syntax: trap 'COMMAND' SIGNAL [SIGNAL...]
# Or:     trap FUNCTION_NAME SIGNAL

#!/usr/bin/env bash

# --- Cleanup function ---
cleanup() {
    echo "Caught signal! Cleaning up before exit..."
    # Kill any background jobs we started
    jobs -p | xargs -r kill 2>/dev/null
    # Remove temp files
    [[ -d "$TEMP_DIR" ]] && rm -rf "$TEMP_DIR"
    echo "Cleanup done."
    exit 0
}

# --- Register signal handlers ---
trap cleanup SIGTERM    # Handle graceful shutdown request
trap cleanup SIGINT     # Handle Ctrl+C
trap cleanup SIGHUP     # Handle hang-up

# Special pseudo-signals:
trap cleanup EXIT       # Run on ANY exit (normal or error)
trap 'echo "Error on line $LINENO"' ERR   # Run when a command fails

# --- Create a temp dir ---
TEMP_DIR=$(mktemp -d)
echo "Working in: $TEMP_DIR"

# --- Long-running work ---
echo "Starting long process. Press Ctrl+C to test signal handling."
for i in {1..60}; do
    echo "Iteration $i..."
    sleep 1
done

echo "Done normally."
```

### A Production-Grade Signal Handler

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: service_wrapper.sh
# Purpose: Run a service with proper signal handling
# =============================================================================

set -euo pipefail

# PID file for tracking
PID_FILE="/var/run/myservice.pid"
CHILD_PID=""

# --- Signal handlers ---
handle_sigterm() {
    echo "[$(date)] SIGTERM received — initiating graceful shutdown..."

    if [[ -n "$CHILD_PID" ]] && kill -0 "$CHILD_PID" 2>/dev/null; then
        echo "[$(date)] Stopping child process (PID: $CHILD_PID)..."
        kill -TERM "$CHILD_PID"

        # Wait for child to stop (up to 30 seconds)
        local WAIT=0
        while kill -0 "$CHILD_PID" 2>/dev/null && (( WAIT < 30 )); do
            sleep 1
            (( WAIT++ ))
        done

        if kill -0 "$CHILD_PID" 2>/dev/null; then
            echo "[$(date)] Child did not stop — forcing with SIGKILL"
            kill -KILL "$CHILD_PID"
        fi
    fi

    rm -f "$PID_FILE"
    echo "[$(date)] Shutdown complete."
    exit 0
}

handle_sighup() {
    echo "[$(date)] SIGHUP received — reloading configuration..."
    # Reload config logic here
    echo "[$(date)] Configuration reloaded."
}

handle_sigusr1() {
    echo "[$(date)] SIGUSR1 received — dumping status..."
    echo "Child PID: $CHILD_PID"
    if [[ -n "$CHILD_PID" ]]; then
        ps -p "$CHILD_PID" -o pid,ppid,pcpu,pmem,etime,cmd
    fi
}

# Register handlers
trap handle_sigterm SIGTERM SIGINT
trap handle_sighup  SIGHUP
trap handle_sigusr1 SIGUSR1

# Write PID file
echo $$ > "$PID_FILE"
echo "[$(date)] Service started. PID: $$"

# Start your actual service in the background
/usr/bin/my-application --config /etc/myapp/config.yaml &
CHILD_PID=$!
echo "[$(date)] Started child process. PID: $CHILD_PID"

# Wait for the child process (this is the main loop)
wait "$CHILD_PID"
EXIT_CODE=$?
echo "[$(date)] Child exited with code: $EXIT_CODE"
rm -f "$PID_FILE"
exit "$EXIT_CODE"
```

### Key Takeaways

- Signals are system-level notifications sent to processes
- `SIGTERM (15)` = graceful stop; `SIGKILL (9)` = forced stop; `SIGHUP (1)` = reload
- `kill -SIGNAL PID` sends a signal; `kill -0 PID` checks if process exists
- Always try SIGTERM first, wait, then SIGKILL if necessary
- `trap FUNCTION SIGNAL` intercepts signals in your script
- `trap cleanup EXIT` is the most important — runs on every exit

---

## Chapter 14 — Script Best Practices: ShellCheck, Modularity, Documentation, Idempotency {#chapter-14}

### The Gap Between "Works" and "Production-Grade"

A script that works on your machine, today, with your user account, is not the same as a script that works reliably in production at 3 AM after a failed deployment, run by a cron job as a different user, on a slightly different Linux distribution.

This final chapter bridges that gap.

### ShellCheck — Your Automated Code Reviewer

ShellCheck is a static analysis tool for shell scripts. It catches bugs, common mistakes, and portability issues before you run the script. Think of it as a strict, knowledgeable colleague who reviews every line of your code.

```bash
# Install ShellCheck
apt-get install shellcheck      # Debian/Ubuntu
yum install shellcheck          # RHEL/CentOS
brew install shellcheck         # macOS

# Run ShellCheck on a script
shellcheck myscript.sh

# Run on multiple files
shellcheck scripts/*.sh

# Output in different formats
shellcheck -f gcc myscript.sh       # GCC-style (great for CI integration)
shellcheck -f json myscript.sh      # JSON output
```

**Example ShellCheck output:**

```bash
# This script has several issues:
NAME=$1
if [ $NAME = "alice" ]; then
    echo "Welcome $NAME"
fi
```

```
In script.sh line 2:
NAME=$1
     ^-- SC2034: NAME appears unused. Verify use (or export if used externally).

In script.sh line 3:
if [ $NAME = "alice" ]; then
        ^-- SC2086: Double quote to prevent globbing and word splitting.
```

ShellCheck identifies:
- Unquoted variables (word splitting bugs)
- Using `[ ]` where `[[ ]]` is safer
- Useless use of cat
- Variables that shadow special builtins
- Incorrect use of `&&`/`||` with if statements
- And hundreds more patterns

**Run ShellCheck before every commit.** Make it a habit.

### The Chapter 14 Task — ShellCheck and Fix

**Task:** Run ShellCheck on all your scripts and fix every warning.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: shellcheck_all.sh
# Purpose: Run ShellCheck on all scripts and produce a report
# =============================================================================

set -euo pipefail

SCRIPT_DIR="${1:-.}"   # Default: current directory
PASS=0
FAIL=0
ERRORS=()

if ! command -v shellcheck &>/dev/null; then
    echo "ShellCheck not installed. Install with: apt-get install shellcheck"
    exit 1
fi

echo "Running ShellCheck on all .sh files in: $SCRIPT_DIR"
echo "====================================================="

while IFS= read -r -d '' SCRIPT; do
    echo -n "Checking: $SCRIPT ... "
    if shellcheck "$SCRIPT"; then
        echo "✅ PASS"
        (( PASS++ ))
    else
        echo "❌ FAIL"
        (( FAIL++ ))
        ERRORS+=("$SCRIPT")
    fi
done < <(find "$SCRIPT_DIR" -name "*.sh" -not -path "*/\.*" -print0)

echo ""
echo "====================================================="
echo "Results: $PASS passed, $FAIL failed"
if [[ ${#ERRORS[@]} -gt 0 ]]; then
    echo "Failed scripts:"
    for E in "${ERRORS[@]}"; do
        echo "  - $E"
    done
    exit 1
fi
echo "All scripts passed ShellCheck!"
```

### Modularity — Writing Scripts That Grow Gracefully

A modular script separates concerns into functions and files:

```
project/
├── bin/
│   ├── deploy.sh         # Main entry point
│   ├── backup.sh
│   └── provision-user.sh
└── lib/
    ├── logging.sh        # Shared logging functions
    ├── aws.sh            # AWS API helpers
    ├── validation.sh     # Input validation functions
    └── notifications.sh  # Email/Slack alerting
```

```bash
#!/usr/bin/env bash
# deploy.sh — clean, modular structure

set -euo pipefail

# Load shared libraries
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "${SCRIPT_DIR}/../lib/logging.sh"
source "${SCRIPT_DIR}/../lib/validation.sh"
source "${SCRIPT_DIR}/../lib/notifications.sh"

# Configuration
readonly APP_NAME="myapp"
readonly DEPLOY_DIR="/opt/${APP_NAME}"
readonly BACKUP_DIR="/backups/${APP_NAME}"
readonly LOG_FILE="/var/log/${APP_NAME}/deploy.log"

# Functions
backup_current() {
    local TIMESTAMP; TIMESTAMP="$(date +%Y%m%d_%H%M%S)"
    log_info "Creating backup: ${BACKUP_DIR}/${TIMESTAMP}"
    cp -r "$DEPLOY_DIR" "${BACKUP_DIR}/${TIMESTAMP}"
}

pull_code() {
    log_info "Pulling latest code..."
    git -C "$DEPLOY_DIR" pull origin main
}

run_tests() {
    log_info "Running tests..."
    cd "$DEPLOY_DIR" && make test
}

restart_service() {
    log_info "Restarting service..."
    systemctl restart "$APP_NAME"
    sleep 2
    systemctl is-active "$APP_NAME" || { log_error "Service failed to start!"; return 1; }
}

# Main
main() {
    log_info "Starting deployment of $APP_NAME"
    backup_current
    pull_code
    run_tests
    restart_service
    notify_slack "#deployments" "✅ $APP_NAME deployed successfully"
    log_info "Deployment complete."
}

main "$@"
```

### Documentation — Scripts as Living Documentation

Every professional script should have a header:

```bash
#!/usr/bin/env bash
# =============================================================================
# Script Name : backup.sh
# Description : Creates timestamped backups of specified directories,
#               verifies integrity, and logs results.
# Author      : DevOps Team <devops@example.com>
# Created     : 2025-01-15
# Modified    : 2025-06-01
# Version     : 2.3.1
#
# Usage       : backup.sh [OPTIONS] <source_dir>
#
# Options:
#   -d DIR     Destination directory (default: /backups)
#   -r NUM     Number of backups to retain (default: 30)
#   -v         Enable verbose output
#   -h         Show this help
#
# Examples:
#   backup.sh /var/www/html
#   backup.sh -d /mnt/nas/backups -r 7 /etc
#
# Dependencies: tar, gzip, sha256sum
# Cron:        0 2 * * * /usr/local/bin/backup.sh /var/www >> /var/log/backup.log 2>&1
# Exit codes:
#   0 — Success
#   1 — Invalid arguments
#   2 — Source directory not found
#   3 — Backup failed
#   4 — Integrity check failed
# =============================================================================
```

### Idempotency — Safe to Run Multiple Times

An idempotent script produces the same result whether run once or ten times. This is critical for automation:

```bash
#!/usr/bin/env bash
# IDEMPOTENT: Safe to run multiple times

# Create a user — idempotent version
create_user() {
    local USERNAME="$1"
    # If user already exists, don't error — just skip
    if id "$USERNAME" &>/dev/null; then
        echo "User '$USERNAME' already exists — skipping."
        return 0
    fi
    useradd -m -s /bin/bash "$USERNAME"
    echo "User '$USERNAME' created."
}

# Create a directory — idempotent (-p doesn't fail if it exists)
setup_directories() {
    mkdir -p /opt/myapp/{bin,conf,logs,data}
    chmod 755 /opt/myapp
}

# Add a line to a file — only if not already there
ensure_line() {
    local LINE="$1"
    local FILE="$2"
    grep -qF "$LINE" "$FILE" 2>/dev/null || echo "$LINE" >> "$FILE"
}

# Install a package — idempotent on Debian/Ubuntu
install_package() {
    local PKG="$1"
    if dpkg -l "$PKG" 2>/dev/null | grep -q "^ii"; then
        echo "$PKG is already installed."
    else
        apt-get install -y "$PKG"
    fi
}

# Example usage
create_user "deploybot"
setup_directories
ensure_line "export APP_HOME=/opt/myapp" /etc/environment
install_package "nginx"
```

### Performance and Safety Checklist

Before considering a script "production-ready," verify:

```bash
#!/usr/bin/env bash
# Production Script Checklist

# ✅ 1. Has shebang
# ✅ 2. Has strict mode
set -euo pipefail

# ✅ 3. Has descriptive header comment

# ✅ 4. Uses readonly for constants that shouldn't change
readonly CONFIG_FILE="/etc/myapp/config.conf"
readonly VERSION="1.0.0"

# ✅ 5. Validates all inputs before proceeding
[[ $# -lt 1 ]] && { echo "Usage: $0 <arg>" >&2; exit 1; }
[[ ! -f "$CONFIG_FILE" ]] && { echo "Config not found: $CONFIG_FILE" >&2; exit 2; }

# ✅ 6. Has cleanup/trap
trap 'rm -f /tmp/myapp_$$.tmp' EXIT

# ✅ 7. All functions use local variables

# ✅ 8. All variables are quoted: "$VAR" not $VAR

# ✅ 9. Logs to a file (not just stdout)
LOG_FILE="/var/log/myapp/$(basename "$0").log"
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"; }

# ✅ 10. Is idempotent (safe to run multiple times)

# ✅ 11. Passes ShellCheck with no warnings

# ✅ 12. Has been tested with edge cases (empty files, missing deps, wrong permissions)
```

### Final Chapter 14 Task — Menu-Driven Sysadmin Tool

**Task:** Build a menu-driven system admin tool with 10+ options.

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: sysadmin_menu.sh
# Purpose: Interactive system administration menu tool
# Usage: ./sysadmin_menu.sh
# =============================================================================

set -euo pipefail

# --- Configuration ---
readonly SCRIPT_VERSION="1.0.0"
readonly LOG_FILE="/var/log/sysadmin_menu.log"

# --- Logging ---
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [${2:-INFO}] $1" | tee -a "$LOG_FILE" >&2
}

# --- UI Helpers ---
clear_screen() { clear; }
pause() { read -r -p "Press Enter to continue..."; }

print_header() {
    echo "╔══════════════════════════════════════════════╗"
    echo "║     System Administration Tool v${SCRIPT_VERSION}      ║"
    echo "║     Host: $(hostname | cut -c1-20 | printf '%-20s' "$(cat -)")  ║"
    echo "║     Date: $(date '+%Y-%m-%d %H:%M')              ║"
    echo "╚══════════════════════════════════════════════╝"
    echo ""
}

print_menu() {
    echo "  System Information:"
    echo "    1)  System Overview (CPU, RAM, Disk)"
    echo "    2)  Running Processes (top 10 by CPU)"
    echo "    3)  Network Interfaces and Connections"
    echo ""
    echo "  User Management:"
    echo "    4)  List All Users"
    echo "    5)  Create New User"
    echo "    6)  Delete User"
    echo "    7)  Change User Password"
    echo ""
    echo "  Service Management:"
    echo "    8)  List Running Services"
    echo "    9)  Start/Stop/Restart a Service"
    echo "    10) Check Service Status"
    echo ""
    echo "  File & Log Operations:"
    echo "    11) Find Large Files (>100MB)"
    echo "    12) View Last 50 Lines of a Log File"
    echo "    13) Search String in Files"
    echo ""
    echo "  Security:"
    echo "    14) Show Failed Login Attempts"
    echo "    15) Show Open Ports"
    echo ""
    echo "    0)  Exit"
    echo ""
}

# --- Menu Actions ---

action_system_overview() {
    clear_screen
    echo "=== System Overview ==="
    echo "Hostname: $(hostname)"
    echo "OS:       $(grep PRETTY_NAME /etc/os-release | cut -d= -f2 | tr -d '"')"
    echo "Kernel:   $(uname -r)"
    echo "Uptime:   $(uptime -p)"
    echo ""
    echo "--- CPU ---"
    top -bn1 | grep "Cpu(s)" | sed 's/  */  /g'
    echo ""
    echo "--- Memory ---"
    free -h
    echo ""
    echo "--- Disk ---"
    df -h | grep -v tmpfs
    pause
}

action_top_processes() {
    clear_screen
    echo "=== Top 10 Processes by CPU ==="
    ps aux --sort=-%cpu | head -11 | awk '{printf "%-10s %-8s %-8s %s\n", $1, $2, $3, $11}'
    echo ""
    echo "=== Top 10 Processes by Memory ==="
    ps aux --sort=-%mem | head -11 | awk '{printf "%-10s %-8s %-8s %s\n", $1, $2, $4, $11}'
    pause
}

action_network() {
    clear_screen
    echo "=== Network Interfaces ==="
    ip -brief addr show
    echo ""
    echo "=== Active Connections ==="
    ss -tuln | head -30
    pause
}

action_list_users() {
    clear_screen
    echo "=== System Users (shell=/bin/bash or /bin/sh) ==="
    awk -F: '$7 ~ /bash|sh$/ {printf "%-20s UID:%-6s Home: %s\n", $1, $3, $6}' /etc/passwd
    pause
}

action_create_user() {
    clear_screen
    echo "=== Create New User ==="
    read -r -p "Username: " NEW_USER
    if id "$NEW_USER" &>/dev/null; then
        echo "ERROR: User '$NEW_USER' already exists."
    else
        useradd -m -s /bin/bash "$NEW_USER"
        passwd "$NEW_USER"
        echo "User '$NEW_USER' created successfully."
        log "Created user: $NEW_USER"
    fi
    pause
}

action_delete_user() {
    clear_screen
    echo "=== Delete User ==="
    read -r -p "Username to delete: " DEL_USER
    if ! id "$DEL_USER" &>/dev/null; then
        echo "ERROR: User '$DEL_USER' not found."
    else
        read -r -p "Delete home directory too? (y/N): " DEL_HOME
        if [[ "$DEL_HOME" =~ ^[yY] ]]; then
            userdel -r "$DEL_USER"
        else
            userdel "$DEL_USER"
        fi
        echo "User '$DEL_USER' deleted."
        log "Deleted user: $DEL_USER"
    fi
    pause
}

action_change_password() {
    clear_screen
    echo "=== Change User Password ==="
    read -r -p "Username: " PASS_USER
    if id "$PASS_USER" &>/dev/null; then
        passwd "$PASS_USER"
        log "Password changed for: $PASS_USER"
    else
        echo "ERROR: User not found."
    fi
    pause
}

action_list_services() {
    clear_screen
    echo "=== Running Services ==="
    systemctl list-units --type=service --state=running --no-legend 2>/dev/null \
        | awk '{printf "%-40s %s\n", $1, $4}' | head -30
    pause
}

action_manage_service() {
    clear_screen
    echo "=== Manage Service ==="
    read -r -p "Service name: " SVC_NAME
    echo "Actions: start, stop, restart, reload"
    read -r -p "Action: " SVC_ACTION
    if systemctl "$SVC_ACTION" "$SVC_NAME"; then
        echo "✅ Service $SVC_NAME ${SVC_ACTION}ed."
        log "Service action: $SVC_ACTION on $SVC_NAME"
    else
        echo "❌ Action failed."
    fi
    pause
}

action_service_status() {
    clear_screen
    echo "=== Service Status ==="
    read -r -p "Service name: " STATUS_SVC
    systemctl status "$STATUS_SVC" --no-pager || true
    pause
}

action_large_files() {
    clear_screen
    echo "=== Files Larger Than 100MB ==="
    find / -type f -size +100M -printf '%s %p\n' 2>/dev/null \
        | sort -rn \
        | awk '{printf "%.1f MB  %s\n", $1/1048576, $2}' \
        | head -20
    pause
}

action_view_log() {
    clear_screen
    echo "=== View Log File ==="
    echo "Common logs: /var/log/syslog, /var/log/auth.log, /var/log/nginx/error.log"
    read -r -p "Log file path: " LOG_PATH
    if [[ -f "$LOG_PATH" && -r "$LOG_PATH" ]]; then
        tail -50 "$LOG_PATH"
    else
        echo "ERROR: Cannot read file: $LOG_PATH"
    fi
    pause
}

action_search_files() {
    clear_screen
    echo "=== Search String in Files ==="
    read -r -p "Search string: " SEARCH_STR
    read -r -p "Directory to search (default: /var/log): " SEARCH_DIR
    SEARCH_DIR="${SEARCH_DIR:-/var/log}"
    grep -r --include="*.log" -l "$SEARCH_STR" "$SEARCH_DIR" 2>/dev/null | head -20
    pause
}

action_failed_logins() {
    clear_screen
    echo "=== Failed Login Attempts (last 20) ==="
    grep "Failed password" /var/log/auth.log 2>/dev/null | tail -20 || \
        journalctl -u sshd | grep "Failed" | tail -20 || \
        echo "No auth log found or no failures recorded."
    echo ""
    echo "=== Top Source IPs ==="
    grep "Failed password" /var/log/auth.log 2>/dev/null \
        | awk '{print $11}' | sort | uniq -c | sort -rn | head -10 || true
    pause
}

action_open_ports() {
    clear_screen
    echo "=== Open Listening Ports ==="
    ss -tlnp | awk 'NR==1 || /LISTEN/'
    echo ""
    echo "=== UDP Ports ==="
    ss -ulnp | awk 'NR==1 || $1=="UNCONN"' | head -10
    pause
}

# --- Main Loop ---
main() {
    mkdir -p "$(dirname "$LOG_FILE")"
    log "Sysadmin menu started by $(whoami)"

    while true; do
        clear_screen
        print_header
        print_menu
        read -r -p "Select option [0-15]: " CHOICE

        case "$CHOICE" in
            1)  action_system_overview ;;
            2)  action_top_processes ;;
            3)  action_network ;;
            4)  action_list_users ;;
            5)  action_create_user ;;
            6)  action_delete_user ;;
            7)  action_change_password ;;
            8)  action_list_services ;;
            9)  action_manage_service ;;
            10) action_service_status ;;
            11) action_large_files ;;
            12) action_view_log ;;
            13) action_search_files ;;
            14) action_failed_logins ;;
            15) action_open_ports ;;
            0)
                log "Sysadmin menu exited"
                echo "Goodbye!"
                exit 0
                ;;
            *)
                echo "Invalid option: $CHOICE"
                sleep 1
                ;;
        esac
    done
}

main "$@"
```

### Key Takeaways

- Run `shellcheck script.sh` on every script before using it in production
- Modular scripts: shared functions in `lib/` files, sourced by main scripts
- Every script needs a header: purpose, usage, options, exit codes
- Idempotent scripts produce the same result when run multiple times — essential for automation
- `readonly` for constants; `local` in every function; `"$VAR"` always
- The production checklist: strict mode, validation, cleanup trap, logging, idempotency

---

## Final Chapter — How It All Connects: A Real-World DevOps Workflow {#final-chapter}

### The Complete Picture

You've now learned 14 topics. Let's step back and see how they form a single, coherent picture of professional Bash automation.

Imagine you've just joined a DevOps team. On day one, they hand you a project: *"We need an automated deployment pipeline for our web application."*

Here's how every chapter connects:

---

**Chapter 1 (Shebang, chmod, PATH):**
You create `deploy.sh`, make it executable with `chmod +x`, and place it in `/usr/local/bin` so the CI system can find it anywhere.

**Chapter 2 (Variables):**
At the top of the script, you define configuration variables: `APP_NAME`, `DEPLOY_DIR`, `REMOTE_SERVER`, `SLACK_WEBHOOK`. This makes the script easy to configure without editing its logic.

**Chapter 3 (Special Variables):**
You use `$#` to validate that the right number of arguments were passed. `$0` appears in usage messages. `$?` checks whether each step succeeded.

**Chapter 4 (User Input / getopts):**
The script accepts `--env staging` and `--branch main` flags, making it usable from both the terminal (by humans) and CI/CD pipelines (by machines).

**Chapter 5 (Conditionals):**
You check: does the environment exist? Is the branch valid? Is the deploy directory writable? If any check fails, you exit with a descriptive error before touching production.

**Chapter 6 (Loops):**
You loop over a list of servers to pull the new code to each one. You loop while waiting for the health check endpoint to return 200 OK.

**Chapter 7 (Functions):**
`backup_current()`, `pull_code()`, `run_tests()`, `deploy()`, `health_check()`, `rollback()` — each step is a well-named function. The main function calls them in order and handles failures.

**Chapter 8 (Error Handling):**
`set -euo pipefail` at the top. `trap cleanup EXIT` ensures temp files are removed. `trap 'rollback_handler $LINENO' ERR` rolls back automatically if any step fails.

**Chapter 9 (Text Processing):**
`grep`, `sed`, and `awk` parse the deployment log to find errors. The health check response is parsed with `grep` and `awk` to extract the status field.

**Chapter 10 (Regex):**
The branch name is validated against a regex: `^(main|staging|release/[0-9]+\.[0-9]+)$`. Version numbers from the build output are extracted with `grep -E`.

**Chapter 11 (File Operations, Heredocs):**
New nginx config files are written using heredocs with variable expansion. The deployment log is built using `tee` — visible in the terminal and saved to disk simultaneously.

**Chapter 12 (Cron):**
The deployment script is called nightly for staging updates. A separate health check script runs every 5 minutes. Log cleanup runs weekly. All scheduled in `/etc/cron.d/myapp`.

**Chapter 13 (Signals):**
The deployment script handles `SIGTERM` — if the CI system cancels the job, the signal handler rolls back any partial deployment before exiting.

**Chapter 14 (Best Practices):**
ShellCheck passes with zero warnings. Every function has `local` variables. The script is idempotent — running it twice produces the same result. The header documents purpose, usage, and exit codes.

---

### A Condensed Production Deployment Script

```bash
#!/usr/bin/env bash
# =============================================================================
# Script: deploy.sh
# Description: Deploy application to staging or production
# Usage: deploy.sh -e <environment> -b <branch> [-v] [-d]
# Exit codes: 0=success, 1=bad args, 2=tests failed, 3=deploy failed, 4=health failed
# =============================================================================

set -euo pipefail

# --- Source libraries ---
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "${SCRIPT_DIR}/../lib/logging.sh"
source "${SCRIPT_DIR}/../lib/notifications.sh"

# --- Constants ---
readonly DEPLOY_USER="deploy"
readonly BASE_DIR="/opt/myapp"
readonly BACKUP_DIR="/opt/myapp_backups"
readonly HEALTH_URL="http://localhost:8080/health"
readonly SLACK_CHANNEL="#deployments"

# --- Defaults ---
ENVIRONMENT=""
BRANCH="main"
VERBOSE=false
DRY_RUN=false

# --- Arg parsing ---
usage() { echo "Usage: $0 -e <env> -b <branch> [-v] [-d]"; exit 1; }
while getopts "e:b:vdh" OPT; do
    case "$OPT" in
        e) ENVIRONMENT="$OPTARG" ;;
        b) BRANCH="$OPTARG" ;;
        v) VERBOSE=true ;;
        d) DRY_RUN=true ;;
        *) usage ;;
    esac
done

[[ -z "$ENVIRONMENT" ]] && { echo "ERROR: -e required" >&2; usage; }
[[ ! "$ENVIRONMENT" =~ ^(staging|production)$ ]] && { echo "Invalid env" >&2; exit 1; }

# --- State tracking for rollback ---
BACKUP_PATH=""
ROLLBACK_NEEDED=false

# --- Cleanup and signal handling ---
cleanup() {
    local CODE=$?
    if [[ "$ROLLBACK_NEEDED" == true && -n "$BACKUP_PATH" ]]; then
        log_warn "Rolling back to: $BACKUP_PATH"
        [[ "$DRY_RUN" == false ]] && cp -r "$BACKUP_PATH" "$BASE_DIR"
        notify_slack "$SLACK_CHANNEL" "⚠️ Rollback executed for $ENVIRONMENT"
    fi
    [[ $CODE -ne 0 ]] && notify_slack "$SLACK_CHANNEL" "❌ Deploy FAILED for $ENVIRONMENT (code $CODE)"
}
trap cleanup EXIT
trap 'ROLLBACK_NEEDED=true' ERR

# --- Functions ---
backup_current() {
    BACKUP_PATH="${BACKUP_DIR}/$(date +%Y%m%d_%H%M%S)"
    log_info "Backup: $BASE_DIR -> $BACKUP_PATH"
    [[ "$DRY_RUN" == false ]] && cp -r "$BASE_DIR" "$BACKUP_PATH"
}

pull_code() {
    log_info "Pulling branch: $BRANCH"
    [[ "$DRY_RUN" == false ]] && git -C "$BASE_DIR" pull origin "$BRANCH"
}

run_tests() {
    log_info "Running tests..."
    if [[ "$DRY_RUN" == false ]]; then
        cd "$BASE_DIR" && make test
    fi
}

deploy() {
    log_info "Deploying to $ENVIRONMENT..."
    [[ "$DRY_RUN" == false ]] && systemctl restart myapp
}

wait_for_health() {
    log_info "Waiting for health check..."
    local ATTEMPTS=0
    until curl -sf "$HEALTH_URL" | grep -q '"status":"ok"'; do
        (( ATTEMPTS++ ))
        if (( ATTEMPTS >= 12 )); then
            log_error "Health check failed after 60s"
            return 1
        fi
        log_info "Not ready (attempt $ATTEMPTS/12)..."
        sleep 5
    done
    log_info "Health check passed!"
}

# --- Main ---
main() {
    log_info "===== Deploy started: $ENVIRONMENT/$BRANCH ====="
    [[ "$DRY_RUN" == true ]] && log_warn "DRY RUN MODE — no changes will be made"

    backup_current
    pull_code
    run_tests
    deploy
    wait_for_health

    ROLLBACK_NEEDED=false   # Everything succeeded — no rollback needed
    log_info "===== Deploy successful ====="
    notify_slack "$SLACK_CHANNEL" "✅ $ENVIRONMENT deployed successfully (branch: $BRANCH)"
}

main "$@"
```

---

### What You've Built

After completing every task in this book, you will have:

| Script | Skills Demonstrated |
|--------|-------------------|
| `file_inspector.sh` | Conditionals, file tests, exit codes, error messages |
| `csv_processor.sh` | Arrays, loops, string ops, associative arrays |
| `deploy.sh` (getopts) | getopts, argument parsing, validation |
| `retry_wrapper.sh` | Functions, loops, exponential backoff, error handling |
| `log_parser.sh` | grep, sed, awk, pipelines, text processing |
| `health_report.sh` | Heredocs, process substitution, HTML output |
| `setup_cron_jobs.sh` | Cron, scheduling, crontab management |
| `service_wrapper.sh` | Signals, trap, SIGTERM, SIGKILL, SIGHUP |
| `shellcheck_all.sh` | Best practices, ShellCheck, modularity |
| `sysadmin_menu.sh` | Functions, case statements, menus, all combined |
| `backup.sh` | Timestamps, tar, integrity checking, logging |
| `csv_processor.sh` | Data validation, transformation, output |

---

### Where to Go From Here

**Deepen your Bash:**
- Read `man bash` — the official reference for everything
- Advanced Bash-Scripting Guide (tldp.org)
- Google's Shell Style Guide

**Related skills to learn next:**
- **Python for sysops** — when Bash starts feeling limiting
- **Ansible** — configuration management built on top of Bash concepts
- **Terraform** — infrastructure as code
- **Git hooks** — Bash scripts that run automatically on git events

**Tools that extend Bash:**
- `fzf` — fuzzy finder for interactive menus
- `jq` — JSON processor (the awk for JSON)
- `yq` — YAML processor
- `parallel` — GNU parallel for running many jobs simultaneously

---

### Final Words

The gap between someone who "knows how to use a terminal" and someone who "can automate a production environment" is exactly the content of this book. You now know how to write scripts that are safe, readable, tested, and reliable.

Every professional DevOps engineer has a personal library of scripts they've built and refined over years. Start yours now. Write a script for everything repetitive you do. Review it with ShellCheck. Improve it as you learn more.

The terminal is not a tool you use occasionally — for a DevOps engineer, it is the primary interface with the entire digital infrastructure they are responsible for. Master it, and you master the job.

Good luck. Keep scripting.

---

*Bash Scripting & Automation — Cloud & DevOps Engineering Curriculum*
*Version 1.0 | All code tested on Ubuntu 22.04 LTS with Bash 5.1*