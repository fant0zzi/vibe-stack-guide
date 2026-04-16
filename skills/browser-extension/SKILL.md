---
name: browser-extension
description: Scaffold and iterate on a vibe-coded browser extension (Chrome, Firefox, or cross-browser) using TypeScript + Chrome Manifest V3 + WXT (or esbuild) + plain HTML / Tailwind for the popup. Use this skill whenever the user says "extension", "browser extension", "Chrome extension", "Firefox add-on", "manifest v3", "MV3", "content script", "service worker extension", or describes functionality that lives inside the browser chrome. Apply it for the full lifecycle — scaffold, content-script / background wiring, Chrome dev-mode testing, packaging for the store — not just day one.
---

# Browser Extension — Vibe Coding Path

This skill is for helping an experienced developer ship a browser extension, mostly with an AI coding agent. Extensions feel simple but have specific failure modes — agents often generate broken messaging patterns between content scripts and the service worker. The playbook here keeps the stack minimal to shrink the agent's surface area for mistakes.

**Default recommendation, no hedging:** TypeScript + Chrome Manifest V3 + [WXT](https://wxt.dev) (built on Vite, sensible MV3 defaults) — or esbuild if the user wants fully manual control. Plain HTML + CSS, or Tailwind, for the popup.

## Why this stack (use only when asked)

MV3 is required on Chrome; the same manifest usually works on Firefox with minor tweaks. Chrome Web Store review is straightforward if permissions stay narrow. Keeping the popup framework-free means fewer things the agent can get wrong.

Don't pull React, Vue, or an SPA framework into the popup unless the UI has 10+ interactive elements — a framework brings build tooling, bundler config, and abstraction layers that multiply the debugging surface. Avoid webpack — the config overhead is exactly what MV3 newcomers get stuck on.

## Scaffold workflow

1. `npx wxt@latest init <project>` and pick the TypeScript template. (Or `npm init -y` + `npm install -D esbuild typescript` and write the manifest by hand — only go this route if the user explicitly wants to avoid frameworks.)
2. Open `wxt.config.ts` / `manifest.json` — trim permissions to the smallest set the extension actually needs. Avoid `"<all_urls>"` unless required.
3. Build one surface first — content script OR background OR popup — and verify it in Chrome developer mode before adding the next.
4. Load unpacked: `chrome://extensions` → Developer mode → Load unpacked → select the build output.
5. Test on the target site(s) at each step. Reload the extension after every change (WXT auto-reloads; manual setups don't).
6. Package for the store only when the core feature works end-to-end.

## First prompt to hand the user

```
I'm building a browser extension that [describe what it does in one
sentence].

Stack: TypeScript, WXT (or esbuild), Chrome Manifest V3. Plain HTML +
Tailwind for popup. No React or Vue in popup.

Apply these stack-specific constraints:

- Minimal manifest.json — request only the permissions needed right now.
  No "all_urls" unless genuinely required.
- Content scripts and background service worker in separate files.
- No message passing between content script and background until the
  extension genuinely needs cross-context communication.
- No React/Vue in popup unless the UI has 10+ interactive elements.
- Keep popup under 1 file unless it exceeds 200 lines.
- Every response includes steps to test in Chrome developer mode.

Start by creating the manifest, one content script or background
worker (whichever my extension needs), and show me how to load it
in Chrome developer mode.
```

## Constraints (enforce on every response)

- **Minimal manifest permissions.** Add only what the current feature needs; no speculative `host_permissions` or `"<all_urls>"`.
- **Separate files for content script, background service worker, and popup.** Don't collapse them.
- **No message passing between content script and background** until the extension genuinely needs cross-context communication. Many extensions don't.
- **No React / Vue in the popup** unless 10+ interactive elements exist.
- **Popup = one file** unless it's already > 200 lines.
- **Every response includes steps to test in Chrome developer mode** — reload extension, navigate to test page, expected observation.

## Verification checklist

```bash
# With WXT
npm run dev                          # builds + autoreloads
# Or with esbuild:
npx esbuild src/background.ts --bundle --outfile=dist/background.js

# chrome://extensions → Developer mode → Load unpacked → dist/
# Navigate to the target site → verify the content script fires
# Click the extension icon → popup renders as expected
```

## Stop signals

- Webpack config longer than ~10 lines.
- "Service worker inactive" errors the user can't debug.
- Extension asks for permissions it doesn't need (e.g., `"<all_urls>"`).
- Agent installs React in the popup for three buttons.
- Chrome Web Store rejects the extension for policy reasons the user can't parse.

## Escape routes

- **Extension genuinely needs a rich UI:** move the heavy logic into a companion web app (Next.js) and keep the extension thin — content script + popup that links to the web app. This is how most successful extensions are structured.
- **Cross-browser headache:** start Chrome-first. Firefox parity is usually a minor manifest tweak, not a rewrite.
- **Store policy rejection:** specialist-hours problem (privacy policy, asset requirements), not a code rewrite.

## Handing off to other skills

- Companion web app for heavy UI → `web-app`.
- Backend endpoints the extension calls → `api-backend`.
- Something clearly going wrong → `red-flags`.
