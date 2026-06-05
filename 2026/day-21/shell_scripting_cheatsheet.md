# Shell Scripting Cheat Sheet

## Quick Reference Table

| Topic        | Key Syntax               | Example                            |
| ------------ | ------------------------ | ---------------------------------- |
| Variable     | `VAR="value"`            | `NAME="DevOps"`                    |
| Argument     | `$1`, `$2`               | `./script.sh arg1`                 |
| If Statement | `if [ condition ]; then` | `if [ -f file ]; then`             |
| For Loop     | `for i in list; do`      | `for i in 1 2 3; do`               |
| Function     | `name() { ... }`         | `greet() { echo "Hi"; }`           |
| Grep         | `grep pattern file`      | `grep -i "error" log.txt`          |
| Awk          | `awk '{print $1}' file`  | `awk -F: '{print $1}' /etc/passwd` |
| Sed          | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt`  |

---

# 1. Basics

## Shebang

Tells Linux which interpreter should execute the script.

```bash
#!/bin/bash
echo "Hello World"
```

---

## Running a Script

Make executable:

```bash
chmod +x script.sh
```

Run directly:

```bash
./script.sh
```

Run with bash:

```bash
bash script.sh
```

---

## Comments

Single-line comment:

```bash
# This is a comment
```

Inline comment:

```bash
echo "Hello" # Print greeting
```

---

## Variables

Declare variable:

```bash
NAME="DevOps"
```

Use variable:

```bash
echo $NAME
```

Quoted variable:

```bash
echo "$NAME"
```

Literal string:

```bash
echo '$NAME'
```

---

## Reading User Input

```bash
read -p "Enter your name: " NAME
echo "Welcome $NAME"
```

---

## Command-Line Arguments

```bash
echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "Total args: $#"
echo "All args: $@"
echo "Last exit status: $?"
```

Run:

```bash
./script.sh devops linux
```

---

# 2. Operators and Conditionals

## String Comparisons

Equal:

```bash
[ "$A" = "$B" ]
```

Not equal:

```bash
[ "$A" != "$B" ]
```

Empty string:

```bash
[ -z "$VAR" ]
```

Not empty:

```bash
[ -n "$VAR" ]
```

---

## Integer Comparisons

Equal:

```bash
[ "$A" -eq "$B" ]
```

Not equal:

```bash
[ "$A" -ne "$B" ]
```

Less than:

```bash
[ "$A" -lt "$B" ]
```

Greater than:

```bash
[ "$A" -gt "$B" ]
```

Less than or equal:

```bash
[ "$A" -le "$B" ]
```

Greater than or equal:

```bash
[ "$A" -ge "$B" ]
```

---

## File Test Operators

File exists:

```bash
[ -e file.txt ]
```

Regular file:

```bash
[ -f file.txt ]
```

Directory:

```bash
[ -d mydir ]
```

Readable:

```bash
[ -r file.txt ]
```

Writable:

```bash
[ -w file.txt ]
```

Executable:

```bash
[ -x script.sh ]
```

Not empty:

```bash
[ -s file.txt ]
```

---

## if / elif / else

```bash
if [ "$AGE" -ge 18 ]; then
    echo "Adult"
elif [ "$AGE" -ge 13 ]; then
    echo "Teen"
else
    echo "Child"
fi
```

---

## Logical Operators

AND:

```bash
[ -f file.txt ] && echo "Exists"
```

OR:

```bash
[ -f file.txt ] || echo "Missing"
```

NOT:

```bash
if ! [ -f file.txt ]; then
    echo "File not found"
fi
```

---

## Case Statement

```bash
case $1 in
    start)
        echo "Starting"
        ;;
    stop)
        echo "Stopping"
        ;;
    *)
        echo "Usage: start|stop"
        ;;
esac
```

---

# 3. Loops

## For Loop (List Based)

```bash
for i in 1 2 3 4 5
do
    echo "$i"
done
```

---

## For Loop (C Style)

```bash
for ((i=1;i<=5;i++))
do
    echo "$i"
done
```

---

## While Loop

```bash
COUNT=1

while [ $COUNT -le 5 ]
do
    echo "$COUNT"
    ((COUNT++))
done
```

---

## Until Loop

Runs until condition becomes true.

```bash
COUNT=1

until [ $COUNT -gt 5 ]
do
    echo "$COUNT"
    ((COUNT++))
done
```

---

## break

```bash
for i in 1 2 3 4 5
do
    [ "$i" -eq 3 ] && break
    echo "$i"
done
```

---

## continue

```bash
for i in 1 2 3 4 5
do
    [ "$i" -eq 3 ] && continue
    echo "$i"
done
```

---

## Loop Over Files

```bash
for file in *.log
do
    echo "$file"
