# Shield Privacy Guide

> **Comprehensive guide to Shield: Secure onchain data sharing with privacy preservation. Covers encryption, cryptography, privacy-preserving architectures, and best practices for protecting sensitive content.**

## Table of Contents

- [Overview](#overview)
- [What is Shield?](#what-is-shield)
- [How Shield Works](#how-shield-works)
- [The Secure Link Generation Flow](#the-secure-link-generation-flow)
- [Data Security & Privacy](#data-security--privacy)
- [Privacy Fundamentals](#privacy-fundamentals)
- [Cryptographic Techniques](#cryptographic-techniques)
- [Best Practices](#best-practices)
- [Advanced Topics](#advanced-topics)
- [Getting Started](#getting-started)
- [Contributing](#contributing)

## Overview

This repository contains comprehensive documentation about Shield, an innovative onchain dApp designed for secure and private data sharing. Shield leverages blockchain technology and advanced cryptographic techniques to protect user data while maintaining transparency and security of distributed ledgers.

Shield enables individuals and organizations to securely share sensitive content onchain with end-to-end encryption and cryptographic verification, ensuring that only authorized recipients can access the shared data.

## What is Shield?

Shield is a Web3 application that enables users to securely share sensitive content onchain with end-to-end encryption and privacy preservation. Built on blockchain principles, Shield combines the transparency and security of distributed ledgers with advanced cryptographic techniques to protect user data.

### Key Features

- **End-to-End Encryption**: All data is encrypted before being shared onchain
- **Smart Contract Verification**: Cryptographic proofs verify data integrity and access permissions
- **Privacy Preservation**: User identities and sensitive data are protected through advanced cryptographic schemes
- **Decentralized Architecture**: No single point of failure; data is distributed across the network
- **Onchain Verification**: All access controls and permissions are verified through smart contracts

## How Shield Works

Shield operates as a decentralized system combining multiple technologies:

### Architecture Overview

1. **Smart Contracts**: The Shield smart contract manages access control policies, permissions, and verification logic
2. **Data Storage**: Encrypted content is stored on IPFS/Pinata (decentralized storage) or alternative storage solutions
3. **Metadata**: Access policies, recipient information, and cryptographic proofs are stored onchain
4. **Encryption Layer**: End-to-end encryption ensures only authorized recipients can decrypt content

### Core Components

- **AccessPolicy Smart Contract**: Manages who can access which content and under what conditions
- **Encryption/Decryption Engine**: Handles cryptographic operations on both client and chain sides
- **Link Generation System**: Creates secure, verifiable links that grant temporary or permanent access
- **Verification Mechanisms**: Proves data integrity and validates access permissions onchain

## The Secure Link Generation Flow

This section details the complete process when a sender generates a secure link to share content:

### Step 1: User Initiates Content Sharing

When a sender clicks the "Generate Link" button in Shield:

```
[Sender clicks "Generate Link"] 
  ↓
[Client captures content to share]
  ↓
[Content is prepared for encryption]
```

### Step 2: Encryption Process

```
[Content is serialized into bytes]
  ↓
[Generate encryption key (client-side)]
  ↓
[Apply symmetric encryption (AES-256)]
  ↓
[Encrypted content is prepared]
```

### Step 3: Storage Preparation

```
[Encrypted content is sent to IPFS/Pinata]
  ↓
[IPFS returns content hash (CID)]
  ↓
[Content hash is stored for retrieval]
```

### Step 4: Smart Contract Interaction

This is the critical onchain portion:

```
[Client prepares AccessPolicy struct with:]
  - sender address
  - recipient address(es)
  - expiry timestamp
  - max attempts allowed
  - other conditions
  ↓
[Client calls createPolicy() on Shield contract]
  ↓
[Smart contract verifies sender has authorization]
  ↓
[Contract stores AccessPolicy and generates policyId]
  ↓
[PolicyCreated event is emitted]
```

### Step 5: Link Construction

```
[Client constructs secure link with:]
  - Policy ID (from smart contract)
  - Encryption key (NOT stored onchain)
  - Link metadata
  ↓
[Link is encoded: /share/[policyId]/[encryptedKey]]
  ↓
[Link is returned to sender]
```

### Step 6: Link Sharing

```
[Sender shares link with recipient]
  ↓
[Recipient clicks link]
  ↓
[Client queries Shield contract for AccessPolicy]
  ↓
[Contract verifies recipient is authorized]
  ↓
[Contract checks expiry and attempt limits]
```

### Step 7: Content Decryption

```
[If authorized:]
  ↓
[Client retrieves encrypted content from IPFS using CID]
  ↓
[Client extracts encryption key from URL]
  ↓
[Client decrypts content using key]
  ↓
[Decrypted content is displayed to recipient]
  ↓
[logAttempt() is called on contract to track access]
```

## Data Security & Privacy

### How Shield Protects Your Data

1. **No Secret Key Storage Onchain**: Encryption keys are never stored in smart contracts, keeping them from being exposed through contract interactions

2. **Client-Side Encryption**: Encryption happens before data leaves the user's device

3. **Decentralized Storage**: Content is stored on IPFS/Pinata, not on a centralized server

4. **Access Control Lists**: Smart contracts enforce who can access what

5. **Attempt Limiting**: Recipients can be limited in how many times they attempt access

6. **Expiry Controls**: Content access can be time-limited

### Data Flow

```
Sender's Device
  ↓
[Content + Encryption Key]
  ↓
Encryption (AES-256)
  ↓
IPFS/Pinata Storage
  (Encrypted Content)
  ↓
Smart Contract Storage
  (AccessPolicy, policyId)
  ↓
Link: [policyId]/[key]
  ↓
Shared with Recipient
  ↓
Recipient's Device
  ↓
Verify AccessPolicy onchain
  ↓
Retrieve Encrypted Content
  ↓
Decrypt using key from link
  ↓
View Content
```

### What Shield Never Stores

- **Secret encryption keys**: These remain only in the secure link URL
- **Unencrypted content**: Content is always encrypted before storage
- **User credentials**: Shield uses wallet-based authentication

## Privacy Fundamentals

Privacy in the context of data sharing involves protecting information from unauthorized access while maintaining system functionality. Key privacy concepts include:

- **Confidentiality**: Only authorized parties can access sensitive data
- **Integrity**: Data has not been modified by unauthorized parties
- **Authentication**: Verifying the identity of parties involved in communication
- **Non-repudiation**: Proving that a party performed an action

## Cryptographic Techniques

Shield employs several cryptographic methods:

### Symmetric Encryption

- **AES-256**: Encrypts the actual content being shared
- **Key derivation**: Secure methods generate encryption keys

### Asymmetric Cryptography

- **ECDSA**: Elliptic Curve Digital Signature Algorithm for wallet signing
- **Public/Private Key Pairs**: Used for authentication and access control

### Hash Functions

- **Keccak-256**: Hash function used throughout the smart contract
- **Content Hashing**: Verifies data integrity

### Smart Contract Verification

- **On-Chain Proofs**: Cryptographic verification of access permissions
- **Event Logging**: Immutable records of access attempts

## Best Practices

### For Senders

1. **Verify Recipient Address**: Ensure you're sharing with the correct recipient onchain
2. **Set Appropriate Expiry Times**: Limit how long links remain valid
3. **Monitor Access Attempts**: Use the contract's event logs to track who accessed your content
4. **Use Strong Encryption Keys**: Rely on the client-side encryption system
5. **Audit Access Logs**: Regularly review logAttempt events

### For Recipients

1. **Verify Link Authenticity**: Ensure the link comes from a trusted source
2. **Check Expiry**: Be aware of when access will expire
3. **Protect the Link**: The link contains your encryption key
4. **Understand Access Limits**: Know how many times you can attempt access
5. **Secure Your Device**: Protect your device from malware that could intercept keys

### For Developers

1. **Understand Access Control**: Review the AccessPolicy structure
2. **Implement Proper Error Handling**: Handle failed decryption gracefully
3. **Cache Verification Results**: Reduce redundant contract calls
4. **Validate Smart Contract Interactions**: Ensure proper nonce and gas management
5. **Test Encryption/Decryption**: Verify cryptographic operations thoroughly

## Advanced Topics

### Zero-Knowledge Proofs

Zero-knowledge proofs allow verification of statements without revealing underlying data—applicable to proving access rights without exposing policies.

### Multi-Signature Schemes

Requiring multiple signatures for sensitive operations enhances security for high-value content sharing.

### Threshold Cryptography

Distributing cryptographic keys across multiple parties ensures no single party can compromise the system.

### Homomorphic Encryption

Allows computation on encrypted data without decryption, useful for privacy-preserving analytics.

## Getting Started

New to Shield? Start here:

1. **Understand the basics**: Read about what Shield is and how it works
2. **Learn about privacy**: Understand fundamental privacy concepts
3. **Explore cryptography**: Familiarize yourself with encryption techniques
4. **Review smart contracts**: Understand the Shield contract architecture
5. **Practice safely**: Use testnet before mainnet
6. **Monitor access**: Track all access attempts through event logs

## Contributing

We welcome contributions to improve this guide! Please see our contribution guidelines for more information on how to contribute to the Shield documentation.

---

**Last Updated**: 2025
**Maintainer**: Babsbuild
**License**: MIT
