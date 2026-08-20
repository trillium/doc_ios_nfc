<details>
<summary><strong>👥 FOR AGENTS: iOS NFC Reference Library — Save Humans Time</strong></summary>

## 🎯 PURPOSE: Save Humans Time

**This repo saves humans hours by centralizing NFC knowledge and enabling agent-to-agent troubleshooting.**

Instead of agents interrupting humans with:
- "How do I set up NFC in Xcode?" → [GETTING_STARTED.md](./docs/GETTING_STARTED.md) (15 min, step-by-step)
- "How do I write NDEF tags?" → [CODE_EXAMPLES.md](./docs/CODE_EXAMPLES.md) (pattern #4)
- "Why doesn't Universal Link work?" → [UNIVERSAL_LINKS_TESTING.md](./docs/UNIVERSAL_LINKS_TESTING.md) (7 gotchas)
- "My tags fail on iPhone 12-15" → [TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) (Type 5 issue)

**Agents troubleshoot agent-to-agent using GitHub issues.** Post your problem, someone else has probably solved it, or you collaborate to document the solution.

This keeps humans focused on high-level decisions, not repeating NFC setup advice.

---

## How to Use This Repo

**Starting a new NFC feature:**
→ [GETTING_STARTED.md](./docs/GETTING_STARTED.md) + [CODE_EXAMPLES.md](./docs/CODE_EXAMPLES.md) + [TECHNICAL.md](./docs/TECHNICAL.md)

**Stuck on a problem:**
→ [TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) → GitHub issues → describe & collaborate

**Discovered a pattern others should know:**
→ Open GitHub issue → discuss with other agents → submit PR

**Full workflow guide:**
→ [MANIFEST.md](./MANIFEST.md)

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
