---
name: ai-agent
description: Scaffold and iterate on a vibe-coded AI agent — chatbot, RAG app, tool-using agent, or agentic workflow — using Pydantic AI + Python 3.13+ + uv + ruff + mypy + pytest + Logfire (observability), deployed to Railway. Use this skill whenever the user says "AI agent", "chatbot", "RAG", "retrieval-augmented", "tool-using agent", "agentic workflow", "LLM app", "Pydantic AI", "agent framework", "LangChain", "LangGraph", "CrewAI", or describes any product where an LLM decides what to do next. Apply it for the full lifecycle — scaffold, tests with TestModel, tool wiring, tracing, deploy — not just day one.
---

# AI Agent — Vibe Coding Path

This skill is for helping an experienced developer ship an AI agent, mostly with an AI coding agent. The goal is a typed, testable, observable agent that runs end-to-end before it picks up more abstractions.

**Default recommendation, no hedging:** Pydantic AI + Python 3.13+ + `uv` + `ruff` + `mypy` + `pytest` + Logfire. Deploy to Railway.

If the user arrives asking for LangChain, CrewAI, or AutoGen, acknowledge they're widely used, note the trade-offs below, and make the case for Pydantic AI as the vibe-coding default — then respect their choice.

## Why this stack (use only when asked)

- **API stability.** Pydantic AI hit v1 in September 2025 with an explicit no-breaking-changes-until-v2 policy. LangChain / LangGraph have multi-page migration guides between major versions — churn that breaks training data and costs the user debugging time.
- **Surface area.** Pydantic AI has three concepts: agents, tools (plain Python functions), typed outputs. LangChain has chains, runnables, callbacks, memory, output parsers, retrievers — each a dimension where the agent can generate the wrong shape. CrewAI / AutoGen introduce bespoke vocabularies on top.
- **Testing ergonomics (the real differentiator).** `TestModel` and `FunctionModel` fake the LLM in `pytest` — tests run in milliseconds, cost nothing, and you can globally block real model calls in CI. `Agent.override` injects test dependencies. No other framework makes deterministic agent testing this clean, and deterministic testing is what makes the vibe-coding verification loop actually work.
- **Deployment.** A Pydantic AI agent is a Python script. `uv run python main.py` runs it; `railway up` deploys it. No graph server, no orchestration runtime, no separate tracing infrastructure.
- **Observability.** Logfire is two lines of setup, built on OpenTelemetry (no lock-in). Langfuse is the drop-in self-hosted alternative.
- **MCP.** Add only when external tool interoperability is a real need; `pydantic-ai[mcp]` supports it first-class.

## Scaffold workflow

1. `uv init <project>`, `cd <project>`, `uv add pydantic-ai logfire`, `uv add --dev pytest ruff mypy pytest-asyncio`.
2. Single `main.py` with one `Agent` that has a typed Pydantic output model and one tool (a plain Python function decorated with `@agent.tool_plain`).
3. Wire Logfire in two lines at the top of `main.py`.
4. Add a `test_agent.py` using `TestModel` — every test must pass with no API key.
5. Verify: `uv run python main.py` (structured output, not raw LLM text), `uv run pytest -v` (all pass, no API key), `uv run mypy .`, open Logfire dashboard.
6. Deploy with `railway init` → `railway up`. Do not add multi-agent orchestration until the single agent works end-to-end in production.

## First prompt to hand the user

```
I'm building an AI agent that [describe what it does in one sentence].

Stack: Pydantic AI, Python 3.13+, uv, ruff, mypy, pytest, Logfire.
Deploy to Railway.

Apply these constraints:

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

## Constraints (enforce on every response)

- **One agent with a typed Pydantic output model.** No multi-agent orchestration until the single agent works end-to-end.
- **Tools as plain Python functions.** No framework abstractions, no LangChain `Tool` objects, no custom base classes.
- **`TestModel` in every `pytest` file from the start.** Tests pass without a real API key. Block real model calls in CI.
- **Logfire from the first run.** The user should see agent runs, tool calls, and latency immediately.
- **No framework-hopping.** If Pydantic AI can't do something, surface it explicitly — don't silently pull in LangChain or CrewAI.
- **Every response ends with a verification step.**
- **Files under 300 lines.** Split agent / tools / schemas into modules when the single file gets heavy.

## Verification checklist

```bash
uv run python main.py            # structured output matching the Pydantic output model (not raw text)
uv run pytest -v                 # all pass using TestModel (no API key)
uv run mypy .                    # types clean
# open Logfire dashboard          # see agent runs, tool calls, latency
railway up                       # public URL
```

## Stop signals

- 3+ LLM providers configured before one working use case.
- Agent introduces a graph / state-machine abstraction for a single-turn tool-calling agent.
- Tests require a real API key to pass (means `TestModel` isn't being used).
- User can't explain what each tool function does in one sentence.
- Token costs climbing but no traces to show why.

## Escape routes

- **Pydantic AI edge case bites:** tool functions are plain Python — extract them and wire to the provider SDK directly (`anthropic`, `openai`). The business logic is preserved.
- **Real multi-agent orchestration need:** graduate to LangGraph, trading simplicity for orchestration depth. Keep tools intact.
- **Observability lock-in concerns:** Langfuse is a self-hosted drop-in for Logfire; both speak OpenTelemetry.
- **MCP interoperability becomes a real requirement:** `pydantic-ai[mcp]` adds MCP server/client support without leaving the framework.

## Handing off to other skills

- HTTP surface for the agent (webhook, REST) → `api-backend` (FastAPI pairs cleanly with Pydantic AI).
- Frontend chat UI → `web-app` (Next.js hitting the FastAPI backend).
- Script/cron wrapping the agent → `cli-automation`.
- Bigger feature that needs a spec → `sdd`.
- Going in circles → `red-flags`.
