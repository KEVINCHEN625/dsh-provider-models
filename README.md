# dsh-provider-models

Channel availability for [dsh-provider-manager](https://github.com/KEVINCHEN625/dsh-provider-manager).
These files say which subscription models are enabled. They do not carry model
parameters, API keys, access tokens, or refresh tokens.

Parameters come from [models.dev](https://models.dev/api.json). This repository
is the only custom dataset.

The plugin ships a snapshot and refreshes it from here when an OAuth details
page opens or a sign-in succeeds. The default age is 24 hours
(`oauthCatalogTtlMs`). A failed download keeps the snapshot that shipped with
the plugin.

## Freeze warning for 0.3.x

Documents on this branch add `status`, `source`, and Codex `noneEnabled`.
Plugin 0.3.x rejects unknown keys, so the whole file fails to parse and that
install falls back to the snapshot packed inside 0.3.x. That snapshot still
injects `reasoningEfforts.none`, which the host schema rejects. Install
**0.4.0** before relying on this catalog. 0.4.0 reads `status` first and
writes the host key `off` with wire value `"none"`.

## Layout

`<providerId>.json`

```json
{
  "providerId": "openai-codex",
  "models": [
    {
      "id": "gpt-6-sol",
      "available": true,
      "status": "available",
      "source": "probe",
      "verifiedAt": "2026-09-24",
      "noneEnabled": true
    }
  ]
}
```

`status` is `available`, `unavailable`, or `unverified`. `source` is
`probe` or `models.dev`. `available` remains for older readers that only
understand the boolean. `noneEnabled` is a channel fact for the Codex
`none` effort. The plugin maps it to `reasoningEfforts.off = "none"`.
`servedModel` is set when a request for `id` returned HTTP 200 from a
different model. `contextWindow` is set only when the channel window
disagrees with models.dev.

Rows that were never probed use `status: "unverified"` and `available: false`.
They stay out of the selector until a sign-in probe returns HTTP 200.
`gpt-5.3-codex` and `gpt-5.3-codex-spark` are `unavailable` because the
Codex probe returned HTTP 400.

## Providers

- `openai-codex.json`: probed. Six models have `noneEnabled: true`. `gpt-6-astra` is false.
- `anthropic.json`, `github-copilot.json`, `kimi-coding.json`, `xai.json`, `openrouter.json`: unverified seeds from models.dev. OpenRouter is the free group, the 20 newest `release_date` values, and a fixed allowlist, capped at 50. xAI lists five chat models that declare reasoning.

## Updating Codex

1. Diff `https://raw.githubusercontent.com/router-for-me/models/main/codex_client_models.json` with `scripts/diff-codex-catalog.mjs` in the plugin repo.
2. Probe each changed id with `scripts/probe-codex.mjs --confirm`. The script reads the credential field `access`, sends `stream: true`, and does not send `max_output_tokens`.
3. A 400 whose body says the model is missing stays `status: "unavailable"`. HTTP 200 with the same response model is `available`. HTTP 200 served by another model sets `servedModel`. 401 does not mark models unavailable.
4. Open a pull request with the JSON only. Do not include credentials.

`/backend-api/models` is not a catalog. An authenticated call can return an empty list.
