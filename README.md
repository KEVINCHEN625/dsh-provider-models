# dsh-provider-models

Public model catalogs for [dsh-provider-manager](https://github.com/KEVINCHEN625/dsh-provider-manager).
These files are data. They do not contain API keys, access tokens, or refresh tokens.

The plugin ships a snapshot of each file and refreshes it from this repository
when an OAuth details page opens or a sign-in succeeds. The default age is 24
hours (`oauthCatalogTtlMs`). A failed download keeps the snapshot that shipped
with the plugin.

## Layout

`<providerId>.json`

```json
{
  "providerId": "openai-codex",
  "models": [
    {
      "id": "gpt-5.3-codex",
      "name": "GPT-5.3 Codex",
      "contextWindow": 400000,
      "maxTokens": 128000,
      "efforts": ["low", "medium", "high", "xhigh"],
      "input": ["text", "image"]
    }
  ]
}
```

`efforts` uses `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, and `max`.
`input` is `text` and, when the provider accepts images, `image`.

## Adding a provider

1. Confirm the model answers through a signed-in dsh channel. A bare HTTP call
   is not enough.
2. Copy context, output, and effort from the provider's own docs. Leave a model
   out when those numbers are unknown.
3. Open a pull request that adds `<providerId>.json` and nothing else. Do not
   include credentials.
