# Contributing to New-GPTImage

Thank you for your interest in the project. This document collects the conventions
and best practices followed during development, so that contributions stay
consistent with the rest of the code.

## License of Contributions

The project is distributed under the **GNU General Public License v3.0**. By
submitting a contribution (pull request, patch) you agree that it will be released
under the same license. Keep the copyright and license notices in the files intact.

## Development Environment

- **PowerShell 7.0+** is mandatory. The script does not support Windows PowerShell
  5.1 (the UTF-8 handling required for multilingual support is reliable only from
  PowerShell 7).
- Development and reference testing happen on **Windows 11**. Generation, logging
  and configuration also work on Linux/macOS; image editing (edit/inpainting)
  requires Windows.
- Before proposing changes, run static analysis with **PSScriptAnalyzer** and fix
  the relevant warnings.

## Code Conventions

### Comments

- The comments in the code are in **English**. Documentation blocks and inline
  comments should be clear and concise.
- The **string values** in the `.lang` file are user-facing text and are translated
  into all supported languages.
- Each function should have a `<# ... #>` documentation block explaining its
  purpose, behavior and main parameters.

### Versioning

The project has three components versioned independently in the script:

- `$script:ScriptVersion` — the script
- `$script:TuiVersion` — the interactive interface
- `$script:GalleryVersion` — the HTML gallery

For the script: a **bugfix** increments by `+0.01`; a **new feature** moves up to
the next decade. Always update the version and the [changelog](changelog.md) when
you propose a significant change.

### Internationalization

- Every user-facing string goes through the `T 'namespace.key'` function and must
  be added to the `New-GPTImage.lang` file.
- The `.lang` is a JSON with one entry per key, each translated into **all 11
  languages**: `it, en, fr, es, pt, de, ru, zh, ja, hi, ar`. An incomplete key is
  considered an error.
- The `_total_keys` field must always match the number of translations present.
- Placeholders (`{0}`, `{1}`, ...) must be consistent across all languages.

## Integrity Check (baseline)

After **every** change to the script, the structural integrity must be verified.
The project maintains a known baseline:

- Delimiter balance: round parentheses `-9`, braces `0`, brackets `0` (the `-9` is
  due to parentheses present inside string literals, and is constant).
- Balanced comment blocks `<# ... #>` (same number of openings and closings).
- Zero CRLF line terminators (the file uses LF).
- Zero duplicate functions.

A quick check in Python:

```python
raw = open('New-GPTImage.ps1','rb').read(); s = raw.decode('utf-8')
print('Round:', s.count('(') - s.count(')'))   # expected -9
print('Braces:', s.count('{') - s.count('}'))  # expected 0
print('Brackets:', s.count('[') - s.count(']'))  # expected 0
print('Comment blocks:', s.count('<#'), s.count('#>'))  # must match
print('CRLF:', raw.count(b'\r\n'))              # expected 0
```

## Security

- **Never commit** the `New-GPTImage.conf` file: it contains the API keys. Use
  `New-GPTImage.conf.example` as a template, with sensitive values replaced by
  placeholders.
- Do not put real endpoints, keys, instrumentation keys or personal paths in
  versioned files.
- API keys must be handled encrypted (DPAPI on Windows); do not introduce code that
  writes them in clear text to logs or disk.

## Pull Request Process

1. First open an **issue** describing the bug or the proposal, so its scope can be
   discussed.
2. Work on a dedicated branch.
3. Keep changes **incremental and focused** on one logical area at a time.
4. Verify the baseline, the completeness of the `.lang` and PSScriptAnalyzer before
   opening the PR.
5. Update the version and changelog if the change requires it.
6. Describe in the PR what changes and how you verified it.

Thank you for your contribution!
