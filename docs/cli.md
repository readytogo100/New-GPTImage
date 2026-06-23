# Command-line reference

This is the complete reference of New-GPTImage's command-line parameters, grouped by
purpose, with examples. You can also get a built-in summary at any time:

```powershell
./New-GPTImage.ps1 -Help            # parameter list with descriptions
./New-GPTImage.ps1 -Help Example    # same, plus 3 usage examples per parameter
./New-GPTImage.ps1 -Version         # print the version and exit
```

The general invocation pattern is:

```powershell
./New-GPTImage.ps1 -Prompt "<text>" [options...]
```

A note on precedence: for any parameter that has a configuration default (size, quality,
format, model, etc.), the command-line value wins; if omitted, the configuration value is
used; if that too is absent, a built-in fallback applies.

## Prompt input

| Parameter | Description |
|-----------|-------------|
| `-Prompt <text>` | The text describing what to generate. Minimum length 5 characters. |
| `-PromptFile <path>` | Read the prompt from a `.txt`/`.md` file. Overrides `-Prompt`. |

If a "standard file" exists (by default `New-GPTImage.md`), its content is appended to every
prompt — handy for default style directives. See the [configuration reference](config.md).

```powershell
./New-GPTImage.ps1 -Prompt "A red fox in a snowy forest, photorealistic"
./New-GPTImage.ps1 -PromptFile ".\prompts\landscape.md"
```

## Model selection

| Parameter | Description |
|-----------|-------------|
| `-Model <name>` | Logical model name; must exist under `models` in the config. Defaults to `defaultModel`. |
| `-Auto <Cost\|Quality\|Time>` | Choose the model automatically by strategy. Overrides `-Model`. |
| `-Force` | With `-Auto`, refresh the supporting data, bypassing the weekly cache. |

`-Auto Cost` picks the cheapest model; `-Auto Quality` the highest quality rating; `-Auto
Time` the fastest. Only models that are active and fully configured are considered, filtered
by the features your request needs.

```powershell
./New-GPTImage.ps1 -Prompt "..." -Model FLUX.2-pro
./New-GPTImage.ps1 -Prompt "..." -Auto Cost
./New-GPTImage.ps1 -Prompt "..." -Auto Quality -Force
```

## Generation parameters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `-NImages <n>` | 1–100 (typical max 10) | How many images to generate in one request. |
| `-Size <WxH>` | e.g. `1024x1024` | Output size. Auto-downscaled if it exceeds the model's limits. |
| `-Quality <q>` | `low`, `medium`, `high` | Generation quality; higher means slower. |
| `-Format <f>` | `png`, `jpg`, `webp` | Local output format (the server always returns PNG). |
| `-Moderation <m>` | `auto`, `low` | Content moderation. `low` is gentler on realistic faces. |
| `-InputFidelity <f>` | `low`, `high` | Adherence to input images. Only for models that support it. |
| `-Background <b>` | `auto`, `opaque`, `transparent` | Background type. `transparent` requires PNG. Only for supporting models. |

```powershell
./New-GPTImage.ps1 -Prompt "..." -Size "1536x1024" -Quality high -Format png
./New-GPTImage.ps1 -Prompt "..." -NImages 4 -Background transparent
```

If you request a size larger than the model allows, the script reduces it to the nearest
valid value (preserving the aspect ratio) with a warning, rather than failing — it only
fails when no valid reduction exists.

## Image editing

| Parameter | Description |
|-----------|-------------|
| `-InputImages <p1,p2,...>` | Reference images for edit mode. `.png`/`.jpg`/`.jpeg`, max 20 MB each; auto-validated, converted to RGB and resized. |
| `-Mask <path>` | A PNG mask with an alpha channel for inpainting. Transparent areas are the regions to change. |

Editing requires a model with `supportsEdit=true`; inpainting additionally requires
`supportsInpainting=true`.

