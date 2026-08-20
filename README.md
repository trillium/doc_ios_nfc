<details>
<summary><strong>👥 FOR AGENTS: iOS NFC Reference Library</strong></summary>

## Purpose

This repository **centralizes NFC knowledge for agents building iOS features**. Instead of hunting docs or repeating setup, you get:

- ✓ Step-by-step setup guide (Xcode → working code in 15 min)
- ✓ 10 copy-paste code patterns for common scenarios
- ✓ Complete API reference with iOS version matrix
- ✓ Troubleshooting guide (symptom → solution)
- ✓ Critical gotchas (7 ways URLs silently fail, Type 5 reliability, etc.)
- ✓ Official sources cited (Apple docs, NFC Forum specs, RFCs)

**Goal:** Save humans time by letting agents solve NFC problems with agents via GitHub issues, not by interrupting humans.

## How to Use This Repo

**You're starting a new NFC feature:**
1. [GETTING_STARTED.md](./docs/GETTING_STARTED.md) — Xcode setup + first working code (15 min)
2. [CODE_EXAMPLES.md](./docs/CODE_EXAMPLES.md) — Find your pattern, copy-paste, adapt
3. [TECHNICAL.md](./docs/TECHNICAL.md) — Reference when you need API details

**You're stuck on a problem:**
1. [TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) — Search by symptom
2. Open GitHub issue if not listed — describe what you tried
3. Check existing issues — someone may have solved it

**You discovered a pattern others should know:**
1. Open a GitHub issue: `gh issue create --title "Pattern: [what]" --body "Description" --repo trillium/doc_ios_nfc`
2. Describe: Use case, how you solved it, why it matters
3. Submit PR with addition to [CODE_EXAMPLES.md](./docs/CODE_EXAMPLES.md)

**For full guidance:** See [MANIFEST.md](./MANIFEST.md)

</details>

# iOS NFC Programming Guide

Comprehensive technical documentation for implementing NFC tag reading and writing in iOS applications. Designed to help developers and agents understand CoreNFC framework capabilities, NDEF message formats, tag types, permissions, and practical implementation patterns.

**Audience:** iOS developers, agents implementing NFC features, technical architects designing tap-to-action experiences.

## Documentation Contents

### For Agents
- [MANIFEST.md](./MANIFEST.md) — **Start here** — What's in the repo and how to use it

### Reference Guides
- [Getting Started Guide](./docs/GETTING_STARTED.md) — Step-by-step Xcode setup (15 min)
- [Code Examples](./docs/CODE_EXAMPLES.md) — 10 copy-paste patterns for common scenarios
- [Complete Technical Writeup](./docs/TECHNICAL.md) — Full API specification and reference
- [Troubleshooting](./docs/TROUBLESHOOTING.md) — Problem → solution guide
- [Universal Links & Testing](./docs/UNIVERSAL_LINKS_TESTING.md) — URL routing gotchas and validation
- [iOS 26 Updates](./docs/IOS_26_CHANGES.md) — Latest features and changes
- [Sources & References](./docs/SOURCES.md) — Official sources (Apple docs, NFC Forum specs, RFCs)

## What You'll Find

### 1. **CoreNFC Framework** 
Hardware requirements (iPhone 7+), iOS version support (11-17.4), main classes and protocols, framework overview.

### 2. **NFC Tag Types**
Complete specifications for Types 1-5, ISO standards, iOS support matrix, features, and compatibility notes.

### 3. **Reading NFC Tags**
Step-by-step NDEF reading flow, delegate methods, native tag protocol support (ISO7816, ISO15693, FeliCa, MIFARE), error handling.

### 4. **Writing NFC Tags**
iOS 13+ capabilities, creating NDEF messages, custom payloads, write operations on different tag types.

### 5. **iOS Permissions & Restrictions**
Info.plist configuration, entitlements setup, provisioning profile requirements, background mode constraints, HCE limitations.

### 6. **Message Format & Encoding**
NDEF record structure, TNF values, well-known types (URI, Text, Smart Poster, Signature), byte-level payload specifications.

### 7. **Event Handling**
Session lifecycle, delegate callbacks, error codes, custom alert messages, state management patterns.

### 8. **Practical Code Examples**
- URL tap-to-action navigation
- vCard contact sharing
- FeliCa transit card reading
- Smart Poster creation
- Background tag detection
- View controller navigation flows

