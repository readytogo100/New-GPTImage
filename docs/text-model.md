# The text model

Several features use a separate **text / multimodal model** to work on prompts rather than
to draw images: rewriting a prompt, building a negative prompt, decomposing a narrative into
a storyboard, and describing an image. This guide explains how to configure that model and
how to get the best results from each feature.

## What the text model is for

The text model is configured in the `textModel` section of `New-GPTImage.conf` (see the
[configuration reference](config.md)) and powers four features:

| Feature | Parameter | What it does |
|---------|-----------|--------------|
| Prompt optimization | `-OptimizePrompt` | Rewrites and enriches your prompt for a stronger image. |
| Negative prompt | `-NegativePrompt` | Generates things to avoid and injects them into the positive prompt. |
| Storyboard | `-Storyboard` / `-SequenceImages` | Decomposes a narrative into N coherent scenes. |
| Image description | `-DescribeImage <path>` | Analyzes an image and produces a descriptive prompt (image-to-prompt). |

These are separate from image generation and have their own, separately-tracked cost (see
the [cost management guide](costs.md)).

## Configuring the text model

The `textModel` fields:

| Field | Meaning |
|-------|---------|
| `endpoint` | The model's endpoint. Both `/responses` and `/chat/completions` style endpoints are supported and auto-detected. |
| `apiKey` | The API key (encrypted on first run, like the image keys). |
| `model` | The deployment/model name. |
| `authMode` | `api-key` or `bearer`. |
| `maxCompletionTokens` | Maximum tokens in the response (the reasoning/output budget). |
| `reasoningEffort` | Reasoning effort hint (e.g. `low`, `medium`, `high`) for models that support it. |
| `timeoutSec` | Request timeout in seconds. Reasoning models and long storyboards can take a while. |
| `inputUsdPerMillion` | Price per million input tokens, for cost tracking. |
| `outputUsdPerMillion` | Price per million output tokens, for cost tracking. |

The script auto-detects the endpoint style: a `/responses` endpoint uses the response-style
request (with the token budget as `max_output_tokens`), while a `/chat/completions` endpoint
uses the chat-style request. You therefore do not need to declare the API type — just point
`endpoint` at the right URL.

Fill in `inputUsdPerMillion` and `outputUsdPerMillion` from your model's pricing page;
until you do, the text-cost tracking reports zero.

## Prompt optimization

```powershell
./New-GPTImage.ps1 -Prompt "a quiet harbor at dawn" -OptimizePrompt -Model gpt-image-2
```

The text model rewrites your prompt into a richer, more descriptive version before it is
sent to the image model. Because the call to the text model can take time, a waiting message
is shown; the full prompts are written only to the log file (subject to your privacy
settings), not to the console, to avoid clutter.

This feature is an **enhancement**: if the text model is unavailable, the script warns and
proceeds with your original prompt rather than failing.

## Negative prompt

```powershell
./New-GPTImage.ps1 -Prompt "portrait of a knight" -NegativePrompt -Model gpt-image-2
```

The text model produces a list of things to avoid. Because the gpt-image and MAI models do
not have a native negative-prompt field, the negative is injected "softly" into the positive
prompt (as an "avoid: ..." clause), which makes the feature work across all models. Like
optimization, it is an enhancement and degrades gracefully if the text model is unavailable.

## Storyboard

```powershell
./New-GPTImage.ps1 -Prompt "a hero's journey through a magical forest" -Storyboard -SequenceImages 8
```

The text model decomposes your narrative into N scenes (2–20, default 5). It also produces a
"style bible" — palette, technique, mood, descriptions of recurring characters — which is
repeated at the top of every scene's prompt to keep the images visually consistent. The
scenes are generated in sequence and saved with numbered names (`story_01`, `story_02`, ...),
and a manifest of the sequence is written.

Storyboard **depends** on the text model (it cannot decompose the narrative without it): if
the text model is unavailable, this is a blocking error, unlike optimization and negative.

For long sequences, the per-scene token budget scales with the number of scenes, which
avoids timeouts on big storyboards. If you hit a timeout anyway, raise `timeoutSec` in the
`textModel` configuration (and remember you can apply the change live in the TUI with
`/reload`).

## Image description (image-to-prompt)

```powershell
./New-GPTImage.ps1 -DescribeImage ".\reference.png"
```

The multimodal text model analyzes the image and produces a descriptive prompt from it,
useful as a starting point for new generations or for understanding how to phrase a prompt
that reproduces a style. The output is in the script's active language. Like the storyboard,
this depends on the multimodal model being available.

## Choosing a good text model

A few practical pointers:

- Use a capable, **multimodal** model if you want `-DescribeImage` to work well, since it
  must understand images, not just text.
- If your model supports reasoning, set `reasoningEffort` to balance quality against speed
  and cost; higher effort generally yields better prompt rewrites and storyboards but costs
  more tokens and takes longer.
- Give storyboards enough room: a higher `maxCompletionTokens` and a generous `timeoutSec`
  prevent truncated or timed-out sequences when you ask for many scenes.
- Always set the two price fields so the [analytics](analytics.md) and
  [cost](costs.md) views reflect the real text-model spend.

## Privacy

What gets logged from these features is governed by the `privacy.supportPrompts` flag (and
`privacy.imagePrompts` for the final image prompt). By default prompts are not written to the
log; enabling the flags writes them obfuscated in Base64. See the
[configuration reference](config.md).

## Cost

All four features consume text-model tokens and are billed separately from image generation,
with their own `[COST]` lines and dedicated analytics charts. See the
[cost management guide](costs.md) for how the text-model cost is computed and tracked.
