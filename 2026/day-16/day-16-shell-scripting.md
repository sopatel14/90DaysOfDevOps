# Day 16 – Shell Scripting Basics

## Objective

Learn the fundamentals of shell scripting using Bash:

* Shebang (`#!/bin/bash`)
* Variables
* User input with `read`
* Conditional statements (`if-else`)
* Basic automation using scripts

---

# Task 1: First Script

### Script

`hello.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat hello.sh


echo "Hello, DevOps, This is Sourav"
ubuntu@ip-172-31-33-19:~/scripts$

```

### Output

<img width="724" height="206" alt="image" src="https://github.com/user-attachments/assets/2aca4cdc-c766-4686-8fc2-cb89bd66b1da" />


### What Happens If We Remove the Shebang?

The shebang (`#!/bin/bash`) tells Linux which interpreter should execute the script.

Without it:

* The script may still run if executed using `bash script.sh`
* Running `./script.sh` can fail or use a different shell depending on the environment
* It is considered a best practice to always include the shebang

---

# Task 2: Variables

### Script

`variables.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat variable.sh
#!/bin/bash

name='Sourav'
role='DevOps Engineer'

echo 'Hello I am $name and I am a $role'
```

### Output

<img width="810" height="178" alt="image" src="https://github.com/user-attachments/assets/59fba23e-ac9e-4706-b24c-9c3f00d0eb78" />


### Single Quotes vs Double Quotes

#### Single Quotes

```bash
echo '$NAME'
```

Output:

```bash
$NAME
```

#### Double Quotes

```bash
echo "$NAME"
```

Output:

```bash
Sourav
```

### Observation

* Single quotes treat everything as plain text.
* Double quotes allow variable expansion.

---

# Task 3: User Input with read

### Script

`greet.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat greet.sh
#!/bin/bash


read -p "what is your name?" name

read -p "what is your favourite tool" tool

echo "My name is $name, and my favourite tool is $tool"
```

### Output

<img width="966" height="318" alt="image" src="https://github.com/user-attachments/assets/9dec154a-48f8-4b48-8a6f-5faaa849542b" />


### Observation

The `read` command allows scripts to accept user input during execution.

---

# Task 4: If-Else Conditions

## Script 1

### File

`check_number.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat check_number.sh
#!/bin/bash


read -p "The number is: " number

if [ "$number" -gt 0 ]; then
	echo "The number is positive"
elif [ "$number" -lt 0 ]; then
	echo "The number is negative"
else
	echo "The number is zero"
fi
```

### Output

<img width="958" height="446" alt="image" src="https://github.com/user-attachments/assets/dd2946a5-bc35-4d37-aa37-31d278abccb5" />


### Observation

Conditional statements can be used to make decisions based on user input.

---

## Script 2

### File

`file_check.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat file_check.sh
#!/bin/bash

read -p "Enter your filename" file

if [ -f "$file" ]; then
	echo "file exists"
else
	echo "file does not exists"
fi

```


### Output

<img width="934" height="312" alt="image" src="https://github.com/user-attachments/assets/b145af53-ab8b-45be-9819-88d44b32eed5" />


### Observation

The `-f` operator checks whether a regular file exists.

---

# Task 5: Combine It All

### Script

`server_check.sh`

```bash
ubuntu@ip-172-31-33-19:~/scripts$ cat server_check.sh
#!/bin/bash

service="nginx"

read -p "Do you want to check the status? (y/n): " choice

if [ "$choice" = "y" ]; then
    systemctl status $service

    if systemctl is-active --quiet $service
    then
        echo "$service is active"
    else
        echo "$service is not active"
    fi
else
    echo "Skipped."
fi
```

### Output

<img width="1826" height="914" alt="image" src="https://github.com/user-attachments/assets/592208e1-4336-40db-a2df-a422e040de28" />



### Observation

This script combines:

* Variables
* User input
* Conditional statements
* Service status checks

It demonstrates a simple real-world automation task.

---

# Key Learnings

### 1. Shebang Defines the Interpreter

`#!/bin/bash` tells Linux to execute the script using the Bash shell.

### 2. Variables Make Scripts Reusable

Variables help store values that can be reused throughout the script.

### 3. Conditions Enable Automation

Using `if`, `elif`, and `else` allows scripts to make decisions and automate tasks based on different situations.

---

# Conclusion

Today I learned the core building blocks of shell scripting. I practiced creating executable scripts, working with variables and user input, using conditional statements, checking file existence, and performing basic service monitoring. These concepts form the foundation for writing automation scripts in DevOps environments.
