# Cost management

New-GPTImage tracks what your generations cost, can estimate a cost before generating, and
can block a run that would exceed a budget you set. This guide explains how costs are
computed, recorded and controlled.

## How costs are computed

Each model has a `pricing` block in the configuration, with per-meter prices (see the
[configuration reference](config.md)). Depending on the model's API schema, the meters
represent input/output text tokens, input/output image tokens, or per-megapixel pricing
(for FLUX). After each generation the script reads the real token usage returned by the API
and computes the cost from those meters, recording it with a `[COST]` log line.

Prices can be refreshed automatically from the Azure Retail Prices API (and, as a fallback,
by scraping the official pricing page). This is governed by the `pricingUpdate` section. You
can force a refresh at any time:

```powershell
./New-GPTImage.ps1 -ModelPrice Force
```

If a model's price cannot be matched on the API, a discovery file
(`New-GPTImage.pricing-discovery.json`) is written to help you identify the correct catalog
names; see the [utilities guide](utilities.md) for `-PricingDiscovery` and
`-DumpPricingPages`.

## Currency

Costs are displayed in the currency set by `defaultCurrency` (`EUR` or `USD`). When the
currency is EUR, the script converts from the USD catalog prices using a USD→EUR rate that
is cached in the configuration and refreshed daily from a public source. The currency is
also auto-detected based on your system region. All cost surfaces — the on-screen summary,
the analytics charts and the gallery — use this same currency, so the figures stay
consistent.

## The cost files

Two local files hold the cost history:

- the **log file** records a `[COST]` line per generation;
- the **`.cost` file** holds a daily cost aggregate.

These are written for every real generation (they are not suppressed by `-NoTelemetry`,
which only affects Azure telemetry and the time-statistics file). They are the basis of the
[analytics](analytics.md) charts and CSV export.

## Estimating before generating

The cost estimate is shown as part of the normal flow. Be aware of its limits: the input
text tokens (from the prompt length) and input image tokens (from the references) are
estimable, but the **output** tokens — often the most expensive part for image models —
depend entirely on the model's response and cannot be predicted in advance. To stay on the
safe side, the script applies a conservative margin (+20%) above the raw estimate. The
estimate is a safety mechanism, not a guarantee: a particularly complex generation can cost
more.

## Budget guardrail

You can cap the spend for a request (or a whole batch or benchmark) with `-Budget`. If the
conservative estimate exceeds the budget, the run is blocked and the estimate is shown,
before any API call:

```powershell
./New-GPTImage.ps1 -Prompt "..." -NImages 4 -Budget 0,50
```

The value is expressed in `defaultCurrency`. The comma decimal separator is handled (PowerShell
would otherwise treat `0,50` as a list), so both `0,50` and `0.50` work.

In [benchmark mode](bench.md) the budget check is also applied: it sums the projected cost of
all prompt × model combinations (per model, because prices differ) and blocks the whole run
if the estimate is over budget.

## Text-model costs

The advanced prompt features that use the text model (`-OptimizePrompt`, `-NegativePrompt`,
`-Storyboard`, `-DescribeImage`) have their own cost, tracked **separately** from image
costs with a dedicated `[COST]` line per operation, and surfaced in their own analytics
charts. For this to produce non-zero figures, the `inputUsdPerMillion` and
`outputUsdPerMillion` fields under `textModel` must be filled in from your model's pricing
page. See the [text model guide](text-model.md).

## Reviewing costs

- On screen, the cost of each generation appears in `[COST]` lines (visible at `verbose`
  verbosity).
- For trends over time, use the [analytics](analytics.md) charts or export all records to
  CSV with `-Analytics -ExportAll`.
- For a live dashboard on Azure, enable telemetry and use the Workbook (see the
  [telemetry guide](telemetry.md)).
