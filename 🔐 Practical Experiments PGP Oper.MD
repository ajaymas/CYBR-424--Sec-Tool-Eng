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

```
sudo apt install gnupg  # Linux (Debian/Ubuntu)
brew install gnupg      # macOS (Homebrew)
```

Verify installation:
```
gpg --version
```

---

## ✅ **2. PGP Authentication: Digital Signatures**
Generate a **PGP key pair** (private & public keys):

## This step will guide you through generating a PGP key pair, which includes both a private key and a public key for a sender (Exmaple Bill Gates).

## Step 1: Run the following command to generate the PGP key pair:**

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
Type your **full name** (e.g., `Bill Gates`) and press `Enter`.

**Expected Prompt:**
```
Email address: 
```
**Action:**  
Type your **email address** (e.g., `billgates@example.com`) and press `Enter`.

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
    "Bill Gates (Research Key) <billgates@example.com>"

Change (N)ame, (C)omment, (E)mail, or (O)kay/(Q)uit? 
```
**Action:**  
Type `O` or letter O not Zero and press `Enter` to proceed.

---

## Step 6: Set a Strong Passphrase
**Expected Prompt:**
```
Enter passphrase:
```
**Action:**  
Type a **strong passphrase** (it will not be visible as you type example `abcd1234` enter before time out maximum 5 sec), then press `Enter`.

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
/home/remnux/.gnupg/pubring.kbx
-------------------------------
pub   rsa4096 2025-02-25 [SC]
      D34F9FE72988136EDA323CA86B7286AD91E90658
uid           [ultimate] Ajay kumara (Research key) <ajay.kumara@gmail.com>
sub   rsa4096 2025-02-25 [E]

pub   rsa4096 2025-02-27 [SC] [expires: 2026-02-27]
      688AD85073241590217B2506D4F6A4797B5E339A
uid           [ultimate] Bill gates (Research Key) <billgates@example.com>
sub   rsa4096 2025-02-27 [E] [expires: 2026-02-27]

```

---

## Step 9: Export Your Public Key (Optional)
If you need to share your **public key**, use:
```bash
gpg --armor --export billgates@example.com
```
**Expected Output:**
```
-----BEGIN PGP PUBLIC KEY BLOCK-----
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
QWehUUKSjCmx/EOC/tNJWidOCWLk6Fwyfw==
=S0iG
-----END PGP PUBLIC KEY BLOCK-----
```
This is your **public key** that others can use to encrypt messages for you.

---

## Step 10: Backup Your Private Key (Optional)
To export and back up your **private key**, use:
```bash
gpg --armor --export-secret-key billgates@example.com > my_private_key.asc
```
Store this file securely, as it is required for decryption.

---

## Final Notes
- Your **private key** is stored securely in `~/.gnupg/`
- Your **public key** can be shared for encryption purposes
- If you forget the **passphrase**, you **cannot** recover your private key

This completes the **PGP key generation process** in Linux! 🚀


Now, it's time to generate the private and public keys for the receiver(example ajayk and his email will be used as ajayk@gmail.com). However, keep the receiver's public key in the sender's public key ring (Bill Gates).

To do this, repeat steps 1 to 8, but in step 9, follow these commands below in step 9 to store the receiver's public key information in a sender (Bill Gates) public keyring :

Step 9: Export Your Public Key (Optional)

If you need to share your public key, use:
```
gpg --armor --export billgates@example.com > receiver_public_key.asc
```

## Experiment 1: Sending and Receiving Emails Securely with PGP

### Objective

To encrypt and sign an email using PGP for confidentiality and authentication.

### Requirements

- GnuPG (GPG) installed (`sudo apt install gnupg` on Linux/macOS or use GPG4Win on Windows).
- An email client supporting PGP (e.g., Thunderbird with Enigmail).
- Key pairs generated for sender and receiver.

### Steps

#### 1. Generate PGP Key Pair

```bash
gpg --full-generate-key
```

- **Reason:** Creates a key pair (public & private) for encryption and signing.
- Choose RSA and key size 4096 bits for strong security.

#### 2. Export Public Key (To Share with Receiver)

```bash
gpg --export -a "user@example.com" > public_key.asc
```

- **Reason:** Public key must be shared for encryption.

#### 3. Import Receiver's Public Key

```bash
gpg --import receiver_public_key.asc
```

- **Reason:** Required to encrypt messages for the recipient.

#### 4. Sign an Email Message

```bash
echo "Confidential Email Content" | gpg --clearsign -a
```

- **Reason:** Ensures integrity and authenticity using the sender’s private key.

#### 5. Encrypt an Email Message

```bash
echo "Confidential Email Content" | gpg --encrypt --sign --armor -r ajayk@example.com > encrypted_email.asc
```

- `--encrypt` → Encrypts data.
- `--sign` → Signs to ensure authenticity.
- `--armor` → Outputs in ASCII for email compatibility.
- `-r receiver@example.com` → Specifies recipient.

#### 6. Decrypt and Verify Signature

Receiver executes:

```bash
gpg --decrypt encrypted_email.asc
```
You will see the decrypted confidential message below.

```
remnux@remnux:~/Desktop/PGP$ gpg --decrypt encrypted_email.asc
gpg: encrypted with 4096-bit RSA key, ID C50E055ADF0404C4, created 2025-02-27
      "ajayk (I will get email from Bill gates) <ajayk@gmail.com>"
Confidential Email Content
gpg: Signature made Thu 27 Feb 2025 12:09:46 AM EST
gpg:                using RSA key 688AD85073241590217B2506D4F6A4797B5E339A
gpg: Good signature from "Bill gates (Research Key) <billgates@example.com>" [ultimate]

```
- **Reason:** Decrypts using the receiver’s private key and verifies the sender's signature.
- The message `Confidential Email Content` recovered from the decryption process


## The following will be discussed later:
✅ **PGP Authentication + Confidentiality** (Combined Approach)  
✅ **PGP Compression** (Efficient Storage & Transmission)  
✅ **PGP Email Compatibility** (Encoding for Email)  
✅ **PGP Session Keys & Key Management**  
