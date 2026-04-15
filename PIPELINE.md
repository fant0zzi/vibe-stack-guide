# Multi-Agent Development Pipeline

## The problem

When code generation is delegated to an LLM, thousands of micro-decisions that a human developer makes intuitively are delegated too. The result is **intention drift**: the agent takes a step left, a step right, drifts into a refactor, and ten iterations later the system does what was easy to write, not what was needed.

A single agent doing design, implementation, testing, and review repeats the same reasoning errors at every stage. DORA's 2025–2026 research confirmed that AI acts as an amplifier: teams with strong engineering practices saw AI accelerate them further, while teams with weak practices saw instability increase alongside throughput. The environment matters more than the prompt.

This pipeline addresses four core failure modes: intention drift, self-review bias, scope creep, and the illusion of correctness.

---

## The approach: separated cognitive roles

The pipeline splits development into four specialized agent roles, each with different tools, models, and constraints. No agent reviews its own work. No agent implements what it designed. This separation is the primary defense against the structural weaknesses of LLM-generated code.

| Agent | Model | Tools | Purpose |
|---|---|---|---|
| **Architect** | Opus (deep reasoning) | Read, Grep, Glob, Write, Edit | Design, specs, task decomposition |
| **Developer** | Sonnet (implementation) | All tools | Implementation from specs; makes QA tests pass |
| **Reviewer** | Sonnet or Haiku | Read, Grep, Glob (read-only) | Reviews specs and code; cannot modify files |
| **QA** | Sonnet | All tools | Test plans before impl; verification after |

**Why these separations:**

- The architect can analyze and produce specs but cannot execute commands, keeping it in design mode.
- The reviewer is strictly read-only — physically cannot modify code, making its findings trustworthy.
- QA writes tests before implementation — tests derived from requirements, not from reading code.
- The developer never sees the spec-writing process — implements exactly what the spec says.

**Dynamic model selection:** Opus for tasks where reasoning depth matters (architecture). Sonnet for implementation and comprehensive review. Haiku for parallel targeted reviews (security, style, naming) where focused speed matters more than broad reasoning. The orchestrator can spawn multiple Haiku reviewers simultaneously, each checking one specific concern.

---

## Spec-Driven Development (SDD)

The pipeline uses Spec-Driven Development as its artifact system. Feature specs are the durable source of truth for product intent, architecture decisions, execution status, validation, and deferred follow-ups. They live under `specs/feature-slug/` at the project root.

### Feature folder structure

```text
specs/
  feature-slug/
    README.md     # entrypoint
    prd.md        # requirements, scope, architecture
    design.md     # optional: larger or cross-cutting work
    tasks/        # required once tracking implementation
      001-task-name.md
      002-another-task.md
    artifacts/    # optional: supporting material
      backlog.md
```

### Task document shape

Each task file must be executable and stateful. Required sections: summary, scope, affected files, acceptance criteria (markdown checklists), implementation checklists, validation, and optionally non-goals, risks/blockers, and follow-ups.

Every feature and task doc starts with frontmatter:

```yaml
---
title: Example Title
status: draft    # draft|active|blocked|review|implemented|abandoned
priority: high
owner: shared    # ai|human|shared
last_updated: 2026-01-01
depends_on: []
---
```

### Status transitions

Status transitions are defined by evidence, not intent:

- `draft` → `active`: requirements and scope are agreed upon.
- `active` → `review`: code changes are complete but not yet fully validated.
- `review` → `implemented`: acceptance criteria validated with named checks (tests passing, commands run, manual verification logged).
- Any → `blocked`: a dependency or prerequisite is missing. State what is missing.
- Any → `abandoned`: decision to not proceed. State why.

A task does not move to `implemented` based on self-review alone. If validation evidence is missing, the task stays in `review`.

---

## The pipeline

**Phase 1 → 2 → 3 → 4**, each gated by reviewer approval:

