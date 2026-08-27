# Mida plugin for Grok Build

Run A/B tests and personalizations on your own website from inside Grok Build.

[Mida](https://www.mida.so) is an A/B testing and experimentation platform. A
JavaScript tag on your site assigns visitors to variants and reports
conversions. This plugin connects Grok to your Mida account so you can build a
test, launch it, read the result, conclude it, and roll the winner out — in one
conversation.

```
"What's running on the pricing page right now?"
"Create a test for a shorter signup form and preview variant B"
"Has the homepage hero test reached significance yet?"
"Conclude experiment #208 — variant B won, we're shipping it"
```

## Install

```
/plugin install mida
```

On first use, Grok opens a browser to sign in to Mida with OAuth. There is no
API key to paste and nothing is written to your machine.

You need a Mida account and the Mida tag installed on the site you want to test.
Sign up at [mida.so](https://www.mida.so); ask Grok for the install snippet once
you're connected.

## What's in the box

**MCP server** — the hosted Mida MCP server, covering experiments (create,
launch, update, conclude, reopen), results and statistics, goals and custom
events, hypotheses, exclusion groups, personalization campaigns, project
configuration, and team access.

**Skills**

| Skill | Covers |
|---|---|
| `mida` | Entry point: project selection, reading Bayesian vs. frequentist results, the order of operations for concluding a test and serving a winner |
| `mida-experiments` | The workflow: hypothesis → variants → goal → preview → launch → call it |

The skills exist because several Mida behaviours are easy to get wrong from the
tool schemas alone — an experiment's display number is not its API id, a
concluded test stops serving even while its status reads active, and setting a
test's status clears a conclusion written before it. Each is documented with the
correct sequence.

## Network and credentials

Declared for review, per the marketplace's security expectations:

| | |
|---|---|
| **Endpoints called** | `https://mcp.mida.so/mcp` — the hosted Mida MCP server, and the only host this plugin contacts. It talks to Mida's API (`api.mida.so`) server-side. |
| **Credentials** | OAuth to your Mida account, handled by the MCP client. No API keys, no environment variables, no local credential files. |
| **Scope** | Your own Mida organizations and projects. The plugin reads and writes experiment configuration and reads result data for those projects. |
| **Local execution** | None. No hooks, no commands, no scripts, no filesystem or shell access — the plugin is one remote MCP server plus Markdown skills. |
| **Telemetry** | None beyond the MCP calls themselves. |

## Links

- Product: [mida.so](https://www.mida.so)
- Dashboard: [app.mida.so](https://app.mida.so)
- API docs: [github.com/mida-so/api-docs](https://github.com/mida-so/api-docs)

## License

MIT — see [LICENSE](LICENSE).
