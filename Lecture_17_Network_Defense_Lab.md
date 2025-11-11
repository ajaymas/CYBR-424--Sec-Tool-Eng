
#  Lecture 17: Network Defense — Practical Lab Guide (Ubuntu-based)

This lab follows the lecture concepts with **hands-on exercises** using Linux (Ubuntu/Debian).  
Each section includes: concept, real-world context, command walkthroughs, expected outputs, and screenshot placeholders.

---

##  1. Defense in Depth (DiD)

**Concept:** Layered security ensures that even if one defense fails, others protect the system.

**Example:** Firewall + IDS + MFA + Backup = Reduced overall risk.

###  Commands
```bash
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw deny 23/tcp
sudo ufw status numbered
```
- `ufw` stands for Uncomplicated Firewall. It is a frontend for `iptables` designed to make firewall management easier on Linux systems, especially for basic setups.

###  Expected Output
```
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
23/tcp                     DENY        Anywhere
```
 *[Screenshot: Terminal showing UFW rules]*

---

##  2. Risk Management

**Concept:** Risk = Threat × Vulnerability. Manage by patching, isolating, and monitoring.

**Example:** Patch outdated PHP; scan network for open services.

###  Commands
```bash
sudo nmap -sS localhost
sudo apt list --upgradable
```
📸 *[Screenshot: nmap scan output and upgrade list]*

---

---

##  3. Threats and Vulnerabilities

**Concept:** Weak passwords, unpatched software, and open services increase attack surface.

###  Commands
```bash
sudo grep -E '^[^:]+:[^\!*]' /etc/shadow
sudo systemctl list-units --type=service --state=running
```

 *[Screenshot: running services list]*

- `systemctl` is the main command to manage systemd services on Linux.
- `list-units` → Lists systemd units (services, sockets, etc.).
- `--type=service` → Only show services (not timers, sockets, etc.).
- `--state=running` → Only show services that are currently active and running.

 **Purpose:** See which services are currently running on your system.


## Example Output (snippet)

```
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION
  accounts-daemon.service     loaded active running Accounts Service
  apache2.service             loaded active running The Apache HTTP Server
  colord.service              loaded active running Manage, Install and Generate Color Profiles
  cron.service                loaded active running Regular background program processing daemon
  dbus.service                loaded active running D-Bus System Message Bus
```
---

### 5.  Commands (RBAC Example)
```bash
sudo groupadd finance
sudo useradd -G finance ajay
sudo mkdir /secure_finance
sudo chown :finance /secure_finance
sudo chmod 770 /secure_finance
```
📸 *[Screenshot: directory permissions]*

---

##  6. Network Attacks Overview

**Concept:** Malicious activities like DoS, spoofing, and routing exploitation.

### 🧪 Commands
```bash
sudo hping3 -S -p 80 -i u100 <target-ip>
sudo tcpdump -n -i eth0 tcp
```
⚠️ Use only in controlled lab networks.

📸 *[Screenshot: tcpdump live capture]*

---

##  6. ICMP and IP Layer Attacks

**Concept:** Exploiting trust in network protocols — ARP spoofing, ICMP redirect, etc.

###  Commands
```bash
sudo arp -n
sudo arping <target-ip>
```
📸 *[Screenshot: ARP table entries]*

---

##  7. TCP Layer Attacks

**Concept:** Exploiting TCP handshake and session trust.

###  Commands
```bash
sudo ss -tnp
cat /etc/resolv.conf
```
Use to inspect active TCP connections and DNS configurations.

📸 *[Screenshot: ss output]*

---

##  8. Denial of Service (DoS/DDoS)

**Concept:** Exhausts bandwidth or processing power to deny service.

###  Commands
```bash
sudo iftop -i eth0
sudo tcpdump -c 1000 -w ddos_trace.pcap
```
**Expected Output:** Network traffic spikes; pcap file captured.

📸 *[Screenshot: iftop live view]*

---

##  9. Network Hardening and Exposure Minimization

**Concept:** Reduce attack surface by disabling unused services, patching, and auditing.

###  Commands
```bash
sudo systemctl disable bluetooth
sudo apt update && sudo apt upgrade -y
sudo cat /var/log/auth.log | grep "Failed"
```

**Expected Output:**
```
Failed password for invalid user admin from 192.168.1.10 port 51234 ssh2
```
 *[Screenshot: auth.log failures]*

---

##  Summary Checklist

| Area | Tool/Command | Purpose |
|------|---------------|----------|
| Firewall | `ufw` | Restrict ports |
| Scanning | `nmap` | Detect open services |
| Encryption | `gpg` | Protect data confidentiality |
| Monitoring | `iftop`, `tcpdump` | Watch network activity |
| Hardening | `systemctl`, `apt` | Reduce exposure |

---
