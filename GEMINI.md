# Mida

[Mida](https://www.mida.so) is an A/B testing and personalization platform. A
JavaScript tag on the user's site puts visitors into variants and reports
conversions. This extension connects to their Mida account over the hosted MCP
server at `https://mcp.mida.so/mcp`, authenticated with OAuth on first use.

The `skills/` directory carries the full guidance. The rules below are the ones
that cause real damage when missed.

## Always select a project first

A Mida login can own several organizations and projects, and the session opens
on a default project that is often not the one the user means. Calls scoped
elsewhere return `Mida API error 404: Experiment not found`, which reads like a
deleted test rather than a scoping miss.

Start with `list_organizations` and `select_organization` (only when there is
more than one), then `list_projects` and `select_project`. Never report a 404 as
"that experiment doesn't exist" until the right project is selected.

## "Experiment #208" is not `test_id` 208

Each account numbers its own experiments, and that number sits at the start of
the experiment's name, e.g. `#208 | Pricing page | Annual toggle default`. Match
it with `list_experiments`, not by passing the number to `get_experiment`.

## Reading results

`get_experiment_result` follows the project's statistics mode, and the two modes
read in opposite directions:

- **Bayesian (default):** "Chance to Beat Original". Higher is better.
- **Frequentist:** "P-value". Lower is better.

Chance-to-beat only answers "is the variant better?". A low value means "not
shown to be better", not "shown to be worse", so use the observed lift and
sample size to describe a loss. Check `compute_experiment_statistics` and
`get_experiment_timeseries` before calling any result: a few hundred visitors
per arm will swing day to day.

## Order of operations

- **Concluding.** `update_experiment_status` resets the completed flag, so set
  the status first and call `conclude_experiment` last, or the write-up is lost.
- **Serving a winner.** `conclude_experiment` ends delivery on its own. A test
  with a served winner keeps serving that variant after it is concluded.
- **Hardcoding a winner.** Stop the test before deploying the winning copy to
  source, or the control arm serves the new copy too and the data turns to noise.

## Before anything runs

Nothing works until Mida's tag is on the site. `get_install_snippet` returns it.
If a live test reports no visitors at all, check the tag before the targeting.
