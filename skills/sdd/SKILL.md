---
name: sdd
description: Set up Spec-Driven Development (SDD) — a durable, file-based source of truth for product intent, architecture decisions, execution status, and deferred follow-ups. Use this skill whenever the user says "spec-driven", "SDD", "feature spec", "task docs", "spec folder", "PRD template", "feature tracking", "how do I organize features", or wants structured artifacts to keep an AI agent on track across sessions. Apply it for the full lifecycle — folder structure, frontmatter, task-doc template, status transitions, and when to defer vs. close. SDD is the substrate the multi-agent `pipeline` skill runs on, but it's useful on its own.
---

# Spec-Driven Development (SDD)

This skill creates and maintains the durable artifacts that keep a vibe-coded project aligned across many agent sessions. Feature specs are the source of truth for what the product does, why it does it, where execution stands, and which follow-ups are intentionally deferred.

Use SDD any time the feature is bigger than one session can hold, any time multiple agents need to hand off work, or any time the user has already felt the pain of "the agent forgot what we agreed on."

## Folder structure

```
specs/
  <feature-slug>/
    README.md            # entrypoint; current state + doc map
    prd.md               # requirements, scope, architecture
    design.md            # optional: larger or cross-cutting work
    tasks/
      001-task-name.md
      002-another-task.md
    artifacts/
      backlog.md         # deferred work
      test-plan.md       # optional, when QA needs one
```

Conventions:

- **Kebab-case slugs.** `add-checkout` not `AddCheckout`.
- **`README.md` is the entrypoint.** It links to everything else in the folder.
- **One task per file.** Each task file is independently implementable and verifiable in a single commit.
- **Deferred work lives in `artifacts/backlog.md` only.** Not scattered across task files.

## Required frontmatter

Every `prd.md`, `design.md`, and task file starts with:

```yaml
---
title: Example Title
status: draft            # draft | active | blocked | review | implemented | abandoned
priority: high           # high | medium | low
owner: shared            # ai | human | shared
last_updated: 2026-01-01
depends_on: []           # list of task slugs this task depends on
---
```

`depends_on` is what lets an orchestrator topologically sort tasks in Phase 3 of the multi-agent pipeline.

## Task document template

Each task doc uses these sections, in order. Sections marked optional can be omitted when they don't apply.

1. **Summary** — one paragraph: what this task does and why.
2. **Scope** — bulleted deliverables.
3. **Non-goals** (optional) — explicit exclusions. Useful when scope is ambiguous.
4. **Affected files** — a table: file, change type (new / edit / delete), purpose. Helps reviewers verify scope.
5. **Acceptance criteria** — markdown checklists. Concrete enough that QA can write tests from this alone, without reading source.
6. **Now / Implemented** — progress tracking during implementation. Checklist that the Developer ticks as it goes.
7. **Backlog links** (optional) — references to deferred items in `artifacts/backlog.md`.
8. **Risks / blockers** (optional) — dependencies, unknowns, or decisions that may derail the task.
9. **Validation** — specific commands to run, pages to check, behaviors to confirm. This is the evidence that lets the task move to `implemented`.
10. **Follow-ups** (optional) — out-of-scope improvements discovered during the work; added to the backlog.

## Status transitions (evidence-based)

Status never advances on self-review alone. It advances on concrete evidence.

| Transition | Evidence required |
|---|---|
| `draft` → `active` | Requirements and scope are agreed upon |
| `active` → `review` | Code changes complete, not yet validated |
| `review` → `implemented` | Acceptance criteria validated with named checks (tests passing, commands run, manual verification logged) |
| Any → `blocked` | A dependency or prerequisite is missing — state what |
| Any → `abandoned` | Decision not to proceed — state why |

If validation evidence is missing, the task stays in `review`. Don't let the agent mark its own work `implemented`.

## SDD workflow

1. **Open.** Create `specs/<feature-slug>/README.md` and `prd.md`. Status: `draft`. The README starts as a short doc map.
2. **Design.** Add `design.md` for larger or cross-cutting work. Define routes, components, data shape, contracts. This is where the Architect lives in the multi-agent pipeline.
3. **Plan.** Create numbered task docs under `tasks/`. Each independently implementable, independently verifiable, one commit. Set `depends_on` where real dependencies exist.
4. **Implement.** Update the task's "Now / Implemented" checklist as work progresses. Advance status to `review` when code is done but not yet validated.
5. **Defer.** Any out-of-scope improvement goes to `artifacts/backlog.md`, not into the task file. Task files stay focused.
6. **Close.** Move `review` → `implemented` only after acceptance criteria have named validation evidence.

## When to add `design.md`

Add `design.md` when the feature spans multiple components, introduces a new pattern, or has non-obvious architecture. Skip it for narrow features where the PRD is enough — no ceremony for its own sake.

## Integration with the multi-agent pipeline

If the user is running the `pipeline` skill, SDD is the substrate:

- **Phase 1 (Design)** produces `prd.md` + numbered task docs.
- **Phase 2 (Test Planning)** reads the spec only; outputs live under `artifacts/test-plan.md` or alongside code as failing tests.
- **Phase 3 (Implementation)** moves tasks through `active` → `review`.
- **Phase 4 (Validation)** advances tasks to `implemented` when evidence is present.

## Without the pipeline, still useful

Even in a solo-agent workflow, SDD solves three concrete problems:

- **Cross-session memory.** The agent re-reads the spec at the start of every conversation and stays aligned.
- **Deferred work isn't lost.** `artifacts/backlog.md` is where good ideas go to survive.
- **Shippable features, not in-progress rewrites.** Tasks have clear acceptance criteria and can be marked done only with evidence.

## Handing off to other skills

- Running multi-agent development on top of these specs → `pipeline` skill.
- Stack-specific scaffolding inside a task's Implementation phase → the path skills (`web-app`, `api-backend`, `ai-agent`, etc.).
- Agent struggling to make progress on a task → `red-flags` skill.
