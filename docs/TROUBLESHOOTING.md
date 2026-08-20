# NFC Troubleshooting Guide

Quick solutions for common NFC issues.

## Session Issues

### "Session timeout - no tag detected"

| Cause | Solution |
|-------|----------|
| Tag too far away | Hold tag 4 inches from top of iPhone |
| Tag not NDEF formatted | Use NFC Type 2-4 tags; verify with Android NFC Tools app |
| Wrong tag type | Some custom tags not supported; try standard NDEF tags |
| Background NFC interfering | Toggle airplane mode; restart app |

**Diagnostic:**
```swift
print("alertMessage shown to user: \(session.alertMessage)")
print("Session queue: \(session.queue ?? "default")")
print("invalidateAfterFirstRead: true/false")
```

### "Session invalidated with error"

```swift
func readerSession(_ session: NFCReaderSession,
                  didInvalidateWithError error: Error) {
    
    if let nfcError = error as? NFCReaderError {
        let code = nfcError.code
        
        if code == .readerSessionInvalidationErrorFirstNDEFTagRead {
            print("First tag read - normal if invalidateAfterFirstRead=true")
        } else if code == .readerSessionInvalidationErrorSessionTimeout {
            print("Timeout - no tag found")
        } else if code == .readerSessionInvalidationErrorUserCanceled {
            print("User tapped 'Cancel' in alert")
        } else if code == .readerSessionInvalidationErrorSecurityViolation {
            print("SECURITY VIOLATION - check entitlements!")
            print(error.localizedDescription)
        }
    }
}
```

**For security violations:**
1. Clean build: `cmd + shift + K`
2. Delete derived data: `rm -rf ~/Library/Developer/Xcode/DerivedData`
3. Rebuild from scratch
4. Verify provisioning profile has NFC entitlement

## Entitlement Issues

### "Missing required entitlement" (Most Common)

| Step | What to Check |
|------|---------------|
| 1 | .entitlements file exists in Xcode (in Build Phases → Copy Bundle Resources) |
| 2 | .entitlements file has `com.apple.developer.nfc.readersession.formats` key |
| 3 | Provisioning profile regenerated after adding entitlement |
| 4 | App signed with correct team ID |
| 5 | Device provisioning profile installed on iPhone |

**Verify in Terminal:**
```bash
# Check app's entitlements
codesign -d --entitlements - /path/to/MyApp.app

# Should show:
# <key>com.apple.developer.nfc.readersession.formats</key>
# <array>
#   <string>NDEF</string>
#   <string>TAG</string>
# </array>
```

**If entitlements not present:**
1. In Xcode: **Signing & Capabilities** → check NFC capability listed
2. Delete derived data
3. Rebuild
4. Verify codesign output again

## NDEF Reading Issues

### Tags detected but data not parsed

```swift
// Common mistake: assuming record.payload is a string
let payload = String(data: record.payload, encoding: .utf8)
// ❌ Might fail if payload isn't UTF-8 encoded

// Better: check type first
if String(data: record.type, encoding: .utf8) == "U" {
    // URI record - first byte is prefix code
    let prefix = parseURIPrefix(record.payload[0])
    let uri = prefix + String(data: record.payload.dropFirst(), encoding: .utf8)!
}
```

### NDEF record type not recognized

```swift
// Debug: inspect raw record data
func debugRecord(_ record: NFCNDEFRecord) {
    print("=== Record Debug Info ===")
    print("TNF: \(record.typeNameFormat.rawValue) (0=Empty, 1=WellKnown, 2=Media, etc.)")
    print("Type bytes: \(record.type.hexDescription)")
    print("Type string: \(String(data: record.type, encoding: .utf8) ?? "(not UTF-8)")")
    print("Payload length: \(record.payload.count)")
    print("Payload (hex): \(record.payload.hexDescription)")
    
    if let utf8 = String(data: record.payload, encoding: .utf8) {
        print("Payload (UTF-8): \(utf8)")
    }
}

extension Data {
    var hexDescription: String {
        map { String(format: "%02x", $0) }.joined(separator: " ")
    }
}
```

## Writing Issues

### Write fails silently

```swift
// Common: tag is read-only
tag.writeNDEFMessage(message) { error in
    if let error = error {
        print("Write failed: \(error.localizedDescription)")
        // Check error type
        if let tagError = error as? NSError {
            print("Domain: \(tagError.domain)")
            print("Code: \(tagError.code)")
            print("UserInfo: \(tagError.userInfo)")
        }
    }
}

// Possible reasons:
// - Tag is read-only (manufacturer-locked)
// - Tag is password-protected
// - Tag is full (no space for message)
// - Wrong tag type (Type 1 is read-only)
// - Connection was lost
```

### "Write not available for tag type"

```swift
// Some tag types don't support writing via CoreNFC
func canWriteToTag(_ tag: NFCTag) -> Bool {
    switch tag {
    case .iso7816:      return true   // Supports NDEF write
    case .iso15693:     return true   // Native write only
    case .feliCa:       return true   // Native write only
    case .mifare:       return true   // Some models only
    @unknown default:   return false
    }
}

// For ISO15693/FeliCa/MIFARE, use native protocol commands
// not NDEF write
```

## Background Reading Not Working

### Tags detected in background but app doesn't launch

