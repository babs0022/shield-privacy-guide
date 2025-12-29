# Shield Smart Contract Security

## Overview

The Shield smart contract is the critical component managing access control, policy creation, and verification. This guide covers security considerations, attack prevention, and best practices.

## Contract Architecture

### Core State

```solidity
mapping(bytes32 => AccessPolicy) public policies;
mapping(address => uint256) public userPolicies;
```

### Key Security Features

1. **Access Control**: Verify msg.sender authorization
2. **Input Validation**: Validate all function parameters
3. **State Consistency**: Maintain policy state integrity
4. **Event Logging**: Emit events for all state changes
5. **Reentrancy Protection**: Protect against reentrancy attacks
6. **Overflow/Underflow**: Use SafeMath for arithmetic

## Security Vulnerabilities & Mitigations

### 1. Unauthorized Policy Modification

**Vulnerability**: Attacker modifies AccessPolicy struct

**Mitigation**:
```solidity
function modifyPolicy(bytes32 policyId, AccessPolicy calldata newPolicy) external {
  require(policies[policyId].sender == msg.sender, "Unauthorized");
  require(newPolicy.recipient != address(0), "Invalid recipient");
  policies[policyId] = newPolicy;
}
```

### 2. Policy Deletion Attack

**Vulnerability**: Sender deletes policy to revoke access

**Mitigation**: Use state flag instead of deletion
```solidity
policies[policyId].valid = false;  // Invalidate instead of delete
```

### 3. Timestamp Dependency

**Vulnerability**: Miners manipulate block.timestamp

**Mitigation**: Use block.number or require time buffer
```solidity
require(expiry > block.timestamp + 1 minutes, "Expiry too soon");
```

### 4. Integer Overflow/Underflow

**Vulnerability**: attempts counter overflows

**Mitigation**: Use SafeMath or Solidity ^0.8.0 checks
```solidity
attempts = SafeMath.add(attempts, 1);
```

### 5. Gas Limit DoS

**Vulnerability**: Large loop in policy enumeration

**Mitigation**: Limit iterations, use pagination
```solidity
function getPolicies(uint offset, uint limit) external view returns (bytes32[] memory) {
  require(limit <= 100, "Limit too large");
  // ... implementation
}
```

### 6. Frontrunning

**Vulnerability**: Attacker sees transaction and creates competing policy

**Mitigation**: Use commit-reveal scheme or private mempools
```solidity
// Commit phase
function commitPolicy(bytes32 commitment) external {
  commits[msg.sender] = commitment;
}

// Reveal phase (after block boundary)
function revealPolicy(bytes32 policyId, string memory salt) external {
  require(keccak256(abi.encodePacked(policyId, salt)) == commits[msg.sender]);
  // Create policy
}
```

## Contract Code Review Checklist

- [ ] All external functions have proper access control
- [ ] Input parameters are validated
- [ ] No unsafe delegatecalls
- [ ] No unchecked external calls
- [ ] Proper event logging
- [ ] No hardcoded addresses (except constants)
- [ ] Error messages are informative
- [ ] State changes are properly ordered (checks-effects-interactions)
- [ ] No obvious gas optimizations missed
- [ ] Compatible with standard interfaces

## Audit Recommendations

### Pre-Deployment

1. **Static Analysis**: Use Slither/MythX
2. **Manual Review**: Line-by-line code review
3. **Testing**: >90% code coverage
4. **Formal Verification**: Prove critical invariants
5. **Testnet Deployment**: Test on public testnet

### Post-Deployment

1. **Monitoring**: Watch for suspicious events
2. **Upgrade Strategy**: Plan for bug fixes
3. **Emergency Pause**: Ability to pause in emergency
4. **Rate Limiting**: Monitor unusual activity

## Testing Strategy

### Unit Tests

```javascript
describe("createPolicy", () => {
  it("should create policy with valid parameters", async () => {
    const policyId = ethers.utils.id("test");
    await expect(shield.createPolicy(
      policyId,
      recipient.address,
      expiryTime,
      5
    )).to.emit(shield, "PolicyCreated");
  });
  
  it("should reject invalid recipient", async () => {
    const policyId = ethers.utils.id("test");
    await expect(shield.createPolicy(
      policyId,
      ethers.constants.AddressZero,
      expiryTime,
      5
    )).to.be.revertedWith("Invalid recipient");
  });
});
```

### Integration Tests

```javascript
describe("Complete flow", () => {
  it("should allow sender to share and recipient to access", async () => {
    // Create policy
    const policyId = ethers.utils.id("test");
    await shield.createPolicy(
      policyId,
      recipient.address,
      expiryTime,
      5
    );
    
    // Verify access
    const hasAccess = await shield.verifyAccess(policyId, recipient.address);
    expect(hasAccess).to.be.true;
    
    // Log attempt
    await shield.connect(recipient).logAttempt(policyId, true);
    
    // Check attempt count
    const policy = await shield.policies(policyId);
    expect(policy.attempts).to.equal(1);
  });
});
```

## Gas Optimization

### Techniques

1. **Packed Storage**: Combine multiple state variables
```solidity
struct AccessPolicy {
  address sender;
  address recipient;
  uint256 expiry;
  uint32 maxAttempts;  // Packed with uint32 attempts
  uint32 attempts;
  bool valid;
}
```

2. **Caching**: Store frequent lookups
```solidity
AccessPolicy memory policy = policies[policyId];  // Cache in memory
require(policy.recipient == msg.sender);
```

3. **Batch Operations**: Process multiple items in one transaction
```solidity
function createPoliciesBatch(
  bytes32[] calldata policyIds,
  address[] calldata recipients,
  uint256[] calldata expiries
) external {
  for (uint i = 0; i < policyIds.length; i++) {
    // Batch create policies
  }
}
```

## Upgrade Strategy

### Proxy Pattern

Use UUPS or Transparent Proxy for upgradability:

```solidity
import "@openzeppelin/contracts/proxy/utils/UUPSUpgradeable.sol";

contract ShieldV2 is UUPSUpgradeable {
  function _authorizeUpgrade(address newImpl) internal override onlyAdmin {}
}
```

### Storage Layout

Maintain storage layout when upgrading:
```solidity
// V1
struct AccessPolicy {
  address sender;
  address recipient;
  uint256 expiry;
}

// V2 - Add fields at the end
struct AccessPolicy {
  address sender;
  address recipient;
  uint256 expiry;
  uint32 maxAttempts;  // New field added
  uint32 attempts;     // New field added
  bool valid;          // New field added
}
```

## Monitoring & Alerting

### Events to Monitor

```solidity
event PolicyCreated(bytes32 indexed policyId, address sender, address recipient);
event VerificationAttempt(bytes32 indexed policyId, address accessor, bool success);
event PolicyInvalidated(bytes32 indexed policyId);
```

### Off-Chain Monitoring

```javascript
const filter = shield.filters.PolicyCreated();
const events = await shield.queryFilter(filter);

// Alert on unusual patterns
const policiesPerDay = events.length;
if (policiesPerDay > THRESHOLD) {
  alert("Unusual policy creation rate");
}
```

## Conclusion

SmartContract security requires comprehensive testing, careful code review, and ongoing monitoring. Shield's design emphasizes immutability, clear state management, and event transparency to ensure secure access control.
