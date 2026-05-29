# Day 14 – Networking Fundamentals & Hands-on Checks

## Quick Concepts

### OSI Model (L1–L7)

* OSI divides networking into 7 layers:

  * Physical
  * Data Link
  * Network
  * Transport
  * Session
  * Presentation
  * Application
* Helps identify where issues happen during troubleshooting.

---

### TCP/IP Model

* TCP/IP has 4 layers:

  * Link
  * Internet
  * Transport
  * Application
* Used in real-world networking and internet communication.

---

### Where Protocols Sit

| Protocol     | Layer              |
| ------------ | ------------------ |
| IP           | Internet / Network |
| TCP / UDP    | Transport          |
| HTTP / HTTPS | Application        |
| DNS          | Application        |

---

### Real Example

```bash
curl https://yahoo.com
```

* Application Layer → HTTP/HTTPS
* Transport Layer → TCP
* Internet Layer → IP

---

# Hands-on Networking Checks

## 1. Identity Check

### Command

```bash
hostname -I
```

OR

```bash
ip addr show
```
Output:

<img width="652" height="434" alt="image" src="https://github.com/user-attachments/assets/628732d9-6b1b-4fd1-a70c-3f6af62699d2" />



### Observation

* My machine received a local IP address.
* Confirms the system is connected to the network.

---

## 2. Reachability Test

### Command

```bash
ping google.com
```
Output:

<img width="637" height="305" alt="image" src="https://github.com/user-attachments/assets/88f560f3-6d40-400c-9695-8d6118a249f1" />



### Observation

* Packets were successfully received.
* Latency and packet loss help measure network health.

---

## 3. Path Check

### Command

```bash
traceroute google.com
```
Output:

<img width="958" height="565" alt="image" src="https://github.com/user-attachments/assets/fb48ec4f-36ec-4f14-b70a-f1d380a9fda6" />



OR

```bash
tracepath google.com
```


### Observation

* Shows the path packets take to reach the destination.
* Some hops may timeout because routers block ICMP replies.

---

## 4. Open Ports / Listening Services

### Command

```bash
ss -tulpn
```
Output:

<img width="941" height="242" alt="image" src="https://github.com/user-attachments/assets/f5749567-a0de-4313-a2f2-40ecfc8c9482" />



OR

```bash
netstat -tulpn
```

### Observation

* Displays listening services and their ports.
* Example: SSH listening on port 22.

---

## 5. DNS Resolution Check

### Command

```bash
dig google.com
```
Output:

<img width="542" height="437" alt="image" src="https://github.com/user-attachments/assets/cf37b0c3-36fa-4949-8784-c4c893b62fa6" />

OR

```bash
nslookup google.com
```

### Observation

* Domain name resolved successfully into an IP address.
* Confirms DNS is working correctly.

---

## 6. HTTP Header Check

### Command

```bash
curl -I https://google.com
```

Output:

<img width="967" height="277" alt="image" src="https://github.com/user-attachments/assets/e911b2d9-d505-4272-b9b9-ab0def62e379" />


### Observation

* Received an HTTP response status.
* Example: `200 OK` means the website is reachable.

---

## 7. Connections Snapshot

### Command

```bash
netstat -an | head
```
Output:

<img width="643" height="211" alt="image" src="https://github.com/user-attachments/assets/3dc3b1ce-fdff-4d46-aeb8-665c249b8a62" />



### Observation

* LISTEN → services waiting for connections
* ESTABLISHED → active communication sessions

---

# Mini Task – Port Probe & Interpretation

## Step 1: Identify Listening Port

### Command

```bash
ss -tulpn
```
Output:

<img width="955" height="262" alt="image" src="https://github.com/user-attachments/assets/edc1e541-373f-4f12-86e6-24dee9b70af3" />


### Example

* SSH service running on port 22

---

## Step 2: Test the Port

### Command

```bash
nc -zv localhost 22
```

Output:

<img width="972" height="114" alt="image" src="https://github.com/user-attachments/assets/9d239268-d0b3-4576-ba5b-ff0e752d0cb0" />


### Observation

* Connection succeeded.
* Confirms the port is reachable locally.

### If Port Was Not Reachable

Next checks:

* Verify service status
* Check firewall rules
* Confirm application is listening on the correct port

---

# Reflection

## Which command gives the fastest signal when something is broken?

* `ping` gives a quick connectivity check.
* `curl -I` quickly verifies web applications.

---

## What layer would you inspect if DNS fails?

* Application layer first (DNS service/configuration)
* Then Internet layer for IP connectivity

---

## What layer would you inspect if HTTP 500 appears?

* Application layer
* Usually indicates a backend/server-side issue

---

## Two Follow-up Checks During a Real Incident

### Check Firewall Rules

```bash
sudo ufw status
```

Output:

<img width="435" height="69" alt="image" src="https://github.com/user-attachments/assets/992192d0-b331-479a-bb67-0515d441df0c" />


### Check Service Status

```bash
systemctl status docker
```

Ouput:

<img width="962" height="413" alt="image" src="https://github.com/user-attachments/assets/a3dad568-5d18-4321-9133-5488d5c6cab0" />


---

# Commands Cheat Sheet

```bash
# Show IP address
hostname -I
ip addr show

# Connectivity test
ping google.com

# Route tracing
traceroute google.com
tracepath google.com

# Open ports
ss -tulpn
netstat -tulpn

# DNS lookup
dig google.com
nslookup google.com

# HTTP check
curl -I https://google.com

# Active connections
netstat -an | head

# Port test
nc -zv localhost 22

# Firewall check
sudo ufw status

# Service status
systemctl status ssh
```
