---
name: mida-setup
description: >-
  Set up Mida A/B testing on a site or app from scratch, working in the user's
  codebase: create the Mida account if they have none, get an API key, install
  the tag or a server-side SDK, confirm data is arriving, and get a first draft
  experiment ready to launch. Use when the user asks to "add A/B testing", "set
  up Mida", "install Mida", "start testing on this site", or wants an experiment
  wired into their own code rather than run from the dashboard.
---

# Setting up Mida from a codebase

This is the path for a user who is starting from nothing, with you working in
their repository. If they already have a Mida account and the tag installed, go
to the `mida-experiments` skill instead.

Everything here uses Mida's REST API with an API key. The endpoint list with
request bodies is at https://docs.mida.so/llms.txt. Read the page for an
endpoint before calling it; do not guess field names.

## 1. Get an API key

Ask first: "Do you already have a Mida account?"

**They have one.** They create a key in the dashboard (Settings, API) and give
you the key and the project key. Store both as secrets (step 2).

**They do not.** You can create the account for them with one emailed code.

1. Show them https://www.mida.so/terms and ask which email address to use.
   Registering accepts the terms on their behalf, so do not skip the question.
2. Ask where their visitors are. EU accounts use `https://api-eu.mida.so`;
   everyone else uses `https://api-us.mida.so`. An account cannot change region
   later, and a tag from the wrong region silently finds no experiments.
3. Register, with your own agent name as `client`:

   ```bash
   curl -X POST https://api-us.mida.so/v2/agent/register \
     -H "Content-Type: application/json" \
     -d '{"email": "person@example.com", "terms_accepted": true, "client": "<your agent name>"}'
   ```

   Keep the `claim_token`. Mida emails the person a 6-digit code.
4. Ask them for the code, then claim:

   ```bash
   curl -X POST https://api-us.mida.so/v2/agent/claim \
     -H "Content-Type: application/json" \
     -d '{"claim_token": "...", "code": "123456"}'
   ```

   The response has `api_key` (shown once) and `project_key`.

In a Node project, `npx mida-node init --email person@example.com --accept-terms`
(mida-node 1.2.0 or later) does steps 3 and 4 and writes the keys to `.env`.
`mida register` and `mida claim --code` are the same steps without a prompt.

If no email arrives within a few minutes, the address most likely has an
account already. Mida gives the same answer either way, on purpose. Send them
to the dashboard for a key.

Tell the person their account has no password yet: they set one at the
dashboard's "Forgot password" page, with the same email, when they want to sign
in.

## 2. Store the keys as secrets

Put `MIDA_API_KEY`, `MIDA_PROJECT_KEY` and `MIDA_REGION` in the project's
secret store or `.env`, and make sure that file is gitignored. Never commit the
API key, print it in a summary, or put it in client-side code. The project key
is public (it appears in the tag); the API key is not.

Every call after this sends `Authorization: Bearer <api_key>` to
`https://api-<region>.mida.so/v2/project/<project_key>/...`.

## 3. Pick the track

| | Client-side tag | Server-side SDK |
|---|---|---|
| The variant is | CSS and JavaScript Mida applies in the browser | A branch in the user's own code |
| Good for | Copy, layout, styling, landing pages | Pricing logic, algorithms, backend flows, server-rendered pages where flicker matters |
| Install | One script tag in `<head>` | A package and two calls |
| The winner ships by | Editing the source to match the variant | Deleting the losing branch |

Ask if it is not obvious. Most marketing sites want the tag.

### Client-side tag

1. `GET .../install-snippet`. Paste `tag` into the site's `<head>` exactly as
   returned: as high as possible and not deferred.
   Never assemble the tag by hand, because the host differs by region.
2. Put it in the shared layout so it is on every page, then deploy.
3. Load a page on the deployed site, then `GET .../install-health`. Poll at
   most every 30 seconds. `not_installed` means no visitor has arrived yet;
   `no_test_collecting` means the tag works and nothing is live, which is the
   state you want before the first test.

### Server-side SDK

1. Install the SDK for the stack. Node: `npm install mida-node`. The others are
   listed at https://docs.mida.so/docs/server-side/sdks.
2. Create one client at startup with the project key (and `region: 'eu'` for an
   EU account).
3. Create the experiment with an `experiment_key` (step 4), then branch on it:

   ```javascript
   const variant = await mida.getExperiment('pricing-page-cta', userId)
   if (variant === 'Variant 1') { /* treatment */ } else { /* control, or null */ }
   ```

   `getExperiment` returns `null` until the experiment is live and whenever the
   request fails. Treat `null` as control, and catch errors so a Mida outage
   never breaks the page.
4. Record the conversion with the same user id: `mida.setEvent('signup_completed', userId)`.
   The event name must match the goal's event name.
5. Deploy the code first, then launch. Launching before the code is deployed
   enrols nobody.

## 4. Build the first experiment as a draft

1. Create a goal (Create Goal). A test with no goal measures nothing.
2. Create the experiment (Create Experiment). It is created as a draft. For
   server-side, send `experiment_key`.
3. Client-side: get the preview links (Get Experiment Preview URLs) and give
   them to the person to open. You cannot see their page unless you have a
   browser.
4. `GET .../experiment/<test_id>/pre-publish-check`. Fix every item under
   `blockers` and tell the person about `warnings`. It reads the saved
   configuration only, so `ready: true` does not mean the variant renders
   correctly; the preview link is what shows that.

## 5. Launching is the person's decision

Do not set an experiment live on your own. Launching changes what real
visitors see. Tell the person what will run, on which pages, for how much
traffic, and ask. Only then call Update Experiment Status.

The REST API does not stop you from launching without asking. That is why this
rule is yours to keep.

## 6. Reading and finishing

Use Get Experiment Result, and read it the way the `mida` skill describes
(Bayesian and frequentist results are read in opposite directions).

When the person calls a winner, the order matters:

1. Stop the experiment first (`POST .../experiment/<test_id>/complete`).
2. Then ship the winner in the source: edit the page to match the winning
   variant, or delete the losing branch and the `getExperiment` call.

If you ship the code while the test is still running, the control arm starts
showing the new version too, both arms become identical, and the data collected
after that is noise.

## When something looks wrong

| Symptom | First thing to check |
|---|---|
| `install-health` stays `not_installed` | Is the tag in the deployed HTML, in `<head>`, and has anyone loaded the page? |
| Tag is present, no experiments run | Region. An EU account needs the `cdn-eu.mida.so` tag |
| `getExperiment` always returns `null` | The experiment is still a draft, or the key does not match |
| `401 Your API key is wrong` | The key belongs to another project |
| `404 Project not found` | The region host is wrong: an EU project only exists on `api-eu.mida.so` |
| Register returns `rate_limited` | Wait an hour. Do not retry in a loop |
