---
name: react-native-expo
description: Build React Native apps with Expo SDK 52-54+. Covers New Architecture, React 19 changes, SDK breaking changes. Use when building Expo apps, migrating to New Architecture, or fixing build errors.
---

# React Native Expo (SDK 52+)

Build React Native 0.76+ apps with Expo. Covers New Architecture (mandatory in 0.82+), React 19, and SDK 54 breaking changes.

## When to use
- Building new Expo apps
- Migrating to New Architecture
- Upgrading to SDK 54+
- Fixing Fabric/TurboModule errors
- iOS build failures with Swift

## Quick Start
```bash
npx create-expo-app@latest my-app
cd my-app
npx expo install react-native@latest expo@latest
npx expo start
```

## Critical Breaking Changes

| Version | Change |
|---------|--------|
| 0.82+ | New Architecture mandatory (cannot disable) |
| React 19 | propTypes removed, forwardRef deprecated |
| SDK 54 | expo-av removed → use expo-audio/video |
| SDK 54 | edge-to-edge mandatory on Android |
| 0.79+ | Chrome debugger removed → use DevTools |

## Common Issues (Load references when needed)

- **Fabric/TurboModule errors** → `references/new-architecture-errors.md`
- **React 19 migration** → `references/react-19-migration.md`
- **SDK 54 breaking changes** → `references/expo-sdk-52-breaking.md`

## Workflow
1. Create/upgrade project: `npx create-expo-app@latest`
2. Verify New Architecture: `npx expo config --type introspect | grep newArchEnabled`
3. Start dev server: `npx expo start` (press 'j' for DevTools)
4. Test on iOS/Android

## References (load only when needed)
- `references/react-19-migration.md` - propTypes, forwardRef, new hooks
- `references/new-architecture-errors.md` - Fabric, TurboModule fixes
- `references/expo-sdk-52-breaking.md` - SDK 52-54 changes
- `assets/css-features-cheatsheet.md` - New CSS properties
- `assets/new-arch-decision-tree.md` - Version selection guide

## Scripts
- `scripts/check-rn-version.sh` - Check React Native version and architecture status

## Key Constraints
- Hermes only (JSC removed from Expo Go)
- Use Redux Toolkit (not legacy redux)
- Use react-i18next (not i18n-js)
- No deep imports: `react-native/Libraries/*`