| Phase | Agent | What happens | Exit gate |
|-------|-------|-------------|-----------|
| **1. Design** | Architect → Reviewer | Architect writes PRD + specs. Reviewer validates. Loop until approved (max 3 cycles). | Reviewer APPROVED, schemas locked |
| **2. Test planning** | QA → Reviewer | QA writes failing tests from spec alone (no source code). Reviewer validates completeness. | Tests exist and FAIL before implementation |
| **3. Implementation** | Developer → Reviewer | Developer makes QA tests pass, per task. Reviewer checks code against acceptance criteria. | All tests pass, reviewer APPROVED |
| **4. Validation** | QA | QA reads source, adds edge-case tests, assesses mutation-awareness. | All acceptance criteria covered |

At every phase boundary: if the reviewer says **REVISE**, loop back to the phase agent. If they say **APPROVED**, advance.

### Phase 1: Design

The architect receives the feature request and produces specs and numbered task documents under `specs/feature-slug/`. Design starts with hard constraints — schemas, API contracts, data flow — because these are concrete walls that downstream agents cannot escape.

The reviewer validates the spec, checking completeness, feasibility, simplicity, and the key question: **can the QA agent write a complete test plan from this spec alone, without seeing source code?**

**Exit criteria:** Reviewer verdict APPROVED (max 3 revision cycles). All task docs have concrete, testable acceptance criteria. Contracts locked before Phase 2 begins. Spec status advanced to `active`.

### Phase 2: Test planning (QA-first)

This is the critical innovation. The QA agent is spawned with explicit instructions to NOT read source files. It reads only the spec, acceptance criteria, and locked contracts. From these alone, it writes test files that initially fail.

Research shows LLM-generated tests can achieve 100% line coverage while catching only 4% of seeded mutations (MUTGEN, 2025). Independence from implementation is the countermeasure. Tests that pass without implementation prove nothing — the orchestrator verifies they fail before proceeding.

**Exit criteria:** Reviewer approves test plan (max 2 cycles). At least one test per acceptance criterion. All tests confirmed to fail against unimplemented code.

### Phase 3: Implementation

Task order is determined by topological sort of `depends_on` fields. If circular dependencies are detected, the orchestrator halts and reports to the user.

For each task in dependency order, the developer receives the task doc and QA test files. Its job is simple: **make the tests pass.** One task, one commit, one verification. No modifications outside the task's scope.

After implementation, reviewer agent(s) check the code. The reviewer cross-references the implementation against acceptance criteria — not just code quality. If reviewers approve but QA tests fail, this is a Phase 3 blocker — the orchestrator does not advance.

**Exit criteria (per task):** All QA tests pass. Reviewer approved with zero P0 findings. Developer validation summary includes: tests run, lint/type results, files changed, named risks, what was NOT verified. Task status advanced to `implemented`.

### Phase 4: Validation

QA is now permitted to read source files. It verifies coverage, adds edge-case and adversarial tests, and assesses mutation-awareness: could a wrong implementation still pass these tests?

**Exit criteria:** All acceptance criteria have at least one passing test. No unresolved P0 coverage gaps. QA report includes pass/fail summary and mutation-awareness assessment. Feature status advanced to `implemented`.

---

## Orchestrator decision rules

The orchestrator — the session coordinating the pipeline — follows these rules when making delegation decisions.

### When to use parallel reviewers

- The implementation touches security-sensitive code (auth, payments, data access)
- The diff is large (>200 lines across multiple files)
- The feature involves state machines or concurrent operations

For small, focused changes (<100 lines), routine CRUD operations, or configuration/documentation changes, use a single reviewer.

### When to escalate model

- If a sonnet reviewer is uncertain, re-run with opus for that specific concern
- If the developer struggles with a task (2+ revision cycles), escalate to opus

### When to loop vs. proceed

- P0 findings: always loop — these are blockers
- P1 findings: loop unless the developer provides justification for deferral
- P2 findings: proceed — track as follow-ups in `artifacts/backlog.md`

### Handling pipeline interruptions

