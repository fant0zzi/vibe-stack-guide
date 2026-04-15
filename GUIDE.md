# Vibe Coding Stack Guide

*For experienced developers building outside their stack with AI agents.*

---

## How to use this guide

You know how to build software — just not in the stack you need right now. You're about to open Cursor, Claude Code, or another AI coding agent and let it write most of the code. This guide helps you pick the right stack **before** you start, so the project doesn't die at environment setup, deployment, or state management — the stages where vibe-coded projects actually fail.

**Three ways to use this:**

1. **Read your path below** (2 minutes), then start building.
2. **Paste the companion file** ([AGENT_CONTEXT.md](./AGENT_CONTEXT.md)) into your agent's context window and say: *"I want to build [X]."* The agent will use the guide's decision criteria to pick your stack and give you a starting plan.
3. **Drop the companion file into your project root** as `CLAUDE.md`, `.cursorrules`, or `AGENTS.md` (depending on your tool). The agent will use it automatically on every conversation — no pasting needed.

**One rule:** every path recommends **one** stack, not three. The recommendation optimizes for the shortest distance between "agent wrote code" and "I can verify and ship it" — not for architectural elegance or long-term scalability.

---

## Pick your path

| # | I want to build... | Go to |
|---|---|---|
| 1 | **Web app** — SaaS, dashboard, internal tool, or authenticated web UI | [Web App](#path-web-app) |
| 2 | **Mobile app that needs the App Store** — iOS, Android, or both | [Mobile — App Store](#path-mobile-app--app-store) |
| 3 | **Mobile app that doesn't need the App Store** — internal tool, field app, MVP | [Mobile — No App Store](#path-mobile-app--no-app-store-pwa) |
| 4 | **API or backend service** | [API / Backend](#path-api--backend-service) |
| 5 | **CLI tool, automation script, or data pipeline** | [CLI / Automation](#path-cli--automation--data-pipeline) |
| 6 | **Browser extension** | [Browser Extension](#path-browser-extension) |
| 7 | **Desktop app** — Windows, macOS, or Linux via Tauri | [Desktop App](#path-desktop-app) |
| 8 | **AI agent** — chatbot, RAG app, tool-using agent, agentic workflow | [AI Agent](#path-ai-agent) |

> **Just building a landing page?** Follow the [Web App](#path-web-app) path but skip the backend. You don't need Supabase or auth — just Next.js + Vercel.

---

## Path: Web App

**SaaS, dashboard, internal tool, authenticated web UI, or landing page.**

### Stack

```
Next.js 16 (App Router) + TypeScript + Tailwind + Vercel + Supabase (auth + Postgres)
```

### Why this, not that

TypeScript scores poorly on repo-level agent benchmarks (Multi-SWE-bench puts it near the bottom). But Next.js compensates with something no other framework has: **measured, published agent evals** — [Claude Opus 4.6 / Claude Code goes from 75% to 100% with AGENTS.md, Cursor Composer 2.0 from 75% to 96%](https://nextjs.org/evals). That means fewer broken iterations for you to review. The verification loop is fast (`npm run build` + `tsc --noEmit` + one-click Vercel deploy), and the community is the largest escape hatch if the agent gets stuck on an edge case.

**Don't use** Remix or Astro for your first vibe-coded full-stack app unless you already know them — [neither publishes agent evals](https://nextjs.org/evals), so you're betting on unmeasured agent reliability. SvelteKit is adding AI tooling (MCP server, AI docs, `llms.txt`) but still has no comparable evals. Don't invent your own auth or billing — use Supabase Auth and Stripe's hosted checkout.

### First prompt

```
I'm building [describe your app in one sentence].

Stack: Next.js 16 App Router, TypeScript, Tailwind CSS, Supabase
(auth + Postgres), deploy to Vercel.

I can evaluate architectural tradeoffs but need implementation guidance
for the React/Next.js ecosystem. Apply these stack-specific constraints:

- Server components by default. Use 'use client' only for components
  that need browser APIs (onClick, useState, useEffect, clipboard).
- Static generation unless I specify otherwise — no getServerSideProps,
  no runtime data fetching without justification.
- All data access in server components or API routes, never in client
  components.
- No state management libraries — useState and server components only
  until I have 5+ interactive pages with shared state.
- No additional dependencies without a one-sentence justification.
- Every response ends with a verification step: a command to run or
  a test to execute.

Start by scaffolding the project and showing me how to run it locally.
```

### Verify it works

```bash
# 1. Does it start?
npm run dev
# → should see "Ready" + localhost URL

# 2. Does it build for production?
npm run build
# → should complete with no errors

# 3. Does it type-check?
npx tsc --noEmit
# → should return with no errors

# 4. Does deploy work?
# → connect your GitHub repo at vercel.com — first deploy is 3 clicks
# → alternative: npx vercel (requires Vercel CLI login)
# → either way, you should get a public URL you can open in a browser
```

### Stop signals

- The agent adds Redux, Zustand, or Jotai to a project with fewer than 5 interactive pages
- You have more than 3 environment variables you don't understand
- The agent creates a custom auth flow instead of using Supabase Auth
- You can't explain what any given API route does in one sentence
- Build errors reference webpack/turbopack config you've never seen

### Escape route

If the full-stack app gets messy, **keep the frontend in Next.js** and move backend logic into a separate FastAPI service you understand better. That simplifies the frontend to "just renders data" and lets you own the backend in your home language. If even the frontend is too much, fall back to a static site with Supabase as the entire backend (auth + DB + edge functions).

---

## Path: Mobile App — App Store

**iOS, Android, or both. Must ship through Apple App Store or Google Play.**

### Stack

```
Expo (managed workflow) + React Native + TypeScript + Expo Router + EAS Build + EAS Submit
```

### Why this, not that

Expo is the only mobile framework that has invested heavily in AI-agent support: [`create-expo-app` generates AGENTS.md by default](https://docs.expo.dev/more/create-expo/), docs are SDK-version-matched, there's an official MCP server, and Replit's own mobile publishing flow is built on Expo. The verification loop is instant — scan a QR code with Expo Go and see the result on your phone. EAS Build + Submit handles the deployment friction of signing, provisioning, and store submission that agents can't automate.

**Don't use** SwiftUI unless the app is Apple-only *and* you need native Apple APIs (HealthKit, ARKit). Don't use Kotlin Multiplatform — the toolchain is complex and agent support is thinner than Expo. Don't use bare React Native (without Expo) — you'll spend days on native module linking that Expo handles automatically. Flutter is improving its AI tooling (MCP server, AI docs), but you're betting on a toolchain without measured agent reliability — no public evals, experimental MCP server.

The complexity Expo eliminates: native module linking, Xcode/Gradle build configuration, signing/provisioning profiles, and store submission mechanics. These are the tasks that derail vibe-coded mobile projects — they require platform-specific expertise agents don't have, and errors are opaque without that expertise. Expo's managed workflow keeps all of that behind a single `eas build` command.

### First prompt

```
I'm building [describe your app in one sentence]. Target iOS and Android.

Stack: Expo (managed workflow), React Native, TypeScript, Expo Router,
EAS Build + Submit.

I can evaluate architectural tradeoffs but need implementation guidance
for the React Native/Expo ecosystem. Apply these stack-specific constraints:

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

### Verify it works

```bash
# 1. Does it start?
npx expo start
# → scan QR code with Expo Go on your phone

# 2. Does it build for both platforms?
eas build --platform all --profile preview
# → should complete and give you installable builds

# 3. Does TypeScript check pass?
npx tsc --noEmit

# 4. Does it pass basic navigation test?
# → manually tap through every screen. If any screen crashes, stop.
```

### Stop signals

- The agent suggests "ejecting" from Expo managed workflow
- You need a native module that isn't in the Expo SDK
- The agent creates more than 2 levels of nested navigation
- You can't run the app on your physical device via Expo Go
- Build times exceed 15 minutes and you don't know why

### Escape route

If app-store friction becomes the blocker (signing, provisioning, review), ask yourself: **does this really need to be in the store?** If the answer is "for distribution only" (not for native APIs), switch to the PWA path — it ships in minutes instead of days. If you genuinely need the store, hire a mobile specialist for the final signing/submission leg instead of rewriting the app. The code stays the same; the bureaucracy is what you're outsourcing.

---

## Path: Mobile App — No App Store (PWA)

**Internal tool, field app, MVP, or anything where App Store distribution isn't required.**

### Stack

```
Next.js 16 (App Router) + TypeScript + Tailwind + PWA setup (service worker + web manifest) + Vercel
```

### Why this, not that

A Progressive Web App is installable, works offline, and deploys like a website. No signing, no provisioning profiles, no store review — you skip the entire category of deployment problems that agents can't automate. You get the same Next.js agent support as the Web App path (published evals, AGENTS.md, version-matched docs), and Lighthouse gives you a concrete verification tool for PWA-specific behavior. The trade-off: no push notifications on iOS (limited), no access to some hardware APIs, and some users expect "a real app" in the store. But for internal tools, dashboards, field apps, and MVPs, none of that matters. For PWA setup, ask your agent to add a service worker and web app manifest — avoid locking into a specific PWA wrapper package, as they frequently go unmaintained or break on new Next.js versions.

**Don't use** native mobile frameworks for an app that doesn't need to be in the store — you're adding signing, provisioning, and store review friction for no benefit. "Mobile" usually means "good phone UX," not "needs provisioning profiles."

### First prompt

```
I'm building [describe your app in one sentence]. It needs to work well
on phones but does NOT need the App Store.

Stack: Next.js 16 App Router, TypeScript, Tailwind CSS, PWA (service
worker + web manifest), deploy to Vercel.

I can evaluate architectural tradeoffs but need implementation guidance
for PWA setup within Next.js. Apply these stack-specific constraints:

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

### Verify it works

```bash
# 1. Does it run locally?
npm run dev
# → open on your phone's browser (same WiFi network)

# 2. Is it installable?
# → in your phone browser: "Add to Home Screen" should appear

# 3. Does it work offline?
# → turn on airplane mode after loading, core features should still work

# 4. Does Lighthouse like it?
# → Chrome DevTools → Lighthouse → check PWA score
```

### Stop signals

- You need push notifications on iOS (PWA support is still limited)
- You need Bluetooth, NFC, or other hardware APIs
- Users or stakeholders insist on "a real app in the store"
- Offline requirements are complex (large datasets, conflict resolution)

### Escape route

If you discover you do need the App Store, your Next.js code and knowledge aren't wasted — the React and TypeScript skills transfer directly to Expo/React Native. You'll rewrite the UI layer but keep the mental model and backend.

---

## Path: API / Backend Service

**REST or GraphQL API, microservice, webhook handler, or any server-side logic.**

### Stack

```
FastAPI + Pydantic + uv + ruff + mypy + pytest
```

### Why this, not that

Python dominates every public agent benchmark for repo-level tasks. On [Multi-SWE-bench](https://arxiv.org/html/2504.02605), the best agent setup resolves 52% of Python tasks vs. 22% for Java, with JavaScript and TypeScript both in single digits. FastAPI has the smallest surface area of the major Python frameworks: one file can be a complete working API, type hints double as validation (via Pydantic), and the docs are auto-generated. The verification loop is fast — pytest runs in seconds, not minutes — and the output is readable even if you're not a Python expert.

**Don't use** Django unless you specifically want its admin panel and ORM batteries. Django adds conceptual surface area (settings.py, apps, migrations, middleware) that an agent will generate but you won't understand — more code you can't review means more risk. Don't split into microservices — start as one FastAPI app and extract later if needed. Don't use Node/Express for a backend-heavy project if you're not a JS developer — you're giving up the highest agent reliability *and* taking on a stack where you can't verify the output.

### First prompt

```
I'm building an API that [describe what it does in one sentence].

Stack: FastAPI, Pydantic, uv, ruff, mypy, pytest. SQLite for dev,
Postgres for prod. Alembic for migrations.

I can evaluate architectural tradeoffs but need implementation guidance
for FastAPI conventions. Apply these stack-specific constraints:

- One endpoint at a time. Write a pytest test for each endpoint before
  moving to the next.
- Every endpoint gets a Pydantic request and response model — no raw
  dicts in signatures.
- No abstract base classes, no service layers until I have 5+ endpoints.
  One file can be a complete working API.
- async def by default, but no async patterns (background tasks,
  websockets, event streams) until I need them.
- No microservices — one FastAPI app.
- Every response includes a curl command to test the endpoint.

Start by creating the project with uv init, adding dependencies with
uv add, and building one health-check endpoint with one test.
```

### Verify it works

```bash
# 1. Does it start?
uv run uvicorn main:app --reload
# → should show "Uvicorn running on http://127.0.0.1:8000"

# 2. Do the auto-docs work?
# → open http://127.0.0.1:8000/docs — interactive Swagger UI

# 3. Do tests pass?
uv run pytest -v
# → all tests should pass

# 4. Do types check?
uv run mypy .
# → should pass (or show only minor issues)

# 5. Does it lint/format?
uv run ruff check .
uv run ruff format --check .
# → should pass with no issues
```

### Stop signals

- The agent creates more than 3 separate "service" layers before you have 5 endpoints
- You have database migration files you can't read
- The agent introduces Celery, Redis, or a message queue before you have 10 users
- Error messages reference SQLAlchemy internals you don't understand
- The agent generates abstract base classes for "future extensibility"

### Escape route

If the API outgrows what you can manage, the cleanest move is to hire a Python backend specialist who can read your FastAPI code directly — it's the most readable Python web framework. If you need to scale horizontally, move the heavy work behind a task queue (but only when you actually need it, not as a precaution).

---

## Path: CLI / Automation / Data Pipeline

**Command-line tools, scripts, cron jobs, scrapers, data transforms.**

### Stack

```
Python 3.13+ + uv + ruff + mypy + pytest
```

### Why this, not that

This is the highest-probability category for vibe coding. Python resolves [52% of tasks on Multi-SWE-bench](https://arxiv.org/html/2504.02605) vs. 22% for Java and single digits for JS/TS — no other language comes close for agent-generated code you need to review and trust. The feedback loop is instant (run the script, see the output), deployment is trivial (uv tool install or just copy the file), and Python errors are readable even if you're not a Python expert. [Anthropic's own usage data](https://www.anthropic.com/research/impact-software-development) shows automation is the #1 category for AI-assisted coding.

**Don't use** Go or Rust unless you specifically need static binaries or high performance — you're trading the highest agent fluency for language benefits most first versions don't need, and Rust compiler errors are not readable without Rust expertise. Don't start with Airflow, Dagster, or any orchestration framework — start with a script that works, then add orchestration when the scheduling problem is real.

The complexity a plain Python script eliminates: build systems, type system gymnastics, dependency resolution headaches, and orchestration configuration. A Python script is one file that runs — no compilation step, no container, no DAG definition. That means the agent can generate it, you can read it, and `uv run python main.py` proves it works. Every layer you add is a layer the agent can get wrong and you'll struggle to debug.

### First prompt

```
I'm building a [CLI tool / automation script / data pipeline] that
[describe what it does in one sentence].

Stack: Python 3.13+, uv, ruff, mypy, pytest. Typer for CLI argument
parsing (if CLI). pandas for tabular data, requests for HTTP.

I can evaluate architectural tradeoffs but need guidance on Python
CLI/scripting conventions. Apply these stack-specific constraints:

- One function that does the core thing, called from __main__. The
  script must be runnable with `uv run python main.py` — no setup
  beyond uv sync.
- stderr for progress, stdout for results. Use the standard logging
  module from the start.
- No classes unless the data model genuinely requires it.
- No orchestration frameworks (Airflow, Dagster, Prefect) — just a
  script that works. Add orchestration when scheduling is a real problem.
- Every response includes a command to run and verify the output.

Start by creating the project with uv init, adding dependencies with
uv add, and building one core function with one test.
```

### Verify it works

```bash
# 1. Does it run?
uv run python main.py --help
# → should show usage information

# 2. Does the core function work?
uv run python main.py [your args]
# → should produce expected output

# 3. Do tests pass?
uv run pytest -v

# 4. Does it lint/format?
uv run ruff check .
uv run ruff format --check .

# 5. Can someone else install it?
uv pip install -e .
# → should install without errors
```

### Stop signals

- You have more than 5 files and no tests
- The agent adds an ORM for what should be a simple CSV transform
- Your script has more imports than lines of logic
- The agent suggests Docker for a script that processes local files
- Error handling is more complex than the happy path

### Escape route

CLI and automation scripts are the easiest projects to hand off — the code is linear, the inputs and outputs are clear, and any Python developer can pick it up. If it outgrows a script, the natural evolution is to add a simple web UI (FastAPI + one page) or move to a proper pipeline framework, but only when you hit the actual scaling wall.

---

## Path: Browser Extension

**Chrome extension, Firefox add-on, or cross-browser extension.**

### Stack

```
TypeScript + Chrome Manifest V3 + esbuild or WXT + plain HTML/CSS or Tailwind
```

### Why this, not that

Browser extensions are a deceptively simple category — the code is usually small, but agents frequently generate broken messaging patterns between content scripts and the background service worker. It's the #1 source of extension bugs for newcomers, and the errors are hard to debug without understanding the MV3 service worker lifecycle. Keeping the stack minimal means fewer things the agent can get wrong. MV3 is required for Chrome (MV2 is deprecated), and the same manifest works in Firefox with minor adjustments. Chrome Web Store review is straightforward for minimal extensions — the deployment friction is low if you keep permissions narrow.

**Don't use** React, Vue, or any SPA framework for the popup/options page unless the UI is genuinely complex (10+ interactive elements) — a framework adds build tooling, bundle config, and abstraction layers that multiply the debugging surface for code you didn't write. For build tooling, [WXT](https://wxt.dev) (built on Vite, provides sensible MV3 defaults) is the strongest current option; esbuild also works for fully manual control. Avoid message passing between content scripts and the background service worker until your extension genuinely needs cross-context communication.

### First prompt

```
I'm building a browser extension that [describe what it does in one
sentence].

Stack: TypeScript, esbuild (or WXT), Chrome Manifest V3. Plain
HTML + Tailwind for popup. No React or Vue in popup.

I can evaluate architectural tradeoffs but need implementation guidance
for extension APIs and MV3 conventions. Apply these stack-specific
constraints:

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

### Verify it works

```bash
# 1. Does it compile?
npx esbuild src/background.ts --bundle --outfile=dist/background.js
# → should produce output with no errors

# 2. Does it load in Chrome?
# → chrome://extensions → Developer mode → Load unpacked → select dist/
# → should appear with no errors

# 3. Does the core feature work?
# → navigate to a test page → verify the extension does its thing

# 4. Does the popup open?
# → click the extension icon → popup should render correctly
```

### Stop signals

- The agent adds webpack config more than 10 lines long
- You get "service worker inactive" errors you can't understand
- The extension requests permissions it doesn't need (like "all_urls")
- The agent creates a React app inside the popup for 3 buttons
- Chrome Web Store rejects it for policy reasons you can't parse

### Escape route

If the extension gets complex enough to need a real UI framework, move the complex logic to a **companion web app** (Next.js) and keep the extension thin — just the content script and a popup that links to the web app. This is a common pattern for successful extensions and it lets you use the Web App path for the complex parts.

---

## Path: Desktop App

**Needs to run as a native desktop application (not just a browser tab).**

### Stack

```
Tauri (v2) + Vite + React + TypeScript
```

### Why this, not that

Desktop is a trap category for vibe coding. It feels like "web app with a wrapper," but shipping introduces code signing and platform-specific distribution that agents can't automate. **If your app could be a browser tab, make it a web app instead.** If you genuinely need local filesystem access, system tray, or native OS integration, Tauri is lighter than Electron — [minimal Tauri apps can be under 600KB](https://v2.tauri.app/start/) since they use the system webview, while Electron bundles Chromium and Node.js — a meaningful size and memory difference. The frontend is standard web tech that agents generate well, and you get a two-stage verification loop: verify in the browser first (standard dev tools), then in the desktop shell. Use Vite + React, not Next.js — Next.js is an SSR framework and its server components, routing, and API routes behave unexpectedly inside a desktop shell that agents struggle to debug.

**Don't use** Electron unless you need Node.js APIs specifically — the agent will generate code that works in dev but has packaging issues in production. Don't use Next.js inside Tauri — the SSR/static export mismatch creates subtle bugs the agent will struggle with. Don't use native desktop frameworks (SwiftUI for Mac, WPF for Windows, GTK) — agent fluency is much lower and you lose cross-platform. Don't start with desktop if you haven't validated the idea as a web app first.

### First prompt

```
I'm building a desktop app that [describe what it does in one sentence].
It must be a desktop app because [reason — filesystem access, system
tray, offline-first, etc.].

Stack: Tauri v2, Vite + React + TypeScript frontend. Minimal Rust
backend — Tauri command handlers only.

I can evaluate architectural tradeoffs but need implementation guidance
for Tauri/Rust integration. Apply these stack-specific constraints:

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

### Verify it works

```bash
# 1. Does the web UI work standalone?
npm run dev
# → should work normally in a browser at localhost:5173

# 2. Does Tauri build?
cargo tauri dev
# → should open a native window with the web UI inside

# 3. Do native features work?
# → test filesystem access, system tray, or whatever required desktop

# 4. Does it package?
cargo tauri build
# → should produce an installer for your platform
```

### Stop signals

- You spend more than 2 hours on Rust compilation errors
- Code signing or notarization blocks your first test distribution
- The agent generates Rust code you can't read and the app crashes
- You realize the "desktop requirement" was actually just "works offline" (use PWA instead)
- The agent suggests inter-process communication patterns you don't understand

### Escape route

If desktop is too much friction, **ship as a web app first** and come back to the desktop wrapper later. The React frontend code transfers directly — you're just removing the Tauri layer. If you've shipped as a web app and proven demand exists, hire a Rust/Tauri specialist for the packaging and distribution leg.

---

## Path: AI Agent

**Chatbot, RAG app, tool-using agent, or agentic workflow.**

### Stack

```
Pydantic AI + Python 3.13+ + uv + ruff + mypy + pytest + Logfire (observability)
```

### Why this, not that

Pydantic AI has the smallest surface area of the serious agent frameworks: typed agents, dependency injection, validated structured outputs, and built-in testing primitives. It hit v1 in September 2025 with an explicit stability policy (no intentional breaking changes until v2), so the code the agent generates today won't break on the next release.

The testing story is the differentiator for vibe coding. `TestModel` and `FunctionModel` let you fake the LLM in pytest — your verification loop is fast and deterministic, and you're not paying for API calls to test tool logic. `Agent.override` injects test dependencies, and you can globally block real model calls in CI. That means every "verify it works" step actually runs, instead of being a manual "try it and see" against a live API.

Logfire is two lines of setup, built on OpenTelemetry so you're not locked in.

**Don't use** LangChain or LangGraph for your first vibe-coded agent — the conceptual surface area is larger (chains, runnables, graph state, checkpointers), both have multi-page migration guides between major versions, and public debugging friction is well-documented. LangGraph is a better fit for teams that specifically need its orchestration depth than for teams optimizing for fast AI-assisted iteration. **Don't use** CrewAI or AutoGen as your default — both introduce bespoke vocabulary (crews/flows, AgentChat/autogen-core) that gives coding agents more ways to generate the wrong shape of solution. smolagents is the closest minimalist alternative but is more experimental than production-default. **Don't use** LangSmith for observability unless you're already in the LangChain ecosystem — it adds lock-in without stack benefit.

MCP: optional. Add MCP when you need external tool interoperability — Pydantic AI has first-class support via `pydantic-ai[mcp]`. Don't set up an MCP server before you have a real interoperability need.

Deploy to Railway (`railway init` → `railway up`) for the simplest local-to-production path.

### First prompt

```
I'm building an AI agent that [describe what it does in one sentence].

Stack: Pydantic AI, Python 3.13+, uv, ruff, mypy, pytest, Logfire.
Deploy to Railway.

I can evaluate architectural tradeoffs but need implementation guidance
for building agents with Pydantic AI. Apply these constraints:

- One agent that does the core thing, with a typed Pydantic output model.
  No multi-agent orchestration until this single agent works end-to-end.
- Tool definitions as plain Python functions — not framework abstractions,
  not LangChain tools, not custom base classes.
- TestModel in every pytest file from the start. Every test must pass
  without a real API key. Block real model calls in CI.
- Logfire instrumentation from the first run — I want to see agent runs,
  tool calls, and latency from day one.
- No LangChain, no CrewAI, no framework-hopping. If Pydantic AI can't do
  something, tell me — don't silently switch frameworks.
- Every response ends with a verification step I can run.

Start by creating the project with uv, a single agent with one tool,
a test using TestModel, and Logfire wired up.
```

### Verify it works

```bash
# 1. Does it run?
uv run python main.py
# → should produce structured output (not raw LLM text)

# 2. Do tests pass without hitting the real API?
uv run pytest -v
# → all tests pass using TestModel (no API key needed)

# 3. Do types check?
uv run mypy .

# 4. Can you see traces?
# → open Logfire dashboard — should show agent runs, tool calls, latency

# 5. Does it deploy?
railway up
# → should give you a public URL
```

### Stop signals

- You have more than 3 LLM providers configured before you have 1 working use case
- The agent creates a graph or state-machine abstraction for a single-turn tool-call agent
- Tests require a real API key to pass (means TestModel isn't being used)
- You can't explain what the agent's tool functions do
- Token costs are climbing but you don't have traces showing why

### Escape route

If the agent logic is sound but Pydantic AI's edge cases bite you, the tool functions are plain Python — extract them and wire to any provider SDK directly (anthropic, openai, etc.). If you need heavy multi-agent orchestration that Pydantic AI doesn't cover, graduate to LangGraph with the understanding that you're trading simplicity for orchestration power. If observability is the bottleneck, Langfuse is the drop-in self-hosted alternative to Logfire.

---

## Universal Red Flags

These apply to every path. If you see any of these, stop building and reassess.

### The agent is going in circles

**What happens:** the agent can't fix a bug after 3 attempts, or each fix introduces a new bug. You're making zero progress per iteration.

**Why it's bad:** this means the agent doesn't have enough context to solve the problem, or the problem is outside its reliable range. More iterations won't help — they'll make the codebase worse.

**What to do:** stop, describe the bug in a new conversation with full context, and ask the agent to *diagnose* (not fix) the problem. If that doesn't clarify things, this is a signal to ask a human.

### The fix requires a new major dependency

**What happens:** the agent proposes adding Redux, a new ORM, a message queue, or a framework to fix what should be a localized bug.

**Why it's bad:** the agent is pattern-matching to "how this is solved in large production codebases," not "what's simplest for your situation." Each new dependency is code you don't understand and can't debug.

**What to do:** tell the agent: "Solve this without adding any new dependencies. If that's not possible, explain why." If it genuinely can't, you may have an architecture problem, not a code problem.

### More config than logic

**What happens:** the project has more configuration files (webpack, tsconfig, docker-compose, CI/CD, lint rules, env files) than files with actual application logic.

**Why it's bad:** config is the least agent-reliable and least debuggable layer. Every config file is a place where a subtle error can break everything with an incomprehensible error message.

**What to do:** delete configs that aren't strictly necessary. Use framework defaults. If the agent generates a config you don't understand, ask it to explain each line — and delete the lines neither of you can justify.

### You can't explain what the code does

**What happens:** the app works but you can't describe the data flow, the auth model, or what happens when a user does X. It's a black box that passes your manual smoke test.

**Why it's bad:** the first time it breaks in a way that isn't a syntax error, you'll be stuck. Vibe coding works for *generating* code, but *maintaining* code requires understanding it.

**What to do:** ask the agent to write a one-paragraph description of the architecture and data flow. If *that description* doesn't make sense to you, the project has grown beyond what vibe coding can sustain.

### The agent touches unrelated files

**What happens:** a bug fix in the payment flow modifies the navigation, the auth config, and a utility file you've never seen.

**Why it's bad:** the agent has lost the thread. It's making global changes to solve local problems, which means the codebase is becoming entangled in ways neither of you can track.

**What to do:** reject the change. Restate the problem with a smaller scope: "Fix ONLY the function X in file Y. Do not modify any other files."

### Deployment requires manual steps the agent can't do

**What happens:** you need to create certificates, configure signing identities, navigate a web console, or fill out store metadata — and the agent can only say "now do this manually."

**Why it's bad:** this is where vibe-coded projects die. The code is ready but you can't ship it because the last mile requires domain knowledge you don't have.

**What to do:** this is a legitimate reason to hire a specialist for a few hours — specifically for the deployment/signing/submission step, not for rewriting the app.

---

## How to Prompt for Vibe Coding

Vibe coding requires a different prompting style than expert-in-the-stack coding. Here's what works:

**Start with the simplest working slice, not the architecture.** Don't describe your ideal system architecture. Describe the one thing the user does, and ask the agent to make that work end-to-end. Architecture emerges from working code, not from planning documents.

**Pin your stack in the first message.** The agent will default to whatever's most popular in its training data unless you tell it otherwise. State your exact stack in the first prompt and add "do not suggest alternative frameworks."

**Ask for verification, not just code.** End every prompt with "give me a command to verify this works" or "write a test for this." Untested vibe code is how projects become unmaintainable.

**Constrain the agent explicitly.** "No new dependencies without explaining why." "No state management libraries." "No classes unless the data model requires it." Agents over-engineer by default because their training data is full of production codebases. Your job is to keep it simple.

**One feature per conversation turn.** Don't ask for the login page, the dashboard, and the settings page in one message. Each turn should produce something you can verify before moving on.

**When stuck, start a new conversation.** Long conversations degrade agent performance. If you're 20 messages deep and going in circles, open a new chat, paste your current code and the specific problem, and start fresh.

---

## Structuring Your Agent

Every path in this guide starts with a "first prompt" — you paste it, the agent scaffolds a project, and you start building. That works for the first session. But by session three or four, you'll notice a problem: the agent doesn't remember what you agreed on. It suggests a different state management library, restructures your files, or adds dependencies you already rejected. Every new conversation starts from zero.

The fix is a **rules file** — a markdown file in your project root that the agent reads automatically at the start of every session. Different tools call it different things: `CLAUDE.md` (Claude Code), `.cursorrules` (Cursor), `AGENTS.md` (Next.js convention), `copilot-instructions.md` (GitHub Copilot). The format doesn't matter. What matters is that your constraints survive between sessions.

### Level 1: Solo agent with a rules file

For MVPs and small projects, one rules file is all you need. Start with the [AGENT_CONTEXT.md](./AGENT_CONTEXT.md) companion file — it already has decision criteria and stack recommendations. Then add 5–10 project-specific rules as you go: which package manager to use, what files not to touch, what the acceptance criteria for "done" looks like, where env vars live.

This is the right level for most vibe-coded projects. It takes 10 minutes to set up and eliminates most session-to-session drift.

### Level 2: Multi-agent pipeline

When your project outgrows a solo agent, you can split the work across specialized roles: an architect that designs, a developer that implements, a QA agent that tests, and a reviewer that checks — each with different tools and constraints. No agent reviews its own work. No agent implements what it designed. This separation is the primary defense against the structural weaknesses of AI-generated code: intention drift, self-review bias, scope creep, and the illusion of correctness.

The companion file [PIPELINE.md](./PIPELINE.md) describes this multi-agent pipeline in detail — the four phases (design → test planning → implementation → validation), how agents hand off work, how revision loops are bounded, and how to configure it for your stack.

### When to move from Level 1 to Level 2

You don't need the pipeline until you feel the pain of not having it. Concrete signals:

- **The agent forgets context between sessions** — you keep re-explaining the same constraints. A rules file (Level 1) fixes this. If it keeps drifting despite the rules file, the project may need the structure of separated roles.
- **One session can't contain a change** — a feature touches enough files that you can't verify the full diff in a single conversation. Spec-driven development with task decomposition helps here.
- **You're afraid the agent will break working code** — the tests pass, but you don't trust the agent to make changes without regressing something. Independent QA (tests written before implementation, by a different agent) addresses this directly.
- **The agent goes in circles on reviews** — you ask it to check its own work and it says "looks good." Self-review is structurally unreliable. A read-only reviewer agent that physically cannot modify code produces trustworthy feedback.

Most projects never need Level 2. If yours does, you'll know — the symptoms are unmistakable.

---

## When to Stop Vibe Coding

Vibe coding is a tool for getting from zero to a working product. It's not a permanent development methodology. Here's when to transition:

**You have paying users or real dependencies.** Once people rely on your software, "I don't fully understand the code" becomes a liability. Hire a specialist to review, test, and harden what the agent built.

**The codebase exceeds your review capacity.** If you can't read the diff of the last change and understand what it does, the project has outgrown vibe coding. As a rough heuristic, this tends to happen around 50+ files or 5,000+ lines of AI-generated code — but the real threshold is comprehension, not line count. A simple CRUD app can stay manageable at 8K lines; a real-time system might hit the wall at 2K.

**You're spending more time debugging than building.** The value proposition of vibe coding is speed. If you're spending 80% of your time figuring out why the agent's code doesn't work, you've passed the point where AI assistance is a net positive.

**Security or compliance matters.** AI-generated code has known patterns of insecure defaults (hardcoded secrets, missing input validation, overly permissive CORS). If your app handles sensitive data, payments, or health information, get a security review before launch.

**You've validated the idea and want to scale.** Vibe coding is excellent for prototyping and MVPs. Once you know the idea works, the right move is usually to keep what works, rewrite what doesn't, and bring in people who know the stack deeply.

---

## Appendix: Why These Stacks

This section is for people who want to understand the reasoning or disagree with a recommendation. Skip it if you just want to build.

### The Python paradox

Python dominates every public agent benchmark. On Multi-SWE-bench (MopenHands + Claude 3.7 Sonnet), agents resolve 52% of Python tasks vs. 22% for Java, 5% for JavaScript, and 2% for TypeScript. A different agent setup (MSWE-agent) shows similar ordering: Python 46%, Java 23%, TypeScript 11%, JavaScript 5%. The exact numbers vary by agent, but the pattern is consistent: **Python first by a wide margin, then Java, then everything else far behind.** If raw agent fluency were the only criterion, every recommendation would be "use Python."

But for web and mobile, Python loses on deployment and ecosystem. There's no Python equivalent of `npx vercel` or Expo's one-command mobile builds. The web ecosystem's deployment story is so much better that it compensates for the weaker language-level agent performance — especially since Next.js has invested in framework-specific agent support that effectively closes the gap.

The rule: **prefer Python when the product is mostly logic (API, CLI, automation). Prefer JS/TS when the product is mostly UI and needs to ship fast.**

### Why Next.js specifically

Vercel publishes an AI agent eval page with before/after comparisons using AGENTS.md. Claude Opus 4.6 / Claude Code goes from 75% to 100%, Cursor Composer 2.0 from 75% to 96%. No other web framework publishes comparable numbers. Laravel is the closest — their Boost benchmarks show similar improvements — but Laravel is a backend framework, not a full-stack web solution for UI-heavy apps.

### Why Expo specifically

Expo has made concrete AI-agent investments: AGENTS.md in scaffolded projects, SDK-version-matched docs, an MCP server, and published agent skills. Replit routes its mobile flow through Expo for the same reason. Flutter is catching up — it now has AI docs, a Dart/Flutter MCP server, and a Gemini CLI extension — but the MCP server is still experimental and there are no public benchmarks comparable to Next.js or Expo's ecosystem. SwiftUI and Kotlin require GUI-dependent workflows that agents can't drive.

### Why FastAPI over Django

Django is excellent if you want its admin, ORM, and full-stack batteries. But for an API built by someone who doesn't know the framework, Django's surface area (settings.py, apps, middleware, template system) creates more opportunities for agent mistakes and more config you don't understand. FastAPI's surface area is smaller: one file can be a complete working API, Pydantic handles validation, and the docs are auto-generated.

### Why Tauri over Electron

Tauri produces dramatically smaller bundles than Electron (minimal Tauri apps can be under 600KB since they use the system webview; Electron bundles Chromium and Node.js — a meaningful size and memory difference), uses less memory, and the frontend is standard web tech. The cost is a Rust compilation step and a thinner Node.js integration story. For vibe coding, the trade-off favors Tauri because the frontend (where agents are reliable) stays in web tech, and the Rust layer should be as thin as possible. We recommend Vite + React rather than Next.js inside Tauri because Next.js is an SSR framework — its server components, API routes, and routing conventions create unexpected behavior inside a desktop shell that agents struggle to debug.

### Why Pydantic AI specifically

There are no reliable agent-framework benchmarks — no equivalent of Multi-SWE-bench or Next.js evals comparing Pydantic AI, LangChain, CrewAI, and others. The recommendation is based on selection criteria that matter for vibe coding, not benchmark charts:

**API stability.** Pydantic AI hit v1 in September 2025 with an explicit stability policy: no intentional breaking changes until v2. LangChain and LangGraph both have multi-page migration guides between major versions — churn that costs you debugging time and breaks the agent's training data. A stable API means the code your agent generates today still works next month.

**Surface area.** Pydantic AI has fewer concepts to learn and fewer ways for an agent to generate wrong-shaped code. There are agents, tools (plain Python functions), and typed outputs. LangChain has chains, runnables, callbacks, memory, output parsers, tools, retrievers, and multiple abstraction layers. CrewAI and AutoGen introduce their own vocabularies (crews, flows, AgentChat). Each concept is a dimension where generated code can be wrong.

**Testing ergonomics.** This is the strongest differentiator. `TestModel` and `FunctionModel` let you fake the LLM in pytest — tests run in milliseconds without API calls, and you can globally block real model calls in CI. `Agent.override` injects test dependencies. No other framework makes deterministic agent testing this straightforward, and deterministic testing is what makes the "verify every step" loop in vibe coding actually work.

**Deployment simplicity.** A Pydantic AI agent is a Python script — `uv run python main.py` runs it, `railway up` deploys it. No graph server, no orchestration runtime, no separate infrastructure for tracing.

### Benchmark sources

- [Multi-SWE-bench](https://arxiv.org/html/2504.02605): Language-level agent performance across Python, Java, TS, JS, Go, Rust, C, C++
- [Next.js Agent Evals](https://nextjs.org/evals): Framework-specific agent performance with and without AGENTS.md
- [Laravel Boost Benchmarks](https://laravel.com/blog/which-ai-model-is-best-for-laravel): Framework-specific agent performance by model
- [Stack Overflow 2025 Survey](https://survey.stackoverflow.co/2025/): Developer AI tool usage, productivity, and trust data
- [Anthropic Claude Code Usage Analysis](https://www.anthropic.com/research/impact-software-development): Real-world vibe coding patterns by language and category
- [Android Bench](https://developer.android.com/bench): Android-specific agent task performance by model

---

*Last updated: April 2026. Stacks and agent capabilities change fast. If you're reading this more than 6 months after the date above, verify recommendations against current benchmarks.*
