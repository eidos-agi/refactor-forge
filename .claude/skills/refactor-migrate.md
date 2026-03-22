# refactor-migrate — Execute the Full Migration Cutover

Once all tools are ported and verified at 100%, execute the cutover: rename the original, install the replacement, update all references.

## Trigger

User says `/refactor-migrate` or asks to cut over, switch, finalize the migration, or go live with the port.

## Pre-flight Checks

ALL must pass before proceeding:

1. **All tools ported** — migration map shows every tool marked as ported
2. **100% fixture pass rate** — run `/refactor-verify` and confirm zero failures
3. **Package builds** — `python -m build` succeeds for the Python replacement
4. **Package installs** — wheel installs in clean venv
5. **MCP server starts** — the replacement server starts and responds to list_tools
6. **Tool count matches** — replacement exposes the same number of tools as original

## Cutover Steps

### Step 1: Rename Original

```bash
mv ~/repos-eidos-agi/<package> ~/repos-eidos-agi/<package>.tsc
```

This preserves the TypeScript version as a backup. Don't delete it.

### Step 2: Create Python Package

The ported Python code becomes the new package at `~/repos-eidos-agi/<package>/`.

Ensure it has:
- `pyproject.toml` (hatchling, full metadata)
- MCP entry point configured
- All forge standards (LICENSE, CHANGELOG, SECURITY, CI, publish workflow, manifest)

### Step 3: Update MCP Config

Update `.mcp.json` in any project that uses this server:
- Change `command` from `npx`/`node` to `uvx` or `python -m`
- Update any args

### Step 4: Smoke Test

Run the replacement in a real project (not just test fixtures):
1. Start the MCP server
2. Call 3-5 tools manually
3. Verify output matches expectations
4. Check files created/modified are correct

### Step 5: Publish

Run `/ship-release` on the Python package to publish to PyPI.

### Step 6: Update Registry

Update `forge-forge/registry.yaml` if the package is a tool forge.

## Rules

- **Never delete the original.** Rename to `.tsc` as backup.
- **Never cut over with failing fixtures.** (GUARD-002)
- **Update every reference.** MCP configs, CLAUDE.md mentions, forge registry, package.json → pyproject.toml.
- **One package at a time.** Don't batch migrate all three simultaneously.
