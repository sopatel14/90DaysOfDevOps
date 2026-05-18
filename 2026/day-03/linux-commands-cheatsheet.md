````markdown
# Linux Commands Cheat Sheet

## File System Commands

| Command | Usage |
|---|---|
| `pwd` | Show current working directory |
| `ls` | List files and directories |
| `ls -la` | List all files with details |
| `cd /path` | Change directory |
| `mkdir dirname` | Create a directory |
| `rmdir dirname` | Remove empty directory |
| `rm file.txt` | Delete a file |
| `rm -rf dirname` | Remove directory recursively |
| `cp source dest` | Copy files/directories |
| `mv source dest` | Move or rename files |
| `touch file.txt` | Create empty file |
| `cat file.txt` | Display file content |
| `less file.txt` | View file page by page |
| `head -n 10 file.txt` | Show first 10 lines |
| `tail -n 10 file.txt` | Show last 10 lines |
| `tail -f logfile.log` | Monitor logs in real time |
| `find / -name file.txt` | Search for files |
| `locate filename` | Quickly locate files |
| `du -sh *` | Show directory sizes |
| `df -h` | Show disk usage |

---

## Process Management Commands

| Command | Usage |
|---|---|
| `ps aux` | Show all running processes |
| `top` | Real-time process monitoring |
| `htop` | Interactive process viewer |
| `kill PID` | Kill process by PID |
| `kill -9 PID` | Force kill process |
| `pkill processname` | Kill process by name |
| `pgrep processname` | Find process ID |
| `jobs` | Show background jobs |
| `bg` | Run job in background |
| `fg` | Bring job to foreground |
| `nohup command &` | Run process after logout |
| `nice -n 10 command` | Start process with priority |
| `renice PID` | Change process priority |
| `uptime` | Show system uptime |
| `free -h` | Show memory usage |

---

## Networking Commands

| Command | Usage |
|---|---|
| `ping google.com` | Test connectivity |
| `ip addr` | Show IP addresses |
| `ifconfig` | Display network interfaces |
| `curl https://example.com` | Fetch webpage content |
| `wget URL` | Download files |
| `ssh user@host` | Connect to remote server |
| `scp file user@host:/path` | Securely copy files |
| `ss -tulnp` | Show listening ports |
| `netstat -tulnp` | Display network statistics |
| `dig google.com` | DNS lookup |
| `nslookup google.com` | Query DNS records |
| `traceroute google.com` | Trace network path |
| `hostname -I` | Show host IP address |
| `route -n` | Show routing table |
| `arp -a` | View ARP cache |

---

## User & Permission Commands

| Command | Usage |
|---|---|
| `whoami` | Show current user |
| `id` | Show user/group IDs |
| `who` | Show logged-in users |
| `passwd` | Change password |
| `chmod +x script.sh` | Make file executable |
| `chmod 755 file` | Change file permissions |
| `chown user:user file` | Change file ownership |
| `sudo command` | Run command as admin |

---

## Log & System Troubleshooting Commands

| Command | Usage |
|---|---|
| `journalctl -xe` | View systemd logs |
| `dmesg` | Show kernel logs |
| `grep "error" file.log` | Search text in logs |
| `history` | Show command history |
| `uname -a` | Show system information |
| `date` | Show current date/time |
| `cal` | Display calendar |

---

## Archive & Compression Commands

| Command | Usage |
|---|---|
| `tar -czvf file.tar.gz dir` | Create tar.gz archive |
| `tar -xzvf file.tar.gz` | Extract tar.gz archive |
| `zip file.zip file` | Create ZIP archive |
| `unzip file.zip` | Extract ZIP archive |
| `gzip file.txt` | Compress file |
| `gunzip file.gz` | Decompress file |

---

## Text Processing Commands

| Command | Usage |
|---|---|
| `grep "text" file.txt` | Search text in file |
| `awk '{print $1}' file.txt` | Print first column |
| `sed 's/old/new/g' file.txt` | Replace text in file |
| `sort file.txt` | Sort lines alphabetically |
| `uniq file.txt` | Remove duplicate lines |
| `cut -d":" -f1 /etc/passwd` | Extract column data |

---

## System Information Commands

| Command | Usage |
|---|---|
| `lscpu` | Show CPU information |
| `lsblk` | List block devices |
| `lsmem` | Show memory details |
| `dmidecode` | Display hardware information |
| `neofetch` | Show system information nicely |

---

## Disk Management Commands

| Command | Usage |
|---|---|
| `fdisk -l` | List disk partitions |
| `mount /dev/sda1 /mnt` | Mount filesystem |
| `umount /mnt` | Unmount filesystem |
| `blkid` | Show block device attributes |
| `sync` | Flush filesystem buffers |

---

# Quick Tips

- Use `man <command>` for documentation
- Use `history` to reuse previous commands
- Use `TAB` for auto-completion
- Combine commands with pipes `|`
- Use `sudo` for administrative access
- Use `CTRL + C` to stop running commands
- Use `CTRL + R` to search command history

---

# Favorite Troubleshooting Commands

## Log Monitoring

```bash
tail -f /var/log/syslog
```

## Network Troubleshooting

```bash
traceroute google.com
```

## Process Monitoring

```bash
top
```

## Memory Usage

```bash
free -h
```

## Disk Usage

```bash
df -h
```

## Open Ports

```bash
ss -tulnp
```

---

# Helpful Notes

- `sudo` gives temporary administrative privileges
- `|` (pipe) sends output of one command to another
- `>` overwrites output into a file
- `>>` appends output into a file
- Use `CTRL + Z` to pause a process
- Use `fg` to resume paused jobs in foreground

---

# Learn More

```bash
man <command>
```

```bash
<command> --help
```
````
