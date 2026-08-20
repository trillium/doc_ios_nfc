# NFC Code Examples

Ready-to-use code patterns for common NFC scenarios.

## 1. Simple NDEF Tag Reader

```swift
import UIKit
import CoreNFC

class SimpleNFCReader: UIViewController, NFCNDEFReaderSessionDelegate {
    
    var nfcSession: NFCNDEFReaderSession?
    
    @IBAction func scanTapped(_ sender: UIButton) {
        nfcSession = NFCNDEFReaderSession(
            delegate: self,
            queue: .main,
            invalidateAfterFirstRead: true
        )
        nfcSession?.alertMessage = "Hold tag near iPhone"
        nfcSession?.begin()
    }
    
    func readerSessionDidBecomeActive(_ session: NFCReaderSession) {
        print("Session active")
    }
    
    func readerSession(_ session: NFCNDEFReaderSession,
                      didDetectNDEFs messages: [NFCNDEFMessage]) {
        DispatchQueue.main.async {
            for message in messages {
                for record in message.records {
                    print("Type: \(String(data: record.type, encoding: .utf8) ?? "unknown")")
                    print("Payload: \(record.payload)")
                }
            }
        }
        session.invalidate()
    }
    
    func readerSession(_ session: NFCReaderSession,
                      didInvalidateWithError error: Error) {
        print("Error: \(error.localizedDescription)")
    }
}
```

## 2. Parse URI Records

```swift
func parseURIRecord(_ record: NFCNDEFRecord) -> String? {
    guard record.typeNameFormat == .nfcWellKnown,
          String(data: record.type, encoding: .utf8) == "U",
          let payload = record.payload,
          payload.count > 1 else {
        return nil
    }
    
    let prefixCode = payload[0]
    let prefixes: [UInt8: String] = [
        0x00: "",
        0x01: "http://www.",
        0x02: "https://www.",
        0x03: "http://",
        0x04: "https://",
        0x05: "tel:",
        0x06: "mailto:"
    ]
    
    let prefix = prefixes[prefixCode] ?? ""
    let uriData = payload.subdata(in: 1..<payload.count)
    
    if let uriString = String(data: uriData, encoding: .utf8) {
        return prefix + uriString
    }
    
    return nil
}

// Usage
func readerSession(_ session: NFCNDEFReaderSession,
                  didDetectNDEFs messages: [NFCNDEFMessage]) {
    for message in messages {
        for record in message.records {
            if let uri = parseURIRecord(record) {
                print("URI: \(uri)")
                // Open in browser
                if let url = URL(string: uri) {
                    UIApplication.shared.open(url)
                }
            }
        }
    }
}
```

## 3. Parse Text Records

```swift
func parseTextRecord(_ record: NFCNDEFRecord) -> (language: String, text: String)? {
    guard record.typeNameFormat == .nfcWellKnown,
          String(data: record.type, encoding: .utf8) == "T",
          let payload = record.payload,
          payload.count > 1 else {
        return nil
    }
    
    let statusByte = payload[0]
    let encoding: String.Encoding = (statusByte & 0x80) == 0 ? .utf8 : .utf16BigEndian
    let languageLength = Int(statusByte & 0x3F)
    
    let langStart = 1
    let langEnd = min(langStart + languageLength, payload.count)
    let language = String(data: payload.subdata(in: langStart..<langEnd), encoding: .utf8) ?? "unknown"
    
    let textData = payload.subdata(in: langEnd..<payload.count)
    if let text = String(data: textData, encoding: encoding) {
        return (language, text)
    }
    
    return nil
}
```

## 4. Create and Write NDEF Message

```swift
func createURIMessage(_ urlString: String) -> NFCNDEFMessage? {
    guard let payload = NFCNDEFPayload.wellKnownTypeURIPayload(string: urlString) else {
        return nil
    }
    return NFCNDEFMessage(records: [payload])
}

func createTextMessage(_ text: String) -> NFCNDEFMessage? {
    guard let payload = NFCNDEFPayload.wellKnownTypeTextPayload(
        string: text,
        locale: Locale(identifier: "en-US")
    ) else {
        return nil
    }
    return NFCNDEFMessage(records: [payload])
}

func createSmartPoster(_ uri: String, _ title: String) -> NFCNDEFMessage? {
    guard let uriPayload = NFCNDEFPayload.wellKnownTypeURIPayload(string: uri),
          let titlePayload = NFCNDEFPayload.wellKnownTypeTextPayload(
              string: title,
              locale: Locale(identifier: "en-US")
          ) else {
        return nil
    }
    return NFCNDEFMessage(records: [uriPayload, titlePayload])
}

// Write to tag
func writeMessageToTag(_ message: NFCNDEFMessage, tag: NFCTag) {
    if case .iso7816(let iso7816Tag) = tag {
        iso7816Tag.writeNDEFMessage(message) { error in
            if let error = error {
                print("Write error: \(error)")
            } else {
                print("Write successful")
            }
        }
    }
}
```

