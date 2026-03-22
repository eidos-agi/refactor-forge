---
id: "GUARD-002"
type: "guardrail"
title: "Never ship without behavioral parity proof"
status: "active"
date: "2026-03-22"
---

## Rule
A rewrite is not done until every golden fixture from the original passes against the replacement. "It seems to work" is not proof. 100% fixture replay pass rate is the bar.

## Why
The whole point of refactor-forge is provable correctness. Shipping without proof defeats the purpose.

## Violation Examples
- Marking a migration as complete with failing fixtures
- Skipping fixture capture for "simple" tools
- Accepting a rewrite that only passes 95% of fixtures
