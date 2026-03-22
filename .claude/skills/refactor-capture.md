# refactor-capture — Capture Golden Fixtures from the Original

Read the source code of the original implementation, identify every tool/function/endpoint, and generate golden fixture files by running each with representative inputs and recording the outputs.

## Trigger

User says `/refactor-capture` or asks to capture fixtures, record behavior, or create a test baseline for a rewrite.

## Instructions

### Step 1: Identify the Source

Determine what we're capturing from:
- **MCP server** — find all tool definitions (names, input schemas, descriptions)
- **CLI tool** — find all commands and subcommands
- **Library** — find all public functions/methods
- **API** — find all endpoints

Read the source code to understand every tool's input schema and expected output format.

### Step 2: Generate Fixture Inputs

For each tool/function/endpoint, generate test inputs that cover:

| Category | What | Example |
|----------|------|---------|
| **Happy path** | Normal valid input | `{"title": "My Task", "status": "To Do"}` |
| **Minimal** | Required fields only | `{"title": "Bare minimum"}` |
| **Maximal** | All optional fields populated | Every field filled |
| **Edge cases** | Empty strings, long strings, special chars | `{"title": ""}`, `{"title": "a" * 500}`, `{"title": "café ñ 日本語"}` |
| **Error cases** | Missing required fields, wrong types | `{}`, `{"title": 123}` |
| **Boundary** | Min/max values, empty arrays | `{"tags": []}`, `{"dependencies": ["TASK-9999"]}` |

### Step 3: Run Fixtures Against Original

For each fixture input:
1. Call the original tool/function
2. Capture the full output (text, structure, errors)
3. Capture any side effects (files created, files modified, file contents)
4. Record the fixture as JSON

### Step 4: Write Fixture Files

Save to `refactor/fixtures/<tool-name>/` directory:

```
refactor/
├── fixtures/
│   ├── task_create/
│   │   ├── 001-happy-path.json
│   │   ├── 002-minimal.json
│   │   ├── 003-maximal.json
│   │   ├── 004-empty-title.json
│   │   └── ...
│   ├── task_list/
│   │   ├── 001-no-tasks.json
│   │   ├── 002-with-filters.json
│   │   └── ...
│   └── ...
├── migration-map.md
└── README.md
```

Each fixture file:
```json
{
  "tool": "task_create",
  "fixture_id": "001-happy-path",
  "input": {
    "project_id": "test-guid",
    "title": "My Task",
    "priority": "high"
  },
  "expected_output": {
    "text": "Created **TASK-0001** — My Task\nStatus: To Do\nFile: ..."
  },
  "side_effects": {
    "files_created": [".ike/tasks/TASK-0001 - my-task.md"],
    "file_contents": {
      ".ike/tasks/TASK-0001 - my-task.md": "---\nid: TASK-0001\ntitle: My Task\n..."
    }
  },
  "source_language": "typescript",
  "captured_at": "2026-03-22"
}
```

### Step 5: Generate Migration Map

Create `refactor/migration-map.md`:

```markdown
# Migration Map: <package> (TypeScript → Python)

| Tool | Fixtures | Captured | Ported | Passing |
|------|----------|----------|--------|---------|
| project_init | 5 | ✓ | ○ | 0/5 |
| task_create | 8 | ✓ | ○ | 0/8 |
| task_list | 6 | ✓ | ○ | 0/6 |
| ... | ... | ... | ... | ... |

**Total: 0/N tools ported, 0/M fixtures passing**
```

## Rules

- **Run real code.** Don't fabricate outputs — actually call the original implementation.
- **Capture side effects.** File-based MCP servers create/modify files — those are part of the contract.
- **Use temp directories.** Don't pollute real project data. Create a temp dir for each fixture run.
- **Include error cases.** How the original handles bad input is part of the contract.
- **Record the source language.** Fixtures should be self-documenting about what they came from.