```powershell
# Edit using a reference image
./New-GPTImage.ps1 -Prompt "make it look like winter" -InputImages ".\photo.png" -Model gpt-image-2

# Inpainting with a mask
./New-GPTImage.ps1 -Prompt "replace the sky with aurora" -InputImages ".\scene.png" -Mask ".\sky-mask.png" -Model gpt-image-2
```

## Output

| Parameter | Values | Description |
|-----------|--------|-------------|
| `-OutputFolder <path>` | | Where to save outputs. Defaults to `defaultOutputFolder`. |
| `-OutputMode <m>` | `file`, `base64` | Save an image file, or a `.txt` with the base64 payload. |
| `-Metadata <v>` | `True`/`False` (lenient) | Write a `.json` sidecar with all parameters needed to regenerate the image. |

```powershell
./New-GPTImage.ps1 -Prompt "..." -OutputFolder "D:\images" -Metadata True
```

The `.json` metadata is also what enables model attribution in the gallery ratings, so use
`-Metadata True` if you plan to rate and compare models.

## Batch processing

| Parameter | Description |
|-----------|-------------|
| `-Batch <path>` | A `.json` file with an array of requests. |
| `-Threads <n>` | Parallelism, 1–5. Default 0 = use `batchThreads` from config (or 1). |

Each batch item may specify `prompt`, `size`, `quality`, `n`, `format`, `inputImages`,
`mask` and `id` (auto-assigned if absent). See the [interactive interface](tui.md) and the
example batch template shipped with the project.

```powershell
./New-GPTImage.ps1 -Batch ".\my-batch.json" -Threads 3
```

## Model benchmarking

| Parameter | Description |
|-----------|-------------|
| `-Bench <m1,m2,...>` | Run the same prompt(s) on several models to compare time/cost/tokens/quality. |
| `-BenchExport <path>` | Export the benchmark results to a CSV (one row per prompt × model). |

When `-Bench` is set, `-Model` is ignored. A model that fails does not block the others. For
the full feature see the [benchmark guide](bench.md).

```powershell
./New-GPTImage.ps1 -Prompt "..." -Bench gpt-image-2,MAI-Image-2.5,FLUX.2-pro -BenchExport ".\bench.csv"
```

## Text-model features

These use the text/multimodal model configured under `textModel`. See the
[text model guide](text-model.md).

| Parameter | Description |
|-----------|-------------|
| `-OptimizePrompt` | Rewrite/enrich the prompt via the text model before generating. |
| `-NegativePrompt` | Generate a negative prompt and inject it into the positive prompt. |
| `-Storyboard` | Generate a narrative sequence of coherent images from the prompt. |
| `-SequenceImages <n>` | Number of storyboard scenes, 2–20 (default 5). |
| `-DescribeImage <path>` | Analyze an image and produce a descriptive prompt (image-to-prompt). |

```powershell
./New-GPTImage.ps1 -Prompt "a quiet harbor at dawn" -OptimizePrompt -Model gpt-image-2
./New-GPTImage.ps1 -Prompt "a hero's journey through a magical forest" -Storyboard -SequenceImages 8
./New-GPTImage.ps1 -DescribeImage ".\reference.png"
```

## Analytics

| Parameter | Description |
|-----------|-------------|
| `-Analytics [EUR\|USD\|Token]` | Print 10 textual cost/usage charts. With no value, uses `defaultCurrency`. |
| `-AnalyticsFile <path>` | Save the charts to a text file instead of the screen. |
| `-Overwrite` / `-Append` | Write policy when `-AnalyticsFile` already exists (mutually exclusive). |
| `-ExportAll <path>` | Export all cost records to a CSV for BI/dashboards, then exit. |

See the [analytics guide](analytics.md).

```powershell
./New-GPTImage.ps1 -Analytics USD
./New-GPTImage.ps1 -Analytics -AnalyticsFile ".\report.txt" -Overwrite
./New-GPTImage.ps1 -Analytics -ExportAll ".\all-costs.csv"
```

## Gallery

| Parameter | Description |
|-----------|-------------|
| `-Gallery` | Generate `gallery.html` in the images folder, then exit. |

