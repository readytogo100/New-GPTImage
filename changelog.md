# Changelog — New-GPTImage.ps1

Complete history of the script's changes, from inception to the current version.

The changelog is in reverse chronological order (the most recent version is at the top).
The project has three components versioned in parallel, each with its own
version variable in the script:

- **Script** — `$script:ScriptVersion` — current version: **5.92**
- **TUI** (full-screen interactive interface, `-TUI`) — `$script:TuiVersion` — current version: **0.92**
- **Gallery** (HTML gallery, `-Gallery`) — `$script:GalleryVersion` — current version: **1.71**

The script numbering follows the MAJOR.MINOR policy: a bugfix increments by
`+0.01`, a new feature moves up to the next decade. The TUI and the
Gallery, being standalone components matured in parallel, have their own
version line that grows independently as they are enriched.

> Note on sources: this file is reconstructed from the project's history (historical
> script headers, "Phase 1/2/3" development sessions, working notes, the
> previous changelogs `CHANGELOG.md` and `change.md`). It is accurate and detailed
> for the documented versions; for the origins (0.x–1.x) a summary is provided,
> as the fine detail of those very first releases is not available.
>
> Note on dates: the dates of the 2.x series are precise (from the historical changelog).
> The dates marked with `~` are approximate, reconstructed from the temporal references
> of the development sessions, and indicate the period in which that version
> matured rather than a single release day.

---

## 5.x series — Privacy, text model costs, downscaling, robustness

The 5.x series is the "production-grade" maturation phase: text model cost
tracking, privacy logging, automatic size downscaling, wizard completeness and a
long code quality review.
In this series the TUI reaches **0.92** and the Gallery **1.71**.

### 5.92 — 2026-06-21 — Complete review of the code comments

- Systematic review of all the script's comments (over 3,400 line comments
  and the 81 function documentation blocks) to align them with the current code
  before publication.
- Fixed the comment of the hardcoded MAI dimensional block: now described as a
  residual safety net that intervenes only when the preemptive downscaling
  cannot reduce the size, no longer as an unconditional block.
- Reordered the comments of the "operational parameters -> downscaling ->
  validation" flow so they reflect the real execution order.
