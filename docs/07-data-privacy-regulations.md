# Data Privacy Regulations & Shield Compliance

## Overview

This guide explains how Shield aligns with major data privacy regulations globally, including GDPR, CCPA, and emerging privacy frameworks.

## GDPR Compliance (EU)

### Article 32: Security of Processing

Shield implements encryption required by GDPR:
- AES-256 encryption for data at rest
- TLS 1.3+ for data in transit
- Client-side encryption prevents Shield access to plaintext

### Right to Erasure (Article 17)

Shield supports the "right to be forgotten":
- Senders can revoke policy access
- Smart contract stores no plaintext content
- IPFS content can be de-pinned
- Blockchain records remain (immutable ledger)

### Data Portability (Article 20)

- Users own their encryption keys
- No vendor lock-in
- Export functionality available

## CCPA Compliance (California, USA)

### Right to Know

Shield users can:
- View all shared policies
- Track access attempts
- Audit on-chain records

### Right to Delete

- Sender can invalidate policies
- Encryption keys remain under user control
- No personal data stored by Shield

### Right to Opt-Out

- Users control information sharing
- No third-party data sales
- No cross-site tracking

## Additional Privacy Frameworks

### PIPEDA (Canada)
- Lawful basis established through consent
- Minimal personal data collection
- Secure encryption practices

### LGPD (Brazil)
- Explicit consent for processing
- Data subject rights protected
- Security measures implemented

## Privacy by Design Principles

### 1. Data Minimization

Shield collects minimal data:
- Only necessary information stored on-chain
- No metadata about content
- No behavioral tracking

### 2. Purpose Limitation

Data used only for intended purposes:
- Access verification
- Attempt logging
- Policy enforcement

### 3. Storage Limitation

Data retention policies:
- Access logs retained for 90 days
- Policies retained until expiration + 30 days
- Immutable blockchain records kept

### 4. Integrity & Confidentiality

Cryptographic guarantees:
- AES-256-GCM for authentication
- HMAC for integrity verification
- No key escrow or backdoors

## Data Processing

### What Shield Processes

**On-Chain:**
- POLICY_ID (encrypted identifier)
- Recipient address
- Expiry timestamp
- Attempt counter
- IPFS content hash

**Off-Chain (IPFS):**
- Encrypted content blob
- No metadata
- No indexing

### What Shield Does NOT Process

- Plaintext content
- Sender identification
- Recipient name or personal data
- Content type or size hints
- Access patterns
- IP addresses (not stored)

## Data Subject Rights Implementation

### Right to Access

```solidity
function getPolicies(address user)
  external view returns (bytes32[] memory) {
    // Returns policies created by user
}
```

### Right to Rectification

- Users can create new policies
- Old policies can be invalidated
- No direct modification (immutable blockchain)

### Right to Erasure

```solidity
function revokePolicy(bytes32 policyId)
  external {
    require(policies[policyId].sender == msg.sender);
    policies[policyId].valid = false;  // Revoke
}
```

### Right to Restrict Processing

- Users can always revoke access
- IPFS content can be de-pinned
- Blockchain record immutable (regulatory requirement)

### Right to Data Portability

- Export all personal policies
- Access raw blockchain data
- Use IPFS directly without Shield

## DPA & Vendor Management

### Shield as Data Processor

When used in enterprise context:
- Shield signs Data Processing Agreements
- Ensures processor compliance
- Maintains audit logs
- Supports regulatory requests

### Subprocessors

- IPFS node operators (decentralized)
- Blockchain validators (decentralized)
- No centralized third parties

## Consent Management

### Explicit Consent

Users explicitly consent by:
1. Creating account
2. Configuring policy
3. Sharing link voluntarily

### Withdrawal of Consent

- Users can revoke access any time
- No penalty for withdrawal
- Immediate effect via smart contract

## Retention & Deletion

### Data Retention Policy

| Data Type | Retention Period | Reason |
|-----------|-----------------|--------|
| Access Logs | 90 days | Audit trail |
| Expired Policies | 30 days | Compliance |
| Blockchain Records | Permanent | Immutable |
| User Encryption Keys | User-Managed | N/A |

### Deletion Procedures

1. **Sender-Initiated**
   - Revoke policy
   - De-pin IPFS content
   - Record immutable on blockchain

2. **Time-Based**
   - Automatic after retention period
   - Depends on policy expiry
   - Blockchain records preserved

## International Data Transfers

### Decentralized Nature

Shield operates globally:
- Content encrypted before any transmission
- IPFS nodes distributed worldwide
- Blockchain validators decentralized
- No central jurisdiction

### Standard Contractual Clauses

For enterprise Shield use:
- SCCs available for data transfers
- BCRs accepted where applicable
- Adequacy decisions respected

## Privacy Impact Assessment

### DPIA Requirements

Shield supports PIA by providing:
- Minimal personal data processing
- Strong encryption safeguards
- Decentralized architecture
- User consent mechanisms
- Data subject rights support

### High-Risk Processing

Shield is low-risk because:
- No systematic monitoring
- No automated decision-making
- No large-scale processing
- No vulnerable population targeting
- User control throughout

## Compliance Documentation

### Available Upon Request

- Data Processing Agreement template
- Privacy Impact Assessment summary
- Technical security documentation
- Audit reports
- Incident response procedures

## Changes & Updates

This document updated to reflect:
- Latest GDPR guidance
- CCPA/CPRA requirements
- Emerging privacy laws
- Technical improvements

Users notified of changes via:
- Email notification
- In-app announcements
- Blog updates
- Privacy policy updates

## Regulatory Contacts

For privacy inquiries:
- privacy@shieldhq.xyz
- Data Protection Officer available
- GDPR Art. 27 representative in EU
- Response time: 30 days

## Conclusion

Shield's architecture is privacy-by-design, supporting global privacy regulations through encryption, decentralization, and user control.
