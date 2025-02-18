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

