# Analytics

The analytics feature turns your accumulated cost and usage history into readable textual
charts, printed in the terminal (or saved to a file), and can also export the full record
set to CSV for external tools.

## Showing the charts

```powershell
./New-GPTImage.ps1 -Analytics
```

This prints ten textual charts laid out in five rows of two columns. With no extra value it
uses the currency from `defaultCurrency`; you can force a specific view:

```powershell
./New-GPTImage.ps1 -Analytics EUR
./New-GPTImage.ps1 -Analytics USD
./New-GPTImage.ps1 -Analytics Token
```

The first eight charts cover the costs and tokens of image generations (broken down by day,
month and year, input versus output, and so on). The last two charts cover the **text-model
costs** — that is, the cost of `-OptimizePrompt`, `-NegativePrompt`, `-Storyboard` and
`-DescribeImage` operations — tracked separately from the image costs. See the
[text model guide](text-model.md) and the [cost management guide](costs.md).

Analytics reads your local history files; it makes no API calls and does not generate
images.

## Saving the report to a file

Instead of printing to the screen, you can write the charts to a text file:

```powershell
./New-GPTImage.ps1 -Analytics -AnalyticsFile ".\report.txt"
```

If the target file already exists, you control what happens with two mutually exclusive
flags:

| Flag | Behavior when the file exists |
|------|-------------------------------|
| (neither) | The script exits with a warning and does not touch the file. |
| `-Overwrite` | The existing file is overwritten (the action is logged with a warning). |
| `-Append` | The new report is appended to the end of the existing file. |

If the file does not exist, it is created normally in all cases. The path is absolute or
relative to the script folder.

```powershell
./New-GPTImage.ps1 -Analytics -AnalyticsFile ".\report.txt" -Overwrite
./New-GPTImage.ps1 -Analytics -AnalyticsFile ".\history.txt" -Append
```

## Exporting all records to CSV

For a spreadsheet, BI tool or dashboard, export every cost record (from the first to the
last) to a CSV and exit:

```powershell
./New-GPTImage.ps1 -Analytics -ExportAll ".\all-costs.csv"
```

The CSV uses a culture-aware decimal separator so it opens cleanly in your locale's
spreadsheet software. Each row corresponds to a recorded generation, with its date, model,
totals and related fields — suitable for pivoting and charting outside the tool.

## Where the data comes from

Analytics is built from the local cost log and the daily cost aggregate (the `.cost` and log
files described in the [cost management guide](costs.md)). The richer your history, the more
informative the charts; a fresh installation with no generations yet will show empty charts.

For an interactive, always-current visualization on Azure rather than textual charts, see the
telemetry Workbook in the [telemetry guide](telemetry.md).
