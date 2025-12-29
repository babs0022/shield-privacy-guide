# Shield Link Format Reference - Actual Implementation

## Actual Shield Link Structure

Shield uses a specific URL format with JSON Web Key (JWK) encoding in the hash fragment:

```
https://app.shieldhq.xyz/r/[POLICY_ID]#[JWK_ENCODED_KEY]
```

### Real Example

```
https://app.shieldhq.xyz/r/0xa5363e6ef2c6703307a827621fe6c6577d0d4407ef7714787a50f45b69736bbc#%7B%22alg%22%3A%22A256GCM%22%2C%22ext%22%3Atrue%2C%22k%22%3A%22vJ38vtHNx2-jrEtNOMqK9X-03JtItJXoPylPYEq6b68%22%2C%22key_ops%22%3A%5B%22encrypt%22%2C%22decrypt%22%5D%2C%22kty%22%3A%22oct%22%7D
```

## URL Components

### 1. Domain
- **Correct**: `app.shieldhq.xyz` (NOT `shield.app`)
- The production domain for Shield application
- Standard HTTPS protocol required

### 2. Path
- **Format**: `/r/[POLICY_ID]`
- `r` = "receive" or "retrieve" endpoint
- `POLICY_ID` = unique identifier for the access policy created on blockchain
- Example: `0xa5363e6ef2c6703307a827621fe6c6577d0d4407ef7714787a50f45b69736bbc`
- This is the policyId returned by `createPolicy()` smart contract function

### 3. Hash Fragment (URL Fragment Identifier)
- **Format**: `#[URL_ENCODED_JWK]`
- Contains JSON Web Key (JWK) format encryption key
- Encoded as URL-encoded JSON
- **Critical**: Never sent to server (browser-only)
- Provides client-side decryption capability

## JSON Web Key (JWK) Format

When decoded, the hash fragment contains a JWK object:

```json
{
  "alg": "A256GCM",
  "ext": true,
  "k": "vJ38vtHNx2-jrEtNOMqK9X-03JtItJXoPylPYEq6b68",
  "key_ops": ["encrypt", "decrypt"],
  "kty": "oct"
}
```

### JWK Fields Explained

| Field | Value | Meaning |
|-------|-------|----------|
| `alg` | `A256GCM` | Algorithm: AES 256-bit in GCM mode |
| `ext` | `true` | Key is extractable |
| `k` | Base64 encoded key | The actual encryption key material |
| `key_ops` | `["encrypt", "decrypt"]` | Allowed operations |
| `kty` | `oct` | Key type: Symmetric (octet sequence) |

## URL Encoding

The JWK is URL-encoded in the hash:

```
Original JSON:
{"alg":"A256GCM","ext":true,...}

URL Encoded:
%7B%22alg%22%3A%22A256GCM%22%2C...
```

### Encoding Map
- `{` = `%7B`
- `}` = `%7D`
- `:` = `%3A`
- `"` = `%22`
- `,` = `%2C`
- `[` = `%5B`
- `]` = `%5D`

## Why Hash Fragment?

### Security Benefits

1. **Not sent to server**: Hash fragment is never transmitted to the server
   - Server only sees: `https://app.shieldhq.xyz/r/0xa536...`
   - Server NEVER sees the encryption key
   - Policy ID allows server to verify access permissions

2. **Browser-only decryption**: Key remains in browser memory
   - Decryption happens client-side only
   - Server has no access to keys or decrypted content

3. **No logging**: Server logs don't contain sensitive keys
   - Even if logs are compromised, keys are safe
   - Privacy preserved at infrastructure level

4. **Bookmark safety**: Sharing link with key via URL
   - Users can bookmark encrypted shares
   - Key persists with bookmark
   - Key not exposed to servers in between

## Complete Flow with Actual Link Format

### Sender Creates Share

