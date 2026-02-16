---
name: microfn
description: Use the MicroFn CLI (`microfn` or `mfn`) to deploy, update, inspect, and execute sandboxed JavaScript functions on the MicroFn cloud platform, including functions exposed as tools.
---

# MicroFn CLI Skill

Use this skill when interacting with the MicroFn platform through the installed CLI commands (`microfn` or `mfn`).

## What MicroFn is

MicroFn is a cloud platform for sandboxed JavaScript functions.
Use it to run focused JavaScript snippets as deployed functions.
Functions can power app behavior and can also be enabled as tools for agent workflows.

## Install and authenticate

1. Install globally with `npm install -g microfn`.
2. Use either command: `microfn` or `mfn`.
3. Create an API key at `https://microfn.dev/users/settings/api`.
4. Pass auth with `--token mfn_xxx` or `MICROFN_API_TOKEN`.

## Core workflows

### List functions

Run `microfn list` (or `mfn list`) to inspect available functions.

### Deploy a new function

1. From file: `microfn create <name> <path-to-file>`.
2. From stdin: `cat file.ts | microfn create <name> -`.

### Update function code

1. From file: `microfn push <owner/name> <path-to-file>`.
2. From stdin: `cat file.ts | microfn push <owner/name> -`.

### Inspect function details

Use:

- `microfn info <owner/name>`
- `microfn code <owner/name>`

### Execute a function

1. Inline payload: `microfn execute <owner/name> '{"key":"value"}'`.
2. Stdin payload: `echo '{"key":"value"}' | microfn execute <owner/name> -`.
3. Include logs when needed: add `--include-logs`.

### Work with tool-enabled functions

1. Check `microfn info <owner/name>` for MCP tool status.
2. Keep function signatures stable when functions are consumed as tools.

## Function source contract (important)

When writing code for `create` or `push`, follow the CLI contract used in command help:

1. Export exactly one entrypoint function style.
2. Prefer `export async function main(input) { ... }`.
3. If `main` is not exported, export exactly one named function; it will be auto-wrapped as `main()`.
4. The function should accept the execution input object and return a JSON-serializable value.
5. Sync and async functions are both valid.
6. Use default imports for `@microfn/*` packages.
7. Do not use named imports for `@microfn/*`.

Valid examples:

```typescript
export async function main(input) {
  const { name = "world" } = input || {};
  return { greeting: `hello ${name}` };
}
```

```typescript
export async function greet(payload) {
  return { greeting: `hello ${payload?.name || "world"}` };
}
```

```typescript
import kv from "@microfn/kv";
import secret from "@microfn/secret";

export async function main() {
  const apiKey = await secret.get("API_KEY");
  const count = (await kv.get("count")) || 0;
  await kv.set("count", count + 1);
  return { apiKeyPresent: !!apiKey, count: count + 1 };
}
```

Invalid import pattern:

```typescript
import { kv } from "@microfn/kv";
```

## Output and troubleshooting

1. Use `--output json` for scriptable output.
2. Use `--debug` for verbose request/response diagnostics.

## Delivery checklist

1. Show examples with `microfn` (note `mfn` is equivalent).
2. Keep command examples reproducible and copy-paste ready.
3. Preserve the product model: sandboxed JavaScript functions that can also be used as tools.
