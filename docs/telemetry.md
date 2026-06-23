# Telemetry, Workbook and Infrastructure as Code

This guide covers three related, optional capabilities: sending generation telemetry to
Azure, visualizing it with an Azure Monitor Workbook, and provisioning the whole Azure
infrastructure with the Infrastructure as Code (IaC) feature.

## Telemetry

When enabled, the script sends per-generation metrics to **Azure Application Insights**.
This is controlled by the `telemetry` section of the configuration (see the
[configuration reference](config.md)):

- `azureMonitor` — whether telemetry is sent.
- `connectionString` — the Application Insights connection string (sensitive — treat it
  like a credential).

Telemetry is entirely optional. You can disable it for a single run, regardless of the
config value, with:

```powershell
./New-GPTImage.ps1 -Prompt "..." -NoTelemetry
```

Each telemetry event carries a `source` property indicating where the generation came from
— `cli`, `tui`, `batch` or `benchmark` — so you can distinguish usage patterns. At startup
the script logs whether telemetry is active, so you always know if a run is being tracked.
The self-test suite always runs its subprocess generations with `-NoTelemetry`, so test
images never pollute your real metrics.

## The Workbook

A companion **Azure Monitor Workbook** visualizes the telemetry as a dashboard of
time-series charts (costs, tokens, images, durations, usage parameters, and more) directly
in the Azure portal. It is created or updated as part of the IaC `execute` flow.

Two practical notes:

- The charts populate as you generate images with telemetry active; immediately after
  creation they may be empty simply because no data has arrived yet.
- The Workbook content (chart titles, labels, axes) is in English and fixed at provisioning
  time; it does not follow the `-Lang` setting, unlike the script's own on-screen output.

The Workbook is identified on Azure by a deterministic identifier, so re-running the
provisioning updates the existing Workbook in place rather than creating duplicates.

## Infrastructure as Code (-IaC)

The `-IaC` feature provisions the Azure resources the script needs, using the Azure CLI
(`az`) as its only dependency. It creates a Resource Group, the Foundry resource
(Cognitive Services of kind `AIServices`), a Foundry Project, and — for telemetry — Log
Analytics and Application Insights, plus optionally an image-model deployment.

### Modes

The `-IaCMode` parameter selects what happens:

| Mode | Effect |
|------|--------|
| `generate` | Write the provisioning script without touching Azure. |
| `dry-run` | Show what would be created, without creating it. |
| `execute` | Create the resources, after explicit confirmation. |

```powershell
# See what would be created
./New-GPTImage.ps1 -IaC -IaCMode dry-run -IaCResourceGroup "rg-newgptimage" -IaCFoundryName "myfoundry"

# Create it
./New-GPTImage.ps1 -IaC -IaCMode execute -IaCResourceGroup "rg-newgptimage" -IaCFoundryName "myfoundry" -IaCDeployModel
```

### Parameters

| Parameter | Meaning |
|-----------|---------|
| `-IaCResourceGroup` | The Resource Group name. |
| `-IaCRegion` | The Azure region. |
| `-IaCFoundryName` | The Foundry resource name. |
| `-IaCProjectName` | The Foundry Project name. |
| `-IaCDeployModel` | Also deploy an image model (optional, see below). |
| `-IaCModelName` | The model to deploy (default `gpt-image-1`). |
| `-IaCModelVersion` | The model version. |
| `-IaCModelFormat` | `OpenAI` (gpt-image) or `Microsoft` (MAI). |
| `-IaCModelSku` | The deployment SKU (default `GlobalStandard`). |
| `-IaCModelCapacity` | The deployment capacity (default 1). |
| `-IaCTeardown` | Destroy the created resources (deletes the Resource Group and its content). Destructive. |
| `-IaCPurgeSoftDeleted` | Permanently remove a soft-deleted Foundry resource before recreating it. |
| `-IaCWizard` | Guided interactive collection of the IaC parameters, then execution. |

### The model deployment is optional

The infrastructure provisioning is the primary goal and succeeds on its own. The image-model
deployment (`-IaCDeployModel`) is separate and **not guaranteed**: whether a given model can
be deployed depends on the region and your subscription quotas. If the deployment step fails,
the infrastructure is still valid — you can deploy a model later, by hand or by re-running
with different parameters.

### Soft-deleted resources

Cognitive Services resources are not removed immediately when deleted; they enter a
"soft-deleted" state for a period. If you try to recreate a Foundry resource whose name is
still soft-deleted, the IaC stops and explains the situation. Add `-IaCPurgeSoftDeleted` to
permanently purge it first and then recreate it.

### Teardown

To remove everything the IaC created:

```powershell
./New-GPTImage.ps1 -IaC -IaCTeardown -IaCResourceGroup "rg-newgptimage"
```

This deletes the Resource Group and all its contents after an explicit confirmation. It is a
destructive operation — use it deliberately.

### The wizard

If you prefer not to remember all the parameters, the guided wizard asks for the Resource
Group, region, names and options (with sensible defaults) and then proceeds:

```powershell
./New-GPTImage.ps1 -IaCWizard
```

## Inspecting the generated provisioning script

In `generate` mode the IaC writes a provisioning script made of plain `az` commands, which
you can review before running. This is the most transparent way to see exactly what will be
created on your subscription.
