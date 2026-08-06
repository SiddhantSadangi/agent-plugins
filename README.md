<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="unify/assets/unify-logo-dark.svg">
    <img src="unify/assets/unify-logo-light.svg" alt="Unify" width="180">
  </picture>
</p>

<p align="center">
  <strong>Unify in your AI agent</strong>: skills and the Unify MCP server,
  packaged for the Agent Plugins standard, Claude Code, Cursor, and Codex.
</p>

<p align="center">
  <a href="https://www.unifygtm.com">Unify</a> · <a href="https://docs.unifygtm.com/developers/mcp">Docs</a> · <a href="https://www.unifygtm.com/legal/privacy-policy">Privacy Policy</a> · <a href="https://www.unifygtm.com/legal/terms-and-conditions">Terms of Service</a> · <a href="https://www.unifygtm.com/legal/customer-data-processing-agreement">DPA</a> · <a href="https://trust.unifygtm.com/">Trust Center</a>
</p>

Unify is AI-powered GTM for sellers. Find companies and people,
enrich contact data, and engage prospects with sequences and tasks, all from natural language in your favorite agent.

## Getting started

### From the Terminal

Copy this prompt into your terminal based agent of choice:

```
Read https://raw.githubusercontent.com/unifygtm/agent-plugins/main/GETTING_STARTED.md and follow to setup the unify plugin.
```

or run the setup script directly.

```bash
curl -fsSL https://raw.githubusercontent.com/unifygtm/agent-plugins/main/scripts/setup.sh | bash
```

See [GETTING_STARTED.md](GETTING_STARTED.md) for agent-specific details.

## Portable plugin

The [`unify/`](unify/) directory conforms to the
[Agent Plugins 1.0.0 specification](https://agent-plugins.org/specification).
Compatible clients can discover its root [`plugin.json`](unify/plugin.json),
Agent Skills under [`skills/`](unify/skills/), and Streamable HTTP MCP server
configuration in [`mcp.json`](unify/mcp.json).
