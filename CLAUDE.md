# TossInvest OpenAPI Context for Coding Agents

This repository can be loaded as reusable context for coding agents beyond Hermes.

## Primary files

- `AGENT_GUIDE.md` — framework-neutral operating guidance
- `SYSTEM_PROMPT.md` — concise reusable instruction block
- `SKILL.md` — Hermes packaging of the same operational guidance
- `references/` — supporting notes and examples

## Core policy

- environment-variable credentials only
- explicit `X-Tossinvest-Account` handling for account-scoped APIs
- approval gate before live execution
- structured error handling
- allowlist/IP diagnosis support
- rate-limit-aware automation

## Companion CLI

Pair this context with:

- `https://github.com/kuil09/tossinvest-openapi-cli`
