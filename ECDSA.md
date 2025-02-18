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

ECDSA is a cryptographic algorithm used to ensure authenticity and integrity in digital communications. It operates on elliptic curve cryptography, offering strong security with smaller key sizes.

## Signature Generation Steps

Given:

- Domain parameters: (p, a, b, G, n, h), where G is the generator point on the elliptic curve.
- Private key: d.
- Message: m.

### Step 1: Select a Random Number

Choose a random integer k, where 1 ≤ k ≤ n − 1.

### Step 2: Compute kG

Multiply the generator point G by k to obtain:
kG = (x1, y1)

### Step 3: Compute r

Convert x1 to an integer and compute:
r = x1 mod n

If r = 0, choose another k.

### Step 4: Compute the Hash of the Message

Calculate the hash of the message:
e = HASH(m)

For example, SHA-256 is commonly used.

### Step 5: Compute s

Compute:
s = k⁻¹ (e + d * r) mod n

where k⁻¹ is the modular inverse of k modulo n.

### Step 6: Output the Signature

The digital signature is:
(r, s)

## Example Calculation

Let's assume:

- n = 23 (a small prime for demonstration).
- G = (5, 19) is a known generator point.
- d = 7 (private key).
- k = 3 (random number).
- e = HASH(m) = 9.

### Step 1: Compute kG

Assume:
3G = (10, 4)

Thus, x1 = 10.

### Step 2: Compute r

r = 10 mod 23 = 10

### Step 3: Compute k⁻¹ mod n

Since k = 3:
3⁻¹ mod 23 = 8

### Step 4: Compute s

s = 8 × (9 + 7 × 10) mod 23  
s = 8 × (9 + 70) mod 23  
s = 8 × 79 mod 23  
s = 632 mod 23 = 11

Thus, the signature is (r, s) = (10, 11).

## Signature Verification

At the receiver’s end, the signature is verified as follows:

### Step 1: Check r, s Range

Verify whether r, s belong to [1, n − 1].

Since r = 10 and s = 11, both are valid.

### Step 2: Compute e

e = HASH(m) = 9

### Step 3: Compute w

w = s⁻¹ mod n

Since s = 11:
11⁻¹ mod 23 = 21

### Step 4: Compute u₁, u₂

u₁ = e * w mod n = 9 * 21 mod 23 = 189 mod 23 = 5  
u₂ = r * w mod n = 10 * 21 mod 23 = 210 mod 23 = 3

### Step 5: Compute (x1', y1')

Assume:
5G = (12, 7), 3Q = (10, 4)

Adding points:
(12, 7) + (10, 4) = (10, y1')

### Step 6: Check Validity

Since x1' = 10 and r = 10, the signature is valid.

