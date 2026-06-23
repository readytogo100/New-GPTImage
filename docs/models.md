# Managing models

New-GPTImage is multi-model: you can configure several image-generation models and switch
between them per request, or let the script choose automatically. This guide explains the
model concept, the three API schemas, and how to add, edit and remove models — both through
the guided wizards (recommended) and by hand.

## What a model is

A "model" is a named entry under the `models` section of `New-GPTImage.conf`. Its key is the
logical name you pass to `-Model` (for example `gpt-image-2`). The entry holds everything
the script needs to talk to that model: the endpoint, the API key, the deployment name, the
feature flags, the pricing, and the operational limits. For the full list of fields see the
[configuration reference](config.md).

The models shipped in the default configuration are:

| Logical name | API schema | Edit support |
|--------------|-----------|--------------|
| `gpt-image-2` | `openai-images` | Yes |
| `gpt-image-1.5` | `openai-images` | Yes |
| `MAI-Image-2e` | `mai-images` | No |
| `MAI-Image-2.5` | `mai-images` | Yes |
| `MAI-Image-2.5-Flash` | `mai-images` | Yes |
| `FLUX.2-pro` | `bfl-flux` | Yes |

## The three API schemas

The `apiSchema` field determines how requests are built and authenticated. Choosing the
right schema is the single most important decision when adding a model.

**`openai-images`** — the Azure OpenAI gpt-image family and recent compatible models. Sizes
are expressed as `WIDTHxHEIGHT`; the schema supports `quality`, `moderation` and
`output_format`; editing happens through a dedicated multipart `/edits` endpoint
(`editEndpoint`). This schema uses the full six-meter pricing set.

**`mai-images`** — the Microsoft MAI image family. Width and height are sent separately, the
request body is minimal, and it uses a smaller pricing set. Edit support varies by model: the
older MAI-Image-2e is generation-only, while the newer MAI-Image-2.5 and 2.5-Flash support
edit on the same endpoint (their `supportsEdit` flag is set accordingly).

**`bfl-flux`** — Black Forest Labs FLUX. Authentication uses a `Bearer` header, width and
height are separate, and editing happens on the **same** endpoint as generation, with the
reference images passed in the body as base64. FLUX uses per-megapixel pricing.

Each schema has a default authentication mode (`openai-images` and `mai-images` default to
`api-key`, `bfl-flux` to `bearer`). You can override it per model with `authMode` — useful
when a model uses one schema's request format but a different auth header (for example,
recent MAI models on Foundry that use the `openai-images` format but require `Bearer`).

## Adding a model (wizard)

The recommended way to add a model is the interactive wizard:

```powershell
./New-GPTImage.ps1 -AddModel
```

The wizard asks guided questions and then writes a complete, correct model entry. It:

- encrypts the API key (DPAPI on Windows, AES elsewhere) so the clear-text key never stays
  on disk;
- pre-fills the `operationalParams` based on the chosen schema (via the schema operational
  defaults), so the limits are sensible from the start;
- pre-fills the `pricing` meters and the `meterRoles` structure based on the schema (via the
  schema pricing defaults), so the entry computes costs correctly right away instead of
  reporting zero because of missing meters.

This last point is the main reason to prefer the wizard over hand-editing: each schema needs
a specific pricing structure (six meters for `openai-images`, a smaller set for
`mai-images`, three per-megapixel meters for `bfl-flux`), and getting it wrong silently
yields a zero cost.

## Editing a model (wizard)

```powershell
./New-GPTImage.ps1 -EditModel
```

The edit wizard always lets you change the connection fields: `endpoint`, `deployment`,
`shortName`, `authMode`, `supportsEdit`/`editEndpoint`, region, and the API key. Beyond
those, it offers three optional sections that you enter only on request, so the common case
stays short:

- **Feature flags** — `supportsInpainting`, `supportsInputFidelity`, `supportsBackground`.
- **Meter prices** — the existing pricing meters, if you want to set them by hand.
- **Operational parameters** — the numeric limits in `operationalParams`.

## Removing a model (wizard)

```powershell
./New-GPTImage.ps1 -RemoveModel
```

The remove wizard asks for confirmation before deleting. If the model you are removing is
the current `defaultModel`, it warns you, so you do not end up with a configuration that
points its default at a model that no longer exists.

## Inspecting models

Two commands help you see the state of your models without spending tokens:

```powershell
# List the configured models with status: active/inactive, endpoint present, key present.
# Active and fully-working models are highlighted in green.
./New-GPTImage.ps1 -ModelList

# Probe the network reachability of each active model's endpoint.
./New-GPTImage.ps1 -CheckModels

# Test that each active endpoint actually responds (network + HTTP), with the deploy region.
./New-GPTImage.ps1 -CheckEndpoint
```

## Editing models by hand

You can edit the `models` section of `New-GPTImage.conf` directly, but keep these points in
mind:

- Set `apiSchema` correctly first; it dictates the rest.
- Provide a complete `pricing.meters` structure appropriate to the schema, or costs will be
  zero.
- Paste the `apiKey` in clear text; the script encrypts it on the next run.
- The `operationalParams` limits are validated before each call; setting them too loosely
  lets bad requests reach the API, setting them too strictly can block valid sizes (though
  the script will try to downscale first).
- Set `active` to `false` to keep a model in the file but exclude it from selection and
  checks.

## Automatic model selection

Instead of naming a model, you can let the script pick one with `-Auto`:

- `-Auto Cost` chooses the cheapest model.
- `-Auto Quality` chooses the highest `qualityRating` (optionally blended with gallery
  votes — see the [gallery guide](gallery.md)).
- `-Auto Time` chooses the fastest model by `speedRating`.

Auto-selection considers only models with `active=true` and a configured endpoint and key,
and it filters by the requested features (for example, a request with `-Mask` only considers
models with `supportsInpainting=true`). See the [command-line reference](cli.md) for
details and the `-Force` refresh behavior.