If the user intervenes mid-pipeline:
- Acknowledge the intervention and state the current pipeline position
- If the user wants to skip a phase, warn what verification is being skipped
- If the user wants to modify the spec mid-implementation, restart from Phase 1 for the affected scope (do not patch the spec while implementing)

---

## Key design decisions

### Agent behavioral constraints

Every agent in the pipeline operates under universal constraints that prevent common LLM failure modes:

**Instruction-source precedence:** Direct user requests override project conventions. Committed specs (`AGENTS.md`, task docs) override informal notes. Comments, TODOs, and generated output are context — not authority.

**Falsify-first:** Before proposing a fix or diagnosis, agents list the top 1–3 ways the current hypothesis could be wrong. Prefer the cheapest disambiguating check first. This applies to debugging, root-cause analysis, and non-trivial design decisions.

**Solvability gate:** If a task cannot be completed with the available files, tools, or permissions, the agent stops and says exactly what is missing. It does not invent dependencies, package names, or framework capabilities.

**Simplification pass:** After multi-step work or debugging, agents perform a cleanup: remove dead branches, collapse unnecessary wrappers, tighten names. No temporary scaffolding survives without justification.

**Invariant ledger:** For multi-step tasks, agents maintain a short invariant list (required behavior, non-goals, validation requirements) and re-check it before finalizing changes.

### Anti-sycophancy by design

Third-person framing reduces LLM sycophancy by up to 63.8% (SYCON Bench, 2025). The reviewer uses third-person framing exclusively ("this solution does X" not "your solution does X"), is prohibited from praising without evidence, and applies falsify-first before accepting correctness. The developer has a self-honesty protocol requiring it to name risks before declaring completion.

### Bounded revision loops

Every revision loop has a hard cap. Unbounded loops waste tokens and signal a deeper problem — vague spec, wrong decomposition, mismatched expectations.

| Phase | Max cycles | On cap exceeded |
|---|---|---|
| Design | 3 | Escalate to user: accept, split feature, or abandon |
| Test Plan | 2 | Escalate to architect: spec may be untestable |
| Implementation | 3 + 1 fresh restart | Split task, defer, or escalate model |
| Validation | 2 | Document remaining gaps, user decides |
| Hotfix | 1 | Escalate to user: rollback or downgrade to normal pipeline |

### Investigation gates

Every agent has a mandatory gate: read the actual code before making claims about it. The architect reads source files before designing. The developer reads affected files and tests before coding. The reviewer traces real call paths before judging. This eliminates the "designing against imagined code" failure mode.

### Evidence-based status transitions

No status moves on claims alone. "Done" requires: which tests pass, which commands succeeded, what was NOT verified. The validation summary shape differs by task type: code changes list commands run, tests run, files changed, and named risks; planning tasks list constraints checked and open questions; reviews list evidence basis per finding.

### Context rot defense

After 3+ revision cycles, agents restart with fresh context containing only the essentials: task doc, acceptance criteria, test files, and latest findings. Short invariant lists outperform long transcripts as context grows (Context Rot, Chroma Research 2025).

---

## Phase advancement checklists

Before advancing any phase, the orchestrator verifies every item. No advancement on agent claims alone — artifacts must exist and conditions must be met.

**End of Phase 1 (Design → Test Planning):**
- Spec status set to `active` (not `draft`)
- PRD and task docs exist under `specs/feature-slug/`
- Every task doc has acceptance criteria checklists
- Reviewer verdict = `APPROVED`
- Contracts and schemas explicitly marked "locked" in the spec
- Reviewer confirmed: acceptance criteria are testable by QA without source code

**End of Phase 2 (Test Planning → Implementation):**
- Test files exist
- At least 1 test per acceptance-criterion item
- Reviewer verdict on test plan = `APPROVED`
- Orchestrator verified tests actually fail against unimplemented code

**End of Phase 3 per task (Implementation → next task or Phase 4):**
- Developer validation summary complete (tests, lint/type results, files changed, risks, what was NOT verified)
- All QA tests for this task pass
- Reviewer verdict = `APPROVED` with zero unresolved P0 findings
- P1 findings either fixed or deferred with written justification
- Task status advanced to `implemented`

