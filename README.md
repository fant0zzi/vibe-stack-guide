# Vibe Coding Stack Guide

**Pick the right stack for AI-assisted development — before the project dies at deployment.**

You're an experienced developer building something outside your stack. You'll use Cursor, Claude Code, Copilot, or another AI agent to write most of the code. This guide helps you make smart stack choices upfront, because wrong choices kill vibe-coded projects at environment setup, deployment, or state management — not at the coding stage.

## What's in this repo

| File | What it is | Who it's for |
|---|---|---|
| [GUIDE.md](./GUIDE.md) | The full guide — decision framework, stack recommendations, first prompts, red flags | You, the developer. Read your path in 2 minutes. |
| [AGENT_CONTEXT.md](./AGENT_CONTEXT.md) | Starter prompt for your AI agent — decision criteria, rules, escape routes | Your AI agent. Paste into context or drop into project root. |
| [PIPELINE.md](./PIPELINE.md) | Multi-agent development pipeline — separated cognitive roles, 4-phase workflow, bounded revision loops | Teams scaling beyond a solo agent. |

## Quick start

**Option A — read the guide:**
Open [GUIDE.md](./GUIDE.md), find your path (web app, mobile, API, CLI, extension, desktop, AI agent), and start building.

**Option B — use the starter prompt:**
Paste [AGENT_CONTEXT.md](./AGENT_CONTEXT.md) into your coding agent's context window and say:
> "I want to build [X]."

The agent will use the guide's decision criteria to pick your stack and give you a starting plan.

**Option C — set it and forget it:**
Copy `AGENT_CONTEXT.md` into your project root as `CLAUDE.md`, `.cursorrules`, or `AGENTS.md`. The agent will use it automatically on every conversation.

## What makes this different

This is not "use React for web apps." Every recommendation is evaluated through criteria specific to vibe coding:

- **Agent fluency** — which stacks do current models generate reliably, backed by Multi-SWE-bench and framework-specific eval data
- **Deployment simplicity** — shortest path from "works locally" to "user can access it"
- **Verification loop** — can you confirm each change works without knowing the stack?
- **Error debuggability** — when it breaks, can you understand the error?
- **Escape hatches** — when you're stuck, what are the concrete options?

## Covered paths

- Web app (SaaS / dashboard / tool / landing page)
- Mobile app — App Store (iOS, Android)
- Mobile app — no App Store (PWA)
- API / backend service
- CLI / automation / data pipeline
- Browser extension
- Desktop app
- AI agent (chatbot, RAG app, tool-using agent, agentic workflow)

## Interactive website

There's also an [interactive website](https://vibe-stack.tech) that makes it easy to find your path, customize a starter prompt, and copy what you need.

## Contributing

Stack recommendations and agent capabilities change fast. If something is outdated or wrong, open an issue or PR. The guide is opinionated by design — but opinions should be backed by evidence, not vibes.

## License

[MIT](./LICENSE)
