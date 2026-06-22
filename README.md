# hermes-skill-tossinvest-openapi

Standalone Hermes skill for integrating Toss Securities Open API.

## Contents

- `SKILL.md` — main Hermes skill
- `references/tossinvest-openapi-summary.md` — concise endpoint/auth/rate-limit notes
- `references/local-cli-cookbook.md` — example commands for the companion CLI

## Install

After publishing to GitHub, install directly from the raw SKILL URL:

```bash
hermes skills install https://raw.githubusercontent.com/kuil09/hermes-skill-tossinvest-openapi/main/SKILL.md
```

## Design goals

- Programmatic, agent-friendly Toss Open API workflows
- Human approval gate before live order execution
- Explicit handling of `X-Tossinvest-Account`
- No embedded secrets; environment variables only

## Companion repository

This skill is designed to pair with `tossinvest-openapi-cli`.

## License

MIT
