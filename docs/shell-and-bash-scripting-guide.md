# Shell & Bash Scripting: Zero to Advanced

**For Android & Java Spring Boot developers on macOS and Windows**

This guide takes you from your first terminal command to production-grade shell scripts you can use in local dev, CI/CD, and deployment workflows.

---

## Table of Contents

1. [Why Shell Scripting Matters for You](#1-why-shell-scripting-matters-for-you)
2. [Your Terminal Setup: Mac vs Windows](#2-your-terminal-setup-mac-vs-windows)
3. [Shell Basics: The First Hour](#3-shell-basics-the-first-hour)
4. [Navigation, Files & Permissions](#4-navigation-files--permissions)
5. [Pipes, Redirection & Data Flow](#5-pipes-redirection--data-flow)
6. [Text Processing Essentials](#6-text-processing-essentials)
7. [Variables, Quoting & Expansion](#7-variables-quoting--expansion)
8. [Control Flow](#8-control-flow)
9. [Functions & Modular Scripts](#9-functions--modular-scripts)
10. [Exit Codes, Error Handling & Debugging](#10-exit-codes-error-handling--debugging)
11. [Environment, PATH & Shell Config](#11-environment-path--shell-config)
12. [Intermediate Patterns Every Developer Needs](#12-intermediate-patterns-every-developer-needs)
13. [Advanced Bash Features](#13-advanced-bash-features)
14. [Process Management & Job Control](#14-process-management--job-control)
15. [sed, awk & xargs in Depth](#15-sed-awk--xargs-in-depth)
16. [Writing Portable Scripts (Mac, Linux, Windows)](#16-writing-portable-scripts-mac-linux-windows)
17. [Android Developer Workflows](#17-android-developer-workflows)
18. [Spring Boot & Java Developer Workflows](#18-spring-boot--java-developer-workflows)
19. [CI/CD Scripting Patterns](#19-cicd-scripting-patterns)
20. [Security & Best Practices](#20-security--best-practices)
21. [Quick Reference Cheat Sheet](#21-quick-reference-cheat-sheet)
22. [Learning Path & Next Steps](#22-learning-path--next-steps)

---

## 1. Why Shell Scripting Matters for You

As an Android and Spring Boot developer, you already touch the shell constantly — you just may not think of it as "scripting":

| Task | Shell involvement |
|------|-------------------|
| `./gradlew assembleDebug` | Runs a Gradle wrapper shell script |
| `./mvnw spring-boot:run` | Maven wrapper shell script |
| `adb devices` | Android Debug Bridge CLI |
| `git commit`, `git push` | Git is a shell-first tool |
| Docker, Kubernetes, Jenkins, GitHub Actions | All orchestrated via shell |
| FastAPI of local env setup | One script replaces 20 manual steps |

**Goal of this guide:** turn unconscious shell usage into deliberate, reliable automation.

---

## 2. Your Terminal Setup: Mac vs Windows

### macOS

| Item | Details |
|------|---------|
| Default shell (macOS Catalina+) | **zsh** — Bash-compatible for most daily use |
| Terminal apps | Terminal.app, iTerm2, Warp, VS Code / Cursor integrated terminal |
| Package manager | Homebrew (`brew install ...`) |
| Config files | `~/.zshrc` (zsh), `~/.bash_profile` or `~/.bashrc` (bash) |

To use Bash explicitly on Mac:

```bash
# Check current shell
echo $SHELL

# Switch to bash (if installed)
chsh -s /bin/bash

# Or just run bash in current session
bash
```

Install modern Bash (macOS ships an old GPLv2 Bash 3.2):

```bash
brew install bash
# Add to /etc/shells, then chsh -s /opt/homebrew/bin/bash
```

### Windows

Windows has **three** common environments for Bash-style work:

| Environment | What it is | Best for |
|-------------|-----------|----------|
| **Git Bash** | Bash bundled with Git for Windows | Quick git/gradle tasks, lightweight scripts |
| **WSL2** (Windows Subsystem for Linux) | Full Linux kernel in a VM | Serious dev: Docker, Android emulator, Linux-native tooling |
| **PowerShell** | Native Windows shell | Windows admin, some cross-platform scripts |

**Recommendation for Android + Spring Boot on Windows:**

1. Install **WSL2** with Ubuntu for primary development
2. Keep **Git Bash** for quick tasks in native Windows repos
3. Use **PowerShell** only when you need native Windows APIs

```powershell
# Install WSL2 (run in PowerShell as Administrator)
wsl --install -d Ubuntu
```

Inside WSL, your Windows drives appear at `/mnt/c/`, `/mnt/d/`, etc.

### Path Differences (Critical)

| Concept | macOS / Linux | Windows (native) | WSL |
|---------|---------------|------------------|-----|
| Home directory | `/Users/you` | `C:\Users\you` | `/home/you` |
| Path separator | `/` | `\` (use `/` in Bash) | `/` |
| Env var syntax | `$HOME`, `$PATH` | `%USERPROFILE%`, `%PATH%` | `$HOME`, `$PATH` |
| Line endings | LF (`\n`) | CRLF (`\r\n`) | LF (`\n`) |

**Always configure Git line endings:**

```bash
git config --global core.autocrlf input   # Mac/Linux
git config --global core.autocrlf true    # Windows (Git Bash)
```

---

## 3. Shell Basics: The First Hour

### What Is a Shell?

A **shell** is a program that reads your commands and executes them. **Bash** (Bourne Again Shell) is the most common scripting shell on macOS/Linux/WSL.

```
You type → Shell parses → Forks process → Executes program → Returns exit code
```

### Command Structure

```bash
command [options] [arguments]
```

Examples:

```bash
ls -la /tmp
git status
./gradlew build --stacktrace
```

- **Options** (flags): usually `-x` (short) or `--long-form`
- **Arguments**: files, URLs, subcommands
- Order often matters: `cp source dest`, not `cp dest source`

### Getting Help

```bash
man ls              # Manual page (press q to quit)
ls --help           # Quick help (GNU tools)
help cd             # Bash built-ins only
type ls             # Is it built-in, alias, or external?
which java          # Path to executable
```

### Built-ins vs External Commands

```bash
type echo    # built-in
type ls      # external: /bin/ls
type cd      # built-in
```

Built-ins run inside the shell process (faster). External commands spawn a child process.

### Command History

```bash
history              # List past commands
!!                   # Repeat last command
!42                  # Run command #42 from history
Ctrl+R               # Reverse search history
↑ / ↓                # Navigate history
```

### Tab Completion

Press **Tab** to autocomplete files, directories, commands, and (with bash-completion) git branches, docker images, etc.

```bash
brew install bash-completion   # Mac
sudo apt install bash-completion  # Ubuntu/WSL
```

---

## 4. Navigation, Files & Permissions

### Essential Navigation

```bash
pwd                  # Print working directory
cd ~/projects        # Change directory (~ = home)
cd -                 # Previous directory
cd ..                # Parent directory
ls -lah              # List all, human-readable sizes
tree -L 2            # Directory tree (install: brew install tree)
```

### Creating & Manipulating Files

```bash
touch app.log                    # Create empty file / update timestamp
mkdir -p src/main/java/com/app  # Create nested dirs (-p = parents)
cp -r source/ dest/              # Copy recursively
mv old.txt new.txt               # Move or rename
rm file.txt                      # Delete file
rm -rf build/                    # Delete directory (DANGEROUS — no undo)
```

**Safety tip:** alias `rm` to ask for confirmation during learning:

```bash
alias rm='rm -i'
```

### Reading Files

```bash
cat file.txt           # Dump entire file
less file.txt          # Paginated (q to quit)
head -n 20 file.txt    # First 20 lines
tail -n 50 app.log     # Last 50 lines
tail -f app.log        # Follow log in real time (essential for Spring Boot)
```

### File Permissions (Unix)

```bash
ls -l script.sh
# -rwxr-xr--  1 user  staff  1234 Jun 23 10:00 script.sh
#  │││││││││
#  │└┴┴ owner: rwx
#   └┴┴ group: r-x
#    └┴ other: r--
```

```bash
chmod +x deploy.sh     # Make executable
chmod 755 deploy.sh    # rwxr-xr-x
./deploy.sh            # Run script in current directory
```

**You must use `./`** — the current directory is not in `$PATH` by design (security).

### Finding Files

```bash
find . -name "*.java" -type f
find . -name "build.gradle*" -maxdepth 3
find . -mtime -1 -name "*.log"    # Modified in last 24h

# Modern alternative (faster, respects .gitignore)
brew install fd     # Mac
fd "\.kt$"
```

---

## 5. Pipes, Redirection & Data Flow

The Unix philosophy: small tools chained together.

### Redirection

```bash
command > file.txt       # stdout to file (overwrite)
command >> file.txt      # stdout to file (append)
command 2> errors.log     # stderr to file
command &> all.log        # stdout + stderr
command < input.txt       # read stdin from file
```

### Pipes

```bash
command1 | command2 | command3
```

Data flows left to right via **stdout → stdin**.

```bash
# Count Java files in project
find . -name "*.java" | wc -l

# Find Spring Boot apps with "UserController"
grep -r "UserController" --include="*.java" .

# Top 10 memory-consuming processes
ps aux | sort -k4 -rn | head -10
```

### Here Documents & Here Strings

```bash
# Here document — multi-line input
cat << 'EOF' > config.env
SPRING_PROFILES_ACTIVE=dev
SERVER_PORT=8080
EOF

# Here string — single line
grep "ERROR" <<< "$log_content"
```

The quotes around `'EOF'` prevent variable expansion inside the heredoc.

---

## 6. Text Processing Essentials

### grep — Search Text

```bash
grep "Exception" app.log
grep -i "error" app.log           # Case insensitive
grep -r "TODO" src/               # Recursive
grep -n "fun main" *.kt           # With line numbers
grep -v "DEBUG" app.log           # Invert match (exclude)
grep -E "ERROR|WARN" app.log      # Extended regex
grep -c "BUILD SUCCESS" build.log # Count matches
```

### cut, sort, uniq

```bash
# Extract 1st column from CSV
cut -d',' -f1 users.csv

# Sort and deduplicate
sort names.txt | uniq
sort names.txt | uniq -c          # With counts
```

### wc — Word Count

```bash
wc -l *.java    # Line counts
```

### diff & comm

```bash
diff build.gradle build.gradle.kts
comm -23 <(sort a.txt) <(sort b.txt)   # Lines only in a.txt
```

---

## 7. Variables, Quoting & Expansion

### Defining Variables

```bash
name="Android Dev"
port=8080
readonly API_KEY="secret"    # Cannot be changed

# Convention: UPPER_CASE for env/constants, lower_case for locals
```

**No spaces around `=`** — `name = "x"` is wrong.

### Using Variables

```bash
echo $name
echo ${name}              # Preferred — explicit boundaries
echo "${name}_suffix"     # Required when concatenating
echo $port
```

### Quoting Rules (Memorize These)

| Quote | Behavior |
|-------|----------|
| `'single'` | Literal — no expansion at all |
| `"double"` | Expands `$vars` and `$(commands)`, but not all escapes |
| `` `backtick` `` | Legacy command substitution — avoid |
| `$(command)` | Modern command substitution — prefer this |

```bash
# BAD — word splitting and globbing
files=$1
rm $files

# GOOD
rm -- "$files"
```

### Command Substitution

```bash
today=$(date +%Y-%m-%d)
branch=$(git rev-parse --abbrev-ref HEAD)
java_version=$(java -version 2>&1 | head -1)

echo "Building on branch $branch with Java: $java_version"
```

### Special Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1`, `$2`, ... | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All arguments as separate words |
| `$*` | All arguments as one string |
| `$?` | Exit code of last command |
| `$$` | Current shell PID |
| `$!` | PID of last background job |

```bash
#!/usr/bin/env bash
echo "Script: $0"
echo "Args count: $#"
echo "All args: $@"
```

### Arithmetic

```bash
count=$((count + 1))
if (( count > 10 )); then echo "many"; fi

# Bash 4+ only (check with bash --version)
echo $(( 2#1010 ))   # Binary to decimal: 10
```

---

## 8. Control Flow

### if / elif / else

```bash
if [[ -f "build.gradle" ]]; then
    echo "Gradle project"
elif [[ -f "pom.xml" ]]; then
    echo "Maven project"
else
    echo "Unknown project type"
    exit 1
fi
```

### Test Operators

**File tests:**

```bash
[[ -f file ]]     # Regular file exists
[[ -d dir ]]      # Directory exists
[[ -x script ]]   # Executable
[[ -r file ]]     # Readable
[[ -s file ]]     # Non-empty
[[ file1 -nt file2 ]]  # file1 newer than file2
```

**String tests:**

```bash
[[ -z "$var" ]]       # Empty string
[[ -n "$var" ]]       # Non-empty
[[ "$a" == "$b" ]]
[[ "$a" != "$b" ]]
```

**Numeric tests (use (( )) or -eq/-gt):**

```bash
[[ $count -gt 0 ]]
(( count > 0 ))
```

**Prefer `[[ ]]` over `[ ]`** — safer with quoting, supports `&&` and `||`.

### case — Pattern Matching

```bash
case "$1" in
    debug|dev)
        ./gradlew assembleDebug
        ;;
    release|prod)
        ./gradlew assembleRelease
        ;;
    *)
        echo "Usage: $0 {debug|release}"
        exit 1
        ;;
esac
```

### Loops

```bash
# for — list iteration
for file in src/**/*.java; do
    echo "Processing $file"
done

# C-style for
for ((i=0; i<10; i++)); do
    echo $i
done

# while
while read -r line; do
    echo "Line: $line"
done < input.txt

# until
until [[ -f "build/success.marker" ]]; do
    sleep 1
done
```

### break / continue

```bash
for i in 1 2 3 4 5; do
    (( i == 3 )) && continue
    echo $i
done
```

---

## 9. Functions & Modular Scripts

### Defining Functions

```bash
greet() {
    local name="$1"    # local scope — always use local
    echo "Hello, $name"
}

greet "Developer"
```

### Return Values

Functions return **exit codes** (0–255), not strings:

```bash
is_port_open() {
    local port="$1"
    nc -z localhost "$port" 2>/dev/null
    return $?
}

if is_port_open 8080; then
    echo "Spring Boot is up"
fi
```

To "return" data, use stdout:

```bash
get_git_branch() {
    git rev-parse --abbrev-ref HEAD 2>/dev/null
}

branch=$(get_git_branch)
```

### Script Template (Production-Ready Starter)

```bash
#!/usr/bin/env bash
#
# deploy.sh — Deploy Spring Boot app to staging
#
set -euo pipefail
IFS=$'\n\t'

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [-e ENV] [-h]

Options:
  -e ENV   Environment: dev|staging|prod (default: dev)
  -h       Show this help
EOF
}

main() {
    local env="dev"

    while getopts ":e:h" opt; do
        case $opt in
            e) env="$OPTARG" ;;
            h) usage; exit 0 ;;
            *) usage; exit 1 ;;
        esac
    done

    log_info "Deploying to: $env"
    # ... your logic
}

log_info()  { echo "[INFO]  $*"; }
log_error() { echo "[ERROR] $*" >&2; }

main "$@"
```

### Sourcing vs Executing

```bash
source ./config.sh    # Runs in current shell — exports vars
. ./config.sh         # Same as source

./config.sh           # Runs in subshell — vars don't persist
```

---

## 10. Exit Codes, Error Handling & Debugging

### The `set` Trifecta (Use in Every Script)

```bash
set -e          # Exit immediately if any command fails
set -u          # Treat unset variables as errors
set -o pipefail # Pipe fails if ANY command in pipe fails
set -euo pipefail  # All three — your default
```

### Checking Exit Codes

```bash
./gradlew build
if [[ $? -ne 0 ]]; then
    echo "Build failed"
    exit 1
fi

# Shorter
./gradlew build || { echo "Build failed"; exit 1; }
```

### trap — Cleanup on Exit

```bash
tmpdir=""
cleanup() {
    [[ -n "$tmpdir" && -d "$tmpdir" ]] && rm -rf "$tmpdir"
}
trap cleanup EXIT

tmpdir=$(mktemp -d)
# ... work in temp dir — always cleaned up, even on error
```

### Debugging

```bash
bash -x script.sh          # Trace every command
set -x                     # Enable tracing in script
set +x                     # Disable tracing

# Debug breakpoint
read -rp "Press enter to continue..."

# Inspect variable
declare -p my_var
```

### Logging Pattern

```bash
readonly LOG_LEVEL="${LOG_LEVEL:-INFO}"

log() {
    local level="$1"; shift
    [[ "$level" == "DEBUG" && "$LOG_LEVEL" != "DEBUG" ]] && return 0
    printf '[%s] [%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$level" "$*"
}

log INFO "Starting build"
log ERROR "Something went wrong"
```

---

## 11. Environment, PATH & Shell Config

### Key Environment Variables

```bash
echo $HOME       # /Users/you or /home/you
echo $PATH       # Colon-separated list of executable directories
echo $SHELL      # Current login shell
echo $USER       # Username
env              # All environment variables
export MY_VAR="value"   # Make available to child processes
```

### PATH — How Your Shell Finds Commands

```bash
# Typical PATH additions for Android dev
export ANDROID_HOME="$HOME/Library/Android/sdk"          # Mac
export ANDROID_HOME="$HOME/Android/Sdk"                  # Linux/WSL
export PATH="$PATH:$ANDROID_HOME/platform-tools"           # adb, fastboot
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin"

# Java (SDKMAN example)
export JAVA_HOME="$HOME/.sdkman/candidates/java/current"
export PATH="$JAVA_HOME/bin:$PATH"
```

### Shell Startup Files (Load Order)

**Bash on Mac:**

```
~/.bash_profile  → login shell (Terminal.app)
~/.bashrc        → non-login interactive shell
```

**zsh on Mac (default):**

```
~/.zprofile  → login
~/.zshrc     → interactive
```

**Best practice:** put `source ~/.bashrc` in `~/.bash_profile` if you use both.

### Useful Aliases for Your Stack

```bash
# ~/.bashrc or ~/.zshrc

alias gs='git status'
alias gd='git diff'
alias gco='git checkout'
alias gcm='git commit -m'
alias gp='git push'

alias gw='./gradlew'
alias gwa='./gradlew assembleDebug'
alias gwt='./gradlew test'
alias mw='./mvnw'
alias mws='./mvnw spring-boot:run'

alias adbdevices='adb devices -l'
alias logcat='adb logcat'

# Safety
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
```

---

## 12. Intermediate Patterns Every Developer Needs

### Parsing Arguments with getopts

```bash
verbose=false
profile="dev"

while getopts ":vp:h" opt; do
    case $opt in
        v) verbose=true ;;
        p) profile="$OPTARG" ;;
        h) usage; exit 0 ;;
        :) echo "Option -$OPTARG requires an argument"; exit 1 ;;
        \?) echo "Invalid option: -$OPTARG"; exit 1 ;;
    esac
done
shift $((OPTIND - 1))   # Remaining positional args
```

For complex CLI parsing, consider external tools: `argparse` (Python), or Bash 4+ `getopts` alternatives.

### Parallel Execution

```bash
# Run 4 jobs in parallel
for module in api web worker scheduler; do
    (cd "$module" && ./mvnw test) &
done
wait    # Wait for all background jobs
echo "All modules tested"
```

### Retry Logic

```bash
retry() {
    local max_attempts="$1"
    shift
    local attempt=1
    local delay=2

    until "$@"; do
        if (( attempt >= max_attempts )); then
            echo "Failed after $max_attempts attempts: $*"
            return 1
        fi
        echo "Attempt $attempt failed. Retrying in ${delay}s..."
        sleep "$delay"
        (( attempt++ ))
        (( delay *= 2 ))   # Exponential backoff
    done
}

retry 5 curl -sf http://localhost:8080/actuator/health
```

### Waiting for a Service

```bash
wait_for_port() {
    local host="${1:-localhost}"
    local port="$2"
    local timeout="${3:-60}"
    local elapsed=0

    while ! nc -z "$host" "$port" 2>/dev/null; do
        if (( elapsed >= timeout )); then
            echo "Timeout waiting for $host:$port"
            return 1
        fi
        sleep 1
        (( elapsed++ ))
    done
    echo "$host:$port is ready"
}

wait_for_port localhost 8080 120
```

### Dry Run Pattern

```bash
DRY_RUN=false

run() {
    if $DRY_RUN; then
        echo "[DRY RUN] $*"
    else
        "$@"
    fi
}

run rm -rf build/
run ./gradlew assembleRelease
```

### Config Files

```bash
# Load KEY=VALUE config (no spaces around =)
if [[ -f ".env" ]]; then
    set -a
    # shellcheck source=/dev/null
    source .env
    set +a
fi
```

---

## 13. Advanced Bash Features

### Arrays

```bash
# Indexed array
modules=("api" "web" "worker")
echo "${modules[0]}"
echo "${#modules[@]}"       # Length
modules+=("scheduler")      # Append

for module in "${modules[@]}"; do
    echo "Building $module"
done

# Associative array (Bash 4+)
declare -A env_ports=([dev]=8080 [staging]=8081 [prod]=443)
echo "${env_ports[dev]}"
```

### Parameter Expansion (Power Features)

```bash
file="/path/to/app.log"

${var:-default}     # Use default if unset or empty
${var:=default}     # Assign default if unset or empty
${var:+alternate}   # Use alternate if set and non-empty
${var:?error msg}   # Exit with error if unset or empty

${file#*/}          # Remove shortest match from start
${file##*/}         # Remove longest match from start → basename
${file%.*}          # Remove shortest from end → strip extension
${file%%/*}         # Remove longest from end → dirname

${var/old/new}      # Replace first match
${var//old/new}     # Replace all matches

${#var}             # String length
```

Example — safe default for Spring profile:

```bash
: "${SPRING_PROFILES_ACTIVE:=dev}"
echo "Using profile: $SPRING_PROFILES_ACTIVE"
```

### Process Substitution

```bash
# Compare outputs without temp files
diff <(./gradlew dependencies) <(git show HEAD:dependencies.txt)

# Feed multiple inputs to command
paste <(cut -d',' -f1 users.csv) <(cut -d',' -f2 users.csv)
```

### Subshells vs Current Shell

```bash
(
    cd subproject
    ./mvnw test    # Directory change doesn't affect parent
)

cd subproject
./mvnw test        # Changes parent shell's directory
```

### Coprocesses & FIFOs (Niche but Powerful)

```bash
mkfifo mypipe
echo "data" > mypipe &
cat mypipe
```

### Brace Expansion

```bash
echo {dev,staging,prod}              # dev staging prod
echo file{1,2,3}.txt                 # file1.txt file2.txt file3.txt
mkdir -p project/{src,test}/{main,resources}
cp config.{bak,orig} config.backup   # Multiple extensions
```

---

## 14. Process Management & Job Control

### Background Jobs

```bash
./gradlew build &       # Run in background
jobs                    # List jobs
fg %1                   # Bring job 1 to foreground
kill %1                 # Kill job 1
wait                    # Wait for all background jobs
```

### Signals

| Signal | Number | Default action | Common use |
|--------|--------|----------------|------------|
| SIGINT | 2 | Terminate | Ctrl+C |
| SIGTERM | 15 | Terminate | `kill PID` |
| SIGKILL | 9 | Kill (cannot catch) | `kill -9 PID` |
| SIGHUP | 1 | Hangup | Reload configs |

```bash
trap 'echo "Caught SIGINT"; exit 130' INT
trap 'cleanup' TERM EXIT
```

### Finding & Killing Processes

```bash
ps aux | grep java
pgrep -f "spring-boot"
pkill -f "gradle"

# Kill process on port 8080
lsof -ti:8080 | xargs kill -9        # Mac
fuser -k 8080/tcp                    # Linux/WSL

# Windows (PowerShell)
# Get-NetTCPConnection -LocalPort 8080 | Select OwningProcess
```

### nohup & disown — Survive Terminal Close

```bash
nohup ./mvnw spring-boot:run > app.log 2>&1 &
disown
```

---

## 15. sed, awk & xargs in Depth

### sed — Stream Editor

```bash
# Replace first occurrence per line
sed 's/debug/info/' config.yml

# Replace all occurrences
sed 's/\r$//' file.txt          # Strip Windows CRLF
sed -i.bak 's/port: 8080/port: 9090/' application.yml   # In-place with backup

# Delete lines matching pattern
sed '/DEBUG/d' app.log

# Print specific lines
sed -n '10,20p' file.txt
```

### awk — Column Processing

```bash
# Print 1st and 3rd column
ps aux | awk '{print $1, $3}'

# Sum a column
awk '{sum += $1} END {print sum}' numbers.txt

# Filter log lines
awk '/ERROR/ {print $1, $2, $NF}' app.log

# CSV: print rows where column 2 > 100
awk -F',' '$2 > 100 {print $0}' data.csv
```

### xargs — Build Argument Lists

```bash
# Delete all .class files
find . -name "*.class" -print0 | xargs -0 rm

# Run 4 parallel jobs
cat modules.txt | xargs -P 4 -I{} ./test-module.sh {}

# With grep
grep -rl "deprecated" src/ | xargs sed -i 's/oldApi/newApi/g'
```

**Always prefer `find -print0 | xargs -0`** for files with spaces in names.

---

## 16. Writing Portable Scripts (Mac, Linux, Windows)

### Shebang Options

```bash
#!/usr/bin/env bash     # BEST — finds bash in PATH (Mac Homebrew, Linux, WSL)
#!/bin/bash             # OK on Linux, may be old on Mac
#!/bin/sh               # POSIX sh — most portable, fewer features
```

### Detect OS

```bash
detect_os() {
    case "$(uname -s)" in
        Darwin*)  echo "macos" ;;
        Linux*)
            if grep -qi microsoft /proc/version 2>/dev/null; then
                echo "wsl"
            else
                echo "linux"
            fi
            ;;
        MINGW*|MSYS*|CYGWIN*) echo "git-bash" ;;
        *) echo "unknown" ;;
    esac
}

OS=$(detect_os)
```

### Cross-Platform Path Helpers

```bash
# Use forward slashes in Bash even on Windows Git Bash
project_root="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"

# Normalize Windows path in Git Bash
# /c/Users/you → usable in Bash
win_path="/c/Users/you/projects"
```

### Use External Tools for Portability

| Task | Portable approach |
|------|-------------------|
| JSON parsing | `jq` |
| YAML parsing | `yq` |
| HTTP requests | `curl` |
| Date formatting | `date` (flags differ — abstract in function) |
| Temp files | `mktemp` |

```bash
# JSON example
version=$(jq -r '.version' package.json)

# HTTP health check
curl -sf http://localhost:8080/actuator/health | jq .
```

### ShellCheck — Lint Your Scripts

```bash
brew install shellcheck    # Mac
sudo apt install shellcheck  # WSL

shellcheck deploy.sh
```

Add to CI:

```yaml
- run: shellcheck scripts/*.sh
```

### POSIX sh vs Bash

If you need maximum portability (Alpine Linux, old systems), stick to POSIX `sh` and avoid:

- `[[ ]]`, arrays, `$(())` with arrays
- `source` (use `.` instead)
- `function name()` (use `name()`)
- `set -o pipefail` (not in POSIX sh)

For dev tooling on Mac/WSL, **Bash 4+ is fine**.

---

## 17. Android Developer Workflows

### Essential Environment Setup

```bash
# ~/.bashrc or ~/.zshrc

# macOS
export ANDROID_HOME="$HOME/Library/Android/sdk"

# Linux / WSL
export ANDROID_HOME="$HOME/Android/Sdk"

export PATH="$PATH:$ANDROID_HOME/platform-tools"
export PATH="$PATH:$ANDROID_HOME/emulator"
export PATH="$PATH:$ANDROID_HOME/cmdline-tools/latest/bin"
export PATH="$PATH:$ANDROID_HOME/build-tools/34.0.0"
```

### adb — Android Debug Bridge

```bash
adb devices                    # List connected devices/emulators
adb -s emulator-5554 shell     # Target specific device
adb install -r app-debug.apk
adb uninstall com.example.app
adb logcat                     # Device logs
adb logcat -s "MyTag:V" "*:S"  # Filter by tag
adb shell pm list packages | grep example
adb shell am start -n com.example/.MainActivity
adb pull /sdcard/screenshot.png .
adb push local.txt /sdcard/
adb reverse tcp:8080 tcp:8080    # Forward device port to localhost (API dev)
```

### Gradle Wrapper Scripts

```bash
./gradlew tasks
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew test
./gradlew connectedAndroidTest
./gradlew clean build --stacktrace
./gradlew :app:dependencies

# Windows (cmd/PowerShell native — not Bash)
gradlew.bat assembleDebug
```

### Useful Android Build Script

```bash
#!/usr/bin/env bash
set -euo pipefail

APK_PATH="app/build/outputs/apk/debug/app-debug.apk"
PACKAGE="com.example.myapp"

build_and_install() {
    ./gradlew assembleDebug
    adb install -r "$APK_PATH"
    adb shell am start -n "$PACKAGE/.MainActivity"
    adb logcat -c
    adb logcat -s "${PACKAGE%%.*}"
}

build_and_install
```

### Emulator Management

```bash
emulator -list-avds
emulator -avd Pixel_7_API_34 -no-snapshot-load &
adb wait-for-device
adb shell getprop sys.boot_completed   # Wait until 1
```

### Signing & Release (Script Sketch)

```bash
#!/usr/bin/env bash
set -euo pipefail

KEYSTORE="${KEYSTORE:-$HOME/keystores/release.jks}"
KEY_ALIAS="${KEY_ALIAS:-release}"

./gradlew bundleRelease \
    -Pandroid.injected.signing.store.file="$KEYSTORE" \
    -Pandroid.injected.signing.store.password="$STORE_PASS" \
    -Pandroid.injected.signing.key.alias="$KEY_ALIAS" \
    -Pandroid.injected.signing.key.password="$KEY_PASS"

echo "AAB: app/build/outputs/bundle/release/"
```

**Never commit keystore passwords** — use env vars or CI secrets.

---

## 18. Spring Boot & Java Developer Workflows

### Java Version Management

```bash
# SDKMAN (Mac/Linux/WSL — recommended)
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 21.0.3-tem
sdk use java 21.0.3-tem
java -version

# jenv (Mac)
brew install jenv
jenv add /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
jenv global 21
```

### Maven Wrapper

```bash
./mvnw clean package
./mvnw spring-boot:run
./mvnw test -Dtest=UserServiceTest
./mvnw verify -Pintegration
./mvnw dependency:tree | grep spring-boot
```

### Spring Boot Run Script

```bash
#!/usr/bin/env bash
set -euo pipefail

PROFILE="${1:-dev}"
PORT="${PORT:-8080}"

export SPRING_PROFILES_ACTIVE="$PROFILE"

./mvnw spring-boot:run \
    -Dspring-boot.run.arguments="--server.port=$PORT" &
APP_PID=$!

trap 'kill $APP_PID 2>/dev/null' EXIT

wait_for_port localhost "$PORT" 120
curl -sf "http://localhost:$PORT/actuator/health" | jq .
echo "Spring Boot running (PID $APP_PID) profile=$PROFILE"
wait $APP_PID
```

### Docker Compose Integration

```bash
#!/usr/bin/env bash
set -euo pipefail

docker compose up -d postgres redis
wait_for_port localhost 5432 30

export SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/mydb"
./mvnw spring-boot:run
```

### Multi-Module Maven Build

```bash
#!/usr/bin/env bash
set -euo pipefail

modules=(common api worker)

for module in "${modules[@]}"; do
    echo "=== Building $module ==="
    (cd "$module" && ../mvnw clean verify)
done
```

### JVM Diagnostics from Shell

```bash
jps -l                          # List Java processes
jstat -gcutil $(jps -q) 1000    # GC stats every second
jmap -heap $(pgrep -f spring-boot)
jcmd <pid> VM.flags
```

---

## 19. CI/CD Scripting Patterns

### GitHub Actions (Bash in YAML)

```yaml
name: Android CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Grant execute permission
        run: chmod +x gradlew

      - name: Build
        run: ./gradlew assembleDebug --stacktrace

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: debug-apk
          path: app/build/outputs/apk/debug/*.apk
```

### Reusable CI Script Pattern

```bash
#!/usr/bin/env bash
# ci/build.sh — called from GitHub Actions, Jenkins, or locally
set -euo pipefail

export GRADLE_OPTS="-Dorg.gradle.daemon=false -Dorg.gradle.jvmargs=-Xmx2g"

log() { echo "[$(date '+%H:%M:%S')] $*"; }

log "Java version: $(java -version 2>&1 | head -1)"
log "Building..."
./gradlew clean test assembleRelease

log "Build complete"
```

### Jenkins-Friendly Script

```bash
#!/usr/bin/env bash
set -euo pipefail

# Jenkins sets WORKSPACE; default for local runs
WORKSPACE="${WORKSPACE:-$(pwd)}"
cd "$WORKSPACE"

./mvnw clean verify \
    -Dmaven.test.failure.ignore=false \
    | tee build.log

# Archive test results
mkdir -p reports
cp -r target/surefire-reports reports/ 2>/dev/null || true
```

### Version Bumping

```bash
#!/usr/bin/env bash
set -euo pipefail

current=$(grep "version =" build.gradle.kts | head -1 | grep -oE '[0-9]+\.[0-9]+\.[0-9]+')
IFS='.' read -r major minor patch <<< "$current"
new_version="$major.$minor.$((patch + 1))"

echo "Bumping $current → $new_version"
sed -i.bak "s/version = \"$current\"/version = \"$new_version\"/" build.gradle.kts
```

---

## 20. Security & Best Practices

### Rules You Must Follow

1. **Always quote variables:** `"$var"`, not `$var`
2. **Use `set -euo pipefail`** in every script
3. **Never use `eval` on user input**
4. **Never store secrets in scripts** — use env vars, `.env` (gitignored), or secret managers
5. **Use `--` before filenames:** `rm -- "$user_file"`
6. **Validate input** before using in commands
7. **Use `mktemp`** instead of predictable temp paths
8. **Run `shellcheck`** before committing scripts
9. **Principle of least privilege** — don't run as root
10. **Avoid `curl | bash`** for untrusted sources

### Safe Input Handling

```bash
# BAD — command injection
eval "rm $user_input"

# GOOD
filename=$(basename "$user_input")   # Strip path traversal
rm -- "$filename"
```

### Secrets in Scripts

```bash
# BAD
API_KEY="sk-abc123"

# GOOD
: "${API_KEY:?API_KEY environment variable is required}"
```

### .gitignore for Local Config

```gitignore
.env
.env.local
*.jks
keystore.properties
```

---

## 21. Quick Reference Cheat Sheet

### File Operations

| Command | Purpose |
|---------|---------|
| `cp -r src dst` | Copy directory |
| `mv a b` | Move/rename |
| `rm -rf dir` | Delete directory |
| `mkdir -p a/b/c` | Create nested dirs |
| `touch file` | Create/update file |
| `chmod +x script` | Make executable |

### Text

| Command | Purpose |
|---------|---------|
| `grep -r pattern dir` | Recursive search |
| `sed 's/a/b/g' file` | Replace text |
| `awk '{print $1}' file` | Print column |
| `sort \| uniq -c` | Count unique |
| `wc -l file` | Line count |
| `head/tail -n N` | First/last N lines |

### Process

| Command | Purpose |
|---------|---------|
| `cmd &` | Background |
| `jobs / fg / kill` | Job control |
| `ps aux \| grep x` | Find process |
| `lsof -i :8080` | What's on port |
| `nohup cmd &` | Immune to hangup |

### Script Header

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

### Test Quick Reference

```bash
[[ -f file ]]    # file exists
[[ -d dir ]]     # directory exists
[[ -z "$s" ]]    # string empty
[[ "$a" == "$b" ]] # string equal
[[ $n -gt 0 ]]   # number greater than
(( n > 0 ))      # arithmetic
```

---

## 22. Learning Path & Next Steps

### Level 1 — Beginner (Week 1)

- [ ] Navigate filesystem with `cd`, `ls`, `pwd`
- [ ] Create/edit files with your editor (`code .`, `vim`, `nano`)
- [ ] Use `grep`, `tail -f`, pipes
- [ ] Understand `$PATH`, `export`, `~/.bashrc` or `~/.zshrc`
- [ ] Run `./gradlew` and `./mvnw` confidently

### Level 2 — Intermediate (Week 2–3)

- [ ] Write scripts with `set -euo pipefail`
- [ ] Use `if`, `for`, `case`, functions
- [ ] Parse arguments with `getopts`
- [ ] Use `trap` for cleanup
- [ ] Automate one repetitive Android or Spring Boot task

### Level 3 — Advanced (Week 4+)

- [ ] Master parameter expansion and arrays
- [ ] Write portable scripts (Mac + WSL)
- [ ] Use `sed`, `awk`, `xargs` fluently
- [ ] Integrate scripts into CI/CD
- [ ] Run `shellcheck` and fix all warnings
- [ ] Read and understand existing Gradle/Maven wrapper scripts

### Practice Projects

1. **Dev environment bootstrap** — script that installs SDKMAN, sets `ANDROID_HOME`, clones repos, runs first build
2. **Pre-commit hook** — run `gradlew test` or `mvnw verify` before every commit
3. **Log analyzer** — parse Spring Boot log for errors, output summary
4. **Release script** — bump version, build, sign APK/AAB, upload to Play Console (fastlane integrates well)
5. **Health monitor** — poll `/actuator/health`, alert on failure

### Recommended Tools to Install

```bash
# Mac
brew install bash bash-completion jq shellcheck fd ripgrep tree

# Ubuntu / WSL
sudo apt update && sudo apt install -y bash-completion jq shellcheck fd-find ripgrep tree
```

### Further Reading

- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [ShellCheck Wiki](https://www.shellcheck.net/wiki/)
- [Gradle Command-Line Interface](https://docs.gradle.org/current/userguide/command_line_interface.html)
- [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/cli.html)

---

## Appendix A: Bash vs zsh vs sh (Quick Comparison)

| Feature | sh (POSIX) | Bash 5 | zsh |
|---------|-----------|--------|-----|
| macOS default | No | Optional | Yes (Catalina+) |
| Arrays | No | Yes | Yes |
| `[[ ]]` | No | Yes | Yes |
| Globbing | Basic | Good | Excellent |
| Script shebang | `#!/bin/sh` | `#!/usr/bin/env bash` | `#!/bin/zsh` |

For **scripts you share** (CI, teammates): use `#!/usr/bin/env bash` with `set -euo pipefail`.

For **interactive use on Mac**: zsh is fine — syntax is 95% compatible with what this guide teaches.

---

## Appendix B: Windows-Specific Tips

### Accessing Project from WSL

```bash
# Windows path C:\Users\you\projects\myapp
cd /mnt/c/Users/you/projects/myapp
./gradlew build    # Works — but line endings and file watching can be slow

# Better: keep repos inside WSL filesystem
cd ~/projects/myapp   # /home/you/projects — much faster I/O
```

### Git Bash Path Conversion

```bash
# Git Bash auto-translates
cd /c/Users/you/projects   # Same as C:\Users\you\projects
```

### Running PowerShell from Bash

```bash
powershell.exe -Command "Get-Process java"
```

### Android Emulator on WSL

The Android Emulator generally requires **native Windows** or **native Mac** — run emulator on host OS, use `adb` from WSL via:

```bash
export WSL_HOST=$(tail -1 /etc/resolv.conf | cut -d' ' -f2)
export ADB_SERVER_SOCKET=tcp:$WSL_HOST:5037
```

Or connect to Windows adb server — consult latest Android/WSL docs as this area evolves.

---

## Appendix C: Sample `scripts/dev-setup.sh`

A complete starter script tying everything together:

```bash
#!/usr/bin/env bash
#
# dev-setup.sh — Bootstrap Android + Spring Boot dev environment
#
set -euo pipefail
IFS=$'\n\t'

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

log()  { printf '\033[0;32m[INFO]\033[0m %s\n' "$*"; }
warn() { printf '\033[1;33m[WARN]\033[0m %s\n' "$*" >&2; }
die()  { printf '\033[0;31m[ERR ]\033[0m %s\n' "$*" >&2; exit 1; }

detect_os() {
    case "$(uname -s)" in
        Darwin*) echo "macos" ;;
        Linux*)  echo "linux" ;;
        *)       echo "other" ;;
    esac
}

install_sdkman() {
    if [[ -d "$HOME/.sdkman" ]]; then
        log "SDKMAN already installed"
    else
        log "Installing SDKMAN..."
        curl -s "https://get.sdkman.io" | bash
    fi
    # shellcheck source=/dev/null
    source "$HOME/.sdkman/bin/sdkman-init.sh"
    sdk install java 21.0.3-tem
}

setup_android_env() {
    local os="$1"
    local profile_shell=""

    if [[ "$os" == "macos" ]]; then
        ANDROID_HOME="${ANDROID_HOME:-$HOME/Library/Android/sdk}"
        profile_shell="$HOME/.zshrc"
    else
        ANDROID_HOME="${ANDROID_HOME:-$HOME/Android/Sdk}"
        profile_shell="$HOME/.bashrc"
    fi

    log "ANDROID_HOME=$ANDROID_HOME"

    if ! grep -q "ANDROID_HOME" "$profile_shell" 2>/dev/null; then
        cat >> "$profile_shell" <<EOF

# Android SDK (added by dev-setup.sh)
export ANDROID_HOME="$ANDROID_HOME"
export PATH="\$PATH:\$ANDROID_HOME/platform-tools:\$ANDROID_HOME/emulator"
EOF
        log "Updated $profile_shell"
    fi
}

verify_tools() {
  local missing=()
  for tool in git java adb; do
    command -v "$tool" &>/dev/null || missing+=("$tool")
  done

  if ((${#missing[@]} > 0)); then
    warn "Missing tools: ${missing[*]}"
  else
    log "All core tools available"
    java -version 2>&1 | head -1
    adb version | head -1
  fi
}

main() {
    local os
    os=$(detect_os)
    log "Detected OS: $os"

    install_sdkman
    setup_android_env "$os"
    verify_tools

    log "Setup complete. Restart your terminal or run: source ~/.bashrc"
}

main "$@"
```

---

*Last updated: June 2026. Written for developers using Android (Gradle/Kotlin/Java) and Spring Boot (Maven/Gradle) on macOS and Windows (Git Bash / WSL2).*
