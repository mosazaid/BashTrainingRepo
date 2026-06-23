# Shell & Bash Scripting: Zero to Advanced
### A Complete Guide for Android & Java Spring Boot Developers on Mac & Windows

---

## Table of Contents

1. [What Is a Shell?](#1-what-is-a-shell)
2. [Environment Setup](#2-environment-setup)
   - Mac Terminal & Zsh/Bash
   - Windows: WSL2, Git Bash, PowerShell
3. [Core Concepts](#3-core-concepts)
4. [Basic Commands](#4-basic-commands)
5. [Variables & Data Types](#5-variables--data-types)
6. [Input / Output & Redirection](#6-input--output--redirection)
7. [Control Flow](#7-control-flow)
8. [Functions](#8-functions)
9. [String Manipulation](#9-string-manipulation)
10. [Arrays & Associative Arrays](#10-arrays--associative-arrays)
11. [File & Directory Operations](#11-file--directory-operations)
12. [Process Management](#12-process-management)
13. [Regular Expressions & Text Processing](#13-regular-expressions--text-processing)
14. [Error Handling & Defensive Scripting](#14-error-handling--defensive-scripting)
15. [Debugging Techniques](#15-debugging-techniques)
16. [Advanced Patterns](#16-advanced-patterns)
17. [Android Development Recipes](#17-android-development-recipes)
18. [Java Spring Boot Recipes](#18-java-spring-boot-recipes)
19. [Cross-Platform Scripting (Mac vs Windows)](#19-cross-platform-scripting-mac-vs-windows)
20. [Security Best Practices](#20-security-best-practices)
21. [Performance & Optimization](#21-performance--optimization)
22. [Real-World Project Scripts](#22-real-world-project-scripts)
23. [Quick Reference Cheatsheet](#23-quick-reference-cheatsheet)

---

## 1. What Is a Shell?

A **shell** is a command-line interpreter that accepts human commands and passes them to the operating system kernel for execution. Think of it as the translator between you and the OS.

```
You → Shell → Kernel → Hardware
```

### Shell Types You Will Encounter

| Shell | Platform | Notes |
|-------|----------|-------|
| **bash** | Linux, Mac (pre-Catalina default), WSL | Most common for scripting |
| **zsh** | Mac (Catalina+ default) | Backward-compatible with bash, better interactive UX |
| **sh** | POSIX standard | Minimal; write portable scripts targeting this |
| **fish** | Linux/Mac | Modern UX, NOT bash-compatible |
| **PowerShell** | Windows (also Linux/Mac) | Microsoft scripting; different syntax entirely |
| **cmd.exe** | Windows | Legacy; avoid for new scripts |

### Why Bash Matters for Android & Spring Boot Developers

- **Android**: Gradle build scripts, ADB automation, emulator management, CI/CD pipelines (GitHub Actions, Bitrise, CircleCI) all rely on bash.
- **Spring Boot**: Maven/Gradle wrappers, Docker builds, Kubernetes deployments, health-check scripts, log analysis — all bash.
- **Mac**: macOS is Unix-based; knowing bash makes you productive immediately.
- **Windows via WSL2**: Run the exact same Linux scripts locally that run on your CI servers.

---

## 2. Environment Setup

### macOS

macOS ships with **zsh** as the default since Catalina (10.15). Bash is still present but at an older version (3.2) due to licensing.

```bash
# Check your current shell
echo $SHELL
# /bin/zsh  or  /bin/bash

# Check bash version
bash --version
# GNU bash, version 3.2.57 (old Apple version)

# Install modern bash via Homebrew
brew install bash
# Now /opt/homebrew/bin/bash is bash 5.x

# Verify
/opt/homebrew/bin/bash --version

# Make it your default shell (optional)
sudo sh -c 'echo /opt/homebrew/bin/bash >> /etc/shells'
chsh -s /opt/homebrew/bin/bash
```

**Zsh vs Bash for scripting:** For script files (`.sh`), always specify the interpreter explicitly via shebang. For interactive use, zsh is fine. This guide focuses on bash scripting (compatible with zsh).

#### Homebrew (essential for Mac developers)

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install essential tools
brew install git curl wget jq yq tree htop
brew install openjdk@21          # For Spring Boot
brew install android-commandlinetools  # ADB, sdkmanager
```

#### Shell Configuration Files on Mac

| File | When it runs | Use for |
|------|-------------|---------|
| `~/.zshrc` | Every interactive zsh session | Aliases, functions, prompt |
| `~/.zprofile` | Login zsh sessions | PATH, env vars |
| `~/.bash_profile` | Login bash sessions | PATH, env vars |
| `~/.bashrc` | Interactive non-login bash | Aliases, functions |
| `~/.profile` | POSIX login shells | Portable env setup |

```bash
# Edit your shell config (zsh)
nano ~/.zshrc
# or with VS Code
code ~/.zshrc

# After editing, reload without restarting terminal
source ~/.zshrc
```

---

### Windows

#### Option 1: WSL2 (Recommended — Full Linux)

WSL2 runs a real Linux kernel inside Windows. Your scripts run identically to a Linux CI server.

```powershell
# In PowerShell (Administrator)
wsl --install
# Installs Ubuntu by default

# After restart, set WSL2 as default
wsl --set-default-version 2

# List installed distros
wsl --list --verbose
```

Once inside WSL2 Ubuntu:
```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install development tools
sudo apt install -y git curl wget jq build-essential

# Install Java 21
sudo apt install -y openjdk-21-jdk

# Verify
java -version
```

**Accessing Windows files from WSL2:**
```bash
# Your Windows C: drive is at:
ls /mnt/c/Users/YourName/

# Your WSL home is separate:
ls ~   # /home/yourusername/
```

**Accessing WSL files from Windows Explorer:**
Type `\\wsl$\Ubuntu\home\yourusername` in Explorer.

#### Option 2: Git Bash (Lightweight, No Linux Kernel)

Git Bash ships with Git for Windows and provides a bash environment running on top of MSYS2.

```bash
# Install: https://git-scm.com/download/win
# Git Bash config file:
# C:\Users\YourName\.bashrc  (or ~/.bashrc inside Git Bash)

# Limitation: No apt, no systemd, paths work differently
# Good for: Git operations, basic scripting, running .sh scripts
```

#### Option 3: PowerShell (Native Windows)

For Windows-native scripts. Covered briefly in cross-platform section.

#### Shebang Lines for Cross-Platform

```bash
#!/usr/bin/env bash     # Portable — finds bash in PATH (recommended)
#!/bin/bash             # Absolute path (fails on some systems)
#!/bin/sh               # POSIX sh (most portable, fewest features)
```

---

## 3. Core Concepts

### The Shell Execution Model

When you run a command, the shell:
1. Parses the command line (tokenisation, variable expansion, globbing)
2. Looks up the command in PATH
3. Forks a child process
4. Executes the command
5. Waits for it to complete
6. Returns the exit code

```bash
# Every command returns an exit code
# 0 = success, non-zero = failure
ls /tmp
echo $?   # prints 0

ls /nonexistent
echo $?   # prints 2 (or 1 — non-zero = failure)
```

### PATH — How Commands Are Found

```bash
# See your PATH
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

# When you type "java", the shell searches these directories in order
# First match wins

# Add a directory to PATH (in ~/.zshrc or ~/.bashrc)
export PATH="$HOME/.local/bin:$PATH"

# Find where a command lives
which java        # /usr/bin/java
which -a java     # All instances in PATH
type java         # More info: alias, function, or file
command -v java   # POSIX-portable version of `which`
```

### Quoting Rules — Critical Concept

```bash
# Double quotes: variables and command substitution expand INSIDE
name="World"
echo "Hello $name"      # Hello World
echo "Today: $(date)"   # Today: Mon Jun 23 11:00:00 UTC 2026

# Single quotes: EVERYTHING is literal — no expansion
echo 'Hello $name'      # Hello $name
echo 'Today: $(date)'   # Today: $(date)

# No quotes: word splitting + glob expansion apply
files=*.txt
echo $files    # expands to all .txt files
echo "$files"  # literal *.txt (because it's a variable, not a glob now)

# Backslash: escape a single character
echo "She said \"hello\""
echo "Price: \$5.00"

# RULE OF THUMB: Always double-quote variable expansions: "$var" not $var
```

### Command Substitution

```bash
# Capture command output into a variable
current_date=$(date +%Y-%m-%d)
echo "Today is $current_date"

# Backtick syntax (older, avoid in new scripts)
current_date=`date +%Y-%m-%d`

# Nested substitution — only $(…) supports this cleanly
user_home=$(dirname $(getent passwd $USER | cut -d: -f6))

# Capture both stdout and stderr
output=$(command 2>&1)
```

---

## 4. Basic Commands

### Navigation

```bash
pwd               # Print Working Directory — where you are now
ls                # List files
ls -la            # Long format, include hidden files
ls -lh            # Human-readable file sizes
ls -lt            # Sort by modification time (newest first)
ls -R             # Recursive listing

cd /path/to/dir   # Change directory (absolute path)
cd relative/path  # Change directory (relative to current)
cd ~              # Go to home directory
cd -              # Go to previous directory
cd ..             # Go up one level
cd ../..          # Go up two levels
```

### File Operations

```bash
# Create
touch file.txt             # Create empty file or update timestamp
mkdir mydir                # Create directory
mkdir -p a/b/c             # Create nested directories (no error if exists)

# Copy
cp source.txt dest.txt     # Copy file
cp -r sourcedir/ destdir/  # Copy directory recursively
cp -p file.txt backup.txt  # Preserve permissions and timestamps

# Move / Rename
mv old.txt new.txt         # Rename file
mv file.txt /tmp/          # Move file to /tmp/
mv dir1/ dir2/             # Rename or move directory

# Delete
rm file.txt                # Delete file
rm -r directory/           # Delete directory recursively
rm -rf directory/          # Force delete (no confirmation) — use with care!
rmdir emptydir/            # Delete empty directory only

# View files
cat file.txt               # Print entire file
less file.txt              # Scroll through file (q to quit)
head file.txt              # First 10 lines
head -n 20 file.txt        # First 20 lines
tail file.txt              # Last 10 lines
tail -n 50 file.txt        # Last 50 lines
tail -f app.log            # Follow log file in real-time (Ctrl+C to stop)
```

### Searching

```bash
# Find files
find . -name "*.java"                    # Find .java files from current dir
find . -name "*.kt" -type f             # Only files, not directories
find . -type d -name "build"            # Find build directories
find . -mtime -1                        # Modified in last 24 hours
find . -size +10M                       # Files larger than 10 MB
find . -name "*.log" -delete            # Find and delete .log files

# Search inside files
grep "pattern" file.txt                 # Search in one file
grep -r "pattern" .                     # Search recursively
grep -ri "pattern" .                    # Case-insensitive recursive
grep -n "TODO" *.java                   # Show line numbers
grep -l "import retrofit" .             # Only print filenames
grep -v "DEBUG" app.log                 # Lines NOT matching pattern
grep -E "error|warning|fatal" app.log  # Extended regex (multiple patterns)

# Modern alternative: ripgrep (install: brew install ripgrep)
rg "pattern" .              # Faster, respects .gitignore
rg -t java "TODO"           # Only Java files
rg -i "error" --glob "*.log"
```

### Permissions

```bash
ls -la   # Shows: drwxr-xr-x  2 user group  64 Jun 23 12:00 dirname
#                  ^ type + permissions ^ links ^ owner ^ group ^ size ^ date ^ name

# Permission breakdown: rwxrwxrwx
#   First rwx = owner permissions
#   Second rwx = group permissions
#   Third rwx = others permissions
#   r = read (4), w = write (2), x = execute (1)

chmod +x script.sh             # Make executable (anyone)
chmod 755 script.sh            # rwxr-xr-x (owner full, others read+exec)
chmod 644 file.txt             # rw-r--r-- (owner read+write, others read)
chmod 600 ~/.ssh/id_rsa        # rw------- (only owner can read/write)
chmod -R 755 directory/        # Recursive

chown username file.txt        # Change owner
chown username:groupname file  # Change owner and group
sudo chown -R www-data /var/www/html
```

### Disk & System Information

```bash
df -h              # Disk free (human-readable)
df -h .            # Disk space for current filesystem
du -sh *           # Disk usage of each item in current dir
du -sh /path/      # Disk usage of specific path

top                # Process monitor (q to quit)
htop               # Better process monitor (brew install htop)
ps aux             # All running processes
ps aux | grep java # Find Java processes

free -h            # Memory usage (Linux/WSL; not on Mac)
vm_stat            # Memory statistics on Mac

uname -a           # OS information
uname -m           # Architecture (x86_64, arm64)
hostname           # Machine name
whoami             # Current user
id                 # User ID and group membership
```

---

## 5. Variables & Data Types

### Declaring and Using Variables

```bash
# Assignment: NO spaces around = sign
name="Alice"
age=30
is_dev=true

# Reading a variable: prefix with $
echo $name
echo "My name is $name and I am $age years old"

# Best practice: use braces for clarity
echo "${name}Script"    # AliceScript (not $nameScript which is undefined)
echo "User: ${name}"

# Readonly variables (constants)
readonly MAX_RETRIES=3
readonly API_URL="https://api.example.com"
# MAX_RETRIES=5  # This would cause an error

# Unset a variable
unset name
echo $name   # Prints empty string
```

### Variable Scoping

```bash
# Variables are global by default within a script
global_var="I am global"

my_function() {
    # local keyword restricts scope to the function
    local local_var="I am local"
    echo "$global_var"     # Works
    echo "$local_var"      # Works
}

my_function
echo "$global_var"   # Works
echo "$local_var"    # Empty — local_var doesn't exist here

# Environment variables: exported to child processes
export MY_VAR="visible to child processes"
# Or combine declaration and export:
export DB_HOST="localhost"
export DB_PORT=5432

# Check if a variable is set
if [ -z "$MY_VAR" ]; then
    echo "MY_VAR is not set or empty"
fi

if [ -n "$MY_VAR" ]; then
    echo "MY_VAR is set to: $MY_VAR"
fi
```

### Special Variables

```bash
$0          # Name of the script
$1, $2, …   # Positional arguments (script parameters)
$#          # Number of arguments passed
$@          # All arguments as separate words
$*          # All arguments as one string
$$          # Current process ID (PID)
$!          # PID of last background command
$?          # Exit code of last command
$-          # Current option flags
$_          # Last argument of previous command

# Example usage
echo "Script name: $0"
echo "First arg:   $1"
echo "All args:    $@"
echo "Arg count:   $#"
```

### String Variables

```bash
name="Spring Boot"

# Length
echo ${#name}          # 11

# Substring: ${var:start:length}
echo ${name:0:6}       # Spring
echo ${name:7}         # Boot (from index 7 to end)
echo ${name: -4}       # Boot (last 4 chars)

# Replace first occurrence
echo ${name/Spring/Quarkus}    # Quarkus Boot

# Replace all occurrences
sentence="Go is good. Go is fast."
echo ${sentence//Go/Rust}      # Rust is good. Rust is fast.

# Delete pattern (first match)
echo ${name/Spring /}          # Boot

# Uppercase/Lowercase (bash 4+)
echo ${name^^}   # SPRING BOOT
echo ${name,,}   # spring boot
echo ${name^}    # Spring Boot (capitalize first letter)

# Remove prefix
path="/home/user/projects/myapp"
echo ${path#/home/user/}       # projects/myapp (shortest prefix match)
echo ${path##*/}               # myapp (longest prefix match — basename)

# Remove suffix
file="MyActivity.java"
echo ${file%.java}             # MyActivity (shortest suffix match)
echo ${file%%.*}               # MyActivity (longest suffix match)
```

### Numeric Variables and Arithmetic

```bash
# Integer arithmetic with (( ))
a=10
b=3

echo $((a + b))    # 13
echo $((a - b))    # 7
echo $((a * b))    # 30
echo $((a / b))    # 3 (integer division)
echo $((a % b))    # 1 (modulo)
echo $((a ** b))   # 1000 (exponent — bash 4+)

# Increment / Decrement
((a++))
((a--))
((a += 5))
((a *= 2))

# Using let
let "result = a * b"
echo $result

# Floating point: use bc or awk (bash has no native float support)
result=$(echo "scale=2; 10 / 3" | bc)   # 3.33

result=$(awk 'BEGIN { printf "%.2f\n", 10/3 }')  # 3.33

# Comparison in arithmetic context
if ((a > b)); then echo "a is greater"; fi
if ((a == 10)); then echo "a is ten"; fi
```

### Default Values & Parameter Expansion

```bash
# ${var:-default}   Use default if var is unset or empty
name="${1:-World}"
echo "Hello, $name!"    # Hello, World! if no argument given

# ${var:=default}   Assign and use default if var is unset or empty
: "${LOG_LEVEL:=INFO}"  # Set LOG_LEVEL to INFO if not already set
echo $LOG_LEVEL

# ${var:?error_message}   Exit with error if var is unset or empty
db_host="${DB_HOST:?Error: DB_HOST environment variable is required}"

# ${var:+alternate}   Use alternate value if var IS set (opposite of :-)
debug_flag="${DEBUG:+--verbose}"
./my_script $debug_flag   # Passes --verbose only if DEBUG is set
```

---

## 6. Input / Output & Redirection

### Standard Streams

Every process has three standard streams:

| Stream | Number | Default | Redirect symbol |
|--------|--------|---------|-----------------|
| stdin  | 0      | Keyboard | `<` or `0<` |
| stdout | 1      | Terminal | `>` or `1>` |
| stderr | 2      | Terminal | `2>` |

```bash
# Redirect stdout to a file (overwrite)
echo "Hello" > output.txt
ls -la > directory_listing.txt

# Redirect stdout (append)
echo "line 1" > file.txt
echo "line 2" >> file.txt   # Appends; doesn't overwrite

# Redirect stderr
./build.sh 2> errors.log

# Redirect both stdout and stderr to the same file
./build.sh > all_output.log 2>&1   # 2>&1 means "send stderr to wherever stdout goes"
./build.sh &> all_output.log       # Bash shorthand for the above

# Discard output (send to /dev/null — the black hole)
./noisy_script.sh > /dev/null          # Discard stdout only
./noisy_script.sh 2> /dev/null         # Discard stderr only
./noisy_script.sh > /dev/null 2>&1     # Discard everything
./noisy_script.sh &> /dev/null         # Same as above (bash)

# Redirect stdin from a file
./import_data.sh < data.csv
sort < names.txt

# Read from stdin interactively
echo "Enter your name:"
read user_name
echo "Hello, $user_name"
```

### Pipes

A pipe (`|`) connects the stdout of one command to the stdin of the next.

```bash
# Count lines in a file
cat access.log | wc -l

# Find all unique IP addresses in a log
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# Filter and view
ps aux | grep java | grep -v grep

# Chain commands
cat /etc/passwd | cut -d: -f1 | sort | less
```

### Here Documents (Heredoc)

A heredoc lets you write multi-line strings inline in a script — very useful for creating config files or SQL queries.

```bash
# Basic heredoc
cat << EOF
This is line 1
This is line 2
Variable expansion works: $USER
EOF

# Heredoc without variable expansion (quote the delimiter)
cat << 'EOF'
This is literal: $USER
No expansion: $(hostname)
EOF

# Redirect heredoc into a file
cat > /tmp/my_config.yml << EOF
server:
  host: localhost
  port: ${APP_PORT:-8080}
database:
  url: ${DB_URL}
EOF

# Indented heredoc (bash 4+ — strips leading tabs)
cat << -EOF
	This works with tabs
	Tabs are stripped
	EOF

# Pipe heredoc to a command
mysql -u root -p"$DB_PASS" my_db << EOF
SELECT * FROM users WHERE active = 1;
EOF
```

### Here Strings

```bash
# Feed a single string to stdin
grep "pattern" <<< "This string contains the pattern"

# Useful for parsing
version=$(java -version 2>&1 | head -1)
major=$(awk -F'"' '{print $2}' <<< "$version" | cut -d. -f1)
```

### tee — Split Output

```bash
# tee writes to both a file AND stdout
./build.sh | tee build.log                 # See output AND save it
./build.sh | tee -a build.log              # Append to file
./build.sh 2>&1 | tee full_output.log      # Capture everything
```

---

## 7. Control Flow

### if / elif / else

```bash
# Basic syntax
if [ condition ]; then
    # commands
elif [ other_condition ]; then
    # commands
else
    # commands
fi

# Single-line form (useful for simple guards)
[ -z "$1" ] && echo "Usage: $0 <arg>" && exit 1
```

### Test Expressions

#### File Tests

```bash
[ -e path ]    # exists (file or directory)
[ -f path ]    # is a regular file
[ -d path ]    # is a directory
[ -L path ]    # is a symbolic link
[ -r path ]    # is readable
[ -w path ]    # is writable
[ -x path ]    # is executable
[ -s path ]    # exists and is non-empty
[ -z file ]    # file exists and has zero size

# Examples
if [ -f "build.gradle" ]; then
    echo "Gradle project detected"
fi

if [ ! -d "build" ]; then
    mkdir build
fi

if [ -x "/usr/bin/java" ]; then
    java -version
else
    echo "Java not found!"
    exit 1
fi
```

#### String Tests

```bash
[ -z "$str" ]        # string is empty (zero length)
[ -n "$str" ]        # string is non-empty
[ "$a" = "$b" ]      # strings are equal (POSIX)
[ "$a" != "$b" ]     # strings are not equal
[ "$a" < "$b" ]      # a sorts before b (lexicographic)
[[ "$str" == *.java ]]   # Pattern matching (bash [[ ]] only)
[[ "$str" =~ ^[0-9]+$ ]] # Regex match (bash [[ ]] only)
```

#### Numeric Tests

```bash
[ "$a" -eq "$b" ]    # equal
[ "$a" -ne "$b" ]    # not equal
[ "$a" -lt "$b" ]    # less than
[ "$a" -le "$b" ]    # less than or equal
[ "$a" -gt "$b" ]    # greater than
[ "$a" -ge "$b" ]    # greater than or equal

# Alternative: arithmetic (( )) — preferred for numbers
if (( a > b )); then echo "a is greater"; fi
if (( version >= 21 )); then echo "Java 21+"; fi
```

#### Logical Operators

```bash
# Inside [ ]: use -a (AND) and -o (OR)
[ -f file.txt -a -r file.txt ]   # file exists AND is readable

# Inside [[ ]]: use && and ||  (preferred)
[[ -f file.txt && -r file.txt ]]

# Outside test: use && and || between commands
[ -f "pom.xml" ] && echo "Maven project"
[ -f "build.gradle" ] || [ -f "build.gradle.kts" ] && echo "Gradle project"

# NOT operator
[ ! -f file.txt ]   # file does NOT exist
[[ ! "$name" =~ error ]]
```

#### [ ] vs [[ ]] — Use [[ ]] in Bash

```bash
# [ ] is POSIX sh, more portable but has gotchas:
# Variables must always be quoted inside [ ]
file=""
[ -f $file ]    # BUG: becomes [ -f ] which is always true
[ -f "$file" ]  # Correct

# [[ ]] is bash-specific, smarter, safer:
[[ -f $file ]]   # Safe even without quotes (but still quote for clarity)
[[ "$a" == "hello world" ]]  # No word splitting; spaces are fine
[[ "$str" =~ ^[A-Z] ]]       # Regex support
[[ "$str" == *.txt ]]        # Glob pattern support
```

### case Statement

```bash
case "$variable" in
    "start" | "run")
        echo "Starting..."
        ;;
    "stop")
        echo "Stopping..."
        ;;
    "status")
        echo "Checking status..."
        ;;
    *.log)
        echo "Log file detected: $variable"
        ;;
    [0-9]*)
        echo "Starts with a digit"
        ;;
    *)
        echo "Unknown command: $variable"
        exit 1
        ;;
esac

# Real-world example: script argument dispatcher
action="${1:-help}"
case "$action" in
    build)   ./gradlew assembleRelease ;;
    test)    ./gradlew test ;;
    clean)   ./gradlew clean ;;
    deploy)  ./deploy.sh ;;
    help|--help|-h)
        echo "Usage: $0 {build|test|clean|deploy}"
        ;;
    *)
        echo "Unknown action: $action"
        exit 1
        ;;
esac
```

### Loops

#### for Loop

```bash
# Iterate over a list
for name in Alice Bob Charlie; do
    echo "Hello, $name"
done

# Iterate over files
for file in *.java; do
    echo "Processing: $file"
    javac "$file"
done

# C-style for loop
for ((i = 0; i < 10; i++)); do
    echo "Iteration $i"
done

# Iterate over a range
for i in {1..5}; do
    echo "Count: $i"
done

# Range with step
for i in {0..100..10}; do
    echo "$i"
done

# Iterate over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# Safer: iterate over array (handles spaces in names)
files=("My File.java" "Other File.kt" "Build.gradle")
for file in "${files[@]}"; do
    echo "File: $file"
done
```

#### while Loop

```bash
# Basic while
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Read file with line numbers
line_num=0
while IFS= read -r line; do
    ((line_num++))
    echo "$line_num: $line"
done < input.txt

# Poll until condition is met
while ! curl -sf http://localhost:8080/health; do
    echo "Waiting for server to start..."
    sleep 2
done
echo "Server is up!"

# Infinite loop with break
while true; do
    read -r -p "Enter command (quit to exit): " cmd
    [[ "$cmd" == "quit" ]] && break
    eval "$cmd"
done
```

#### until Loop

```bash
# until loops while condition is FALSE (opposite of while)
count=0
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

#### Loop Control

```bash
# break: exit the loop
for i in {1..10}; do
    [ $i -eq 5 ] && break
    echo $i
done    # Prints 1 2 3 4

# continue: skip to next iteration
for i in {1..10}; do
    [ $((i % 2)) -eq 0 ] && continue
    echo $i
done    # Prints only odd numbers

# break/continue with nested loops (use level number)
for i in {1..3}; do
    for j in {1..3}; do
        [ $j -eq 2 ] && break 2   # Break out of BOTH loops
        echo "$i,$j"
    done
done
```

---

## 8. Functions

### Defining and Calling Functions

```bash
# Function definition (two equivalent syntaxes)
my_function() {
    echo "Hello from function"
}

function my_function {    # 'function' keyword optional in bash
    echo "Hello from function"
}

# Call the function (no parentheses when calling)
my_function

# Functions must be defined BEFORE they are called
# (unless using a loader pattern)
```

### Parameters and Return Values

```bash
# Parameters are accessed as $1, $2, etc. (same as script args)
greet() {
    local name="$1"
    local greeting="${2:-Hello}"   # Default value
    echo "$greeting, $name!"
}

greet "Alice"              # Hello, Alice!
greet "Bob" "Good morning" # Good morning, Bob!

# Return status code (0-255 only; not a value)
is_java_installed() {
    command -v java > /dev/null 2>&1
    # command -v returns 0 if found, non-zero otherwise
    # The function returns that exit code automatically
}

if is_java_installed; then
    echo "Java is available"
else
    echo "Java not found"
fi

# Return a value via stdout (use command substitution to capture)
get_java_version() {
    java -version 2>&1 | head -1 | awk -F'"' '{print $2}'
}

version=$(get_java_version)
echo "Java version: $version"
```

### Local Variables and Scope

```bash
# Without local: variable leaks into global scope
bad_function() {
    result="I leaked"   # This modifies the global $result!
}

# With local: variable is scoped to the function
good_function() {
    local result="I am contained"
    echo "$result"
}

# Nameref (bash 4.3+) — function can modify caller's variable by name
increment() {
    local -n ref=$1    # -n creates a nameref to the variable named by $1
    ((ref++))
}

counter=5
increment counter
echo $counter   # 6
```

### Advanced Function Patterns

```bash
# Function that validates arguments
create_user() {
    if [[ $# -lt 2 ]]; then
        echo "Usage: create_user <username> <email>" >&2
        return 1
    fi

    local username="$1"
    local email="$2"

    # Validate email format
    if [[ ! "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
        echo "Error: Invalid email: $email" >&2
        return 1
    fi

    echo "Creating user: $username ($email)"
    # ... actual creation logic
}

# Function that returns multiple values via global vars or output
get_db_info() {
    # Technique 1: set global vars (fragile but common)
    DB_HOST="localhost"
    DB_PORT=5432
    DB_NAME="myapp"
}

get_db_info
echo "Connecting to $DB_HOST:$DB_PORT/$DB_NAME"

# Technique 2: print key=value pairs, let caller parse
get_dimensions() {
    echo "width=1920"
    echo "height=1080"
}

eval "$(get_dimensions)"
echo "Resolution: ${width}x${height}"
```

### Library Pattern — Reusable Function Files

```bash
# File: lib/logging.sh
#!/usr/bin/env bash

readonly LOG_LEVELS=([DEBUG]=0 [INFO]=1 [WARN]=2 [ERROR]=3)
LOG_LEVEL="${LOG_LEVEL:-INFO}"

log() {
    local level="$1"
    local message="$2"
    local timestamp
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')

    # Only log if level >= configured LOG_LEVEL
    if (( LOG_LEVELS[$level] >= LOG_LEVELS[$LOG_LEVEL] )); then
        printf "[%s] [%s] %s\n" "$timestamp" "$level" "$message" >&2
    fi
}

log_debug() { log "DEBUG" "$*"; }
log_info()  { log "INFO"  "$*"; }
log_warn()  { log "WARN"  "$*"; }
log_error() { log "ERROR" "$*"; }

# File: my_script.sh
#!/usr/bin/env bash
# Source the library (relative to script location)
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "$SCRIPT_DIR/lib/logging.sh"

log_info "Starting deployment..."
log_error "Database connection failed"
```

---

## 9. String Manipulation

### Essential String Operations

```bash
str="  Hello, Spring Boot World!  "

# Length
echo ${#str}

# Trim leading/trailing whitespace (no built-in; use sed or parameter expansion)
trimmed=$(echo "$str" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
# Or with bash parameter expansion:
shopt -s extglob
trimmed="${str##+([[:space:]])}"  # leading
trimmed="${trimmed%%+([[:space:]])}"  # trailing

# Convert case (bash 4+)
upper="${str^^}"
lower="${str,,}"

# Split string into array
sentence="one:two:three:four"
IFS=':' read -ra parts <<< "$sentence"
echo "${parts[0]}"   # one
echo "${parts[2]}"   # three
echo "${#parts[@]}"  # 4

# Join array into string
arr=("apple" "banana" "cherry")
joined=$(IFS=','; echo "${arr[*]}")
echo "$joined"  # apple,banana,cherry

# Check if string contains substring
haystack="Hello, Spring Boot"
needle="Spring"
if [[ "$haystack" == *"$needle"* ]]; then
    echo "Found!"
fi

# Or:
if echo "$haystack" | grep -q "$needle"; then
    echo "Found!"
fi

# Extract between patterns
text="version=1.2.3-RELEASE"
version="${text#version=}"            # 1.2.3-RELEASE
version="${version%-RELEASE}"         # 1.2.3
major="${version%%.*}"                # 1
minor="${version#*.}"; minor="${minor%%.*}"  # 2
patch="${version##*.}"                # 3

# Replace using sed for complex patterns
echo "foo_bar_baz" | sed 's/_/-/g'   # foo-bar-baz
echo "2026-06-23" | sed 's/-/\//g'   # 2026/06/23
```

### printf — Formatted Output

```bash
# printf is more powerful and reliable than echo
printf "Name: %s\n" "Alice"
printf "Age:  %d\n" 30
printf "PI:   %.4f\n" 3.14159
printf "Hex:  %x\n" 255           # ff
printf "Padded: %10s|\n" "hi"     #          hi|
printf "Padded: %-10s|\n" "hi"    # hi        |

# Print table
printf "%-20s %-10s %-10s\n" "Name" "Version" "Status"
printf "%-20s %-10s %-10s\n" "spring-boot" "3.3.0" "UP"
printf "%-20s %-10s %-10s\n" "my-service" "1.2.1" "DOWN"

# Useful: print without newline
printf "Processing... "
sleep 1
printf "Done!\n"
```

### awk — Text Processing Engine

```bash
# awk is a full language; these are the patterns you'll use daily

# Print specific columns (fields separated by whitespace by default)
ps aux | awk '{print $1, $2}'                  # user, pid
df -h | awk '{print $1, $5}'                   # filesystem, use%

# Custom field separator
cat /etc/passwd | awk -F: '{print $1, $7}'     # user, shell
echo "a,b,c,d" | awk -F, '{print $3}'         # c

# Filter rows
ps aux | awk '$3 > 50.0 {print $1, $2, $3}'   # processes using > 50% CPU
df -h | awk 'NR > 1 {print}'                  # skip header line (NR = row number)

# Sum a column
df -k | awk 'NR>1 {sum += $3} END {print "Total used:", sum, "KB"}'

# Replace field
echo "2026-06-23" | awk -F- '{print $3"/"$2"/"$1}'  # 23/06/2026

# Print between patterns
awk '/START/,/END/ {print}' file.txt

# BEGIN and END blocks
awk 'BEGIN {print "--- Start ---"} {print NR": "$0} END {print "--- End ---"}' file.txt
```

### sed — Stream Editor

```bash
# Substitute (most common use)
echo "Hello World" | sed 's/World/Bash/'           # Hello Bash
echo "aabbcc" | sed 's/b/X/'                      # aaXbcc (first occurrence)
echo "aabbcc" | sed 's/b/X/g'                     # aaXXcc (all occurrences)
echo "UPPER" | sed 's/.*/\L&/'                    # upper (GNU sed only)

# Delete lines
sed '/pattern/d' file.txt                          # Delete lines matching pattern
sed '1d' file.txt                                  # Delete first line
sed '$d' file.txt                                  # Delete last line
sed '/^$/d' file.txt                              # Delete empty lines
sed '/^[[:space:]]*$/d' file.txt                  # Delete blank/whitespace lines

# Print specific lines
sed -n '5p' file.txt                               # Print line 5
sed -n '5,10p' file.txt                           # Print lines 5-10
sed -n '/START/,/END/p' file.txt                  # Print between patterns

# Edit file in-place
sed -i 's/old/new/g' file.txt                     # Linux
sed -i '' 's/old/new/g' file.txt                  # Mac (requires empty string arg)

# Insert/append lines
sed '3i\New line inserted before line 3' file.txt
sed '3a\New line appended after line 3' file.txt

# Multiple commands with -e
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt
```

---

## 10. Arrays & Associative Arrays

### Indexed Arrays

```bash
# Declaration
declare -a fruits=("apple" "banana" "cherry")
# Or simpler:
fruits=("apple" "banana" "cherry")

# Access elements (0-indexed)
echo "${fruits[0]}"     # apple
echo "${fruits[1]}"     # banana
echo "${fruits[-1]}"    # cherry (last element; bash 4.2+)

# All elements
echo "${fruits[@]}"     # apple banana cherry (as separate words — use in loops)
echo "${fruits[*]}"     # apple banana cherry (as one string — use for joining)

# Length
echo "${#fruits[@]}"    # 3

# Append
fruits+=("date" "elderberry")
echo "${fruits[@]}"     # apple banana cherry date elderberry

# Slice: ${array[@]:start:length}
echo "${fruits[@]:1:2}"  # banana cherry

# Iterate
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Delete element
unset 'fruits[1]'   # Removes "banana" but leaves a gap (sparse array)
echo "${fruits[@]}"  # apple cherry date elderberry (index 1 is gone)

# Re-index after deletion
fruits=("${fruits[@]}")  # Rebuilds the array with consecutive indices
```

### Associative Arrays (Dictionaries / HashMaps)

```bash
# Requires bash 4+; on Mac with bash 3.2, use Homebrew bash
declare -A config

config["db_host"]="localhost"
config["db_port"]="5432"
config["db_name"]="myapp"
config["db_user"]="admin"

# Access
echo "${config["db_host"]}"  # localhost

# All keys
echo "${!config[@]}"   # db_host db_port db_name db_user

# All values
echo "${config[@]}"    # localhost 5432 myapp admin

# Check if key exists
if [[ -v config["db_host"] ]]; then
    echo "db_host is set"
fi

# Iterate over key-value pairs
for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done

# Delete a key
unset 'config["db_port"]'

# Practical: parse a .env file into an associative array
declare -A env_vars
while IFS='=' read -r key value; do
    [[ "$key" =~ ^[[:space:]]*# ]] && continue  # skip comments
    [[ -z "$key" ]] && continue                  # skip empty lines
    env_vars["$key"]="${value//\"/}"             # strip quotes
done < .env
```

### Working with Command Output as Arrays

```bash
# Store lines of output in an array
mapfile -t lines < file.txt      # Bash 4+ (also: readarray)
echo "${lines[0]}"               # First line
echo "${#lines[@]}"              # Number of lines

# Store command output
mapfile -t processes < <(ps aux | grep java)
for p in "${processes[@]}"; do
    echo "$p"
done

# Safer alternative to $(command) that handles newlines
IFS=$'\n' read -r -d '' -a gradle_tasks < <(./gradlew tasks --all 2>/dev/null | grep -E "^[a-zA-Z]" && printf '\0')
```

---

## 11. File & Directory Operations

### Finding and Testing Files

```bash
# Comprehensive file test function
check_file() {
    local file="$1"

    echo "File: $file"
    [ -e "$file" ] && echo "  Exists: yes" || echo "  Exists: no"
    [ -f "$file" ] && echo "  Type: regular file"
    [ -d "$file" ] && echo "  Type: directory"
    [ -L "$file" ] && echo "  Type: symlink -> $(readlink "$file")"
    [ -r "$file" ] && echo "  Readable: yes"
    [ -w "$file" ] && echo "  Writable: yes"
    [ -x "$file" ] && echo "  Executable: yes"
    [ -s "$file" ] && echo "  Non-empty: yes" || echo "  Empty: yes"
}

# Find files by various criteria
find . -name "*.java" -newer pom.xml           # Modified after pom.xml
find . -name "*.log" -mtime +7                 # Older than 7 days
find . -type f -perm -u+x                      # Executable files
find . -name "*.class" -delete                 # Find and delete .class files
find . -maxdepth 2 -name "*.gradle"            # Max 2 levels deep
find . -path "*/build/*" -prune -o -name "*.java" -print  # Exclude build dir
```

### Reading Files

```bash
# Read entire file into variable
content=$(cat file.txt)

# Read file line by line (correct way — handles spaces and special chars)
while IFS= read -r line; do
    process_line "$line"
done < "file.txt"

# Read CSV file (field separator)
while IFS=',' read -r name email role; do
    echo "Name: $name, Email: $email, Role: $role"
done < users.csv

# Read file with line count
line_num=0
while IFS= read -r line; do
    ((line_num++))
    echo "$line_num: $line"
done < file.txt
echo "Total: $line_num lines"

# Read specific lines using sed or awk
sed -n '10,20p' large_file.txt    # Lines 10-20
awk 'NR==15' large_file.txt       # Exactly line 15
```

### Writing Files

```bash
# Write (overwrite)
echo "content" > file.txt
printf "line1\nline2\n" > file.txt

# Append
echo "new line" >> file.txt

# Write multi-line with heredoc
cat > config.yml << EOF
server:
  port: ${PORT:-8080}
  host: ${HOST:-0.0.0.0}
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
EOF

# Write to file requiring sudo
echo "text" | sudo tee /etc/myapp/config.conf > /dev/null
echo "more text" | sudo tee -a /etc/myapp/config.conf > /dev/null
```

### Temporary Files and Directories

```bash
# Create secure temp file (unique name, prevents race conditions)
tmpfile=$(mktemp)                    # /tmp/tmp.XXXXXX
tmpdir=$(mktemp -d)                  # /tmp/tmp.XXXXXX (directory)
tmpfile=$(mktemp /tmp/myapp.XXXXXX)  # Custom prefix
tmpfile=$(mktemp --suffix=.json)     # With suffix

# Always clean up temp files on exit
cleanup() {
    rm -f "$tmpfile"
    rm -rf "$tmpdir"
}
trap cleanup EXIT   # EXIT runs on any exit including errors

# Use temp files for atomic writes (write to temp, then move)
tmpfile=$(mktemp)
generate_config > "$tmpfile"           # Write new content to temp file
mv "$tmpfile" /etc/myapp/config.json   # Atomic replacement
```

### File Locking

```bash
# Prevent concurrent script execution using a lock file
LOCK_FILE="/tmp/myapp.lock"

acquire_lock() {
    if ! mkdir "$LOCK_FILE" 2>/dev/null; then
        echo "Another instance is running (lock: $LOCK_FILE)" >&2
        exit 1
    fi
    trap 'rm -rf "$LOCK_FILE"' EXIT
}

acquire_lock
echo "Doing exclusive work..."
```

---

## 12. Process Management

### Running Commands

```bash
# Foreground (blocks until complete)
./long_build.sh

# Background (runs independently)
./long_build.sh &
echo "Build started in background, PID: $!"

# Wait for background job
pid=$!
wait $pid
echo "Build finished with exit code: $?"

# Wait for all background jobs
for service in serviceA serviceB serviceC; do
    ./start_service.sh "$service" &
done
wait   # Wait for all background jobs to finish
echo "All services started"

# Run with timeout (kills process if it takes too long)
timeout 30 ./health_check.sh || echo "Health check timed out!"
timeout --kill-after=5 30 ./script.sh  # Send KILL after 5 more seconds
```

### Signals

```bash
# Send signals to processes
kill PID            # SIGTERM (graceful shutdown request)
kill -9 PID         # SIGKILL (force kill — cannot be caught)
kill -HUP PID       # SIGHUP (reload config without restart)
kill -0 PID         # Check if process exists (no signal sent)

# Find and kill by name
pkill -f "java.*MyApp"       # Kill by process name pattern
killall java                  # Kill all processes named "java"
pgrep -f "spring.*8080"      # Find PIDs matching pattern

# Signal trapping — respond to signals in your script
handle_sigterm() {
    echo "SIGTERM received, cleaning up..."
    # Clean up resources
    rm -f "$LOCK_FILE"
    exit 0
}

handle_sigint() {
    echo "Interrupted by user"
    exit 130
}

trap handle_sigterm SIGTERM
trap handle_sigint  SIGINT
trap handle_sigint  SIGQUIT

# Common trap patterns
trap 'echo "Error on line $LINENO"; exit 1' ERR
trap 'rm -f "$TMPFILE"; echo "Cleanup done"' EXIT
```

### Job Control

```bash
jobs              # List background jobs
fg                # Bring last background job to foreground
fg %1             # Bring job 1 to foreground
bg                # Resume last stopped job in background
bg %2             # Resume job 2 in background

# Suspend a foreground job
# Press Ctrl+Z — sends SIGSTOP

# Disown — detach job from terminal (survives terminal close)
./long_process.sh &
disown $!

# nohup — run immune to hangup signal
nohup ./server.sh > server.log 2>&1 &
echo "Server started, PID: $!, logs: server.log"
```

### Process Substitution

```bash
# Process substitution creates a temporary file-like object
# <(command) — output of command as a readable file
# >(command) — input to command as a writable file

# Compare two command outputs without temp files
diff <(./script_v1.sh) <(./script_v2.sh)
diff <(sort file1.txt) <(sort file2.txt)

# Use multiple inputs that would normally require temp files
comm <(sort list1.txt) <(sort list2.txt)   # Lines in common

# Tee to multiple processes
./build.sh | tee >(grep -i error > errors.log) >(wc -l > line_count.txt)
```

---

## 13. Regular Expressions & Text Processing

### Regex in Bash

```bash
# Test if string matches regex with [[ =~ ]]
email="user@example.com"
if [[ "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
    echo "Valid email"
fi

# Capture groups — stored in BASH_REMATCH array
version="v1.23.456-beta"
if [[ "$version" =~ ^v([0-9]+)\.([0-9]+)\.([0-9]+) ]]; then
    echo "Full match: ${BASH_REMATCH[0]}"   # v1.23.456
    echo "Major: ${BASH_REMATCH[1]}"        # 1
    echo "Minor: ${BASH_REMATCH[2]}"        # 23
    echo "Patch: ${BASH_REMATCH[3]}"        # 456
fi

# grep with regex
grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}" logfile.txt  # ISO dates
grep -P "\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b" access.log  # IP addresses (Perl regex)
```

### cut — Extract Fields

```bash
# -d: delimiter, -f: field number
echo "apple:banana:cherry" | cut -d: -f2       # banana
echo "apple:banana:cherry" | cut -d: -f1,3     # apple:cherry
echo "apple:banana:cherry" | cut -d: -f2-      # banana:cherry (2 to end)

# Cut by character position
echo "Hello World" | cut -c1-5                 # Hello
echo "2026-06-23" | cut -c6-7                  # 06 (month)

# Real-world: extract data from /etc/passwd
cut -d: -f1,7 /etc/passwd    # username:shell
```

### sort — Sorting

```bash
sort file.txt                     # Alphabetical
sort -r file.txt                  # Reverse
sort -n numbers.txt               # Numeric sort
sort -k2 -n data.txt              # Sort by 2nd field numerically
sort -k2 -rn data.txt             # Reverse numeric by 2nd field
sort -t: -k3 -n /etc/passwd       # Sort passwd by UID (field 3, : separator)
sort -u file.txt                  # Sort and remove duplicates
sort -k1,1 -k2,2n file.txt        # Sort by field 1 alpha, then field 2 numeric
```

### uniq — Unique Lines

```bash
# uniq works on SORTED input
sort file.txt | uniq                # Unique lines
sort file.txt | uniq -d             # Only lines that appear more than once
sort file.txt | uniq -u             # Only lines that appear exactly once
sort file.txt | uniq -c             # Count occurrences, prefix each line
sort file.txt | uniq -c | sort -rn  # Most frequent first

# Count occurrences of HTTP status codes in access log
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

### jq — JSON Processing (Must-Know for APIs)

```bash
# Install: brew install jq  or  apt install jq

# Parse JSON
echo '{"name":"Alice","age":30}' | jq '.name'         # "Alice"
echo '{"name":"Alice","age":30}' | jq -r '.name'       # Alice (raw, no quotes)

# Nested
echo '{"user":{"name":"Alice","roles":["admin","dev"]}}' | jq '.user.name'
echo '{"user":{"roles":["admin","dev"]}}' | jq '.user.roles[0]'

# Array operations
echo '[1,2,3,4,5]' | jq '.[]'          # Explode array to individual values
echo '[1,2,3,4,5]' | jq 'map(. * 2)'   # [2,4,6,8,10]
echo '[1,2,3,4,5]' | jq 'select(. > 3)' # Does nothing (use with .[])
echo '[1,2,3,4,5]' | jq '[.[] | select(. > 3)]'   # [4,5]

# Build new JSON
jq -n --arg name "Alice" --argjson age 30 '{"name":$name,"age":$age}'

# Process API response
curl -s "https://api.github.com/repos/spring-projects/spring-boot/releases/latest" \
    | jq -r '.tag_name'   # Latest Spring Boot version

# Filter array of objects
curl -s "https://api.example.com/users" \
    | jq '[.[] | select(.role == "admin") | {name: .name, email: .email}]'

# Iterate over JSON array in bash
curl -s "https://api.example.com/services" | jq -r '.[].name' | while read -r service; do
    echo "Checking: $service"
    curl -sf "https://api.example.com/$service/health" || echo "$service is DOWN"
done
```

---

## 14. Error Handling & Defensive Scripting

### The Three Essential Flags

Every production bash script should start with these:

```bash
#!/usr/bin/env bash
set -e    # Exit immediately if any command returns non-zero exit code
set -u    # Treat unset variables as errors (prevents typo bugs)
set -o pipefail  # A pipe fails if ANY command in it fails (not just the last)

# Shorthand
set -euo pipefail

# Or in the shebang area:
set -euo pipefail
IFS=$'\n\t'   # Safer IFS: split on newlines and tabs only (not spaces)
```

**Why these matter:**

```bash
# Without set -e:
rm -rf "$OUTPUT_DIR/"   # If $OUTPUT_DIR is empty, this is: rm -rf /
echo "Done"              # Script continues happily

# With set -e + set -u:
set -euo pipefail
rm -rf "${OUTPUT_DIR:?OUTPUT_DIR is not set}/"   # Fails safely if empty
```

### Handling Errors Explicitly

```bash
# Don't let set -e stop a command you expect might fail
if ! grep -q "pattern" file.txt; then
    echo "Pattern not found"
fi

# The || idiom for simple fallbacks
mkdir -p "$dir" || { echo "Failed to create $dir" >&2; exit 1; }

# The && idiom for success chains
./configure && make && make install

# Temporarily disable set -e
set +e
result=$(command_that_might_fail)
exit_code=$?
set -e
if [ $exit_code -ne 0 ]; then
    echo "Command failed with exit code $exit_code"
fi

# Explicit error checking function
die() {
    echo "ERROR: $*" >&2
    exit 1
}

check() {
    local desc="$1"
    shift
    "$@" || die "Failed: $desc"
}

check "Install dependencies" npm install
check "Run tests" npm test
check "Build" npm run build
```

### trap for Cleanup and Error Handling

```bash
#!/usr/bin/env bash
set -euo pipefail

# State tracking
TEMP_DIR=""
START_TIME=$(date +%s)

cleanup() {
    local exit_code=$?
    local end_time
    end_time=$(date +%s)

    echo ""
    if [ $exit_code -eq 0 ]; then
        echo "Script completed successfully in $((end_time - START_TIME))s"
    else
        echo "Script FAILED with exit code $exit_code (line $LINENO)" >&2
    fi

    # Clean up temp files
    [ -n "$TEMP_DIR" ] && rm -rf "$TEMP_DIR"
    echo "Cleanup done"
}

# Register cleanup to run on ANY exit
trap cleanup EXIT

# Register specific signal handlers
trap 'echo "Interrupted!"; exit 130' INT TERM

# Create temp dir (cleanup will remove it automatically)
TEMP_DIR=$(mktemp -d)
echo "Working in $TEMP_DIR"

# ... rest of script
```

### Error Propagation Through Functions

```bash
# Good: functions return meaningful exit codes
fetch_data() {
    local url="$1"
    local output="$2"

    if ! curl -sf --retry 3 --max-time 30 "$url" -o "$output"; then
        echo "ERROR: Failed to fetch $url" >&2
        return 1   # Return failure; caller decides what to do
    fi
    return 0
}

process_data() {
    local data_file="$1"
    if ! fetch_data "$API_URL" "$data_file"; then
        echo "ERROR: Cannot proceed without data" >&2
        return 1
    fi
    # Process the data...
}

# Main
if ! process_data "/tmp/data.json"; then
    echo "FATAL: Processing failed" >&2
    exit 1
fi
```

### Input Validation Patterns

```bash
validate_args() {
    local usage="Usage: $0 <environment> <version>"

    [[ $# -eq 2 ]]                        || { echo "$usage" >&2; exit 1; }
    [[ "$1" =~ ^(dev|staging|prod)$ ]]    || { echo "Invalid environment: $1" >&2; exit 1; }
    [[ "$2" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || { echo "Invalid version: $2" >&2; exit 1; }
}

validate_args "$@"
ENVIRONMENT="$1"
VERSION="$2"
```

---

## 15. Debugging Techniques

### Bash Debug Mode

```bash
# Run entire script with tracing
bash -x ./myscript.sh

# Enable/disable tracing within script
set -x    # Enable: prints each command before executing
set +x    # Disable

# Trace only a specific section
{
    set -x
    ./complicated_function
    set +x
} 2>&1 | grep -v "^+ echo"   # Filter out echo traces

# Verbose mode: print each line before expansion
set -v
```

### Custom Debug Output

```bash
# Debug flag pattern
DEBUG="${DEBUG:-false}"

debug() {
    [[ "$DEBUG" == "true" ]] && echo "[DEBUG] $*" >&2
}

debug "Processing file: $filename"
debug "Config values: host=$host port=$port"

# Run with: DEBUG=true ./script.sh
```

### Tracing Variables

```bash
# Print variable name + value (bash 4.2+ with typeset -p)
name="World"
typeset -p name    # declare -- name="World"

# Watch variable for changes (using DEBUG trap)
watch_var() {
    declare -g "${1}_WATCH=true"
}

# Trap assignments to a variable (advanced)
# Use: TRACE=1 bash ./script.sh

# LINENO — current line number
echo "At line $LINENO"

# FUNCNAME — call stack
show_stack() {
    local i
    for ((i = 1; i < ${#FUNCNAME[@]}; i++)); do
        echo "  #$((i-1)) ${FUNCNAME[$i]} (line ${BASH_LINENO[$((i-1))]})"
    done
}

# PS4 — customize the trace prefix for set -x
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x
```

### Testing Script Logic

```bash
# Dry-run pattern
DRY_RUN="${DRY_RUN:-false}"

execute() {
    if [[ "$DRY_RUN" == "true" ]]; then
        echo "[DRY-RUN] $*"
    else
        "$@"
    fi
}

execute rm -rf "$BUILD_DIR"
execute ./gradlew assembleRelease
execute adb install -r app-release.apk

# Run with: DRY_RUN=true ./deploy.sh
```

---

## 16. Advanced Patterns

### Argument Parsing with getopts

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() {
    cat << EOF
Usage: $0 [OPTIONS] <input_file>

Options:
  -e <environment>  Target environment (dev|staging|prod) [default: dev]
  -v <version>      App version to deploy
  -n                Dry-run mode (print commands, don't execute)
  -f                Force deployment (skip confirmation)
  -h                Show this help
EOF
    exit 1
}

ENVIRONMENT="dev"
VERSION=""
DRY_RUN=false
FORCE=false

while getopts "e:v:nfh" opt; do
    case "$opt" in
        e) ENVIRONMENT="$OPTARG" ;;
        v) VERSION="$OPTARG" ;;
        n) DRY_RUN=true ;;
        f) FORCE=true ;;
        h) usage ;;
        ?) usage ;;
    esac
done

# Shift past options to positional arguments
shift $((OPTIND - 1))

INPUT_FILE="${1:-}"
[ -z "$INPUT_FILE" ] && usage

echo "Environment: $ENVIRONMENT"
echo "Version:     $VERSION"
echo "Input:       $INPUT_FILE"
```

### Long Options with Manual Parsing

```bash
# getopts doesn't support --long-flags; parse manually:
parse_args() {
    while [[ $# -gt 0 ]]; do
        case "$1" in
            --env=*)        ENVIRONMENT="${1#*=}"; shift ;;
            --env)          ENVIRONMENT="$2"; shift 2 ;;
            --version=*)    VERSION="${1#*=}"; shift ;;
            --version)      VERSION="$2"; shift 2 ;;
            --dry-run)      DRY_RUN=true; shift ;;
            --force)        FORCE=true; shift ;;
            --help|-h)      usage ;;
            --)             shift; break ;;  # End of options
            -*)             echo "Unknown option: $1" >&2; usage ;;
            *)              POSITIONAL+=("$1"); shift ;;
        esac
    done
}

declare -a POSITIONAL=()
parse_args "$@"
set -- "${POSITIONAL[@]}"   # Restore positional parameters
```

### Configuration Loading

```bash
# Load .env files safely
load_env() {
    local env_file="${1:-.env}"

    if [[ ! -f "$env_file" ]]; then
        echo "Warning: $env_file not found" >&2
        return 0
    fi

    while IFS='=' read -r key value; do
        # Skip comments and empty lines
        [[ "$key" =~ ^[[:space:]]*# ]] && continue
        [[ -z "${key// }" ]] && continue

        # Strip inline comments and surrounding quotes
        value="${value%%#*}"           # Remove inline comment
        value="${value%"${value##*[![:space:]]}"}"  # Trim trailing whitespace
        value="${value#\"}" value="${value%\"}"    # Strip double quotes
        value="${value#\'}" value="${value%\'}"    # Strip single quotes

        # Export only valid variable names
        if [[ "$key" =~ ^[A-Za-z_][A-Za-z0-9_]*$ ]]; then
            export "$key=$value"
        fi
    done < "$env_file"
}

load_env ".env.local"
load_env ".env"
```

### Parallel Execution

```bash
# Run multiple tasks in parallel and wait for all
run_parallel() {
    local -a pids=()
    local -a results=()

    for task in "$@"; do
        $task &         # Start each task in background
        pids+=($!)      # Track PIDs
    done

    local all_ok=true
    for pid in "${pids[@]}"; do
        if ! wait "$pid"; then
            all_ok=false
        fi
    done

    $all_ok
}

# Practical: run tests in parallel, collect results
run_tests_parallel() {
    local max_jobs="${1:-4}"
    local -a pids=()
    local -a test_files=()

    mapfile -t test_files < <(find . -name "*Test.java" -o -name "*Spec.kt")

    for test_file in "${test_files[@]}"; do
        # Throttle to max_jobs parallel jobs
        while (( $(jobs -r | wc -l) >= max_jobs )); do
            sleep 0.1
        done

        (
            echo "Running: $test_file"
            ./gradlew test --tests "${test_file%.java}" 2>&1
        ) &
        pids+=($!)
    done

    local failed=0
    for pid in "${pids[@]}"; do
        wait "$pid" || ((failed++))
    done

    echo "Tests completed. Failed: $failed"
    return $failed
}
```

### Coprocesses (Advanced Inter-process Communication)

```bash
# coproc creates a background process with bidirectional pipes
coproc my_server { python3 -c "
while True:
    line = input()
    print(f'processed: {line}')
    import sys; sys.stdout.flush()
"; }

# Write to coprocess stdin
echo "hello" >&"${my_server[1]}"

# Read from coprocess stdout
read -r response <&"${my_server[0]}"
echo "Response: $response"   # processed: hello
```

### Script Modules and Namespacing

```bash
# Namespace functions to avoid collisions
db::connect() { local host="$1"; echo "Connecting to $host"; }
db::query()   { local sql="$1"; echo "Running: $sql"; }
db::close()   { echo "Closing connection"; }

utils::log()  { echo "[$(date +%H:%M:%S)] $*" >&2; }
utils::die()  { utils::log "FATAL: $*"; exit 1; }

# Source with guard (prevent double-sourcing)
# In each library file:
[[ -n "${_MY_LIB_LOADED:-}" ]] && return 0
readonly _MY_LIB_LOADED=1
# ... library functions ...
```

---

## 17. Android Development Recipes

### ADB Automation

```bash
#!/usr/bin/env bash
# android_tools.sh — Common ADB tasks

# Check connected devices
adb_check_devices() {
    local devices
    devices=$(adb devices | grep -v "^List" | grep -v "^$" | grep "device$")
    if [ -z "$devices" ]; then
        echo "No devices connected" >&2
        return 1
    fi
    echo "$devices" | awk '{print $1}'
}

# Install APK to all connected devices
install_apk() {
    local apk="$1"
    [[ -f "$apk" ]] || { echo "APK not found: $apk" >&2; return 1; }

    while read -r device; do
        echo "Installing on $device..."
        adb -s "$device" install -r "$apk" && \
            echo "  Success: $device" || \
            echo "  FAILED: $device" >&2
    done < <(adb_check_devices)
}

# Take screenshot on device
screenshot() {
    local output="${1:-screenshot_$(date +%Y%m%d_%H%M%S).png}"
    adb exec-out screencap -p > "$output"
    echo "Screenshot saved: $output"
}

# Pull crash logs
pull_crashes() {
    local package="${1:?Package name required}"
    local output_dir="${2:-./crash_logs}"
    mkdir -p "$output_dir"

    adb logcat -d "*:E" | grep "$package" > "$output_dir/crash_$(date +%Y%m%d_%H%M%S).log"
    echo "Crash logs saved to $output_dir"
}

# Clear app data
clear_app_data() {
    local package="${1:?Package name required}"
    adb shell pm clear "$package"
    echo "Cleared data for $package"
}

# Start specific activity
launch_activity() {
    local package="$1"
    local activity="${2:-$package.MainActivity}"
    adb shell am start -n "$package/$activity"
}

# Enable/disable mobile data
toggle_mobile_data() {
    local state="$1"  # enable or disable
    if [[ "$state" == "enable" ]]; then
        adb shell svc data enable
    else
        adb shell svc data disable
    fi
}

# Simulate network conditions
simulate_network() {
    local speed_kbps="${1:-100}"  # Default: 100 kbps (2G-like)
    local latency_ms="${2:-200}"
    # Requires rooted device or emulator
    adb shell tc qdisc add dev rmnet0 root netem rate "${speed_kbps}kbit" delay "${latency_ms}ms"
}
```

### Emulator Management

```bash
#!/usr/bin/env bash
# emulator_manager.sh

# List available AVDs
list_avds() {
    "$ANDROID_HOME/cmdline-tools/latest/bin/avdmanager" list avd -c
}

# Start emulator by name
start_emulator() {
    local avd_name="${1:?AVD name required}"
    local headless="${2:-false}"

    local flags="-avd $avd_name -no-snapshot-load"
    [[ "$headless" == "true" ]] && flags="$flags -no-window -no-audio -no-boot-anim"

    echo "Starting emulator: $avd_name"
    "$ANDROID_HOME/emulator/emulator" $flags &
    local emu_pid=$!

    echo "Waiting for emulator to boot..."
    adb wait-for-device
    adb shell 'while [[ -z $(getprop sys.boot_completed) ]]; do sleep 1; done'
    echo "Emulator ready! PID: $emu_pid"
}

# Stop all running emulators
stop_emulators() {
    adb devices | grep emulator | cut -f1 | while read -r device; do
        echo "Stopping $device"
        adb -s "$device" emu kill
    done
}

# Create a new AVD
create_avd() {
    local name="${1:?AVD name required}"
    local api_level="${2:-34}"
    local abi="${3:-x86_64}"

    "$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager" \
        "system-images;android-$api_level;google_apis;$abi"

    echo no | "$ANDROID_HOME/cmdline-tools/latest/bin/avdmanager" create avd \
        --name "$name" \
        --package "system-images;android-$api_level;google_apis;$abi" \
        --device "pixel_6"
}
```

### Gradle Build Automation

```bash
#!/usr/bin/env bash
# build_android.sh
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_DIR="$(dirname "$SCRIPT_DIR")"
GRADLEW="$PROJECT_DIR/gradlew"

log() { echo "[$(date +%H:%M:%S)] $*"; }

# Build and test
build_debug() {
    log "Building debug APK..."
    cd "$PROJECT_DIR"
    $GRADLEW clean assembleDebug --parallel --build-cache
    log "Debug APK: $(find . -name 'app-debug.apk' | head -1)"
}

build_release() {
    local keystore="${KEYSTORE_FILE:?Set KEYSTORE_FILE}"
    local key_alias="${KEY_ALIAS:?Set KEY_ALIAS}"
    local key_password="${KEY_PASSWORD:?Set KEY_PASSWORD}"
    local store_password="${STORE_PASSWORD:?Set STORE_PASSWORD}"

    log "Building release AAB..."
    cd "$PROJECT_DIR"
    $GRADLEW bundleRelease \
        -Pandroid.injected.signing.store.file="$keystore" \
        -Pandroid.injected.signing.store.password="$store_password" \
        -Pandroid.injected.signing.key.alias="$key_alias" \
        -Pandroid.injected.signing.key.password="$key_password"

    log "AAB: $(find . -name '*.aab' | head -1)"
}

run_tests() {
    log "Running unit tests..."
    $GRADLEW test --parallel

    log "Running instrumented tests..."
    $GRADLEW connectedAndroidTest
}

analyze_size() {
    log "Analyzing APK size..."
    $GRADLEW assembleRelease --analyze-size
}

check_dependencies() {
    log "Checking for dependency updates..."
    $GRADLEW dependencyUpdates -Drevision=release
}

# Main dispatcher
case "${1:-build}" in
    build)   build_debug ;;
    release) build_release ;;
    test)    run_tests ;;
    size)    analyze_size ;;
    deps)    check_dependencies ;;
    *)       echo "Usage: $0 {build|release|test|size|deps}" ;;
esac
```

### CI/CD Pipeline Script for Android

```bash
#!/usr/bin/env bash
# ci_android.sh — Runs on GitHub Actions / Bitrise / CircleCI
set -euo pipefail

BRANCH="${GITHUB_REF_NAME:-$(git rev-parse --abbrev-ref HEAD)}"
COMMIT="${GITHUB_SHA:-$(git rev-parse HEAD)}"
SHORT_COMMIT="${COMMIT:0:8}"
BUILD_NUM="${BUILD_NUMBER:-local}"

log() { echo ">>> $*"; }
section() { echo ""; echo "=== $* ==="; }

section "Environment"
log "Branch:  $BRANCH"
log "Commit:  $SHORT_COMMIT"
log "Build:   $BUILD_NUM"

section "Setup"
log "Java version: $(java -version 2>&1 | head -1)"
log "Android SDK: $(ls "$ANDROID_HOME/platforms" 2>/dev/null | tail -1 || echo 'not found')"

section "Tests"
./gradlew test --parallel --build-cache 2>&1 | tee test_output.log
UNIT_TEST_COUNT=$(grep -o "tests" test_output.log | wc -l || echo "0")
log "Unit tests: $UNIT_TEST_COUNT"

section "Static Analysis"
./gradlew lint || { log "WARN: Lint issues found (non-blocking)"; }
./gradlew detekt || true  # Non-blocking for CI

section "Build"
if [[ "$BRANCH" == "main" || "$BRANCH" == release/* ]]; then
    log "Building release AAB..."
    ./gradlew bundleRelease
    AAB_FILE=$(find . -name "*.aab" | head -1)
    log "AAB: $AAB_FILE ($(du -sh "$AAB_FILE" | cut -f1))"
else
    log "Building debug APK..."
    ./gradlew assembleDebug --build-cache
fi

section "Done"
log "Build $BUILD_NUM completed: $SHORT_COMMIT"
```

---

## 18. Java Spring Boot Recipes

### Project Lifecycle Scripts

```bash
#!/usr/bin/env bash
# springboot_manager.sh
set -euo pipefail

APP_NAME="${APP_NAME:-myapp}"
JAR_FILE="${JAR_FILE:-target/$APP_NAME.jar}"
PID_FILE="/tmp/$APP_NAME.pid"
LOG_FILE="/var/log/$APP_NAME/$APP_NAME.log"
PROFILE="${SPRING_PROFILES_ACTIVE:-dev}"

start() {
    if [ -f "$PID_FILE" ] && kill -0 "$(cat "$PID_FILE")" 2>/dev/null; then
        echo "Application is already running (PID: $(cat "$PID_FILE"))"
        return 1
    fi

    echo "Starting $APP_NAME with profile: $PROFILE"
    mkdir -p "$(dirname "$LOG_FILE")"

    nohup java \
        -jar "$JAR_FILE" \
        --spring.profiles.active="$PROFILE" \
        -Xms256m -Xmx512m \
        -XX:+UseG1GC \
        -XX:+HeapDumpOnOutOfMemoryError \
        -XX:HeapDumpPath="/tmp/$APP_NAME-heapdump.hprof" \
        > "$LOG_FILE" 2>&1 &

    echo $! > "$PID_FILE"
    echo "Started with PID: $(cat "$PID_FILE")"

    wait_for_health
}

stop() {
    if [ ! -f "$PID_FILE" ]; then
        echo "PID file not found. App may not be running."
        return 0
    fi

    local pid
    pid=$(cat "$PID_FILE")

    if ! kill -0 "$pid" 2>/dev/null; then
        echo "Process $pid not found. Removing stale PID file."
        rm -f "$PID_FILE"
        return 0
    fi

    echo "Stopping $APP_NAME (PID: $pid)..."
    kill -TERM "$pid"

    local timeout=30
    while kill -0 "$pid" 2>/dev/null && ((timeout-- > 0)); do
        sleep 1
    done

    if kill -0 "$pid" 2>/dev/null; then
        echo "Graceful stop failed; sending SIGKILL..."
        kill -9 "$pid"
    fi

    rm -f "$PID_FILE"
    echo "Stopped."
}

wait_for_health() {
    local max_wait="${1:-60}"
    local health_url="http://localhost:${SERVER_PORT:-8080}/actuator/health"
    local elapsed=0

    echo -n "Waiting for health check"
    while ! curl -sf "$health_url" > /dev/null 2>&1; do
        sleep 2
        ((elapsed += 2))
        echo -n "."
        if (( elapsed >= max_wait )); then
            echo ""
            echo "ERROR: Application failed to start within ${max_wait}s" >&2
            echo "Last 20 lines of log:" >&2
            tail -20 "$LOG_FILE" >&2
            return 1
        fi
    done
    echo ""
    echo "Application is healthy after ${elapsed}s"
}

status() {
    if [ -f "$PID_FILE" ] && kill -0 "$(cat "$PID_FILE")" 2>/dev/null; then
        local pid
        pid=$(cat "$PID_FILE")
        echo "$APP_NAME is running (PID: $pid)"
        local health
        health=$(curl -sf "http://localhost:${SERVER_PORT:-8080}/actuator/health" 2>/dev/null || echo '{"status":"UNKNOWN"}')
        echo "Health: $(echo "$health" | jq -r '.status' 2>/dev/null || echo 'UNKNOWN')"
    else
        echo "$APP_NAME is NOT running"
        return 1
    fi
}

case "${1:-help}" in
    start)   start ;;
    stop)    stop ;;
    restart) stop; start ;;
    status)  status ;;
    logs)    tail -f "$LOG_FILE" ;;
    *)       echo "Usage: $0 {start|stop|restart|status|logs}" ;;
esac
```

### Database Migration Scripts

```bash
#!/usr/bin/env bash
# db_migrate.sh — Runs Flyway / Liquibase and verifies DB state
set -euo pipefail

DB_HOST="${DB_HOST:?DB_HOST required}"
DB_PORT="${DB_PORT:-5432}"
DB_NAME="${DB_NAME:?DB_NAME required}"
DB_USER="${DB_USER:?DB_USER required}"
DB_PASS="${DB_PASS:?DB_PASS required}"

PSQL="psql postgresql://$DB_USER:$DB_PASS@$DB_HOST:$DB_PORT/$DB_NAME"

check_db_connectivity() {
    echo "Checking DB connectivity..."
    if ! $PSQL -c "SELECT 1" > /dev/null 2>&1; then
        echo "ERROR: Cannot connect to database" >&2
        return 1
    fi
    echo "Connected to $DB_NAME on $DB_HOST"
}

run_migrations() {
    echo "Running Flyway migrations..."
    ./mvnw flyway:migrate \
        -Dflyway.url="jdbc:postgresql://$DB_HOST:$DB_PORT/$DB_NAME" \
        -Dflyway.user="$DB_USER" \
        -Dflyway.password="$DB_PASS" \
        -Dflyway.locations="classpath:db/migration"

    echo "Migration status:"
    ./mvnw flyway:info -Dflyway.url="jdbc:postgresql://$DB_HOST:$DB_PORT/$DB_NAME" \
        -Dflyway.user="$DB_USER" -Dflyway.password="$DB_PASS"
}

backup_db() {
    local backup_file="backups/${DB_NAME}_$(date +%Y%m%d_%H%M%S).sql.gz"
    mkdir -p backups
    echo "Creating backup: $backup_file"
    PGPASSWORD="$DB_PASS" pg_dump -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" "$DB_NAME" \
        | gzip > "$backup_file"
    echo "Backup complete: $(du -sh "$backup_file" | cut -f1)"
}

check_db_connectivity
backup_db
run_migrations
```

### Log Analysis Scripts

```bash
#!/usr/bin/env bash
# analyze_logs.sh — Spring Boot log analysis

analyze() {
    local log_file="${1:-application.log}"
    [ -f "$log_file" ] || { echo "Log file not found: $log_file" >&2; exit 1; }

    echo "=== Log Analysis: $log_file ==="
    echo ""

    echo "--- Summary ---"
    echo "ERROR count:   $(grep -c " ERROR " "$log_file" || echo 0)"
    echo "WARN count:    $(grep -c " WARN  " "$log_file" || echo 0)"
    echo "Time range:    $(head -1 "$log_file" | awk '{print $1,$2}') — $(tail -1 "$log_file" | awk '{print $1,$2}')"

    echo ""
    echo "--- Top 10 Errors ---"
    grep " ERROR " "$log_file" | \
        awk '{$1=$2=$3=""; print}' | \
        sort | uniq -c | sort -rn | head -10

    echo ""
    echo "--- Slow Requests (>1s) ---"
    grep "completed in" "$log_file" | \
        awk '{match($0, /([0-9]+)ms/, arr); if (arr[1]+0 > 1000) print}' | \
        head -20

    echo ""
    echo "--- Exception Types ---"
    grep -oE "[A-Z][a-zA-Z]+Exception[^:]*" "$log_file" | \
        sort | uniq -c | sort -rn | head -10

    echo ""
    echo "--- Requests per Minute (last 10 mins) ---"
    tail -5000 "$log_file" | \
        grep "HTTP" | \
        awk '{print substr($2,1,5)}' | \
        sort | uniq -c
}

# Watch live logs for errors
watch_errors() {
    local log_file="${1:-application.log}"
    echo "Watching for errors in $log_file (Ctrl+C to stop)..."
    tail -f "$log_file" | grep --line-buffered " ERROR "
}

case "${1:-analyze}" in
    analyze) analyze "${2:-application.log}" ;;
    watch)   watch_errors "${2:-application.log}" ;;
    *)       echo "Usage: $0 {analyze|watch} [logfile]" ;;
esac
```

### Docker & Spring Boot

```bash
#!/usr/bin/env bash
# docker_springboot.sh
set -euo pipefail

APP_NAME="${APP_NAME:-myapp}"
IMAGE_NAME="${IMAGE_NAME:-$APP_NAME}"
IMAGE_TAG="${IMAGE_TAG:-$(git rev-parse --short HEAD)}"
REGISTRY="${DOCKER_REGISTRY:-}"
FULL_IMAGE="${REGISTRY:+$REGISTRY/}${IMAGE_NAME}:${IMAGE_TAG}"

build_image() {
    echo "Building Docker image: $FULL_IMAGE"
    docker build \
        --build-arg BUILD_DATE="$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
        --build-arg GIT_COMMIT="$(git rev-parse HEAD)" \
        --build-arg VERSION="$IMAGE_TAG" \
        -t "$FULL_IMAGE" \
        -t "${REGISTRY:+$REGISTRY/}${IMAGE_NAME}:latest" \
        .
    echo "Image size: $(docker image inspect "$FULL_IMAGE" --format='{{.Size}}' | numfmt --to=iec)"
}

push_image() {
    [ -n "$REGISTRY" ] || { echo "DOCKER_REGISTRY not set" >&2; exit 1; }
    echo "Pushing $FULL_IMAGE..."
    docker push "$FULL_IMAGE"
    docker push "${REGISTRY}/${IMAGE_NAME}:latest"
}

run_local() {
    echo "Running $APP_NAME locally..."
    docker run --rm \
        --name "$APP_NAME" \
        -p 8080:8080 \
        -e SPRING_PROFILES_ACTIVE=docker \
        -e DB_HOST=host.docker.internal \
        -e DB_PORT=5432 \
        -e DB_NAME="${DB_NAME:-myapp}" \
        "$FULL_IMAGE"
}

health_check() {
    local max_wait=30
    echo -n "Waiting for container health"
    for ((i=0; i<max_wait; i++)); do
        status=$(docker inspect --format='{{.State.Health.Status}}' "$APP_NAME" 2>/dev/null || echo "starting")
        case "$status" in
            healthy)  echo ""; echo "Healthy!"; return 0 ;;
            unhealthy) echo ""; echo "UNHEALTHY!" >&2; return 1 ;;
        esac
        echo -n "."
        sleep 1
    done
    echo ""; echo "Timeout waiting for health check" >&2; return 1
}

case "${1:-build}" in
    build)   build_image ;;
    push)    build_image && push_image ;;
    run)     run_local ;;
    health)  health_check ;;
    *)       echo "Usage: $0 {build|push|run|health}" ;;
esac
```

---

## 19. Cross-Platform Scripting (Mac vs Windows)

### Detecting the Platform

```bash
detect_platform() {
    case "$(uname -s)" in
        Darwin)    echo "macos" ;;
        Linux)     echo "linux" ;;
        MINGW*|MSYS*|CYGWIN*)  echo "windows" ;;
        *)         echo "unknown" ;;
    esac
}

PLATFORM=$(detect_platform)
echo "Running on: $PLATFORM"

# Architecture
ARCH=$(uname -m)   # x86_64 or arm64

# Conditional behavior
case "$PLATFORM" in
    macos)
        OPEN_CMD="open"
        SED_INPLACE="sed -i ''"
        CPU_COUNT=$(sysctl -n hw.ncpu)
        ;;
    linux)
        OPEN_CMD="xdg-open"
        SED_INPLACE="sed -i"
        CPU_COUNT=$(nproc)
        ;;
    windows)
        OPEN_CMD="start"
        SED_INPLACE="sed -i"
        CPU_COUNT="${NUMBER_OF_PROCESSORS:-4}"
        ;;
esac
```

### Path Differences

```bash
# Mac/Linux paths use /
JAVA_HOME="/Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home"

# Windows paths use \ and drive letters
# In Git Bash/WSL, Windows paths are mapped:
# Git Bash: C:\Users\Alice  →  /c/Users/Alice
# WSL2:     C:\Users\Alice  →  /mnt/c/Users/Alice

# Portable path function
normalize_path() {
    local path="$1"
    case "$(detect_platform)" in
        windows)
            # Convert /c/... to C:/... for some Windows tools
            echo "$path" | sed 's|^/\([a-z]\)/|\u\1:/|'
            ;;
        *)
            echo "$path"
            ;;
    esac
}

# JAVA_HOME on different platforms
set_java_home() {
    case "$(detect_platform)" in
        macos)
            JAVA_HOME=$(/usr/libexec/java_home -v 21 2>/dev/null || echo "")
            ;;
        linux)
            JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
            ;;
        windows)
            # In Git Bash/WSL
            JAVA_HOME="${JAVA_HOME:-/c/Program Files/Eclipse Adoptium/jdk-21}"
            ;;
    esac
    export JAVA_HOME
}
```

### sed Differences: Mac vs GNU

```bash
# This is the most common cross-platform gotcha:

# GNU sed (Linux/WSL): in-place edit with no backup
sed -i 's/foo/bar/g' file.txt

# BSD sed (macOS): requires empty string for no backup
sed -i '' 's/foo/bar/g' file.txt

# Cross-platform solution:
sed_inplace() {
    if sed --version 2>/dev/null | grep -q GNU; then
        sed -i "$@"          # GNU sed
    else
        sed -i '' "$@"       # BSD sed (macOS)
    fi
}

sed_inplace 's/localhost/127.0.0.1/g' config.properties

# Alternative: always use perl for in-place (works everywhere)
perl -pi -e 's/foo/bar/g' file.txt
```

### date Command Differences

```bash
# GNU date (Linux/WSL) vs BSD date (macOS)

# Get yesterday's date:
# GNU:
yesterday=$(date -d "yesterday" +%Y-%m-%d)
# BSD:
yesterday=$(date -v-1d +%Y-%m-%d)

# Portable yesterday:
yesterday() {
    if date --version 2>/dev/null | grep -q GNU; then
        date -d "yesterday" +%Y-%m-%d
    else
        date -v-1d +%Y-%m-%d
    fi
}

# Add N days:
# GNU: date -d "+7 days" +%Y-%m-%d
# BSD: date -v+7d +%Y-%m-%d

# Epoch time:
# GNU: date -d "2026-01-01" +%s
# BSD: date -j -f "%Y-%m-%d" "2026-01-01" +%s

# Cross-platform: use Python when date differences arise
python3 -c "from datetime import date, timedelta; print(date.today() - timedelta(days=1))"
```

### WSL2 Specific Tips

```bash
# Run Windows executables from WSL
/mnt/c/Windows/System32/notepad.exe
explorer.exe .    # Open Windows Explorer in current WSL directory
cmd.exe /c dir    # Run Windows cmd command

# Access Windows environment variables
windows_user=$(cmd.exe /c "echo %USERNAME%" 2>/dev/null | tr -d '\r')

# Open URLs in Windows browser from WSL
open_url() {
    local url="$1"
    if grep -qi microsoft /proc/version 2>/dev/null; then
        # We're in WSL
        powershell.exe -c "Start-Process '$url'" 2>/dev/null || \
        cmd.exe /c "start $url" 2>/dev/null
    else
        xdg-open "$url" 2>/dev/null || open "$url" 2>/dev/null || true
    fi
}

# Forward ports from WSL to Windows (done automatically in WSL2)
# Access http://localhost:8080 from Windows when running in WSL2

# Run Android emulator from WSL2:
# Currently not supported — emulators need GPU access via hardware virtualization.
# Workaround: Install Android Studio on Windows; use ADB from WSL2 with:
export ADB_SERVER_SOCKET=tcp:127.0.0.1:5037
adb devices   # Connects to Windows ADB server
```

### Windows-Aware Script Template

```bash
#!/usr/bin/env bash
# cross_platform_template.sh

set -euo pipefail

# ---- Platform detection ----
OS="unknown"
case "$(uname -s)" in
    Darwin)    OS="macos" ;;
    Linux)
        if grep -qi microsoft /proc/version 2>/dev/null; then
            OS="wsl"
        else
            OS="linux"
        fi
        ;;
    MINGW*|MSYS*|CYGWIN*) OS="gitbash" ;;
esac

readonly OS
echo "Platform: $OS"

# ---- Tool availability check ----
require_tools() {
    local missing=()
    for tool in "$@"; do
        command -v "$tool" > /dev/null 2>&1 || missing+=("$tool")
    done
    if (( ${#missing[@]} > 0 )); then
        echo "ERROR: Missing required tools: ${missing[*]}" >&2
        echo "Install with:"
        case "$OS" in
            macos)   echo "  brew install ${missing[*]}" ;;
            linux|wsl) echo "  sudo apt install ${missing[*]}" ;;
            gitbash) echo "  Install manually or use choco/scoop" ;;
        esac
        exit 1
    fi
}

require_tools java git curl jq

# ---- Rest of script ----
```

---

## 20. Security Best Practices

### Secure Scripting Fundamentals

```bash
# 1. NEVER hardcode secrets in scripts
# BAD:
DB_PASS="mysecretpassword"

# GOOD: Read from environment
DB_PASS="${DB_PASS:?DB_PASS environment variable is required}"

# GOOD: Read from a secrets file (with strict permissions)
# chmod 600 ~/.secrets/myapp.env && source ~/.secrets/myapp.env

# GOOD: Read from macOS Keychain
DB_PASS=$(security find-generic-password -a "$USER" -s "myapp-db" -w)

# GOOD: Read from secret manager
DB_PASS=$(aws secretsmanager get-secret-value --secret-id myapp/db --query SecretString --output text | jq -r .password)
```

```bash
# 2. Validate and sanitize all inputs

# Never use raw user input in eval or command substitution
# BAD:
user_input="$1"
eval "$user_input"   # DANGEROUS: arbitrary code execution!

# ALSO BAD:
filename="$1"
rm "$filename"   # What if user passes "-rf /"?

# GOOD: Validate input against whitelist
validate_filename() {
    local filename="$1"
    # Only allow alphanumeric, dash, underscore, dot
    if [[ ! "$filename" =~ ^[A-Za-z0-9._-]+$ ]]; then
        echo "Invalid filename: $filename" >&2
        return 1
    fi
    echo "$filename"
}

# GOOD: Use -- to separate options from arguments
rm -- "$user_filename"   # Prevents -rf style attacks

# GOOD: Use absolute paths for security-critical operations
/bin/rm -- "$filename"   # Use absolute path to prevent PATH hijacking
```

```bash
# 3. Secure temp files

# BAD: predictable temp file name
tmpfile="/tmp/myapp.tmp"   # Race condition! Attacker can create symlink

# GOOD: mktemp creates with random suffix and secure permissions
tmpfile=$(mktemp)
trap 'rm -f "$tmpfile"' EXIT

# Even better: restrict to owner only
tmpfile=$(mktemp)
chmod 600 "$tmpfile"
trap 'rm -f "$tmpfile"' EXIT
```

```bash
# 4. Restrict script permissions
chmod 700 sensitive_script.sh   # Only owner can read, write, execute
chmod 500 deployed_script.sh    # Owner read+execute only; no write

# 5. Use strict mode
set -euo pipefail
IFS=$'\n\t'

# 6. Avoid world-writable files
umask 027   # New files: owner rw, group r, others nothing
            # New dirs: owner rwx, group rx, others nothing

# 7. Check file permissions before reading sensitive files
check_permissions() {
    local file="$1"
    local expected_perm="${2:-600}"
    local actual_perm
    actual_perm=$(stat -c "%a" "$file" 2>/dev/null || stat -f "%Lp" "$file")

    if [ "$actual_perm" != "$expected_perm" ]; then
        echo "WARNING: $file has permissions $actual_perm (expected $expected_perm)" >&2
    fi
}

check_permissions ~/.ssh/id_rsa 600
check_permissions .env 600
```

### Secrets in CI/CD

```bash
# GitHub Actions: secrets are injected as environment variables
# Reference: ${{ secrets.MY_SECRET }}  →  $MY_SECRET in bash

# Never print secrets
echo "DB_PASS is: $DB_PASS"     # BAD: secret in CI logs!
echo "DB_PASS is set: ${DB_PASS:+yes}"  # GOOD: just confirm it's set

# Mask output when unavoidable (GitHub Actions)
echo "::add-mask::$DB_PASS"     # GitHub Actions: redacts from all log output

# Zero out variables after use
use_secret() {
    local secret="$1"
    # ... use secret ...
    secret=""   # Clear from memory
    unset secret
}
```

---

## 21. Performance & Optimization

### Profiling Scripts

```bash
# Time a command
time ./build.sh
time (./step1.sh && ./step2.sh)

# Detailed timing with bash built-in
TIMEFORMAT='Real: %Rs, User: %Us, Sys: %Ss'
time {
    sleep 0.1
    echo "done"
}

# Profile line-by-line (PS4 + xtrace)
PS4='$(date +%s%N) '    # Nanosecond timestamp
exec 3>&2 2>/tmp/trace.log
set -x
# ... your code ...
set +x
exec 2>&3
# Analyze: awk 'NR>1 {print $1 - prev, $0; prev=$1}' /tmp/trace.log | sort -rn | head
```

### Optimization Techniques

```bash
# 1. Avoid unnecessary subshells
# BAD: two subshells
count=$(echo "$text" | wc -w)
# GOOD: one subshell
count=$(wc -w <<< "$text")
# BEST: zero subshells (for string length)
count="${#text}"  # No subshell needed for length

# 2. Use built-in string manipulation instead of external commands
# BAD: calls external grep/sed
filename=$(echo "$path" | sed 's|.*/||')
# GOOD: pure bash
filename="${path##*/}"

# 3. Read files with redirection, not cat
# BAD: useless use of cat
while read -r line; do ... done < <(cat file.txt)
# GOOD: direct redirection
while IFS= read -r line; do ... done < file.txt

# 4. Avoid fork-heavy constructs in loops
# BAD: calls date on every iteration
for i in {1..1000}; do
    echo "$(date): item $i"
done

# GOOD: batch the external calls
start_time=$(date +%s)
for i in {1..1000}; do
    echo "$start_time: item $i"
done

# 5. Use arrays instead of repeated string operations
# BAD: string concatenation
args=""
for item in "${items[@]}"; do
    args="$args --item $item"
done
eval "command $args"   # Also dangerous!

# GOOD: array
declare -a args=()
for item in "${items[@]}"; do
    args+=(--item "$item")
done
command "${args[@]}"   # Safe, no eval needed

# 6. Parallelize independent tasks
run_parallel_with_limit() {
    local max_jobs="$1"; shift
    local -a pids=()

    for cmd in "$@"; do
        while (( $(jobs -rp | wc -l) >= max_jobs )); do sleep 0.1; done
        eval "$cmd" &
        pids+=($!)
    done

    for pid in "${pids[@]}"; do wait "$pid"; done
}
```

---

## 22. Real-World Project Scripts

### Complete Deployment Script

```bash
#!/usr/bin/env bash
# deploy.sh — Full deployment pipeline for Spring Boot app
set -euo pipefail
IFS=$'\n\t'

# ============================================================
# Configuration
# ============================================================
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly APP_NAME="myapp"
readonly DEPLOY_DIR="/opt/$APP_NAME"
readonly LOG_DIR="/var/log/$APP_NAME"
readonly CONFIG_DIR="/etc/$APP_NAME"

ENVIRONMENT="${DEPLOY_ENV:-staging}"
VERSION="${1:-}"
DRY_RUN="${DRY_RUN:-false}"
ROLLBACK=false

# ============================================================
# Logging
# ============================================================
LOG_FILE="/tmp/deploy_$(date +%Y%m%d_%H%M%S).log"
log()   { echo "[$(date +%H:%M:%S)] [INFO]  $*" | tee -a "$LOG_FILE"; }
warn()  { echo "[$(date +%H:%M:%S)] [WARN]  $*" | tee -a "$LOG_FILE" >&2; }
error() { echo "[$(date +%H:%M:%S)] [ERROR] $*" | tee -a "$LOG_FILE" >&2; }
die()   { error "$*"; exit 1; }

run() {
    if [[ "$DRY_RUN" == "true" ]]; then
        log "[DRY-RUN] $*"
    else
        log "Running: $*"
        "$@"
    fi
}

# ============================================================
# Pre-flight Checks
# ============================================================
preflight() {
    log "=== Pre-flight Checks ==="

    [[ -n "$VERSION" ]] || die "Usage: $0 <version>"
    [[ "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || die "Invalid version: $VERSION"
    [[ "$ENVIRONMENT" =~ ^(dev|staging|prod)$ ]] || die "Invalid environment: $ENVIRONMENT"

    command -v java > /dev/null || die "Java not found"
    command -v curl > /dev/null || die "curl not found"

    java_version=$(java -version 2>&1 | head -1 | awk -F'"' '{print $2}' | cut -d. -f1)
    (( java_version >= 21 )) || die "Java 21+ required (found: $java_version)"

    log "  Version:     $VERSION"
    log "  Environment: $ENVIRONMENT"
    log "  Java:        $java_version"
    log "Pre-flight passed"
}

# ============================================================
# Download Artifact
# ============================================================
download_artifact() {
    log "=== Downloading Artifact ==="
    local artifact_url="${ARTIFACT_URL:?ARTIFACT_URL required}"
    local artifact_path="/tmp/$APP_NAME-$VERSION.jar"

    if [[ -f "$artifact_path" ]]; then
        log "Using cached artifact: $artifact_path"
    else
        log "Downloading $artifact_url..."
        run curl -fL --retry 3 "$artifact_url/$VERSION/$APP_NAME-$VERSION.jar" \
            -o "$artifact_path"
    fi

    # Verify checksum if available
    if curl -sf "$artifact_url/$VERSION/$APP_NAME-$VERSION.jar.sha256" -o /tmp/expected.sha256 2>/dev/null; then
        log "Verifying checksum..."
        run sha256sum -c /tmp/expected.sha256 ||
            sha256sum -c /tmp/expected.sha256 2>/dev/null ||
            log "WARN: Checksum verification not available on this platform"
    fi

    echo "$artifact_path"
}

# ============================================================
# Backup Current Deployment
# ============================================================
backup_current() {
    log "=== Backing Up Current Deployment ==="
    local backup_dir="/opt/backups/$APP_NAME/$(date +%Y%m%d_%H%M%S)"

    if [[ -d "$DEPLOY_DIR" ]]; then
        run mkdir -p "$backup_dir"
        run cp -r "$DEPLOY_DIR" "$backup_dir"
        log "Backup created: $backup_dir"
        echo "$backup_dir"
    else
        log "No current deployment to back up"
        echo ""
    fi
}

# ============================================================
# Deploy
# ============================================================
deploy() {
    local jar_file="$1"
    log "=== Deploying $VERSION to $ENVIRONMENT ==="

    # Stop current app
    if systemctl is-active --quiet "$APP_NAME" 2>/dev/null; then
        log "Stopping current instance..."
        run systemctl stop "$APP_NAME"
    fi

    # Deploy files
    run mkdir -p "$DEPLOY_DIR" "$LOG_DIR" "$CONFIG_DIR"
    run cp "$jar_file" "$DEPLOY_DIR/$APP_NAME.jar"
    run chmod 500 "$DEPLOY_DIR/$APP_NAME.jar"

    # Generate environment-specific config
    run cat > "$CONFIG_DIR/application.yml" << EOF
spring:
  profiles:
    active: $ENVIRONMENT
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT:-5432}/${DB_NAME}
server:
  port: ${SERVER_PORT:-8080}
EOF

    # Start app
    log "Starting $APP_NAME..."
    run systemctl start "$APP_NAME"

    # Wait for health
    wait_for_health "${SERVER_PORT:-8080}" 60
    log "Deployment successful: $VERSION"
}

# ============================================================
# Health Check
# ============================================================
wait_for_health() {
    local port="${1:-8080}"
    local timeout="${2:-60}"
    local url="http://localhost:$port/actuator/health"
    local elapsed=0

    log "Waiting for health check at $url..."
    while (( elapsed < timeout )); do
        if curl -sf "$url" | grep -q '"status":"UP"'; then
            log "Health check passed after ${elapsed}s"
            return 0
        fi
        sleep 2
        (( elapsed += 2 ))
    done

    error "Health check timed out after ${timeout}s"
    error "Recent logs:"
    tail -30 "$LOG_DIR/$APP_NAME.log" >&2
    return 1
}

# ============================================================
# Rollback
# ============================================================
rollback() {
    local backup_dir="${1:?Backup dir required}"
    warn "=== Rolling Back ==="
    run systemctl stop "$APP_NAME" || true
    run cp -r "$backup_dir/$APP_NAME"/* "$DEPLOY_DIR/"
    run systemctl start "$APP_NAME"
    wait_for_health
    warn "Rollback complete"
}

# ============================================================
# Main
# ============================================================
main() {
    log "Deploy script started: $APP_NAME v$VERSION → $ENVIRONMENT"
    log "Log file: $LOG_FILE"

    preflight
    local artifact
    artifact=$(download_artifact)
    local backup_dir
    backup_dir=$(backup_current)

    if ! deploy "$artifact"; then
        error "Deployment FAILED"
        if [[ -n "$backup_dir" ]]; then
            warn "Initiating automatic rollback..."
            rollback "$backup_dir"
        fi
        die "Deployment failed. Check $LOG_FILE for details."
    fi

    log "=== Deployment Complete ==="
    log "Version $VERSION is live on $ENVIRONMENT"
}

main "$@"
```

### Git Workflow Automation

```bash
#!/usr/bin/env bash
# git_workflow.sh — Automates common git tasks for feature branching

set -euo pipefail

MAIN_BRANCH="${MAIN_BRANCH:-main}"
FEATURE_PREFIX="feature/"
HOTFIX_PREFIX="hotfix/"
RELEASE_PREFIX="release/"

# Start a feature branch
feature_start() {
    local name="${1:?Feature name required}"
    local branch="$FEATURE_PREFIX$(echo "$name" | tr ' ' '-' | tr '[:upper:]' '[:lower:]')"

    git fetch origin
    git checkout "$MAIN_BRANCH"
    git pull origin "$MAIN_BRANCH"
    git checkout -b "$branch"
    echo "Created: $branch"
}

# Finish a feature branch (creates PR-ready state)
feature_finish() {
    local branch
    branch=$(git rev-parse --abbrev-ref HEAD)
    [[ "$branch" == "$FEATURE_PREFIX"* ]] || { echo "Not on a feature branch" >&2; exit 1; }

    # Rebase onto latest main
    git fetch origin
    git rebase "origin/$MAIN_BRANCH"

    # Push
    git push -u origin "$branch"
    echo "Feature branch ready for PR: $branch"
    echo "Create PR: https://github.com/$(git remote get-url origin | sed 's/.*github.com[/:]//' | sed 's/.git$//')/compare/$branch"
}

# Create a release branch
release_start() {
    local version="${1:?Version required}"
    local branch="$RELEASE_PREFIX$version"

    git fetch origin
    git checkout "$MAIN_BRANCH"
    git pull origin "$MAIN_BRANCH"
    git checkout -b "$branch"

    # Bump version in build files
    if [ -f "pom.xml" ]; then
        ./mvnw versions:set -DnewVersion="$version" -DgenerateBackupPoms=false
    elif [ -f "build.gradle.kts" ]; then
        sed -i.bak "s/version = \".*\"/version = \"$version\"/" build.gradle.kts && rm build.gradle.kts.bak
    fi

    git add -A
    git commit -m "chore: bump version to $version"
    git push -u origin "$branch"
    echo "Release branch created: $branch"
}

# Clean merged branches
clean_branches() {
    echo "Fetching and pruning remote tracking branches..."
    git fetch --prune

    echo "Local branches merged into $MAIN_BRANCH:"
    git branch --merged "$MAIN_BRANCH" | grep -v "^\*" | grep -v "$MAIN_BRANCH"

    read -r -p "Delete these branches? [y/N] " confirm
    if [[ "$confirm" =~ ^[Yy]$ ]]; then
        git branch --merged "$MAIN_BRANCH" | grep -v "^\*" | grep -v "$MAIN_BRANCH" | xargs -r git branch -d
        echo "Deleted merged branches"
    fi
}

case "${1:-help}" in
    feature-start)   feature_start "${2:-}" ;;
    feature-finish)  feature_finish ;;
    release-start)   release_start "${2:-}" ;;
    clean)           clean_branches ;;
    *)
        echo "Usage: $0 {feature-start <name>|feature-finish|release-start <version>|clean}"
        ;;
esac
```

---

## 23. Quick Reference Cheatsheet

### Variable Operations

```bash
${var}              # Use variable
${var:-default}     # Use default if unset/empty
${var:=default}     # Set and use default if unset/empty
${var:?message}     # Error and exit if unset/empty
${var:+other}       # Use other if var IS set
${#var}             # Length of string
${var:offset:len}   # Substring
${var/old/new}      # Replace first occurrence
${var//old/new}     # Replace all occurrences
${var#prefix}       # Remove shortest prefix match
${var##prefix}      # Remove longest prefix match
${var%suffix}       # Remove shortest suffix match
${var%%suffix}      # Remove longest suffix match
${var^^}            # Uppercase
${var,,}            # Lowercase
```

### File Tests

```bash
-e  exists          -f  regular file    -d  directory
-L  symlink         -r  readable        -w  writable
-x  executable      -s  non-empty       -z  empty (zero size)
-nt newer than      -ot older than      -ef same file (inode)
```

### Arithmetic

```bash
$((a + b))    $((a - b))    $((a * b))    $((a / b))
$((a % b))    $((a ** b))   $((a++))      $((a--))
((a > b))     ((a == b))    ((a != b))
```

### String Tests

```bash
[ -z "$s" ]         # empty
[ -n "$s" ]         # non-empty
[ "$a" = "$b" ]     # equal
[ "$a" != "$b" ]    # not equal
[[ "$s" =~ regex ]] # regex match (bash only)
[[ "$s" == glob* ]] # glob match (bash only)
```

### Exit Codes & Boolean Logic

```bash
cmd1 && cmd2    # Run cmd2 only if cmd1 succeeded
cmd1 || cmd2    # Run cmd2 only if cmd1 failed
! cmd1          # Negate exit code
true            # Always exits 0
false           # Always exits 1
$?              # Exit code of last command
```

### Redirections

```bash
> file      # Redirect stdout (overwrite)
>> file     # Redirect stdout (append)
< file      # Redirect stdin
2> file     # Redirect stderr
2>&1        # Redirect stderr to stdout
&> file     # Redirect both stdout+stderr (bash)
| cmd       # Pipe stdout to next command
tee file    # Split stdout to file and terminal
/dev/null   # Discard output
```

### Useful One-Liners for Developers

```bash
# Find all TODOs in Java/Kotlin files
grep -rn "TODO\|FIXME\|HACK" --include="*.java" --include="*.kt" .

# Count lines of code per language
find . -name "*.java" | xargs wc -l | tail -1

# Find the largest files in project
find . -type f -printf '%s %p\n' | sort -rn | head -20

# Watch a port for connections
watch -n 1 "ss -tlnp | grep 8080"  # Linux
watch -n 1 "lsof -i :8080"          # Mac

# Kill process on a port
lsof -ti :8080 | xargs kill         # Mac
fuser -k 8080/tcp                   # Linux

# Follow Spring Boot logs, highlight errors
tail -f application.log | grep --color=auto -E "ERROR|WARN|$"

# Quick HTTP server for testing
python3 -m http.server 8000

# Check SSL certificate expiry
echo | openssl s_client -connect api.example.com:443 2>/dev/null \
    | openssl x509 -noout -dates

# Monitor disk usage every 5 seconds
watch -n 5 df -h

# See which process is using the most memory
ps aux --sort=-%mem | head -10     # Linux
ps aux -m | head -10               # Mac

# Generate a random password
openssl rand -base64 32

# Encode/decode base64
echo "Hello World" | base64
echo "SGVsbG8gV29ybGQ=" | base64 --decode

# Check open ports
ss -tlnp          # Linux
netstat -an | grep LISTEN  # Mac/Cross-platform
```

---

## Further Learning

### Essential Tools to Install

```bash
# Mac
brew install bash jq yq ripgrep fd bat tree htop watch shellcheck shfmt

# Linux/WSL
sudo apt install -y jq ripgrep fd-find bat tree htop watch shellcheck

# ShellCheck: static analysis for shell scripts (catches most bugs)
shellcheck myscript.sh

# shfmt: auto-format shell scripts
shfmt -w -i 4 myscript.sh
```

### Learning Progression

| Level | Topics to Master |
|-------|-----------------|
| Beginner | Navigation, file ops, pipes, variables, if/for/while |
| Intermediate | Functions, arrays, error handling (set -euo pipefail), trap, getopts |
| Advanced | Process substitution, coprocesses, associative arrays, parallel execution, IPC |
| Expert | Performance profiling, signal handling, portable POSIX scripting, security hardening |

### Key References

- [Bash Manual](https://www.gnu.org/software/bash/manual/) — Official GNU Bash manual
- [ShellCheck](https://www.shellcheck.net/) — Online shell script linter
- [Bash Pitfalls](https://mywiki.wooledge.org/BashPitfalls) — Common mistakes and how to avoid them
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html) — Industry style reference
- [Bash Hackers Wiki](https://wiki.bash-hackers.org/) — Deep dive into bash features

---

*Guide version: 2026-06 | Targets: bash 5.x, macOS (zsh-compatible), WSL2 Ubuntu*
