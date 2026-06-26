# OpenTag — build a Slack agent with CopilotKit

Build a Slack agent with two packages:

- **[`@copilotkit/bot`](https://www.npmjs.com/package/@copilotkit/bot)** — the platform-agnostic bot engine (`createBot`, `Thread`, tools, human-in-the-loop). [source](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot)
- **[`@copilotkit/bot-slack`](https://www.npmjs.com/package/@copilotkit/bot-slack)** — the Slack adapter (Socket-Mode ingress, Block Kit egress, native streaming, modals, the assistant pane). [source](https://github.com/CopilotKit/CopilotKit/tree/main/packages/bot-slack)

Write your bot once; @mention it in a thread and it answers, calls your tools, renders rich
Block Kit, and gates writes behind a human-in-the-loop confirm.

## See it in action

https://github.com/user-attachments/assets/a74fa1cb-add0-463e-a23c-aa09b95d5135

▶️ **[Watch the demo](https://github.com/user-attachments/assets/a74fa1cb-add0-463e-a23c-aa09b95d5135)** (~50s) — a Slack agent built on these packages: generative UI (a breakdown, a table, a chart rendered inline) and a human-in-the-loop **Approve** gate before it files a ticket.

> 🚀 **Building a real Slack agent?** The CopilotKit Slack bot SDK is in early access. **[Sign up for early access →](https://docs.copilotkit.ai/slack)** — or **[talk to an engineer](https://copilotkit.ai/talk-to-an-engineer)**.

## Quickstart

Install the engine, the Slack adapter, and the JSX message vocabulary:

```sh
npm i @copilotkit/bot @copilotkit/bot-slack @copilotkit/bot-ui
```

Wire it up:

```ts
import { createBot } from "@copilotkit/bot";
import {
  slack,
  defaultSlackTools,
  defaultSlackContext,
} from "@copilotkit/bot-slack";

const bot = createBot({
  adapters: [
    slack({
      botToken: process.env.SLACK_BOT_TOKEN!, // xoxb-…
      appToken: process.env.SLACK_APP_TOKEN!, // xapp-… (Socket Mode)
    }),
  ],
  agent: (threadId) => makeAgent(threadId), // your AG-UI agent (e.g. a CopilotKit runtime)
  tools: [...defaultSlackTools, ...appTools], // lookup_slack_user + your tools
  context: [...defaultSlackContext, ...appContext], // tagging / mrkdwn / thread guidance
});

bot.onMention(({ thread }) => thread.runAgent());

await bot.start();
```

`makeAgent` builds your agent (any AG-UI agent — point it at a CopilotKit runtime), and
`appTools` / `appContext` are your additions. The full walkthrough — creating the Slack app,
the two tokens, and the agent backend — is in the
**[CopilotKit Slack quickstart](https://docs.copilotkit.ai/slack)**.

## A complete, runnable example

For an end-to-end bot wiring all of this against a real workspace — tools, rich cards, a Block
Kit modal, the `confirm_write` gate, chart/diagram/table rendering, and slash commands — see
**[`examples/slack`](https://github.com/CopilotKit/CopilotKit/tree/main/examples/slack)** in the
CopilotKit monorepo.

## Going to production?

When you're ready to ship for real, **CopilotKit's managed Intelligence Platform** (coming soon)
is the production layer beside your runtime — durable threads, persistence, hosted inspection,
and Continuous Learning from Human Feedback. **[Sign up for Intelligence →](https://go.copilotkit.ai/enterprise-intelligence-platform)** · **[Talk to an engineer →](https://copilotkit.ai/talk-to-an-engineer)**.

## License

MIT — see [LICENSE](./LICENSE).
