#  Attack Surface Mapping - Safe Practice Guide: Educational purpose only 

##  Objective
Learn to identify and analyze your system's attack surface using Linux tools **without causing errors or system damage**.

---

##  Prerequisites Check

Before starting, verify you have the required tools:

```bash
# Check if tools are installed
which ss && echo "✓ ss installed" || echo "✗ ss missing"
which curl && echo "✓ curl installed" || echo "✗ curl missing"
which nmap && echo "✓ nmap installed" || echo "✗ nmap missing"
```

### Install Missing Tools

```bash
# For Debian/Ubuntu
sudo apt-get update
sudo apt-get install -y iproute2 curl nmap
```

---

##  Phase 1: List All Listening Services

### Command 1: `sudo ss -tulpn`

**What it does:** Shows all TCP/UDP listening ports and the programs using them

**Breaking down the flags:**
- `-t` = TCP sockets
- `-u` = UDP sockets
- `-l` = Listening sockets only
- `-p` = Show process using the socket
- `-n` = Numeric addresses (don't resolve names)

### Safe Practice Steps:

```bash
# Step 1: Run basic command
sudo ss -tulpn

# Step 2: Save output to file for analysis
sudo ss -tulpn > ~/attack_surface_ports.txt

# Step 3: Count how many services are listening
sudo ss -tulpn | grep LISTEN | wc -l

# Step 4: Filter for specific ports (e.g., web servers)
sudo ss -tulpn | grep ':80\|:443\|:8080'

# Step 5: See only TCP services
sudo ss -tlpn

# Step 6: See only UDP services
sudo ss -ulpn
```

###  Understanding the Output:

```
Netid  State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=1234,fd=3))
tcp    LISTEN  0       128     127.0.0.1:631       0.0.0.0:*          users:(("cupsd",pid=5678,fd=7))
```

**What each column means:**
- **Netid:** Protocol (tcp/udp)
- **State:** LISTEN means waiting for connections
- **Local Address:Port:** Where service is listening
  - `0.0.0.0:22` = SSH listening on ALL interfaces
  - `127.0.0.1:631` = CUPS printer service on localhost only
- **Process:** Which program is running

###  Analysis Exercise:

```bash
# Create a simple analysis script
cat > analyze_ports.sh << 'EOF'
#!/bin/bash
echo "=== Attack Surface Analysis ==="
echo ""
echo "Public-facing services (listening on 0.0.0.0):"
sudo ss -tulpn | grep '0.0.0.0:' | awk '{print $5, $7}'
echo ""
echo "Localhost-only services (safer):"
sudo ss -tulpn | grep '127.0.0.1:' | awk '{print $5, $7}'
echo ""
echo "Total listening ports:"
sudo ss -tulpn | grep LISTEN | wc -l
EOF

chmod +x analyze_ports.sh
./analyze_ports.sh
```

---

##  Phase 2: Check Web Server Exposure

### Command 2: `curl -I localhost`

**What it does:** Fetches HTTP headers from web server (reveals version info)

### Safe Practice Steps:

```bash
# Step 1: Check if web server is running
curl -I localhost

# Common outcomes:
# ✓ If you see HTTP headers = web server is running
# ✗ "Connection refused" = no web server (this is NORMAL!)
```

### If You DON'T Have a Web Server (Most Common):

```bash
# Install a simple test web server
sudo apt-get install -y apache2  # Debian/Ubuntu
# OR
sudo yum install -y httpd        # RHEL/CentOS

# Start the service
sudo systemctl start apache2  # Ubuntu
# OR
sudo systemctl start httpd    # RHEL

# Now try curl again
curl -I localhost
```

### Understanding curl Output:

```bash
# Good output example:
HTTP/1.1 200 OK
Date: Mon, 27 Oct 2025 10:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)  # ← VERSION EXPOSED!
Content-Type: text/html
```

**Security Analysis:**
- **Server: Apache/2.4.41** = Reveals web server type and version
- Attackers use this to find known vulnerabilities
- Solution: Hide version info or keep updated

### Advanced curl Testing:

```bash
# Test different ports
curl -I localhost:80     # HTTP
curl -I localhost:443    # HTTPS (will fail if no SSL)
curl -I localhost:8080   # Alternative HTTP port

# Test with verbose output
curl -v localhost

# Save headers to file
curl -I localhost > ~/webserver_headers.txt

# Check for security headers
curl -I localhost | grep -i "security\|x-frame\|x-xss\|strict-transport"
```

###  What if Nothing Works?

```bash
# This is NORMAL if you don't have a web server!
# Create a simple test server using Python:

# Python 3 simple server
python3 -m http.server 8000 &

# Now test it
curl -I localhost:8000

# Stop it when done
killall python3
```

---

##  Phase 3: Full Attack Surface Scan

### Command 3: `sudo nmap -A -T4 localhost`

**What it does:** Comprehensive scan of your system

**Breaking down the flags:**
- `-A` = Aggressive scan (OS detection, version detection, scripts, traceroute)
- `-T4` = Timing template (faster, 0=slowest, 5=fastest)
- `localhost` = Target (127.0.0.1)

###  Important Safety Notes:

1. **ONLY scan localhost/127.0.0.1** (your own machine)
2. **Never scan external IPs** without permission (it's illegal!)
3. **This scan is noisy** (logs will show it)
4. **Takes 5-10 minutes** depending on services

### Safe Practice Steps:

```bash
# Step 1: Basic port scan first (faster, safer)
sudo nmap localhost

# Step 2: Scan specific ports only
sudo nmap -p 22,80,443 localhost

# Step 3: Version detection only
sudo nmap -sV localhost

# Step 4: Full aggressive scan (use carefully)
sudo nmap -A -T4 localhost

# Step 5: Save results
sudo nmap -A -T4 localhost -oN ~/nmap_localhost_scan.txt
```

### Understanding nmap Output:

```
Starting Nmap 7.80 ( https://nmap.org )
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000020s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
443/tcp open  ssl/http Apache httpd 2.4.41 ((Ubuntu))
```

**What it reveals:**
- **Open ports:** 22 (SSH), 80 (HTTP), 443 (HTTPS)
- **Service versions:** OpenSSH 8.2p1, Apache 2.4.41
- **OS detection:** Ubuntu Linux
- **This is your attack surface!**

###  Focused Scanning:

```bash
# Scan common web ports only
sudo nmap -p 80,443,8080,8443 localhost

# Scan ALL ports (takes longer)
sudo nmap -p- localhost

# Fast scan (top 100 ports)
sudo nmap -F localhost

# Scan with scripts for vulnerabilities
sudo nmap --script vuln localhost

# UDP scan (slower)
sudo nmap -sU localhost
```

---

##  Complete Practice Lab

### Lab Exercise: Full Attack Surface Assessment

```bash
#!/bin/bash
# Save as: attack_surface_lab.sh

echo "╔════════════════════════════════════════╗"
echo "║  Attack Surface Assessment Lab         ║"
echo "╚════════════════════════════════════════╝"
echo ""

# Create results directory
mkdir -p ~/security_lab_results
cd ~/security_lab_results

echo "[1/5] Checking listening services..."
sudo ss -tulpn > step1_listening_services.txt
echo "✓ Saved to step1_listening_services.txt"
echo ""

echo "[2/5] Counting exposed services..."
EXPOSED_COUNT=$(sudo ss -tulpn | grep '0.0.0.0:' | wc -l)
echo "    Found $EXPOSED_COUNT public-facing services"
echo ""

echo "[3/5] Testing web server (if exists)..."
curl -I localhost > step3_webserver.txt 2>&1
if [ $? -eq 0 ]; then
    echo "✓ Web server detected"
else
    echo "✗ No web server found (this is normal)"
fi
echo ""

echo "[4/5] Quick port scan..."
sudo nmap -F localhost > step4_port_scan.txt
echo "✓ Saved to step4_port_scan.txt"
echo ""

echo "[5/5] Service version detection..."
sudo nmap -sV localhost > step5_versions.txt
echo "✓ Saved to step5_versions.txt"
echo ""

echo "════════════════════════════════════════"
echo " Summary Report"
echo "════════════════════════════════════════"
echo "Total listening ports: $(sudo ss -tulpn | grep LISTEN | wc -l)"
echo "Public-facing services: $EXPOSED_COUNT"
echo "Open ports: $(sudo nmap -F localhost | grep open | wc -l)"
echo ""
echo " All results saved in: ~/security_lab_results/"
echo ""
echo " Next steps:"
echo "   1. Review each .txt file"
echo "   2. Identify unnecessary services"
echo "   3. Close unused ports"
echo "   4. Update outdated versions"
```

**Run the lab:**
```bash
chmod +x attack_surface_lab.sh
./attack_surface_lab.sh
```

---

##  Common Issues & Solutions

### Issue 1: "Permission denied"
```bash
# Solution: Use sudo
sudo ss -tulpn
sudo nmap localhost
```

### Issue 2: "Command not found"
```bash
# Install the tool first
sudo apt-get install iproute2  # for ss
sudo apt-get install nmap      # for nmap
sudo apt-get install curl      # for curl
```

### Issue 3: nmap shows "All ports closed"
```bash
# This means good security! But if you want to test:
# Start a simple service
python3 -m http.server 8000 &
sudo nmap localhost
# Kill it when done
killall python3
```

### Issue 4: curl connection refused
```bash
# Normal if no web server is running!
# Check if any service is on port 80:
sudo ss -tulpn | grep :80

# If nothing, install test web server:
sudo apt-get install apache2
sudo systemctl start apache2
curl -I localhost
```

### Issue 5: nmap scan takes forever
```bash
# Use faster scan:
sudo nmap -F localhost           # Fast scan (100 ports)
sudo nmap -T5 localhost          # Insane speed
sudo nmap --top-ports 20 localhost  # Only top 20 ports
```

---

##  Practice Exercises

### Exercise 1: Baseline Assessment
```bash
# Document your current attack surface
sudo ss -tulpn > baseline_before.txt
sudo nmap -F localhost > baseline_ports_before.txt

# Compare later to see changes
```

### Exercise 2: Service Reduction
```bash
# Find unnecessary services
sudo ss -tulpn | grep LISTEN

# Stop a test service
sudo systemctl stop cups  # Printer service example

# Rescan
sudo ss -tulpn | grep LISTEN

# Restart if needed
sudo systemctl start cups
```

### Exercise 3: Web Server Hardening
```bash
# Check current headers
curl -I localhost > headers_before.txt

# Configure server to hide version
# (Edit Apache config, then restart)

# Check again
curl -I localhost > headers_after.txt

# Compare
diff headers_before.txt headers_after.txt
```

---


##  Key Takeaways

1.  **Always scan only localhost** for practice
2.  **Document your baseline** before making changes
3.  **Fewer open ports = smaller attack surface**
4.  **Hide version information** when possible
5.  **Services on 127.0.0.1** are safer than 0.0.0.0
6.  **Regular scans** help detect unwanted changes
7.  **Update software** to patch vulnerabilities

---

##  Additional Resources

```bash
# man pages for more details
man ss
man nmap
man curl

# Online nmap tutorial
firefox https://nmap.org/book/man.html

# Practice safely with intentionally vulnerable VMs:
# - Metasploitable
# - DVWA (Damn Vulnerable Web App)
# - VulnHub machines
```

---

##  Completion Checklist

- [ ] Installed all required tools
- [ ] Successfully ran `ss -tulpn`
- [ ] Understood listening services output
- [ ] Tested `curl -I localhost`
- [ ] Completed basic nmap scan
- [ ] Ran full attack surface lab script
- [ ] Documented findings
- [ ] Identified at least 3 services on your system

---
*Last Updated: October 27, 2025*
*Version: 1.0*
*Author: Network Security Lab*
