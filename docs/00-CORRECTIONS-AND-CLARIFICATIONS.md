# Important Corrections and Clarifications

## CRITICAL UPDATE: Shield Link Format

This document clarifies critical corrections made to the documentation after discovering inaccuracies in the initial guides.

### ⚠️ IMPORTANT: The Actual Shield Link Format

Shield link URLs have been corrected to reflect the actual implementation:

**CORRECT FORMAT:**
```
https://app.shieldhq.xyz/r/[POLICY_ID]#[JWK_ENCODED_KEY]
```

**INCORRECT FORMAT (previously documented):**
```
https://shield.app/share/[POLICY_ID]/[ENCRYPTED_KEY]/[OPTIONAL_METADATA]  ❌
```

### Key Corrections

1. **Domain**: `app.shieldhq.xyz` (NOT `shield.app`)
2. **Path**: `/r/[POLICY_ID]` (NOT `/share/[POLICY_ID]/...`)
3. **Encryption Key**: In URL hash fragment `#[JWK_ENCODED_KEY]` (NOT in path)
4. **Key Format**: JSON Web Key (JWK) format, URL-encoded (NOT plain encrypted key)
5. **What is POLICY_ID**: Blockchain smart contract policy identifier (NOT content ID or hash)

### Why These Corrections Matter

- **Domain**: Only `app.shieldhq.xyz` is the official Shield application domain
- **Path Structure**: `/r/` endpoint for "retrieve"/"receive" access
- **Hash Fragment Security**: The `#` portion is never sent to the server, keeping encryption keys secure
- **JWK Standard**: Uses industry-standard JSON Web Key format for cryptographic key encoding
- **Policy ID**: References the on-chain AccessPolicy created by the smart contract

## Files Affected by These Corrections

The following documents may contain references to the older (incorrect) link format and should be updated:

1. **01-shield-architecture.md** - Contains link structure diagrams
2. **02-secure-link-generation.md** - Contains detailed link generation process
3. **04-smart-contract-security.md** - May reference link components

## Files With Correct Information

The following documents contain the CORRECT and up-to-date information:

1. **05-shield-link-format-reference.md** ✅ - Complete technical reference with correct format
2. **README.md** ✅ - Updated overview (verified accurate)

## Understanding the Correct Format

### Link Components Explained

#### 1. Domain: `app.shieldhq.xyz`
- Official production domain for Shield application
- Requires HTTPS protocol
- Do NOT confuse with other Shield-related domains

#### 2. Path: `/r/[POLICY_ID]`
- `r` = "retrieve" or "receive" endpoint
- `POLICY_ID` = 32-byte identifier for the access policy stored on blockchain
- Example: `0xa5363e6ef2c6703307a827621fe6c6577d0d4407ef7714787a50f45b69736bbc`

#### 3. Hash Fragment: `#[JWK_ENCODED_KEY]`
- Contains JSON Web Key (JWK) format encryption key
- URL-encoded JSON object
- **CRITICAL**: Never sent to server (stays in browser)
- Necessary for client-side content decryption

### JWK Format

When decoded, the hash contains:
```json
{
  "alg": "A256GCM",          // Algorithm: AES-256 in GCM mode
  "ext": true,              // Key is extractable
  "k": "base64url...",       // Encryption key material
  "key_ops": ["encrypt", "decrypt"],  // Allowed operations
  "kty": "oct"              // Key type: octet (symmetric)
}
```

## Real Example

**Full Shield Link:**
```
https://app.shieldhq.xyz/r/0xa5363e6ef2c6703307a827621fe6c6577d0d4407ef7714787a50f45b69736bbc#%7B%22alg%22%3A%22A256GCM%22%2C%22ext%22%3Atrue%2C%22k%22%3A%22vJ38vtHNx2-jrEtNOMqK9X-03JtItJXoPylPYEq6b68%22%2C%22key_ops%22%3A%5B%22encrypt%22%2C%22decrypt%22%5D%2C%22kty%22%3A%22oct%22%7D
```

**Breaking this down:**
- **Domain**: `app.shieldhq.xyz`
- **Endpoint**: `/r/`
- **Policy ID**: `0xa5363e6ef2c6703307a827621fe6c6577d0d4407ef7714787a50f45b69736bbc`
- **Hash**: Contains URL-encoded JWK (encryption key)

## Why the Change Was Necessary

The initial documentation was based on the expected Shield link structure. However, after reviewing the actual Shield implementation, the real link format uses:

1. URL hash fragments for maximum security (keys never transmitted to server)
2. JWK format for cryptographic key standards compliance
3. Specific domain and endpoint routing
4. Policy IDs that reference blockchain access control

These corrections ensure documentation accurately reflects how Shield actually works in production.

## Migration Guide

If you've created documentation or integrations using the old (incorrect) link format:

1. **Update domain references**: Change all `shield.app` to `app.shieldhq.xyz`
2. **Update path structure**: Change `/share/[id]/[key]` to `/r/[policyId]#[jwk]`
3. **Review key handling**: Ensure encryption keys are handled as JWK in URL hash, not path parameters
4. **Test with real links**: Validate against actual Shield links like the example above

## Security Implications

The correct format provides superior security:

- **Server never sees keys**: Hash fragments don't get transmitted to servers
- **Standard compliance**: Uses industry-standard JWK format
- **Policy verification**: Server can verify access through on-chain policy ID
- **Client-side decryption**: Keys remain in browser, never sent over network

## References

For complete, up-to-date information on Shield link format, see:
- **05-shield-link-format-reference.md** - Comprehensive technical reference
- **README.md** - Overview and getting started guide

## Questions or Issues?

If you find additional discrepancies between documentation and actual Shield behavior, please:

1. Document the discrepancy
2. Reference this corrections file
3. Suggest updates to ensure accuracy

---

**Last Updated**: December 29, 2025
**Status**: Active Correction
**Priority**: HIGH - Affects all documentation
