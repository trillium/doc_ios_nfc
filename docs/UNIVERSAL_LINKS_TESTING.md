# Universal Links & NFC: Testing and Pitfalls

Critical information for using Universal Links with NFC tags.

## Key Footguns

### 1. **Universal Link Lookup Happens ONLINE**

⚠️ **This is the biggest footgun.** Universal Links require network connectivity to iOS to verify domain ownership.

**What happens:**
1. NFC tag contains URL → `https://myapp.com/product/123`
2. iOS reads tag
3. iOS **makes HTTP request** to `https://myapp.com/.well-known/apple-app-site-association`
4. Only if valid: app is launched with URL

**If offline:**
- iOS does NOT launch your app
- iOS does NOT open URL in Safari
- URL is silently ignored
- User sees nothing happen

**Mitigation:** You MUST validate the apple-app-site-association file BEFORE tags go into production.

### 2. **Caching Issues**

iOS caches the `apple-app-site-association` file:
- **First fetch:** ~24 hours after app first runs
- **Subsequent fetches:** Every ~24 hours
- **If validation fails:** iOS stops checking for 3+ days

**Footgun scenario:**
- Deploy app with tags (both point to `https://myapp.com`)
- apple-app-site-association not yet set up
- iOS caches "invalid" for 3+ days
- Set up apple-app-site-association correctly
- Tags still don't work for 3+ days
- Users confused why it stopped working

**Fix:** Set up apple-app-site-association BEFORE any public tags exist.

### 3. **HTTPS Requirement**

Universal Links ONLY work over HTTPS. HTTP is silently rejected.

**Footgun:**
```
NFC tag: http://myapp.com/product/123  ❌ Does NOT work
NFC tag: https://myapp.com/product/123 ✓ Works
```

Your NFC tag creation code MUST enforce HTTPS:

```swift
func validateURLForNFC(_ urlString: String) -> Bool {
    guard let url = URL(string: urlString) else { return false }
    return url.scheme == "https"  // Reject http
}
```

### 4. **Bundle ID Mismatch**

apple-app-site-association must list your correct Bundle ID.

**Footgun:**
```json
{
  "applinks": {
    "myapp.com": {
      "apps": [],
      "details": [
        {
          "appID": "ABC123.com.mycompany.myapp",  // Wrong team prefix
          "paths": ["*"]
        }
      ]
    }
  }
}
```

```swift
// Your app's actual Bundle ID: 123ABC.com.mycompany.myapp
// apple-app-site-association says: ABC123.com.mycompany.myapp
// Result: Universal Links silently fail
```

**Verify your Bundle ID:**
```swift
// In your app code
if let bundleID = Bundle.main.bundleIdentifier {
    print("Bundle ID: \(bundleID)")  // Copy this exact value
}
```

### 5. **Path Matching Errors**

apple-app-site-association uses path matching to decide what to open.

**Footgun:**
```json
{
  "applinks": {
    "myapp.com": {
      "apps": [],
      "details": [
        {
          "appID": "...",
          "paths": ["/product/*"]  // Only product/* paths
        }
      ]
    }
  }
}
```

```swift
// NFC tag: https://myapp.com/coupon/456 ❌ Doesn't match /product/*
// NFC tag: https://myapp.com/product/123 ✓ Matches /product/*
```

**Solution:** Use wildcard patterns correctly:
```json
"paths": ["*"]              // All paths
"paths": ["/product/*"]     // Only /product/...
"paths": ["/product/*/buy"] // Only /product/.../buy
"paths": ["NOT /admin/*"]   // All except /admin/...
```

### 6. **Wrong File Location**

apple-app-site-association must be at exact root location.

```
✓ https://myapp.com/.well-known/apple-app-site-association
✗ https://myapp.com/apple-app-site-association
✗ https://www.myapp.com/.well-known/apple-app-site-association (wrong subdomain)
✗ https://myapp.com/public/.well-known/apple-app-site-association
```

**Verify:**
```bash
curl -v https://myapp.com/.well-known/apple-app-site-association
# Should return 200 with JSON content
# Should have Content-Type: application/json
```

### 7. **Invalid JSON**

If the JSON is malformed, iOS silently rejects it.

**Footgun:**
```json
{
  "applinks": {
    "myapp.com": {
      "apps": [],
      "details": [
        {
          "appID": "...",
          "paths": ["/product/*"]
        }
      ]
    }
  }
  // Missing closing brace!
}
```

**Verify JSON:**
```bash
# Online: jsonlint.com
# Or locally:
python3 -m json.tool apple-app-site-association
```