**End of Phase 4 (Validation → Feature complete):**
- QA Mode B report complete (pass/fail summary, coverage gaps, mutation-awareness)
- All acceptance-criterion subsets have at least one passing test
- No unresolved P0 gaps
- Feature status advanced to `implemented`
- Follow-up items logged in `artifacts/backlog.md`

---

## Hotfix variant

For critical production issues, the full pipeline is too slow. The hotfix variant skips Phases 1–2:

1. Developer reproduces the bug (writes a failing test first)
2. Implements the minimal fix to make the test pass
3. Single reviewer pass
4. Deploy
5. Backfill: retroactive spec + QA coverage verification
6. Follow-up tasks logged in `artifacts/backlog.md`

Max 1 revision cycle. Reviewer verdict requires zero P0 findings; P1 findings are logged but do not block. If the reviewer still rejects, escalate to user: rollback or downgrade to normal pipeline.

**Hotfix completion checklist:**
- Failing test reproduces the bug
- Fix makes the test pass
- Reviewer verdict `APPROVED` (P0-clean; P1 findings logged, not blocking)
- Retroactive spec documents what changed and why
- QA Mode B coverage verified around the fix
- Tech debt follow-ups logged in `artifacts/backlog.md`

---

## Status tracking

The orchestrator maintains a progress summary after each agent completes:

```
## Pipeline Status: [feature-slug]
- Phase: [1-Design | 2-TestPlan | 3-Implementation | 4-Validation]
- Current task: [task number / total] (Phase 3 only)
- Revision cycle: [number] (within current phase)
- Blocking findings: [count or "none"]
- Next action: [what agent to spawn next and why]
```

### Objective exit criteria summary

| Phase | Success = advance | Failure = escalate |
|---|---|---|
| 1-Design | Reviewer `APPROVED` + schemas locked + all tasks have testable criteria | Reviewer `REVISE` on cycle 4+ |
| 2-TestPlan | Reviewer `APPROVED` + test files exist + tests fail before impl | Reviewer `REVISE` on cycle 3+ or QA solvability gap unresolvable |
| 3-Impl | QA tests pass + reviewer `APPROVED` + zero P0 + validation summary complete | Developer restart-cycle-2 still failing |
| 4-Validation | All acceptance subsets covered + no P0 gaps + QA report complete | QA-developer loop 3+ |
| Hotfix | Reviewer `APPROVED` (P0-clean) + test passes + backfill created | Reviewer rejects on cycle 2+ |

---

## Configuration structure

The pipeline is configured through two systems: thin agent definitions (always in context) and deep skill folders (loaded on demand).

```
.claude/
  agents/
    architect.md      # Role, model, tools, skill reference
    developer.md
    reviewer.md
    qa.md
  skills/
    architect-design/
      SKILL.md        # Design methodology
      references/     # Task templates, design checklists
    developer-impl/
      SKILL.md        # Implementation workflow
      references/     # Code style, architecture guardrails
    reviewer-analysis/
      SKILL.md        # Review methodology
      references/     # Spec review checklist, code review checklist
    qa-testing/
      SKILL.md        # Test planning methodology
      references/     # Adversarial patterns, edge-case checklist
```

**Progressive disclosure:** Agent definitions (~100 words) are always in context. Skill bodies (~500 words) load when the agent is spawned. Reference files load only when the skill directs the agent to read them. This keeps token usage efficient while giving each role deep domain knowledge.

The orchestrator-level rules (pipeline flow, SDD workflow, behavioral constraints, phase checklists) live in `AGENTS.md` at the project root. Agents read it as their first action.

---

## Stack independence

The pipeline logic — phases, revision loops, advancement checklists, agent roles, SDD workflow — is entirely stack-agnostic. Stack-specific content (tooling commands, framework references, code style) lives only in the skill reference files and agent operational rules.

To adapt for a different stack:

