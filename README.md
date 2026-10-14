# Ranla Agent Skills

Official [Agent Skills](https://agentskills.io) for [Ranla](https://ranla.ai), the AI growth agent for software businesses. They work in Claude Code, Cursor, Codex and other agents that read the Agent Skills format.

## Install

```bash
# All skills
npx skills add Super-Send/ranla-skills -y

# Or one at a time
npx skills add Super-Send/ranla-skills --skill ranla -y
npx skills add Super-Send/ranla-skills --skill ranla-mcp -y
npx skills add Super-Send/ranla-skills --skill ranla-cli -y
```

Restart or reload your agent afterwards so it picks the skills up.

## Skills

| Skill | What it teaches the agent |
|-------|---------------------------|
| [`ranla`](./skills/ranla/SKILL.md) | Wire sign-ups, activation and payment events into Ranla, and send transactional email when the product needs it |
| [`ranla-mcp`](./skills/ranla-mcp/SKILL.md) | Work with Ranla through its MCP server: growth health, campaigns, pages, replies and approvals |
| [`ranla-cli`](./skills/ranla-cli/SKILL.md) | Drive Ranla from the terminal with `ranla-cli`: browser sign-in, Ranla's commands, approvals |

Nothing is sent, spent or published until you approve it.

Docs: [docs.ranla.ai/ai/agent-skills](https://docs.ranla.ai/ai/agent-skills) · Questions: contact@ranla.ai
