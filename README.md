# Ralph Town — Autonomous Overnight Coding System

> Define a goal before bed. Wake up to completed work.

Ralph Town is a local-first autonomous coding system I built to run complex software engineering tasks overnight — without cloud API costs, rate limits, or manual supervision.

---

## The Problem

AI coding tools are powerful but expensive to run at scale:
- Cloud API costs compound fast during multi-hour sessions
- Rate limits interrupt long-running tasks
- You can't just "set it and go" — most tools require constant supervision

I needed a way to batch complex Magento 2 and Node.js work overnight, let it run autonomously, and wake up to verified, committed output.

---

## What I Built

A three-stage autonomous pipeline:

```
GOAL.md → Mayor Plan → Ralph Town Executor → LLM Council Verification
```

**Stage 1 — Mayor Plan**
A cloud LLM (Claude) reads a structured `GOAL.md` brief and produces a granular execution plan with scoped subtasks, constraints, and success criteria.

**Stage 2 — Ralph Town Executor**
A local model (Ollama/MLX running on-device) works through the plan step-by-step inside the target codebase. No cloud dependency — runs at zero marginal cost overnight.

**Stage 3 — LLM Council**
Multiple models independently review the output before marking any task complete. Disagreements surface for human review. Consensus required to pass.

---

## Key Design Decisions

**Local-first execution** — The executor runs on-device via Ollama. Only the planning step uses cloud APIs (~$1 per overnight session vs $20–50 for a fully cloud-based equivalent).

**Structured goal format** — `GOAL.md` defines objective, scope boundaries, success criteria, and hard constraints (e.g., "never push to remote", "exclude vendor/"). This prevents scope creep and runaway edits.

**Safety gates** — Resource guards monitor CPU and memory in real time, pausing execution if thresholds are exceeded. Prevents the system from thrashing the machine overnight.

**Council verification** — Single-model output is unreliable for production code. The LLM Council pattern uses model disagreement as a signal — if models don't agree the task is done correctly, it flags for human review instead of auto-committing.

---

## Architecture

```
┌─────────────────────────────────────┐
│           GOAL.md Brief             │
│  (objective, scope, constraints)    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│         Mayor Plan (Cloud)          │
│  Claude API → structured task list  │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│      Ralph Town Executor (Local)    │
│  Ollama/MLX → step-by-step coding   │
│  Resource guard + safety gates      │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│       LLM Council Review            │
│  Multi-model verification           │
│  Consensus required to pass         │
└─────────────────────────────────────┘
```

---

## Results

- Runs Magento 2 and Node.js tasks overnight with no supervision
- ~95% cost reduction vs fully cloud-based sessions for equivalent output
- Safety gates have prevented runaway processes on 100% of overnight runs
- Council verification catches model errors before they reach git

---

## Tech Stack

- **Execution**: Ollama + MLX (Apple Silicon local inference)
- **Planning**: Claude API (Anthropic)
- **Verification**: LLM Council (multi-model consensus)
- **Languages**: Bash, Python, JavaScript
- **Target codebases**: Magento 2 (PHP), Node.js

---

## Status

Active and in use. Source code is private.

For questions or collaboration: [joe.sebastian6079@gmail.com](mailto:joe.sebastian6079@gmail.com)