## Testing Checklist

### Phase 1: Pre-Deployment Validation

```bash
# 1. Validate apple-app-site-association exists
curl -i https://myapp.com/.well-known/apple-app-site-association
# Should return 200, not 404, 301, or 500

# 2. Validate JSON format
curl https://myapp.com/.well-known/apple-app-site-association | python3 -m json.tool

# 3. Verify content-type
curl -i https://myapp.com/.well-known/apple-app-site-association | grep "Content-Type"
# Should contain: application/json (NOT text/html)

# 4. Check DNS resolution
nslookup myapp.com
# Verify it resolves to your server

# 5. Verify SSL certificate
openssl s_client -connect myapp.com:443 -servername myapp.com
# Check certificate is valid and not expired
```

### Phase 2: iOS Simulator Testing

```swift
// In your app's Debug menu or settings screen
// Add a test button that verifies Universal Link setup

@IBAction func testUniversalLinks(_ sender: UIButton) {
    // Test 1: Can we reach the apple-app-site-association endpoint?
    testAppleAppSiteAssociation()
    
    // Test 2: Can we parse the URL correctly?
    testURLParsing()
    
    // Test 3: Does our routing work?
    testDeepLinkRouting()
}

func testAppleAppSiteAssociation() {
    guard let url = URL(string: "https://myapp.com/.well-known/apple-app-site-association") else {
        print("❌ Invalid URL")
        return
    }
    
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            print("❌ Network error: \(error)")
            return
        }
        
        if let httpResponse = response as? HTTPURLResponse {
            if httpResponse.statusCode == 200 {
                print("✓ apple-app-site-association found (200)")
                
                // Verify it's valid JSON
                if let data = data {
                    do {
                        let _ = try JSONSerialization.jsonObject(with: data)
                        print("✓ Valid JSON format")
                    } catch {
                        print("❌ Invalid JSON: \(error)")
                    }
                }
            } else {
                print("❌ HTTP \(httpResponse.statusCode)")
            }
        }
    }
    
    task.resume()
}

func testURLParsing() {
    let testURLs = [
        "https://myapp.com/product/123",
        "https://myapp.com/coupon/456",
        "https://myapp.com/event/789"
    ]
    
    for urlString in testURLs {
        if let url = URL(string: urlString) {
            print("✓ Can parse: \(urlString)")
            print("  Path: \(url.path)")
            print("  Query: \(url.query ?? "none")")
        } else {
            print("❌ Cannot parse: \(urlString)")
        }
    }
}

func testDeepLinkRouting() {
    let testURLs = [
        (url: "https://myapp.com/product/123", expectedView: "ProductView"),
        (url: "https://myapp.com/coupon/456", expectedView: "CouponView")
    ]
    
    for test in testURLs {
        if let url = URL(string: test.url) {
            let view = determineViewFromURL(url)
            if view == test.expectedView {
                print("✓ \(test.url) → \(view)")
            } else {
                print("❌ \(test.url) → \(view) (expected \(test.expectedView))")
            }
        }
    }
}

func determineViewFromURL(_ url: URL) -> String {
    if url.path.contains("product") {
        return "ProductView"
    } else if url.path.contains("coupon") {
        return "CouponView"
    }
    return "UnknownView"
}
```

### Phase 3: Device Testing with NFC

1. **Create test NFC tags**
   ```swift
   // Write test URLs to actual NFC tags
   let testURLs = [
       "https://myapp.com/product/123",
       "https://myapp.com/coupon/456"
   ]
   // Use NFC Tools app or your own tag writer to create test tags
   ```

2. **Test on real iPhone**
   - Clean app (uninstall, reinstall)
   - Wait 30 seconds for OS to check apple-app-site-association
   - Put phone on WiFi (not just cellular)
   - Scan NFC tags
   - App should launch with correct view

3. **Test offline behavior**
   - Enable Airplane Mode
   - Scan NFC tags
   - Verify tags DON'T work (expected)
   - Disable Airplane Mode
   - Scan again
   - Tags should work

### Phase 4: Production Validation

```bash
# After going live, run these monthly

# 1. Is apple-app-site-association still accessible?
curl -f https://myapp.com/.well-known/apple-app-site-association > /dev/null && echo "✓ OK" || echo "❌ FAIL"

# 2. Are there any DNS issues?
dig myapp.com +short

# 3. Is SSL cert valid?
echo | openssl s_client -servername myapp.com -connect myapp.com:443 2>/dev/null | openssl x509 -noout -dates

# 4. Check app analytics for deep link opens
# (Implement in your app)
```

