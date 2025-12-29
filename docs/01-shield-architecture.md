# Shield Architecture Deep Dive

## Overview

Shield's architecture is a sophisticated system that combines smart contracts, decentralized storage, and client-side encryption to enable secure, private data sharing on the blockchain. This document explores the technical foundations of how Shield works.

## System Components

### 1. Smart Contract Layer

The core of Shield's on-chain logic is implemented through the Shield smart contract, which manages:

#### AccessPolicy Structure

```solidity
struct AccessPolicy {
  address sender;          // Who created this policy
  address recipient;       // Who can access the content
  uint256 expiry;         // When access expires
  uint32 maxAttempts;     // Maximum access attempts allowed
  uint32 attempts;        // Current access attempt count
  bool valid;             // Whether policy is still valid
}
```

This structure is the gateway keeper for all content access. Every shared link corresponds to a unique AccessPolicy stored immutably on the blockchain.

#### Key Contract Functions

**createPolicy(policyId, recipient, expiry, maxAttempts)**
- Validates sender has authorization to create policies
- Stores the AccessPolicy in the contract's mapping
- Emits PolicyCreated event for off-chain tracking
- Returns the policyId for link construction

**logAttempt(policyId, success)**
- Records access attempts against a policy
- Enforces attempt limits
- Emits VerificationAttempt event
- Prevents brute force attacks

**verifyAccess(policyId, recipient)**
- Checks if recipient is authorized
- Validates policy hasn't expired
- Ensures attempt limits not exceeded
- Returns access grant decision

### 2. Encryption Layer

#### Client-Side Encryption

Before any data leaves the user's device:

1. **Key Generation**: A cryptographically secure random key is generated
2. **Serialization**: Content is converted to bytes
3. **Encryption**: AES-256 symmetric encryption is applied
4. **Key Splitting**: The encryption key is NOT sent to the blockchain

#### Why No On-Chain Keys?

- **Security**: Keys stored on-chain could be extracted through contract analysis
- **Privacy**: No single source that reveals complete decryption capability
- **Architecture**: Keys remain only in the secure link URL

### 3. Decentralized Storage (IPFS/Pinata)

Encrypted content is stored off-chain:

```
┌─────────────────────┐
│  Encrypted Content  │
│     (AES-256)       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  IPFS/Pinata       │
│  Content Hash      │
│  (CID)             │
└──────────┬──────────┘
           │
           ▼
    Stored On-Chain
    (Smart Contract)
```

**Benefits**:
- Content is not stored on expensive blockchain storage
- IPFS ensures content availability and redundancy
- Content hash allows verification of integrity
- Distributed storage prevents single points of failure

### 4. Link Generation System

A secure link encodes all necessary access information:

```
https://shield.app/share/[POLICY_ID]/[ENCRYPTED_KEY]
                         ↓               ↓
              On-chain verification    URL parameter
                (contract lookup)      (client-side decryption)
```

## Data Flow Architecture

### Sender Workflow

```
┌──────────────┐
│ Sender       │
│ clicks       │
│ "Share"      │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────┐
│ Client encrypts content with key  │
│ (AES-256)                        │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Encrypted content → IPFS          │
│ Returns CID (content hash)        │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ createPolicy() transaction        │
│ - sender, recipient, expiry       │
│ - maxAttempts                     │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Smart contract stores policy      │
│ Emits PolicyCreated event         │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Generate link:                   │
│ /share/[policyId]/[encKey]       │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Share link with recipient         │
│ (via email, messaging, etc.)      │
└──────────────────────────────────┘
```

### Recipient Workflow

```
┌──────────────────────────────────┐
│ Recipient receives link           │
│ Clicks link                       │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Extract policyId and encKey       │
│ from link parameters              │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Query verifyAccess(policyId)      │
│ Smart contract checks:            │
│ - recipient is authorized         │
│ - policy not expired              │
│ - attempt limit not exceeded      │
└──────┬───────────────────────────┘
       │
       ├─ Access Denied ─► Show Error
       │
       ▼ Access Granted
┌──────────────────────────────────┐
│ Fetch from IPFS using CID         │
│ (encrypted content)               │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Decrypt using key from link       │
│ (AES-256 decryption)              │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Display decrypted content         │
│ to recipient                      │
└──────┬───────────────────────────┘
       │
       ▼
┌──────────────────────────────────┐
│ Call logAttempt() on contract     │
│ Records access attempt            │
│ Emits VerificationAttempt event   │
└──────────────────────────────────┘
```

## Security Architecture

### Access Control

1. **Authentication**: Wallet-based authentication ensures sender identity
2. **Authorization**: Smart contract verifies recipient address matches policy
3. **Expiration**: Time-based access limits prevent indefinite sharing
4. **Rate Limiting**: maxAttempts prevents brute force decryption attempts

### Cryptographic Security

1. **AES-256**: Industry-standard symmetric encryption
2. **Secure Key Generation**: Cryptographically secure random number generation
3. **Key Isolation**: Keys never stored on blockchain
4. **Integrity Verification**: Content hash prevents tampering

### Privacy Architecture

1. **Sender Privacy**: Policies don't expose detailed content
2. **Recipient Privacy**: Only recipient address is stored (not identifiable info)
3. **Content Privacy**: Encrypted content cannot be viewed without key
4. **Metadata Privacy**: Limited metadata stored on-chain

## Performance Considerations

### On-Chain Operations

- **createPolicy()**: ~100k gas (varies by implementation)
- **verifyAccess()**: ~50k gas (read-heavy)
- **logAttempt()**: ~80k gas (write operation)

### Off-Chain Operations

- **IPFS Upload**: Depends on content size (typically <5s for KB-MB range)
- **IPFS Retrieval**: Fast distributed retrieval
- **Encryption/Decryption**: Milliseconds (client-side, hardware-accelerated)

## Scalability

### Current Limitations

- Gas costs for each policy creation
- On-chain state grows with number of policies
- IPFS retrieval depends on network conditions

### Future Optimizations

- Batch policy creation (multiple recipients in single tx)
- Rollup integration for reduced gas costs
- Off-chain signature verification
- Decentralized access control (multi-sig, DAOs)

## Integration Points

### Frontend Integration

```javascript
// Simplified example
const shield = new ShieldClient(contractAddress);

// Sender: Create and share
const { policyId, encryptionKey } = await shield.shareContent(
  content,
  recipientAddress,
  expiryTime
);
const link = `https://shield.app/share/${policyId}/${encryptionKey}`;

// Recipient: Access
const content = await shield.accessContent(policyId, encryptionKey);
```

### Blockchain Integration

```solidity
// Contract interactions
shield.createPolicy(policyId, recipient, expiry, maxAttempts);
shield.verifyAccess(policyId, msgSender);
shield.logAttempt(policyId, success);
```

## Conclusion

Shield's architecture achieves the delicate balance between security, privacy, and usability. By separating concerns (encryption on client, storage on IPFS, access control on blockchain), it provides a robust system for secure data sharing without relying on centralized intermediaries.
