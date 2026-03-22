# refactor-verify — Verify Behavioral Parity Between Original and Port

Run all golden fixtures against the ported implementation and report pass/fail with detailed diffs for any mismatches.

## Trigger

User says `/refactor-verify` or asks to verify, test, validate, or check a port/rewrite.

### Arguments

- No args: verify all tools in the migration
- `<tool-name>`: verify a specific tool only

## Instructions

### Step 1: Load Migration Map

Read `refactor/migration-map.md` to understand what's been captured and ported.

### Step 2: For Each Ported Tool

1. Load all fixtures from `refactor/fixtures/<tool-name>/`
2. Set up a clean temp directory
3. Run prerequisites (project_init, etc.)
4. For each fixture:
   a. Call the ported tool with fixture input
   b. Capture actual output and side effects
   c. Compare to expected (from fixture file)
   d. Record PASS or FAIL with diff

### Step 3: Report

```
## Verification Report: <package>

### Summary
- Tools ported: 15/21
- Fixtures total: 127
- Passing: 124
- Failing: 3

### Failures

#### task_create / 004-empty-title
Expected: "Error: Title is required"
Actual:   "Error: title is required"
Diff: capitalization of "Title" vs "title"

#### task_edit / 007-append-notes
Expected file content:
  "Previous notes\n\nNew notes"
Actual file content:
  "Previous notes\n\n\nNew notes"
Diff: extra newline between sections

### Pass Rate by Tool
| Tool | Fixtures | Passing | Rate |
|------|----------|---------|------|
| project_init | 5 | 5 | 100% |
| task_create | 8 | 7 | 87% |
| task_edit | 10 | 9 | 90% |
| ... | ... | ... | ... |
```

### Step 4: Update Migration Map

Update pass counts in `refactor/migration-map.md`.

## Rules

- **Run in clean temp directories.** Every verification must start from scratch.
- **Exact comparison for text output.** No fuzzy matching. Case matters. Whitespace matters.
- **Fuzzy comparison for timestamps and GUIDs.** These will differ between runs — compare structure, not values.
- **Show the diff for failures.** Don't just say "failed" — show exactly what's different.
- **100% is the bar.** Any failure means the port is not done. (GUARD-002)
