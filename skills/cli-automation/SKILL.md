---
name: cli-automation
description: Scaffold and iterate on a vibe-coded CLI tool, automation script, cron job, scraper, or data pipeline using Python 3.13+ + uv + ruff + mypy + pytest (Typer for CLI args, pandas for tabular data, requests for HTTP). Use this skill whenever the user says "CLI", "script", "automation", "cron", "scraper", "data pipeline", "ETL", "command-line tool", "batch job", or describes a task-runner/one-shot program without a UI. Apply it for the full lifecycle — initial script, tests, packaging, scheduling — not just day one. If the task is mainly LLM orchestration, route to `ai-agent` instead.
---

# CLI / Automation / Data Pipeline — Vibe Coding Path

This skill is for helping an experienced developer ship a CLI tool, script, or data pipeline, mostly with an AI coding agent. This is the highest-probability category for vibe coding — Python resolves the most tasks, the feedback loop is instant, and deployment is trivial.

**Default recommendation, no hedging:** Python 3.13+ + `uv` for packaging + `ruff` for lint/format + `mypy` for types + `pytest`. Add `typer` for CLI parsing, `pandas` for tabular data, `requests` or `httpx` for HTTP — only when each is actually used.

## Why this stack (use only when asked)

Python resolves ~52% of tasks on [Multi-SWE-bench](https://arxiv.org/html/2504.02605) vs. 22% for Java and single digits for JS/TS — no other language is close for agent-generated code. The feedback loop is instant: run the script, see output. Deployment is trivial (`uv tool install` or scp the file). [Anthropic's own usage data](https://www.anthropic.com/research/impact-software-development) puts automation as the #1 category for AI-assisted coding.

Don't default to Go or Rust unless static binaries / raw throughput are hard requirements — you trade the highest agent fluency for benefits first versions rarely need, and Rust compile errors are unreadable without Rust expertise. Don't reach for Airflow / Dagster / Prefect on day one — start with a script that works, add orchestration when scheduling is a real problem.

## Scaffold workflow

1. `uv init <project>`, `cd <project>`, `uv add typer`, `uv add --dev pytest ruff mypy`.
2. Single `main.py` with one function that does the core thing, called from `if __name__ == "__main__": typer.run(main)`.
3. Add one `pytest` test that exercises the core function directly (not via the CLI) — keeps tests fast.
4. Verify: `uv run python main.py --help`, `uv run python main.py <args>`, `uv run pytest -v`.
5. Only add `pandas` / `httpx` / `rich` / etc. when the code actually uses them — one line of justification per addition.

## First prompt to hand the user

```
I'm building a [CLI tool / automation script / data pipeline] that
[describe what it does in one sentence].

Stack: Python 3.13+, uv, ruff, mypy, pytest. Typer for CLI argument
parsing (if CLI). pandas for tabular data, requests for HTTP.

Apply these stack-specific constraints:

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

## Constraints (enforce on every response)

- **One core function called from `__main__`.** Runnable with `uv run python main.py` — no extra setup beyond `uv sync`.
- **stderr for progress, stdout for results.** Use the standard `logging` module from the start so output is pipe-friendly.
- **No classes unless the data model actually requires them.** Functions and dataclasses cover most pipelines.
- **No orchestration frameworks** until scheduling is a real problem. A cron line or GitHub Actions schedule is usually enough at first.
- **Every response includes a command to run and verify output** — ideally with a small sample file or fixture.
- **Keep files under 300 lines.** Split when exceeded, usually into `cli.py`, `core.py`, `io.py`.

## Verification checklist

```bash
uv run python main.py --help         # usage info
uv run python main.py [your args]    # expected output on stdout
uv run pytest -v                     # tests pass
uv run ruff check . && uv run ruff format --check .   # lint / format
uv pip install -e .                  # installable as a console script
```

## Stop signals

- More than 5 files and still no tests.
- Agent adds an ORM for what should be a CSV transform.
- More `import` lines than lines of logic.
- Agent proposes Docker for a script that reads local files.
- Error handling is more complex than the happy path.

## Escape routes

- **Script outgrew "just a script":** add a small web UI with FastAPI + one page (`api-backend`) before reaching for an orchestrator.
- **Real scheduling problem arrives:** then graduate to a scheduler. Don't pre-adopt Airflow.
- **Handing off:** Python CLIs are the easiest code to hand off to another developer — almost any Python developer can pick it up.

## Handing off to other skills

- Scraper becomes a whole backend → `api-backend`.
- Need to orchestrate LLM calls → `ai-agent`.
- Something feels stuck → `red-flags`.
