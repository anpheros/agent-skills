# Anpheros MCP server and agent skills for FHIR health data

**Teach coding agents to build health apps on a FHIR R4 healthcare API.** An Agent Skill, a Claude Code plugin,
Cursor rules, AGENTS.md and MCP configuration for [Anpheros](https://developers.anpheros.com/guides/?utm_source=github&utm_medium=readme&utm_content=agent-skills):
an HL7 FHIR R4 backend with patient consent, provenance and a free sandbox with 30 synthetic patients.

| Tool | What to use |
|---|---|
| Claude Code | `/plugin marketplace add anpheros/agent-skills` then `/plugin install anpheros@anpheros` |
| Agent Skills (Claude, other agents) | [`skills/anpheros-health-backend/SKILL.md`](skills/anpheros-health-backend/SKILL.md) |
| Cursor | copy [`.cursor/rules/anpheros.mdc`](.cursor/rules/anpheros.mdc) into your project |
| Codex, Jules, others | copy [`AGENTS.md`](AGENTS.md) into your project |
| Any MCP client | Docs server, no account: `https://platform.anpheros.com/mcp/docs` · FHIR data server: `https://platform.anpheros.com/mcp` |

## Quick start in 2 minutes

1. [Get a free sandbox key](https://platform.anpheros.com/dashboard/?utm_source=github&utm_medium=readme&utm_content=agent-skills) (`sk_test_…`).
2. Install the plugin and connect the FHIR MCP server in Claude Code:

   ```bash
   claude plugin marketplace add anpheros/agent-skills
   claude plugin install anpheros@anpheros
   claude mcp add --transport http anpheros-fhir https://platform.anpheros.com/mcp --header "X-API-Key: sk_test_…"
   ```

3. Ask your agent: *"Build a small TypeScript service that lists my sandbox patients and their latest lab results.
   Check the calls against the sandbox first."*

The agent reads the docs through the Docs MCP server, tries calls on your sandbox through the FHIR MCP server and
writes code with the official SDKs ([TypeScript, Dart / Flutter](https://github.com/anpheros/anpheros-sdk)).
Everything about the MCP server, with configs for Cursor and VS Code: [anpheros/mcp-server](https://github.com/anpheros/mcp-server).

## The rules every file teaches

- start on the sandbox (`sk_test_…`), never with real patient data;
- never invent medical codes: look up LOINC, ICD-10 and ATC codes;
- reach a person's data only with their consent (OAuth 2.1 / SMART, PKCE);
- never present diagnoses or treatment advice.

## Guides

[Building with AI coding tools](https://developers.anpheros.com/guides/ai-developers?utm_source=github&utm_medium=readme&utm_content=agent-skills) ·
[MCP server](https://developers.anpheros.com/guides/mcp?utm_source=github&utm_medium=readme&utm_content=agent-skills) ·
[AI agents and medical data](https://developers.anpheros.com/guides/ai-agents-medical-data?utm_source=github&utm_medium=readme&utm_content=agent-skills) ·
[FHIR sandbox](https://developers.anpheros.com/guides/fhir-sandbox?utm_source=github&utm_medium=readme&utm_content=agent-skills) ·
[Getting started](https://developers.anpheros.com/guides/getting-started?utm_source=github&utm_medium=readme&utm_content=agent-skills)

Apache-2.0 · [anpheros.com](https://anpheros.com/?utm_source=github&utm_medium=readme&utm_content=agent-skills)
