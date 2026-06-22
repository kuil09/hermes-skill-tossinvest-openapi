# tossinvest-openapi-agent-pack

Agent-compatible instruction pack for integrating the Toss Securities Open API.

This repository is **not Hermes-only**.
It includes a Hermes `SKILL.md`, but the core guidance is plain Markdown so it can also be reused by other agent frameworks, coding assistants, and prompt-driven automation systems.

## What is in this repository

- `SKILL.md` — Hermes-compatible skill packaging
- `AGENT_GUIDE.md` — agent-agnostic operational guide
- `SYSTEM_PROMPT.md` — copy/pasteable prompt block for non-Hermes agents
- `references/tossinvest-openapi-summary.md` — concise endpoint/auth/rate-limit notes
- `references/local-cli-cookbook.md` — example commands for the companion CLI

## Intended consumers

- Hermes Agent
- Codex / OpenAI tool-using agents
- Claude Code / other coding agents
- Custom internal agents that ingest Markdown instructions
- Workflow engines that want a reusable policy + command cookbook

## Design goals

- Programmatic, agent-friendly Toss Open API workflows
- Human approval gate before live order execution
- Explicit handling of `X-Tossinvest-Account`
- No embedded secrets; environment variables only
- Reusable guidance in framework-neutral Markdown

## Use with Hermes

Install directly from the raw `SKILL.md` URL:

```bash
hermes skills install https://raw.githubusercontent.com/kuil09/hermes-skill-tossinvest-openapi/main/SKILL.md
```

## Use with non-Hermes agents

Start with these files:

1. `AGENT_GUIDE.md` — operational rules and workflow
2. `SYSTEM_PROMPT.md` — concise instruction block to paste into your agent system prompt or reusable context
3. `references/*.md` — supporting details and command examples

## Companion repository

This instruction pack is designed to pair with:

- [kuil09/tossinvest-openapi-cli](https://github.com/kuil09/tossinvest-openapi-cli)

## License

MIT
