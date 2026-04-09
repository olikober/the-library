# Skill: openrouter-api

Integrate with OpenRouter API to fetch credit balances, manage API keys, and monitor usage. Use when building apps that need OpenRouter credit information.

## When to Use

- Fetching OpenRouter account credit balance
- Checking API usage and remaining credits
- Authenticating with OpenRouter API keys
- Building credit monitoring utilities

## API Overview

### Base URL
```
https://openrouter.ai/api/v1
```

### Authentication
All requests require a Bearer token in the Authorization header:
```
Authorization: Bearer <your-api-key>
```

### Credits Endpoint

**GET** `/api/v1/credits`

Retrieves total credits purchased and used for the authenticated user.

**Request:**
```bash
curl https://openrouter.ai/api/v1/credits \
  -H "Authorization: Bearer <token>"
```

**Response:**
```json
{
  "data": {
    "total_credits": 100.50,
    "total_usage": 25.75
  }
}
```

## Swift Implementation

### CreditInfo Model

```swift
import Foundation

struct CreditInfo: Codable {
    let data: CreditData
    
    struct CreditData: Codable {
        let totalCredits: Double
        let totalUsage: Double
        
        enum CodingKeys: String, CodingKey {
            case totalCredits = "total_credits"
            case totalUsage = "total_usage"
        }
    }
    
    var remainingCredits: Double {
        data.totalCredits - data.totalUsage
    }
}
```

### Network Client

```swift
import Foundation

actor OpenRouterClient {
    private let session: URLSession
    private let baseURL = URL(string: "https://openrouter.ai/api/v1")!
    
    init() {
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        self.session = URLSession(configuration: config)
    }
    
    func fetchCredits(apiKey: String) async throws -> CreditInfo {
        let url = baseURL.appendingPathComponent("credits")
        
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        request.setValue("Bearer \(apiKey)", forHTTPHeaderField: "Authorization")
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw OpenRouterError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200:
            let decoder = JSONDecoder()
            return try decoder.decode(CreditInfo.self, from: data)
        case 401:
            throw OpenRouterError.unauthorized
        case 403:
            throw OpenRouterError.forbidden
        default:
            throw OpenRouterError.serverError(httpResponse.statusCode)
        }
    }
}

enum OpenRouterError: Error, LocalizedError {
    case invalidResponse
    case unauthorized
    case forbidden
    case serverError(Int)
    case noAPIKey
    case networkError(Error)
    
    var errorDescription: String? {
        switch self {
        case .invalidResponse:
            return "Invalid response from server"
        case .unauthorized:
            return "Invalid API key"
        case .forbidden:
            return "Access forbidden"
        case .serverError(let code):
            return "Server error: \(code)"
        case .noAPIKey:
            return "No API key configured"
        case .networkError(let error):
            return "Network error: \(error.localizedDescription)"
        }
    }
}
```

### Usage in ViewModel

```swift
import Foundation
import Observation

@Observable
class CreditViewModel {
    var credits: CreditInfo?
    var isLoading = false
    var errorMessage: String?
    
    private let client = OpenRouterClient()
    private var pollingTask: Task<Void, Never>?
    
    func fetchCredits(apiKey: String) async {
        isLoading = true
        errorMessage = nil
        
        do {
            credits = try await client.fetchCredits(apiKey: apiKey)
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func startPolling(apiKey: String, interval: TimeInterval) {
        stopPolling()
        
        pollingTask = Task {
            while !Task.isCancelled {
                await fetchCredits(apiKey: apiKey)
                try? await Task.sleep(nanoseconds: UInt64(interval * 1_000_000_000))
            }
        }
    }
    
    func stopPolling() {
        pollingTask?.cancel()
        pollingTask = nil
    }
}
```

## API Key Management

### Where to Get API Keys

1. Log into [OpenRouter](https://openrouter.ai)
2. Go to **Account** → **Keys**
3. Create a Management API key for credit access

### Key Types

| Key Type | Access | Use Case |
|----------|--------|----------|
| API Key | Chat completions only | Making LLM requests |
| Management Key | Credits + API access | Credit monitoring apps |

### Security Best Practices

- Store API keys in **Keychain** (never UserDefaults)
- Use `kSecClassGenericPassword` for storage
- Set `kSecAttrAccessible` to `kSecAttrAccessibleWhenUnlocked`
- Never log or expose API keys in UI

## Error Handling

```swift
// Map HTTP status codes to user-friendly messages
func handleError(_ error: Error) -> String {
    if let openRouterError = error as? OpenRouterError {
        return openRouterError.errorDescription ?? "Unknown error"
    }
    
    if (error as NSError).code == NSURLErrorNotConnectedToInternet {
        return "No internet connection"
    }
    
    return "Failed to fetch credits"
}
```

## References

- [OpenRouter API Docs](https://openrouter.ai/docs/api-reference)
- [Credits Endpoint](https://openrouter.ai/docs/api-reference/credits/get-credits)
- [Authentication Guide](https://openrouter.ai/docs/guides/overview/auth/management-api-keys)