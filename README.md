# OpenTag — a CopilotKit triage bot (Slack · Discord · Telegram · WhatsApp)

OpenTag is a **runnable, on-call triage bot** built with
[CopilotKit](https://github.com/CopilotKit/CopilotKit). @mention it in a thread
and it turns incident chatter into tracked work: it pulls and files **Linear**
issues, finds and writes **Notion** runbooks/postmortems, renders charts,
diagrams, and tables inline, and gates every write behind a human-in-the-loop
confirm.

**One app, any platform — or all at once.** `createBot` takes an array of
adapters; `app/index.ts` starts the Slack adapter when `SLACK_*` secrets are
present, Discord when `DISCORD_*` are present, Telegram when `TELEGRAM_BOT_TOKEN`
is present, and WhatsApp when `WHATSAPP_*` are present. Everything else in `app/`
(tools, components, the `confirm_write` HITL gate, chart/diagram/table rendering)
is platform-agnostic and shared verbatim.

It's built on:

- **[`@copilotkit/bot`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot)** — the platform-agnostic bot engine.
- **[`@copilotkit/bot-slack`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot-slack)** / **[`-discord`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot-discord)** / **[`-telegram`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot-telegram)** / **[`-whatsapp`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot-whatsapp)** — the platform adapters.
- **[`@copilotkit/bot-ui`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot-ui)** — a cross-platform JSX vocabulary for rich messages (Block Kit on Slack, Components V2 on Discord, HTML on Telegram).
- **[`@copilotkit/runtime`](https://github.com/CopilotKit/CopilotKit/tree/main/packages/runtime)** — the AG-UI agent backend (`BuiltInAgent` = an LLM + MCP; no Python, no LangGraph).

## See it in action

https://github.com/user-attachments/assets/a74fa1cb-add0-463e-a23c-aa09b95d5135

▶️ **[Watch the demo](https://github.com/user-attachments/assets/a74fa1cb-add0-463e-a23c-aa09b95d5135)** (~50s) — the bot triaging issues in Slack: it renders a breakdown, a table, and a bar chart inline (**generative UI**) and files a ticket only after an **Approve** gate (**human-in-the-loop**).

> [!IMPORTANT]
> **This is a faithful port of CopilotKit's [`examples/slack`](https://github.com/CopilotKit/CopilotKit/tree/main/examples/slack).** It uses the modern (`0.1.x`) bot API — multi-adapter, modals, `respondTo`. A standalone `npm install` is **pending publish**: `@copilotkit/bot-telegram`, `-whatsapp`, and `-store-redis` aren't on npm yet, and the bot packages need a coherent `0.1.x` release. **Until then, run it from the [CopilotKit monorepo](https://github.com/CopilotKit/CopilotKit) as `examples/slack`** (`pnpm --filter slack-example dev`). The moment the packages publish, this repo's `npm install` lights up — see [Run it](#run-it).

## What it does

It connects to **Linear** and **Notion** over MCP and can:

- **Query Linear** — _"what's open in CPK this cycle?"_ → renders the issues as a rich card.
- **File a Linear issue** — _"file this thread as a bug"_ → drafts it, asks you to **confirm**, then creates it.
- **Find Notion pages** — _"find the runbook for the auth outage"_ → renders matching pages with links.
- **Write a postmortem** — _"write this thread up as a Notion doc"_ → reads the thread, summarizes, **confirms**, then creates the page.
- **Chart / diagram / table data** — drop a CSV and say _"chart revenue by month"_, _"diagram this incident flow"_, or _"show it as a table"_.

Every write goes through a human-in-the-loop **`confirm_write`** gate: the agent
must call that tool and wait for a Create/Cancel click before it performs any
Linear/Notion write.

## How it fits together

```
Slack / Discord / Telegram / WhatsApp ──@mention──▶  bot (app/)  ──AG-UI──▶  runtime (runtime.ts)
                                                          │  BuiltInAgent (LLM)
                                                          ├── Linear  MCP  (hosted)
                                                          └── Notion  MCP  (sidecar)
```

| Concept                                                            | Where                                                              |
| ------------------------------------------------------------------ | ------------------------------------------------------------------ |
| `createBot({ adapters, agent, tools, context, commands })`         | `app/index.ts`                                                     |
| Multi-adapter wiring (Slack/Discord/Telegram/WhatsApp, secret-gated) | `app/index.ts`                                                     |
| `read_thread` — grounds the agent in the real conversation         | `app/tools/read-thread.ts`                                         |
| Render-tools + JSX components (issue card/list, Notion pages)      | `app/tools/render-tools.tsx`, `app/components/`                    |
| Chart / diagram / table rendering (Playwright → PNG)               | `app/tools/render-chart.tsx`, `render-diagram.tsx`, `render-table.tsx`, `app/render/` |
| Status / incident / links showcase cards                           | `app/tools/showcase-tools.tsx`, `app/components/_status.ts`        |
| Blocking **human-in-the-loop** gate (`confirm_write`)              | `app/human-in-the-loop/confirm-write.tsx`                          |
| Slash commands (`/agent`, `/triage`, `/preview`, `/file-issue`)    | `app/commands/index.ts`                                            |
| A Block Kit **modal** (`/file-issue`)                              | `app/modals/file-issue.tsx`                                        |
| The agent backend — one `BuiltInAgent` (LLM + Linear/Notion MCP)   | `runtime.ts`                                                       |

- **`app/`** — the platform-agnostic bot. **This is the directory you copy to start your own bot.**
- **`runtime.ts`** — the agent backend: a single CopilotKit `BuiltInAgent` (LLM + Linear/Notion MCP), served over AG-UI. No Python, no LangGraph.
- **`e2e/`** — live test harnesses (the Slack harness is legacy/WIP; the Telegram harness is a manual-trigger smoke test — see [`e2e/TELEGRAM-README.md`](./e2e/TELEGRAM-README.md)).

## Run it

Pieces: the **chat-platform app(s)** (created once), the optional **Notion MCP
sidecar**, the **agent** (`runtime.ts`), and the **bot** (`app/`). Set up
whichever platform(s) you want — the bot starts an adapter for each one whose
secrets are present.

### 1. Create the Slack app

- <https://api.slack.com/apps?new_app=1> → **From a manifest** → paste
  [`slack-app-manifest.yaml`](./slack-app-manifest.yaml). It declares all four
  slash commands, the assistant pane, the `users:read.email` scope, and **Socket
  Mode** (so the bot connects outbound — no public URL needed).
- _OAuth & Permissions_ → **Install to Workspace** → copy the `xoxb-` **Bot User
  OAuth Token** → `SLACK_BOT_TOKEN`.
- _Basic Information → App-Level Tokens_ → generate one with the
  `connections:write` scope → copy the `xapp-` token → `SLACK_APP_TOKEN`.

(Discord, Telegram, and WhatsApp setup is in [`.env.example`](./.env.example).)

### 2. Credentials

```bash
cp .env.example .env
# Fill in (set the platform(s) you want):
#   SLACK_BOT_TOKEN / SLACK_APP_TOKEN          (to run on Slack)
#   DISCORD_BOT_TOKEN / DISCORD_APP_ID         (to run on Discord)
#   TELEGRAM_BOT_TOKEN                         (to run on Telegram)
#   OPENAI_API_KEY  (or ANTHROPIC_API_KEY / GOOGLE_API_KEY + AGENT_MODEL)
#   LINEAR_API_KEY          (linear.app → Settings → API → Personal API keys)
#   NOTION_TOKEN            (notion.so → Settings → Connections → integrations)
#   NOTION_MCP_AUTH_TOKEN   (any strong string; shared with the sidecar)
```

Linear and Notion are independent — set only the ones you want; the agent wires
up whichever credentials are present.

### 3. Install & run

> **Today, run it from the monorepo.** Until the bot packages publish a coherent
> `0.1.x` set, the dependable path is to run this code as `examples/slack` inside
> the [CopilotKit monorepo](https://github.com/CopilotKit/CopilotKit) (it builds
> the adapters from source via `workspace:*`):
>
> ```bash
> pnpm install                              # repo root
> pnpm --filter slack-example notion-mcp    # only if using Notion → http://127.0.0.1:3001/mcp
> pnpm --filter slack-example runtime       # CopilotKit runtime on :8200, agent "triage"
> pnpm --filter slack-example dev           # the bot (tsx watch app/index.ts)
> ```

**Standalone (once `@copilotkit/bot-*` publish):** `npm install` here, then:

```bash
npm run notion-mcp     # terminal 1 — only if using Notion
npm run runtime        # terminal 2 — the agent backend on :8200
npm run dev            # terminal 3 — the bot
```

The chart/diagram renderers need a Chromium binary: `npx playwright install chromium`.

### 4. Try it

@mention the bot in a channel thread, DM it, or run a slash command:

> @OpenTag what are the open CPK issues this cycle?
> @OpenTag file this thread as a bug in CPK
> @OpenTag write this thread up as a Notion postmortem

## Slash commands

Four app-owned commands, registered via `createBot({ commands })`:

- **`/agent <text>`** — a mention-free entry point; runs the agent with the command text.
- **`/triage [note]`** — summarizes the conversation and proposes Linear issues to file.
- **`/preview <title>`** — privately previews the issue the bot would file (only you see it); degrades to a DM where ephemerals aren't supported.
- **`/file-issue`** — opens a structured Linear issue **modal**; degrades to a conversational flow on platforms without modals (e.g. Telegram).

On Slack, all four must be declared under **Slash Commands** — the manifest already does this.

## Files → charts, diagrams & tables

Upload a file and the bot analyzes it: images and **PDFs** go straight to the
model; CSV/JSON/text are decoded and handed over as text. The chart/diagram
libraries load from a CDN into a **local** headless browser (override
`CHART_JS_URL` / `MERMAID_URL`) — your data is rendered locally and never sent to
a rendering service.

> **PDFs and images need a vision/document-capable model.** The default
> `openai/gpt-5.5` reads both natively, as do recent Claude and Gemini models.

## Make it yours

- **Change the taxonomy / prompt.** The triage behavior is steered entirely by the system prompt in `runtime.ts`.
- **Run one platform, or all.** Set only the secrets for the platform(s) you want; `createBot` takes several adapters at once.
- **Durable buttons.** Pass a `@copilotkit/bot-store-redis` store to `createBot` (`docker compose up -d` for a local Redis) so an Approve/Cancel click still resolves after a restart — see `app/demo-restart.tsx`.
- **Swap the data sources.** Linear and Notion are wired in `runtime.ts` over MCP; point the transports at your own MCP servers.

## Tests

```bash
npm test               # unit tests: read_thread, render tools, components, confirm_write, modals, commands
npm run check-types    # tsc --noEmit
```

> The live-Slack e2e harness (`npm run e2e`) is being migrated to the new
> `createBot` API and doesn't run against this code as-is. The Telegram harness
> (`npm run e2e:telegram`) is a working manual-trigger smoke test — see
> [`e2e/TELEGRAM-README.md`](./e2e/TELEGRAM-README.md).

## Learn more

The official **[CopilotKit Slack quickstart](https://docs.copilotkit.ai/slack)**
is the canonical guide to building a Slack bot — read it alongside this starter.

## License

MIT — see [LICENSE](./LICENSE).
