---
name: red-flags
description: Diagnose common failure patterns in vibe-coded projects and apply effective prompting and agent-structure fixes. Use this skill whenever the user says "red flags", "something feels wrong", "when should I stop", "agent going in circles", "how should I prompt", "prompting tips", "agent keeps breaking things", "I don't understand the code anymore", or describes symptoms like repeated failed fixes, runaway config, or losing trust in the agent. Apply it both reactively (diagnose and unblock) and proactively (teach the prompting and agent-structure patterns that prevent the symptoms).
---

# Red Flags, Prompting, and Agent Structure

This skill is the diagnostic and prompting companion to the path skills. Use it when the user reports symptoms ("agent keeps going in circles", "I can't explain what this code does") or when they ask for prompting help. It covers three things: universal red flags (what to do right now), how to prompt for vibe coding, and how to structure the agent (Level 1 rules file vs. Level 2 pipeline).

## Universal red flags — stop and reassess

Each red flag below is paired with a concrete action. When any of these fires, the right move is usually not "try harder" — it's to restructure the request.

### Agent going in circles

**Symptoms:** same bug unfixed after 3 attempts, or each fix introduces a new bug. Zero net progress per iteration.

**Why it's bad:** the agent doesn't have enough context to solve the problem, or the problem is outside its reliable range. More iterations make the codebase worse, not better.

**Action:** stop. Start a new conversation with full code and a tight problem statement. Ask the agent to *diagnose* the bug, not fix it: *"Don't write code. Describe the root cause in three sentences, then propose the smallest change that would fix it."* If the diagnosis doesn't make sense, ask a human.

### Fix requires a new major dependency

**Symptoms:** the agent proposes Redux / a new ORM / a message queue / a framework to solve what should be a localized bug.

**Why it's bad:** the agent is pattern-matching to "how this is solved in large production codebases," not "what's simplest here." Each new dependency is code the user can't debug.

**Action:** *"Solve this without adding any new dependency. If that's not possible, explain why in two sentences."* If it genuinely can't, the user has an architecture problem, not a code problem.

### More config than logic

**Symptoms:** more config files (webpack, tsconfig, docker-compose, CI, lint rules, env) than files with actual application logic.

**Why it's bad:** config is the least debuggable layer. Every config file is a place where one subtle error breaks everything with an incomprehensible message.

**Action:** delete configs that aren't strictly necessary. Use framework defaults. If the agent generates config the user can't read, ask it to explain each line — and remove lines nobody can justify.

### You can't explain what the code does

**Symptoms:** the app works, passes manual smoke tests, but the user can't describe the data flow, auth model, or what happens on a given user action.

**Why it's bad:** the first non-trivial bug will be unfixable — you can only change what you understand.

**Action:** ask the agent for a one-paragraph architecture summary. If *that* doesn't make sense, the project has outgrown vibe coding. Stop adding features; write tests; consider a specialist review.

### Agent touches unrelated files

**Symptoms:** a bug fix in checkout modifies navigation, auth config, and a utility file.

**Why it's bad:** the agent has lost the thread. Global changes to solve local problems = entangled codebase.

**Action:** reject the change. Restate scope tightly: *"Fix ONLY the function `X` in file `Y`. Do not modify any other files. If that's impossible, explain why."*

### Deployment needs manual steps the agent can't do

**Symptoms:** certificates, signing identities, web consoles, store metadata — the agent says "now do this manually."

**Why it's bad:** this is where vibe-coded projects die. Code is ready but shipping it requires domain knowledge the user doesn't have.

**Action:** this is a legitimate reason to hire a specialist for a few hours — for that specific step, not for rewriting the app.

## When to stop vibe coding entirely

Vibe coding is a tool for zero-to-working. It is not a permanent methodology. Surface these decisions explicitly when they apply.

- **Paying users or real dependencies** — "I don't fully understand the code" is now a liability. Get a specialist review before the first real incident.
- **Codebase exceeds review capacity** — can't read the diff and understand it. Rough heuristic ~50+ files or ~5k+ lines of AI-generated code, but the real threshold is comprehension, not line count.
- **More time debugging than building** — 80%+ on "why doesn't this work" = past the net-positive point.
- **Security or compliance matters** — AI code has known insecure-default patterns (hardcoded secrets, missing validation, permissive CORS). Get a security review before launch.
- **Idea validated, ready to scale** — keep what works, rewrite what doesn't, bring in stack experts.

## How to prompt for vibe coding

These are the core habits that keep sessions productive. Teach them by demonstrating them.

- **Start with the simplest working slice.** Don't describe ideal architecture. Describe the single thing the user does. Architecture emerges from working code.
- **Pin the stack in the first message.** Agents default to whatever's most popular in their training data unless told otherwise. State the exact stack and add *"do not suggest alternative frameworks."*
- **Ask for verification, not just code.** End every prompt with *"give me a command to verify this works"* or *"write a test for this."* Untested vibe code becomes unmaintainable.
- **Constrain explicitly.** *"No new dependencies without explaining why." "No state management libraries." "No classes unless the data model requires it."* Agents over-engineer by default; your job is to keep it simple.
- **One feature per conversation turn.** Each turn produces something verifiable before the next.
- **When stuck, start a new conversation.** Long conversations degrade performance. 20+ messages deep and circling = open a fresh chat, paste the current code and the specific problem.

## Structuring your agent

### Level 1 — Solo agent with a rules file

Right level for most projects. One markdown file at the project root: `CLAUDE.md` (Claude Code), `AGENTS.md` (Codex), `.cursorrules` (Cursor), `copilot-instructions.md` (Copilot). Format doesn't matter; consistency does.

Seed it with 5–10 project-specific rules as you go:

- Package manager (`uv` / `npm` / `pnpm` / `bun`)
- Files not to touch
- What "done" means for this project
- Where env vars live
- Testing expectations
- Forbidden dependencies

Takes ten minutes. Eliminates most session-to-session drift.

### Level 2 — Multi-agent pipeline

Upgrade only when Level 1's drift is unmistakable. Architect designs, Developer implements, QA tests from the spec alone, Reviewer checks read-only. No agent reviews its own work. Full setup in the `pipeline` skill.

### Signals it's time to go from Level 1 → Level 2

- Rules file doesn't stop context drift between sessions.
- One session can't contain a single change.
- The user is afraid the agent will regress working code.
- The agent self-reviews its own work with "looks good" and misses obvious issues.

Most projects never need Level 2. If yours does, the symptoms are unmistakable.

## Handing off to other skills

- "Help me set up the multi-agent pipeline" → `pipeline` skill.
- "Help me organize specs and tasks" → `sdd` skill.
- "What stack should I use for this?" → `vibe-stack-picker` skill (or a specific path skill).
