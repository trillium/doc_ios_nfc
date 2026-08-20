# iOS 26 NFC Updates

New features and changes in iOS 26 and 26.x releases.

## Major Additions in iOS 26

### 1. Government ID Support (iOS 26.4+)

The NFC & SE Platform now supports digital government identity verification through NFC scanning.

**Use Cases:**
- Digital driver license verification
- Passport authentication
- National ID scanning
- Age verification

**API Changes:**
None required at framework level — government ID support is handled through existing CoreNFC APIs with expanded tag type support and secure processing.

**Entitlements:**
May require additional entitlements for specific government ID formats. Check Apple Developer portal for regional availability.

### 2. Apple Pay Tap to Share

Merchants can now tap their iPhone to customer devices to initiate checkout and data transfer via NFC.

**Features:**
- Seamless NFC tap for payment acceptance
- Dynamic content injection in Apple Pay buttons
- Bi-directional NFC communication for transaction data
- Integrated with Apple Pay infrastructure

**For Developers:**
If building merchant/payment apps, use the NFC & SE Platform entitlements. Apple Pay Tap to Share uses CoreNFC under the hood but requires special entitlements and compliance.

**Code Pattern:**
```swift
// Apple Pay Tap to Share uses Apple Pay framework, not direct CoreNFC
// Requires: com.apple.developer.nfc.hce entitlement
// Only available in specific regions and on specific devices
```

### 3. Enhanced Background Tag Reading

Improved background NDEF tag detection with better routing and notification delivery.

**Improvements:**
- More reliable tag detection when screen is on
- Better app routing for detected tags with Universal Links
- Improved notification delivery timing
- Support for additional tag types in background mode

**Requirements:**
- Target iOS 14+ for background reading
- Add `nfc` to `UIBackgroundModes` in Info.plist
- Include Universal Link URL in NDEF message
- Request appropriate entitlements

**Example:**
```swift
// Background reading now works better for these patterns:
// 1. Tag with Universal Link → app launches to correct view
// 2. App in background + tag detected → notification posted
// 3. No foreground requirement for NDEF-only tags
```

### 4. Expanded NFC & SE Platform Regions

Geographic expansion for NFC & SE Platform capabilities:
- **New regions:** Check Apple Developer portal for latest supported countries
- **Existing: EEA region** still supported for HCE and payment capabilities

## Changes Summary

| Feature | iOS 25 | iOS 26 | Note |
|---------|--------|--------|------|
| Government ID | No | iOS 26.4+ | Regional availability varies |
| Background NDEF | Limited | Enhanced | Better routing and reliability |
| Apple Pay Tap | No | Yes | Merchant-facing, requires special entitlements |
| HCE Payments | iOS 17.4+ EEA | iOS 17.4+ EEA | No change; still EEA-only |
| FeliCa Support | Yes | Yes | No change |
| Type 2-4 Tags | Yes | Yes | No change |

## No Breaking Changes

**Important:** All code written for iOS 13-25 remains compatible with iOS 26. No API changes or deprecations in CoreNFC itself.

Existing implementations will:
- ✓ Continue to work without modification
- ✓ Automatically benefit from improved background tag detection
- ✓ Optionally opt-in to new government ID features

## Migration Guide

### If You're On iOS 11-13
```swift
// Your code works fine on iOS 26
// No changes needed
// Optionally add: @available(iOS 26.4, *) for new features
```

### If You're On iOS 14+ with Background Reading
```swift
// iOS 26 improves background reading reliability
// Existing background reading code works unchanged
// Consider re-testing on iOS 26 device for improved behavior
```

### If You're Building Payment Apps
```swift
// iOS 26 doesn't change HCE or Apple Pay integration
// Still requires: iOS 17.4+ in EEA region
// Apple Pay Tap to Share requires special merchant entitlements
// Contact Apple for access to new Tap to Share APIs
```

## Action Items for Developers

### For Existing Apps
- [ ] Test on iOS 26 beta/release to verify continued operation
- [ ] Consider re-testing background reading (may see improved reliability)
- [ ] Update minimum deployment target to iOS 26 if dropping iOS 25 support

### For New Apps Targeting iOS 26
- [ ] Follow standard setup in [Getting Started](./GETTING_STARTED.md)
- [ ] All iOS 11-25 code patterns work unchanged
- [ ] Optional: Implement government ID scanning if in supported region
- [ ] Enjoy improved background tag detection automatically

### For Apps Requiring Government ID
- [ ] Contact Apple Developer support for government ID entitlements
- [ ] Verify regional availability for your target markets
- [ ] Review compliance requirements for your region
- [ ] Test with actual government IDs in beta

## Compatibility Matrix

| iOS Version | NDEF Read | NDEF Write | Native Protocols | Background | Government ID |
|-------------|-----------|------------|------------------|------------|---------------|
| 11-12       | ✓         | ✗         | ✗               | ✗         | ✗             |
| 13-16       | ✓         | ✓         | ✓               | ✗         | ✗             |
| 17-25       | ✓         | ✓         | ✓               | ✓         | ✗             |
| 26-26.3     | ✓         | ✓         | ✓               | ✓ (enhanced) | ✗             |
| 26.4+       | ✓         | ✓         | ✓               | ✓ (enhanced) | ✓ (regional)  |

## Recommended Testing Checklist

When updating to iOS 26:

- [ ] NDEF reading still works (basic scenario)
- [ ] NDEF writing works (if implemented)
- [ ] Error handling works as expected
- [ ] Background reading (if enabled) works with improved reliability
- [ ] App permissions/entitlements appear in System Settings
- [ ] Provisioning profile still valid (Apple often requires updates)
- [ ] No console warnings or errors

## Resources

- [iOS 26 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-26-release-notes)
- [Core NFC Framework](https://developer.apple.com/documentation/corenfc)
- [NFC & SE Platform](https://developer.apple.com/support/nfc-se-platform/)

---

**Last Updated:** August 2026  
**Coverage:** iOS 26.0 through 26.6
