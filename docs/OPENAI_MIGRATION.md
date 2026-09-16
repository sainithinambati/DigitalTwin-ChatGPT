# OpenAI Codex subscription migration reference

Verified against the official OpenAI model catalog and the Hermes v0.20.0
provider implementation on 2026-09-17.

| Lab tier | Requested name | Model ID | Hermes provider |
|---|---|---|---|
| Economy | GPT-5.6 Terra | `gpt-5.6-terra` | `openai-codex` |
| Frontier | GPT-6 Astra | `gpt-6-astra` | `openai-codex` |

The model IDs are never prefixed in `model.default`. The pinned Hermes Codex
catalog explicitly includes `gpt-5.6-terra`; access to any model, including
`gpt-6-astra`, remains subject to the live catalog and the signed-in account's
entitlements. The assignment's second case uses Terra only. See the official
[GPT-5.6 Terra model page](https://developers.openai.com/api/docs/models/gpt-5.6-terra).

## Runtime path

Hermes v0.20.0 is configured with the explicit `openai-codex` provider. This
provider authenticates through the student's ChatGPT/Codex subscription by
device-code OAuth. It is distinct from `openai-api`, which is the public,
metered API-key provider.

The generated Hermes configuration pins `api_mode: codex_responses` and
`reasoning_effort: medium` for the main agent and every auxiliary model call.
`dtlab-start` also clears inherited `OPENAI_API_KEY` and `OPENAI_BASE_URL`
variables so they cannot change the subscription route.

`dtlab-start` keeps the existing Digital Twin safeguards:

- a fresh `HERMES_HOME` and configuration for every run;
- an explicit `model.provider: openai-codex` plus a bare model ID in
  `model.default`;
- file-level and `hermes config get model.default` verification before launch;
- one model ID and configuration hash in each run's evidence.

The earlier failed migration used `model.id`, which Hermes v0.20.0 ignores;
this version uses `model.default`. Another failed form prefixed the model as
`openai-api/gpt-5.6-terra`, causing the provider to receive a nonexistent
literal model ID. Both failure modes are now covered by tests.

## Authentication and secret handling

No `OPENAI_API_KEY`, Platform billing project, or prepaid API credits are
required for the `openai-codex` route. On the first `dtlab-start`, Hermes shows
a device URL and one-time code. Complete that login with the ChatGPT account
whose plan includes Codex access.

Hermes stores the resulting credential in `~/.hermes/auth.json`. The launcher
rejects an unsafe symlink/non-file credential path and restricts the directory
and file to owner-only access. Per-run Hermes homes use Hermes's global auth
fallback; the OAuth credential is not copied into run directories, evidence,
Git, or recordings. Never paste tokens into `dtlab_config.env` or any tracked
file.

To renew an expired or revoked login, run:

```bash
HERMES_HOME="$HOME/.hermes" hermes auth add openai-codex --type oauth
```

Then rerun `dtlab-start`. Subscription usage is governed by the signed-in
account's plan and current Codex limits. Token-cost fields in evidence are
normalized research estimates, not an OpenAI invoice or a subscription charge.

Provisioning retains `openai==2.24.0`, the exact core SDK version required by
the pinned Hermes release's Codex Responses transport.

Hermes's pinned provider documentation is available in its
[`providers.md`](https://github.com/NousResearch/hermes-agent/blob/3c27eb6234bf91b8ceee9e9071591b31e9b148cb/website/docs/integrations/providers.md).
