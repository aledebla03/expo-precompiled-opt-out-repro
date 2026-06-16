# Repro — `expo-build-properties` `ios.usePrecompiledModules: false` is ignored when `EXPO_USE_PRECOMPILED_MODULES=1` is pre-set (e.g. EAS Build)

Minimal reproducible example for an `expo/expo` SDK bug report.

`app.json` sets:

```json
["expo-build-properties", { "ios": { "usePrecompiledModules": false } }]
```

This writes `"EXPO_USE_PRECOMPILED_MODULES": "false"` into `ios/Podfile.properties.json`, yet precompiled Expo modules stay **enabled** whenever the env var is already `1` (the default on EAS Build for SDK 56).

## Reproduce

```bash
npm install
npx expo prebuild --clean -p ios

# A) No env var → opt-out is honored, modules build from source:
pod install --project-directory=ios | grep -c "Expo-precompiled"
#   => 0

# B) Env var pre-set (as EAS Build does) → opt-out is IGNORED:
EXPO_USE_PRECOMPILED_MODULES=1 pod install --project-directory=ios | grep "Expo-precompiled"
#   => prints the full precompiled module list (ExpoImage, ExpoModulesCore, …)
```

The only difference between A and B is the pre-set `EXPO_USE_PRECOMPILED_MODULES=1`, which `usePrecompiledModules: false` does not override.

## Environment

- Expo SDK 56 (`expo` 56.0.12), `react-native` 0.85.3, `react` 19.2.3
- `expo-build-properties` 56.0.19, `expo-modules-autolinking` 56.0.16, `expo-image` 56.0.11
- CocoaPods 1.16.2, Xcode 26.5, macOS 26.2, Node 22.22.2
- `npx expo-doctor`: 21/21 checks passed
