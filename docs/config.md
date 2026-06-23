# Configuration reference

This is a complete reference of `New-GPTImage.conf`, the JSON configuration file that lives
next to the script. Generate a starting point with `./New-GPTImage.ps1 -Setup`, then edit
the values described below. For the model entries specifically, the guided wizards
described in [managing models](models.md) are usually easier than hand-editing.

The file is a single JSON object. The sections below follow its structure: top-level
defaults first, then the nested sections (`models`, `textModel`, `privacy`,
`pricingUpdate`, `autoUpdate`, `telemetry`, `tui`).

## Top-level defaults

These keys define the behavior used when a corresponding command-line parameter is not
supplied. The priority is always: command-line parameter, then configuration value, then a
built-in fallback.

| Key | Type | Meaning |
|-----|------|---------|
| `defaultModel` | string | Logical model name used when `-Model` is omitted. Must match a key under `models`. |
| `defaultSize` | string | Output size `WIDTHxHEIGHT` used when `-Size` is omitted (e.g. `1024x1024`). |
| `defaultQuality` | string | `low`, `medium` or `high`, used when `-Quality` is omitted. |
| `defaultOutputFolder` | string | Folder where images are saved when `-OutputFolder` is omitted. Absolute or relative. |
| `defaultOutputCompression` | int | JPG/WebP compression quality (0–100) for local conversion. |
| `defaultModeration` | string | `auto` or `low`. `low` is recommended for realistic portraits. |
| `defaultMaxImageDimension` | int | Maximum long side (px) to which input reference images are resized before sending. |
| `defaultFormat` | string | Output file format: `png`, `jpg` or `webp`. |
| `defaultBackground` | string | `auto`, `opaque` or `transparent` (only for models that support it). |
| `fileNameTemplate` | string | Template for output file names. Placeholders: `{timestamp}`, `{index}`, `{promptHash}`. |
| `maxRetries` | int | Number of retries for a failing API call. |
| `logEncryption` | bool | If `true`, the log file is encrypted; read it back with `-DecryptLog`. |
| `requestTimeoutMinutes` | int | HTTP timeout per image request. High-quality generations can take several minutes. |
| `maxStatsRecordsToRead` | int | How many recent `.stat` records to read at startup for time estimation. |
| `knnK` | int | Number of neighbors used by the kNN time estimator. |
| `knnMinRecords` | int | Minimum records needed to activate kNN (below this, it falls back to a segment/EMA estimate). |
| `segmentMinRecords` | int | Minimum records for the "homogeneous segment" time-estimate fallback. |
| `maxLogFileSizeMB` | int | Log rotation threshold in megabytes. |
| `logRetentionDays` | int | How many days of log history to keep. |
| `metadata` | bool | Default for `-Metadata`: whether to write a `.json` sidecar next to each image. |
| `batchThreads` | int | Default parallelism for `-Batch` when `-Threads` is not given (1 = sequential). |
| `usdToEurRate` | float | Cached USD→EUR exchange rate, refreshed daily from a public source. |
| `usdToEurLastUpdate` | string | Date of the last exchange-rate refresh (managed by the script). |
| `pricingLastUpdated` | string | Date of the last model-price refresh (managed by the script). |
| `defaultCurrency` | string | `EUR` or `USD`: the currency used for cost display and analytics. |
| `defaultCurrencyLastCheck` | string | Date of the last currency auto-detection (managed by the script). |
| `language` | string | Default UI language when `-Lang` is omitted (`it`, `en`, `fr`, `es`, `pt`, `de`, `ru`, `zh`, `ja`, `hi`, `ar`). |
| `standardFile` | string | Name of the optional "standard file" appended to every prompt (default `New-GPTImage.md`). |

The keys marked "managed by the script" (`usdToEurLastUpdate`, `pricingLastUpdated`,
`defaultCurrencyLastCheck`) are written automatically; you normally do not edit them.

## models

An object whose keys are the logical model names you reference with `-Model`. Each model is
itself an object with the fields below. The currently shipped configuration defines six
models across three API schemas (see [managing models](models.md) for the full list).

### Connection fields

