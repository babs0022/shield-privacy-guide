# Shield User Experience Guide

## Overview

Shield is designed with user experience as a core principle. This guide walks through the journey of both senders and recipients, explaining how the interface guides users through secure content sharing.

## Sender Experience

### Step 1: Initiate Sharing
When a user clicks "Share Content" on Shield:

1. **Content Selection**: Users select or upload content they want to share
2. **Recipient Specification**: Enter recipient wallet address (with optional name tag)
3. **Access Settings**: Configure sharing parameters

### Step 2: Configure Access Policy

- **Expiry Time**: Set when access expires (hours/days/weeks)
- **Max Attempts**: Limit how many times recipient can access
- **Notifications**: Enable/disable access notifications

### Step 3: Generate Secure Link

Once configured, Shield:
1. Encrypts content client-side (AES-256)
2. Uploads encrypted data to IPFS
3. Stores access policy on blockchain
4. Generates secure shareable link

```
https://app.shieldhq.xyz/r/[POLICY_ID]#[ENCRYPTED_KEY]
```

### Step 4: Share Link

- Copy to clipboard
- Share via email, messaging, or social media
- Access tracking shows when recipient views

## Recipient Experience

### Step 1: Receive Link
Recipient receives the secure link through their preferred channel.

### Step 2: Click Link
Opening the link:
1. Extracts POLICY_ID and ENCRYPTED_KEY from URL
2. Verifies policy on smart contract
3. Checks if recipient is authorized
4. Confirms policy hasn't expired

### Step 3: Verify Access
Shield performs silent verification:
- **Authentication**: Connected wallet must match policy recipient
- **Expiration**: Current time < policy expiry
- **Attempt Limits**: Remaining attempts > 0

### Step 4: View Content
If verified:
1. Fetch encrypted content from IPFS
2. Decrypt using key from URL fragment
3. Display to user
4. Log access attempt on blockchain

## Key UX Features

### Privacy Preservation
- **No Server Logs**: Content never touches servers
- **Client-Side**: All encryption/decryption happens locally
- **Minimal Data**: Only necessary info stored on-chain

### Intuitive Interface
- **Clear Step-by-Step**: Simple flow for creation and access
- **Visual Feedback**: Status indicators for each step
- **Error Messages**: Clear explanations if access denied

### Security Indicators
- **Lock Icons**: Show encryption status
- **Expiry Countdown**: Display remaining access time
- **Attempt Counter**: Show how many views remaining

### Accessibility
- **Mobile Responsive**: Works on all devices
- **Keyboard Navigation**: Full keyboard support
- **Contrast Compliance**: WCAG AA compliant

## Access Denied Scenarios

### Invalid Recipient
**Issue**: Connected wallet doesn't match policy recipient
**Solution**: Connect correct wallet or request new link

### Expired Access
**Issue**: Access period has ended
**Solution**: Request fresh link from sender

### Exceeded Attempts
**Issue**: Maximum access attempts reached
**Solution**: Sender must create new policy if more access needed

### Revoked Access
**Issue**: Sender invalidated the policy
**Solution**: Contact sender for new sharing link

## Mobile Experience

### Responsive Design
- Optimized for all screen sizes
- Touch-friendly buttons and controls
- Mobile-optimized forms

### Native Wallet Integration
- WalletConnect support
- MetaMask Mobile compatibility
- Argent, Rainbow, and other wallets

## Accessibility Considerations

### Screen Readers
- Proper ARIA labels
- Semantic HTML structure
- Clear link descriptions

### Keyboard Navigation
- Tab order follows logical flow
- Skip to main content link
- All functionality keyboard accessible

### Color Blindness
- Not relying solely on color for information
- Sufficient contrast ratios
- Pattern/icon usage for status indicators

## Performance Optimization

### Load Time
- Progressive loading
- Minimal dependencies
- Optimized assets

### Network Efficiency
- Minimal blockchain calls
- IPFS caching
- Progressive enhancement

## User Feedback & Analytics

### Non-Intrusive Tracking
- Anonymous usage metrics
- Feature usage statistics
- Error rate monitoring

### User Feedback
- Optional feedback surveys
- Contact support easily
- Report issues

## Best Practices for Users

### For Senders
1. **Choose Reasonable Expiry**: Balance security with access needs
2. **Set Appropriate Limits**: Don't make attempts too restrictive
3. **Verify Recipients**: Ensure you have correct wallet address
4. **Test Before Sharing**: Verify the link works as expected

### For Recipients
1. **Verify URL**: Check for HTTPS and correct domain
2. **Use Trusted Wallet**: Connect from your primary wallet
3. **Download If Needed**: Save content locally if permanent copy needed
4. **Respect Sender Intent**: Don't share received content externally

## Troubleshooting

### Common Issues

**Can't connect wallet**
- Ensure wallet extension is installed
- Check network is correct
- Try different wallet if one fails

**Link not working**
- Verify full URL is copied
- Check URL doesn't contain typos
- Ensure using HTTPS connection

**Access denied**
- Connect correct wallet
- Check policy hasn't expired
- Verify not exceeded attempt limit

## Future Enhancements

### Planned Features
- Password protection
- Download limits
- Screenshot prevention
- Selective content preview
- Collaborative sharing

## Conclusion

Shield's UX design prioritizes user privacy without sacrificing ease of use. Every interface element is optimized for secure, intuitive content sharing.
