# Linux Firewall Hands-on Exercises with Detailed Commands and Explanations

## ✅ Exercise 1: Setting Up a Basic Firewall Using Iptables

**Objective:**

Learn how to use iptables to create basic firewall rules.



1. **Check Current Rules:**

    ```bash
    sudo iptables -L
    ```

    - `iptables` → Command-line tool to configure the Linux firewall.
    - `-L` → Lists all current rules.

2. **Allow SSH Traffic from a Specific IP:**

    ```bash
    sudo iptables -A INPUT -s 192.168.1.100 -p tcp --dport 22 -j ACCEPT
    ```

    - `-A INPUT` → Append the rule to the INPUT chain.
    - `-s 192.168.1.100` → Source IP allowed to access SSH.
    - `-p tcp` → Specifies the protocol (TCP).
    - `--dport 22` → Destination port (SSH).
    - `-j ACCEPT` → Accept the packet.

3. **Block All Other SSH Traffic:**

    ```bash
    sudo iptables -A INPUT -p tcp --dport 22 -j DROP
    ```

    - `-j DROP` → Drops all other SSH traffic not explicitly allowed.

4. **Verify Rules:**

    ```bash
    sudo iptables -L -n -v
    ```

    - `-n` → Display numeric IP addresses.
    - `-v` → Display verbose output.

## Expected Output

When you run `sudo iptables -L -n -v`, you'll see output similar to this:

```
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     tcp  --  *      *       192.168.1.100        0.0.0.0/0            tcp dpt:22
    0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

---

## Output Explanation

* **Chain INPUT** - Shows rules for incoming traffic to your server

* **pkts / bytes** - Counters showing how many packets/bytes matched each rule (initially 0)

* **First rule (ACCEPT)** - Only allows SSH connections from IP 192.168.1.100
  * Source: 192.168.1.100
  * Destination port: 22 (SSH)
  * Action: ACCEPT

* **Second rule (DROP)** - Blocks all other SSH connection attempts
  * Source: 0.0.0.0/0 (any IP address)
  * Destination port: 22 (SSH)
  * Action: DROP (silently discards packets)

* **Rule order matters** - iptables processes rules sequentially, so the ACCEPT rule for 192.168.1.100 is evaluated first, then the DROP rule catches everything else

* **Security result** - Only the specific IP (192.168.1.100) can access SSH; all other SSH attempts are blocked, preventing unauthorized access


## ✅ Exercise 2: Implementing a Packet Filtering Firewall

**Objective:**

Create packet filtering rules using iptables.



1. **Block Traffic from a Specific IP Range:**

    ```bash
    sudo iptables -A INPUT -s 137.0.0.0/8 -j DROP
    ```

    - Blocks all incoming traffic from the 137.0.0.0/8 IP range.

2. **Allow HTTP and HTTPS Traffic:**

    ```bash
    sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
    ```

    - `--dport 80` → Allows HTTP (port 80).
    - `--dport 443` → Allows HTTPS (port 443).

3. **Allow Established Connections and Outgoing Traffic:**

    ```bash
    sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
    sudo iptables -A OUTPUT -j ACCEPT
    ```

    - `-m conntrack` → Use connection tracking module.
    - `--ctstate ESTABLISHED,RELATED` → Accepts established or related connections.

4. **Step 4: Save the Rules**

**First, create the directory:**
```bash
sudo mkdir -p /etc/iptables
```

**Then save the rules:**
```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

**Explanation:**
* `mkdir -p`→  Creates directory and parent directories if they don't exist
* `iptables-save`→  Outputs current iptables rules
* `|`→  Pipes output to the next command
* `sudo tee`→  Writes to file with sudo privileges
* `/etc/iptables/rules.v4`→  Standard location for persistent IPv4 rules



## ✅ Exercise 3: Simulating a SYN Flood Attack and Firewall Protection

**Objective:**

Understand SYN flood attacks and mitigate them with a firewall.



1. **Simulate a SYN Flood Attack:**

    ```bash
    hping3 -S -p 80 -c 10000 --flood <target-ip>
    ```

    - `-S` → SYN flag.
    - `-p 80` → Target port.
    - `-c 10000` → Send 10,000 packets.
    - `--flood` → Send packets continuously.

2. **Detect SYN Flood Attack:**

    ```bash
    sudo netstat -ant | grep SYN
    ```

    - `netstat -ant` → Display active network connections.

3. **Block SYN Flood Attack:**

    ```bash
    sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
    ```

    - `--syn` → Match SYN packets.
    - `--limit 1/s` → Limit to 1 packet per second.

## ✅ Exercise 4: Configuring an Application-Level Gateway with Squid Proxy

**Objective:**

Set up a simple application-level gateway.



1. **Install Squid Proxy:**

    ```bash
    sudo apt install squid
    ```

2. **Edit the Squid Configuration:**

    ```bash
    sudo nano /etc/squid/squid.conf
    ```

3. **Allow Only Specific IPs:**

    ```plaintext
    acl mynetwork src 192.168.1.0/24
    http_access allow mynetwork
    ```

4. **Restart Squid:**

    ```bash
    sudo systemctl restart squid
    ```

## ✅ Exercise 5: Configuring a DMZ Using iptables

**Objective:**

Set up a basic DMZ configuration.



1. **Define Network Interfaces:**

    - `eth0` → Internet-facing interface
    - `eth1` → Internal network
    - `eth2` → DMZ

2. **Allow Traffic from DMZ to Internet Only:**

    ```bash
    sudo iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
    ```

3. **Block Traffic from DMZ to Internal Network:**

    ```bash
    sudo iptables -A FORWARD -i eth2 -o eth1 -j DROP
    ```

## ✅ Exercise 6: Testing Stateful Firewall Rules

**Objective:**

Understand stateful firewalls.



1. **Allow Established and Related Connections:**

    ```bash
    sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
    ```

2. **Allow Outgoing Traffic:**

    ```bash
    sudo iptables -A OUTPUT -j ACCEPT
    ```

3. **Block New Incoming Traffic (except SSH):**

    ```bash
    sudo iptables -A INPUT -p tcp ! --dport 22 -m conntrack --ctstate NEW -j DROP
    ```

## ✅ Exercise 7: Monitoring and Logging Firewall Activity

**Objective:**

Enable logging of blocked packets.



1. **Add Logging Rule:**

    ```bash
    sudo iptables -A INPUT -j LOG --log-prefix "Dropped: " --log-level 4
    ```

2. **View the Logs:**

    ```bash
    sudo tail -f /var/log/syslog | grep "Dropped"
    ```

## ✅ Exercise 8: Firewall Persistence and Backup

**Objective:**

Save and restore firewall rules.



1. **Save the Rules:**

    ```bash
    sudo iptables-save > /etc/iptables/rules.v4
    ```

2. **Restore the Rules on Reboot:**

    ```bash
    sudo iptables-restore < /etc/iptables/rules.v4
    ```

## 💡 Key Learning Outcomes:

- Configuring and managing iptables firewall rules.
- Protecting against SYN flood attacks.
- Creating a DMZ and managing traffic.
- Configuring an application-level gateway.
- Monitoring and logging firewall activities.



