---
layout: default
title: Context & Bootstrap Optimization
description: How we audited and reduced OpenClaw prompt overhead, bootstrap caps, and compaction behavior
nav_order: 6
---

# Context & Bootstrap Optimization

OpenClaw injects a lot of context into every message — workspace files, system prompts, runtime headers, memory logs. When you're running a 27B model locally, that overhead adds up fast and eats into your usable context window.

## The Defaults

OpenClaw caps bootstrap content at **20,000 tokens per message** and **60,000 tokens total** across all injected messages. These are the hard defaults — you can trim them further.

## What We Audited

We went through every workspace file and trimmed fat:

- **AGENTS.md** — removed redundant explanations, kept only operational rules
- **HEARTBEAT.md** — stripped verbose examples, kept job definitions
- **TOOLS.md** — removed commentary, kept only command references
- **SOUL.md** — already tight, minor prose cleanup

The goal was keeping the signal (how the agent works, what the rules are) while cutting the noise (repeated explanations, verbose examples).

## Compaction

OpenClaw's QMD (quantized message distillation) compaction works well out of the box. It trims old conversation history while preserving key decisions and context. At 16% token usage (21k/131k), it was holding steady without manual intervention.

If you're hitting context walls, you can lower the bootstrap caps further:

- Per-message cap: 20k → 10k
- Total cap: 150k → 60k

These go in your OpenClaw config under the QMD/bootstrap settings.

## Key Takeaways

- Every workspace file counts toward the bootstrap budget. Trim ruthlessly.
- Compaction handles old conversation history. Don't manually compact unless you're hitting hard limits.
- The 20k per-message default is generous for most setups. If you're running small models locally, go lower.
- Use the `openclaw status` command to check current token usage in a session.

## Links

- [OpenClaw docs](https://docs.openclaw.ai)
- [GitHub source](https://github.com/openclaw/openclaw)
