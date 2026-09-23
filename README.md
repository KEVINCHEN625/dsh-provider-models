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

## Layout

`<providerId>.json`

```json
{
  "providerId": "openai-codex",
  "models": [
    {
      "id": "gpt-5.6-luna",
      "available": true,
      "verifiedAt": "2026-09-23",
      "contextWindow": 272000
    }
  ]
}
```

`available` is the channel measurement. `verifiedAt` is the day that
measurement was accepted. `servedModel` is set when a request for `id`
returned HTTP 200 from a different model. `contextWindow` is set only when the
channel window disagrees with models.dev; the channel figure wins.

## Updating Codex

1. Diff `https://raw.githubusercontent.com/router-for-me/models/main/codex_client_models.json` with `scripts/diff-codex-catalog.mjs` in the plugin repo.
2. Probe each changed id with `scripts/probe-codex.mjs --confirm`. The script reads the credential field `access`, sends `stream: true`, and does not send `max_output_tokens`.
3. HTTP 400 stays `available: false`. HTTP 200 with the same response model is `available: true`. HTTP 200 served by another model sets `servedModel`.
4. Open a pull request with the JSON only. Do not include credentials.

`/backend-api/models` is not a catalog. An authenticated call can return an empty list.
