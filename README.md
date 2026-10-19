# MCP Manifest Breaking-Change Diff

Compare two MCP server tool manifests (the `tools/list` result) and flag the changes that break agents **before** you upgrade a server.

It is a single file with zero dependencies and runs on Node 18+. The exit codes let you use it as a CI gate before you bump an MCP server version.

**Download (pay what you want, from $1):** https://renevibe76.gumroad.com/l/uzuvnj

This repository holds the documentation. The script itself (`manifest-diff.mjs`) and the example manifests come in the download.

## Usage

```
node manifest-diff.mjs old.json new.json
node manifest-diff.mjs old.json new.json --json
node manifest-diff.mjs old.json new.json --fail-on=warning
```

Accepted input:
- a `{"tools":[...]}` object
- a JSON-RPC reply `{"result":{"tools":[...]}}`
- a bare array of tools

Exit codes:
- `0`: OK
- `1`: findings at or above `--fail-on` (default: `breaking`)
- `2`: usage or input error

## What it flags

**BREAKING**
- A tool was removed (or renamed).
- A new REQUIRED parameter was added.
- An optional parameter became required.
- A type was narrowed.
- An enum value was removed, or an enum restriction was added.
- A constraint was tightened: maxLength, maximum, maxItems, minLength, minimum, minItems, or pattern.
- `additionalProperties` was closed.
- An output field was removed.
- A `readOnlyHint` was dropped.
- A tool became `destructiveHint`.

**WARNING**
- A tool description changed. This can change which tool the model picks.
- A parameter was removed while extra properties are still allowed.
- Output types or enums were widened.

**INFO**
- New tools, with a likely-rename hint when a new tool has the same description as a removed one.
- New optional parameters.
- Loosened input constraints.

## Example CI step (GitHub Actions)

```yaml
- name: MCP manifest breaking-change gate
  run: node manifest-diff.mjs manifests/previous.json manifests/current.json --fail-on=breaking
```

## Honest limits

- It is a structural diff of JSON Schema keywords. It does not resolve `$ref`, `oneOf`, `anyOf` or `allOf`, and it does not call your server.
- Rename detection is a heuristic based on identical descriptions.
- No benchmark or adoption numbers are claimed here. Run the bundled example to see real output.
- Built with AI assistance and reviewed by the author.

## License

MIT. Made by Rene Vergara.
