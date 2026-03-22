---
title: "refactor-forge — Rewrite Software with Proof"
type: "vision"
date: "2026-03-22"
---

## North Star

refactor-forge manages language migrations and rewrites with behavioral parity testing. The agent reads the original, captures golden fixtures, writes the replacement, and proves they behave identically.

## Why This Exists

Eidos AGI has three MCP servers (visionlog, ike.md, research.md) written in TypeScript that need to become Python. The rewrite must be provably correct — not "seems to work" but "1000+ tests prove identical behavior."

This is a general problem: any time you rewrite software in a different language, framework, or architecture, you need:
1. A way to capture the original's behavior (golden fixtures)
2. A way to verify the replacement matches (replay tests)
3. A way to track what's ported and what isn't (migration map)
4. A way to ensure nothing was lost (coverage proof)

## The Pattern

```
Original (TypeScript) → Capture golden fixtures → Port to Python → 
Replay fixtures against Python → Assert identical output → 
Ship Python, rename TypeScript to .tsc backup
```

## What refactor-forge IS

- Skills for managing rewrites: capture, port, verify, migrate
- Templates for test harnesses, fixture formats, migration tracking
- A tool that any agent can use to rewrite any MCP server from any language to any other

## What refactor-forge is NOT

- Not a transpiler (no automatic code conversion)
- Not a one-time script (reusable across multiple migrations)
- Not just for TypeScript→Python (the pattern works for any A→B)

