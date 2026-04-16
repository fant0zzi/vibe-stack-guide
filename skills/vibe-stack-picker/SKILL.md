---
name: vibe-stack-picker
description: Help an experienced developer pick a stack and scaffold a new project that will be built mostly with an AI coding agent (Cursor, Claude Code, Windsurf, Copilot, etc.). Use this skill whenever the user says "I want to build ___", "help me pick a stack", "which framework should I use", "vibe coding", "I'm starting a new project", or describes a product idea without a pinned stack. Also use it whenever a request is ambiguous about project type — call this skill first, and it will route you to the correct path-specific skill (web-app, ai-agent, api-backend, cli-automation, browser-extension, desktop-app, mobile-app-store, mobile-pwa).
---

# Vibe Stack Picker

This skill is the entry point for "I want to build something with an AI coding agent." It picks the right path, hands off to the path-specific skill, and enforces the universal execution rules while the project gets scaffolded.

The user is an experienced developer who can evaluate architectural trade-offs but is likely working outside their usual stack. They don't want three choices — they want one recommendation, one scaffold, and a verification command they can actually run.

## When this skill triggers — what to do first

1. **Identify the project category** using the decision ladder below. If the user's description is ambiguous, ask one targeted question (e.g., "Does this need to live in the App Store, or is the web enough?") rather than branching into multiple options.
2. **Announce the recommendation and the path skill you're routing to.** Example: "Based on what you described, this is a Web App. I'll apply the `web-app` skill." Then invoke that skill's playbook.
3. **Apply the universal execution constraints** below for the rest of the conversation — they stay active even after the path skill takes over.
4. **Watch for red flags** and stop the build if any appear (see final section).

## Decision ladder

Walk top-to-bottom. First match wins. Do not suggest alternative frameworks once a path is chosen.

1. **Chatbot, RAG, tool-using agent, or agentic workflow?** → `ai-agent` (Pydantic AI + Python)
2. **No UI — it's an API, webhook, or server-side logic?** → `api-backend` (FastAPI)
3. **No UI — it's a script, cron, scraper, or data pipeline?** → `cli-automation` (Python)
4. **Browser extension (Chrome, Firefox, cross-browser)?** → `browser-extension` (TypeScript + MV3)
5. **Must run as a native desktop app (filesystem, tray, OS integration)?** → `desktop-app` (Tauri v2). First confirm the user really needs desktop — if the answer is "works offline," a PWA is usually better.
6. **Runs on phones and must ship through Apple/Google store?** → `mobile-app-store` (Expo)
7. **Runs on phones but does NOT need the store?** → `mobile-pwa` (Next.js PWA)
8. **Anything else with a UI** (SaaS, dashboard, internal tool, landing page) → `web-app` (Next.js + Supabase)

If the user hasn't said what they're building yet, ask: "In one sentence, what does the user do with this?" Then walk the ladder.

## Validate before scaffolding

Before running any `create-*` or `init` command, confirm in one short sentence:

> Based on what you described, I'd recommend **[stack]** via the `[path]` path. Does this match, or is there a requirement I'm missing (App Store, desktop-only features, offline, etc.)?

If they confirm, proceed. If they push back, walk the ladder again with the new constraint.

## Universal execution constraints (keep active for every response)

These apply no matter which path skill is running. They're the single most important thing the skill enforces, because they're why vibe-coded projects don't collapse at week two.

1. **Start with the simplest working slice.** One page, one endpoint, one function — verified end-to-end — before adding anything else. Architecture emerges from working code.
2. **Every response ends with a verification step.** A command to run, a test to execute, or a manual check the user can perform in under a minute. Never hand off code without a way to confirm it works.
3. **No new dependency without a one-sentence justification.** If the alternative is ~20 lines of code, write the 20 lines instead.
4. **No state management libraries** (Redux, Zustand, Jotai, Pinia, etc.) until the project has 5+ interactive pages with shared state. Use built-in state until the pain is concrete.
5. **Pin the stack.** Do not suggest switching frameworks mid-project. If the chosen stack can't do something, say so and explain the options — don't silently introduce a new tool.
6. **Keep files under 300 lines.** Split when exceeded, so the user can review in coherent chunks.
7. **Write a test before moving on.** One per endpoint, per core function, per user flow. Tests are the safety net for code the user didn't write.
8. **Explain errors in plain language.** Describe what went wrong, not which framework internal threw. Then fix.

## Escape routes (offer when the user hits a wall)

| Stuck on… | Offer |
|---|---|
| Frontend complexity in Next.js | Keep Next.js frontend, move backend to a separate FastAPI service |
| App Store signing / submission | Ship as a PWA first; hire a specialist only for the store leg |
| Desktop packaging / code signing | Ship as a web app; add Tauri later once demand is proven |
| Database / ORM tangle | Fall back to SQLite in dev, Supabase Postgres in prod |
| Auth complexity | Supabase Auth or Clerk — never custom auth |
| Agent going in circles | Fresh conversation with full code + specific problem |
| "I don't understand this code anymore" | Stop adding features. Write tests. Consider a specialist review. |

## Red flags — stop and reassess

If any of these appear during scaffolding or early iteration, stop coding and surface the problem to the user. Don't try to power through.

- Same bug unfixed after 3 attempts, or each fix introduces a new bug.
- The "fix" requires a new major dependency (Redux, new ORM, message queue) to solve a localized issue.
- More config files than logic files.
- The user can't explain what a route / endpoint / function does in one sentence.
- A bug fix modifies unrelated files (nav, auth, random utilities).
- Deployment requires a manual step the agent can't do (certificates, store metadata, signing) — this is a legitimate reason to hire a specialist for that step only.

Full details live in the `red-flags` skill — route there when the user wants the diagnostic playbook.

## Reference data (share only when relevant)

- Python resolves ~52% of tasks on Multi-SWE-bench; Java ~22%; JS/TS in single digits. This is why backend/CLI/agent paths default to Python.
- Next.js publishes [agent evals](https://nextjs.org/evals): Claude Opus 4.6 / Claude Code 75% → 100% with `AGENTS.md`. No other web framework publishes comparable numbers.
- Expo ships `AGENTS.md` by default from `create-expo-app`, has SDK-version-matched docs, and an official MCP server.
- Automation/CLI is the #1 category for AI-assisted coding per Anthropic usage data.

*Source: Vibe Coding Stack Guide, April 2026. If you're reading this more than ~6 months later, re-verify the benchmarks before citing them.*
