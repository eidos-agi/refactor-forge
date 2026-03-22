# refactor-port — Port a Tool from One Language to Another

Rewrite a tool/function/module from the source language to the target language, maintaining identical behavior as verified by golden fixtures.

## Trigger

User says `/refactor-port` or asks to port, rewrite, migrate, or convert code from one language to another.

### Arguments

- `<tool-name>` — which tool to port (e.g., `task_create`)
- `<source>` — source language/framework (default: detect from source code)
- `<target>` — target language/framework (default: Python + FastMCP)

## Instructions

### Step 1: Read the Original

Read the source implementation of the specific tool being ported. Understand:
- Input parameters and types
- Business logic (what it actually does)
- Output format (exact text, structure)
- Side effects (files created/modified/deleted)
- Error handling (what errors are raised, with what messages)
- Dependencies on shared utilities (slug generation, ID generation, file I/O)

### Step 2: Check Fixtures Exist

Verify `refactor/fixtures/<tool-name>/` exists and has fixtures. If not, run `/refactor-capture` first.

### Step 3: Write the Port

Write the target language implementation. Key rules:

- **Match behavior exactly.** The output text must be identical. Not "similar" — identical.
- **Match side effects exactly.** Files created must have the same names, same frontmatter, same content.
- **Match error messages exactly.** Same error text for the same bad input.
- **Shared utilities first.** Port helpers (slug generation, ID generation, frontmatter I/O) before tools that use them.
- **One tool at a time.** Don't try to port everything in one pass.

### Step 4: Run Fixtures

After writing the port, run every fixture for this tool:
1. Set up a clean temp directory
2. Run any prerequisite fixtures (e.g., project_init before task_create)
3. Call the ported tool with the fixture input
4. Compare output to expected_output
5. Compare side effects (files created, file contents) to expected

### Step 5: Update Migration Map

Edit `refactor/migration-map.md` to mark this tool as ported and record pass/fail counts.

## Output

```
## Port Complete: <tool-name>

Source: TypeScript (server.ts:392-413)
Target: Python (ike_md/tools/task_create.py)

Fixtures: 8/8 passing ✓

Changes from original:
- None (exact behavioral parity)

OR

- Output text differs at line 3: "project_id" vs "project-id" (MUST FIX)
```

## Rules

- **Never mark a port as done with failing fixtures.** (GUARD-002)
- **Port shared utilities first.** Slug generation, ID generation, frontmatter parsing, path safety — these are the foundation.
- **One tool per PR/commit.** Atomic, reviewable, testable.
- **Don't "improve" during the port.** The goal is behavioral parity. Improvements come after the port is complete and proven.