- Fixed the analytics chart count (from "8 in a 2x4 layout" to "10 in a
  5x2 layout") everywhere it was cited.
- Made descriptive (no longer hardcoded numeric) the comment on the number of
  translation keys, which now refers to the `_total_keys` field of the `.lang` file.
- Added the FLUX schema (3 per-megapixel meters) to the comment of the pricing
  meter mapping, which cited only the 6-meter and 2-meter schemas.
- Added the `describe` operation to all the comments that list the operations
  with tracked text model cost (operation label, telemetry,
  record parser, analytics and workbook charts, privacy log, model
  availability check, text model prices).
- Enriched the documentation blocks of `Invoke-ImageGeneration` (auto-recovery
  and `OperationalParams` parameter), `Invoke-Benchmark` (budget and per-model
  downscaling), `Invoke-AddModelWizard` and `Invoke-EditModelWizard` (schema-aware
  pricing and optional sections); added the missing doc block to
  `Get-DefaultTuiConfig`.

### 5.91 — 2026-06-21 — Preemptive size downscaling

- Resolved the inconsistency whereby a size that was too large (e.g. `2048x2048` on a
  MAI model with maximum area `1048576`px) produced a blocking error instead
  of automatic downscaling: the preemptive validation exited before
  the post-API auto-recovery could intervene.
- New function `Invoke-PreemptiveSizeDownscale`: before validation, if the
  requested size exceeds the model's constraints, it reduces it to the closest
  valid value keeping the aspect ratio, with a `[WARN]`. Downscaling becomes
  the **primary** behavior; validation blocks only when reducing is
  impossible (e.g. aspect ratio not achievable within the model's constraints).
- Applied to all four generation flows: single, batch, benchmark
  (per-model, without altering the shared size) and TUI.
- Added a test to the self-test suite that verifies the reduction of invalid
  sizes and the preservation of those already valid.

### 5.90 — 2026-06-20 — Schema-aware model management wizards

- The `-AddModel` / `-EditModel` wizards now generate a correct pricing
  structure based on the model's schema, preventing OpenAI or FLUX models
  from computing a zero cost due to missing meters.
- New function `Get-SchemaPricingDefaults`: returns the pricing meters, the
  `providerTab`, the `serviceName`, the pricing page URL and the correct `meterRoles`
  for each of the three schemas (6 meters for OpenAI, 2 for MAI, 3 for FLUX).
- `-AddModel` uses these defaults and shows in the summary the created meters and the fields
  with localized values (no longer raw `True`/`False`).
- `-EditModel` extended with three optional sections (on demand, so as not to overload
  the wizard): feature flags (inpainting, input fidelity, background),
  prices of the existing meters, and numeric operational parameters.

### 5.80–5.84 — 2026-06-20 — Automatic downscaling (auto-recovery) and robustness

- **5.80** — Post-API automatic downscaling: new function `Get-DownscaledSize`
  that reduces the size to the closest valid value for the two families of constraints
  (closed list of allowed dimensions, or geometric constraints of side/area/
  divisibility/aspect ratio). If the API rejects a size, the script retries
  automatically with the reduced dimension without consuming the retry budget.
- **5.81** — Modernized `-Setup` template: the default configuration now
  generates all the real models and the entire TUI section with the nine color themes,
  via the new function `Get-DefaultTuiConfig` used as the single source both by
  `-Setup` and by the TUI auto-initialization.
- **5.82–5.83** — Localized budget parsing: correct handling of the comma
  decimal separator (e.g. `-Budget 1,5`), which PowerShell otherwise interprets as
  an array, via recomposition and culture-invariant parsing.
- **5.84** — Budget guardrail added to the benchmark mode too: it estimates the
  total cost of all the prompt x model combinations (summing per model,
  since the prices differ) and exits before generating if the conservative estimate
  (+20%) exceeds the budget.

### 5.85 — 2026-06-20 — Complete code audit

- Complete review on request: verified the false positives on the "unused"
  variables (actually used via splatting or interpolation), confirmed the
  presence of all the custom functions, fixed a typo in a comment.
- Removed 10 orphan translation keys no longer referenced.
- Confirmed safe: API keys all encrypted, no `Invoke-Expression`,
  anti-symlink protection in the cleanup, `HttpClient` always disposed.

### 5.70–5.71 — ~2026-06-19 — Text model cost tracking

- The operations that use the text model (`-OptimizePrompt`, `-NegativePrompt`,
  `-Storyboard`, `-DescribeImage`) now return the real token counts from the
  API's `usage` field and track the cost with a dedicated `[COST]` line per
  operation.
- New functions `Write-TextCostLog` (verbose cost line to file + on-screen
  summary in the local currency) and `Send-TextCostTelemetry` (separate
  `TextModelCost` event, so as not to mix the text costs with the image ones).
- Two new analytics charts: text model cost per month and per function,
  bringing the total to 10 charts (5x2 layout).
- Function `Test-TextModelAvailable` to verify the availability of the text
  model before use (WARN and skip for the non-critical operations, ERROR and
  block for the storyboard and the describe).
- Two new price fields in the `textModel` section of the `.conf`
  (`inputUsdPerMillion`, `outputUsdPerMillion`), to be set manually from the
  pricing page of your model.

### Cross-cutting developments of the 5.x series

Matured during the 5.x series:

- **Privacy logging**: new log level `[PROMPT]` (file only, never on
  screen), `privacy` section in the `.conf` with the `imagePrompts` / `supportPrompts`
  flags, optional Base64 obfuscation and helper `Write-PromptLog` that also records the
  length in characters and a token estimate.
- **`-DescribeImage` command**: analysis of an image via the multimodal text
  model (image-to-prompt), with output in the script's language.
- **Auto-recovery of unsupported parameters**: if the API rejects an optional
  parameter (background, input fidelity, mask), the script retries immediately
  without that parameter, without consuming the retry budget.
- **Ordering of the Workbook bar charts**: resolved the unwanted reordering
  using string labels on the X axis plus a sortable key column.

---

## 3.x–4.x series — Version centralization, TUI, telemetry, storyboard, IaC

The 3.x series introduces version centralization and the interactive interface
(TUI) is born. The subsequent phases add telemetry, advanced prompt commands,
the storyboard, the encrypted log and infrastructure as code (IaC). At the
end of this phase the script reaches **5.43**, with TUI **0.92** and Gallery
**1.71**.

### 4.x–5.43 — ~2026-06-13 → 2026-06-17 — Telemetry, storyboard, advanced prompts, gallery ratings, IaC

- **Generation telemetry**: shared function `Send-GenerationTelemetry`
  used by the CLI / TUI / batch flows, with a `source` property (cli/tui/batch);
  `-NoTelemetry` parameter to exclude runs (e.g. self-test) from sending;
  status line at startup indicating whether telemetry is active.
- **`-OptimizePrompt` / `-NegativePrompt`**: rewriting and enrichment of the prompt
  via the text model (`textModel` section in the `.conf`). Since gpt-image
  and MAI do not have a native `negative_prompt` field, the negative is injected in
  a "soft" way into the positive prompt ("avoid: ...").
- **`-Storyboard` / `-SequenceImages`**: narrative decomposition of a text into N
  scenes (2–20) via the text model, with a "style bible" (palette,
  technique, mood, description of the recurring characters) repeated at the top of each
  scene for visual consistency; numbered output `story_01..NN`. `MaxTokensOverride`
  parameter with budget scaling by number of scenes (resolves the timeouts
  on long sequences).
- **`-DescribeImage`**: multimodal analysis of an image, with output in the script's
  language.
- **Encrypted log**: `-DecryptLog` parameter and log file encryption.
- **IaC (`-IaC`)**: wizard for creating the Azure infrastructure via Azure CLI
  (`az`). It generates a Bicep (AI Foundry, Application Insights / Log Analytics) and
  applies it via `az deployment`. The infrastructure is the primary goal; the
  model deployment is optional (subject to availability in the region and
  subscription quotas). Access to the models via API key, consistent with the existing
  setup. (Documented decision: Terraform was evaluated but discarded because
  it would have required external tools anyway.)
- **Azure Monitor Workbook**: expanded to 27 charts in 6 sections, with formatted
  time axes, dynamic chart count and idempotency
  (create-or-update) of the Workbook creation.
- **TUI (toward 0.92)**: new commands `/optimize`, `/negative`, `/storyboard`,
  `/sequenceimages`, `/reload` (refreshes the in-memory configuration without
  closing the TUI), copy with Ctrl+C in the prompt editor, scroll with
  Ctrl+Home / Ctrl+End and a "Functions" status bar that shows the active
  features.
- **Gallery (toward 1.71)**: zoom with the mouse wheel in the lightbox (from 0.25x to
  8x) with pan and zoom indicator; per-image ratings exported to
  `gallery_ratings.json` via the File System Access API; display of only the
  configured currency; gallery version shown in the regeneration log.
- **`-Auto Quality` with the gallery votes**: weighted average between Arena score and
  gallery votes when a model has at least 5 votes.

### 3.42–3.45 — ~2026-06-08 → 2026-06-12 — Quality, security, documentation alignment

- **3.45** — Rewrote the script header (removed the old name
  "ENTERPRISE ULTRA-PRO v2.8" and the inline changelog, moved to the
  changelog file); added a descriptive comment to all the functions that lacked
  one (101 functions, mostly TUI helpers).
- **3.44** — Aligned the command-line help (`-Help`) with the script's real
  parameters (documented `-Force`, `-TUI`, `-SelfTest` with the
  `-Simulate` / `-Real` / `-AllTest` modes); fixed a typo in an example.
- **3.43** — Security and quality audit: warning (WARN) when an API key is
  saved obfuscated (Base64) and not encrypted; replaced the `+=` accumulators on
  arrays with `List[object]` in the loops that can grow; more robust extraction
  of the API error message.
- **3.42** — Dedicated classification of the Azure safety system rejections
  (HTTP 400 "rejected by the safety system", `content_filter`, etc.), no longer
  generically labeled as "unsupported parameter".

### 3.4x — ~2026-06-07 — TUI: real generation (Increment 3)

- The TUI moves from the simulated adapter to real in-process generation: resolution
  of the generation context from the config, progress callback hooked to the
  generation engine, budget guardrail, logging of times and costs directly
  from the interactive session.

### 3.x — ~2026-05-23 → 2026-06-06 — TUI interactive interface (TuiVersion 0.10 → 0.62)

Introduction and maturation of the full-screen interactive interface (`-TUI`),
grown in parallel with the script via `$script:TuiVersion`:

- Full-screen layout: header with current model and parameters and a clock,
  message body with word-wrap and scroll, two status bars, command line.
- Full-screen prompt editor with word-wrap, multi-line cursor
  movement, save to file.
- Command line with history and autocompletion of commands and paths.
- Commands to set at runtime all the generation parameters (model,
  size, quality, format, moderation, background, mask, reference
  images, output, etc.), color themes, `/outputmode <file|base64>`,
  `/budget <amount|off>`.
- "Live" clock via non-blocking input polling.
- Terminal tab title set dynamically.
- Rendering via Synchronized Output (DEC 2026 sequence) to eliminate the
  flicker, with targeted update of only the progress bar during
  generation.
- Progressive refinement of the rendering and the layout (around TuiVersion 0.40).

### 3.0 — ~2026-05-23 — Version centralization and foundations for the TUI

- Script version centralized in a numeric variable
  (`$script:ScriptVersion`) with MAJOR.MINOR policy, instead of the string "v2.8"
  scattered through the code.
- Code reorganization and preparation of the foundations of the interactive
  interface.

---

## 2.x series — Multi-model, multilingual, pricing, benchmark

The 2.x series transforms the script from a single-model generator to a
multi-model, multilingual platform, with cost tracking, automatic pricing and benchmark.

### 2.8 — 2026-05-21 / 2026-05-22 — MAI models, per-model auth, wizards, robustness

- New MAI-Image-2.5 and 2.5-Flash models (text-to-image and image-to-image edit).
- `authMode` configurable per model (`api-key` | `bearer`): models with the
  same schema but different authentication, without touching the code.
- Interactive wizards `-AddModel` / `-EditModel` / `-RemoveModel` to manage the
  `.conf` models without editing it by hand (encrypted key, per-schema defaults).
- Extended dimensional constraints: side divisibility and aspect ratio range
  (gpt-image-2 supports arbitrary dimensions, not just the three standard ones).
- Resizing of reference images to the model's maximum area
  (avoids timeouts on the MAI models).
- Benchmark mode `-Bench` / `-BenchExport`: comparison between models with
  times / costs / tokens / image paths, CSV export with culture-aware decimal
  separator.
- Code cleanup: removal of dead code, neutralization of about 290
  historical comments, reordering of all the functions before the main flow.
- Robustness and security: image validation via magic-bytes
  (`Test-ImageFileSignature`), JSON parsing with `-ErrorAction Stop`,
  `ValidateRange(1,100)` on `-NImages`, validation of the batch structure,
  prompt length limit (120,000 characters), disk space warning,
  real write-probe on the output folder, file name sanitization,
  endpoint URL validation.
- Bugfix: `$pid` (PowerShell read-only automatic variable) renamed to
  `$promptId`; guard against NaN when the time estimate was zero; CSV export made
  culture-aware.

#### Cross-cutting developments of the 2.x series

Matured during the 2.x series and merged into the versions above (they correspond to
the "Round A–D" of the first development sessions):

- **Complete internationalization** via the `T()` function and the
  `New-GPTImage.lang` file (11 languages: it, en, fr, es, pt, de, ru, zh, ja, hi, ar). The
  translation started from 5 languages (Round C) and then extended to 11. Early
  loading of the language pack right after the definition of `T()`,
  to resolve the utility commands (`-Version`, `-Help`, `-ModelList`,
  `-CheckModels`, `-Analytics`) that ran before the loading and showed the
  raw keys. The log tags `[INFO]`, `[WARN]`, `[ERROR]`, `[TIME]`, `[COST]`
  stay in English (standard levels, not translatable).
- **Four-phase `Update-ModelPricing` price update subsystem**
  (Round D): (1) exact lookup on the Retail API with `serviceName` +
  `productHints[0]` + `armRegionName`; (2) relaxed lookup iterating all the
  `productHints` with and without region; (3) HTML scraping of the official pricing page
  (two layouts: "labeled" Azure OpenAI and "columnar" Microsoft MAI); (4) discovery with
  saving of the candidates to `New-GPTImage.pricing-discovery.json` and fallback to
  the local prices of the `.conf`. Resolved the wrong assumption of the fixed
  x1000 multiplier (now it normalizes the `unitOfMeasure`), added pagination via
  `NextPageLink` (up to 5 pages) and a dedicated timeout (raised from 5 to 15 seconds).
- **New `.conf` structure for pricing**: `pricingMapping` field per model
  (`officialName`, `providerTab`, `serviceName`, `armRegionName`, `productHints`,
  `meterRoles`, `pricingPageUrl`) and `pricingUpdate` section at root level
  (`enabled`, `timeoutSec`, `writeDiscoveryFile`, `frequencyDays`). Automatic
  migration of the `.conf` with the old structure at the first launch. Deprecated the old
  `azureMeterFilter` field (replaced by `pricingMapping`, with WARN).
- **`-PricingDiscovery <keyword>` command** to explore the Retail
  API catalog; handling of region aliases (`Get-ArmRegionAlias`) between ARM names and
  pricing page names.
- **HTML Gallery** (Round A → 2.x series, toward GalleryVersion ~1.x): generated with
  `-Gallery`; light/dark theme switch; sorting by creation/modification date/
  name/size; extended image info (px dimensions, aspect ratio, DPI,
  color depth, transparency); localization in 11 languages; search, filters,
  statistics and visual comparison between models (benchmark); model labels on two
  lines (`shortName`).
- **Self-test suite** (`Test-GPTImage.ps1`, later integrated as `-SelfTest`): test
  suite with "hardened" cleanup of the temporary folder (5 levels of safety
  checks before deleting, unique marker `Test-GPTImage-temp-`,
  anti-symlink protection).

### 2.7 — 2026-05-15 → 2026-05-20 — Setup, costs, parallel batch, per-model pricing

- **2.7.0** — `-Setup` command (generates the default `.conf` if absent);
  `-Metadata $true/$false` (`.json` file next to each image); parameters
  `-Moderation`, `-InputFidelity`, `-Background`, `-Mask`; `-Batch <path>` with
  sequential or parallel execution (`-Threads`); cost tracking (`.cost` file
  with daily aggregate); time estimation via kNN on historical data (`.stat`
  file, with EMA + segment fallback).
- **2.7.3** — Partial cross-platform (Linux/macOS): system info, log and config
  work; image editing remains Windows-only. `Get-SystemInfo` function with
  OS/CPU/RAM/disk details; cleaned-up on-screen output.
- **2.7.4** — Critical bugfix: `moderation` is now also sent to the
  `/images/edits` endpoint (previously only in generation), a possible cause of the
  "face similarity" problem with `moderation=auto`; `output_format` always `png` on the
  server side with jpg/webp conversion only local; input image limit from 50 to 20 MB
  (real Foundry limit); detailed comments for each parameter; fix of two memory
  leaks in the conversion/compression of images.
- **2.7.5** — `-CheckModels` (verifies network reachability of the endpoints);
  `-Analytics EUR|USD|Token` with 8 textual charts in a 2x4 layout; `-Gallery` to
  generate the HTML gallery; internet connectivity test at launch; daily
  update of the USD/EUR rate via frankfurter.app (ECB data); `active` field per
  model; extended cross-platform (OS detection in the header, absolute paths,
  cross-OS `standardFile`).

### 2.6 — 2026-05-10 — Multi-model

- `.conf` configuration restructured with the `models` section for multi-model
  support; per-model pricing in nested JSON; rationalization of the
  nomenclature.

### 2.5 — ~2026-05-05 — Time estimation

- Generation time estimation engine based on kNN on historical data.

### 2.4 — ~2026-05-03 — Logging

- Enriched logging.

### 2.3 — 2026-05-01 — Stability

- First public version with basic generation, configuration (initially
  monolithic) and textual application log; bug-fixes and hardening.

### 2.2 — ~2026-04-28 — Input images (resizing)

- Automatic resizing of input images that are too large; parameter
  `defaultMaxImageDimension` in the config; warning if the converted PNG exceeds 20 MB.

### 2.1 — ~2026-04-26 — Input validation

- Automatic validation and conversion of input images (RGB, size,
  format); detailed diagnostics for each image; automatic cleanup of the temporary
  files.

### 2.0 — ~2026-04-24 — Generation and editing (dual-mode)

- Dual-mode: generation (text-to-image via `/images/generations`, JSON)
  and editing (image-to-image via `/images/edits`, multipart/form-data); multiple
  input images sent via multipart; automatic endpoint selection based
  on the presence of input images.

---

## Origins (0.1 – 1.x)

First versions of the script: initial core for text-to-image image generation
via a single model on Azure Foundry (gpt-image-2). In this
phase the script was monolithic, with configuration in a single file (endpoint,
deployment, API key, default size/quality/format all at root level,
without the `models` section) and the basic logic of calling the API, saving
the output and handling errors. From this base the script then evolved toward
the dual-mode (generation and editing) of 2.0 and the multi-model and
multilingual architecture of the subsequent series. The precise detail of these very first
versions cannot be reconstructed from the available sources and is therefore reported in
summary form.

---

## Current state (reference)

- **Script** 5.92 · **TUI** 0.92 · **Gallery** 1.71
- **Configured models** (6): gpt-image-2, gpt-image-1.5 (schema `openai-images`);
  MAI-Image-2e, MAI-Image-2.5, MAI-Image-2.5-Flash (schema `mai-images`);
  FLUX.2-pro (schema `bfl-flux`). Plus a text model in the `textModel` section.
- **Internationalization**: 11 languages (it, en, fr, es, pt, de, ru, zh, ja, hi, ar).
- **Runtime**: PowerShell 7.0+ (sviluppo e test su Windows 11 ARM64, Snapdragon X).