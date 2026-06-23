# The interactive interface (TUI)

`-TUI` launches a full-screen terminal interface for working interactively, without
retyping command-line parameters for every generation. It is ideal for iterating on prompts
and parameters in a single session.

```powershell
./New-GPTImage.ps1 -TUI
```

The TUI reads the main configuration file (the `tui` section). If that section is missing,
it is created in memory with default values, so the interface works even on a minimal
config. For the `tui` configuration fields and the available themes, see the
[configuration reference](config.md).

## Layout

The screen is divided into regions:

- a **header** showing the current model and parameters and a live clock;
- a scrollable **message body** with word wrap;
- two **status bars**, one of which lists the active features;
- a **command line** at the bottom, with history and autocompletion of commands and paths.

Rendering uses synchronized terminal output to avoid flicker, and during generation only the
progress bar region is updated.

## Commands

You drive the TUI by typing slash-commands on the command line. The complete set:

### Setting generation parameters

| Command | Effect |
|---------|--------|
| `/prompt` | Open the full-screen prompt editor (see below). |
| `/promptfile <path>` | Load the prompt from a file. |
| `/model <name>` | Set the model. |
| `/auto <Cost\|Quality\|Time>` | Enable automatic model selection by strategy. |
| `/nimages <n>` | Set how many images to generate. |
| `/size <WxH>` | Set the output size. |
| `/quality <low\|medium\|high>` | Set the quality. |
| `/format <png\|jpg\|webp>` | Set the output format. |
| `/moderation <low\|auto>` | Set the moderation policy. |
| `/inputfidelity <low\|high>` | Set input fidelity (for models that support it). |
| `/metadata <true\|false>` | Toggle writing the `.json` metadata sidecar. |
| `/background <auto\|opaque\|transparent>` | Set the background type (for supporting models). |
| `/mask <path>` | Set an inpainting mask. |
| `/inputimages <path1,path2,...>` | Set reference images for editing. |
| `/outputfolder <path>` | Set the output folder. |
| `/outputmode <file\|base64>` | Set the output mode. |
| `/budget <amount\|off>` | Set or clear a spend limit for the session. |

### Text-model features

| Command | Effect |
|---------|--------|
| `/optimize [on\|off]` | Toggle prompt optimization via the text model. |
| `/negative [on\|off]` | Toggle negative-prompt injection. |
| `/storyboard [on\|off]` | Toggle storyboard mode. |
| `/sequenceimages <2-20>` | Set the number of storyboard scenes. |

### Session and appearance

| Command | Effect |
|---------|--------|
| `/theme <name>` | Switch the color theme at runtime. |
| `/generate` | Run a generation with the current parameters. |
| `/clear` | Clear the message body. |
| `/reset` | Reset all parameters to the configuration defaults. |
| `/reload` | Re-read `New-GPTImage.conf` from disk and update the session without closing the TUI. |
| `/help` | Show the list of commands and shortcuts. |
| `/exit` | Leave the TUI. |

The `/reload` command is particularly useful: if you edit the `.conf` by hand while the TUI
is open (for example, the `timeoutSec` or `maxCompletionTokens` of the text model), `/reload`
applies the change without restarting.

## The prompt editor

Typing `/prompt` opens a full-screen editor with word wrap and multi-line cursor movement,
so you can compose long prompts comfortably. You confirm the text with the key configured in
`tui.editorConfirmKey` (default `F2`). The editor supports copying with Ctrl+C, and you can
scroll the message body with Ctrl+Home / Ctrl+End.

## Themes

The interface ships with nine color themes: `midnight`, `forest`, `amber`, `arctic`,
`paper`, `matrix`, `dracula`, `ocean` and `contrast`. Switch at runtime with, for example:

```
/theme dracula
```

Each theme defines colors for the header, body (including info/success/warning/error
message colors), separators, the two status bars, the command line, the editor and the
progress bar. You can set the default theme with the `tui.theme` key, and customize or add
themes under `tui.themes` in the configuration.

## A typical session

```
/model gpt-image-2
/size 1536x1024
/quality high
/prompt              (opens the editor; type the prompt, confirm with F2)
/optimize on
/generate
```

After generating, you can change one parameter (say `/quality medium`) and `/generate`
again, iterating quickly within the same session. When you are done, `/exit`.

## Relationship to the CLI

Everything the TUI does is also available from the command line (see the
[command-line reference](cli.md)); the TUI is a convenience layer for interactive work.
Generations made in the TUI are logged and tracked exactly like CLI generations, including
cost tracking and (unless disabled) telemetry, with the telemetry source recorded as `tui`.
