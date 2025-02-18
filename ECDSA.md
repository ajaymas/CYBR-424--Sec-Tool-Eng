# Elliptic Curve Digital Signature Algorithm (ECDSA) on Linux

Elliptic Curve Digital Signature Algorithm (ECDSA) is widely used for secure digital signatures in cryptographic applications. It is more efficient than traditional algorithms like RSA, providing the same level of security with smaller key sizes.

## **Practical Steps for ECDSA on Linux**

We will go through the following steps:

- Generate an ECDSA key pair
- Create a message file
- Sign the message
- Verify the signature

We will use the `openssl` command-line tool.

## **Step 1: Generate an ECDSA Private Key**

Run the following command to generate an ECDSA private key using the `prime256v1` (also known as `secp256r1`) curve:

```bash
openssl ecparam -name prime256v1 -genkey -noout -out ecdsa_private.pem
```

- `ecparam`: Generates elliptic curve parameters.
- `-name prime256v1`: Specifies the curve (you can also use `secp384r1` or `secp521r1` for stronger security).
- `-genkey`: Generates a private key.
- `-noout`: Hides the curve parameters from output.
- `-out ecdsa_private.pem`: Saves the private key to a file.

## **Step 2: Extract the Public Key**

Now, extract the public key from the private key:

```bash
openssl ec -in ecdsa_private.pem -pubout -out ecdsa_public.pem
```

- `ec`: Handles EC key processing.
- `-in ecdsa_private.pem`: Input private key.
- `-pubout`: Extracts the public key.
- `-out ecdsa_public.pem`: Saves the public key.

## **Step 3: Create a Sample Message**

Let's create a sample message file:

```bash
echo "This is a test message for ECDSA signing." > message.txt
```

## **Step 4: Sign the Message**

To sign the message file, we first generate a hash of it and then sign it using the private key.

```bash
openssl dgst -sha256 -sign ecdsa_private.pem -out signature.bin message.txt
```

- `dgst`: Computes message digest.
- `-sha256`: Uses SHA-256 for hashing.
- `-sign ecdsa_private.pem`: Signs with the private key.
- `-out signature.bin`: Saves the binary signature.

## **Step 5: Verify the Signature**

To verify the signature using the public key:

```bash
openssl dgst -sha256 -verify ecdsa_public.pem -signature signature.bin message.txt
```

- `-verify ecdsa_public.pem`: Uses the public key for verification.
- `-signature signature.bin`: Specifies the signature file.

If the signature is valid, you will see:

```
Verified OK
```

Otherwise, an error message will be displayed.

## **Alternative: Base64 Encoding for Readability**

Since `signature.bin` is in binary format, you may want to convert it to Base64 for easier handling:

```bash
base64 signature.bin > signature.b64
```

To decode:

```bash
base64 -d signature.b64 > signature.bin
```

## **Conclusion**

ECDSA is efficient and widely used in security protocols like TLS, SSH, and cryptocurrencies. Using `openssl`, we generated keys, signed a message, and verified the signature on Linux.



## Elliptic Curve Digital Signature Algorithm (ECDSA)

## Signature Generation

### Given:
- **Domain parameters**: \((p,a,b,G,n,h)\) where \(G\) is the generator point on the elliptic curve.
- **Private key**: \(d\)
- **Message**: \(m\)

### Steps:
1. Choose a **random number** \( k \), where \( 1 \leq k \leq n-1 \).
2. Compute \( kG = (x_1, y_1) \), where \( G \) is the generator point.
3. Compute \( r = x_1 \mod n \). If \( r = 0 \), select another \( k \).
4. Compute \( e = \text{HASH}(m) \) (e.g., using SHA-256).
5. Compute \( s = k^{-1} (e + d r) \mod n \).
6. The signature is \((r,s)\).

### Example Calculation:
Let’s assume:
- \( n = 23 \) (a small prime for demonstration).
- \( G = (5,19) \) is a known generator point.
- \( d = 7 \) (private key).
- \( k = 3 \) (random number).
- \( e = \text{HASH}(m) = 9 \).

Now:
1. Compute \( kG \):  
   Assume \( 3G = (10,4) \), so \( x_1 = 10 \).
2. Compute \( r \):  
   \( r = 10 \mod 23 = 10 \).
3. Compute \( k^{-1} \mod n \):  
   Since \( k = 3 \), we compute \( 3^{-1} \mod 23 \), which is **8**.
4. Compute \( s \):  
   \[
   s = 8 \times (9 + 7 \times 10) \mod 23
   \]
   \[
   = 8 \times (9 + 70) \mod 23
   \]
   \[
   = 8 \times 79 \mod 23
   \]
   \[
   = 632 \mod 23 = 11
   \]
   Thus, the **signature** is \( (r, s) = (10, 11) \).

---

## Signature Verification

The receiver verifies the signature by:

1. **Check if \( r, s \) belong to** \( [1, n-1] \).
   - Since \( r = 10 \) and \( s = 11 \), both are in \( [1,22] \), so we continue.
2. Compute \( e = \text{HASH}(m) = 9 \).
3. Compute \( w = s^{-1} \mod n \).  
   - \( 11^{-1} \mod 23 = 21 \).
4. Compute \( u_1 \) and \( u_2 \):  
   \[
   u_1 = e w \mod n = 9 \times 21 \mod 23 = 189 \mod 23 = 5
   \]
   \[
   u_2 = r w \mod n = 10 \times 21 \mod 23 = 210 \mod 23 = 3
   \]
5. Compute \( (x_1', y_1') = u_1G + u_2Q \).  
   - Assume \( 5G = (12,7) \) and \( 3Q = (10,4) \).
   - Adding points: \( (12,7) + (10,4) = (10,y_1) \).
6. The signature is valid if \( r = x_1' \mod n \).  
   - Since \( x_1' = 10 \) and \( r = 10 \), the signature is **valid**.

---

## Why Verification Works

We started with:
\[
 s = k^{-1} (e + d r) \mod n
\]
Multiplying both sides by \( s^{-1} \), we get:
\[
 k = s^{-1} (e + d r) = u_1 + u_2 d
\]
From the elliptic curve equation:
\[
 u_1 G + u_2 Q = (u_1 + u_2 d) G = k G
\]
which means:
\[
 r = x_1 \mod n
\]
confirming the validity of the signature.

---

## Conclusion
This step-by-step breakdown shows how ECDSA ensures authenticity and integrity in cryptographic applications using elliptic curve mathematics.
