# Different Types of Attacks on Cryptography with Practical Command-Line Examples

Cryptographic systems are vulnerable to various types of attacks. Below, I explain some common attacks along with practical command-line examples using Linux-based tools.

## 1. Brute Force Attack

### Concept:
- Tries all possible keys until the correct one is found.
- Feasible for short keys but impractical for strong encryption like AES-256.

### Example (Cracking a ZIP File Encrypted with a Weak Password)

#### Step 1: Create an Encrypted ZIP File
```bash
zip --password secret123 encrypted.zip myfile.txt
```

#### Step 2: Brute Force with `fcrackzip`
```bash
fcrackzip -v -u -D -p /usr/share/wordlists/rockyou.txt encrypted.zip
```

### Explanation:
- `-v` → verbose mode
- `-u` → unzip test
- `-D -p` → dictionary attack using `rockyou.txt`
- If the password is in the dictionary, it will be cracked.

---

## 2. Dictionary Attack

### Concept:
- Uses a predefined list of common passwords instead of trying all possibilities.
- Faster than brute force but limited to known passwords.

### Example (Cracking a Hash with `hashcat`)

#### Step 1: Generate a SHA-256 Hash
```bash
echo -n "password123" | sha256sum > hash.txt
```

#### Step 2: Crack with `hashcat` Using a Dictionary
```bash
hashcat -m 1400 hash.txt /usr/share/wordlists/rockyou.txt
```

### Explanation:
- `-m 1400` → SHA-256 mode
- Uses `rockyou.txt` as a dictionary to guess the password.

---

## 3. Man-in-the-Middle (MITM) Attack

### Concept:
- The attacker intercepts and possibly modifies communication between two parties.
- Can be done using `ettercap` or `mitmproxy`.

### Example (Intercepting HTTP Traffic Using `mitmproxy`)

#### Step 1: Start `mitmproxy`
```bash
mitmproxy -p 8080
```

#### Step 2: Configure Victim to Use Proxy
- Set the browser’s proxy to `http://<attacker_ip>:8080`.

#### Step 3: Capture and Inspect Traffic
- Open `http://example.com` in the victim’s browser.

```bash
mitmproxy -T | grep "username\|password"
```

### Mitigation:
- Use HTTPS and ensure certificate validation.

---

## 4. Side-Channel Attack (Timing Attack)

### Concept:
- Observes execution time, power consumption, or electromagnetic leaks to extract cryptographic keys.

### Example (Timing Attack on OpenSSL RSA)

#### Step 1: Measure RSA Decryption Time
```bash
time openssl rsautl -decrypt -inkey private.pem -in secret.enc
```

### Mitigation:
- Implement constant-time cryptographic operations.

---

## 5. Birthday Attack (Collision Attack)

### Concept:
- Exploits the birthday paradox to find two different inputs with the same hash.
- Used against weak hashing algorithms like MD5.

### Example (Finding MD5 Collisions Using `hashdeep`)

#### Step 1: Generate MD5 Hash for Two Files
```bash
echo "Hello" > file1.txt
echo "World" > file2.txt
md5sum file1.txt file2.txt
```

### Mitigation:
- Use SHA-256 or SHA-3 instead of MD5.

---

## 6. Cryptanalysis Attack (Frequency Analysis)

### Concept:
- Analyzes character frequency to break substitution ciphers.

### Example (Breaking a Caesar Cipher Using `tr` Command)

#### Step 1: Encrypt a Message Using Caesar Cipher (+3 Shift)
```bash
echo "hello world" | tr 'a-z' 'd-za-c'
```

#### Step 2: Decrypt Using Frequency Analysis
```bash
echo "khoor zruog" | tr 'd-za-c' 'a-z'
```

### Mitigation:
- Use modern encryption (AES, RSA) instead of weak ciphers.

---

## 7. Replay Attack

### Concept:
- An attacker captures and replays network requests to gain unauthorized access.

### Example (Capturing and Replaying a Login Request Using `tcpdump`)

#### Step 1: Capture Network Packets
```bash
sudo tcpdump -i eth0 port 80 -w capture.pcap
```

#### Step 2: Replay the Captured Request Using `tcpreplay`
```bash
sudo tcpreplay -i eth0 capture.pcap
```

### Mitigation:
- Use session tokens and timestamps in authentication requests.

---

## 8. Chosen Plaintext Attack (CPA)

### Concept:
- The attacker encrypts a known plaintext and compares it with an intercepted ciphertext.

### Example (Testing AES Encryption Using OpenSSL)

#### Step 1: Encrypt a Known Message
```bash
echo "Attack at dawn" | openssl enc -aes-128-cbc -K 00112233445566778899AABBCCDDEEFF -iv 0102030405060708 -base64
```

### Mitigation:
- Use random IVs (Initialization Vectors) to ensure uniqueness.

---

## 9. Padding Oracle Attack

### Concept:
- Exploits incorrect padding error messages in block ciphers.

### Example (Detecting Padding Oracle Vulnerability)

#### Step 1: Use `PadBuster` to Test a Web App
```bash
perl padbuster http://example.com/encrypt?data=XYZ 16
```

### Mitigation:
- Use authenticated encryption (AES-GCM) instead of CBC mode.

---

## Conclusion
Each attack highlights different cryptographic weaknesses. Using strong encryption, randomization, and secure implementations helps mitigate these attacks.
