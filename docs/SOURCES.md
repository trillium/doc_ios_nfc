# Sources & References

Official documentation and specifications used to compile this repository.

## Apple Official Documentation

### CoreNFC Framework
- **[Core NFC Documentation](https://developer.apple.com/documentation/corenfc)** — Main framework reference with all classes, protocols, and methods
- **[Building an NFC Tag-Reader App](https://developer.apple.com/documentation/corenfc/building-an-nfc-tag-reader-app)** — Step-by-step tutorial for NDEF tag reading
- **[Adding Support for Background Tag Reading](https://developer.apple.com/documentation/corenfc/adding-support-for-background-tag-reading)** — Background mode implementation guide
- **[NFCNDEFReaderSession](https://developer.apple.com/documentation/corenfc/nfcndefreadersession)** — NDEF reader session API reference
- **[NFCTagReaderSession](https://developer.apple.com/documentation/corenfc/nfctagreadersession)** — Native tag protocol reader reference
- **[NFCNDEFReaderSessionDelegate](https://developer.apple.com/documentation/corenfc/nfcndefreadersessiondelegate)** — Delegate protocol for NDEF sessions
- **[NFCTagReaderSessionDelegate](https://developer.apple.com/documentation/corenfc/nfctagreadersessiondelegate-2joku)** — Delegate protocol for tag sessions

### Entitlements & Configuration
- **[NFC Reader Session Formats Entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.developer.nfc.readersession.formats)** — Entitlements configuration
- **[ISO7816 Application Identifiers](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.developer.nfc.readersession.iso7816.select-identifiers)** — ISO7816 entitlements
- **[NFC & SE Platform](https://developer.apple.com/support/nfc-se-platform/)** — Secure payment and identity features

### iOS Release Notes
- **[iOS & iPadOS 26 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-26-release-notes)** — Latest iOS 26 features and changes
- **[iOS & iPadOS 26.4 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-26_4-release-notes)** — Government ID support in iOS 26.4
- **[iOS & iPadOS 26.5 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-26_5-release-notes)** — Subsequent OS updates
- **[iOS & iPadOS 26.6 Release Notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-26_6-release-notes)** — Latest patch releases

### WWDC Sessions (Video)
- **[WWDC 2017 Session 718: Introducing Core NFC](https://developer.apple.com/videos/play/wwdc2017/718/)** — Original CoreNFC announcement and overview
- **[WWDC 2019 Session 715: Core NFC Enhancements](https://developer.apple.com/videos/play/wwdc2019/715/)** — NDEF writing and native tag protocol support
- **[WWDC 2020 Session 10209: What's New in Core NFC](https://developer.apple.com/videos/play/wwdc2020/10209/)** — iOS 14 background reading and improvements
- **[Tech Talk 702: What's New in Core NFC](https://developer.apple.com/videos/play/tech-talks/702/)** — Recent CoreNFC updates and best practices

### Design Guidelines
- **[NFC Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/nfc)** — User experience best practices for NFC apps

## NFC Forum Specifications

### Technical Specifications
- **[NDEF Technical Specification](https://nfc-forum.org/build/specifications/data-exchange-format-ndef-technical-specification/)** — NFC Data Exchange Format message structure and encoding
- **[Record Type Definition (RTD) Technical Specification](https://nfc-forum.org/build/specifications/record-type-definition-rtd-technical-specification/)** — Well-known record types (URI, Text, etc.)
- **[Type 2 Tag Specification](https://nfc-forum.org/build/specifications/type-2-tag-specification/)** — ISO14443A Type 2 standard
- **[Type 3 Tag Specification](https://nfc-forum.org/build/specifications/type-3-tag-specification/)** — FeliCa Type 3 standard
- **[Type 4 Tag Specification](https://nfc-forum.org/build/specifications/type-4-tag-specification/)** — ISO14443A/B Type 4 standard
- **[Simple NDEF Exchange Protocol (SNEP) Specification](https://nfc-forum.org/build/specifications/simple-ndef-exchange-protocol-technical-specification/)** — P2P message exchange protocol
- **[Tag NDEF Exchange Protocol (TNEP) Specification](https://nfc-forum.org/build/specifications/tag-ndef-exchange-protocol-tnep-technical-specification-10/)** — Advanced tag communication protocol

### Record Type Definitions
- **[Multiple URI Record Type Definition](https://nfc-forum.org/build/specifications/multiple-uri-record-type-definition-candidate-technical-specification/)** — URI encoding and prefixes
- **[Signature Record Type Definition](https://nfc-forum.org/build/specifications/signature-record-type-definition-technical-specification/)** — Digital signature for tag authenticity
- **[Device Information Record Type Definition](https://nfc-forum.org/build/specifications/device-information-rtd-technical-specification/)** — Device capability metadata

### Application Documents
- **[Cross-Platform NFC Tag UX Application Document](https://nfc-forum.org/uploads/specifications/NFCForum-AD-CPUX-1.1.pdf)** — User experience guidelines across platforms
- **[NFC Forum Technical Committee Glossary](https://nfc-forum.org/uploads/Certification-Files/NFCForum-TC_Glossary_1.1.00.pdf)** — Terminology and definitions

## Apple Developer Forums

### Discussion Threads
- **[Core NFC Forum Tag](https://developer.apple.com/forums/tags/core-nfc)** — Community discussions on CoreNFC implementation
- **[Type 5 (ISO15693) Compatibility Issues](https://developer.apple.com/forums/thread/811220/)** — Known reliability issues on iPhone 12-15
- **[Missing Entitlements Troubleshooting](https://developer.apple.com/forums/thread/117482/)** — Common entitlements configuration problems
- **[Background Tag Reading](https://developer.apple.com/forums/thread/756426/)** — Background mode implementation questions
- **[MIFARE Plus and iOS Support](https://developer.apple.com/forums/thread/121524/)** — MIFARE tag type compatibility

## Key Standards & Protocols

### ISO Standards
- **ISO 14443 (Part A & B)** — Contactless smart card standards for Types 2, 3, 4
- **ISO 15693** — Vicinity Integrated Circuit Card (VICC) for Type 5
- **ISO/IEC 6955-4 (FeliCa)** — Japanese smart card standard for Type 3
- **ISO 7816** — Electrical and logical interface for smart cards
- **RFC 3066** — Language tag identification for text records

### NFC Forum Standards
- **NFC Forum Type 1-4 Specifications** — Tag type definitions and capabilities
- **NDEF Message Format** — Binary message encapsulation format
- **Record Type Definition** — Application-level record types

## Internet Standards

- **[RFC 2141: URN Syntax](https://tools.ietf.org/html/rfc2141)** — Universal Resource Identifier format
- **[RFC 3986: URI Generic Syntax](https://tools.ietf.org/html/rfc3986)** — URI scheme specifications
- **[RFC 5234: ABNF Syntax](https://tools.ietf.org/html/rfc5234)** — Augmented Backus-Naur Form for protocol specifications
- **[RFC 3629: UTF-8 Encoding](https://tools.ietf.org/html/rfc3629)** — Character encoding standard

## Implementation References

### Apple Sample Code
- **[Apple Sample Code Library](https://developer.apple.com/sample-code/)** — Various CoreNFC sample implementations
- **[WWDC Sample Code](https://developer.apple.com/sample-code/wwdc/2023/)** — Session-specific sample projects

### Third-Party References
- **[vCard Format (RFC 6350)](https://tools.ietf.org/html/rfc6350)** — Contact card format specification
- **[iCalendar Format (RFC 5545)](https://tools.ietf.org/html/rfc5545)** — Calendar event format specification

## Security & Compliance Standards

- **PCI DSS (Payment Card Industry Data Security Standard)** — Payment processing security requirements
- **EMVCo (Europay, Mastercard, Visa)** — Contactless payment specifications
- **GDPR (General Data Protection Regulation)** — EU privacy regulations for NFC data handling
- **HIPAA** — Healthcare data protection (relevant for healthcare NFC applications)

## Device & Platform Information

### Hardware Support Matrix
- **Apple Technical Specifications** — iPhone 7+ NFC chip specifications
- **A10 Fusion to A18 Processor Specs** — NFC chipset capabilities by generation
- **IEEE 802.11 Standards** — NFC RF specifications and interference considerations

### Platform Comparison
- **Android NFC Documentation** — For iOS vs Android capability comparison
- **Windows NFC Implementation** — Cross-platform compatibility notes

## How This Repository Uses These Sources

### CoreNFC Framework Section
- Apple's CoreNFC documentation and WWDC sessions
- iOS release notes for version timeline
- Device technical specifications for hardware requirements

### NFC Tag Types Section
- NFC Forum Type specifications (Types 1-4)
- ISO standards documents (ISO 14443, ISO 15693, FeliCa)
- Apple forum discussions on Type 5 reliability issues

### NDEF Message Format Section
- NFC Forum NDEF technical specification
- Record Type Definition specifications
- RFC standards for URI and text encoding

### Reading/Writing Implementation
- Apple's official CoreNFC API documentation
- WWDC session code examples
- Apple Developer Forum implementations and patterns

### Permissions & Configuration
- Apple's entitlements documentation
- Official provisioning profile requirements
- Xcode capabilities configuration guides

### Troubleshooting Section
- Apple Developer Forum threads
- Known issues documented in release notes
- User-reported problems and solutions

### iOS 26 Updates
- Official iOS 26 release notes
- Apple pay and HCE documentation
- Government ID feature announcements

### Universal Links & Testing
- Apple's Universal Links documentation
- apple-app-site-association specification
- URLSession documentation for validation

## Version Information

**Documentation Updated:** August 2026
**iOS Versions Referenced:** iOS 11 through iOS 26.6
**CoreNFC Framework:** Latest through iOS 26
**NFC Forum Specifications:** Current as of 2024-2025
**Apple Developer Resources:** Current as of August 2026

## How to Verify Sources

All sources listed above are publicly available and can be verified:

1. **Apple Documentation:** Visit developer.apple.com and search for the resource name
2. **NFC Forum Specifications:** Download from nfc-forum.org/build/specifications/
3. **WWDC Videos:** Search developer.apple.com/videos/
4. **Developer Forums:** Search developer.apple.com/forums/
5. **RFC Standards:** Available at tools.ietf.org/html/rfc[number]

## Contributing Additional Sources

If you discover additional authoritative sources that should be included:

1. Open an issue with the source URL and description
2. Explain how it improves the documentation
3. Submit a PR to add it to the appropriate section

---

**Note:** This repository prioritizes official Apple documentation, NFC Forum specifications, and peer-reviewed standards. Third-party tutorials and blog posts are not listed here but are often valuable for implementation patterns — see [Code Examples](./CODE_EXAMPLES.md) for community patterns.
