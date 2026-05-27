# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Learn Claude Code — a progressive curriculum for building agent harnesses. The core philosophy: **the model IS the agent; this repo teaches you how to build the harness.**

## Commands

```bash
pip install -r requirements.txt   # Python dependencies (anthropic, python-dotenv, pyyaml)
python -m pytest tests/ -q         # Run all tests
python -m pytest tests/test_agents_smoke.py -q  # Smoke tests (compile-check all agent scripts)
python -m pytest tests/test_s_full_background.py -v  # Background manager unit tests
python agents/s01_agent_loop.py   # Run an individual session agent (s01-s12, s_full)
```

Requires `.env` with `ANTHROPIC_API_KEY` and `MODEL_ID`.

## Architecture

### Sessions (s01–s20)

Each `sXX_*/code.py` is a **self-contained teaching implementation** of one harness mechanism. Sessions are progressive:

| Session | Mechanism | Pattern |
|---------|-----------|---------|
| s01 | Agent Loop | `while stop_reason == "tool_use": LLM → execute → append` |
| s02 | Multiple Tools | dispatch table mapping tool names to handlers |
| s03 | Permission | deny list + user prompt for destructive commands |
| s04 | Hooks | event pipeline (PreToolUse, PostToolUse, Stop) |
| s05 | TodoWrite | structured task list tracking |
| s06 | Subagents | isolated child agent loop returning a summary |
| s07 | Skill Loading | frontmatter-parsed skill lookup from `skills/` |
| s08 | Context Compaction | micro-compact, snip, summary-based compaction |
| s09 | Memory | file-backed memory index with frontmatter entries |
| s10 | System Prompts | dynamic prompt assembly from live context sections |
| s11 | Error Recovery | retry with backoff, model fallback, reactive compact |
| s12 | Task System | file-backed tasks with dependencies + status |
| s13 | Background Tasks | thread-pool execution + notification injection |
| s14 | Cron Scheduler | 5-field cron matching + durable job persistence |
| s15 | Agent Teams | message bus (JSONL mailboxes) + teammate threads |
| s16 | Team Protocols | shutdown/plan handshake via request_id matching |
| s17 | Autonomous Agents | idle-poll → auto-claim tasks → work |
| s18 | Worktree Isolation | `git worktree` create/remove/keep per task |
| s19 | MCP Plugin | late-bound tool discovery + prefixed dispatch |
| s20 | Comprehensive | all mechanisms integrated in one loop |

### agents/ — Self-Contained Reference Implementations

`agents/s01_*.py` through `agents/s12_*.py` + `agents/s_full.py` are **standalone harness scripts** that parallel the session implementations. Each is runnable directly.

- `agents/__init__.py` — docstring-only, marks as package
- `agents/s_full.py` — capstone implementation combining all mechanisms (the full reference)

### skills/ — Loadable Skill Manifests

Each subdirectory has a `SKILL.md` with YAML frontmatter. Skills provide domain knowledge loaded on-demand:
- `agent-builder/` — agent design philosophy, templates, scaffolding
- `code-review/` — code review checklist
- `mcp-builder/` — MCP server patterns
- `pdf/` — PDF processing patterns

### docs/ — Teaching Material

Multi-language (en/zh/ja) documentation for sessions s01–s12.

### Tests

- `tests/test_agents_smoke.py` — parametrized `py_compile` across all agent scripts
- `tests/test_s_full_background.py` — unit test for `BackgroundManager` (uses module mocking for `anthropic`/`dotenv`)

### web/ — Next.js Curriculum Explorer

Next.js 16 app with Tailwind CSS v4, Framer Motion, and i18n (en/zh/ja).

```
web/src/
├── app/              # i18n-routed pages (locale-based routing)
├── components/       # (architecture, code, diff, docs, layout, simulator,
│                     #  timeline, ui, visualizations)
├── data/
│   ├── scenarios/    # s01-s20 interactive scenario steps
│   ├── annotations/  # s01-s20 code annotation overlays
│   └── generated/    # docs.json + versions.json (maps sXX → file paths)
├── i18n/messages/    # Translation strings (en.json, zh.json, ja.json)
└── lib/              # Shared utilities
```

```bash
cd web && npm ci && npm run dev   # Start curriculum web app
cd web && npm run build            # Production build (runs extract first)
```

Web build runs a content extraction step (`npm run extract` → `tsx scripts/extract-content.ts`) before Next.js build. This converts markdown docs and code files into the `web/src/data/generated/` JSON consumed by the UI.

### Data Model (web)

Version IDs (`s01`–`s20`) are the primary key across data files:
- `data/scenarios/s01.json` — array of `{type, content, annotation}` steps for the interactive walkthrough
- `data/annotations/s01.json` — code highlight annotations (line ranges, labels)
- `data/generated/versions.json` — maps version ID to file path + metadata
- `data/generated/docs.json` — extracted markdown content by version + locale

## Git Conventions

- **Commit style**: conventional commits — `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `refine:`
- **PR workflow**: merge commits on `main`, PRs from forks (external contributors)
- **Branch naming**: fix/s09-memory-turn-context, chore/update-agent-formula-main, etc.

## Important Conventions

- `.env` contains `ANTHROPIC_API_KEY` and `MODEL_ID` (see `.env.example`). Agent scripts use `python-dotenv` to load it.
- Agent scripts are designed to be **read end-to-end** — each is one file, ~80–300 lines for teaching sessions, ~2000 lines for s_full. They are not libraries; they are references.
- The `.env` supports multiple Anthropic-compatible providers (MiniMax, GLM, Kimi, DeepSeek) via `ANTHROPIC_BASE_URL`.
- CI runs Python smoke tests (compile-check) and web build. See `.github/workflows/test.yml` (Python + web) and `.github/workflows/ci.yml` (web type-check + build).
- The project requires Python 3.12+.
