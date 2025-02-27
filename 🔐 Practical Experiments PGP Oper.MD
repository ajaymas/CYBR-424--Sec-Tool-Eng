# 🔐 Practical Experiments: PGP Operations in Linux

This document provides step-by-step **PGP experiments** using **GnuPG (GPG)** on **Linux/macOS**.

# 🔥 What This Covers:

✅ **PGP Authentication** (Digital Signatures)  
✅ **PGP Confidentiality** (Encryption & Decryption)  
✅ **PGP Authentication + Confidentiality** (Combined Approach)  
✅ **PGP Compression** (Efficient Storage & Transmission)  
✅ **PGP Email Compatibility** (Encoding for Email)  
✅ **PGP Session Keys & Key Management**  

---

## 📌 **1. Setup: Install GnuPG (GPG)**
Install **GnuPG**, an open-source PGP implementation.

```bash
sudo apt install gnupg  # Linux (Debian/Ubuntu)
brew install gnupg      # macOS (Homebrew)
```

Verify installation:
```bash
gpg --version
```

---

## ✅ **2. PGP Authentication: Digital Signatures**
Generate a **PGP key pair** (private & public keys):

## This step will guide you through generating a PGP key pair, which includes both a private key and a public key.

1. **Run the following command to generate the PGP key pair:**

    ```bash
    gpg --full-generate-key
    ```

# Generating a PGP Key Pair in Linux

## Step 1: Run the Key Generation Command
```bash
gpg --full-generate-key
```
**Expected Output:**
```
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) ECC (sign and encrypt)
   (6) ECC (sign only)
   (7) ECC (encrypt only)
   (8) RSA (encrypt only)
   (9) Existing key
Your selection? 
```
**Action:**  
Type `1` and press `Enter` to select **RSA and RSA** (recommended).

---

## Step 2: Choose the Key Size
**Expected Prompt:**
```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072)
```
**Action:**  
Type `4096` and press `Enter` for **stronger encryption**.

---

## Step 3: Set the Key Expiration
**Expected Prompt:**
```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
```
**Action:**  
- Type `0` and press `Enter` if you want the key to never expire.
- Otherwise, enter a duration (e.g., `1y` for one year).

**Confirmation Prompt:**
```
Key does not expire at all
Is this correct? (y/N)
```
**Action:**  
Type `y` and press `Enter` to confirm.

---

## Step 4: Enter Your Identity Information
**Expected Prompt:**
```
GnuPG needs to construct a user ID to identify your key.

Real name: 
```
**Action:**  
Type your **full name** (e.g., `Ajay Kumara`) and press `Enter`.

**Expected Prompt:**
```
Email address: 
```
**Action:**  
Type your **email address** (e.g., `ajay@example.com`) and press `Enter`.

**Expected Prompt:**
```
Comment: 
```
**Action:**  
You can enter a **comment** (e.g., `Research Key`) or **leave it blank** by pressing `Enter`.

---

## Step 5: Confirm User Information
**Expected Prompt:**
```
You selected this USER-ID:
    "Ajay Kumara (Research Key) <ajay@example.com>"

Change (N)ame, (C)omment, (E)mail, or (O)kay/(Q)uit? 
```
**Action:**  
Type `O` and press `Enter` to proceed.

---

## Step 6: Set a Strong Passphrase
**Expected Prompt:**
```
Enter passphrase:
```
**Action:**  
Type a **strong passphrase** (it will not be visible as you type), then press `Enter`.

**Expected Prompt:**
```
Repeat passphrase:
```
**Action:**  
Re-enter your passphrase and press `Enter`.

---

## Step 7: Key Generation Process
At this point, **GPG will generate the key pair**. You may see:
```
gpg: key ABCD1234 marked as ultimately trusted
gpg: Done
```
**Action:**  
To speed up the process, you can **move your mouse** or **type random keys**.

---

## Step 8: Verify the Key
To check if the key was successfully generated, run:
```bash
gpg --list-keys
```
**Expected Output:**
```
/home/user/.gnupg/pubring.kbx
---------------------------------
pub   rsa4096 2025-02-26 [SC]
      ABCD1234567890ABCDEF1234567890ABCDEF1234
uid           [ultimate] Ajay Kumara (Research Key) <ajay@example.com>
sub   rsa4096 2025-02-26 [E]
```

---

## Step 9: Export Your Public Key (Optional)
If you need to share your **public key**, use:
```bash
gpg --armor --export ajay@example.com
```
**Expected Output:**
```
-----BEGIN PGP PUBLIC KEY BLOCK-----
...
-----END PGP PUBLIC KEY BLOCK-----
```
This is your **public key** that others can use to encrypt messages for you.

---

