# refactor-learnings — Known Pitfalls for Language Migrations

Hard-won lessons from real ports. Read this before starting any migration.

## Trigger

This is a reference document, not an invocable skill. `/refactor-capture`, `/refactor-port`, and `/refactor-verify` should reference these learnings automatically.

## Learnings

### L-001: YAML frontmatter list indentation differs between libraries

**Source:** ike.md TypeScript → Python port (2026-03-22)

**Problem:** gray-matter (js-yaml) indents list items 2 spaces under their key:
```yaml
tags:
  - urgent
  - backend
```

PyYAML puts list items at the key level:
```yaml
tags:
- urgent
- backend
```

Both are valid YAML. But if the port must produce byte-identical files (which it should for behavioral parity), you need a post-processing step.

**Fix:** After `yaml.dump()`, scan for lines starting with `- ` and add 2-space indent:
```python
for line in lines:
    if line.startswith("- "):
        fixed.append("  " + line)
```

**Applies to:** Any port involving YAML frontmatter where the source uses js-yaml (gray-matter, front-matter, etc.) and the target uses PyYAML.

---

### L-002: Date strings get parsed by YAML loaders

**Source:** ike.md TypeScript → Python port (2026-03-22)

**Problem:** `yaml.safe_load()` in Python parses `2026-03-22` as a `datetime.date` object, not a string. gray-matter with js-yaml keeps it as a string (when single-quoted). This causes type mismatches when comparing frontmatter.

**Fix:** After loading, convert any `date` objects back to ISO strings:
```python
for k, v in frontmatter.items():
    if hasattr(v, "isoformat"):
        frontmatter[k] = v.isoformat()
```

Also: when writing, use a custom representer that single-quotes date-like strings to match gray-matter output.

---

### L-003: Temp directory names in output break fixture comparison

**Source:** ike.md TypeScript → Python port (2026-03-22)

**Problem:** When a tool's output includes file paths (e.g., `File: /tmp/ike-fixtures-ABC123/.ike/tasks/TASK-0001.md`), the temp dir name differs between capture (TS) and verification (Python). This causes false negatives even when behavior is identical.

**Fix:** Normalize paths in both expected and actual output before comparison:
```python
text = re.sub(r"/var/folders/[^\s]+", "<TMPDIR>", text)
text = re.sub(r"/tmp/[^\s]+", "<TMPDIR>", text)
```

Similarly normalize GUIDs:
```python
text = re.sub(r"[0-9a-f]{8}-[0-9a-f]{4}-...", "<GUID>", text)
```

**Rule:** Any output field that contains a runtime-generated value (paths, GUIDs, timestamps) must be normalized before comparison. Document which fields need normalization in the fixture format.

---

### L-004: Fixtures that depend on temp dir basename are inherently non-portable

**Source:** ike.md `project_init/002-default-name` fixture

**Problem:** When `project_init` is called without a name, it uses `path.basename(targetDir)` as the project name. During capture, the temp dir was `ike-fixtures-pIP0tG`. During verification, it was `ike-v-fy8ep9h_`. The behavior is identical — both use the dir basename — but the output text differs.

**Fix:** Mark these fixtures as `comparison: "structural"` rather than `comparison: "exact"`. Structural comparison normalizes runtime-dependent values. Or better: always provide the `name` parameter in fixtures to avoid this class of problem entirely.

**Rule:** When designing fixtures, prefer explicit inputs over defaults that depend on runtime state.

---

### L-005: MCP SDK internal APIs differ — don't reach into private fields

**Source:** ike.md fixture capture script

**Problem:** The capture script needed to call tool handlers directly (not via stdio). The MCP Server object stores handlers in `_requestHandlers` (private Map), not `requestHandlers`. This internal API is undocumented and may change between versions.

**Fix:** For capture scripts, accept the fragility — you're testing the tool logic, not the transport. Use `server._requestHandlers.get("tools/call")` and document the SDK version.

For the Python port, FastMCP exposes tool functions as regular Python functions, so you can call them directly without reaching into SDK internals.

**Rule:** Always test tool logic directly, not through the transport layer. The transport is the SDK's responsibility; behavioral parity means the tool functions produce the same output for the same input.

---

### L-006: Stateful tool sequences require ordered fixture execution

**Source:** ike.md fixture verification

**Problem:** Fixtures for `task_list` assume specific tasks already exist (created by prior `task_create` fixtures). Running fixtures in arbitrary order causes false failures because the prerequisite state doesn't exist.

**Fix:** Execute fixtures in capture order. The capture script creates state sequentially; the verification script must replay in the same order.

**Alternative:** Make fixtures fully self-contained (each fixture sets up its own state). This is more work but produces more robust tests.

**Rule:** Document fixture dependencies. If fixture B requires state from fixture A, make this explicit in the fixture metadata or enforce ordering in the verification script.

---

### L-007: Private repos need explicit `contents: read` in GitHub Actions

**Source:** railguey PyPI publish failure (2026-03-22)

**Problem:** When a GitHub Actions workflow sets `permissions: { id-token: write }`, ALL other permissions default to `none` — including `contents: read`. This means `actions/checkout` fails with "repository not found" on private repos. Public repos work because they have implicit read access.

**Fix:** Always include `contents: read` when setting any explicit permissions:
```yaml
permissions:
  id-token: write
  contents: read
```

**Rule:** This isn't specific to language migrations, but it came up during the first port's release. Add it to `/ship-check` as a mandatory check for any publish workflow.

---

### L-008: Slugification regex must match exactly across languages

**Source:** ike.md slug parity testing

**Problem:** The TS slugify is: `title.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-|-$/g, "").slice(0, 50)`. The Python equivalent must handle Unicode identically. JavaScript's `[^a-z0-9]` only matches ASCII; Python's `re` module does the same by default (no `re.UNICODE` on character classes like `[a-z]`).

**Verified:** Python `re.sub(r"[^a-z0-9]+", "-", title.lower())` produces identical slugs to the JS version, including for Unicode input like `café ñ 日本語 émojis 🚀` → `caf-mojis`.

**Rule:** When porting regex, verify with Unicode test cases. JS and Python handle Unicode differently in some regex contexts (`\w`, `\d`, character classes with Unicode flag). Always test with non-ASCII input.

---

### L-009: Capture side effects, not just return values

**Source:** ike.md fixture design

**Problem:** MCP tools have two outputs: (1) the return value (text shown to the agent) and (2) side effects (files created, modified, or deleted). A port that returns the right text but writes the wrong file format is subtly broken.

**Fix:** Golden fixtures must capture both:
```json
{
  "output": { "content": [{"type": "text", "text": "Created TASK-0001"}] },
  "side_effects": {
    "tasks": {
      "TASK-0001 - my-task.md": "---\nid: TASK-0001\n..."
    }
  }
}
```

The verification script must check side effects too — not just the return value.

**Rule:** Every fixture captures: input, output, AND side effects. File-based tools must snapshot the affected directories before and after.
