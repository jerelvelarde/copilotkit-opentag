# OpenTag — a Slack agent starter, built on CopilotKit

OpenTag is a **runnable Slack agent you can clone and make your own**. It's built on
**[`@copilotkit/bot`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot)** —
CopilotKit's SDK for building chat-platform agents (Slack first; the same code also runs on
Discord, Telegram, and WhatsApp).

Out of the box it ships as an on-call triage assistant — @mention it in a thread and it
answers, files issues, and renders charts inline — but that's just the example. The point is
the **starting point**: a clean, real Slack agent you can bend to whatever your team needs.

## See it in action

https://github.com/user-attachments/assets/a74fa1cb-add0-463e-a23c-aa09b95d5135

▶️ **[Watch the demo](https://github.com/user-attachments/assets/a74fa1cb-add0-463e-a23c-aa09b95d5135)** (~50s) — the agent working a Slack thread: it renders a breakdown, a table, and a bar chart inline (**generative UI**) and files a ticket only after an **Approve** gate (**human-in-the-loop**).

> 🚀 **Building a real Slack agent?** The CopilotKit Slack bot SDK is in early access. **[Sign up for early access →](https://docs.copilotkit.ai/slack)** — or **[talk to an engineer](https://copilotkit.ai/talk-to-an-engineer)**.

## Quick start

OpenTag ships inside the [CopilotKit monorepo](https://github.com/CopilotKit/CopilotKit) as a
first-class example (`examples/slack`) — that's the dependable way to run it today while the
bot SDK packages finish publishing to npm. (A standalone `npm install` from this repo lights
up the moment they land — see [setup.md](./setup.md).)

You'll run two processes — the **agent** (the LLM backend) and the **bot** (the Slack
connection) — and set three secrets.

**1. Create a Slack app.** At [api.slack.com/apps](https://api.slack.com/apps?new_app=1) →
*From a manifest* → paste [`slack-app-manifest.yaml`](./slack-app-manifest.yaml). Install it,
then grab the **Bot User OAuth Token** (`xoxb-…`) and an **App-Level Token** (`xapp-…`, with the
`connections:write` scope). Step-by-step in [setup.md](./setup.md#1-create-a-slack-app).

**2. Set three secrets** in `.env` (`cp .env.example .env`):

```bash
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...
OPENAI_API_KEY=sk-...
```

**3. Run it** from the CopilotKit monorepo root:

```bash
pnpm install
pnpm --filter slack-example runtime   # the agent backend, on :8200
pnpm --filter slack-example dev        # the bot
```

**4. Talk to it.** @mention the bot in any channel thread:

> @OpenTag summarize this thread and file it as a bug

That's the whole loop. To wire up Linear, Notion, inline charts, Redis persistence, or to run
on Discord / Telegram / WhatsApp, see **[setup.md](./setup.md)**.

## Make it your own

OpenTag is deliberately small and hackable:

- **Change what it does.** The agent's behavior is steered by a single system prompt in
  [`runtime.ts`](./runtime.ts) — rewrite it and you have a different agent.
- **Copy `app/` to start your own bot.** It's the platform-agnostic bot (tools, components, the
  human-in-the-loop gate). `runtime.ts` is the agent backend: one CopilotKit `BuiltInAgent` (an
  LLM + optional MCP tools — no Python, no LangGraph), served over AG-UI.
- **One platform, or all of them.** `createBot` takes an array of adapters; set the secrets for
  whichever platform(s) you want and the bot starts an adapter for each.

The full architecture, the file-by-file map, and every integration live in
**[setup.md](./setup.md)**.

## Going to production?

OpenTag is the open, runnable starting point. When you're ready to ship an agent for real,
**CopilotKit's managed Intelligence Platform** (coming soon) is the production layer beside your
runtime — durable threads, persistence, hosted inspection, and **Continuous Learning from Human
Feedback** (agents that improve with every interaction). Learn more about
[CopilotKit Intelligence](https://www.copilotkit.ai/copilotkit-intelligence).

- **[Sign up for Intelligence →](https://go.copilotkit.ai/enterprise-intelligence-platform)** — the managed production platform (coming soon).
- **[Talk to an engineer →](https://copilotkit.ai/talk-to-an-engineer)** — building something real on this? We'd love to help you ship it.

## Learn more

The **[CopilotKit Slack quickstart](https://docs.copilotkit.ai/slack)** is the canonical guide
to building a Slack agent — read it alongside this starter. Detailed setup and configuration
lives in **[setup.md](./setup.md)**.

## License

MIT — see [LICENSE](./LICENSE).
