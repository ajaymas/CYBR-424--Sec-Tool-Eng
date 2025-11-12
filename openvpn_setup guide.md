# OpenVPN hands-on

## Table of Contents
1. [OpenVPN Practical Lab Guide (20 Minutes)](#openvpn-practical-lab-guide)
2. [OpenVPN introduction](#openvpn introduction)
3. [Assessment](#Assessment)

---

# OpenVPN Practical Lab Guide (20 Minutes)

A quick hands-on OpenVPN setup instruction.

## Setup Overview (2 min)
You'll need 2 Linux machines (VMs or containers):
- **Server**: 192.168.1.10
- **Client**: 192.168.1.20

## Part 1: Server Setup (8 min)

### Step 1: Install OpenVPN and Easy-RSA
```bash
# On Ubuntu/Debian
sudo apt update
sudo apt install openvpn easy-rsa -
```

### Step 2: Set Up PKI (Certificate Authority)
```bash
# Create Easy-RSA directory
make-ca-dir ~/openvpn-ca
cd ~/openvpn-ca

# Initialize PKI
./easyrsa init-pki

# Build CA (press Enter for defaults, set a passphrase)
./easyrsa build-ca nopass

# Generate server certificate
./easyrsa gen-req server nopass
./easyrsa sign-req server server

# Generate Diffie-Hellman parameters (takes 1-2 min)
./easyrsa gen-dh

# Generate TLS-auth key for extra security
openvpn --genkey secret ta.key
```

### Step 3: Copy Certificates to OpenVPN Directory
```bash
sudo cp pki/ca.crt /etc/openvpn/
sudo cp pki/issued/server.crt /etc/openvpn/
sudo cp pki/private/server.key /etc/openvpn/
sudo cp pki/dh.pem /etc/openvpn/
sudo cp ta.key /etc/openvpn/
```

### Step 4: Create Server Configuration
```bash
sudo nano /etc/openvpn/server.conf
```

Paste this configuration:
```
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt

push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"

keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun

status openvpn-status.log
verb 3
```

### Step 5: Enable IP Forwarding and Start Server
```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# Start OpenVPN server
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server

# Check status
sudo systemctl status openvpn@server
```

## Part 2: Client Setup (6 min)

### Step 1: Generate Client Certificate (on server)
```bash
cd ~/openvpn-ca
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

### Step 2: Transfer Files to Client
Copy these files from the server to the client:
- `pki/ca.crt`
- `pki/issued/client1.crt`
- `pki/private/client1.key`
- `ta.key`

```bash
# On server - create transfer directory
mkdir ~/client-configs
cp ~/openvpn-ca/pki/ca.crt ~/client-configs/
cp ~/openvpn-ca/pki/issued/client1.crt ~/client-configs/
cp ~/openvpn-ca/pki/private/client1.key ~/client-configs/
cp ~/openvpn-ca/ta.key ~/client-configs/

# Use SCP to transfer to the client
scp -r ~/client-configs/* user@192.168.1.20:~/
```

### Step 3: Install OpenVPN on Client
```bash
sudo apt install openvpn -y
```

### Step 4: Create Client Configuration
```bash
sudo nano /etc/openvpn/client.conf
```

Paste this:
```
client
dev tun
proto udp

remote 192.168.1.10 1194

resolv-retry infinite
nobind
persist-key
persist-tun

ca ca.crt
cert client1.crt
key client1.key
tls-auth ta.key 1

cipher AES-256-CBC
verb 3
```

### Step 5: Move certificates and connect
```bash
sudo cp ~/ca.crt ~/client1.* ~/ta.key /etc/openvpn/

# Connect to VPN
sudo openvpn --config /etc/openvpn/client.conf
```

## Part 3: Testing the VPN (4 min)

### Test 1: Check Interface Creation
```bash
# On client - should see tun0 interface
ip addr show tun0
```

### Test 2: Ping VPN Gateway
```bash
# Ping the VPN server's tunnel IP
ping 10.8.0.1
```

### Test 3: Check Routing
```bash
# View routing table
ip route
# Should see route via 10.8.0.x
```

### Test 4: Verify IP Address
```bash
# Check your public IP (should show server's IP)
curl ifconfig.me
```

### Test 5: Test Traffic Routing
```bash
# Traceroute to external site
traceroute google.com
# First hop should be 10.8.0.1
```

### Test 6: Check Connection Logs
```bash
# On server
sudo tail -f /var/log/openvpn/openvpn-status.log

# On client - check connection messages
sudo journalctl -u openvpn@client -f
```

## Quick Troubleshooting Commands

```bash
# Check if OpenVPN is running
sudo systemctl status openvpn@server

# View real-time logs
sudo tail -f /var/log/syslog | grep ovpn

# Check firewall rules
sudo iptables -L -n -v

# Verify listening port
sudo netstat -tulpn | grep 1194

# Test server connectivity
nc -zvu 192.168.1.10 1194
```

## Key Points to Remember
- Server uses `tls-auth ta.key 0`, client uses `1`
- VPN subnet is 10.8.0.0/24
- Server tunnel IP: 10.8.0.1
- Client gets IP from 10.8.0.x range
- Always check logs first when troubleshooting

---
```
