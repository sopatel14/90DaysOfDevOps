# Day 17 – Shell Scripting: Loops, Arguments & Error Handling

## Objective

Build practical Bash scripting skills using:

* For loops
* While loops
* Command-line arguments
* Package installation automation
* Error handling techniques

---

# Task 1: For Loops

## Script: for_loop.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat for_loop.sh
#!/bin/bash

for item in apple mango cherries litchies pineapple
do
	echo "Fruits:  $item"
done
```

### Purpose

Loop through a list of five fruits and print each fruit.

### Output

<img width="389" height="124" alt="image" src="https://github.com/user-attachments/assets/16ca9441-6198-4008-9f2e-ea3018dfec74" />


### Observation

The `for` loop is useful when performing the same action on multiple items.

---

## Script: count.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat count.sh
#!/bin/bash

for i in $(seq 1 10);
do
	echo $i
done
```

### Purpose

Print numbers from 1 to 10 using a for loop.

### Output

<img width="346" height="179" alt="image" src="https://github.com/user-attachments/assets/70f4db18-fe64-4935-8efd-0498f454b278" />


### Observation

A `for` loop can be used to repeat actions a fixed number of times.

---

# Task 2: While Loop

## Script: countdown.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat countdown.sh
#!/bin/bash

read -p "Enter a number: " num

while [ $num -ge 0 ]
do
    echo $num
    num=$((num - 1))
done

echo "Done"
```

### Purpose

Accept a number from the user and count down to zero.

### Output

<img width="394" height="225" alt="image" src="https://github.com/user-attachments/assets/7d8a6d57-5461-4a8c-8608-3925e1c6db4f" />



### Observation

A `while` loop continues executing as long as the specified condition remains true.

---

# Task 3: Command-Line Arguments

## Script: greet1.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat greet1.sh
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "Usage: ./greet.sh <name>"
else
    echo "Hello, $1!"
fi
```
### Purpose

Accept a username as a command-line argument and display a greeting.

### Example Execution

```bash
./greet.sh Sourav
```

### Output

<img width="936" height="180" alt="image" src="https://github.com/user-attachments/assets/ab05bd6c-9faa-40bb-97e3-4caea98ce9a4" />


### Observation

Command-line arguments allow scripts to receive input without prompting the user.

---

## Script: args_demo.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat args_demo.sh
#!/bin/bash

echo "Script name: $0"
echo "Total arguments: $#"
echo "All arguments: $@"
```

### Purpose

Display information about command-line arguments.

### Example Execution

```bash
./args_demo.sh apple mango honey
```

### Output

<img width="1034" height="260" alt="image" src="https://github.com/user-attachments/assets/663680f5-f236-4dfd-9f8a-74f7454b2af0" />


### Observation

* `$0` contains the script name.
* `$#` contains the total number of arguments.
* `$@` contains all arguments passed to the script.

---

# Task 4: Install Packages via Script

## Script: install_packages.sh

```bash
packages="nginx curl wget"

for package in $packages
do
    if dpkg -s $package >/dev/null 2>&1
    then
        echo "$package is already installed. Skipping..."
    else
        echo "$package is not installed. Installing..."
        sudo apt install -y $package
    fi
done
```

### Purpose

Automate package installation for:

* nginx
* curl
* wget

### Functionality

* Loops through the package list.
* Checks whether each package is already installed.
* Installs missing packages.
* Skips packages that are already present.
* Verifies that the script is run as root before proceeding.

### Output

<img width="1018" height="206" alt="image" src="https://github.com/user-attachments/assets/e5e1528a-c666-4707-93a3-de0f607aefce" />


### Observation

Automating package installation ensures consistency and saves time during server provisioning.

---

# Task 5: Error Handling

## 1.Script: safe_script.sh

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat safe_script.sh
#!/bin/bash

set -e

mkdir /tmp/devops-test || echo "Directory already exists"

cd /tmp/devops-test123 || echo "Failed to enter directory"

touch test.txt || echo "Failed to create file"

echo "Script completed successfully"
```

### Purpose

Perform filesystem operations while handling errors safely.

### Operations Performed

1. Create a directory.
2. Navigate into the directory.
3. Create a file.
4. Display custom messages when errors occur.

### Output

<img width="1124" height="328" alt="image" src="https://github.com/user-attachments/assets/03a930f8-74e9-4230-a35d-b18ab4f385d9" />

## 2. Modify your install_packages.sh to check if the script is being run as root — exit with a message if not.

```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]
then
    echo "Please run this script as root or using sudo."
    exit 1
fi


packages="nginx curl wget"

for package in $packages
do
    if dpkg -s $package >/dev/null 2>&1
    then
        echo "$package is already installed. Skipping..."
    else
        echo "$package is not installed. Installing..."
        sudo apt install -y $package
    fi
done
```

### Output

<img width="988" height="252" alt="image" src="https://github.com/user-attachments/assets/274e5f2f-8a3b-4427-9db8-34141fa53b31" />



### Observation

* `set -e` helps stop scripts when unexpected errors occur.
* `||` provides fallback actions when commands fail.
* Error handling improves script reliability and troubleshooting.
* Package installation requires administrative privileges.
* The script checks the current user's Effective User ID (EUID).
* If the user is not root, the script exits safely before making any system changes.
* This prevents permission-related failures during package installation.
---

# Key Learnings

## 1. Loops Reduce Repetition

Both `for` and `while` loops help automate repetitive tasks without writing the same commands multiple times.

---

## 2. Command-Line Arguments Make Scripts Flexible

Using variables such as `$0`, `$1`, `$#`, and `$@` allows scripts to accept dynamic input and behave differently based on user-provided values.

---

## 3. Error Handling Makes Scripts Safer

Using `set -e`, root-user checks, and fallback actions with `||` helps prevent unexpected failures and improves script stability.

---

# Conclusion

Day 17 introduced more practical shell scripting concepts used in real-world DevOps environments. I learned how to automate repetitive tasks with loops, pass values using command-line arguments, automate package installation, verify permissions, and handle errors effectively. These concepts are essential for building reliable automation scripts and server management workflows.