## 5. Native Tag Reader (ISO7816)

```swift
class NativeTagReader: NSObject, NFCTagReaderSessionDelegate {
    
    var session: NFCTagReaderSession?
    
    func startReading() {
        session = NFCTagReaderSession(
            pollingOption: .iso14443,
            delegate: self
        )
        session?.alertMessage = "Hold tag near iPhone"
        session?.begin()
    }
    
    func tagReaderSessionDidBecomeActive(_ session: NFCReaderSession) {
        print("Tag reader active")
    }
    
    func tagReaderSession(_ session: NFCTagReaderSession,
                        didDetect tags: [NFCTag]) {
        guard let tag = tags.first else { return }
        
        session.connect(to: tag) { error in
            if let error = error {
                print("Connection error: \(error)")
                return
            }
            
            self.handleTag(tag, session: session)
        }
    }
    
    func handleTag(_ tag: NFCTag, session: NFCTagReaderSession) {
        switch tag {
        case .iso7816(let iso7816Tag):
            handleISO7816(iso7816Tag, session: session)
        case .iso15693(let iso15693Tag):
            handleISO15693(iso15693Tag, session: session)
        case .feliCa(let feliCaTag):
            handleFeliCa(feliCaTag, session: session)
        case .mifare(let mifareTag):
            handleMifare(mifareTag, session: session)
        default:
            print("Unknown tag type")
        }
    }
    
    func handleISO7816(_ tag: NFCISO7816Tag, session: NFCTagReaderSession) {
        print("ISO7816 tag detected")
        print("ID: \(tag.identifier)")
        print("Historical bytes: \(tag.historicalBytes ?? Data())")
        session.invalidate()
    }
    
    func handleISO15693(_ tag: NFCISO15693Tag, session: NFCTagReaderSession) {
        print("ISO15693 tag detected")
        print("ID: \(tag.identifier)")
        session.invalidate()
    }
    
    func handleFeliCa(_ tag: NFCFeliCaTag, session: NFCTagReaderSession) {
        print("FeliCa tag detected")
        print("ID: \(tag.identifier)")
        session.invalidate()
    }
    
    func handleMifare(_ tag: NFCMifareTag, session: NFCTagReaderSession) {
        print("MIFARE tag detected")
        print("ID: \(tag.identifier)")
        session.invalidate()
    }
    
    func tagReaderSession(_ session: NFCTagReaderSession,
                        didInvalidateWithError error: Error) {
        print("Session error: \(error)")
    }
}
```

## 6. Universal Link Navigation Pattern

```swift
// In SceneDelegate
func scene(_ scene: UIScene,
          continue userActivity: NSUserActivity) {
    
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else {
        return
    }
    
    // Route based on URL
    handleUniversalLink(url)
}

func handleUniversalLink(_ url: URL) {
    let pathComponents = url.pathComponents
    
    if pathComponents.contains("product"), let id = pathComponents.last {
        showProductView(productID: id)
    } else if pathComponents.contains("coupon") {
        showCouponView()
    } else if pathComponents.contains("event") {
        showEventView()
    }
}

// NFC tag contains: https://myapp.com/product/12345
// Scanning triggers handleUniversalLink → shows product view
```

## 7. Background Tag Detection with Notification

