# Getting Started with NFC on iOS

Step-by-step guide to set up your first NFC tag reader app.

## Prerequisites

- Xcode 14+
- iPhone 7 or later (A10 Fusion chip minimum)
- iOS 11+ target (or iOS 13+ for writing/native protocols)
- Apple Developer Account (for provisioning)

## Step 1: Configure Project in Xcode

### 1.1 Link CoreNFC Framework

1. Select your project in Xcode
2. Select your target
3. Go to **Build Phases** → **Link Binary With Libraries**
4. Click **+** and add `CoreNFC.framework`
5. Framework should now appear in the linked libraries list

### 1.2 Add NFC Capability

1. In Xcode, go to **Signing & Capabilities**
2. Click **+ Capability**
3. Search for "NFC Tag Reading"
4. Select it and Xcode auto-generates an `.entitlements` file

Your app now has the default NFC entitlements.

## Step 2: Configure Info.plist

Add the usage description that users see when prompted:

```xml
<key>NFCReaderUsageDescription</key>
<string>We need NFC to read tag information</string>
```

### Optional: For Specific Protocols

If using ISO7816 or FeliCa in background:

```xml
<!-- FeliCa System Codes (for transit cards) -->
<key>com.apple.developer.nfc.readersession.felica.systemcodes</key>
<array>
    <string>88B4</string>  <!-- JR East -->
</array>

<!-- ISO7816 Application IDs -->
<key>com.apple.developer.nfc.readersession.iso7816.select-identifiers</key>
<array>
    <string>A0000002471001</string>
</array>
```

## Step 3: Configure Entitlements File

Xcode created a `.entitlements` file. Edit it to match your use case:

**For basic NDEF reading:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.nfc.readersession.formats</key>
    <array>
        <string>NDEF</string>
        <string>TAG</string>
    </array>
</dict>
</plist>
```

**For native tag protocols:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.nfc.readersession.formats</key>
    <array>
        <string>NDEF</string>
        <string>TAG</string>
    </array>
    
    <!-- Native Tag Protocols -->
    <key>com.apple.developer.nfc.readersession.iso7816</key>
    <true/>
    <key>com.apple.developer.nfc.readersession.iso15693</key>
    <true/>
    <key>com.apple.developer.nfc.readersession.felica</key>
    <true/>
    <key>com.apple.developer.nfc.readersession.mifare</key>
    <true/>
</dict>
</plist>
```

## Step 4: Update Provisioning Profile

The entitlements in Xcode need to be reflected in your provisioning profile:

1. Go to [Apple Developer Portal](https://developer.apple.com/account)
2. Navigate to **Certificates, Identifiers & Profiles** → **Identifiers**
3. Select your App ID
4. Click **Edit**
5. Check "NFC Tag Reading" and save
6. Go to **Profiles**
7. Select your provisioning profile and regenerate it
8. Download and install in Xcode

## Step 5: Write Basic Code

### Simple NDEF Reader

```swift
import UIKit
import CoreNFC

class NFCReaderViewController: UIViewController, NFCNDEFReaderSessionDelegate {
    
    var nfcSession: NFCNDEFReaderSession?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    func setupUI() {
        let button = UIButton(type: .system)
        button.setTitle("Scan NFC Tag", for: .normal)
        button.addTarget(self, action: #selector(startScanning), for: .touchUpInside)
        view.addSubview(button)
        button.center = view.center
    }
    
    @objc func startScanning() {
        // Create session
        nfcSession = NFCNDEFReaderSession(
            delegate: self,
            queue: .main,
            invalidateAfterFirstRead: true
        )
        
        // Show help text
        nfcSession?.alertMessage = "Hold your NFC tag near the top of your iPhone"
        
        // Start scanning
        nfcSession?.begin()
    }
    
    // Required: Called when session becomes active
    func readerSessionDidBecomeActive(_ session: NFCReaderSession) {
        print("NFC scanning active")
    }
    
    // Called when NDEF tags detected
    func readerSession(_ session: NFCNDEFReaderSession,
                      didDetectNDEFs messages: [NFCNDEFMessage]) {
        
        DispatchQueue.main.async {
            self.handleMessages(messages)
        }
    }
    
    func handleMessages(_ messages: [NFCNDEFMessage]) {
        var results = [String]()
        
        for message in messages {
            for record in message.records {
                if let text = String(data: record.payload, encoding: .utf8) {
                    results.append("Type: \(String(data: record.type, encoding: .utf8) ?? "unknown")")
                    results.append("Payload: \(text)")
                }
            }
        }
        
        // Display results
        let alert = UIAlertController(title: "NFC Tag Read",
                                      message: results.joined(separator: "\n"),
                                      preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
    
    // Required: Called on error or session end
    func readerSession(_ session: NFCReaderSession,
                      didInvalidateWithError error: Error) {
        print("NFC session ended: \(error.localizedDescription)")
    }
}
```

## Step 6: Test

### With Actual NFC Tags
1. Build and run on iPhone 7+
2. Tap "Scan NFC Tag"
3. Hold an NFC tag near the top of your iPhone
4. Wait for detection and response

### Without Physical Tags
- Use iOS Simulator with an NFC tag simulator (limited options)
- Use another iPhone running your own tag-writing app
- Purchase test NFC tags (Type 2 tags are cheap and widely available)

## Troubleshooting

### "Missing required entitlement"
- Regenerate provisioning profile
- Delete derived data: `rm -rf ~/Library/Developer/Xcode/DerivedData`
- Clean build folder and rebuild

### Session times out without detecting
- Make sure tag is close (within 4 inches)
- Verify tag is NDEF-formatted
- Try different tag position (top of phone, center)
- Check tag type is supported (Type 2-4 recommended)

### No permission prompt appears
- Check `NFCReaderUsageDescription` in Info.plist
- Rebuild and reinstall app

## Next Steps

- Read [Technical Reference](./TECHNICAL.md) for API details
- Check [Code Examples](./CODE_EXAMPLES.md) for common patterns
- See [Troubleshooting](./TROUBLESHOOTING.md) for common issues

---

**Next:** Create your first complete app by following [Code Examples](./CODE_EXAMPLES.md)