1. Swap the reference files in `developer-impl/references/` and `qa-testing/references/`
2. Update operational rules (package manager, linter, test runner) in developer and QA agent definitions
3. Update code style and architecture guardrails for your framework

The pipeline itself, phase advancement checklists, behavioral constraints, and revision cycle limits carry over unchanged.

---

## Underlying principles

The pipeline is grounded in evidence from 2025–2026 research on LLM-assisted development:

**Environment over prompts.** Hard constraints (schemas, type systems, failing tests) constrain agents far better than instructions. Invest in environment, not prompt engineering.

**Separation of planning and execution.** METR 2025 found developers were 19% slower with AI when the agent started acting before a plan was agreed upon. The architect designs; the developer implements. Never simultaneously.

**Independent verification.** Tests from the same process that wrote the code repeat the same reasoning errors (ICSE 2025). The QA agent writes tests before seeing implementation, making them genuinely independent.

**Bounded iteration.** Every loop has a cap. Escalation paths exist for every stuck state. Unbounded iteration signals a structural problem, not insufficient effort.

**Evidence-based transitions.** No phase moves on claims alone. "Done" means: which checks passed, which commands ran, what was not verified. The distinction between "implemented," "reasoned about," and "verified by execution" is always explicit.

---

## Getting started

1. Copy the `.claude/` directory into your project root
2. Place `AGENTS.md` at the root — it contains the orchestrator-level rules (pipeline flow, SDD workflow, behavioral constraints, phase checklists)
3. Update the skill reference files for your stack's tooling and code style
4. Add your project's requirements document to the root (the agents read it as their first action)
5. Start a Claude Code session and describe the feature — the orchestrator reads `AGENTS.md`, spawns the agents, and follows the pipeline automatically

