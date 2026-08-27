<p align="center">
  <img src="assets/logo.svg" alt="Mida" width="220">
</p>

<h3 align="center">Test everything. Personalize everyone.</h3>

<p align="center">
  <a href="https://www.mida.so">Website</a> ·
  <a href="https://app.mida.so">Dashboard</a> ·
  <a href="https://github.com/mida-so/api-docs">API docs</a>
</p>

---

# Mida plugin

Run A/B tests and personalizations on your own website, from your coding agent.

[Mida](https://www.mida.so) is an A/B testing and personalization platform. A
small JavaScript tag on your site puts visitors into variants and reports
conversions. This plugin connects your agent to your Mida account, so you can
build a test, launch it, read the result, conclude it, and roll the winner out
without switching tabs.

```
"What's running on the pricing page right now?"
"Create a test for a shorter signup form and preview variant B"
"Has the homepage hero test reached significance yet?"
"Conclude experiment #208, variant B won, we're shipping it"
```

## Install

**Gemini CLI**

```
gemini extensions install https://github.com/mida-so/mida-plugin
```

**Any MCP client**, point it at the hosted server:

```json
{ "mcpServers": { "mida": { "type": "http", "url": "https://mcp.mida.so/mcp" } } }
```

Per-client setup instructions live at [mida.so/mcp](https://www.mida.so/mcp).

Listings for Grok Build and Cursor are in review. Once they land, it's
`/plugin install mida` in either one.

On first use your client opens a browser to sign in to Mida with OAuth. There is
no API key to paste and nothing is written to your machine.

You need a [Mida account](https://www.mida.so) and the Mida tag installed on the
site you want to test. Ask your agent for the install snippet once you're
connected.

## What's in the box

**MCP server.** The hosted [Mida MCP server](https://mcp.mida.so/mcp), covering
experiments (create, launch, update, conclude, reopen), results and statistics,
goals and custom events, hypotheses, exclusion groups, personalization
campaigns, project configuration, and team access.

**Skills**

| Skill | Covers |
|---|---|
| `mida` | Entry point: picking the right project, reading Bayesian vs. frequentist results, and the order of operations for concluding a test and serving a winner |
| `mida-experiments` | The workflow: hypothesis, variants, goal, preview, launch, then calling the result |

The skills are here because a few Mida behaviours are easy to get wrong from the
tool schemas alone. An experiment's display number is not its API id. A
concluded test stops serving even while its status still reads active. Setting a
test's status clears a conclusion written before it. Each one is documented with
the correct sequence.

## Network and credentials

| | |
|---|---|
| **Endpoints called** | `https://mcp.mida.so/mcp`, the hosted Mida MCP server, and the only host this plugin contacts. It reaches Mida's API server-side. |
| **Credentials** | OAuth to your Mida account, handled by the MCP client. No API keys, no environment variables, no local credential files. |
| **Scope** | Your own Mida organizations and projects. The plugin reads and writes experiment configuration and reads result data for those projects. |
| **Local execution** | None. No hooks, no commands, no scripts. One remote MCP server plus Markdown skills. |
| **Telemetry** | None beyond the MCP calls themselves. |

## About Mida

[Mida](https://www.mida.so) is A/B testing and personalization in one engine.
Testing answers which version works. Personalization answers who it works for.
Both run as real variants in a real test, so personalization is something you
can actually prove rather than take on faith.

- [A/B testing](https://www.mida.so) on any site, with a tag that loads in about 20ms
- [Mida dashboard](https://app.mida.so) for results, goals, and rollouts
- [Public API docs](https://github.com/mida-so/api-docs)

## License

MIT, see [LICENSE](LICENSE).
