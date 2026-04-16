---
name: mobile-pwa
description: Scaffold and iterate on a vibe-coded mobile-friendly Progressive Web App (PWA) that does NOT need the App Store — internal tool, field app, MVP, or anything where store distribution isn't required. Stack is Next.js 16 + TypeScript + Tailwind + service worker + web app manifest on Vercel. Use this skill whenever the user says "PWA", "progressive web app", "no app store", "installable web app", "internal tool on phones", "field app", "mobile MVP", "add to home screen", or describes a mobile use case without store distribution. If they need the Apple/Google store, route to `mobile-app-store` instead.
---

# Mobile App — No App Store (PWA) — Vibe Coding Path

This skill is for helping an experienced developer ship a phone-friendly web app that installs to the home screen, mostly with an AI coding agent. The point of this path is to skip the entire category of deployment problems (signing, provisioning, store review) that agents can't automate.

**Default recommendation, no hedging:** Next.js 16 (App Router) + TypeScript + Tailwind + directly-authored service worker and web app manifest, deployed on Vercel.

**Gate question before scaffolding:** *"Do you need push notifications on iOS, Bluetooth/NFC, or does a stakeholder insist on a store listing? If yes to any, we should use `mobile-app-store` instead."*

## Why this stack (use only when asked)

A PWA is installable, offline-capable, and deploys like a website — no signing, no provisioning, no store review. You get the same Next.js agent support as the web-app path (published evals, `AGENTS.md`, version-matched docs), and Lighthouse gives a concrete verification tool for PWA behavior. Trade-offs: iOS push is limited, no Bluetooth/NFC/hardware APIs, some users expect "a real app." For internal tools, dashboards, field apps, and MVPs, those trade-offs almost never bite.

Author the service worker and manifest directly in the Next.js app. Do not pull in a PWA wrapper package — they reliably break across Next.js upgrades.

## Scaffold workflow

1. Start from the `web-app` scaffold: `npx create-next-app@latest <project> --typescript --tailwind --app`. Verify it boots.
2. Add `/public/manifest.webmanifest` with `name`, `short_name`, `start_url`, `display: standalone`, `icons` (at minimum 192px and 512px).
3. Reference the manifest from the root layout: `<link rel="manifest" href="/manifest.webmanifest">`.
4. Author `/public/sw.js` with minimal cache-first strategy for app shell, network-first for API. Register it in a small `'use client'` component on the root layout.
5. Mobile-first styling — start at 375px and work up. Tailwind's default breakpoints work; avoid desktop-first.
6. Open the dev URL on a real phone (same Wi-Fi), confirm "Add to Home Screen" appears, then test airplane-mode behavior.

## First prompt to hand the user

```
I'm building [describe app]. It needs to work well on phones but does
NOT need the App Store.

Stack: Next.js 16 App Router, TypeScript, Tailwind CSS, PWA (service
worker + web manifest), deploy to Vercel.

Apply these stack-specific constraints:

- Mobile-first responsive design — start with phone layout at 375px.
- Implement service worker and web app manifest directly — no PWA
  wrapper packages (they break on Next.js upgrades).
- Warn me proactively about iOS PWA limitations (push notifications,
  background sync, storage limits).
- Test installability on both iOS Safari and Android Chrome.
- Every response ends with a verification step I can run on my phone.

Start by scaffolding the project with PWA support and showing me
how to install it on my phone.
```

## Constraints (enforce on every response)

- **Mobile-first at 375px.** Desktop is a progressive enhancement, not the default.
- **Service worker + manifest authored directly** — no PWA wrapper packages.
- **Proactively call out iOS PWA limits** when the user mentions push, background sync, large storage, or Bluetooth.
- **Test on both iOS Safari and Android Chrome** before declaring a feature done — install behavior differs.
- **Every response ends with a phone-testable verification step.**
- All the universal web-app rules apply too: server components by default, no state libraries early, files under 300 lines, etc.

## Verification checklist

```bash
npm run dev   # open on phone browser (same Wi-Fi network)
# Phone: "Add to Home Screen" should appear (Safari Share / Chrome menu)
# Phone: airplane mode after first load — core features still work
# Chrome DevTools → Lighthouse → PWA score ≥ 90
```

## Stop signals

- The user needs iOS push notifications.
- The user needs Bluetooth, NFC, or other hardware APIs.
- Stakeholders insist on "a real app in the store."
- Offline requirements get complex: large datasets, conflict resolution, background sync.

When any of these fire, surface the trade-off and offer the `mobile-app-store` path.

## Escape route

If the product genuinely needs the store later, the React + TypeScript + Next.js skills transfer directly to Expo/React Native. You'll rewrite the UI layer, but the mental model and backend stay.

## Handing off to other skills

- Backend API → `api-backend` skill.
- Tricky data/auth work that needs structured planning → `sdd` skill.
- Agent going sideways → `red-flags` skill.