**Checklist:**
- [ ] `nfc` added to `UIBackgroundModes` in Info.plist
- [ ] NDEF message contains URL record
- [ ] URL is a valid Universal Link (HTTPS only)
- [ ] apple-app-site-association is valid and accessible
- [ ] URL path matches pattern in apple-app-site-association
- [ ] Screen was on when tag was scanned (iOS requires this)

**Test background detection:**
```swift
// In AppDelegate
func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    
    // Check if launched via Universal Link
    if let userActivityDictionary = launchOptions?[.userActivityDictionary] as? [AnyHashable: Any] {
        if let userActivity = userActivityDictionary.values.first as? NSUserActivity {
            if let url = userActivity.webpageURL {
                print("🎉 Launched from NFC tag: \(url)")
            }
        }
    }
    
    return true
}
```

## Type 5 (ISO15693) Specific Issues

### "90%+ failure rate on iPhone 12-15"

**Known issue:** ISO15693 tags have severe reliability problems on iPhone 12-15.

**Solutions:**
1. **Use different tag type:** Switch to Type 2-4 NDEF tags (much more reliable)
2. **Test on iPhone 7-11:** Verify it works there; if yes, it's a hardware issue on 12-15
3. **Increase retry logic:** Add exponential backoff and retry failed operations
4. **Check antenna position:** Some iPhone models have antenna placement issues with ISO15693

**Workaround code:**
```swift
let maxRetries = 5
var retryCount = 0

func readISO15693WithRetry(_ tag: NFCISO15693Tag) {
    tag.readSingleBlock(requestFlags: [.highDataRate, .address],
                       blockNumber: 0) { response, error in
        if error != nil && retryCount < maxRetries {
            retryCount += 1
            DispatchQueue.main.asyncAfter(deadline: .now() + Double(retryCount) * 0.5) {
                self.readISO15693WithRetry(tag)
            }
        } else if let error = error {
            print("Failed after \(self.retryCount) retries: \(error)")
        } else {
            print("Success on attempt \(self.retryCount + 1)")
        }
    }
}
```

## Permission/Privacy Issues

### User never sees NFC permission prompt

**Cause:** `NFCReaderUsageDescription` missing or empty

**Fix:**
```xml
<!-- In Info.plist -->
<key>NFCReaderUsageDescription</key>
<string>We use NFC to read your tag information</string>
```

After adding, must restart app.

### App has permission but NFC doesn't work

```swift
// Check NFC availability at runtime
import CoreNFC

func isNFCAvailable() -> Bool {
    return NFCNDEFReaderSession.readingAvailable
}

@IBAction func scanTapped(_ sender: UIButton) {
    if !isNFCAvailable() {
        print("⚠️ NFC not available on this device")
        showAlert("NFC Not Available", "This device doesn't support NFC")
        return
    }
    
    startScanning()
}
```

## Network/Connection Issues

### Universal Link doesn't work (app doesn't launch from NFC)

See: [UNIVERSAL_LINKS_TESTING.md](./UNIVERSAL_LINKS_TESTING.md)

**Quick checklist:**
```bash
# 1. Is apple-app-site-association accessible?
curl -v https://myapp.com/.well-known/apple-app-site-association

# 2. Is it valid JSON?
curl https://myapp.com/.well-known/apple-app-site-association | python3 -m json.tool

# 3. Does it list your Bundle ID?
# Check the appID field matches your actual Bundle ID
```

## Performance Issues

### Session takes >10 seconds to detect tag

```swift
// Check polling options
let session = NFCTagReaderSession(
    pollingOption: .iso14443  // Faster for Type 2-4
)

// vs

let session = NFCTagReaderSession(
    pollingOption: [.iso14443, .iso15693, .iso18092]  // Slower: more options
)
```

**Optimization:**
- Only enable protocols you actually use
- ISO14443 is fastest for Type 2-4 tags
- Add ISO15693 only if needed
- Avoid scanning for all protocols simultaneously

## Debugging Tips

### Enable verbose NFC logging (Xcode)

```swift
import os.log

let nfcLogger = OSLog(subsystem: "com.myapp.nfc", category: "corenfc")

func readerSessionDidBecomeActive(_ session: NFCReaderSession) {
    os_log("NFC session active", log: nfcLogger, type: .debug)
}

func readerSession(_ session: NFCNDEFReaderSession,
                  didDetectNDEFs messages: [NFCNDEFMessage]) {
    os_log("Detected %d messages", log: nfcLogger, messages.count)
    for (idx, msg) in messages.enumerated() {
        os_log("Message %d has %d records", log: nfcLogger, idx, msg.records.count)
    }
}
```

View logs in Xcode Console or with:
```bash
log stream --predicate 'subsystem contains "com.myapp.nfc"'
```

### Use third-party NFC tools for verification

- **NFC Tools (iOS)** — Read any tag, verify data
- **TagWriter (iOS)** — Write test tags
- **Proxmark3** — Advanced RF debugging
- **Android NFC Tools** — Cross-platform verification

## Still Stuck?

1. **Check [Technical Reference](./TECHNICAL.md)** for API details
2. **Review [Code Examples](./CODE_EXAMPLES.md)** for working patterns
3. **Verify [Universal Links Setup](./UNIVERSAL_LINKS_TESTING.md)** if using URL routing
4. **File an issue** on this repository with:
   - iOS version
   - iPhone model
   - Tag type (if known)
   - Exact error message
   - Minimal code example