For most projects, start with a solo rules file (Level 1 in the [Vibe Coding Stack Guide](./GUIDE.md#structuring-your-agent)). Move to this pipeline when the symptoms described there become unmistakable.

---

## Starter workflow pack

This is a portable reference for setting up a multi-agent development pipeline. It is tool-agnostic and stack-agnostic — adapt role names, commands, and file paths to your setup. Use as-is for a full pipeline, or cherry-pick sections for a lighter workflow.

### Role hierarchy

| Role | Scope | Capabilities | Produces | Must NOT |
|---|---|---|---|---|
| **Architect** | Design, decomposition, contracts | Read/write specs and task docs | PRD, task docs, acceptance criteria | Write production code, run tests |
| **Developer** | Implementation from specs | Read/write code, run commands | Code changes, validation summary | Design features, invent tests, redefine acceptance criteria |
| **Reviewer** | Evaluate specs and code | Read-only access to all files | Findings by severity, verdict | Modify any files, write code or tests |
| **QA** | Test planning and validation | Read/write tests, run commands | Test files, coverage report | Change requirements (Mode A: also must not read source) |
| **Designer** *(optional)* | UI/interaction design | Read specs, produce visual artifacts | Component mockups, interaction specs, visual guidelines | Implement code, define backend contracts |

**When to add a Designer:** For UI-heavy projects where visual and interaction decisions are significant. In simpler projects, the Architect can absorb design review. Separating design prevents the Architect from making aesthetic decisions without visual context.

### Role responsibilities (expanded)

**Architect**
- Reads requirements, maps content to routes and components
- Defines contracts (API shapes, schemas, data flow) before implementation begins
- Decomposes features into independently implementable, independently verifiable tasks
- Each task is small enough to be one commit
- Does not write production code — illustrative snippets only

**Developer**
- Implements exactly what the task doc specifies
- Makes QA tests pass — tests define "done"
- Runs validation after every change: build, type check, lint, visual verification
- Reports what was verified and what was NOT verified
- If the task is ambiguous, stops and reports the ambiguity

**Reviewer**
- Cannot modify files — this constraint makes findings trustworthy
- Cross-references implementation against acceptance criteria, not just code quality
- Uses third-person framing ("this component does X" not "your component does X")
- Classifies findings by severity: P0 (must fix), P1 (should fix), P2 (consider)
- Verdict: APPROVED, REVISE (with blocking findings), or BLOCKED (missing info)

**QA**
- Mode A (pre-implementation): writes tests from spec alone, without reading source
- Mode B (post-implementation): reads source, adds edge-case and adversarial tests
- Tests that pass before implementation prove nothing — verifies they fail first
- Assesses mutation-awareness: could a wrong implementation still pass these tests?

### Skill / capability map

| Skill area | Used by | Covers |
|---|---|---|
| Design methodology | Architect | Requirements analysis, route/content mapping, component hierarchy |
| Task decomposition | Architect | Breaking features into implementable units with clear acceptance criteria |
| Implementation workflow | Developer | Read contract → implement minimally → validate → simplify → report |
| Code style & guardrails | Developer | Stack-specific naming, patterns, file structure, framework conventions |
| Spec review | Reviewer | Completeness, testability, simplicity, feasibility |
| Code review | Reviewer | Correctness, security, complexity, intent conformance |
| Test planning | QA | Deriving test cases from acceptance criteria without implementation knowledge |
| Edge-case generation | QA | Adversarial inputs, boundary conditions, failure modes |

### Pipeline phases with gates

**Phase 1 — Design**
- Architect produces specs and task docs
- Reviewer validates: is every task testable by QA without seeing source?
- Gate: reviewer APPROVED, contracts locked, spec status `active`
- Max 3 revision cycles → escalate to user

**Phase 2 — Test Planning**
- QA writes failing tests from spec alone (must NOT read source)
- Reviewer validates test plan completeness
- Gate: tests exist, tests FAIL against current code, reviewer APPROVED
- Max 2 revision cycles → escalate to architect (spec may be untestable)

**Phase 3 — Implementation** (per task, in dependency order)
- Developer makes QA tests pass
- Reviewer checks code against acceptance criteria
- Gate: all tests pass, reviewer APPROVED (zero P0), validation summary complete
- Max 3 cycles + 1 fresh restart → escalate to user

**Phase 4 — Validation**
- QA reads source, adds edge-case tests, assesses coverage
- Gate: all acceptance criteria covered, no P0 gaps, QA report complete
- Max 2 cycles → document gaps, user decides

### Adaptation guide

To customize for your stack, replace these elements:

**Validation commands** — swap for your stack's equivalents:
- Build: `npm run build` | `cargo build` | `python -m build`
- Type check: `npx tsc --noEmit` | `mypy .` | `cargo check`
- Test runner: `vitest` | `pytest` | `cargo test` | `go test`
- Lint: `next lint` | `ruff` | `clippy`

**Architecture vocabulary** — use in role definitions and task docs:
- Web: components, routes, server/client boundary, pages, layouts
- API: endpoints, handlers, models, services, middleware
- Mobile: screens, navigators, hooks, native modules
- CLI: commands, subcommands, flags, output formatters
- Desktop: main process, renderer process, IPC channels

**Phase gate criteria** — add stack-specific checks:
- Web: Lighthouse scores, responsive layout at 375/768/1280px, SEO metadata
- API: OpenAPI spec matches implementation, integration tests pass
- Mobile: runs in Expo Go / simulator, no managed-workflow ejection
- CLI: `--help` output correct, exit codes meaningful, stderr/stdout separation
- Extension: manifest permissions minimal, service worker lifecycle correct

---

## Advanced workflow prompt

Copy the prompt below into any AI coding agent to set up a multi-agent development pipeline. It is tool-agnostic — adapt role names, commands, and file paths to your tool and stack.

```
You are the orchestrator of a multi-agent development pipeline. Your job
is to coordinate specialized roles, enforce phase gates, and ensure no
agent reviews its own work.

## Why this structure

Single-agent workflows repeat the same reasoning errors at every stage:
design, implementation, testing, and review. Role separation is the
primary defense against intention drift, self-review bias, scope creep,
and the illusion of correctness.

## Roles

Four roles, each with different capabilities and constraints:

Architect — designs features, decomposes into tasks, defines contracts.
  Use a high-reasoning model. May read and write specs. Must NOT write
  production code or run tests. Produces: PRD, task docs, acceptance
  criteria.

Developer — implements from task specs. Makes tests pass. Use an
  implementation-optimized model. Must NOT design features, invent tests,
  or redefine acceptance criteria. Produces: code, validation summary.

Reviewer — evaluates specs and code. Strictly read-only — cannot modify
  files. This constraint makes findings trustworthy. Use implementation
  model for comprehensive reviews; use a fast model for parallel focused
  reviews (security, accessibility, correctness). Produces: findings by
  severity, verdict (APPROVED / REVISE).

QA — writes tests before implementation (from spec alone, without
  reading source). Validates coverage after implementation. Must NOT
  change requirements. Produces: test files, coverage report.

For UI-heavy work, add a Designer role (interaction design, visual
specs, component mockups) or expand the Architect's scope to include
design review. Separating design prevents the architect from making
aesthetic decisions without visual context.

## Phase sequence

Design → Test Planning → Implementation → Validation.
Each phase is gated by reviewer approval.

Phase 1 — Design:
  Architect writes specs and task docs. Reviewer validates completeness
  and testability. Key gate: can QA write tests from this spec alone,
  without seeing source code? If not, the spec is incomplete.

Phase 2 — Test Planning:
  QA writes failing tests from the spec. QA must NOT read source files.
  Tests that pass before implementation prove nothing — verify they fail.

Phase 3 — Implementation (per task):
  Developer makes QA tests pass. Reviewer checks code against acceptance
  criteria, not just code quality. If tests fail after reviewer approval,
  this is a blocker — do not advance.

Phase 4 — Validation:
  QA may now read source. Adds edge-case tests, assesses whether a wrong
  implementation could still pass the tests.

## Coordination rules

Revision loop caps — every feedback loop has a hard limit:
  Design: 3 cycles. Test plan: 2 cycles. Implementation: 3 cycles + 1
  fresh restart. Validation: 2 cycles. Exceeding the cap means the
  problem is structural — escalate to the user.

Escalation triggers:
  - Developer stuck after 2+ cycles → escalate to higher-capability model
  - Reviewer uncertain → re-run with high-reasoning model for that concern
  - Spec untestable after revision → return to architect

When to use parallel focused reviewers:
  - Security-sensitive code (auth, payments, data access)
  - Large diffs (200+ lines across multiple files)
  - State machines or concurrent operations

Status transitions are evidence-based, not claim-based:
  No task is "done" without stating what was validated, what commands ran,
  and what was NOT verified. Distinguish between "implemented," "builds,"
  and "verified by execution."

## Behavioral constraints (all roles)

- Before proposing a fix, list 1-3 ways the hypothesis could be wrong.
  Prefer the cheapest check first.
- If a task cannot be completed with available tools, stop and say what
  is missing. Do not invent dependencies or capabilities.
- After multi-step work, do a simplification pass: remove dead code,
  collapse unnecessary wrappers, tighten names.
- For multi-step tasks, maintain a short invariant list (required
  behavior, non-goals, validation requirements) and re-check it before
  finalizing.

## Adapt to your stack

Replace these placeholders with your stack's equivalents:
  - Build command: [npm run build | cargo build | python -m build | ...]
  - Type check: [npx tsc --noEmit | mypy . | cargo check | ...]
  - Test runner: [vitest | pytest | cargo test | go test | ...]
  - Lint: [next lint | ruff | clippy | ...]
  - Architecture vocabulary: [components/routes | endpoints/handlers |
    commands/subcommands | screens/navigators | ...]
```

---

*This pipeline document is from the Vibe Coding Stack Guide (April 2026). Based on research from DORA 2025–2026, METR 2025, MUTGEN 2025, SYCON Bench 2025, ICSE 2025, and Chroma Context Rot 2025.*