```swift
// Info.plist additions
// <key>UIBackgroundModes</key>
// <array>
//   <string>nfc</string>
// </array>

class BackgroundNFCHandler {
    
    static func setupBackgroundNFC() {
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound]) { granted, error in
            if granted {
                DispatchQueue.main.async {
                    UIApplication.shared.registerForRemoteNotifications()
                }
            }
        }
    }
    
    static func postNotification(title: String, body: String, userInfo: [AnyHashable: Any]) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        content.userInfo = userInfo
        
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
        let request = UNNotificationRequest(identifier: UUID().uuidString,
                                           content: content,
                                           trigger: trigger)
        UNUserNotificationCenter.current().add(request)
    }
}

// In AppDelegate
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                              didReceive response: UNNotificationResponse,
                              withCompletionHandler completionHandler: @escaping () -> Void) {
        
        let userInfo = response.notification.request.content.userInfo
        if let action = userInfo["action"] as? String {
            routeToView(for: action)
        }
        
        completionHandler()
    }
    
    func routeToView(for action: String) {
        // Navigate based on action from NFC tag
        switch action {
        case "checkout":
            showCheckoutView()
        case "loyalty":
            showLoyaltyView()
        default:
            break
        }
    }
}
```

## 8. Error Handling Pattern

```swift
func handleNFCError(_ error: Error) {
    if let nfcError = error as? NFCReaderError {
        switch nfcError.code {
        case .readerSessionInvalidationErrorSessionTimeout:
            print("Timeout: No tag detected in time")
            showError("No tag found. Try again.")
            
        case .readerSessionInvalidationErrorSessionTerminatedUnexpectedly:
            print("Session terminated by system")
            showError("Session ended unexpectedly")
            
        case .readerSessionInvalidationErrorUserCanceled:
            print("User canceled")
            // Don't show error for user-initiated cancel
            
        case .readerSessionInvalidationErrorSecurityViolation:
            print("Security violation - likely missing entitlements")
            showError("NFC not configured. Check entitlements.")
            
        case .tagConnectionLost:
            print("Connection lost to tag")
            showError("Lost connection to tag. Keep it close.")
            
        case .tagResponseError:
            print("Invalid response from tag")
            showError("Tag didn't respond correctly")
            
        default:
            print("Other NFC error: \(nfcError)")
            showError("NFC Error: \(nfcError.localizedDescription)")
        }
    }
}

func showError(_ message: String) {
    DispatchQueue.main.async {
        let alert = UIAlertController(title: "Error",
                                      message: message,
                                      preferredStyle: .alert)
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        // Present alert
    }
}
```

## 9. FeliCa Transit Card Reader

```swift
func readFeliCaCard(_ tag: NFCFeliCaTag, session: NFCTagReaderSession) {
    // JR East system code (common in Japan)
    let jrEastSystemCode = Data([0x88, 0xB4])
    
    tag.polling(systemCode: jrEastSystemCode) { pollResponse, error in
        guard error == nil else {
            print("Polling failed: \(error!)")
            return
        }
        
        // Request service for reading
        let requestService = Data([0x0F, 0x01])
        
        tag.readWithoutEncryption(systemCode: jrEastSystemCode,
                                  services: [requestService],
                                  blocks: [Data([0x80, 0x00])]) { response, error in
            
            if error == nil, let responseData = response {
                self.parseTransitData(responseData)
            } else {
                print("Read failed: \(error?.localizedDescription ?? "unknown")")
            }
            
            session.invalidate()
        }
    }
}

func parseTransitData(_ data: Data) {
    print("Transit data received: \(data.hexDescription)")
    // Parse specific transit system format
    // Structure varies by system (JR East, Suica, Pasmo, etc.)
}
```

## 10. NDEF Message Inspection

```swift
func inspectMessage(_ message: NFCNDEFMessage) {
    print("=== NDEF Message ===")
    print("Number of records: \(message.records.count)")
    print("Message length: \(message.length) bytes")
    
    for (index, record) in message.records.enumerated() {
        print("\n--- Record \(index + 1) ---")
        print("TNF: \(record.typeNameFormat.rawValue)")
        print("Type: \(String(data: record.type, encoding: .utf8) ?? "(binary)")")
        print("Identifier: \(String(data: record.identifier, encoding: .utf8) ?? "(empty)")")
        print("Payload length: \(record.payload.count) bytes")
        
        if let payloadString = String(data: record.payload, encoding: .utf8) {
            print("Payload (UTF-8): \(payloadString)")
        } else {
            print("Payload (hex): \(record.payload.hexDescription)")
        }
    }
}

// Extension for hex representation
extension Data {
    var hexDescription: String {
        return map { String(format: "%02x", $0) }.joined(separator: " ")
    }
}
```

---

**More patterns available in:** [Technical Reference](./TECHNICAL.md)
