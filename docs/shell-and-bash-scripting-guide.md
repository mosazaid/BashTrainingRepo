# Shell and Bash Scripting Guide for Android and Spring Boot Developers

This document is a practical path from zero knowledge to advanced Shell and
Bash scripting. It is written for developers who build Android apps and Java
Spring Boot services on macOS and Windows.

The goal is not only to memorize commands. The goal is to understand how the
terminal works, how to automate repeatable development tasks, how to debug
scripts, and how to write safe scripts that can run locally or in CI/CD.

---

## Table of Contents

1. [Mental Model](#1-mental-model)
2. [Shells, Terminals, Bash, Zsh, PowerShell, and WSL](#2-shells-terminals-bash-zsh-powershell-and-wsl)
3. [Recommended Setup for macOS and Windows](#3-recommended-setup-for-macos-and-windows)
4. [Filesystem Basics](#4-filesystem-basics)
5. [Essential Commands](#5-essential-commands)
6. [Quoting, Expansion, and Substitution](#6-quoting-expansion-and-substitution)
7. [Input, Output, Pipes, and Redirection](#7-input-output-pipes-and-redirection)
8. [Permissions, Executables, and Shebangs](#8-permissions-executables-and-shebangs)
9. [Environment Variables and PATH](#9-environment-variables-and-path)
10. [Package Managers and Developer Tooling](#10-package-managers-and-developer-tooling)
11. [Bash Script Fundamentals](#11-bash-script-fundamentals)
12. [Conditionals, Tests, and Comparisons](#12-conditionals-tests-and-comparisons)
13. [Loops](#13-loops)
14. [Functions](#14-functions)
15. [Arguments and Flags](#15-arguments-and-flags)
16. [Exit Codes and Error Handling](#16-exit-codes-and-error-handling)
17. [Debugging Bash Scripts](#17-debugging-bash-scripts)
18. [Text Processing](#18-text-processing)
19. [Working with JSON, YAML, and Properties Files](#19-working-with-json-yaml-and-properties-files)
20. [Arrays, Maps, and Advanced Bash Data Handling](#20-arrays-maps-and-advanced-bash-data-handling)
21. [Files, Directories, and Safe Temporary Workspaces](#21-files-directories-and-safe-temporary-workspaces)
22. [Processes, Jobs, Signals, and Traps](#22-processes-jobs-signals-and-traps)
23. [Networking from the Shell](#23-networking-from-the-shell)
24. [Git Automation](#24-git-automation)
25. [Android Development Automation](#25-android-development-automation)
26. [Java and Spring Boot Automation](#26-java-and-spring-boot-automation)
27. [Docker and Local Infrastructure](#27-docker-and-local-infrastructure)
28. [Cross-Platform Scripting for macOS and Windows](#28-cross-platform-scripting-for-macos-and-windows)
29. [CI/CD Scripting](#29-cicd-scripting)
30. [Security Practices](#30-security-practices)
31. [Performance and Reliability](#31-performance-and-reliability)
32. [Reusable Script Architecture](#32-reusable-script-architecture)
33. [Advanced Patterns](#33-advanced-patterns)
34. [Cheat Sheets](#34-cheat-sheets)
35. [Practice Roadmap](#35-practice-roadmap)

---

## 1. Mental Model

A shell is a command interpreter. You type a command, the shell parses it, then
starts a program or runs shell built-in behavior.

Example:

```bash
ls -la docs
```

The shell sees:

- `ls`: command name
- `-la`: option flags
- `docs`: argument

The command returns:

- standard output, also called `stdout`
- standard error, also called `stderr`
- an exit code

Exit codes matter:

- `0` usually means success
- non-zero usually means failure

```bash
echo "hello"
echo $?

ls missing-file
echo $?
```

As a developer, you use this model to automate tasks:

- build an Android app
- run Gradle tests
- start a Spring Boot service
- check if a port is busy
- call an API
- parse logs
- prepare release artifacts
- clean generated files
- run CI/CD steps

---

## 2. Shells, Terminals, Bash, Zsh, PowerShell, and WSL

These names are related but different.

| Term | Meaning |
| --- | --- |
| Terminal | The application window where you type commands. |
| Shell | The program that interprets commands. |
| Bash | A popular Unix shell and scripting language. |
| Zsh | Default shell on modern macOS. Mostly compatible with Bash interactively, but not identical for scripting. |
| PowerShell | Microsoft's object-oriented shell. Different syntax from Bash. |
| CMD | Legacy Windows command prompt. Avoid for modern scripting. |
| Git Bash | Bash environment installed with Git for Windows. |
| WSL | Windows Subsystem for Linux. Runs a real Linux environment on Windows. |

Recommended approach:

- On macOS: use Terminal or iTerm2 with Zsh interactively, but write scripts for
  Bash using `#!/usr/bin/env bash`.
- On Windows: use WSL2 for serious Bash scripting and Android/Spring automation.
  Git Bash is useful for simple tasks, but WSL behaves closer to CI Linux.
- In CI/CD: most Linux runners use Bash or POSIX `sh`.

### Bash vs POSIX sh

`sh` is more portable but less powerful. Bash has arrays, `[[ ... ]]`, better
string operations, and safer scripting features.

Use Bash when:

- scripts are for your team or CI environment
- you need arrays or advanced conditionals
- you need predictable behavior across Linux/macOS/WSL

Use POSIX `sh` when:

- the script must run on very minimal systems
- portability is more important than convenience

---

## 3. Recommended Setup for macOS and Windows

### macOS

Install Homebrew:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install common tools:

```bash
brew install bash coreutils git jq yq curl wget ripgrep fd shellcheck shfmt
```

macOS includes an old Bash version by default because of licensing. Homebrew
installs a modern Bash, often at:

```bash
/opt/homebrew/bin/bash
```

For scripts, prefer:

```bash
#!/usr/bin/env bash
```

That finds Bash from the user's `PATH`.

### Windows

Recommended:

1. Install WSL2.
2. Install Ubuntu from Microsoft Store.
3. Use Windows Terminal.
4. Install Git for Windows for IDE integration.
5. Install Android Studio normally on Windows.
6. Share only needed project folders between Windows and WSL.

Inside WSL:

```bash
sudo apt update
sudo apt install -y bash git curl wget jq ripgrep fd-find shellcheck shfmt unzip zip
```

Important Windows notes:

- Windows paths look like `C:\Users\you\project`.
- WSL paths look like `/mnt/c/Users/you/project`.
- Bash uses `/` separators, not `\`.
- File line endings may differ: Unix uses LF, Windows often uses CRLF.

Check line endings:

```bash
file script.sh
```

Fix CRLF:

```bash
dos2unix script.sh
```

or:

```bash
sed -i 's/\r$//' script.sh
```

---

## 4. Filesystem Basics

Important paths:

```bash
pwd                 # print current directory
ls                  # list files
ls -la              # list hidden files and details
cd app              # enter directory
cd ..               # go up one directory
cd ~                # go to home directory
```

Absolute path:

```bash
/Users/alex/projects/my-app
/home/alex/projects/my-app
```

Relative path:

```bash
./gradlew
../shared-config/application.yml
```

Special names:

| Path | Meaning |
| --- | --- |
| `.` | current directory |
| `..` | parent directory |
| `~` | current user's home |
| `/` | filesystem root |

Always quote paths that may contain spaces:

```bash
cd "$HOME/Android Projects/My App"
```

---

## 5. Essential Commands

### Navigation and inspection

```bash
pwd
ls -la
tree -L 2
du -sh build
df -h
```

### Create, copy, move, remove

```bash
mkdir -p docs/scripts
touch notes.txt
cp source.txt backup.txt
cp -R app app-backup
mv old-name new-name
rm file.txt
rm -rf build
```

Be careful with `rm -rf`. A bad variable can delete the wrong directory.

Safer pattern:

```bash
target="${BUILD_DIR:-}"

if [[ -z "$target" || "$target" == "/" ]]; then
  echo "Refusing to delete unsafe target: '$target'" >&2
  exit 1
fi

rm -rf "$target"
```

### Read files

```bash
less application.yml
sed -n '1,80p' build.gradle
tail -f logs/app.log
```

### Search

```bash
grep -R "TODO" src
rg "TODO" src
rg --files | rg "Controller.java$"
```

Prefer `rg` when available. It is faster and respects `.gitignore`.

### Find files

```bash
find . -name "*.java"
find . -type f -name "*.kt"
find . -type d -name build
```

Modern alternative:

```bash
fd "\.java$"
fd build -t d
```

### Command help

```bash
man ls
ls --help
tldr tar
```

---

## 6. Quoting, Expansion, and Substitution

Quoting is one of the most important Bash topics.

### Double quotes

Double quotes allow variable expansion:

```bash
name="Spring Boot"
echo "Hello $name"
```

Use double quotes around variables by default:

```bash
rm -rf "$build_dir"
```

### Single quotes

Single quotes prevent expansion:

```bash
echo '$HOME'
```

Output:

```text
$HOME
```

### No quotes

Unquoted variables can split into multiple words and expand wildcards:

```bash
path="My Project"
cd $path      # wrong
cd "$path"    # correct
```

### Command substitution

```bash
current_branch="$(git branch --show-current)"
echo "Branch: $current_branch"
```

Prefer `$(...)` over backticks.

### Brace expansion

```bash
mkdir -p src/{main,test}/{java,resources}
touch app-{dev,staging,prod}.yml
```

### Globs

```bash
ls *.gradle
ls src/**/*.java
```

`**` needs `globstar` enabled in Bash:

```bash
shopt -s globstar
```

---

## 7. Input, Output, Pipes, and Redirection

Every process has common streams:

| Stream | Number | Meaning |
| --- | --- | --- |
| stdin | `0` | input |
| stdout | `1` | normal output |
| stderr | `2` | error output |

### Redirect output

```bash
echo "hello" > file.txt      # overwrite
echo "again" >> file.txt     # append
```

### Redirect errors

```bash
./gradlew test 2> errors.log
./gradlew test > output.log 2> errors.log
./gradlew test > all.log 2>&1
```

### Pipe output to another command

```bash
ps aux | rg "java"
adb logcat | rg "MainActivity"
curl -s http://localhost:8080/actuator/health | jq .
```

### Here documents

Useful for generating config files:

```bash
cat > application-local.yml <<'YAML'
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/app
YAML
```

Quote the delimiter (`'YAML'`) when you do not want variable expansion.

---

## 8. Permissions, Executables, and Shebangs

Make a script executable:

```bash
chmod +x scripts/build-android.sh
```

Run it:

```bash
./scripts/build-android.sh
```

Shebang:

```bash
#!/usr/bin/env bash
```

It tells the OS which interpreter should run the file.

Recommended script header:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
```

Meaning:

- `-E`: inherit error traps in functions and subshells
- `-e`: exit on unhandled command failure
- `-u`: fail on unset variables
- `-o pipefail`: fail a pipeline if any command fails

Use it carefully. It makes scripts safer, but you must intentionally handle
commands that may fail.

---

## 9. Environment Variables and PATH

Environment variables configure processes.

```bash
echo "$HOME"
echo "$PATH"
echo "$JAVA_HOME"
echo "$ANDROID_HOME"
```

Set for current shell:

```bash
export SPRING_PROFILES_ACTIVE=local
```

Set for one command:

```bash
SPRING_PROFILES_ACTIVE=test ./gradlew bootRun
```

Add a directory to `PATH`:

```bash
export PATH="$HOME/bin:$PATH"
```

Common developer variables:

```bash
export JAVA_HOME="/path/to/jdk"
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$PATH"
```

Windows WSL example:

```bash
export ANDROID_HOME="/mnt/c/Users/$USER/AppData/Local/Android/Sdk"
export PATH="$ANDROID_HOME/platform-tools:$PATH"
```

Check if a command exists:

```bash
command -v java
command -v adb
command -v ./gradlew
```

---

## 10. Package Managers and Developer Tooling

### macOS

```bash
brew search openjdk
brew install openjdk@21
brew install jq yq shellcheck shfmt
```

### Ubuntu or WSL

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk jq shellcheck shfmt
```

### SDKMAN for Java

SDKMAN is useful for switching JDKs:

```bash
curl -s "https://get.sdkman.io" | bash
sdk list java
sdk install java 21.0.4-tem
sdk use java 21.0.4-tem
```

### Verify Java and Gradle

```bash
java -version
javac -version
./gradlew --version
```

---

## 11. Bash Script Fundamentals

Create `scripts/hello.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

name="${1:-developer}"
echo "Hello, $name"
```

Run:

```bash
chmod +x scripts/hello.sh
./scripts/hello.sh Alex
```

### Variables

```bash
app_name="orders-service"
version="1.0.0"
echo "$app_name:$version"
```

No spaces around `=`:

```bash
name="ok"    # correct
name = "bad" # wrong
```

### Read input

```bash
read -r -p "Environment: " env_name
echo "Selected: $env_name"
```

### Constants by convention

Bash does not have true constants, but uppercase names are commonly used:

```bash
readonly PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
readonly BUILD_DIR="$PROJECT_ROOT/build"
```

---

## 12. Conditionals, Tests, and Comparisons

Use `[[ ... ]]` in Bash scripts.

```bash
if [[ "$environment" == "prod" ]]; then
  echo "Production"
else
  echo "Non-production"
fi
```

### File checks

```bash
[[ -f "build.gradle" ]] && echo "Gradle project"
[[ -d ".git" ]] && echo "Git repo"
[[ -x "./gradlew" ]] && echo "Gradle wrapper is executable"
```

Common tests:

| Test | Meaning |
| --- | --- |
| `-f path` | regular file exists |
| `-d path` | directory exists |
| `-e path` | path exists |
| `-x path` | executable |
| `-z "$value"` | empty string |
| `-n "$value"` | non-empty string |

### Numeric comparisons

```bash
if (( port > 1024 )); then
  echo "Non-privileged port"
fi
```

Operators:

```bash
(( a == b ))
(( a != b ))
(( a < b ))
(( a <= b ))
(( a > b ))
(( a >= b ))
```

### Pattern matching

```bash
if [[ "$file" == *.apk ]]; then
  echo "Android package"
fi
```

Regex:

```bash
if [[ "$version" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "Semantic version"
fi
```

---

## 13. Loops

### For loop over values

```bash
for env in dev staging prod; do
  echo "Deploy target: $env"
done
```

### For loop over files

```bash
for file in src/main/resources/*.yml; do
  echo "Config: $file"
done
```

Handle empty globs safely:

```bash
shopt -s nullglob

for file in reports/*.xml; do
  echo "Report: $file"
done
```

### While loop

```bash
while read -r line; do
  echo "Log: $line"
done < app.log
```

### Retry loop

```bash
for attempt in {1..5}; do
  if curl -fsS http://localhost:8080/actuator/health; then
    echo "Service is ready"
    break
  fi

  echo "Waiting for service, attempt $attempt"
  sleep 2
done
```

---

## 14. Functions

Functions make scripts reusable and readable.

```bash
log() {
  printf '[%s] %s\n' "$(date +'%Y-%m-%dT%H:%M:%S%z')" "$*"
}

die() {
  echo "ERROR: $*" >&2
  exit 1
}

require_command() {
  local command_name="$1"

  if ! command -v "$command_name" >/dev/null 2>&1; then
    die "Missing required command: $command_name"
  fi
}

require_command java
require_command adb
```

Use `local` for function variables:

```bash
build_module() {
  local module="$1"
  ./gradlew ":$module:build"
}
```

Return values:

- return small success/failure status with `return`
- print data to stdout for command substitution

```bash
current_branch() {
  git branch --show-current
}

branch="$(current_branch)"
```

---

## 15. Arguments and Flags

Positional parameters:

```bash
echo "$0" # script name
echo "$1" # first argument
echo "$2" # second argument
echo "$#" # number of arguments
echo "$@" # all arguments, safely when quoted
```

Basic parser:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

environment="local"
skip_tests=false

while [[ $# -gt 0 ]]; do
  case "$1" in
    --env)
      environment="${2:?Missing value for --env}"
      shift 2
      ;;
    --skip-tests)
      skip_tests=true
      shift
      ;;
    -h|--help)
      echo "Usage: $0 [--env local|dev|prod] [--skip-tests]"
      exit 0
      ;;
    *)
      echo "Unknown argument: $1" >&2
      exit 1
      ;;
  esac
done

echo "Environment: $environment"
echo "Skip tests: $skip_tests"
```

For very complex CLIs, consider writing the tool in a general-purpose language
such as Kotlin, Java, Python, Go, or Node.js.

---

## 16. Exit Codes and Error Handling

Exit codes drive automation.

```bash
if ./gradlew test; then
  echo "Tests passed"
else
  echo "Tests failed" >&2
  exit 1
fi
```

Recommended strict mode:

```bash
set -Eeuo pipefail
```

Handle expected failures:

```bash
if ! rg "TODO" src; then
  echo "No TODO comments found"
fi
```

Pipeline failure:

```bash
set -o pipefail
adb logcat -d | rg "FATAL EXCEPTION" > crash.log
```

Without `pipefail`, a failing `adb logcat` may be hidden if `rg` succeeds or
exits differently.

### Error trap

```bash
trap 'echo "Failed at line $LINENO: $BASH_COMMAND" >&2' ERR
```

Example script skeleton:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

trap 'echo "ERROR: line $LINENO: $BASH_COMMAND" >&2' ERR

main() {
  ./gradlew clean test
}

main "$@"
```

---

## 17. Debugging Bash Scripts

Syntax check:

```bash
bash -n scripts/build.sh
```

Trace execution:

```bash
bash -x scripts/build.sh
```

Trace inside a script:

```bash
set -x
./gradlew test
set +x
```

Use ShellCheck:

```bash
shellcheck scripts/*.sh
```

Format:

```bash
shfmt -w scripts
```

Common debugging helpers:

```bash
echo "value='$value'"
printf '%q\n' "$value"
declare -p array_name
```

---

## 18. Text Processing

### grep and ripgrep

```bash
rg "TODO|FIXME" src
rg -n "class .*Controller" src/main/java
rg --files | rg "\.kt$"
```

### sed

Print lines:

```bash
sed -n '1,40p' application.yml
```

Replace text:

```bash
sed 's/dev/staging/g' config.txt
```

In-place replacement differs on macOS and Linux:

```bash
# Linux
sed -i 's/dev/staging/g' config.txt

# macOS
sed -i '' 's/dev/staging/g' config.txt
```

Cross-platform helper:

```bash
sed_in_place() {
  local expression="$1"
  local file="$2"

  if [[ "$(uname -s)" == "Darwin" ]]; then
    sed -i '' "$expression" "$file"
  else
    sed -i "$expression" "$file"
  fi
}
```

### awk

Print first column:

```bash
awk '{print $1}' access.log
```

Filter rows:

```bash
awk '$9 >= 500 {print $0}' access.log
```

Sum values:

```bash
awk '{sum += $1} END {print sum}' numbers.txt
```

### cut, sort, uniq

```bash
cut -d',' -f1 users.csv
sort names.txt
sort names.txt | uniq -c
```

### xargs

```bash
rg -l "old.package.name" app/src | xargs sed -i 's/old.package.name/new.package.name/g'
```

Safer with null separators:

```bash
fd -0 "\.java$" src | xargs -0 rg "TODO"
```

---

## 19. Working with JSON, YAML, and Properties Files

### JSON with jq

Pretty print:

```bash
curl -s http://localhost:8080/actuator/health | jq .
```

Extract field:

```bash
jq -r '.status' health.json
```

Filter:

```bash
jq -r '.dependencies[] | select(.version | startswith("1.")) | .name' deps.json
```

Create JSON:

```bash
jq -n --arg version "$VERSION" '{version: $version}'
```

### YAML with yq

```bash
yq '.spring.profiles.active' application.yml
yq '.server.port = 9090' -i application.yml
```

### Java properties

Read a property:

```bash
get_property() {
  local key="$1"
  local file="$2"

  awk -F= -v key="$key" '$1 == key {print substr($0, index($0, "=") + 1)}' "$file"
}

app_name="$(get_property "spring.application.name" "src/main/resources/application.properties")"
```

For complex properties handling, prefer Java, Kotlin, Gradle, or a dedicated
parser instead of fragile shell parsing.

---

## 20. Arrays, Maps, and Advanced Bash Data Handling

### Indexed arrays

```bash
modules=("app" "core" "feature-auth")

for module in "${modules[@]}"; do
  ./gradlew ":$module:test"
done
```

Always quote `"${array[@]}"`.

### Associative arrays

```bash
declare -A ports=(
  [api]=8080
  [postgres]=5432
  [redis]=6379
)

echo "${ports[api]}"
```

Requires Bash 4+. macOS system Bash is usually 3.2, so install modern Bash via
Homebrew if you need associative arrays.

### Read command output into an array

```bash
mapfile -t changed_files < <(git diff --name-only)

for file in "${changed_files[@]}"; do
  echo "$file"
done
```

`mapfile` may not exist in old Bash versions.

---

## 21. Files, Directories, and Safe Temporary Workspaces

### Script location

```bash
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"
```

This lets a script work regardless of where it is called from.

### Temporary directories

```bash
tmp_dir="$(mktemp -d)"
trap 'rm -rf "$tmp_dir"' EXIT

echo "Working in $tmp_dir"
```

### Safe file writes

Write to a temp file, then move into place:

```bash
tmp_file="$(mktemp)"
generate_config > "$tmp_file"
mv "$tmp_file" application-generated.yml
```

This prevents partially written files when a command fails.

---

## 22. Processes, Jobs, Signals, and Traps

List Java processes:

```bash
ps aux | rg "java"
jps -l
```

Find process using a port:

```bash
lsof -i :8080
```

Kill process:

```bash
kill 12345
kill -9 12345
```

Prefer normal `kill` first. `kill -9` prevents graceful shutdown.

### Background jobs

```bash
./gradlew bootRun &
server_pid=$!

echo "Server PID: $server_pid"
```

Wait:

```bash
wait "$server_pid"
```

Cleanup:

```bash
cleanup() {
  if [[ -n "${server_pid:-}" ]]; then
    kill "$server_pid" 2>/dev/null || true
  fi
}

trap cleanup EXIT INT TERM
```

---

## 23. Networking from the Shell

### curl basics

```bash
curl http://localhost:8080/actuator/health
curl -i http://localhost:8080/actuator/health
curl -fsS http://localhost:8080/actuator/health
```

Common flags:

| Flag | Meaning |
| --- | --- |
| `-f` | fail on HTTP errors |
| `-s` | silent |
| `-S` | show errors with silent |
| `-L` | follow redirects |
| `-H` | header |
| `-d` | request body |
| `-o` | output file |

POST JSON:

```bash
curl -fsS \
  -H "Content-Type: application/json" \
  -d '{"name":"demo"}' \
  http://localhost:8080/api/projects
```

Avoid exposing tokens in shell history:

```bash
read -r -s -p "Token: " TOKEN
echo

curl -fsS -H "Authorization: Bearer $TOKEN" http://localhost:8080/api/me
```

### Check open port

```bash
nc -z localhost 8080
```

### Wait for service

```bash
wait_for_http() {
  local url="$1"
  local attempts="${2:-30}"

  for ((i = 1; i <= attempts; i++)); do
    if curl -fsS "$url" >/dev/null; then
      return 0
    fi

    sleep 1
  done

  echo "Timed out waiting for $url" >&2
  return 1
}

wait_for_http "http://localhost:8080/actuator/health" 60
```

---

## 24. Git Automation

Useful commands:

```bash
git status --short --branch
git branch --show-current
git diff
git diff --name-only
git log --oneline -n 10
```

Check for clean working tree:

```bash
if [[ -n "$(git status --porcelain)" ]]; then
  echo "Working tree has changes" >&2
  exit 1
fi
```

Run tests only for changed Java files:

```bash
mapfile -t changed_java < <(git diff --name-only origin/main...HEAD -- '*.java')

if (( ${#changed_java[@]} > 0 )); then
  ./gradlew test
fi
```

Create release tag:

```bash
version="${1:?Usage: $0 VERSION}"
git tag -a "v$version" -m "Release $version"
git push origin "v$version"
```

---

## 25. Android Development Automation

### Common Android commands

```bash
adb devices
adb shell getprop ro.build.version.release
adb install app/build/outputs/apk/debug/app-debug.apk
adb uninstall com.example.app
adb logcat
adb logcat -d > logcat.txt
```

Filter logs:

```bash
adb logcat | rg "AndroidRuntime|FATAL EXCEPTION|com.example"
```

### Gradle wrapper

Always prefer the wrapper:

```bash
./gradlew tasks
./gradlew clean
./gradlew test
./gradlew connectedAndroidTest
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew bundleRelease
```

### Build APK script

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

trap 'echo "ERROR: line $LINENO: $BASH_COMMAND" >&2' ERR

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"

cd "$PROJECT_ROOT"

./gradlew clean assembleDebug

apk_path="$(find app/build/outputs/apk/debug -name "*.apk" | head -n 1)"
echo "APK created: $apk_path"
```

### Install latest debug APK

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

package_name="${1:?Usage: $0 PACKAGE_NAME}"

./gradlew assembleDebug

apk_path="$(find app/build/outputs/apk/debug -name "*.apk" | head -n 1)"

adb install -r "$apk_path"
adb shell monkey -p "$package_name" 1
```

### Clear app data

```bash
adb shell pm clear com.example.app
```

### Capture screenshot

```bash
adb exec-out screencap -p > screenshot.png
```

### Record screen

```bash
adb shell screenrecord /sdcard/demo.mp4
adb pull /sdcard/demo.mp4 .
adb shell rm /sdcard/demo.mp4
```

### Run emulator

List AVDs:

```bash
emulator -list-avds
```

Start:

```bash
emulator -avd Pixel_7_API_35
```

Wait for boot:

```bash
adb wait-for-device

until [[ "$(adb shell getprop sys.boot_completed 2>/dev/null | tr -d '\r')" == "1" ]]; do
  sleep 2
done

echo "Emulator is ready"
```

### Android release checks

```bash
./gradlew lint
./gradlew test
./gradlew connectedAndroidTest
./gradlew bundleRelease
```

Find generated artifacts:

```bash
find app/build/outputs -type f \( -name "*.apk" -o -name "*.aab" \)
```

### Decode APK metadata

```bash
aapt dump badging app-release.apk
```

or:

```bash
apkanalyzer manifest application-id app-release.apk
```

---

## 26. Java and Spring Boot Automation

### Common Gradle commands

```bash
./gradlew clean
./gradlew test
./gradlew bootRun
./gradlew bootJar
./gradlew check
```

### Common Maven commands

```bash
./mvnw clean test
./mvnw spring-boot:run
./mvnw package
```

### Start Spring Boot with profile

Gradle:

```bash
SPRING_PROFILES_ACTIVE=local ./gradlew bootRun
```

Jar:

```bash
java -jar build/libs/orders-service.jar --spring.profiles.active=local
```

Maven:

```bash
SPRING_PROFILES_ACTIVE=local ./mvnw spring-boot:run
```

### Wait for Spring Boot health

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

./gradlew bootRun &
server_pid=$!

cleanup() {
  kill "$server_pid" 2>/dev/null || true
}
trap cleanup EXIT

for attempt in {1..60}; do
  if curl -fsS http://localhost:8080/actuator/health | jq -e '.status == "UP"' >/dev/null; then
    echo "Service is healthy"
    wait "$server_pid"
    exit 0
  fi

  sleep 1
done

echo "Service did not become healthy" >&2
exit 1
```

### Run integration tests after startup

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

./gradlew bootRun &
server_pid=$!

cleanup() {
  kill "$server_pid" 2>/dev/null || true
}
trap cleanup EXIT

until curl -fsS http://localhost:8080/actuator/health >/dev/null; do
  sleep 1
done

./gradlew integrationTest
```

### Check port before starting service

```bash
port="${PORT:-8080}"

if lsof -i ":$port" >/dev/null 2>&1; then
  echo "Port $port is already in use" >&2
  lsof -i ":$port"
  exit 1
fi
```

### JVM memory options

```bash
JAVA_OPTS="-Xms512m -Xmx1g" java $JAVA_OPTS -jar build/libs/app.jar
```

Prefer arrays for command construction:

```bash
java_args=(
  "-Xms512m"
  "-Xmx1g"
  "-Dspring.profiles.active=local"
  "-jar"
  "build/libs/app.jar"
)

java "${java_args[@]}"
```

---

## 27. Docker and Local Infrastructure

Start dependencies:

```bash
docker compose up -d postgres redis
```

View logs:

```bash
docker compose logs -f api
```

Stop:

```bash
docker compose down
```

Reset volumes:

```bash
docker compose down -v
```

Wait for container:

```bash
until docker compose exec -T postgres pg_isready -U postgres; do
  sleep 1
done
```

Run database migration:

```bash
./gradlew flywayMigrate
```

Local dev script:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

docker compose up -d postgres redis

until docker compose exec -T postgres pg_isready -U postgres >/dev/null 2>&1; do
  echo "Waiting for postgres..."
  sleep 1
done

SPRING_PROFILES_ACTIVE=local ./gradlew bootRun
```

---

## 28. Cross-Platform Scripting for macOS and Windows

Cross-platform scripting requires discipline.

### Prefer these rules

- Use WSL2 for Bash on Windows.
- Use `#!/usr/bin/env bash`.
- Quote all variable expansions.
- Avoid GNU-only flags unless you install GNU tools on macOS.
- Avoid hardcoded absolute paths.
- Use Gradle wrapper or Maven wrapper.
- Normalize line endings to LF for `.sh` files.

### Detect OS

```bash
case "$(uname -s)" in
  Darwin)
    os="macos"
    ;;
  Linux)
    os="linux"
    ;;
  MINGW*|MSYS*|CYGWIN*)
    os="windows-git-bash"
    ;;
  *)
    os="unknown"
    ;;
esac

echo "$os"
```

### Path conversion

In Git Bash:

```bash
pwd -W
```

In WSL:

```bash
wslpath -w /mnt/c/Users/alex/project
wslpath -u 'C:\Users\alex\project'
```

### PowerShell equivalent examples

| Task | Bash | PowerShell |
| --- | --- | --- |
| Print text | `echo "hi"` | `Write-Output "hi"` |
| List files | `ls -la` | `Get-ChildItem -Force` |
| Current path | `pwd` | `Get-Location` |
| Env var | `$JAVA_HOME` | `$env:JAVA_HOME` |
| Set env var | `export X=1` | `$env:X = "1"` |
| Run command if success | `cmd && next` | `cmd; if ($?) { next }` |

When a team includes Windows developers who do not use WSL, consider providing
both:

- `scripts/dev.sh`
- `scripts/dev.ps1`

Keep core logic in Gradle tasks when possible so both scripts call the same
source of truth.

---

## 29. CI/CD Scripting

Good CI scripts:

- fail fast
- print useful context
- avoid secrets in logs
- are deterministic
- can run locally

Example CI entrypoint:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

trap 'echo "ERROR: line $LINENO: $BASH_COMMAND" >&2' ERR

echo "Java:"
java -version

echo "Gradle:"
./gradlew --version

./gradlew clean check
```

GitHub Actions usage:

```yaml
- name: Run checks
  shell: bash
  run: ./scripts/ci-check.sh
```

### CI environment variables

```bash
if [[ "${CI:-false}" == "true" ]]; then
  echo "Running in CI"
fi
```

GitHub Actions:

```bash
echo "$GITHUB_REF"
echo "$GITHUB_SHA"
echo "$GITHUB_HEAD_REF"
```

### Group logs in GitHub Actions

```bash
echo "::group::Gradle tests"
./gradlew test
echo "::endgroup::"
```

### Upload artifact path discovery

```bash
find . -type f \( -name "*.apk" -o -name "*.aab" -o -name "*.jar" \)
```

---

## 30. Security Practices

### Do not hardcode secrets

Bad:

```bash
TOKEN="abc123"
```

Better:

```bash
TOKEN="${TOKEN:?TOKEN environment variable is required}"
```

### Do not print secrets

Avoid:

```bash
set -x
curl -H "Authorization: Bearer $TOKEN" ...
```

If tracing is needed, disable around secret commands:

```bash
set +x
curl -fsS -H "Authorization: Bearer $TOKEN" "$url"
set -x
```

### Use secure input

```bash
read -r -s -p "Password: " password
echo
```

### Validate dangerous inputs

```bash
environment="${1:?Environment required}"

case "$environment" in
  local|dev|staging|prod)
    ;;
  *)
    echo "Invalid environment: $environment" >&2
    exit 1
    ;;
esac
```

### Avoid command injection

Bad:

```bash
eval "git checkout $branch"
```

Good:

```bash
git checkout -- "$branch"
```

Avoid `eval` unless there is no safer design.

### Use `--` before user-provided file names

```bash
rm -- "$file"
git checkout -- "$file"
```

This prevents a filename beginning with `-` from being treated as an option.

---

## 31. Performance and Reliability

### Avoid unnecessary subshells in loops

This is often slow:

```bash
for file in $(find . -name "*.java"); do
  echo "$file"
done
```

Better:

```bash
find . -name "*.java" -print0 |
  while IFS= read -r -d '' file; do
    echo "$file"
  done
```

### Parallel work

Use `xargs -P`:

```bash
printf '%s\n' app core feature-auth |
  xargs -n1 -P4 -I{} ./gradlew ":{}:test"
```

Use GNU Parallel if available:

```bash
parallel ./gradlew :{}:test ::: app core feature-auth
```

### Cache expensive results

```bash
changed_files_cache="$(mktemp)"
git diff --name-only origin/main...HEAD > "$changed_files_cache"
```

### Keep scripts idempotent

An idempotent script can run multiple times safely.

Good:

```bash
mkdir -p build/reports
docker compose up -d postgres
```

Risky:

```bash
mkdir build/reports
docker run --name postgres postgres
```

---

## 32. Reusable Script Architecture

Suggested project layout:

```text
scripts/
  lib/
    common.sh
    android.sh
    spring.sh
  build-android.sh
  run-api.sh
  ci-check.sh
```

`scripts/lib/common.sh`:

```bash
#!/usr/bin/env bash

log() {
  printf '[%s] %s\n' "$(date +'%Y-%m-%dT%H:%M:%S%z')" "$*"
}

die() {
  echo "ERROR: $*" >&2
  exit 1
}

require_command() {
  local command_name="$1"
  command -v "$command_name" >/dev/null 2>&1 || die "Missing command: $command_name"
}
```

`scripts/build-android.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"

# shellcheck source=scripts/lib/common.sh
source "$SCRIPT_DIR/lib/common.sh"

require_command adb

cd "$PROJECT_ROOT"
./gradlew assembleDebug
```

Guidelines:

- entrypoint scripts should use strict mode
- library files should not call `exit` except through explicit helpers like `die`
- keep side effects in `main`
- pass values as arguments instead of relying on global variables

---

## 33. Advanced Patterns

### Main function pattern

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

main() {
  echo "All logic starts here"
}

main "$@"
```

### Subcommands

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

usage() {
  cat <<'EOF'
Usage: dev COMMAND

Commands:
  android-build
  api-run
  test
EOF
}

android_build() {
  ./gradlew assembleDebug
}

api_run() {
  SPRING_PROFILES_ACTIVE=local ./gradlew bootRun
}

run_tests() {
  ./gradlew test
}

command="${1:-}"
shift || true

case "$command" in
  android-build) android_build "$@" ;;
  api-run) api_run "$@" ;;
  test) run_tests "$@" ;;
  -h|--help|"") usage ;;
  *) echo "Unknown command: $command" >&2; usage; exit 1 ;;
esac
```

### Lock files

Prevent two runs at the same time:

```bash
lock_file="/tmp/my-script.lock"

exec 9>"$lock_file"
if ! flock -n 9; then
  echo "Another instance is running" >&2
  exit 1
fi
```

`flock` may not be installed on macOS by default.

### Atomic deploy-style symlink switch

```bash
release_dir="/opt/app/releases/$VERSION"
current_link="/opt/app/current"

ln -sfn "$release_dir" "$current_link.new"
mv -Tf "$current_link.new" "$current_link"
```

macOS `mv` does not support all GNU flags. Test deployment scripts on the target
environment.

### Semantic version bump

```bash
bump_patch() {
  local version="$1"
  local major minor patch

  IFS=. read -r major minor patch <<< "$version"
  printf '%s.%s.%s\n' "$major" "$minor" "$((patch + 1))"
}

next_version="$(bump_patch "1.2.3")"
echo "$next_version"
```

### Running cleanup even on failure

```bash
tmp_dir="$(mktemp -d)"

cleanup() {
  local exit_code=$?
  rm -rf "$tmp_dir"
  exit "$exit_code"
}

trap cleanup EXIT
```

### Avoiding `cd` side effects

Use subshells:

```bash
(
  cd android-app
  ./gradlew test
)

(
  cd spring-api
  ./gradlew test
)
```

The parent shell remains in the original directory.

---

## 34. Cheat Sheets

### Daily terminal commands

```bash
pwd
ls -la
cd path
mkdir -p path
cp source target
mv source target
rm file
rm -rf dir
rg "text"
find . -name "*.java"
chmod +x script.sh
```

### Bash scripting essentials

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

name="${1:-default}"

if [[ -f "$name" ]]; then
  echo "file exists"
fi

for item in "$@"; do
  echo "$item"
done

my_function() {
  local value="$1"
  echo "$value"
}

main() {
  my_function "$name"
}

main "$@"
```

### Android

```bash
adb devices
adb logcat
adb install -r app-debug.apk
adb shell pm clear com.example.app
./gradlew assembleDebug
./gradlew connectedAndroidTest
./gradlew bundleRelease
```

### Spring Boot

```bash
./gradlew bootRun
./gradlew test
./gradlew bootJar
SPRING_PROFILES_ACTIVE=local ./gradlew bootRun
curl -fsS http://localhost:8080/actuator/health | jq .
lsof -i :8080
```

### Git

```bash
git status --short --branch
git diff
git diff --name-only
git branch --show-current
git log --oneline -n 10
```

---

## 35. Practice Roadmap

### Beginner exercises

1. Create a script that prints your current Java version.
2. Create a script that checks whether `adb`, `java`, and `git` are installed.
3. Create a script that creates `src/main/java`, `src/test/java`, and
   `src/main/resources`.
4. Create a script that runs `./gradlew test` and exits with the same status.

### Intermediate exercises

1. Create a script that accepts `--env local|dev|staging` and runs Spring Boot
   with that profile.
2. Create a script that builds a debug APK, installs it, and launches the app.
3. Create a script that starts Docker dependencies, waits for readiness, then
   starts the Spring Boot service.
4. Create a script that parses `curl /actuator/health` with `jq` and fails if
   the service is not `UP`.

### Advanced exercises

1. Build a `dev` CLI with subcommands:
   - `dev android build`
   - `dev android install`
   - `dev api run`
   - `dev test`
2. Add structured logging, strict mode, `trap` cleanup, and dependency checks.
3. Make the CLI work on both macOS and WSL.
4. Add ShellCheck and shfmt checks to CI.
5. Add safe secret handling for tokens used by local API calls.

### Mastery checklist

- You can explain stdout, stderr, stdin, and exit codes.
- You quote variables by default.
- You understand `set -Eeuo pipefail`.
- You can parse arguments with `case`.
- You can safely handle files with spaces.
- You can debug scripts with `bash -x` and ShellCheck.
- You can automate Android builds, installs, logs, and tests.
- You can automate Spring Boot profiles, health checks, and integration tests.
- You can write scripts that run locally and in CI.
- You know when not to use Bash and when to move logic into Gradle, Java,
  Kotlin, Python, Go, or another language.

---

## Final Advice

Use Bash as glue code. It is excellent for connecting tools: Gradle, adb, Git,
Docker, curl, jq, and CI runners. It is not ideal for large business logic,
complex data structures, or long-lived applications.

For Android and Spring Boot development, the best automation usually looks like
this:

- domain-specific work stays in Gradle, Java, Kotlin, or application code
- Bash orchestrates commands, checks prerequisites, and prepares environments
- CI uses the same scripts developers can run locally
- secrets are supplied by environment variables or secret managers
- scripts fail clearly and clean up after themselves
