# Setup

This guide covers everything needed to get New-GPTImage running: prerequisites, the
files involved, first-time configuration, the Azure infrastructure it talks to, and how
to verify that the installation works.

## Prerequisites

**PowerShell 7.0 or later is mandatory.** The script declares `#requires -Version 7.0`
and refuses to run on Windows PowerShell 5.1. The reason is the reliable UTF-8 handling
needed for the multilingual output: only PowerShell 7+ guarantees it. You can check your
version with:

```powershell
$PSVersionTable.PSVersion
```

If you do not have PowerShell 7, install it from the official Microsoft distribution
(`https://aka.ms/powershell`).

**Operating system.** Generation, logging, configuration, analytics, gallery, telemetry
and the interactive interface work on Windows, Linux and macOS. Image editing
(inpainting and reference-image edits that rely on `System.Drawing` for local format
conversion) is reliable only on Windows; on other systems the script falls back to PNG
where conversion is not available.

**Azure CLI (optional).** Only required if you intend to use the Infrastructure as Code
feature (`-IaC`) to provision Azure resources, or the price-discovery features that query
the Azure Retail Prices API. Day-to-day image generation does not need it.

**An Azure AI Foundry resource** with at least one image model deployed, and its endpoint
and API key. The script can also create this infrastructure for you — see
[Infrastructure](#azure-infrastructure) below.

## The files

The project revolves around a single self-contained script plus a few companion files
that live next to it in the same folder.

| File | Role | Published to a repo? |
|------|------|----------------------|
| `New-GPTImage.ps1` | The script itself. | Yes |
| `New-GPTImage.lang` | Translations for all 11 languages. Required for localized output. | Yes |
| `New-GPTImage.conf` | Your configuration: models, endpoints, encrypted API keys, defaults. | No — contains secrets |
| `New-GPTImage.conf.example` | A template of the configuration, with placeholders instead of secrets. | Yes |
| `New-GPTImage.md` | Optional "standard file": its text is appended to every prompt. | No — personal |
| `New-GPTImage.md.example` | Template for the standard file. | Yes |
| `New-GPTImage.log` | Application log (rotated). May contain prompts if privacy logging is on. | No |
| `New-GPTImage.stat` | Time-telemetry records, used by the kNN time estimator. | No |
| `New-GPTImage.cost` | Daily cost aggregate. | No |
| `New-GPTImage.pricing-discovery.json` | Generated when a model price cannot be matched on the API. | No |

The files that should never be committed to a public repository are already excluded by
the project's `.gitignore`. The configuration file is the most important one to keep
private, because it stores your API keys (encrypted, but still secrets).

## First-time configuration

The fastest way to start is to let the script generate a default configuration:

```powershell
./New-GPTImage.ps1 -Setup
```

This creates `New-GPTImage.conf` with sensible defaults and a full set of model
definitions. If the file already exists, `-Setup` does not overwrite it; it exits with a
message, so you cannot accidentally lose your configuration.

After running `-Setup`, open `New-GPTImage.conf` and fill in, for each model you want to
use, the real `endpoint`, `apiKey` and `deployment` values from your Azure AI Foundry
resource. You can paste the API key in clear text: at the first run the script encrypts
it in place (DPAPI on Windows, AES elsewhere) and rewrites the file, so the clear-text key
never stays on disk. See the [configuration reference](config.md) for the meaning of every
field, and [managing models](models.md) for the guided wizards that edit models without
hand-editing the file.

Alternatively, if you are publishing or cloning the project, copy
`New-GPTImage.conf.example` to `New-GPTImage.conf` and edit it the same way.

## Azure infrastructure

New-GPTImage talks to **Azure AI Foundry** (a Cognitive Services resource of kind
`AIServices`) on which one or more image-generation models are deployed. Each model in the
configuration points to an endpoint on that resource and authenticates with an API key.

You have two ways to obtain the infrastructure:

**Provision it yourself** through the Azure portal or CLI: create the Foundry resource,
deploy the image model(s), and copy the endpoint and key into the configuration.

**Let the script provision it** with the Infrastructure as Code feature:

```powershell
./New-GPTImage.ps1 -IaC -IaCMode execute -IaCResourceGroup "rg-newgptimage" -IaCFoundryName "myfoundry" -IaCDeployModel
```

This creates a Resource Group, the Foundry resource, a Foundry Project, optional
Application Insights and Log Analytics for telemetry, and (with `-IaCDeployModel`) an image
model deployment. The model deployment is optional and not guaranteed: its availability
depends on the region and your subscription quotas. The infrastructure provisioning is the
primary goal and succeeds independently of the model deployment. For the full IaC reference
see the [telemetry and infrastructure guide](telemetry.md).

Optionally, telemetry can send generation metrics to Azure Application Insights, which a
companion Azure Monitor Workbook visualizes. This is entirely optional and disabled per run
with `-NoTelemetry`.

## Verifying the installation

Once the configuration has real endpoints and keys, confirm everything is wired correctly
without spending tokens:

```powershell
# Show the configured models and their status (active, endpoint, key present)
./New-GPTImage.ps1 -ModelList

# Probe network reachability of the active models' endpoints
./New-GPTImage.ps1 -CheckModels

# Test that each active endpoint actually responds
./New-GPTImage.ps1 -CheckEndpoint

# Validate parameters and show the final prompt WITHOUT calling the API
./New-GPTImage.ps1 -Prompt "a test prompt" -DryRun
```

When you are ready for a real generation:

```powershell
./New-GPTImage.ps1 -Prompt "A lighthouse on a cliff at sunset, watercolor style" -Model gpt-image-2
```

There is also a built-in self-test suite. The simulated mode runs the checks that do not
consume tokens; the real mode performs actual generations:

```powershell
./New-GPTImage.ps1 -SelfTest -Simulate   # no token cost
./New-GPTImage.ps1 -SelfTest -Real        # generates real images
./New-GPTImage.ps1 -SelfTest -AllTest     # both
```

## Where to go next

- [Configuration reference](config.md) — every field of `New-GPTImage.conf`.
- [Managing models](models.md) — add, edit and remove models.
- [Command-line reference](cli.md) — all CLI parameters with examples.
- [Interactive interface](tui.md) — the full-screen TUI.
