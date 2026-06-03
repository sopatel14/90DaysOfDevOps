# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## Objective

Learn how to write cleaner and more maintainable shell scripts using:

* Functions
* Function arguments
* Local variables
* Strict mode (`set -euo pipefail`)
* Modular scripting practices

---

# Task 1: Basic Functions

## Script: functions.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat functions.sh
#!/bin/bash

greet() {
    echo "Hello, $1!"
}

add() {
    sum=$(( $1 + $2 ))
    echo "Sum is: $sum"
}

greet "Sourav"
add 10 20

```


### Purpose

Create reusable functions that:

* Greet a user by name
* Add two numbers and display the result

### Output

<img width="780" height="176" alt="image" src="https://github.com/user-attachments/assets/d1b3cbcb-6e83-4d5f-b653-96f5f59b2e17" />


### Observation

Functions help organize code into reusable blocks. Instead of writing the same commands repeatedly, a function can be called whenever needed.

---

# Task 2: Functions with Return Values

## Script: disk_check.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat disk_check.sh
#!/bin/bash

check_disk() {
    echo "Disk Usage:"
    df -h /
}

check_memory() {
    echo "Memory Usage:"
    free -h
 }

check_disk
echo
check_memory

```

### Purpose

Create separate functions to:

* Check disk usage
* Check memory usage

### Output

<img width="567" height="171" alt="image" src="https://github.com/user-attachments/assets/e23b5be3-eefe-484a-b216-c10c61d2b04a" />


```

### Observation

Breaking checks into separate functions makes scripts easier to read and maintain.

---

# Task 3: Strict Mode (`set -euo pipefail`)

## Script: strict_demo.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat strict_demo.sh
#!/bin/bash

set -euo pipefail

echo "=== Testing set -u ==="
echo $city

echo "=== Testing set -e ==="
cd /wrong-directory

echo "=== Testing pipefail ==="
cat missing_file.txt | grep hello

echo "Script completed"
```

### Output

<img width="475" height="67" alt="image" src="https://github.com/user-attachments/assets/e0aac8f2-452a-4006-bd6c-9eb6a590225f" />

### To test each option separately, create three small scripts.

## Script set-u.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat set-u.sh
#!/bin/bash

set -u

echo $city
```
#### Observation

The script stops because an undefined variable was used.

### Output

<img width="393" height="62" alt="image" src="https://github.com/user-attachments/assets/97973a15-3c18-4071-9910-6fe32e705c38" />

## Script set-e.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat set-e.sh
#!/bin/bash

set -e

echo "Before"

cd /wrong-directory

echo "After"
```

#### Observation

The script exits immediately after the failed command.

### Output

<img width="501" height="73" alt="image" src="https://github.com/user-attachments/assets/76142a3b-e70f-43fc-8337-9e40d12e32c2" />

## Script set-o-pipefail.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat set-o-pipefail.sh
#!/bin/bash

set -o pipefail

cat missing_file.txt | grep hello

echo $?

```
#### Observation

Without `pipefail`, some pipeline failures may go unnoticed.

### Output

<img width="477" height="69" alt="image" src="https://github.com/user-attachments/assets/0f851c80-fa11-4388-94a3-770248c898af" />


### Purpose

Understand how strict mode improves script reliability.

### Strict Mode

```bash
set -euo pipefail
```

### What Does Each Option Do?

| Option            | Description                                    |
| ----------------- | ---------------------------------------------- |
| `set -e`          | Exit immediately if a command fails            |
| `set -u`          | Treat undefined variables as errors            |
| `set -o pipefail` | Fail a pipeline if any command within it fails |

---

### Why Use Strict Mode?

Strict mode helps:

* Catch mistakes early
* Prevent unexpected behavior
* Make automation safer
* Improve troubleshooting

This is considered a best practice in production shell scripts.

---

# Task 4: Local Variables

## Script: local_demo.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat local_demo.sh
#!/bin/bash

show_local() {
    local name="Sourav"
    echo "Inside function: $name"
}

show_global() {
    role="DevOps Engineer"
}

show_local

echo "Outside function: $name"

show_global

echo "Outside function role: $role"

```


### Purpose

Understand the difference between local and global variables.

### Output

<img width="415" height="87" alt="image" src="https://github.com/user-attachments/assets/fa1dee0f-3d8c-437e-b923-1e6fc61553ba" />


### Observation

#### Local Variables

* Exist only within the function.
* Cannot be accessed outside the function.

#### Global Variables

* Remain available throughout the script.
* Can be accessed by multiple functions.

### Why Use Local Variables?

Using `local` prevents accidental modification of variables elsewhere in the script and makes functions more predictable.

---

# Task 5: System Information Reporter

## Script: system_info.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat system_info.sh
#!/bin/bash

set -euo pipefail

hostname_info() {
    echo "===== HOSTNAME & OS ====="
    hostname
    cat /etc/os-release | grep PRETTY_NAME
}

uptime_info() {
    echo
    echo "===== UPTIME ====="
    uptime
}

disk_info() {
    echo
    echo "===== DISK USAGE ====="
    du -sh /* 2>/dev/null | sort -hr | head -n 5 || true
}

memory_info() {
    echo
    echo "===== MEMORY USAGE ====="
    free -h
}

cpu_info() {
    echo
    echo "===== TOP CPU PROCESSES ====="
    ps aux --sort=-%cpu | head -6
}

main() {
    hostname_info
    uptime_info
    disk_info
    memory_info
    cpu_info
}

main

```

### Purpose

Create a reusable system reporting script using functions.

### Information Collected

* Hostname
* Operating System details
* System uptime
* Disk usage
* Memory usage
* Top CPU-consuming processes

### Output

<img width="690" height="475" alt="image" src="https://github.com/user-attachments/assets/967279b6-12c5-4c11-a64a-658371745199" />



### Observation

Creating separate functions for each section makes the script:

* Easier to understand
* Easier to maintain
* Easier to extend in the future

---

# Key Learnings

## 1. Functions Improve Reusability

Functions allow code to be written once and used multiple times throughout a script.

---

## 2. Local Variables Prevent Side Effects

Using `local` keeps variables confined to a function and avoids unexpected changes elsewhere.

---

## 3. Strict Mode Makes Scripts Safer

Using:

```bash
set -euo pipefail
```

helps catch errors early, prevents undefined variables from being used, and detects failures inside pipelines.

---

### You can also run the script in debug mode:

```bash
bash -x system_info.sh
```
# Conclusion

Today I learned how to write cleaner and more reliable shell scripts using functions and local variables. I explored strict mode (`set -euo pipefail`) and understood why it is commonly used in production automation. Finally, I combined multiple functions into a system information reporting script, demonstrating how larger scripts can be organized into reusable and maintainable components.
