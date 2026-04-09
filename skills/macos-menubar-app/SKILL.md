# Skill: macos-menubar-app

Build macOS menu bar-only utility apps using SwiftUI's MenuBarExtra. Use when creating menu bar apps, status bar utilities, or apps without Dock presence.

## When to Use

- Creating menu bar-only macOS apps (no Dock icon)
- Implementing status bar utilities with dropdown menus
- Building apps that live entirely in the system menu bar
- Using MenuBarExtra with SwiftUI

## MenuBarExtra Basics

### Minimal Menu Bar App

```swift
@main
struct CreditCounterApp: App {
    var body: some Scene {
        MenuBarExtra("Credit Counter", systemImage: "dollarsign.circle") {
            MenuContent()
        }
    }
}
```

### MenuBarExtra with State Binding

```swift
@main
struct AppWithMenuBarExtra: App {
    @AppStorage("showMenuBarExtra") private var showMenuBarExtra = true

    var body: some Scene {
        MenuBarExtra(
            "Credit Counter",
            systemImage: "dollarsign.circle",
            isInserted: $showMenuBarExtra
        ) {
            MenuContent()
        }
    }
}
```

### Key Behaviors

- **Auto-quit**: When user removes menu bar extra, app terminates automatically
- **No Dock presence**: MenuBarExtra-only apps don't appear in Dock
- **System image**: Use SF Symbols for menu bar icon
- **Menu content**: Any SwiftUI View as dropdown menu

## Menu Structure

### Basic Menu with Actions

```swift
struct MenuContent: View {
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Status display
            Text("Credits: $45.50")
                .font(.headline)
            
            Divider()
            
            // Menu items
            Button("Refresh") {
                // Action
            }
            .keyboardShortcut("r", modifiers: .command)
            
            Divider()
            
            Button("Quit") {
                NSApplication.shared.terminate(nil)
            }
        }
        .padding()
    }
}
```

### Menu with Polling Interval Selection

```swift
struct MenuContent: View {
    @State private var pollingInterval: TimeInterval = 60
    @State private var isPaused = false
    
    let intervals: [(String, TimeInterval)] = [
        ("2 seconds", 2),
        ("5 seconds", 5),
        ("10 seconds", 10),
        ("60 seconds", 60)
    ]
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // Balance display
            HStack {
                Image(systemName: "dollarsign.circle.fill")
                Text("$\(balance, specifier: "%.2f")")
            }
            
            Divider()
            
            // Polling interval submenu
            Menu("Polling Interval") {
                ForEach(intervals, id: \.1) { label, interval in
                    Button(label) {
                        pollingInterval = interval
                        isPaused = false
                    }
                }
                
                Divider()
                
                Button("Paused") {
                    isPaused = true
                }
            }
            
            Divider()
            
            Button("Open OpenRouter") {
                if let url = URL(string: "https://openrouter.ai/account") {
                    NSWorkspace.shared.open(url)
                }
            }
            
            Divider()
            
            Button("Quit") {
                NSApplication.shared.terminate(nil)
            }
        }
        .padding()
    }
}
```

## App Configuration

### Info.plist Requirements

For menu bar-only apps, set:
- `LSUIElement`: `true` (hides from Dock)
- `NSHighResolutionCapable`: `true`

### App Entry Point

```swift
import SwiftUI

@main
struct CreditCounterApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        MenuBarExtra(
            "Credit Counter",
            systemImage: appState.isPaused ? "pause.circle" : "dollarsign.circle"
        ) {
            MenuContentView()
                .environmentObject(appState)
        }
    }
}
```

## State Management

### AppState with Observable

```swift
import Foundation
import Observation

@Observable
class AppState {
    var creditBalance: Double = 0
    var creditsUsed: Double = 0
    var pollingInterval: TimeInterval = 60
    var isPaused: Bool = false
    var isLoading: Bool = false
    var errorMessage: String?
    var apiKey: String?
    
    var remainingCredits: Double {
        creditBalance - creditsUsed
    }
    
    func startPolling() {
        // Timer-based polling implementation
    }
    
    func stopPolling() {
        // Stop timer
    }
}
```

## Best Practices

1. **Use @Observable** for state management (iOS 17+/macOS 14+)
2. **Store API keys in Keychain** - never in UserDefaults
3. **Use AppStorage** for user preferences (polling interval, etc.)
4. **Handle errors gracefully** - show error state in menu
5. **Use SF Symbols** for menu bar icon
6. **Keep menu responsive** - don't block on network calls
7. **Support keyboard shortcuts** for common actions

## References

- [SwiftUI MenuBarExtra](https://developer.apple.com/documentation/swiftui/menubarextra)
- [Creating menu bar apps](https://developer.apple.com/documentation/swiftui/menubarextra#)
- [LSUIElement Info.plist key](https://developer.apple.com/documentation/bundleresources/information_property_list/lsuielement)