| Field | Meaning |
|-------|---------|
| `active` | If `false`, the model is hidden from selection, `-Auto`, and the reachability checks. |
| `endpoint` | The generation endpoint URL. |
| `editEndpoint` | The edit/inpainting endpoint (only for `openai-images` models that support edit). |
| `failoverEndpoint` | Optional backup generation endpoint. |
| `failoverEditEndpoint` | Optional backup edit endpoint. |
| `apiKey` | The API key. Paste it in clear text once; the script encrypts it in place on first run. |
| `deployment` | The Azure deployment name of the model. |
| `shortName` | A compact label used in analytics charts and the gallery (e.g. `GPT\|2`). |
| `authMode` | `api-key` or `bearer`. Overrides the schema default when a model needs a different auth header. |
| `apiSchema` | `openai-images`, `mai-images` or `bfl-flux`. Determines the request format and auth. |
| `supportsEdit` | If `false`, the model rejects `-InputImages`/`-Mask` (text-to-image only). |

### Feature flags

| Field | Meaning |
|-------|---------|
| `supportsInpainting` | The model accepts a `-Mask` for localized editing. |
| `supportsInputFidelity` | The model accepts `-InputFidelity` (`low`/`high`). |
| `supportsBackground` | The model accepts `-Background` (`auto`/`opaque`/`transparent`). |

### Auto-selection ratings

| Field | Meaning |
|-------|---------|
| `qualityRating` | A quality score used by `-Auto Quality`. Can be refreshed from an external leaderboard. |
| `speedRating` | A speed score used by `-Auto Time`. |
| `qualityArenaAlias` | The model's name on the external quality leaderboard, used when refreshing `qualityRating`. |
| `qualityRatingLastUpdate` | Date of the last quality-rating refresh. |
| `speedRatingLastUpdate` | Date of the last speed-rating refresh. |

### pricing

Holds the per-meter prices used to compute costs.

| Field | Meaning |
|-------|---------|
| `imagePerUnitUsd` | Legacy flat per-image price (kept for back-compat). |
| `lastUpdated` | Date the prices were last refreshed. |
| `lastUpdatedSource` | How they were obtained (e.g. exact API match, relaxed match, HTML scrape, local). |
| `meters` | The per-meter USD prices per million units: `inputText`, `cachedInputText`, `inputImage`, `cachedInputImage`, `outputText`, `outputImage`. |

Not every schema uses all meters. The `openai-images` schema uses the full six-meter set;
`mai-images` uses a smaller set; `bfl-flux` uses per-megapixel pricing.

### pricingMapping

Tells the automatic price-update pipeline how to find this model in the Azure Retail Prices
API and on the official pricing page.

| Field | Meaning |
|-------|---------|
| `officialName` | The model's official commercial name. |
| `providerTab` | The provider grouping (e.g. `aoai` for Azure OpenAI). |
| `serviceName` | The Azure service name used in the Retail API query. |
| `armRegionName` | The ARM region used for the price lookup. |
| `productHints` | A list of product-name variants to try when matching the API catalog. |
| `meterRoles` | For each meter role, the list of meter-name fragments that identify it in the catalog. |
| `pricingPageUrl` | The official pricing page, used by the HTML-scrape fallback and by `-DumpPricingPages`. |

### operationalParams

The declared operational limits of the model, validated before each call (so an invalid
request fails fast with a clear message instead of a cryptic API error). For each limit
there is usually an `...Enforce` flag: when `true`, exceeding the limit is a blocking error;
when `false`, it is only a warning.

| Field | Meaning |
|-------|---------|
| `maxImages` / `maxImagesEnforce` | Maximum images per request. |
| `allowedSizes` / `allowedSizesEnforce` | A closed list of allowed sizes (empty = use the geometric rules below). |
| `minSide` / `maxSide` | Minimum / maximum side length in pixels. |
| `maxArea` | Maximum total area (width × height) in pixels (0 = no area limit). |
| `sizeRulesEnforce` | Whether the geometric size rules are enforced. |
| `maxReferenceImages` / `maxReferenceImagesEnforce` | Maximum reference images for edit. |
| `allowedQuality` / `allowedQualityEnforce` | The accepted quality values. |
| `allowedFormat` / `allowedFormatEnforce` | The accepted output formats. |
| `sizeDivisibleBy` | The side length must be divisible by this value. |
| `aspectRatioMin` / `aspectRatioMax` | The allowed range of width/height ratio. |