## Unit Test Suite

```swift
import XCTest

class UniversalLinkTests: XCTestCase {
    
    // MARK: - URL Validation Tests
    
    func testURLMustBeHTTPS() {
        XCTAssertTrue(isValidNFCURL("https://myapp.com/product/123"))
        XCTAssertFalse(isValidNFCURL("http://myapp.com/product/123"))  // HTTP fails
        XCTAssertFalse(isValidNFCURL("ftp://myapp.com/product/123"))   // Other schemes fail
    }
    
    func testURLMustHaveDomain() {
        XCTAssertTrue(isValidNFCURL("https://myapp.com/path"))
        XCTAssertFalse(isValidNFCURL("https:///path"))  // No domain
        XCTAssertFalse(isValidNFCURL(""))                // Empty
    }
    
    func testURLParsing() {
        let url = URL(string: "https://myapp.com/product/123?ref=nfc")!
        XCTAssertEqual(url.host, "myapp.com")
        XCTAssertEqual(url.path, "/product/123")
        XCTAssertEqual(url.query, "ref=nfc")
    }
    
    // MARK: - Routing Tests
    
    func testProductRouting() {
        let url = URL(string: "https://myapp.com/product/123")!
        let view = determineViewFromURL(url)
        XCTAssertEqual(view, "ProductView")
    }
    
    func testCouponRouting() {
        let url = URL(string: "https://myapp.com/coupon/456")!
        let view = determineViewFromURL(url)
        XCTAssertEqual(view, "CouponView")
    }
    
    func testUnknownPathDefaultsToHome() {
        let url = URL(string: "https://myapp.com/unknown/789")!
        let view = determineViewFromURL(url)
        XCTAssertEqual(view, "HomeView")
    }
    
    // MARK: - Edge Cases
    
    func testURLWithSpecialCharacters() {
        let url = URL(string: "https://myapp.com/product/abc-123_xyz%20test")!
        XCTAssertNotNil(url)
        XCTAssertTrue(url.path.contains("product"))
    }
    
    func testURLWithSubdomain() {
        // Only apex domain should work, not subdomain
        XCTAssertTrue(isValidNFCURL("https://myapp.com/product/123"))
        XCTAssertFalse(isValidNFCURL("https://www.myapp.com/product/123"))  // Subdomain
    }
    
    func testURLWithPort() {
        // Standard HTTPS port 443 is implicit
        let url1 = URL(string: "https://myapp.com:443/product/123")!
        let url2 = URL(string: "https://myapp.com/product/123")!
        XCTAssertEqual(url1.host, url2.host)
    }
    
    // MARK: - Helper Functions
    
    func isValidNFCURL(_ urlString: String) -> Bool {
        guard let url = URL(string: urlString) else { return false }
        guard url.scheme == "https" else { return false }
        guard url.host != nil else { return false }
        return true
    }
    
    func determineViewFromURL(_ url: URL) -> String {
        if url.path.contains("product") {
            return "ProductView"
        } else if url.path.contains("coupon") {
            return "CouponView"
        }
        return "HomeView"
    }
}
```

## Integration Test: End-to-End

```swift
import XCTest

class UniversalLinkIntegrationTests: XCTestCase {
    
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        app = XCUIApplication()
        app.launch()
    }
    
    func testOpenURLFromNotification() {
        // Simulate NFC tag triggering Universal Link
        let url = URL(string: "https://myapp.com/product/123")!
        
        app.open(url)
        
        // Verify correct view is displayed
        let productTitle = app.staticTexts["Product ID: 123"]
        XCTAssertTrue(productTitle.waitForExistence(timeout: 2.0))
    }
    
    func testDeepLinkParameterExtraction() {
        let url = URL(string: "https://myapp.com/coupon/SAVE50?ref=nfc")!
        app.open(url)
        
        let couponCode = app.staticTexts["SAVE50"]
        XCTAssertTrue(couponCode.waitForExistence(timeout: 2.0))
    }
}
```

## apple-app-site-association Example

```json
{
  "applinks": {
    "myapp.com": {
      "apps": [],
      "details": [
        {
          "appID": "TEAM123.com.mycompany.myapp",
          "paths": [
            "/product/*",
            "/coupon/*",
            "/event/*"
          ]
        }
      ]
    }
  },
  "webcredentials": {
    "apps": ["TEAM123.com.mycompany.myapp"]
  }
}
```

---

**Key Takeaway:** Always test apple-app-site-association validation BEFORE deploying tags to production.
