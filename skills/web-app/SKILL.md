---
name: web-app
description: Scaffold and iterate on a vibe-coded web app — SaaS, dashboard, internal tool, authenticated web UI, or landing page — on Next.js 16 + TypeScript + Tailwind + Vercel + Supabase. Use this skill whenever the user says "web app", "SaaS", "dashboard", "internal tool", "landing page", "Next.js project", "admin panel", "marketing site with auth", or describes a UI-centric product that runs in a browser (and they haven't already pinned a different stack). Apply this skill for the full lifecycle: initial scaffold, feature iteration, deploy, and stopping conditions — not just day one.
---

# Web App — Vibe Coding Path

This skill is for helping an experienced developer build a web app mostly with an AI coding agent. The goal is the shortest verifiable path from "empty folder" to "URL a user can open," not architectural elegance.

**Default recommendation, no hedging:** Next.js 16 (App Router) + TypeScript + Tailwind CSS, deployed on Vercel, with Supabase for auth + Postgres. For a pure landing page, skip Supabase and auth entirely.

## Why this stack (use only when asked)

Next.js is the only web framework that publishes measured agent evals — [Claude Opus 4.6 / Claude Code goes 75% → 100% with `AGENTS.md`](https://nextjs.org/evals). The verification loop is fast (`npm run build` + `tsc --noEmit` + one-click Vercel deploy), and the community is the largest escape hatch when the agent gets stuck. TypeScript is not the best agent-fluency language by itself, but Next.js's framework-specific support closes the gap for web.

Don't argue the user into a different framework. If they ask for Remix, Astro, or SvelteKit, note that none publish comparable agent evals — then respect their choice and adapt the scaffold.

## Scaffold workflow

Run these in order. Don't skip verification between steps.

1. `npx create-next-app@latest <project> --typescript --tailwind --app --src-dir` — or without `--src-dir` if the user prefers. Confirm the app boots on `localhost` before touching anything else.
2. Add Supabase only when the first auth or data feature is actually needed: `npm install @supabase/supabase-js @supabase/ssr`. Stripe is added the same way — only when the first paid feature is in scope.
3. Wire `@/lib/supabase/server.ts` and `@/lib/supabase/client.ts` using `@supabase/ssr`. Keep data fetching in server components or route handlers.
4. Commit and push. First Vercel deploy should take under 5 minutes via the Vercel GitHub integration.

## First prompt to hand the user (paste verbatim when useful)

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
- Every response ends with a verification step.

Start by scaffolding the project and showing me how to run it locally.
```

## Constraints (enforce on every response while this skill is active)

- **Server components by default.** `'use client'` is only for browser APIs (`onClick`, `useState`, `useEffect`, clipboard, drag/drop).
- **Static generation unless explicitly needed.** No `getServerSideProps`, no runtime data fetching without justification.
- **Data access in server components or route handlers** — never in client components.
- **No state management libraries** until 5+ interactive pages share state. Server components + `useState` + URL state get you surprisingly far.
- **No new dependency without a one-sentence justification** in the same response.
- **Keep files under 300 lines.** Split by route, component, or lib module when exceeded.
- **Never generate a custom auth flow.** Use Supabase Auth. Billing? Stripe hosted checkout.
- **End every response with a verification step** — a command, a test, or a URL to open.

## Verification checklist (run after every meaningful change)

```bash
npm run dev          # "Ready" + localhost URL
npm run build        # no errors (this catches the bulk of typed/routing mistakes)
npx tsc --noEmit     # types clean
# Deploy: Vercel GitHub integration (3 clicks) or `npx vercel`
```

If `npm run build` fails and the agent can't resolve it in one attempt, stop and read the error carefully before asking for another fix — build errors in Next.js usually point at a specific file and line.

## Stop signals — pause and warn the user

- The agent adds Redux / Zustand / Jotai to a project with fewer than 5 interactive pages.
- The user has 3+ environment variables they can't explain.
- The agent creates a custom auth flow instead of using Supabase Auth.
- The user can't explain what any given API route does in one sentence.
- Build errors reference `webpack` / `turbopack` config the user has never seen.

When any of these fire, surface them explicitly: *"I want to flag something — you now have X, which usually means Y. Want me to simplify before continuing?"*

## Escape routes

- **Frontend is fine, backend got messy:** keep Next.js on the UI, move server logic to a separate FastAPI service (use the `api-backend` skill). The frontend becomes "just renders data."
- **Even the frontend is too much:** fall back to a static site with Supabase as the entire backend (auth + DB + edge functions).
- **Deployment blocker (domains, DNS, edge config):** that's a specialist-hours problem, not a rewrite problem.

## Handing off to other skills

- Auth/db architecture questions deep enough to need a spec → `sdd` skill.
- Multi-agent workflow (architect + dev + QA + reviewer) for larger features → `pipeline` skill.
- Diagnostic triage when things are clearly off the rails → `red-flags` skill.
