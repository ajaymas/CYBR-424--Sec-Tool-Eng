# Hands-on Lab: RSA and ECC Encryption/Decryption using OpenSSL

This lab demonstrates **RSA encryption/decryption** and **Elliptic Curve Cryptography (ECC) key exchange with AES encryption** using Linux `openssl` commands.

---

## Part 1: RSA — Public Key Encryption/Decryption

> Notes: RSA directly encrypts only small messages (≤ key_size − padding). For larger files, use hybrid encryption (RSA + AES).

### Steps

```bash
# 1) Create a small plaintext file
echo "Hello from RSA test" > plain.txt

# 2) Generate a 2048-bit RSA private key
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out rsa_priv.pem

# 3) Extract the public key
openssl pkey -in rsa_priv.pem -pubout -out rsa_pub.pem

# 4) Encrypt the plaintext with the public key using OAEP
openssl rsautl -encrypt -oaep -inkey rsa_pub.pem -pubin -in plain.txt -out rsa_cipher.bin

# 5) Decrypt the ciphertext with the private key
openssl rsautl -decrypt -oaep -inkey rsa_priv.pem -in rsa_cipher.bin -out rsa_decrypted.txt

# 6) Verify the decrypted contents
cat rsa_decrypted.txt
```

---

## Part 2: ECC — ECDH-derived AES Encryption/Decryption

- **Mathematical Basis**  
  - ECC is built on the algebraic structure of **elliptic curves** over finite fields.  
  - The general curve equation is:  
    \[ y^2 = x^3 + ax + b \pmod{p} \]  
    where `a`, `b` are curve parameters and `p` is a prime.  

- **Security from Hard Problems**  
  - Security relies on the **Elliptic Curve Discrete Logarithm Problem (ECDLP)** â€” very hard to solve, even with powerful computers.  
  - Unlike RSA (factoring integers), ECC offers stronger security with smaller key sizes.  

- **Key Size Advantage**  
  - ECC provides equivalent security with much shorter keys:  
    - 256-bit ECC = 3072-bit RSA  
    - 384-bit ECC = 7680-bit RSA  
    - 521-bit ECC = 15,360-bit RSA  

- **Performance Benefits**  
  - Faster key generation, encryption, and decryption.  
  - Lower memory and bandwidth usage.  
  - Efficient for devices with limited resources (IoT, smartcards, mobile).  

- **Common Curves**  
  - **prime256v1 (secp256r1 / P-256)** = Widely used, standardized by NIST.  
  - **secp384r1, secp521r1** = Higher security levels.  
  - **Curve25519** = Modern, designed for speed and security, used in TLS and SSH.  
  - **Ed25519** = Signature scheme using Curve25519, very fast and secure.  

- **Applications of ECC**  
  - **Encryption:** Elliptic Curve Integrated Encryption Scheme (ECIES).  
  - **Digital Signatures:** Elliptic Curve Digital Signature Algorithm (ECDSA), EdDSA.  
  - **Key Exchange:** Elliptic Curve Diffie-Hellman (ECDH).  

- **Standardization & Usage**  
  - Adopted by NIST, ISO, ANSI, and used in TLS (HTTPS), SSH, Bitcoin, and modern cryptosystems.  
  - Preferred in modern protocols (TLS 1.3, Signal messenger, WhatsApp).  

- **Quantum Threat**  
  - Vulnerable to Shorâ€™s algorithm on large-scale quantum computers (same as RSA).  
  - Post-quantum cryptography (lattice-based, code-based, etc.) is being developed to replace it in the future.  


> ECC is usually used for **key exchange** (ECDH), not direct encryption. Here we derive a shared secret, turn it into an AES key, and use it for symmetric encryption.

We simulate two parties: **Alice** and **Bob**.

### Steps

```bash
# 1) Generate Alice’s EC key pair (prime256v1)
openssl ecparam -name prime256v1 -genkey -noout -out alice_priv.pem
openssl ec -in alice_priv.pem -pubout -out alice_pub.pem

# 2) Generate Bob’s EC key pair
openssl ecparam -name prime256v1 -genkey -noout -out bob_priv.pem
openssl ec -in bob_priv.pem -pubout -out bob_pub.pem

# 3) Create a plaintext file
echo "Hello from ECC-derived-key test" > ecc_plain.txt

# 4) Alice derives a shared secret using her private key and Bob’s public key
openssl pkeyutl -derive -inkey alice_priv.pem -peerkey bob_pub.pem -out alice_secret.bin

# 5) Hash the raw secret into a 256-bit key
openssl dgst -sha256 -binary alice_secret.bin > alice_key.bin

# 6) Convert key to hex and generate an IV
KEYHEX=$(xxd -p alice_key.bin | tr -d '\n')
IVHEX=$(openssl rand -hex 16)
echo "KEYHEX=$KEYHEX"
echo "IVHEX=$IVHEX"

# 7) Encrypt plaintext with AES-256-CBC
openssl enc -aes-256-cbc -in ecc_plain.txt -out ecc_cipher.enc -K $KEYHEX -iv $IVHEX

# 8) Bob derives the same shared secret using his private key and Alice’s public key
openssl pkeyutl -derive -inkey bob_priv.pem -peerkey alice_pub.pem -out bob_secret.bin
openssl dgst -sha256 -binary bob_secret.bin > bob_key.bin

# 9) Verify keys match (optional)
KEYHEX_BOB=$(xxd -p bob_key.bin | tr -d '\n')
[ "$KEYHEX" = "$KEYHEX_BOB" ] && echo "keys match" || echo "keys differ"

# 10) Bob decrypts using the same AES key and IV
openssl enc -d -aes-256-cbc -in ecc_cipher.enc -out ecc_decrypted.txt -K $KEYHEX_BOB -iv $IVHEX

# 11) View decrypted result
cat ecc_decrypted.txt
```

---

## Notes & Pitfalls

- RSA can encrypt only short messages; use **RSA + AES hybrid** for larger files.
- Always use **OAEP padding** with RSA for security.
- In ECC, the **shared secret must be hashed** (or run through HKDF) to produce a usable AES key.
- The **IV must be shared** along with ciphertext (safe to send in plaintext).
- OpenSSL versions may differ slightly; check `man openssl-pkeyutl` and `man openssl-enc`.

---

✅ End of Lab
