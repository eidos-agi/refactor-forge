# CLAUDE.md — refactor-forge

> Rewrite software with proof. Language migrations with behavioral parity testing.

## What This Is

refactor-forge manages rewrites from any language to any other language. The agent captures golden fixtures from the original, ports the code, and proves the replacement behaves identically.

The pattern: capture → port → verify → migrate.

## Skills

| Skill | What It Does |
|-------|-------------|
| `/refactor-capture` | Capture golden fixtures from the original — run every tool, record inputs/outputs/side effects |
| `/refactor-port` | Port a single tool from source to target language, matching behavior exactly |
| `/refactor-verify` | Run all fixtures against the port, report pass/fail with diffs |
| `/refactor-migrate` | Full cutover: rename original, install replacement, update references |

## Guardrails

1. **No software.** Skills and templates only.
2. **Never ship without behavioral parity proof.** 100% fixture pass rate is the bar.

## Related Forges

- **ship-forge** — shipping standards for the ported package
- **foss-forge** — FOSS standards for the new package
- **test-forge** — additional testing beyond fixture replay
- **forge-forge** — meta-forge, registry
