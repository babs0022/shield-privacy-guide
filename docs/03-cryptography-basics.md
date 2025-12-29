# Cryptography Basics

## What is Cryptography?

Cryptography is the practice of securing information by converting it into a code that only authorized parties can decode. It protects confidentiality, ensures authenticity, and provides non-repudiation.

## Symmetric Encryption

Symmetric encryption uses a single key to both encrypt and decrypt data. Examples include AES and DES.

**Advantages:** Fast, efficient for large data volumes
**Disadvantages:** Key distribution problem

## Asymmetric Encryption

Asymmetric encryption uses two related keys - a public key for encryption and a private key for decryption.

**Advantages:** Solves key distribution problem
**Disadvantages:** Slower than symmetric encryption

## Hash Functions

Hash functions produce fixed-size digital fingerprints of data. They're one-way functions - you cannot reverse them.

**Uses:**
- Data integrity verification
- Password storage
- Digital signatures

## Digital Signatures

Digital signatures authenticate the origin and integrity of data using asymmetric cryptography.

**Process:**
1. Hash the message
2. Encrypt the hash with private key
3. Send message + encrypted hash
4. Recipient verifies with public key

## Public Key Infrastructure (PKI)

PKI manages cryptographic keys and digital certificates for authentication and encryption.

## AES (Advanced Encryption Standard)

AES is the standard symmetric encryption algorithm widely used globally. It supports 128, 192, and 256-bit key lengths.

## Elliptic Curve Cryptography (ECC)

ECC provides equivalent security to RSA with smaller key sizes, making it more efficient for resource-constrained environments.

## Conclusion

Understanding cryptographic fundamentals is essential for implementing privacy-preserving systems like Shield.
