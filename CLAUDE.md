# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin implementing the ALMAS (Autonomous Learning and Multi-Agent System) framework. It maps agile team roles to LLM agents to automate the full software development lifecycle. There is no compiled code — the entire system is markdown-based agent prompts and skill definitions.

## Usage

Load as a plugin from a target project:
```bash
claude --plugin-dir /path/to/sdlc
```

Skills: `/sdlc:develop <task>`, `/sdlc:plan <task>`, `/sdlc:test`, `/sdlc:review`, `/sdlc:summarize`

## Architecture

**7 agents** orchestrated by a Supervisor in a sequential pipeline:

```
Supervisor (opus) — orchestrates everything via Task tool
  ├── Summary Agent (sonnet) — generates/updates .sdlc/summaries/
  ├── Sprint Agent (sonnet) — decomposes task into sub-tasks
  ├── Control Agent (sonnet) — localizes code via Meta-RAG
  ├── Developer Agent (opus) — implements changes + writes tests
  ├── Test Agent (sonnet) — detects infrastructure, runs tests, reports results
  └── Peer Agent (opus) — 6-point code review
```

**Pipeline flow**: Summaries → Sprint Plan → [per sub-task: Localize → Implement → Retry if needed] → Test → Review → Update Summaries → Final Report

**Key design decisions**:
- Agents are stateless — each receives full context in its Task prompt
- Model selection: opus for complex tasks (Supervisor, Developer, Peer), sonnet for lighter tasks (Summary, Sprint, Control, Test)
- All agents are launched as `subagent_type: "general-purpose"` with agent-specific prompts from `agents/*.md`
- Approval gates (via `AskUserQuestion`) are mandatory at: sprint plan, code localization, and test plan stages
- The pipeline never auto-commits — user stays in control
- Developer Agent failures trigger Control Agent re-localization (up to 3 retries)

**Meta-RAG** (used by Control Agent): 3-level hierarchical retrieval from `.sdlc/summaries/` — file-level → function-level → dependency tracing. Falls back to direct Glob/Grep if summaries are unavailable.

## File Layout

- `agents/*.md` — Agent system prompts with YAML frontmatter (model, tools). These are the core logic.
- `skills/*/SKILL.md` — User-facing skill definitions that wire skills to agents.
- `.claude-plugin/plugin.json` — Plugin metadata (name, version, author).
- `scripts/collect-summaries.sh` — Utility for extracting summaries.

## Editing Conventions

- Agent prompts use YAML frontmatter to declare `model` and `tools` list.
- Approval gates use `AskUserQuestion` — never skip them in Supervisor logic.
- The Peer Agent's 6-point checklist: Functionality, Vulnerabilities, Performance, Hallucination Detection, Code Quality, Test Coverage (+ Documentation Sync).
- `.sdlc/` directory is generated in the target project (gitignored) — never in this repo.
