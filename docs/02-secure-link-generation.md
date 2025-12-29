# Secure Link Generation & Management in Shield

## Overview

The secure link is the critical bridge between the blockchain's access control and the client's encryption key. This document explains how Shield generates, manages, and validates secure links throughout their lifecycle.

## Link Structure

### Anatomy of a Shield Link

```
https://app.shieldhq.xyz/r/[POLICY_ID]#[JWK_ENCODED_KEY]
                        └──────────┬──────────┘ └────────┬─────────┘
                                   │                      │
                        On-chain verification     Client-side decryption
                        (contract lookup)         (URL hash fragment)
```

**POLICY_ID**: 
- 32-byte identifier
- Stored on-chain in smart contract
- Enables access verification
- Immutable once created

**JWK_ENCODED_KEY**:
- JSON Web Key format encryption key
- URL-encoded in hash fragment
- Never sent to server
- Necessary for decryption

### Example Link

```
https://app.shieldhq.xyz/r/0xa5363e6ef2c6703307a827621fe6c6577d0d4407ef7714787a50f45b69736bbc#%7B%22alg%22%3A%22A256GCM%22%2C%22ext%22%3Atrue%2C%22k%22%3A%22vJ38vtHNx2-jrEtNOMqK9X-03JtItJXoPylPYEq6b68%22%2C%22key_ops%22%3A%5B%22encrypt%22%2C%22decrypt%22%5D%2C%22kty%22%3A%22oct%22%7D
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
  // - Included only in URL hash fragment
  
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
  // Create JWK from encryption key
  const jwk = {
    alg: "A256GCM",
    ext: true,
    k: base64url(encryptionKey),
    key_ops: ["encrypt", "decrypt"],
    kty: "oct"
  };
  
  // URL encode JWK
  const encodedJwk = encodeURIComponent(JSON.stringify(jwk));
  
  // Construct link with correct format
  const link = `https://app.shieldhq.xyz/r/${policyId}#${encodedJwk}`;
  
  return link;
}
```

## Link Validation

### Recipient Accesses Link

```javascript
async function verifyAndAccessContent(link, recipientAddress) {
  // 1. Parse link
  const url = new URL(link);
  const policyId = url.pathname.split('/r/')[1];
  const jwkEncoded = url.hash.substring(1);
  
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
  
  // 6. Decode JWK and decrypt
  const jwk = JSON.parse(decodeURIComponent(jwkEncoded));
  const key = await crypto.subtle.importKey(
    "jwk",
    jwk,
    { name: "AES-GCM" },
    true,
    ["decrypt"]
  );
  
  // 7. Fetch and decrypt content
  const encrypted = await fetchFromIPFS(ipfsHash);
  const decrypted = await crypto.subtle.decrypt(
    { name: "AES-GCM", iv: /* from encrypted */ },
    key,
    encrypted
  );
  
  // 8. Log access attempt
  await contract.logAttempt(policyId, true);
  
  return decrypted;
}
```

## Security Considerations

### Why URL Hash Fragment?

1. **Not sent to server**: Hash fragment stays in browser, never transmitted
2. **Server cannot access keys**: Only sees policy ID, never the encryption key
3. **Client-side decryption**: Keys remain secure in browser memory
4. **No logging of keys**: Server logs don't contain sensitive encryption material

### Attack Prevention

#### Brute Force Decryption
- AES-256 provides 2^256 possible keys
- Computationally infeasible to brute force
- Attempt limiting on smart contract prevents repeated tries

#### Link Interception
- Use HTTPS-only links
- Share over secure channels
- Time-limited access through smart contract expiry

#### IPFS Enumeration
- IPFS hashes are cryptographically secure
- Content-addressed (impossible to enumerate)
- Only accessible with valid policy AND key

## Best Practices

### For Senders
1. Verify recipient address before creating policy
2. Set appropriate expiry times (hours/days, not years)
3. Limit access attempts to prevent brute force
4. Monitor access through contract events
5. Use secure channels to share links

### For Recipients
1. Verify link source and authenticity
2. Check domain is app.shieldhq.xyz
3. Protect the link (contains encryption key)
4. Access before expiry time
5. Do not forward to untrusted parties

### For Developers
1. Validate all link components before use
2. Handle decryption errors gracefully
3. Implement retry logic with backoff
4. Log access attempts for audit trails
5. Test with real links before deployment

## Conclusion

Shield's secure link format combines on-chain access control with client-side encryption. The URL hash fragment approach ensures encryption keys never reach the server, while POLICY_ID on the blockchain manages access permissions. This design provides maximum security and privacy for shared sensitive content.
