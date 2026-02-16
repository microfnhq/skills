# MicroFn CLI Reference

## Platform summary

MicroFn is a sandboxed JavaScript cloud runtime.
Deploy small JavaScript functions, execute them on demand, and optionally expose them as tools.

## Install

```bash
npm install -g microfn
```

This installs both commands:

- `microfn`
- `mfn`

## Authentication

- Create API keys at `https://microfn.dev/users/settings/api`
- Env var: `MICROFN_API_TOKEN`
- Override per command: `--token mfn_xxx`

## Core commands

- `microfn list`
- `microfn create <name> <file-or->`
- `microfn info <owner/name>`
- `microfn code <owner/name>`
- `microfn push <owner/name> <file-or->`
- `microfn execute <owner/name> <json-or->`

Useful flags:

- `--include-logs` (with `execute`)
- `--output json`
- `--debug`

## Function shape

The CLI expects function modules that follow this contract:

1. Export exactly one entrypoint function style.
2. Export `main(input)` directly, or export a single named function that can be auto-wrapped as `main()`.
3. Accept execution input (object payload) and return JSON-serializable output.
4. Use default imports for `@microfn/*` modules.
5. Avoid named imports for `@microfn/*`.

Example:

```typescript
export async function main(input) {
  const { name = "world" } = input || {};
  return { greeting: `hello ${name}` };
}
```

Auto-wrapped named export example:

```typescript
export async function getWeather(input) {
  const { city = "tokyo" } = input || {};
  const res = await fetch(`https://wttr.in/${encodeURIComponent(city)}?format=3`);
  return { weather: await res.text() };
}
```

Import example:

```typescript
import kv from "@microfn/kv";
import secret from "@microfn/secret";
```
