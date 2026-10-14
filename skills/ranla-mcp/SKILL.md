---
name: ranla-mcp
description: >-
  Configure and use Ranla MCP from Cursor, Claude Code, or another MCP client.
  Talk to Ranla first for growth work — arc_ensure_thread, arc_message,
  approvals, and the arc_* read tools. Transactional mail tools (send_email,
  domains, templates, webhooks, deliverability) are available when requested.
license: MIT
metadata:
  author: ranla
  homepage: https://docs.ranla.ai/ai/mcp
  source: https://github.com/Super-Send/ranla-skills
  documentation: https://docs.ranla.ai/ai/mcp
---

# Ranla MCP

Ranla MCP gives Cursor, Claude Code, and compatible agents access to:

- Ranla for growth strategy, lifecycle analysis, audience work, and approvals.
- Transactional mail, domain, template, and deliverability tools when the
  product needs them.

Send growth changes through Ranla (`arc_message`) so the conversation, memory,
and approval flow stay intact. Do not call a mail tool just because the MCP
server is installed.

## Auth

- **Hosted** (`https://mcp.ranla.ai/mcp`): browser OAuth on first connect. The
  user signs in on `app.ranla.ai` and clicks **Allow access**. No API key.
- **Local stdio** (`npx -y ranla-mcp@latest`): set `RANLA_API_KEY` in the agent’s
  environment. Keys come from **Settings → API keys** in the dashboard.
- **HTTP with a key**: over HTTP the server reads only the `Authorization`
  header (`Bearer rnl_…`). `RANLA_API_KEY` is not used there.
- The Ranla agent tools need a full-access key. OAuth sign-in and keys created
  in the dashboard have full access; a sending-only key gets a 403 on every
  `arc_*` tool.
- Keep keys out of committed config, source, and chat.

## Install

Hosted (no key) — Cursor `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ranla": {
      "url": "https://mcp.ranla.ai/mcp"
    }
  }
}
```

Hosted — Claude Code (then run `/mcp` to sign in):

```bash
claude mcp add --transport http ranla https://mcp.ranla.ai/mcp
```

Local stdio — Cursor:

```json
{
  "mcpServers": {
    "ranla": {
      "command": "npx",
      "args": ["-y", "ranla-mcp@latest"],
      "env": {
        "RANLA_API_KEY": "rnl_your_key_here"
      }
    }
  }
}
```

Local stdio — Claude Code (export `RANLA_API_KEY` where Claude Code runs):

```bash
claude mcp add --transport stdio ranla -- npx -y ranla-mcp@latest
```

Restart Cursor after changing its configuration.

## Growth work (default)

1. `arc_ensure_thread` (the title can be the current task).
2. `arc_message` with the user’s request. Ranla chooses reads, drafts, and
   changes that need approval. The result has `text`, `artifacts`
   (`{ type, id, label }` for drafts, plans, and documents it made or opened),
   and `pendingApproval` when a step needs a yes.
3. If the turn returns `pendingApproval`, show it and wait for the user. Then
   `arc_list_approvals` → `arc_approve` or `arc_reject`.
4. Call `arc_*` read tools directly when the user wants a snapshot without a
   strategy turn: audience, campaigns, templates, analytics, revenue, SEO, AI
   search, Reddit, paid, site traffic, connected repos. The list and input
   schemas come from the workspace when tools are listed.

Do **not** schedule or send campaigns except by asking Ranla via
`arc_message`. Ranla’s write tools are not exposed directly;
`arc_request_connection` is the only exception.

`arc_request_connection` returns a connect link when the user needs an
integration. `slot` is one of `marketing_site`, `slack`, `search_console`,
`stripe`, `google_ads`, `dns`, `audience`, `github`, `product_events`,
`posthog`, `amplitude`, or `mixpanel`.

## Transactional mail (same server)

Use these only when the user asks to send, inspect, or configure mail —
receipts, auth codes, alerts, or domain setup:

- Send: `send_email` (now, or later with `scheduled_at`), `cancel_email`,
  `list_emails`, `get_email`
- Inbound: `list_received_emails`, `get_received_email`
- Domains: `list_domains`, `get_domain`, `create_domain`, `apply_domain_dns`,
  `verify_domain`
- Templates: `list_templates`, `get_template`, `create_template`,
  `publish_template`
- Webhooks: `list_webhooks`, `create_webhook`, `delete_webhook`,
  `send_test_webhook_event`
- Suppressions and deliverability: `list_suppressions`, `add_suppression`,
  `remove_suppression`, `get_deliverability`

Production `from` addresses need a verified domain. Before that, send from
`noreply@mail.ranla.ai` to the account email only. This is application mail,
not a growth campaign — do not route it through `arc_message`.

## Application code

For adding Audience contacts and lifecycle events in the repo, use the `ranla`
skill and the Ranla SDK. Use `emails.send` on that SDK when the user also wants
transactional mail in code.

## References

- MCP setup: https://docs.ranla.ai/ai/mcp
- Instrumentation: https://docs.ranla.ai/instrument-quickstart
