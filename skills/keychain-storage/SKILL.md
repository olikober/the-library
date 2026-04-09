# Skill: keychain-storage

Securely store and retrieve sensitive data (API keys, tokens, passwords) using Apple's Keychain Services API. Use when implementing secure credential storage in macOS/iOS apps.

## When to Use

- Storing API keys securely
- Saving authentication tokens
- Managing user credentials
- Protecting sensitive configuration data

## Why Keychain?

| Storage | Pros | Cons |
|---------|------|------|
| **Keychain** | Encrypted, secure, system-managed | More complex API |
| **UserDefaults** | Simple API | Not encrypted, unsuitable for secrets |
| **Files** | Flexible | Must implement own encryption |

**Never store API keys or passwords in UserDefaults or files without encryption.**

## Basic Keychain Operations

### Save to Keychain

```swift
import Foundation
import Security

enum KeychainError: Error {
    case duplicateItem
    case itemNotFound
    case unexpectedStatus(OSStatus)
    case invalidData
}

struct KeychainService {
    static let service = "com.yourapp.CreditCounter"
    
    static func save(key: String, value: String) throws {
        guard let data = value.data(using: .utf8) else {
            throw KeychainError.invalidData
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlocked
        ]
        
        let status = SecItemAdd(query as CFDictionary, nil)
        
        switch status {
        case errSecSuccess:
            return
        case errSecDuplicateItem:
            try update(key: key, value: value)
        default:
            throw KeychainError.unexpectedStatus(status)
        }
    }
    
    static func update(key: String, value: String) throws {
        guard let data = value.data(using: .utf8) else {
            throw KeychainError.invalidData
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        
        let attributes: [String: Any] = [
            kSecValueData as String: data
        ]
        
        let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
        
        guard status == errSecSuccess else {
            throw KeychainError.unexpectedStatus(status)
        }
    }
}
```

### Retrieve from Keychain

```swift
extension KeychainService {
    static func retrieve(key: String) throws -> String {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        switch status {
        case errSecSuccess:
            guard let data = result as? Data,
                  let string = String(data: data, encoding: .utf8) else {
                throw KeychainError.invalidData
            }
            return string
        case errSecItemNotFound:
            throw KeychainError.itemNotFound
        default:
            throw KeychainError.unexpectedStatus(status)
        }
    }
}
```

### Delete from Keychain

```swift
extension KeychainService {
    static func delete(key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.unexpectedStatus(status)
        }
    }
}
```

### Check Existence

```swift
extension KeychainService {
    static func exists(key: String) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: false
        ]
        
        let status = SecItemCopyMatching(query as CFDictionary, nil)
        return status == errSecSuccess
    }
}
```

## Observable Keychain Manager

```swift
import Foundation
import Observation

@Observable
class KeychainManager {
    enum Key: String {
        case openRouterAPIKey = "openrouter-api-key"
    }
    
    static let shared = KeychainManager()
    private let service = "com.yourapp.CreditCounter"
    
    private init() {}
    
    func save(_ value: String, for key: Key) throws {
        try KeychainService.save(key: key.rawValue, value: value)
    }
    
    func retrieve(_ key: Key) -> String? {
        try? KeychainService.retrieve(key: key.rawValue)
    }
    
    func delete(_ key: Key) throws {
        try KeychainService.delete(key: key.rawValue)
    }
    
    func hasKey(_ key: Key) -> Bool {
        KeychainService.exists(key: key.rawValue)
    }
}
```

## Convenience Wrapper

```swift
// Simple one-liners for common operations
extension KeychainManager {
    func setAPIKey(_ key: String) {
        try? save(key, for: .openRouterAPIKey)
    }
    
    var apiKey: String? {
        retrieve(.openRouterAPIKey)
    }
    
    func clearAPIKey() {
        try? delete(.openRouterAPIKey)
    }
}
```

## Usage in App

```swift
@Observable
class AppState {
    var apiKey: String? {
        didSet {
            if let key = apiKey {
                try? KeychainManager.shared.setAPIKey(key)
            }
        }
    }
    
    init() {
        // Load API key on init
        apiKey = KeychainManager.shared.apiKey
    }
}
```

## Accessibility Levels

Choose the right accessibility level for your use case:

| Level | When Unlocked | Use Case |
|-------|---------------|----------|
| `kSecAttrAccessibleWhenUnlocked` | Only when unlocked | Default for most apps |
| `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` | Only when unlocked, not synced | Highly sensitive data |
| `kSecAttrAccessibleAfterFirstUnlock` | After first unlock | Background tasks |
| `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly` | After first unlock, not synced | Background tasks, sensitive |

## Troubleshooting

### Common Status Codes

| Status | Meaning | Solution |
|--------|---------|----------|
| `-25299` | Duplicate item | Use update instead of add |
| `-25300` | Item not found | Check key name |
| `-25293` | Auth failed | Check accessibility level |
| `-25294` | Too many items | Delete old items first |

### Debugging

```swift
func handleKeychainError(_ status: OSStatus) -> String {
    if let message = SecCopyErrorMessageString(status, nil) {
        return message as String
    }
    return "Unknown error: \(status)"
}
```

## References

- [Keychain Services API](https://developer.apple.com/documentation/security/keychain_services)
- [Storing Keys in Keychain](https://developer.apple.com/documentation/security/storing-keys-in-the-keychain)
- [SecItemAdd](https://developer.apple.com/documentation/security/secitemadd(_:_:))
- [SecItemCopyMatching](https://developer.apple.com/documentation/security/secitemcopymatching(_:_:))
- [Security Framework Result Codes](https://developer.apple.com/documentation/security/1542101-security_framework_result_codes)