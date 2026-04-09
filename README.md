# Ralph Town — Autonomous Overnight Coding System

## Overview

Ralph Town is a locally-run autonomous coding system designed to handle complex engineering tasks without cloud dependencies or human supervision during execution. Feed it a PRD, and a local LLM decomposes it into phased subtasks — review and fix bugs in the morning.

**Problem Addressed:** Interactive AI coding sessions incur continuous cloud API costs and require babysitting, whereas overnight batch work needs no such constraints.

**Workflow:** A structured GOAL.md file triggers a "Mayor Plan" that decomposes tasks into phases, each executing in isolated Git worktrees with RAG-injected codebase context.

**Early validation across 4 module PRDs: 50–65% of generated code passed initial review with minor fixes, running entirely on local hardware with zero API cost.**

---

## How It Works

```
GOAL.md (PRD)
     │
     ▼
┌─────────────┐
│  Mayor Plan │  ← Decomposes PRD into phased subtasks
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Git Worktree (x N) │  ← Each phase runs in isolation
│  + RAG Context      │  ← nomic-embed-text injects codebase context
│  + Qwen3-8B (local) │  ← vLLM-MLX, no cloud calls
└──────┬──────────────┘
       │
       ▼
┌──────────────────────┐
│  Verification Layer  │
│  • CoVe (5 questions)│
│  • PHPStan analysis  │
│  • Safety rails      │
└──────┬───────────────┘
       │
       ▼
┌──────────────────┐
│  /ralph-review   │  ← Morning review: diff, approve, commit
└──────────────────┘
```

---

## Core Stack

| Component | Tool |
|-----------|------|
| Inference | vLLM-MLX + Qwen3-8B (local) |
| Embeddings | nomic-embed-text |
| Static Analysis | PHPStan |
| Isolation | Git Worktrees |
| Orchestration | Bash + Python |
| Containers | Docker |

---

## Quality & Safety Mechanisms

- **Chain of Verification (CoVe):** Self-interrogation loop with five verification questions per output before acceptance
- **Static Analysis:** PHPStan automatically validates PHP code and feeds errors back into retry loops
- **Safety Rails:** Critical module detection, change ratio limits, dangerous pattern scanning, automatic file backups, and branch protection

---

## Human-in-the-Loop Review

Rather than auto-committing, the system stages all changes for morning review via `/ralph-review` — a Claude Code skill that diffs changes, compares planned vs. completed work, and requires explicit approval before applying anything to the main branch.

No overnight supervision needed. Just review in the morning.

---

## Example GOAL.md (sanitized)

```markdown
# GOAL.md — User Auth Module

## Objective
Implement JWT-based authentication for the admin panel.

## Phases
1. Create AuthService class with login/logout/refresh methods
2. Add middleware for route protection
3. Write PHPStan-compatible type hints throughout
4. Add unit test stubs for each public method

## Constraints
- Do not modify existing UserModel
- All new files go in /src/Auth/
- Follow existing PSR-12 coding standards
```

---

## Design Decisions

- **Local inference over cloud APIs** — slower but zero per-token cost; overnight batch work makes latency irrelevant
- **Semantic retrieval over full-context loading** — RAG injects targeted codebase context per phase rather than overloading the context window
- **Verification loops** — catch bad outputs (CoVe flags ~2–3 per overnight session) before human review, not after

---

## Current Status

Actively deployed on Magento 2 and Node.js projects (source code private — active development). Built and maintained by [Joe Sebastian](https://github.com/joesebastian6079).