```javascript
// 1. Create access policy on blockchain
const policyId = ethers.utils.id(Date.now().toString());
await contract.createPolicy(
  policyId,
  recipientAddress,
  expiryTime,
  maxAttempts
);
// Policy is stored on-chain

// 2. Encrypt content
const encryptionKey = generateEncryptionKey();  // Random 256-bit key
const encrypted = await encryptContent(content, encryptionKey);

// 3. Upload encrypted content to IPFS
const contentHash = await uploadToIPFS(encrypted);

// 4. Create JWK from encryption key
const jwk = {
  alg: "A256GCM",
  ext: true,
  k: base64url(encryptionKey),  // vJ38vtHNx2-jrEtNOMqK9X-03JtItJXoPylPYEq6b68
  key_ops: ["encrypt", "decrypt"],
  kty: "oct"
};

// 5. URL encode JWK
const encodedJwk = encodeURIComponent(JSON.stringify(jwk));

// 6. Construct link with POLICY_ID
const link = `https://app.shieldhq.xyz/r/${policyId}#${encodedJwk}`;
// Result: https://app.shieldhq.xyz/r/0xa536...#%7B%22alg%22...
```

### Recipient Accesses Share

```javascript
// 1. Extract components from URL
const url = new URL(window.location.href);
const policyId = url.pathname.split('/r/')[1];
const jwkEncoded = url.hash.substring(1);  // Remove #

// 2. Verify access on blockchain
const hasAccess = await contract.verifyAccess(policyId, recipientAddress);
if (!hasAccess) {
  throw new Error('Access denied');
}

// 3. Decode JWK from URL fragment
const jwk = JSON.parse(decodeURIComponent(jwkEncoded));

// 4. Import key using Web Crypto API
const key = await crypto.subtle.importKey(
  "jwk",
  jwk,
  { name: "AES-GCM" },
  true,
  ["decrypt"]
);

// 5. Fetch encrypted content from IPFS
const encrypted = await fetchContent(contentHash);

// 6. Decrypt
const decrypted = await crypto.subtle.decrypt(
  { name: "AES-GCM", iv: /* extracted from encrypted data */ },
  key,
  encrypted
);

// 7. Log access attempt on blockchain
await contract.logAttempt(policyId, true);
```

## Security Considerations

### Key Exposure Prevention

1. **Hash fragment not in HTTP requests**: The `#` part is never sent to servers
2. **Policy verification on-chain**: Server verifies policyId against smart contract
3. **Visible in browser address bar**: Users should verify they're on `app.shieldhq.xyz`
4. **Bookmark history**: Links with keys may appear in browser history
5. **Copy/paste**: Users should treat links as sensitive passwords

### Best Practices

1. **Share over secure channels**: Use HTTPS-only links, encrypted messaging
2. **Verify domain**: Always check for `app.shieldhq.xyz` (not other variations)
3. **Time-limited shares**: Backend enforces expiry through smart contract
4. **Warn users**: Educate about not sharing links publicly
5. **Incognito browsing**: Use private windows for sensitive content
6. **Verify policy**: Check blockchain that policy matches expected parameters

## Policy ID vs Content

Important distinction:

- **Policy ID**: Unique identifier for access control policy
  - Created by `createPolicy()` transaction
  - Stored on-chain in smart contract
  - Server can verify access permissions
  - Tied to specific sender, recipient, expiry, attempt limits

- **Content**: Encrypted data
  - Stored on IPFS or decentralized storage
  - Identified by content hash
  - Content is always encrypted
  - Only decryptable with key from URL fragment

## Testing the Format

### Validate Link Structure

```javascript
function validateShieldLink(url) {
  const parsed = new URL(url);
  
  // Check domain
  if (!parsed.hostname.endsWith('shieldhq.xyz')) {
    return false;  // Invalid domain
  }
  
  // Check path
  if (!parsed.pathname.startsWith('/r/')) {
    return false;  // Invalid path
  }
  
  // Check hash (JWK)
  if (!parsed.hash) {
    return false;  // Missing JWK
  }
  
  // Try to parse JWK
  try {
    const jwk = JSON.parse(decodeURIComponent(parsed.hash.substring(1)));
    if (jwk.alg !== 'A256GCM' || jwk.kty !== 'oct') {
      return false;  // Invalid key type
    }
  } catch (e) {
    return false;  // Invalid JSON
  }
  
  return true;
}

// Verify policy on blockchain
async function verifyPolicyId(policyId, recipientAddress) {
  const hasAccess = await contract.verifyAccess(policyId, recipientAddress);
  return hasAccess;
}
```

## Conclusion

Shield's use of URL hash fragments with JWK encoding provides an elegant solution for sharing encryption keys while maintaining complete client-side privacy. The policy ID stored on blockchain enables access control, while the key in the URL hash ensures only the recipient can decrypt. The server never sees the encryption key, providing maximum security and privacy for users sharing sensitive content.
