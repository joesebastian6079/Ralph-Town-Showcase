# Ralph Town

## The Problem

AI coding tools are powerful in short bursts — but complex engineering tasks (refactors, module builds, migrations) take hours of back-and-forth. Running these interactively means paying cloud API rates for every token, babysitting the session, and losing the time you could be sleeping.

I needed a way to hand off a product requirement at night and wake up to working code.

## The Solution

An autonomous overnight coding system: feed it a PRD (`GOAL.md`), and a local LLM decomposes it into phased subtasks, executes each phase with RAG-powered codebase context, verifies output through a self-checking quality layer, and applies safety rails before touching anything critical.

No cloud dependency during execution. No supervision required.

---

**Tech Stack:** Bash – Python – vLLM-MLX – Qwen3-8B – nomic-embed-text – PHPStan – Docker – Git Worktrees

---

## Why I Built This

I run a Magento 2 e-commerce project alongside a Node.js bot platform. Both have large, interconnected codebases where a single bad change cascades. I needed a coding assistant that could:

- Work through a multi-hour task without me watching
- Understand the codebase structure before writing code (not just guess)
- Catch its own mistakes before I reviewed in the morning
- Never touch critical files without explicit scope

Existing tools either required cloud APIs (expensive at scale), didn't maintain codebase context across phases, or had no safety mechanisms for autonomous runs.

So I built something that did all of it locally.

## How It Works

```
GOAL.md (PRD)
     │
     ▼
Mayor Plan — decomposes the PRD into phased subtasks with scoped git worktrees
     │
     ▼
RAG Pipeline — indexes codebase structure via nomic-embed-text,
               injects targeted context into each worker iteration
     │
     ▼
vLLM-MLX Executor (Qwen3-8B) — executes each phase locally, overnight
     │
     ▼
Chain of Verification (CoVe) — 5-question self-verification loop per output
PHPStan Static Analysis — error-fed retry loops until clean
     │
     ▼
Safety Rails — scan before committing anything
```

## System Architecture

**PRD-to-Plan decomposition**
The system reads a structured `GOAL.md` brief (objective, scope, constraints, success criteria) and decomposes it into phased subtasks. Each phase gets its own git worktree — isolated branches prevent phases from contaminating each other.

**RAG-powered context**
A custom Python RAG pipeline using `nomic-embed-text` indexes the codebase before execution starts. Each worker iteration gets relevant context injected — not just file names, but semantically matched code sections. This significantly improves generation accuracy on large, unfamiliar codebases.

**Local inference**
Execution runs entirely on-device via `vLLM-MLX` with `Qwen3-8B`. No cloud API calls during the overnight run. Planning uses Claude API (~$1/session); execution is free.

## Quality & Safety

**Chain of Verification (CoVe)**
Every generated output goes through a 5-question self-verification loop before being accepted. The model interrogates its own output — checking logic, scope adherence, edge cases — and retries if verification fails.

**PHPStan static analysis**
For PHP codebases (Magento 2), PHPStan runs automatically after each phase. Errors are fed back into the retry loop. The system doesn't move to the next phase until static analysis passes.

**Production safety rails**
Before any write operation, the safety layer checks:
- Critical module detection (core files, payment modules, auth)
- Change ratio limits (flags if a phase modifies too large a % of the codebase)
- Dangerous pattern scanning (DROP TABLE, rm -rf, etc.)
- Automatic file backups before overwrites
- Branch protection — never commits to main without explicit approval

## Morning Review — `/ralph-review`

The overnight system deliberately doesn't auto-commit. In the morning, a single command hands off to regular Claude (cloud) for human-in-the-loop review before anything lands in git.

**`/ralph-review`** is a custom Claude Code skill that:
1. Reads `plan.json` and `completed.json` — shows what the goal was and what got done
2. Runs `git diff --stat` and `git log` — surfaces all changed files
3. Lists `.proposed` files (critical-module changes held for approval) and active git worktrees
4. Walks through each diff, asking for sign-off before moving on
5. On approval — applies staged changes, prunes stale worktrees, cleans up temp files, and commits

The local model (Ralph) is explicitly blocked from running `/ralph-review` on itself. The review step only runs when a human opens Claude Code in the morning and invokes it.

This keeps the human in the loop on every overnight run without requiring them to watch it happen.

---

## Tradeoffs & Decisions

**Local inference over cloud for execution** — Qwen3-8B via vLLM-MLX is slower than GPT-4 but runs at zero marginal cost. For overnight batch work, speed doesn't matter. Cost does.

**Git worktrees over branches** — Each phase in an isolated worktree means failures in phase 3 don't affect the committed state from phase 1 or 2. Rollback is clean.

**RAG over full-context loading** — Full codebase context exceeds local model context windows. Semantic retrieval gives the model what it actually needs per-task rather than flooding it with irrelevant code.

**CoVe over external reviewer** — Self-verification is slower than just committing, but it catches a surprisingly high number of off-scope or logically broken outputs before they hit the review queue.

## What I Learned

– Local models are good enough for scoped, well-specified tasks. The bottleneck isn't model capability — it's context quality. The RAG pipeline mattered more than the model choice.

– Safety rails are load-bearing, not optional. The first version had no change ratio limits. It once modified 60% of a module's files in a single phase because the scope in GOAL.md was ambiguous.

– Verification loops add time but reduce morning review time by more than they cost. The CoVe layer typically catches 2–3 bad outputs per overnight session that would otherwise land in my review queue.

– Decomposing PRDs into phased subtasks is the hardest part of the system — not the execution. Ambiguous goals produce bad plans. The GOAL.md format exists to force specificity before the session starts.

## Status

Active and in use on Magento 2 and Node.js projects. Source code is private.

Built by [Joe Sebastian](https://linkedin.com/in/joesebastian) — PM/PgM building at the intersection of product and engineering.
