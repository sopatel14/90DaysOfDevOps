# Day 13 – LVM (Logical Volume Manager)


Learn how Linux storage management works using LVM.

Understand:
- Physical Volumes (PV)
- Volume Groups (VG)
- Logical Volumes (LV)
- Formatting and mounting storage
- Extending storage dynamically

---

# What is LVM?

LVM (Logical Volume Manager) allows flexible storage management in Linux.

Instead of fixed partitions, LVM lets you:
- Create flexible storage pools
- Resize storage later
- Extend volumes without recreating partitions

---



Create the file in /home/ubuntu 

Run this:

```bash
dd if=/dev/zero of=/home/ubuntu/disk1.img bs=1M count=1024
```

Output:

<img width="1138" height="202" alt="image" src="https://github.com/user-attachments/assets/71691461-069b-42b3-aa91-54babae73710" />

Perfect — now your fake disk is successfully attached as a loop device.

The important line is:

```bash
/dev/loop3: []: (/home/ubuntu/disk1.img)
```

This means:

| Part                     | Meaning                                      |
| ------------------------ | -------------------------------------------- |
| `/home/ubuntu/disk1.img` | your fake 1 GB disk file                     |
| `/dev/loop3`             | Linux converted it into a usable disk device |

So now Linux treats `disk1.img` like a real hard disk.

---

# What is a Loop Device?

A loop device lets Linux use a normal file as if it were a real disk.

Normally:

```text
Real Disk → /dev/sda or /dev/nvme0n1
```

But here:

```text
disk1.img file → /dev/loop3
```

So:

* No real extra disk needed
* Great for practice/testing

---

# Why do you already have loop0, loop1, loop2?

These are from Snap packages.

Example:

```bash
/dev/loop1 → core22 snap package
/dev/loop2 → snapd package
```

Ubuntu mounts snap packages as read-only loop devices.

Ignore them.

Your important one is:

```bash
/dev/loop3
```

---

# Task 1 — Check Current Storage

Run:

```bash
lsblk
```
Output :

<img width="510" height="231" alt="image" src="https://github.com/user-attachments/assets/f800bc4c-dc19-49f4-b329-b73cea7cdf0e" />



Now you should see something like:

```bash
NAME         SIZE TYPE
loop3          1G loop
nvme0n1       15G disk
```

Meaning:

* `loop3` = your fake disk
* `nvme0n1` = actual EC2 disk

---

Run:

Switch to sudo su

```bash
pvs
vgs
lvs
df -h
```

Output:

<img width="571" height="373" alt="image" src="https://github.com/user-attachments/assets/0f5ea503-fefd-4ac1-afdb-0bbaed2b34b2" />


Right now:

* likely empty output for pvs/vgs/lvs
* because no LVM created yet

---

# Task 2 — Create Physical Volume (PV)

## Command

```bash
sudo pvcreate /dev/loop3
```

Output:

<img width="428" height="55" alt="image" src="https://github.com/user-attachments/assets/3555e678-4709-4c06-bafd-4493205a4361" />


Meaning:

* Prepare disk for LVM usage

Think:

```text
Normal disk → LVM-enabled disk
```

---

## Verify

```bash
pvs
```

Output:

<img width="311" height="80" alt="image" src="https://github.com/user-attachments/assets/2dbf525d-5faa-447c-95d8-8453f83a14df" />





### Understanding Output

| Column  | Meaning               |
| ------- | --------------------- |
| `PV`    | physical volume       |
| `VG`    | volume group attached |
| `PSize` | total size            |
| `PFree` | free space            |

Currently:

* no VG yet
* all 1 GB free

---

# Task 3 — Create Volume Group (VG)

## Command

```bash
vgcreate devops-vg /dev/loop3
```

Output:

<img width="483" height="57" alt="image" src="https://github.com/user-attachments/assets/d42c7020-4266-4ece-abc7-72ba026f4340" />


Meaning:

* Create storage pool named `devops-vg`

Think:

```text
Disk storage collected into one pool
```

---

## Verify

```bash
vgs
```

Output:

<img width="384" height="82" alt="image" src="https://github.com/user-attachments/assets/dbac2996-a71a-4736-af12-7ea36875d5f7" />



### Meaning

| Column  | Meaning              |
| ------- | -------------------- |
| `#PV`   | number of disks      |
| `#LV`   | logical volumes      |
| `VSize` | total VG size        |
| `VFree` | free available space |

---

# Task 4 — Create Logical Volume (LV)

## Command

```bash
lvcreate -L 500M -n app-data devops-vg
```

Output:

<img width="537" height="58" alt="image" src="https://github.com/user-attachments/assets/70659880-80f8-4950-8a15-db61cf0765b0" />


Meaning:

| Part          | Meaning |
| ------------- | ------- |
| `-L 500M`     | size    |
| `-n app-data` | LV name |
| `devops-vg`   | VG name |

This creates a virtual partition.

---

## Verify

```bash
lvs
```

Output:

<img width="566" height="114" alt="image" src="https://github.com/user-attachments/assets/0e7d131e-45db-4c0b-9d84-b3cabecbb9c8" />




---

# Task 5 — Format and Mount

Right now LV exists but has no filesystem.

Like:

* new pendrive before formatting

---

## Format

```bash
sudo mkfs.ext4 /dev/devops-vg/app-data
```

Output:

<img width="540" height="216" alt="image" src="https://github.com/user-attachments/assets/de7b9299-2fe2-410b-bfd1-da580e656429" />


Meaning:

* Create EXT4 filesystem

---

## Create Mount Folder

```bash
mkdir -p /mnt/app-data
```

---

## Mount LV

```bash
mount /dev/devops-vg/app-data /mnt/app-data
```

Meaning:

* Attach storage to Linux filesystem

---

## Verify

```bash
df -h /mnt/app-data
```

Output:

<img width="566" height="103" alt="image" src="https://github.com/user-attachments/assets/2b22c122-2a7e-42c9-a5ba-178546c5690e" />


Now your LV is usable storage.

---

# Task 6 — Extend Storage

This is the main power of LVM.

---

## Extend LV

```bash
lvextend -L +200M /dev/devops-vg/app-data
```
Output:

<img width="573" height="77" alt="image" src="https://github.com/user-attachments/assets/a3ca6ffa-8891-4d7b-bd62-c2befce23a49" />



Meaning:

* Add 200 MB more storage

---

## Resize Filesystem

```bash
resize2fs /dev/devops-vg/app-data
```

Output:

<img width="570" height="136" alt="image" src="https://github.com/user-attachments/assets/5b598408-02f8-4dce-a1af-a43b0a466fd7" />



Why needed?
Because:

* LV became larger
* filesystem still thinks old size exists

This updates filesystem size.

---

## Verify Again

```bash
df -h /mnt/app-data
```

Output:

<img width="536" height="69" alt="image" src="https://github.com/user-attachments/assets/5b8b4d58-dce6-4f27-8393-4a1a3f9d993c" />


Now size should increase.

---

# Final Understanding

## Real Flow

```text
disk1.img
    ↓
/dev/loop3
    ↓
PV (Physical Volume)
    ↓
VG (Volume Group)
    ↓
LV (Logical Volume)
    ↓
Filesystem (ext4)
    ↓
Mount Point (/mnt/app-data)
```

This entire setup is simulating real production storage management.
