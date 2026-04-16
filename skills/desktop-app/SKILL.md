---
name: desktop-app
description: Scaffold and iterate on a vibe-coded native desktop app (Windows, macOS, Linux) using Tauri v2 + Vite + React + TypeScript with a minimal Rust backend for command handlers only. Use this skill whenever the user says "desktop app", "Tauri", "Electron", "native app", "system tray", "local filesystem access", "offline-first desktop", or describes functionality that must run outside a browser tab. Apply it for the full lifecycle — pre-flight sanity check, scaffold, browser-first verification, Tauri wrap, packaging — not just day one. Before scaffolding, confirm desktop is actually required — if not, route to `web-app` or `mobile-pwa`.
---

# Desktop App — Vibe Coding Path

This skill is for helping an experienced developer ship a native desktop app, mostly with an AI coding agent. Before anything else: desktop is the trickiest vibe-coding category because of code signing, notarization, and platform-specific distribution that agents can't automate. Make sure the user actually needs desktop.

**Pre-flight question (ask first, always):**

> Why does this have to be a desktop app and not a web app or PWA? Pick one: (a) needs local filesystem access, (b) needs system tray / menu-bar presence, (c) needs offline-first behavior that PWA can't handle, (d) integrates with native OS features (global shortcuts, notifications, etc.), (e) other.

If the answer is "works offline" or "feels like an app," a PWA (`mobile-pwa` skill) almost always beats Tauri for time-to-ship. Only if the answer is (a)–(d) should you proceed with this skill.

**Default recommendation, no hedging:** Tauri v2 + Vite + React + TypeScript. Rust only inside Tauri command handlers — everything else stays in TypeScript.

## Why this stack (use only when asked)

[Minimal Tauri apps can be under 600KB](https://v2.tauri.app/start/) because they use the system webview. Electron bundles Chromium + Node.js, which is a meaningful size and memory penalty. The Tauri frontend is standard web tech — the place where agents are most reliable — and you get a two-stage verification loop: verify in the browser first (normal dev tools), then in the desktop shell.

**Use Vite + React, not Next.js.** Next.js is an SSR framework; its server components, routing conventions, and API routes behave unexpectedly inside a desktop shell, and the agent will struggle to debug the mismatch.

Don't default to Electron unless Node.js APIs are specifically required. Don't use native frameworks (SwiftUI, WPF, GTK) — agent fluency is much lower and you lose cross-platform.

## Scaffold workflow

1. `npm create vite@latest <project> -- --template react-ts`, `cd <project>`, `npm install`, `npm run dev`. **Verify the app works in a browser first.** This is non-negotiable.
2. `npm install --save-dev @tauri-apps/cli`, then `npx tauri init` and accept sane defaults.
3. `npx tauri dev` — verify the same UI renders inside a native window.
4. Add native features via `invoke` from the frontend to Rust command handlers. Keep handlers as small as possible; no business logic in Rust.
5. `npx tauri build` when there's a feature worth packaging.
6. Defer code signing / notarization until there's a real need to distribute. When it arrives, this is usually the point where a Rust/Tauri specialist adds the most value.

## First prompt to hand the user

```
I'm building a desktop app that [describe what it does in one sentence].
It must be a desktop app because [reason — filesystem access, system
tray, offline-first, etc.].

Stack: Tauri v2, Vite + React + TypeScript frontend. Minimal Rust
backend — Tauri command handlers only.

Apply these stack-specific constraints:

- Build the web UI first and verify it works in a browser BEFORE
  adding Tauri. The React frontend is the primary codebase.
- No Next.js inside Tauri — use Vite + React. SSR frameworks behave
  unexpectedly in desktop shells.
- Minimize Rust — only Tauri command handlers for features that need
  native access. Everything else in TypeScript.
- No custom window chrome or frameless windows until core features work.
- Every response includes verification for both browser and desktop.

Start by creating the Vite React app, verify it works in a browser,
then wrap it with Tauri.
```

## Constraints (enforce on every response)

- **Browser-first verification.** The web UI must work in `npm run dev` before Tauri is wrapped around it.
- **No Next.js inside Tauri.** Vite + React only.
- **Minimal Rust surface.** Tauri command handlers only for features that need native access; everything else stays in TypeScript.
- **No frameless windows, no custom chrome** until the core features work inside the default window.
- **Every response includes verification for both browser and desktop** — `npm run dev` for one, `npx tauri dev` for the other.
- **Files under 300 lines**, same as the other paths.

## Verification checklist

```bash
npm run dev              # works in a browser at http://localhost:5173
npx tauri dev            # native window opens with the same UI inside
# Manually test the native features that justified desktop in the first place.
npx tauri build          # produces an installer for the current platform
```

Before packaging, confirm the user has a plan for distribution — signing certs, store listings, or a simple "download the .dmg / .exe" channel. If they don't, stop and talk about it.

## Stop signals

- More than 2 hours spent on Rust compilation errors.
- Code signing or notarization is blocking a first test distribution.
- Agent generates Rust the user can't read and the app crashes.
- User realizes the real requirement was "works offline" — route to `mobile-pwa`.
- Agent proposes IPC patterns the user doesn't understand.

## Escape routes

- **Desktop friction is too high:** ship as a web app first (`web-app` skill). The React code transfers directly; just remove the Tauri layer. Add Tauri back when there's proven demand for desktop-only features.
- **Signing / notarization stuck:** hire a Rust/Tauri specialist for packaging only. The app code stays.
- **Agent struggles with Rust commands:** rewrite the feature in TypeScript using Tauri's built-in JS APIs where possible (`@tauri-apps/plugin-fs`, `@tauri-apps/plugin-shell`, etc.). Keep custom Rust commands as a last resort.

## Handing off to other skills

- Backing API for the desktop app → `api-backend`.
- Larger features that deserve a spec → `sdd`.
- Something going in circles → `red-flags`.
