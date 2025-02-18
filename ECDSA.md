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
- **Domain parameters**: \((p, a, b, G, n, h)\)
- **Private key**: \( d \)
- **Message**: \( m \)

### Steps:
1. Choose a random integer \( k \) where \( 1 \leq k \leq n-1 \).
2. Compute \( kG = (x_1, y_1) \).
3. Compute \( r = x_1 \mod n \).
4. Compute \( e = \text{HASH}(m) \).
5. Compute \( s = k^{-1} (e + d r) \mod n \).
6. Output the signature \( (r, s) \).

## Signature Verification

### Steps:
1. Verify \( r, s \) belong to \( [1, n-1] \).
2. Compute \( e = \text{HASH}(m) \).
3. Compute \( w = s^{-1} \mod n \).
4. Compute:
   - \( u_1 = e w \mod n \)
   - \( u_2 = r w \mod n \)
5. Compute \( (x_1', y_1') = u_1 G + u_2 Q \).
6. If \( r = x_1' \mod n \), the signature is valid.

## Example Calculation
- \( n = 23 \), \( G = (5, 19) \), \( d = 7 \), \( k = 3 \), \( e = 9 \).
- Compute \( kG \): \( 3G = (10, 4) \).
- Compute \( r = 10 \mod 23 = 10 \).
- Compute \( k^{-1} \mod n \): \( 3^{-1} \mod 23 = 8 \).
- Compute \( s = 8 \times (9 + 7 \times 10) \mod 23 = 11 \).
- Signature: \( (r, s) = (10, 11) \).

## Verification Example
- Compute \( w = 11^{-1} \mod 23 = 21 \).
- Compute \( u_1 = 5 \), \( u_2 = 3 \).
- Compute \( 5G = (12, 7) \), \( 3Q = (10, 4) \).
- Compute \( (x_1', y_1') = (10, y_1') \).
- Since \( r = x_1' \), the signature is valid.
