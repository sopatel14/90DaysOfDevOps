# Day 20 – Log Analyzer and Report Generator


## Objective

Create a Bash script that analyzes system log files and generates a summary report.

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat log_analyzer.sh
#!/bin/bash


# Task 1 - Input Validation

if [ $# -eq 0 ]; then
    echo "Error: Please provide a log file."
    echo "Usage: $0 <logfile>"
    exit 1
fi

LOG_FILE="$1"

if [ ! -f "$LOG_FILE" ]; then
    echo "Error: File '$LOG_FILE' does not exist."
    exit 1
fi

DATE=$(date +%Y-%m-%d)
REPORT_FILE="log_report_${DATE}.txt"

echo "Analyzing: $LOG_FILE"
echo

# Task 2 - Error Count

ERROR_COUNT=$(grep -Ei "ERROR|Failed" "$LOG_FILE" | wc -l)

echo "Total Errors/Failed Events: $ERROR_COUNT"
echo

# Task 3 - Critical Events

echo "--- Critical Events ---"

CRITICAL_EVENTS=$(grep -ni "CRITICAL" "$LOG_FILE")

if [ -z "$CRITICAL_EVENTS" ]; then
    echo "No critical events found."
else
    echo "$CRITICAL_EVENTS"
fi

echo

# Task 4 - Top Error Messages

echo "--- Top 5 Error Messages ---"

TOP_ERRORS=$(grep "ERROR" "$LOG_FILE" \
    | sed 's/.*ERROR[: ]*//' \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -5)

if [ -z "$TOP_ERRORS" ]; then
    echo "No ERROR messages found."
else
    echo "$TOP_ERRORS"
fi

# Task 5 - Summary Report

TOTAL_LINES=$(wc -l < "$LOG_FILE")

{
echo "========================================"
echo "LOG ANALYSIS REPORT"
echo "========================================"
echo "Date of Analysis : $DATE"
echo "Log File         : $LOG_FILE"
echo "Total Lines      : $TOTAL_LINES"
echo "Total Errors     : $ERROR_COUNT"
echo

echo "Top 5 Error Messages"
echo "--------------------"

if [ -z "$TOP_ERRORS" ]; then
    echo "No ERROR messages found."
else
    echo "$TOP_ERRORS"
fi

echo
echo "Critical Events"
echo "---------------"

if [ -z "$CRITICAL_EVENTS" ]; then
    echo "No critical events found."
else
    echo "$CRITICAL_EVENTS"
fi

} > "$REPORT_FILE"

echo
echo "Report generated: $REPORT_FILE"

# Task 6 - Archive Processed Logs

mkdir -p archive

mv "$LOG_FILE" archive/

echo "Log file moved to archive/"

```

### Task 1 – Input Validation

* Accepts log file as a command-line argument
* Checks if argument is provided
* Checks if file exists

### Task 2 – Error Count

Used:

```bash
grep -Ei "ERROR|Failed"
wc -l
```

Counts all lines containing ERROR or Failed.

### Task 3 – Critical Events

Used:

```bash
grep -ni "CRITICAL"
```

Displays critical events with line numbers.

### Task 4 – Top Error Messages

Used:

```bash
grep "ERROR"
sort
uniq -c
sort -rn
head -5
```

Finds the five most common error messages.

### Task 5 – Summary Report

Generates:

```bash
log_report_<date>.txt
```

Contains:

* Analysis date
* Log file name
* Total lines processed
* Error count
* Top 5 errors
* Critical events

### Task 6 – Archive Processed Logs

Used:

```bash
mkdir -p archive
mv logfile archive/
```

Moves analyzed logs into archive/.

## Output

<img width="1160" height="846" alt="image" src="https://github.com/user-attachments/assets/657e19a2-9a3b-4796-8108-58231205f86f" />

<img width="501" height="83" alt="image" src="https://github.com/user-attachments/assets/512965e8-0495-41a7-95e3-192030af77fb" />



## Commands and Tools Used

* grep
* sed
* wc
* sort
* uniq
* head
* mkdir
* mv
* date

## Key Learnings

1. grep can search and filter logs quickly.
2. sort and uniq help identify recurring issues.
3. Bash scripts can automate repetitive log analysis tasks.
