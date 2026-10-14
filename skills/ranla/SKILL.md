---
name: ranla
description: >-
  Connect Ranla to a product for lifecycle email — Audience contacts, lifecycle
  events, activation, conversion, and campaign context. Use transactional email
  only when the user explicitly asks for it.
license: MIT
metadata:
  author: ranla
  homepage: https://docs.ranla.ai/ai/agent-skills
  source: https://github.com/Super-Send/ranla-skills
  documentation: https://docs.ranla.ai/instrument-quickstart
---

# Ranla

Ranla is an AI agent for the signups you already have. Its primary integration is the
product’s lifecycle data: who signed up, what they did, when they activated,
and when they became or stopped being a customer.

## Defaults

- Use `RANLA_API_KEY` from the server environment. Never expose it to a browser
  bundle, commit it, or print it in logs.
- Use `https://api.ranla.ai` unless the user gives a different base URL.
- Read the repository’s existing analytics and identity conventions before
  adding instrumentation.
- Prefer a small, reliable integration over a new analytics abstraction.
- Do not change working transactional mail paths unless the user explicitly
  asks for transactional email too.

## Instrumentation

At signup, upsert the person into Audience with the email, stable product user
id, and a small set of useful attributes already available at the call site.
Do not dump the full user or database row.

Emit the lifecycle events named in the business brief at their real product
moments. Include `email` and `user_id` when available. At minimum, wire the
brief’s activation and payment events when those call sites exist.

With the Node SDK:

```ts
import { Ranla } from '@supersend/ranla'

const ranla = new Ranla(process.env.RANLA_API_KEY!)

await ranla.contacts.upsert({
  email: user.email,
  external_id: user.id,
  attributes: { plan: user.plan },
})

await ranla.events.trigger({
  name: 'user.activated',
  user_id: user.id,
  email: user.email,
  data: { plan: user.plan },
  idempotencyKey: `evt_user_activated_${user.id}`,
})
```

Use the repository’s package manager and existing test conventions. Add a
focused test at the call site when practical.

## Optional transactional email

Ranla can also send transactional email when the product needs it:

```ts
await ranla.emails.send({
  from: 'you@yourdomain.com',
  to: user.email,
  subject: 'Your receipt',
  html: '<p>Thanks for your purchase.</p>',
})
```

This is an available capability, not the default setup task. Do not replace a
working provider or add a parallel mail path without an explicit request.

When you do wire it:

- **Keys:** `RANLA_API_KEY` (`rnl_…`) in server env only. Older `stx_` keys and
  the `SUPERSENDTX_API_KEY` variable still work; don't migrate a working setup
  unprompted. Never ship a key to a browser or mobile bundle.
- **Sandbox first:** before the customer's domain is verified, send from
  `noreply@mail.ranla.ai` to the account owner's email only. Other recipients
  are rejected with `403`.
- **Verify the domain:** `POST /domains` with `{ "name": "yourdomain.com" }`,
  then `{ "action": "apply" }` and, after DNS propagates, `{ "action": "verify" }`
  on `POST /domains/{id}`. Production `from` must be on that verified domain.
  Say plainly when the customer still needs DNS or registrar access.
- **Templates:** send a published template with `template: { id, variables }`
  instead of `html`/`text`/`react`. For React Email, author with React Email's
  own tooling and pass `react` or the rendered HTML.
- **Bounces and complaints:** check `GET /suppressions` before retrying; treat a
  complaint as a permanent opt-out. Use webhooks (`POST /webhooks`) or poll
  `GET /events` for delivered, bounced and complained.
- **Errors:** `401` bad key; `403` unverified domain, sandbox recipient, or an
  account restriction; `400` validation. Body: `{ "error": { "message", "details" } }`.

## References

- Instrumentation: https://docs.ranla.ai/instrument-quickstart
- API reference: https://docs.ranla.ai/api
- MCP setup: https://docs.ranla.ai/ai/mcp
- Deliverability: https://docs.ranla.ai/deliverability
- Email guides: https://docs.ranla.ai/guides/react-email
