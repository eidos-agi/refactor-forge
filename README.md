# refactor-forge

Rewrite software with proof. Any language to any other language, with behavioral parity testing.

## Why?

Rewrites fail when "seems to work" replaces "proven identical." refactor-forge captures the original's behavior as golden fixtures, then verifies the replacement matches — every tool, every edge case, every error message.

## The Pattern

```
Original → Capture golden fixtures → Port to target language →
Replay fixtures → Assert identical output → Ship replacement
```

## Skills

| Skill | What |
|-------|------|
| `/refactor-capture` | Run every tool in the original, record inputs + outputs + side effects as JSON fixtures |
| `/refactor-port` | Rewrite one tool at a time, matching behavior exactly |
| `/refactor-verify` | Replay all fixtures against the port, report pass/fail with diffs |
| `/refactor-migrate` | Full cutover: rename original to .tsc backup, install replacement, update all references |

## Usage

```bash
git clone https://github.com/eidos-agi/refactor-forge.git ~/repos-eidos-agi/refactor-forge
cp ~/repos-eidos-agi/refactor-forge/.claude/skills/refactor-*.md .claude/skills/
```

Then in the project you're rewriting:
1. `/refactor-capture` — generate golden fixtures
2. `/refactor-port task_create` — port one tool
3. `/refactor-verify task_create` — prove it matches
4. Repeat for all tools
5. `/refactor-migrate` — cut over

## Part of the Forge Ecosystem

- [forge-forge](https://github.com/eidos-agi/forge-forge) — the meta-forge
- [refactor-forge](https://github.com/eidos-agi/refactor-forge) — this repo
- [ship-forge](https://github.com/eidos-agi/ship-forge) — shipping standards for the ported package
- [test-forge](https://github.com/eidos-agi/test-forge) — additional testing

## License

MIT — [Eidos AGI](https://github.com/eidos-agi)
