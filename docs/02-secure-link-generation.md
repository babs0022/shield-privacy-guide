# Secure Link Generation & Management in Shield

## Overview

The secure link is the critical bridge between the blockchain's access control and the client's encryption key. This document explains how Shield generates, manages, and validates secure links throughout their lifecycle.

## Link Structure

### Anatomy of a Shield Link

```
https://shield.app/share/[POLICY_ID]/[ENCRYPTED_KEY]/[OPTIONAL_METADATA]
                        └─────────────────────┬─────────────────────┘
                                     Link Components
```

**POLICY_ID**: 
- 32-byte identifier
- Stored on-chain in smart contract
- Enables access verification
- Immutable once created

**ENCRYPTED_KEY**:
- Client-side encryption key
- URL parameter (not stored on-chain)
- Necessary for decryption
- Sensitive - must be protected

**OPTIONAL_METADATA**:
- Share expiry indicators
- Content type hints
- Recipient identifiers (hashed)

### Example Link

```
https://shield.app/share/0x7a4b2c1e9f3d5a8b6c2e4f7a9b1d3e5f/aGVsbG8d29sdGQzd29ybGQ
```

## Link Generation Process

### Step 1: Encryption Key Generation

```javascript
// Client-side: Secure random key generation
function generateEncryptionKey() {
  // Use cryptographically secure randomness
  const key = crypto.getRandomValues(new Uint8Array(32));
  
  // Key properties:
  // - 256 bits (32 bytes) for AES-256
  // - Cryptographically secure source
  // - Never stored on-chain
  // - Included only in URL
  
  return key;
}
```

### Step 2: Content Encryption

```javascript
function encryptContent(content, key) {
  // 1. Serialize content
  const encoder = new TextEncoder();
  const contentBytes = encoder.encode(content);
  
  // 2. Generate IV (Initialization Vector)
  const iv = crypto.getRandomValues(new Uint8Array(16));
  
  // 3. Encrypt using AES-GCM
  const algorithm = {
    name: 'AES-GCM',
    iv: iv
  };
  
  const encryptedData = await crypto.subtle.encrypt(
    algorithm,
    key,
    contentBytes
  );
  
  // 4. Combine IV + encrypted data
  const result = new Uint8Array(iv.length + encryptedData.byteLength);
  result.set(iv, 0);
  result.set(new Uint8Array(encryptedData), iv.length);
  
  return result;
}
```

### Step 3: IPFS Upload

```javascript
async function uploadToIPFS(encryptedContent, pinataApiKey) {
  const formData = new FormData();
  const blob = new Blob([encryptedContent]);
  formData.append('file', blob);
  
  const response = await fetch(
    'https://api.pinata.cloud/pinning/pinFileToIPFS',
    {
      method: 'POST',
      headers: {
        'pinata_api_key': pinataApiKey,
        'pinata_secret_api_key': pinataSecretKey
      },
      body: formData
    }
  );
  
  const { IpfsHash } = await response.json();
  
  // IpfsHash is the Content Identifier (CID)
  // Example: QmXxxx...xxxxx
  
  return IpfsHash;
}
```

### Step 4: Smart Contract Policy Creation

```solidity
// Create AccessPolicy on-chain
function createPolicy(
  bytes32 policyId,
  address recipient,
  uint256 expiry,
  uint32 maxAttempts
) external {
  // Validate inputs
  require(recipient != address(0), "Invalid recipient");
  require(expiry > block.timestamp, "Expiry in the past");
  require(maxAttempts > 0, "Max attempts must be > 0");
  
  // Create policy
  AccessPolicy memory policy = AccessPolicy({
    sender: msg.sender,
    recipient: recipient,
    expiry: expiry,
    maxAttempts: maxAttempts,
    attempts: 0,
    valid: true
  });
  
  // Store in mapping
  policies[policyId] = policy;
  
  // Emit event
  emit PolicyCreated(
    policyId,
    msg.sender,
    recipient,
    expiry,
    maxAttempts
  );
}
```

### Step 5: Link Construction

```javascript
function constructLink(policyId, encryptionKey, ipfsHash) {
  // Encode encryption key (hex format)
  const keyHex = Array.from(encryptionKey)
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
  
  // Base URL
  const baseUrl = 'https://shield.app/share';
  
  // Construct full link
  const link = `${baseUrl}/${policyId}/${keyHex}`;
  
  // Optional: Add metadata
  const linkWithMetadata = `${link}?ipfs=${ipfsHash}`;
  
  return linkWithMetadata;
}
```

## Link Validation

### Recipient Receives Link

```javascript
function parseLink(link) {
  // Example: https://shield.app/share/0x7a4b.../aGVs...
  
  const urlParams = new URL(link);
  const pathParts = urlParams.pathname.split('/');
  
  return {
    policyId: pathParts[3],      // 0x7a4b...
    encryptionKey: pathParts[4],  // aGVs...
    ipfsHash: urlParams.searchParams.get('ipfs')
  };
}
```