done
```

---

## Loop Over Command Output

```bash
cat users.txt | while read line
do
    echo "$line"
done
```

---

# 4. Functions

## Define Function

```bash
greet() {
    echo "Hello"
}
```

---

## Call Function

```bash
greet
```

---

## Function Arguments

```bash
greet() {
    echo "Hello $1"
}

greet Sourav
```

---

## Return vs Echo

Return exit status:

```bash
check() {
    return 0
}
```

Return data:

```bash
get_name() {
    echo "Sourav"
}
```

---

## Local Variables

```bash
greet() {
    local NAME="Sourav"
    echo "$NAME"
}
```

---

# 5. Text Processing Commands

## grep

Search text:

```bash
grep "error" app.log
```

Ignore case:

```bash
grep -i "error" app.log
```

Recursive:

```bash
grep -r "TODO" .
```

Count matches:

```bash
grep -c "error" app.log
```

Show line numbers:

```bash
grep -n "error" app.log
```

Invert match:

```bash
grep -v "INFO" app.log
```

Extended regex:

```bash
grep -E "error|warning" app.log
```

---

## awk

Print first column:

```bash
awk '{print $1}' file.txt
```

Custom delimiter:

```bash
awk -F: '{print $1}' /etc/passwd
```

Pattern matching:

```bash
awk '/error/ {print $0}' app.log
```

BEGIN/END:

```bash
awk 'BEGIN {print "Start"} {print $1} END {print "End"}' file.txt
```

---

## sed

Replace text:

```bash
sed 's/old/new/g' file.txt
```

Delete line:

```bash
sed '5d' file.txt
```

In-place edit:

```bash
sed -i 's/foo/bar/g' file.txt
```

---

## cut

```bash
cut -d: -f1 /etc/passwd
```

---

## sort

Alphabetical:

```bash
sort file.txt
```

Numerical:

```bash
sort -n numbers.txt
```

Reverse:

```bash
sort -r file.txt
```

Unique:

```bash
sort -u file.txt
```

---

## uniq

Remove duplicates:

```bash
uniq file.txt
```

Count duplicates:

```bash
uniq -c file.txt
```

---

## tr

Lowercase to uppercase:

```bash
echo "devops" | tr 'a-z' 'A-Z'
```

Delete characters:

```bash
echo "abc123" | tr -d '0-9'
```

---

## wc

```bash
wc file.txt
```

Lines only:

```bash
wc -l file.txt
```

Words only:

```bash
wc -w file.txt
```

Characters only:

```bash
wc -m file.txt
```

---

## head / tail

First 10 lines:

```bash
head file.txt
```

Last 10 lines:

```bash
tail file.txt
```

Follow log:

```bash
tail -f app.log
```

---

# 6. Useful Patterns and One-Liners

## Delete Files Older Than 30 Days

```bash
find /backup -type f -mtime +30 -delete
```

## Count Lines in All Log Files

```bash
wc -l *.log
```

## Replace Text Across Multiple Files

```bash
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;
```

## Check if a Service is Running

```bash
systemctl is-active nginx
```

## Disk Usage Alert

```bash
df -h | awk '$5+0 > 80 {print "Warning:", $0}'
```

## Parse CSV

```bash
awk -F, '{print $1,$3}' users.csv
```

## Parse JSON

```bash
jq '.name' file.json
```

## Monitor Errors in Real Time

```bash
tail -f app.log | grep --line-buffered ERROR
```

---

# 7. Error Handling and Debugging

## Exit Codes

Success:

```bash
exit 0
```

Failure:

```bash
exit 1
```

Check last command:

```bash
echo $?
```

---

## set -e

Exit immediately on error.

```bash
set -e
```

---

## set -u

Treat unset variables as errors.

```bash
set -u
```

---

## set -o pipefail

Fail if any command in pipeline fails.

```bash
set -o pipefail
```

---

## set -x

Debug mode.

```bash
set -x
```

---

## Strict Mode

Recommended for production scripts.

```bash
set -euo pipefail
```

---

## Trap Cleanup

```bash
cleanup() {
    echo "Cleaning up..."
}

trap cleanup EXIT
```

---

# DevOps Script Template

```bash
#!/bin/bash

set -euo pipefail

cleanup() {
    echo "Cleaning up..."
}

trap cleanup EXIT

echo "Starting script..."
```

---

# Quick Revision Checklist

* Shebang (`#!/bin/bash`)
* Variables and Arguments
* User Input (`read`)
* Conditionals (`if`, `case`)
* Loops (`for`, `while`, `until`)
* Functions and Local Variables
* Text Processing (`grep`, `awk`, `sed`)
* Error Handling (`set -euo pipefail`)
* Debugging (`set -x`)
* Cleanup (`trap`)
* Real-world One-Liners
