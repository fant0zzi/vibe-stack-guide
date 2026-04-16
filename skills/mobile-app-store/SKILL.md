---
name: mobile-app-store
description: Scaffold and iterate on a vibe-coded mobile app that must ship through Apple App Store or Google Play, using Expo (managed workflow) + React Native + TypeScript + Expo Router + EAS Build/Submit. Use this skill whenever the user says "mobile app", "iOS app", "Android app", "native app", "App Store", "Google Play", "React Native", "Expo", or describes a product that must be installable from the app stores. Apply it for the full lifecycle — scaffold, Expo Go verification, EAS build, and store submission — not just day one. If the user doesn't actually need the store, route them to the `mobile-pwa` skill instead.
---

# Mobile App (App Store) — Vibe Coding Path

This skill is for helping an experienced developer ship a mobile app through the App Store or Google Play, mostly with an AI coding agent. The goal is the shortest verifiable path from "empty folder" to "app installed on my phone via Expo Go" and then to "store submission."

**Default recommendation, no hedging:** Expo (managed workflow) + React Native + TypeScript + Expo Router + EAS Build + EAS Submit.

**Before anything else:** confirm the store is actually required. "I want an app on my phone" is often "I want a PWA." Ask: *"Do you need the Apple/Google store, or is a web app installable to the home screen enough?"* If they don't need the store, hand off to `mobile-pwa`.

## Why this stack (use only when asked)

Expo is the only mobile framework with deep AI-agent investment: `create-expo-app` [generates `AGENTS.md` by default](https://docs.expo.dev/more/create-expo/), docs are SDK-version-matched, there's an official MCP server, and Replit's mobile publishing flow is built on Expo. Verification is instant — scan a QR code with Expo Go and see it on the phone. EAS Build and EAS Submit absorb the signing / provisioning / store-submission work that agents can't automate.

Don't default to SwiftUI (only when Apple-only + native APIs like HealthKit or ARKit), Kotlin Multiplatform (toolchain complexity, thin agent support), bare React Native (native-module linking pain that Expo hides), or Flutter (improving AI tooling but no public evals, MCP server still experimental).

## Scaffold workflow

1. `npx create-expo-app@latest <project> --template` — pick the default TypeScript template. Verify it boots with `npx expo start` and that the QR code opens in Expo Go on a physical device.
2. Set up `expo-router` (included in the default template) before adding screens.
3. Use Expo SDK libraries before any third-party package. Only add a third-party package with a one-sentence justification in the same response.
4. Add EAS: `eas init`, then `eas build --profile preview --platform all` for the first installable builds. Do this before adding app-icon/splash customization — it catches signing issues early.
5. Store submission via `eas submit`. Only run this when the app has passed a full manual tap-through on a real device.

## First prompt to hand the user

```
I'm building [describe your app in one sentence]. Target iOS and Android.

Stack: Expo (managed workflow), React Native, TypeScript, Expo Router,
EAS Build + Submit.

Apply these stack-specific constraints:

- Never eject from Expo managed workflow. If a feature requires ejecting,
  tell me and suggest an Expo SDK alternative.
- Use Expo SDK libraries before any third-party package. Justify any
  third-party addition with one sentence.
- Keep navigation flat — no nested navigators until 5+ screens.
- Use expo-secure-store for sensitive data, not AsyncStorage.
- Test every change in Expo Go before proceeding.
- Every response ends with a verification step I can run on my phone.

Start by creating the project and showing me how to run it on my phone
with Expo Go.
```

## Constraints (enforce on every response)

- **Never eject from the managed workflow.** If a feature genuinely requires it, flag the trade-off and propose an Expo SDK alternative first.
- **Expo SDK first, third-party second.** Every third-party package needs a one-sentence justification.
- **Flat navigation** until 5+ screens. No nested navigators for the first milestones.
- **`expo-secure-store` for sensitive data.** Never `AsyncStorage` for tokens or PII.
- **Every change is verified in Expo Go** on a real device before moving on.
- **Keep files under 300 lines.** React Native components trend long — split early.
- **Every response ends with a phone-testable verification step.**

## Verification checklist

```bash
npx expo start                                      # scan QR with Expo Go
eas build --platform all --profile preview          # installable builds
npx tsc --noEmit                                    # types clean
# Manual tap-through: every screen, every tab. If any screen crashes, stop.
```

Before `eas submit`: run through the full app once on a real device with network off, then on again.

## Stop signals

- Agent suggests "ejecting" from the managed workflow.
- A required feature depends on a native module that isn't in the Expo SDK.
- Navigation nests 3+ levels deep.
- App won't run on a physical device via Expo Go.
- EAS build times exceed 15 minutes and nobody can explain why.

Surface these explicitly instead of silently routing around them.

## Escape routes

- **App Store friction is the blocker, not the code:** switch to `mobile-pwa` if the store isn't actually required. If it is, hire a mobile specialist for the signing/submission leg only — keep the code.
- **Need a native capability Expo doesn't expose:** evaluate a dev client build (`expo-dev-client`) before ejecting. This keeps most of the managed workflow intact.
- **Design-heavy UI work:** React and TypeScript skills transfer — but mobile layout is its own beast; the agent should show you each screen on device before iterating.

## Handing off to other skills

- Backend API for the mobile app → `api-backend` skill (FastAPI).
- Multi-agent separation for a bigger build → `pipeline` skill.
- Something feels off during development → `red-flags` skill.
