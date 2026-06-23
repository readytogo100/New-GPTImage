# System utilities

Beyond generating images, New-GPTImage includes a set of utility commands for diagnosing the
configuration, inspecting models, managing prices, and debugging. Most of these commands do
their job and exit, and most make no image-generating API calls.

## Diagnostics and configuration checks

| Command | What it does |
|---------|--------------|
| `-Setup` | Generate a default `New-GPTImage.conf`. Does not overwrite an existing file. |
| `-ModelList` | Print the configured models with their status: active/inactive, endpoint present, key present. Active and fully-working models are highlighted in green. |
| `-CheckModels` | Probe the network reachability of each active model's endpoint. |
| `-CheckEndpoint` | Test that each active endpoint actually responds (network + HTTP), reporting the deploy region and whether it is available or unreachable. |
| `-DryRun` | Validate all parameters and show the final prompt, without calling the API. Ideal for prompt debugging. |

```powershell
./New-GPTImage.ps1 -ModelList
./New-GPTImage.ps1 -CheckModels
./New-GPTImage.ps1 -CheckEndpoint
./New-GPTImage.ps1 -Prompt "test" -DryRun
```

`-CheckModels` is the lighter check (network reachability only); `-CheckEndpoint` goes
further and confirms each endpoint actually answers. Use them after editing the
configuration to catch typos in endpoints or missing keys before spending tokens.

## Pricing utilities

| Command | What it does |
|---------|--------------|
| `-ModelPrice Force` | Force a model-price refresh via the Azure Retail Prices API, bypassing the daily check. |
| `-PricingDiscovery <keyword>` | Query the Retail Prices catalog for a keyword, to find the real service/product/meter names of a model. Shows results on screen and writes a JSON file. |
| `-DumpPricingPages` | Download the official Azure pricing pages of all configured models (deduplicated) into a subfolder next to the `.conf`, using the same HTTP headers as the internal scraper. |

```powershell
./New-GPTImage.ps1 -ModelPrice Force
./New-GPTImage.ps1 -PricingDiscovery "GPT-Image"
./New-GPTImage.ps1 -PricingDiscovery "MAI"
./New-GPTImage.ps1 -DumpPricingPages
```

These are the tools to reach for when a model's automatic price lookup fails: `-PricingDiscovery`
helps you identify how Microsoft cataloged the model (so you can fix the `productHints` /
`meterRoles` in `pricingMapping`), and `-DumpPricingPages` lets you inspect exactly the HTML
the scraper sees. See the [cost management guide](costs.md) and the
[configuration reference](config.md).

Note on `-PricingDiscovery`: pass at least an empty string or a keyword
(`-PricingDiscovery ""` or `-PricingDiscovery "GPT-Image"`). Passing the bare flag with no
following token triggers a PowerShell binder error for typed parameters; an empty string
shows the usage.

## Logging utilities

| Command | What it does |
|---------|--------------|
| `-DecryptLog` | Print the decrypted log to stdout, for grep and debugging (when `logEncryption` is on). |

```powershell
./New-GPTImage.ps1 -DecryptLog
./New-GPTImage.ps1 -DecryptLog | Select-String "ERROR"
```

The log file is rotated according to `maxLogFileSizeMB` and `logRetentionDays` (see the
[configuration reference](config.md)). When `logEncryption` is enabled, the file on disk is
encrypted and `-DecryptLog` is how you read it back.

## Self-test

| Command | What it does |
|---------|--------------|
| `-SelfTest -Simulate` | Run the tests that do not consume tokens (validations, local commands, dry runs). |
| `-SelfTest -Real` | Run the tests that generate real images (token cost). |
| `-SelfTest -AllTest` | Run all tests. |

```powershell
./New-GPTImage.ps1 -SelfTest -Simulate
```

The self-test relaunches the script as a subprocess for end-to-end checks and exits with
code 0 (all pass) or 1 (at least one failure). The simulated mode is free and a good
post-installation sanity check; the real mode verifies the full generation path end to end.

## Companion scripts

The project ships two small helper scripts alongside the main one:

- **`Download-PricingPages.ps1`** — a standalone version of the pricing-page download, which
  reads the `.conf`, deduplicates the pricing pages of all configured models and saves the
  HTML locally with readable names, for offline inspection of the published prices.
- **`Diag-Timestamp.ps1`** — a small diagnostic that inspects a file's timestamps and the
  system time zone, useful when the gallery's "created on" time appears shifted.

Both are run directly with PowerShell, for example:

```powershell
./Download-PricingPages.ps1
./Diag-Timestamp.ps1 -Path ".\some-image.png"
```

## System information at startup

Every run logs a system-information header (operating system, CPU, RAM, system disk) and the
paths in use (script folder, configuration file, language file, standard file). This is
useful context when sharing a log for support, and it confirms the script is reading the
files you expect.