## Step 10: Backup Your Private Key (Optional)
To export and back up your **private key**, use:
```bash
gpg --armor --export-secret-key ajay@example.com > my_private_key.asc
```
Store this file securely, as it is required for decryption.

---

## Final Notes
- Your **private key** is stored securely in `~/.gnupg/`
- Your **public key** can be shared for encryption purposes
- If you forget the **passphrase**, you **cannot** recover your private key

This completes the **PGP key generation process** in Linux! 🚀




















2. **Select the key type:**
 ```bash
   Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) ECC (sign and encrypt)
   (6) ECC (sign only)
   (7) ECC (encrypt only)
   (8) RSA (encrypt only)
   (9) Existing key
Your selection?
    ```
    - When prompted, select the key type. For most users, **RSA** is the recommended choice. Choose **1** to select the RSA option.

3. **Choose the key size:**
    - Next, you will be asked to choose the key size. For stronger encryption, select **4096 bits** by typing **4096**. This is the recommended size for enhanced security.

4. **Set expiration date:**
    - You will then be prompted to set an expiration date for the key. You can either set a specific expiration date or choose **0** to keep it valid indefinitely.

5. **Enter your name and email:**
    - Provide your name and email address when prompted. This is how others will identify you when verifying your PGP key.

6. **Add an optional comment:**
    - You can enter an optional comment (e.g., your organization or purpose for the key) or leave it blank.

7. **Set a strong passphrase:**
    - Finally, set a strong passphrase to protect your private key. Make sure the passphrase is secure and not easily guessable.

After completing these steps, your PGP key pair (private and public keys) will be generated and stored in your keyring.
```

Experiment 1: PGP Authentication (Digital Signature)
Objective
Verify message authenticity using SHA-1 hashing and RSA digital signatures.

Steps
1️⃣ Sender Creates a Message
```
echo "Confidential Research Document" > message.txt
```
2️⃣ Generate SHA-1 Hash of Message
```
sha1sum message.txt > message_hash.txt
```
Generates a 160-bit SHA-1 hash of message.txt.
3️⃣ Encrypt Hash with Sender’s Private Key (Digital Signature)
```
gpg --output message.sig --sign message.txt
```
Signs message.txt by encrypting its hash with the sender’s private key.
4️⃣ Send message.txt and message.sig to the Receiver
Use any method (email, scp, etc.) to send the files.

5️⃣ Receiver Verifies the Signature
```
gpg --verify message.sig message.txt
```
GPG extracts the signature, decrypts the hash using the sender’s public key, and compares it with the newly generated SHA-1 hash.
If both match, the message is authentic.


## Experiment 2: PGP Confidentiality (Encryption & Decryption)
Objective
Encrypt a message using a session key and protect the session key using RSA.

Steps
1️⃣ Sender Generates a Message
```
echo "Confidential Research Findings" > secret_message.txt
```
2️⃣ Encrypt Message Using Receiver’s Public Key
```
gpg --output secret_message.gpg --encrypt --recipient receiver@example.com secret_message.txt
```
Uses CAST-128 / IDEA / 3DES for encryption.
Generates a 128-bit session key (random).
Encrypts the session key using the receiver’s public key.
Attaches the encrypted session key to the message.
3️⃣ Send secret_message.gpg to the Receiver
4️⃣ Receiver Decrypts the Message Using Private Key

```
gpg --output decrypted_message.txt --decrypt secret_message.gpg
```
Decrypts the session key using RSA and receiver’s private key.
Uses the session key to decrypt secret_message.gpg.
✅ Receiver Can Now Read the Original Message!
🔹 Experiment 3: Combining Authentication & Confidentiality
Objective
Ensure both authentication and confidentiality by signing & encrypting a message.

Steps
1️⃣ Sender Creates & Signs the Message
```
gpg --output signed_message.txt --sign secret_message.txt
```
Attaches a digital signature to secret_message.txt.
2️⃣ Encrypt Signed Message
```
gpg --output secure_message.gpg --encrypt --recipient receiver@example.com signed_message.txt
```
Encrypts both message & signature.
Encrypts the session key using RSA.
3️⃣ Send secure_message.gpg to the Receiver
4️⃣ Receiver Decrypts the Message
```
gpg --output decrypted_signed_message.txt --decrypt secure_message.gpg
```
Recovers both the message and signature.
5️⃣ Receiver Verifies the Signature
```
gpg --verify decrypted_signed_message.txt
```
Ensures the sender actually signed the message.
📌 Summary
✅ Authentication: Verifies sender using SHA-1 + RSA signatures.
✅ Confidentiality: Encrypts message using CAST-128 / 3DES + RSA.
✅ Both Together: Sign + Encrypt to ensure privacy & authenticity.

This real-world PGP workflow helps protect sensitive emails. 🚀
