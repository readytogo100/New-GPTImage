# Security Policy

## Supported Versions

The most recent version of the script receives security fixes. You are strongly
encouraged to always update to the latest release before reporting an issue.

| Version          | Supported |
|------------------|-----------|
| 5.x (latest)     |   Yes     |
| Earlier versions |    No     |

## Reporting a Vulnerability

If you discover a security vulnerability, **do not open a public issue**: an issue
is visible to everyone and could expose the problem before it is fixed.

Instead, report the vulnerability privately through the
**"Report a vulnerability"** function in the *Security* tab of the GitHub
repository (Private Vulnerability Reporting), or by contacting the maintainer
directly.

In your report, please include as much of the following as possible:

- a description of the problem and its impact;
- the steps to reproduce it;
- the version of the script affected;
- any relevant logs, **with sensitive information removed**.

You will receive a response as soon as possible. Once the vulnerability has been
confirmed and fixed, we will agree together on the timing of any public disclosure.

## Security Notes for Users

This script handles credentials and interfaces with paid Azure services.
Some recommendations:

- **API keys.** They are stored encrypted in the configuration file (DPAPI on
  Windows). Never share your `New-GPTImage.conf`, do not include it in
  screenshots or logs, and do not commit it to the repository (the `.gitignore`
  excludes it).
- **Log files.** With prompt logging enabled, prompts can end up in the log file
  (possibly obfuscated). Consider the content before sharing a log; by default
  this logging is disabled.
- **Telemetry.** Sending telemetry to Azure Application Insights is optional and
  can be disabled (`-NoTelemetry` or via configuration). The connection string is
  sensitive data: treat it like a credential.
- **Costs.** The script makes calls to pay-as-you-go services. Use the `-Budget`
  parameter as a guardrail to avoid unexpected expenses.

Thank you for helping keep the project secure.
