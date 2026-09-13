# OpenAI migration reference

Verified against the official OpenAI model catalog on 2026-09-14.

| Lab tier | Requested name | API model ID | Hermes provider |
|---|---|---|---|
| Economy | GPT-5.6 Terra | `gpt-5.6-terra` | `openai-api` |
| Frontier | GPT-6 Astra | `gpt-6-astra` | `openai-api` |

Both requested names have exact official API identifiers; no substitute
or invented mapping is used. See the official model pages for
[GPT-5.6 Terra](https://developers.openai.com/api/docs/models/gpt-5.6-terra)
and [GPT-6 Astra](https://developers.openai.com/api/docs/models/gpt-6-astra).

## Runtime path

Hermes v0.20.0 is configured with the explicit `openai-api` provider.
Do not shorten this to `openai`: in this pinned Hermes release that alias
routes through OpenRouter rather than directly to OpenAI. Hermes sends
model traffic through OpenAI's Responses API. The kit does not set
temperature, `top_p`, or other sampling overrides.

The generated Hermes configuration pins `api_mode: codex_responses`.
`dtlab-start` also clears any inherited `OPENAI_BASE_URL`, because Hermes
honors that variable and this kit must send first-party keys only to
`https://api.openai.com/v1`.

`dtlab-start` keeps the existing Digital Twin safeguards:

- a fresh `HERMES_HOME` and configuration for every run;
- a combined `model.default` value of `openai-api/<model-id>`;
- file-level and `hermes config get model.default` verification before
  launch;
- one model ID and configuration hash in each run's evidence.

The previous migration attempt visible in the template addressed a
different issue: the old Hermes schema used `model.id`, while v0.20.0
requires `model.default`. That old configuration was ignored and could
fall back to a default model. Its fail-closed repair remains intact.

## Credential and dependency setup

Set `OPENAI_API_KEY` in the runtime environment. `dtlab-start` prompts
with hidden input, verifies the credential using Bearer authentication
against `GET https://api.openai.com/v1/models`, and writes it only to the
student's permission-restricted `~/.dtlab_env`. The launcher parses that
file as a single assignment rather than executing it as shell code, rejects
symlinks, and passes the validation header to curl over stdin so the key is
not present in curl's command-line arguments. The key is exported only to the
launcher/Hermes process and is not sourced globally from `.bashrc`. Never add a key to
`dtlab_config.env` or any tracked file.

Cost capture uses the published per-model rates, including cached input at
0.1x and prompt-cache writes at 1.25x the uncached input rate. OpenAI's
long-context surcharge applies per request above 272K input tokens; Hermes's
stored run totals are aggregated and cannot reliably identify those requests,
so `token_usage.json` remains an estimate rather than a billing statement.

Provisioning installs `openai==2.24.0`, the exact core OpenAI SDK version
pinned by Hermes v0.20.0. A live end-to-end model call still requires a
funded OpenAI API project, a project API key, and sufficient project
limits.

OpenAI's current API data-control details are documented in
[Data controls in the OpenAI platform](https://developers.openai.com/api/docs/guides/your-data).
