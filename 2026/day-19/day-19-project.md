# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

## Scripts in This Project

| File | Purpose |
|------|---------|
| `log_rotate.sh` | Compress old logs, purge ancient archives |
| `backup.sh` | Generic timestamped tar.gz backup |
| `backup_scripts.sh` | Capstone-style backup for `/home/ubuntu/scripts` |
| `maintenance.sh` | Orchestrator — calls rotation + backup, logs everything |

---

## Task 1: Log Rotation Script — `log_rotate.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat log_rotate.sh
#!/bin/bash

set -euo pipefail

if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <log_directory>" >&2
    exit 1
fi

LOG_DIR="$1"

if [[ ! -d "$LOG_DIR" ]]; then
    echo "ERROR: Directory '$LOG_DIR' does not exist." >&2
    exit 1
fi

echo "=== Log Rotation: $(date '+%Y-%m-%d %H:%M:%S') ==="
echo "Directory : $LOG_DIR"

# Compress .log files older than 7 days
COMPRESSED=0
while IFS= read -r -d '' file; do
    gzip "$file" && echo "  [GZIP] $file" && (( COMPRESSED++ )) || true
done < <(find "$LOG_DIR" -maxdepth 1 -name "*.log" -mtime +7 -print0)

# Delete .gz files older than 30 days
DELETED=0
while IFS= read -r -d '' file; do
    rm -f "$file" && echo "  [DEL]  $file" && (( DELETED++ )) || true
done < <(find "$LOG_DIR" -maxdepth 1 -name "*.gz" -mtime +30 -print0)

echo ""
echo "Done. Files compressed : $COMPRESSED"
echo "      Files deleted    : $DELETED"

```

### Sample Output

<img width="520" height="116" alt="image" src="https://github.com/user-attachments/assets/4c61ccff-7aa1-4f54-96c9-76dc01e34929" />

---


## Task 2 — `backup.sh`


```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat backup.sh
#!/bin/bash
set -e

SOURCE_DIR="/home/ubuntu/scripts"
BACKUP_DIR="${BACKUP_DIR:-$HOME/shell-backups}"
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)
ARCHIVE="$BACKUP_DIR/practice-scripts_${TIMESTAMP}.tar.gz"

prepare_backup_dir() {
    if [[ ! -d "$BACKUP_DIR" ]]; then
        mkdir -p "$BACKUP_DIR"
        echo "Backup folder bana diya: $BACKUP_DIR"
    fi
}

take_backup() {
    if [[ ! -d "$SOURCE_DIR" ]]; then
        echo "ERROR: Source '$SOURCE_DIR' nahi mila." >&2
        exit 1
    fi
    tar -czf "$ARCHIVE" -C "$(dirname "$SOURCE_DIR")" "$(basename "$SOURCE_DIR")"
    echo "Backup ban gaya: $ARCHIVE"
    echo "Size: $(du -h "$ARCHIVE" | awk '{print $1}')"
}

cleanup_old_backups() {
    cd "$BACKUP_DIR"
    ls -1t practice-scripts_*.tar.gz 2>/dev/null | tail -n +14 | xargs -r rm -f
    echo "Cleanup done — sirf last 14 backups rakhe gaye."
}

prepare_backup_dir
take_backup
cleanup_old_backups

echo ""
echo "Done. Saare backups:"
ls -lh "$BACKUP_DIR"/practice-scripts_*.tar.gz
```

### Sample Output

<img width="657" height="560" alt="image" src="https://github.com/user-attachments/assets/6c18b9c7-6ee5-4629-baab-fefe17b83892" />


---

## Task 3: Crontab

### Cron Syntax Refresher

```
* * * * *  command
│ │ │ │ │
│ │ │ │ └── Day of week  (0–7, 0 and 7 = Sunday)
│ │ │ └──── Month        (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour         (0–23)
└────────── Minute       (0–59)
```

### Cron Entries


<img width="660" height="240" alt="image" src="https://github.com/user-attachments/assets/960474f2-433c-4904-accc-b53fc0c5f87d" />


> **Note:** Always use absolute paths in cron — cron has a minimal `$PATH`. Redirect stdout and stderr so you have a log to inspect if something breaks.

### View Current Crontab

```bash
crontab -l        # list current entries
crontab -e        # open editor to add/edit
crontab -r        # DANGER: removes all entries
```

---

## Task 4: Maintenance Script — `maintenance.sh`

Calls log rotation and backup, logs everything with timestamps.

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat maintenance.sh
#!/bin/bash

set -uo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
LOG_DIR="${LOG_DIR:-/var/log/myapp}"
SOURCE_DIR="${SOURCE_DIR:-/home/ubuntu/scripts}"
BACKUP_DIR="${BACKUP_DIR:-$HOME/shell-backups}"
MAINTENANCE_LOG="/var/log/maintenance.log"

if [[ ! -w /var/log ]]; then
    MAINTENANCE_LOG="$HOME/maintenance.log"
fi

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') | $*" | tee -a "$MAINTENANCE_LOG"
}

run_log_rotation() {
    log "--- Log Rotation START ---"
    if bash "$SCRIPT_DIR/log_rotate.sh" "$LOG_DIR" >> "$MAINTENANCE_LOG" 2>&1; then
        log "--- Log Rotation OK ---"
    else
        log "--- Log Rotation FAILED (exit $?) ---"
    fi
}

run_backup() {
    log "--- Backup START ---"
    if bash "$SCRIPT_DIR/backup.sh" "$SOURCE_DIR" "$BACKUP_DIR" >> "$MAINTENANCE_LOG" 2>&1; then
        log "--- Backup OK ---"
    else
        log "--- Backup FAILED (exit $?) ---"
    fi
}

log "====== Maintenance job started ======"
run_log_rotation
run_backup
log "====== Maintenance job complete ======"
```

### Cron Entry

```cron
0 1 * * * /home/ubuntu/scripts/maintenance.sh
```

### Sample Output

<img width="1070" height="270" alt="image" src="https://github.com/user-attachments/assets/3aa44bbc-01d4-4ac5-af04-2c7e6bd09323" />

---

## What I Learned

**1. `find -print0` + `read -d ''` is the safe way to handle filenames**  
Filenames can contain spaces or newlines. Piping `find` output into a `for` loop breaks on spaces. The null-delimiter combo (`-print0` + `while IFS= read -r -d ''`) handles any filename safely.

**2. Count-based vs time-based cleanup are different strategies**  
`backup_scripts.sh` keeps the last 7 backups using `ls -1t | tail -n +8 | xargs rm` — this is count-based and immune to clock issues. `backup.sh` uses `find -mtime +14` — time-based, simpler to reason about for retention policies.

**3. Always use absolute paths in cron, and always redirect output**  
Cron runs with a stripped-down environment (`PATH=/usr/bin:/bin`). Scripts that work in your shell can silently fail in cron if they rely on `~/`, aliases, or tools only in `/usr/local/bin`. Redirecting to a log file (`>> /var/log/backup.log 2>&1`) is the only way to debug cron failures.
