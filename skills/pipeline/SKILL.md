---
name: pipeline
description: Set up and operate a multi-agent development pipeline with separated roles — Architect, Developer, QA, Reviewer (and optionally Designer) — so no agent reviews its own work. Use this skill whenever the user says "pipeline", "multi-agent", "agent pipeline", "separated roles", "orchestrator", "code review workflow", "AGENTS.md setup", "architect / developer / reviewer / QA split", or describes wanting more rigor than a solo-agent session can provide. Apply it end-to-end: role definition, phase gates, revision caps, orchestrator decisions, and escalation. This is the Level 2 structure — if the user hasn't hit the pain of solo-agent drift yet, recommend a simple rules file (CLAUDE.md / AGENTS.md) first.
---

# Multi-Agent Development Pipeline

This skill is for setting up Level 2 agent structure — four specialized roles that hand off work through gated phases. It addresses the failure modes a single-agent workflow can't escape: intention drift, self-review bias, scope creep, and the illusion of correctness.

**When to actually use it:** most projects don't need this. Recommend Level 1 (a rules file: `CLAUDE.md` / `AGENTS.md` / `.cursorrules`) first and only escalate to the pipeline when the user has concrete symptoms (listed at the end). DORA 2025–2026 confirmed AI is an amplifier — strong process + AI accelerates; weak process + AI destabilizes.

## The four roles

| Role | Model tier | Tools | Produces | Must NOT |
|---|---|---|---|---|
| **Architect** | reasoning-strong (e.g., Opus) | read/write for specs | PRD, task docs, acceptance criteria | Write production code; run tests |
| **Developer** | implementation (e.g., Sonnet) | all tools | Code, validation summary | Design features; invent tests; redefine acceptance criteria |
| **Reviewer** | Sonnet or Haiku | **read-only** | Findings by severity, verdict | Modify any files |
| **QA** | Sonnet | all tools | Test files, coverage report | Change requirements (in Phase 2, also must NOT read source) |

Optional: **Designer** for UI-heavy work (interaction specs, mockups, visual guidelines).

The separation is the point. Read-only enforcement on the Reviewer is what makes its verdicts trustworthy. Blind test-authoring by QA (Phase 2) is what keeps tests from encoding the same bugs as the implementation.

## The four phases

Every phase is gated by Reviewer approval. A phase only advances on APPROVED.

1. **Design.** Architect writes `specs/<feature>/prd.md` and numbered task docs under `tasks/`. Reviewer validates. Exit gate: APPROVED, schemas and contracts locked, testable acceptance criteria. Max 3 cycles.
2. **Test planning.** QA writes failing tests **from the spec only**. Must NOT read source. Reviewer validates completeness. Exit gate: tests exist AND fail. Max 2 cycles.
3. **Implementation** (per task, topological on `depends_on`). Developer makes QA tests pass, scoped to one task / one commit. Reviewer checks code vs. acceptance criteria. Exit gate: tests pass, APPROVED, zero P0. Max 3 cycles + 1 fresh restart.
4. **Validation.** QA is now allowed to read source. Adds edge-case tests, assesses mutation-awareness (would a wrong implementation still pass?). Exit gate: all acceptance criteria covered, no P0 gaps.

## Phase-boundary prohibitions

| Phase | Must NOT |
|---|---|
| 1 Design | Write production code or tests |
| 2 Test planning | Read source files; assume an implementation |
| 3 Implementation | Invent tests; redefine acceptance criteria |
| 4 Validation | Change requirements; modify production code |

## Universal behavioral constraints (all roles)

- **Falsify-first.** Before proposing a fix, list 1–3 ways the hypothesis could be wrong. Cheapest disambiguating check first.
- **Solvability gate.** If a task can't be completed with available tools, stop and say exactly what's missing. Don't invent package names or framework capabilities.
- **Simplification pass.** After multi-step work, remove dead branches, collapse unnecessary wrappers, tighten names. No scaffolding survives without justification.
- **Invariant ledger.** Maintain a short invariant list; re-check before finalizing.
- **Evidence-based transitions.** "Done" = what was validated, what commands ran, what was explicitly NOT verified.

## Revision-cycle caps

| Phase | Cap | On exceeded |
|---|---|---|
| Design | 3 | Escalate to user: accept trade-offs, split the feature, or abandon |
| Test planning | 2 | Escalate to Architect: the spec may be untestable |
| Implementation | 3 + 1 restart | Split the task, defer, or escalate to a stronger model |
| Validation | 2 | Document gaps; user decides whether to proceed |
| Hotfix variant | 1 | Rollback or downgrade to the normal pipeline |

## Orchestrator decision rules

- **Parallel reviewers** for security-sensitive code, large diffs (>200 lines), or state machines. Single reviewer for small changes (<100 lines), routine CRUD, config/docs.
- **Loop on P0 always.** Loop on P1 unless the Developer provides explicit justification for deferral. **Proceed on P2** — track in `artifacts/backlog.md`.
- **Escalate model tier** when the Reviewer is uncertain, or when the Developer has burned 2+ revision cycles on one task.
- **Respect user intervention.** If the user modifies the spec mid-implementation, restart Phase 1 for the affected scope rather than patching mid-flight.

## Hotfix variant

For urgent production fixes: skip Phases 1–2. Developer writes a failing test → minimal fix → single Reviewer pass → deploy → backfill spec + QA coverage afterwards. Cap at one revision cycle; beyond that, roll back.

## Status tracking template

Use this at the start of every orchestrator turn so the user always knows where the pipeline is.

```
## Pipeline Status: [feature-slug]
- Phase: [1-Design | 2-TestPlan | 3-Implementation | 4-Validation]
- Current task: [number / total]
- Revision cycle: [number]
- Blocking findings: [count or "none"]
- Next action: [which agent runs next and why]
```

## Orchestrator prompt (paste into any AI coding tool)

```
You are the orchestrator of a multi-agent development pipeline. Coordinate
specialized roles, enforce phase gates, and ensure no agent reviews its own
work.

Roles: Architect (design, specs), Developer (implement, make tests pass),
Reviewer (read-only evaluation), QA (tests before implementation, validation
after).

Phases: Design → Test Planning → Implementation → Validation.
Each gated by reviewer approval.

Loop caps: Design 3, Test Plan 2, Implementation 3+1, Validation 2.
Exceed the cap → escalate to the user.

All roles operate under: falsify-first, evidence-based transitions,
simplification pass, solvability gate, invariant ledger.
```

## When to escalate from Level 1 (rules file) to Level 2 (this pipeline)

Upgrade only when one of these is persistent, not a single bad session.

- The agent keeps forgetting context despite a rules file — constraints don't stick.
- One session can't contain a change — the feature touches enough files that the full diff can't be verified in one conversation.
- You don't trust the agent to make changes without regressing something.
- The agent goes in circles on self-review ("looks good") and misses obvious issues.

Most projects never hit these. If you do, you'll know — the symptoms are unmistakable.

## Handing off to other skills

- Spec folder / task-doc conventions → `sdd` skill (required substrate for this pipeline).
- Diagnostic help when the pipeline is stalling → `red-flags` skill.
- Stack-specific scaffolding happens inside Phase 3 via the path skills (`web-app`, `api-backend`, etc.).

*Grounded in DORA 2025–2026, METR 2025, MUTGEN 2025, SYCON Bench 2025, ICSE 2025, and Chroma Context Rot 2025.*