### 9. **iOS vs Android**
Direct comparison table highlighting capabilities, limitations, and cross-platform compatibility issues.

### 10. **Real-World Use Cases**
- QR-alternative tap-to-action
- Digital identity and transit
- Loyalty programs
- Contactless payments (iOS 17.4+ EEA only)
- Device pairing and setup

## Key Findings

### Supported Tag Types
| Type | ISO Standard | iOS Support | Status |
|------|--------------|-------------|--------|
| Type 1 | ISO14443A | Read via NDEF | ✓ |
| Type 2 | ISO14443A | Read/Write (iOS 13+) | ✓ |
| Type 3 | FeliCa | Read/Write (iOS 13+) | ✓ |
| Type 4 | ISO14443A/B | Read/Write (iOS 13+) | ✓ |
| Type 5 | ISO15693 | Native protocol only | ⚠ Unreliable |

### iOS Version Timeline
- **iOS 11:** Initial CoreNFC — NDEF reading only
- **iOS 13:** Major expansion — writing, native protocols, FeliCa
- **iOS 14:** Background tag reading support
- **iOS 17.4:** HCE payments (EEA region only)
- **iOS 26:** Government ID support, improved background reading, Tap to Share

### Critical Limitations
- **No P2P mode** — Cannot communicate device-to-device (Android has Android Beam)
- **No background HCE** — Contactless payments require foreground operation
- **Type 5 unreliability** — 90%+ failure rates on iPhone 12-15 compared to iPhone 7-11
- **Geographic restriction** — HCE payments iOS 17.4+ EEA-only
- **Foreground requirement** — Most operations require active app

## Hardware Support

| Feature | Minimum Device | Requirements |
|---------|----------------|--------------|
| NDEF Reading | iPhone 7+ | A10 Fusion chip, iOS 11+ |
| Native Protocols | iPhone 7+ | A10 Fusion chip, iOS 13+ |
| Background Reading | iPhone 7+ | iOS 14+, specific entitlements |
| HCE Payments | iPhone XS+ | iOS 17.4+, EEA region only |

## Configuration Checklist

- [ ] Add `CoreNFC.framework` to Xcode project
- [ ] Add `NFCReaderUsageDescription` to Info.plist
- [ ] Configure `.entitlements` file with NFC capabilities
- [ ] Enable NFC capability in Apple Developer portal
- [ ] Generate provisioning profile with NFC entitlements
- [ ] For tag writing: iOS 13+ target
- [ ] For background reading: Add `nfc` to UIBackgroundModes
- [ ] For ISO7816/ISO15693: Enable specific protocol entitlements

## Common Patterns

### 1. Universal Link Navigation (Simplest)
Embed URL in NDEF tag → iOS launches app → Universal Link routing → view displays.

### 2. Direct Reader State Management
Scan tag → parse custom record → update @State → view changes.

### 3. Background + Notification
Tag detected in background → notification posted → user taps → app shows view.

## Troubleshooting Quick Reference

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| "Missing required entitlement" | Entitlements not in profile | Regenerate provisioning profile |
| Session timeout | Tag too far/wrong type | Bring closer, verify tag type |
| Type 5 tags unreliable | iPhone 12-15 hardware issue | Use Type 2/3/4 or test on iPhone 7-11 |
| Write fails silently | Tag read-only or protected | Check tag state and write capability |
| No background detection | Missing entitlements or URL | Add background mode + Universal Link URL |

## Related Documentation

- [Apple CoreNFC Framework](https://developer.apple.com/documentation/corenfc)
- [NFC Forum Specifications](https://nfc-forum.org/build/specifications/)
- [NDEF Technical Specification](https://nfc-forum.org/build/specifications/data-exchange-format-ndef-technical-specification/)
- [iOS HCI Guidelines for NFC](https://developer.apple.com/design/human-interface-guidelines/nfc)

## License

Documentation for iOS NFC development. For questions or contributions, open an issue.

## What's in Each Section

- **Specifications** — exact API signatures, error codes, configuration requirements
- **Code examples** — copy-paste ready implementations for common patterns
- **Decision trees** — when to use NDEF reading vs native protocols, when to use background modes
- **Compatibility matrices** — what works on which iOS version, device, or tag type
- **Testing guides** — how to validate your implementation end-to-end

---

**Last Updated:** August 2026  
**iOS Versions Covered:** iOS 11 through iOS 26.6  
**Hardware:** iPhone 7 and later
