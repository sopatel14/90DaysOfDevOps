# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## Task 1: DNS – How Names Become IPs

### What Happens When You Type google.com in a Browser?

1. Your system checks whether the IP address for `google.com` is already cached.
2. If not, it sends a DNS query to a DNS server.
3. The DNS server resolves the domain name to an IP address.
4. The browser connects to that IP address and loads the website.

---

### DNS Record Types

| Record Type | Purpose                                               |
| ----------- | ----------------------------------------------------- |
| A           | Maps a domain name to an IPv4 address                 |
| AAAA        | Maps a domain name to an IPv6 address                 |
| CNAME       | Creates an alias from one domain to another           |
| MX          | Specifies mail servers for a domain                   |
| NS          | Identifies the authoritative DNS servers for a domain |

---

### DNS Lookup

#### Command

```bash
dig google.com
```

Output:

<img width="590" height="476" alt="image" src="https://github.com/user-attachments/assets/cab99750-18b4-45a8-bef6-4d63fffd445d" />


---

# Task 2: IP Addressing

## What is an IPv4 Address?

An IPv4 address is a 32-bit numeric address used to identify devices on a network. It consists of four octets separated by dots, such as `192.168.1.10`.

---

## Public vs Private IP Addresses

| Type       | Example      | Description                 |
| ---------- | ------------ | --------------------------- |
| Public IP  | 8.8.8.8      | Reachable over the internet |
| Private IP | 192.168.1.10 | Used within local networks  |

---

## Private IP Ranges

```text
10.0.0.0 - 10.255.255.255
172.16.0.0 - 172.31.255.255
192.168.0.0 - 192.168.255.255
```

---

### Command

```bash
ip addr show
```

### Observation

Example:

<img width="1304" height="868" alt="image" src="https://github.com/user-attachments/assets/8ebe39af-d469-46fe-baaa-2452b44ac6a1" />


This is a private IP because it falls within the `192.168.x.x` range.

---

# Task 3: CIDR & Subnetting

## What Does /24 Mean?

`/24` means the first 24 bits are used for the network portion of the address, leaving 8 bits for hosts.

Example:

```text
192.168.1.0/24
```

---

## Why Do We Subnet?

* Improves network organization.
* Reduces broadcast traffic.
* Enhances security and management.
* Allows efficient IP address allocation.

---

## CIDR Table

| CIDR | Subnet Mask     | Total IPs | Usable Hosts |
| ---- | --------------- | --------- | ------------ |
| /24  | 255.255.255.0   | 256       | 254          |
| /16  | 255.255.0.0     | 65,536    | 65,534       |
| /28  | 255.255.255.240 | 16        | 14           |

---

## Host Calculation Notes

### /24

* Total IPs: 256
* Usable Hosts: 254

### /16

* Total IPs: 65,536
* Usable Hosts: 65,534

### /28

* Total IPs: 16
* Usable Hosts: 14

---

# Task 4: Ports – The Doors to Services

## What is a Port?

A port is a logical communication endpoint used by applications and services on a device.

Ports allow multiple services to run on the same IP address.

---

## Common Ports

| Port  | Service |
| ----- | ------- |
| 22    | SSH     |
| 80    | HTTP    |
| 443   | HTTPS   |
| 53    | DNS     |
| 3306  | MySQL   |
| 6379  | Redis   |
| 27017 | MongoDB |

---

## Listening Services

### Command

```bash
ss -tulpn
```

Output:

<img width="1910" height="524" alt="image" src="https://github.com/user-attachments/assets/ab2fd1b1-516d-4028-855e-38db15b50fb1" />


### Observation

| Port | Service |
| ---- | ------- |
| 22   | SSH     |


---

# Task 5: Putting It Together

## Scenario 1

### Question

You run:

```bash
curl http://myapp.com:8080
```

### Answer

Several networking concepts are involved:

* DNS resolves `myapp.com` into an IP address.
* IP routing sends traffic to the destination host.
* Port `8080` identifies the target application.
* HTTP is used at the application layer to request data.

---

## Scenario 2

### Question

Your app can't reach a database at:

```text
10.0.1.50:3306
```

### Answer

First checks:

* Verify network connectivity using `ping`.
* Confirm MySQL is listening on port `3306`.
* Check firewall/security group rules.
* Ensure the application is using the correct IP and port.

---

# What I Learned

1. DNS translates human-friendly domain names into IP addresses.
2. CIDR notation determines network size and available hosts.
3. Ports allow multiple services to communicate over the same IP address.

---

# Commands Used

```bash
# DNS lookup
dig google.com

# Show IP addresses
ip addr show

# Show listening ports
ss -tulpn
```