### Verification Process

```javascript
async function verifyAndAccessContent(link, recipientAddress) {
  // 1. Parse link
  const { policyId, encryptionKey, ipfsHash } = parseLink(link);
  
  // 2. Query smart contract
  const policy = await contract.policies(policyId);
  
  // 3. Verify recipient is authorized
  if (policy.recipient !== recipientAddress) {
    throw new Error('Recipient not authorized');
  }
  
  // 4. Check expiry
  if (policy.expiry < Date.now() / 1000) {
    throw new Error('Link expired');
  }
  
  // 5. Check attempt limit
  if (policy.attempts >= policy.maxAttempts) {
    throw new Error('Max attempts exceeded');
  }
  
  // 6. Fetch from IPFS
  const encryptedContent = await fetchFromIPFS(ipfsHash);
  
  // 7. Decrypt content
  const decryptedContent = await decryptContent(
    encryptedContent,
    encryptionKey
  );
  
  // 8. Log attempt
  await contract.logAttempt(policyId, true);
  
  return decryptedContent;
}
```

## Security Considerations

### Key Protection

1. **Transport Security**: Always use HTTPS
2. **Link Sharing**: Treat links as sensitive as passwords
3. **Browser History**: Links appear in browser history (user responsibility)
4. **Link Expiry**: Set reasonable expiration times
5. **Attempt Limiting**: Prevent brute force attempts

### Attack Vectors

#### 1. Brute Force Decryption

**Risk**: Attacker intercepts encrypted content and tries keys
**Mitigation**: 
- AES-256 provides 2^256 possible keys
- Computationally infeasible to brute force
- Add attempt limiting on contract

#### 2. Link Interception

**Risk**: Attacker intercepts link in transit
**Mitigation**:
- HTTPS encryption
- Only share over secure channels
- Time-limited links

#### 3. IPFS Enumeration

**Risk**: Attacker tries to guess IPFS hashes
**Mitigation**:
- IPFS hashes are cryptographically secure
- Content-addressed (impossible to enumerate)
- Only accessible with valid policy

## Link Lifecycle

```
┌─────────────┐
│Link Created │
│  by Sender  │
└──────┬──────┘
       │
       ▼
┌──────────────┐
│ Link Active  │
│ Shared with  │
│ Recipient    │
└──────┬───────┘
       │
       ├─ If Recipient Accesses
       │  ↓
       │ ┌──────────────┐
       │ │ Decryption   │
       │ │ Successful   │
       │ │ Access       │
       │ │ Logged       │
       │ └──────┬───────┘
       │        │
       │        ├─ If Attempts < maxAttempts
       │        │  ↓
       │        │ ┌──────────────┐
       │        │ │ Can Access   │
       │        │ │ Again        │
       │        │ └──────────────┘
       │        │
       │        └─ If Attempts ≥ maxAttempts
       │           ↓
       │        ┌──────────────┐
       │        │ Access       │
       │        │ Blocked      │
       │        └──────────────┘
       │
       └─ If Expiry Time Passes
          ↓
       ┌──────────────┐
       │ Link Expired │
       │ Access       │
       │ Denied       │
       └──────────────┘
```

## Best Practices

### For Link Creators (Senders)

1. **Set Appropriate Expiry**: Balance between recipient access time and security
2. **Limit Attempts**: Prevent repeated decryption attempts
3. **Verify Recipient Address**: Ensure correct recipient authorization
4. **Use HTTPS**: Always share links over secure channels
5. **Document Expiry**: Inform recipient when access expires
6. **Monitor Access**: Review contract events for access attempts

### For Link Recipients

1. **Verify Link Source**: Confirm link comes from trusted sender
2. **Use HTTPS**: Access link over secure connection
3. **Save Content**: Download/save before expiry if needed
4. **Protect the Link**: Don't forward to untrusted parties
5. **Check Expiry**: Be aware of access window
6. **Verify Content**: Confirm received content is as expected

### For Developers

1. **Validate Inputs**: Check all link components
2. **Error Handling**: Provide clear error messages
3. **Rate Limiting**: Prevent rapid access attempts
4. **Logging**: Log all access attempts
5. **Timeout Handling**: Handle long-running decryption
6. **User Feedback**: Clear status during decryption process

## Advanced Features

### Conditional Access

Links could be extended to require:
- Multi-signature authorization
- Time-based access windows
- Geographic restrictions
- Device-specific decryption

### Link Delegation

```solidity
// Future: Allow recipient to delegate access
function delegateAccess(
  bytes32 policyId,
  address newRecipient
) external {
  require(msg.sender == policies[policyId].recipient);
  policies[policyId].recipient = newRecipient;
}
```

## Conclusion

Shield's link generation system balances convenience with security. By keeping encryption keys out of the blockchain and using time-limited, attempt-limited access policies, it ensures secure data sharing without trusting centralized servers.