See the [gallery guide](gallery.md).

## Cost control

| Parameter | Description |
|-----------|-------------|
| `-Budget <amount>` | Block execution if the conservative cost estimate (+20%) exceeds this amount (in `defaultCurrency`). |

The comma decimal separator is handled, so `1,5` and `1.5` both work. See the
[cost management guide](costs.md).

```powershell
./New-GPTImage.ps1 -Prompt "..." -NImages 4 -Budget 0,50
```

## Language and verbosity

| Parameter | Values | Description |
|-----------|--------|-------------|
| `-Lang <code>` | `it en fr es pt de ru zh ja hi ar` | UI language for this run. Defaults to `language` from config. |
| `-Verbosity <v>` | `verbose`, `normal`, `silent`, `ultrasilent` | On-screen output level. Errors always go to stderr; the log file always contains everything. |

`verbose` shows everything; `normal` hides TIME and COST lines; `silent` shows only errors;
`ultrasilent` shows nothing on screen.

```powershell
./New-GPTImage.ps1 -Prompt "..." -Lang en -Verbosity normal
```

## Utility and management commands

These commands do their job and exit; most do not generate images. See the
[utilities guide](utilities.md) and [managing models](models.md).

| Parameter | Description |
|-----------|-------------|
| `-Setup` | Generate a default `New-GPTImage.conf` (does not overwrite an existing one). |
| `-ModelList` | List configured models and their status. |
| `-CheckModels` | Probe network reachability of active models' endpoints. |
| `-CheckEndpoint` | Test that each active endpoint actually responds, with the deploy region. |
| `-AddModel` / `-EditModel` / `-RemoveModel` | Interactive model-management wizards. |
| `-ModelPrice Force` | Force a model-price refresh via the Azure Retail API. |
| `-PricingDiscovery <keyword>` | Explore the Azure Retail Prices catalog for a keyword. |
| `-DumpPricingPages` | Download the official pricing pages of all configured models. |
| `-DecryptLog` | Print the decrypted log to stdout (for grep/debugging). |
| `-DryRun` | Validate parameters and show the final prompt without calling the API. |
| `-Help [Example]` | Show help, optionally with examples. |
| `-Version` | Print the version. |

## Self-test

| Parameter | Description |
|-----------|-------------|
| `-SelfTest` | Run the built-in test suite (relaunches itself as a subprocess for end-to-end tests). |
| `-Simulate` | Only the tests that do not consume tokens/images. |
| `-Real` | Only the tests that generate real images (token cost). |
| `-AllTest` | All tests (Simulate + Real). |

`-SelfTest` must be combined with exactly one of the three modes. It exits with code 0 (all
pass) or 1 (at least one failure).

```powershell
./New-GPTImage.ps1 -SelfTest -Simulate
```

## Interactive interface

| Parameter | Description |
|-----------|-------------|
| `-TUI` | Start the full-screen interactive interface. |

See the [TUI guide](tui.md).

## Infrastructure as Code

The `-IaC` family provisions Azure resources. See the [telemetry and infrastructure
guide](telemetry.md) for the complete reference. The principal parameters:

| Parameter | Description |
|-----------|-------------|
| `-IaC` | Generate (and optionally run) the Azure provisioning script. |
| `-IaCMode <m>` | `generate`, `dry-run` or `execute`. |
| `-IaCResourceGroup` / `-IaCRegion` / `-IaCFoundryName` / `-IaCProjectName` | Non-interactive parameters. |
| `-IaCDeployModel` and `-IaCModel*` | Optionally deploy an image model (name, version, format, SKU, capacity). |
| `-IaCTeardown` | Destroy the created resources (destructive, with confirmation). |
| `-IaCPurgeSoftDeleted` | Permanently remove a soft-deleted Foundry resource before recreating it. |
| `-IaCWizard` | Guided interactive collection of the IaC parameters. |

## Telemetry control

| Parameter | Description |
|-----------|-------------|
| `-NoTelemetry` | Disable telemetry sending for this run, regardless of the config value. |
