# Day 06 – Linux Fundamentals: Read and Write Text Files

## Creating a File

### 1. Create an empty file

```bash
touch notes.txt
```

Observation:
Created a new empty file named notes.txt.

---

## Writing Data into File

### 2. Add first line using overwrite operator

```bash
echo "This is Day 06 of 90 Days Of DevOps Challenge" > notes.txt
```


Observation:
Created first line inside the file using `>`.

---

### 3. Append second line

```bash
echo "Practicing redirection operators" >> notes.txt
```

Output:

<img width="569" height="124" alt="image" src="https://github.com/user-attachments/assets/5aa76fda-5a4b-4037-980f-1b720d8dc56e" />


Observation:
Added a new line without overwriting existing content.


---

### 4. Append third line using tee

```bash
echo "Linux Fundamentals" | tee -a notes.txt
```
Output:

<img width="537" height="105" alt="image" src="https://github.com/user-attachments/assets/118c5387-77e7-4931-9ccf-d5af7561a88f" />


Observation:
Displayed output on terminal and appended it into the file simultaneously.

---

### 5. Append additional lines

```bash
echo "Reading files with cat command" >> notes.txt
echo "Using head and tail commands" >> notes.txt
echo "Linux fundamentals are important for DevOps" >> notes.txt
```

Observation:
Added more lines to make the file content complete.

---

## Reading File Content

### 6. Display full file

```bash
cat notes.txt
```

Output:

<img width="473" height="122" alt="image" src="https://github.com/user-attachments/assets/f976902b-ad54-428c-a20a-48e9b8e54c90" />


Observation:
Displayed the complete contents of the file.

---

### 7. Read first 2 lines

```bash
cat notes.txt | head -n 2
```
Output:

<img width="487" height="79" alt="image" src="https://github.com/user-attachments/assets/0cf86a7a-8564-44db-92a5-77de81d0989d" />



Observation:
Viewed only the beginning section of the file.

---

### 8. Read last 2 lines

```bash
cat notes.txt | tail -n 2
```

Output:

<img width="515" height="53" alt="image" src="https://github.com/user-attachments/assets/875a7266-b8b4-4eb7-9d3a-6e260f406e8e" />


Observation:
Viewed the latest appended lines from the file.

---

# Final File Content

```text
This is Day 06 of 90 Days Of DevOps Challenge
Practicing redirection operators
Linux Fundamentals
Reading files with cat command
Using head and tail commands
Linux fundamentals are important for DevOps
```

---

# What I Learned

* How to create files using touch
* Difference between > and >>
* How tee works for writing and displaying output
* How to read complete and partial file contents
* Basic file handling operations in Linux
