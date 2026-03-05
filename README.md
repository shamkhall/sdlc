# SDLC — Multi-Agent Development Pipeline for Claude Code

A Claude Code plugin that maps agile team roles to LLM agents, covering the full software development lifecycle. Inspired by the [ALMAS framework](https://arxiv.org/abs/2510.03463).

## Quick Start

```bash
cd /path/to/your-project
claude --plugin-dir /path/to/sdlc
```

Then inside Claude Code:

```
/sdlc:develop Add user authentication with JWT tokens
```

## Commands

| Command | Description |
|---------|-------------|
| `/sdlc:develop <task>` | Full pipeline: plan → localize → implement → review |
| `/sdlc:plan <task>` | Break a task into sub-tasks with acceptance criteria |
| `/sdlc:review` | 6-point code review on uncommitted changes |
| `/sdlc:summarize` | Generate natural-language summaries of the codebase |

## How It Works

`/sdlc:develop` launches a **Supervisor Agent** that orchestrates five specialized agents:

```
User Task
  │
  ▼
┌─────────────────┐
│  Summary Agent   │  Scans codebase, generates NL summaries
└────────┬────────┘
         ▼
┌─────────────────┐
│  Sprint Agent    │  Breaks task into sub-tasks + acceptance criteria
└────────┬────────┘
         ▼
   ┌─────────────────────────┐
   │  For each sub-task:     │
   │                         │
   │  ┌─────────────────┐   │
   │  │  Control Agent   │   │  Localizes relevant code (Meta-RAG)
   │  └────────┬────────┘   │
   │           ▼             │
   │  ┌─────────────────┐   │
   │  │  Developer Agent │   │  Implements changes + writes tests
   │  └────────┬────────┘   │
   │           │             │
   │    (retry up to 3x      │
   │     on test failure)    │
   └─────────────────────────┘
         ▼
┌─────────────────┐
│   Peer Agent     │  6-point code review
└─────────────────┘
```

## Agents

| Agent | Role | Model |
|-------|------|-------|
| **Supervisor** | Pipeline orchestrator | opus |
| **Summary** | Codebase summarizer | sonnet |
| **Sprint** | Task decomposition + planning | sonnet |
| **Control** | Code localization (Meta-RAG) | sonnet |
| **Developer** | Implementation + testing | opus |
| **Peer** | Code review (6-point checklist) | opus |

### Peer Review Checklist

The Peer Agent evaluates changes against six criteria:

1. **Functionality Alignment** — Do changes meet acceptance criteria?
2. **Vulnerability Check** — Any security issues?
3. **Performance** — N+1 queries, memory leaks, etc.?
4. **Hallucination Detection** — References to non-existent APIs or methods?
5. **Code Quality** — Naming, structure, DRY, SOLID
6. **Test Coverage** — Are tests sufficient?

## Codebase Summaries

The Summary Agent generates hierarchical natural-language summaries stored in `.sdlc/summaries/` inside your project. These summaries enable the Control Agent to quickly localize relevant code without reading every file.

```
.sdlc/summaries/
├── src/
│   ├── services/
│   │   └── auth.ts.summary.md
│   └── routes/
│       └── auth.routes.ts.summary.md
└── ...
```

Summaries are created on first run and incrementally updated on subsequent runs. Add `.sdlc/` to your `.gitignore` if you don't want to track them.

## File Structure

```
sdlc/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   ├── supervisor.md       # Pipeline orchestrator
│   ├── summary.md          # Codebase summarizer
│   ├── sprint.md           # Task decomposition
│   ├── control.md          # Code localizer (Meta-RAG)
│   ├── developer.md        # Code implementation
│   └── peer.md             # Code reviewer
├── skills/
│   ├── develop/SKILL.md    # /develop — full pipeline
│   ├── plan/SKILL.md       # /plan — planning only
│   ├── review/SKILL.md     # /review — review only
│   └── summarize/SKILL.md  # /summarize — summaries only
└── scripts/
    └── collect-summaries.sh
```

## Error Handling

- If the Sprint Agent finds the task ambiguous, it will ask clarification questions before proceeding.
- If the Developer Agent's tests fail, the Supervisor retries up to 3 times with re-localization.
- If the Peer Agent verdict is "NEEDS CHANGES", action items are reported for you to decide on.
- The pipeline never commits to git — you stay in control.

## License

MIT
