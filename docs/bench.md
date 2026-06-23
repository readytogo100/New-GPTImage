# Benchmark mode

Benchmark mode runs the **same** prompt (or the same set of prompts) across several models
and compares the results — time, cost, tokens, and the produced image paths — so you can
decide which model best fits a given kind of work.

## Basic usage

Pass a comma-separated list of models to `-Bench`:

```powershell
./New-GPTImage.ps1 -Prompt "A lighthouse on a cliff at sunset, watercolor style" -Bench gpt-image-2,MAI-Image-2.5,FLUX.2-pro
```

Both forms of the list work:

```powershell
-Bench gpt-image-2,MAI-Image-2.5,FLUX.2-pro
-Bench "gpt-image-2","FLUX.2-pro"
```

When `-Bench` is set, the `-Model` parameter is ignored: the models under test are exactly
those you list in `-Bench`.

## What it measures

For each prompt × model combination, the benchmark records the generation **time**, the
**cost**, the **token** usage, and the path of the produced image. At the end of the run the
results are shown grouped by prompt, so you can compare the models side by side for each
prompt.

## Robustness

A model that fails does not abort the benchmark: the failure is reported and the remaining
models still run. This means a single misconfigured or unavailable model will not cost you
the whole run.

## Benchmarking multiple prompts

Combine `-Bench` with `-Batch` to run every prompt in the batch file against every model:

```powershell
./New-GPTImage.ps1 -Batch ".\prompts.json" -Bench gpt-image-2,FLUX.2-pro
```

The benchmark handles both prompt sources itself: a single `-Prompt`, a `-PromptFile`, or
the multiple prompts of a `-Batch`.

## Per-model size handling

Different models accept different size limits. In benchmark mode the per-model preemptive
downscaling is applied independently for each model, without altering the shared requested
size — so each model receives a size it can actually handle, and the comparison stays fair.

## Budget guardrail

Benchmark mode respects `-Budget`. Because each model can have different prices, the
estimate sums the projected cost of all prompt × model combinations per model. If the
conservative estimate (with a safety margin) exceeds your budget, the run is blocked before
any generation, and the estimate is shown — so a large benchmark cannot surprise you with an
unexpected bill.

```powershell
./New-GPTImage.ps1 -Prompt "..." -Bench gpt-image-2,FLUX.2-pro -Budget 1,00
```

## Exporting results

Use `-BenchExport` to write the results to a CSV — one row per prompt × model combination —
suitable for a spreadsheet or dashboard:

```powershell
./New-GPTImage.ps1 -Prompt "..." -Bench gpt-image-2,MAI-Image-2.5 -BenchExport ".\bench.csv"
```

The benchmark output images are named with their own template that includes the prompt id
and the model, so you can tell at a glance which file came from which model.

## Using benchmark results

The benchmark gives you the raw evidence (time/cost/tokens/output) to set the
`qualityRating` and `speedRating` of your models in the configuration, which in turn feed the
`-Auto Quality` and `-Auto Time` selection strategies. See [managing models](models.md) and
the [command-line reference](cli.md). Visual comparison of the resulting images across models
is also available in the [HTML gallery](gallery.md).