When a requested size exceeds these limits, the script first tries to **downscale** it to
the nearest valid value preserving the aspect ratio (with a warning), and only fails if no
valid reduction is possible.

## textModel

Configuration of the text/multimodal model used by the advanced prompt features
(`-OptimizePrompt`, `-NegativePrompt`, `-Storyboard`, `-DescribeImage`). See the
[text model guide](text-model.md) for usage.

| Field | Meaning |
|-------|---------|
| `endpoint` | The text model's endpoint (typically a `/responses` endpoint). |
| `apiKey` | The text model's API key (encrypted on first run, like the image keys). |
| `model` | The deployment/model name. |
| `authMode` | `api-key` or `bearer`. |
| `maxCompletionTokens` | Maximum tokens in the model's response. |
| `reasoningEffort` | Reasoning effort hint (e.g. `low`, `medium`, `high`) for models that support it. |
| `timeoutSec` | Request timeout in seconds (storyboards of many scenes can take a while). |
| `inputUsdPerMillion` | Price per million input tokens, for text-cost tracking. |
| `outputUsdPerMillion` | Price per million output tokens, for text-cost tracking. |

The two price fields must be filled in manually from your model's pricing page; until they
are set, text-cost tracking reports zero.

## privacy

Controls what ends up in the log file.

| Field | Meaning |
|-------|---------|
| `imagePrompts` | If `true`, image prompts are written to the log (obfuscated in Base64). |
| `supportPrompts` | If `true`, the prompts sent to the text model are logged (obfuscated). |

Both default to `false`, so prompts are not written to the log unless you opt in.

## pricingUpdate

Controls the automatic model-price refresh.

| Field | Meaning |
|-------|---------|
| `enabled` | Whether the daily price refresh runs. |
| `timeoutSec` | Timeout for the price-lookup HTTP calls. |
| `writeDiscoveryFile` | Whether to write `New-GPTImage.pricing-discovery.json` when a price cannot be matched. |
| `frequencyDays` | How often (in days) to refresh prices. |

## autoUpdate

Controls the refresh of the `-Auto Quality` and `-Auto Time` supporting data.

| Field | Meaning |
|-------|---------|
| `arenaUrl` | The external quality-leaderboard URL used to refresh `qualityRating`. |
| `arenaTimeoutSec` | Timeout for the leaderboard download. |
| `qualityRefreshDays` | How often to refresh the quality ratings. |
| `timeRefreshDays` | How often to recompute the time medians from the log. |
| `timeMinRecords` | Minimum records needed to compute a time median. |
| `timeMaxRecords` | Maximum records used for the time median. |

## telemetry

Controls sending generation metrics to Azure Application Insights. See the
[telemetry guide](telemetry.md).

| Field | Meaning |
|-------|---------|
| `azureMonitor` | Whether telemetry is sent. Can be disabled per run with `-NoTelemetry`. |
| `connectionString` | The Application Insights connection string (sensitive — treat like a credential). |

## tui

Controls the interactive interface. See the [TUI guide](tui.md).

| Field | Meaning |
|-------|---------|
| `scrollbackLines` | How many lines of message history the TUI keeps. |
| `commandHistorySize` | How many commands the command-line history keeps. |
| `showHints` | Whether to show inline hints on the command line. |
| `clockFormat` | The .NET date/time format string for the header clock. |
| `editorConfirmKey` | The key that confirms text in the full-screen prompt editor (e.g. `F2`). |
| `progressBarWidth` | Width, in characters, of the generation progress bar. |
| `theme` | The active color theme name. |
| `themes` | An object of theme definitions. |

The shipped configuration includes nine themes: `midnight`, `forest`, `amber`, `arctic`,
`paper`, `matrix`, `dracula`, `ocean` and `contrast`. Each theme defines colors for the
header, body, separators, the two status bars, the command line, the editor and the
progress bar. You can switch theme at runtime with `/theme <name>`.

## API key encryption

You never need to encrypt keys yourself. Paste them in clear text into `apiKey` (for both
image models and the text model); on the next run the script encrypts each key in place and
rewrites the file. On Windows this uses DPAPI (the encrypted value is tied to your user
account); on other systems it uses AES with a per-user master key. Because of this, a
configuration file copied to another machine or user will not be able to decrypt the keys —
re-enter them there.
