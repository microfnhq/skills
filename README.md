# MicroFn Skills

**Repo**: microfnhq/skills

Agent kills for working with MicroFn - a cloud platform that turns JavaScript snippets into persistent, callable functions for your AI agent.

This is using the [microfn CLI](https://github.com/microfnhq/cli) under the hood

## What is MicroFn?

MicroFn is a programmable toolbox for AI agents. Instead of being limited to built-in tools, you can write custom JavaScript functions, deploy them to isolated sandboxes in the cloud, and have your AI agent execute them on demand.

**Think of it like this**: Your AI agent can now fetch weather data, send Discord messages, query databases, store state between conversations, access secrets securely, and call any API - all by deploying tiny JavaScript functions that live in the cloud and can be invoked anytime.

**The magic**: Your AI agent can write and deploy its own tools on the fly. Need a capability that doesn't exist yet? The agent can create a function, deploy it to MicroFn, and immediately start using it. It's like giving your AI the ability to extend itself with custom tools as needed.

## Installation

```bash
npx skills add microfnhq/skills
```

To list all available skills:

```bash
npx skills add microfnhq/skills --list
```

## Available Skills

### microfn

This skill teaches Claude how to use the MicroFn CLI (`microfn` or `mfn`) to deploy, update, inspect, and execute sandboxed JavaScript functions. Functions can power your applications and be exposed as MCP tools that your AI agent can call directly.

**Why is this powerful?**

Your AI agent can now have custom capabilities beyond its built-in tools. Need to check the weather? Deploy a function. Want to send a Discord notification? Deploy a function. Need to remember something between conversations? Use KV storage. Want to call a private API? Store the API key as a secret and deploy a function.

**Self-extending AI agents**: The most powerful feature is that your AI agent can create new tools for itself. During a conversation, if your agent needs a capability that doesn't exist, it can:

1. Write a JavaScript function for that capability
2. Deploy it to MicroFn with `microfn create`
3. Immediately execute it with `microfn execute`
4. Optionally expose it as an MCP tool for reuse

This means your agent gets smarter and more capable over time by building its own toolbox.

**Real-world examples**:

🌤️ **Weather service with caching**

```typescript
import kv from "@microfn/kv";

export async function main(input) {
  const cached = await kv.get(`weather:${input.city}`);
  if (cached) return cached;

  const response = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?q=${input.city}`,
  );
  const weather = await response.json();

  await kv.set(`weather:${input.city}`, weather, { ttl: 3600 });
  return weather;
}
```

Deploy once, call anytime: `microfn execute myuser/weather '{"city":"Tokyo"}'`

💬 **Discord/Slack notifications**

```typescript
import secret from "@microfn/secret";

export async function main(input) {
  const webhook = await secret.get("DISCORD_WEBHOOK");

  await fetch(webhook, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      content: input.message,
      username: "AI Agent Bot",
    }),
  });

  return { sent: true, timestamp: new Date().toISOString() };
}
```

Your AI can now send notifications to your team!

📊 **Stateful counter with analytics**

```typescript
import kv from "@microfn/kv";

export async function main(input) {
  const count = (await kv.get("total_requests")) || 0;
  const newCount = count + 1;

  await kv.set("total_requests", newCount);
  await kv.set(`request:${newCount}`, {
    timestamp: Date.now(),
    data: input,
  });

  return { requestNumber: newCount, totalRequests: newCount };
}
```

Track state across conversations and function calls.

🔐 **Authenticated API calls**

```typescript
import secret from "@microfn/secret";

export async function main(input) {
  const apiKey = await secret.get("GITHUB_TOKEN");

  const response = await fetch(
    `https://api.github.com/repos/${input.repo}/issues`,
    {
      headers: {
        Authorization: `Bearer ${apiKey}`,
        Accept: "application/vnd.github.v3+json",
      },
    },
  );

  return await response.json();
}
```

Securely call private APIs without exposing credentials.

**More possibilities**:

- Process and transform data in custom ways
- Validate inputs with your own business logic
- Integrate with any REST API (Stripe, Twilio, SendGrid, etc.)
- Build stateful workflows that remember context
- Create custom tools specifically for your AI agent's needs
- Schedule recurring tasks or webhooks

**Invoke**: `/microfn`

## Resources

- [MicroFn Platform](https://microfn.dev)
