# TogetherLink Models Reference

TogetherLink is beta and its model list, Claude tier mapping, and prices change; the CLI fetches
them from the gateway at launch. Run `togetherlink models` (alias `togetherlink list`) for the live
table before pinning a model or quoting a price. The table below is a snapshot from CLI v0.9.82 on
2026-10-13; the [docs page](https://docs.together.ai/docs/togetherlink) can lag the CLI.

## Models

Prices are USD per 1M tokens (input, cached input, output).

| Model | Model ID | Claude Code tier | Context | In | Cache | Out |
|---|---|---|---|---|---|---|
| Auto (default) | `auto` | routes per request | 1M | | | |
| Kimi K3 | `moonshotai/Kimi-K3` | Opus | 1M | 2.70 | 0.27 | 13.50 |
| GLM 5.3 | `zai-org/GLM-5.3` | Fable | 1M | 1.40 | 0.26 | 4.40 |
| DeepSeek V4.1 Flash | `deepseek-ai/DeepSeek-V4.1-Flash` | Sonnet | 1M | 0.30 | 0.01 | 1.20 |
| MiniMax M3 | `MiniMaxAI/MiniMax-M3` | Haiku | 524K | 0.30 | 0.06 | 1.20 |
| Qwen 3.8 | `Qwen/Qwen3.8-2.4T-A95B` | none | 1M | 2.00 | 0.25 | 6.00 |

TogetherLink's prices can differ from the serverless catalog's (Kimi K3 is listed at 3.00 / 15.00
there), so quote `togetherlink models`, not the catalog.

## Auto Router

`auto` is a virtual model. For each request the gateway classifies the prompt and routes it:

- straightforward requests go to a Together AI model (such as GLM 5.3)
- harder requests go to Claude Opus through Anthropic, billed to the Anthropic API key configured
  for TogetherLink, but only in Claude Code and Claude Desktop sessions that have that key

Codex, OpenCode, Pi Code, and ChatGPT Desktop sessions always stay on Together AI models, as do
Claude sessions without an Anthropic key.

## Pinning A Model

```bash
togetherlink --main zai-org/GLM-5.3 claude
togetherlink --main moonshotai/Kimi-K3 opencode
togetherlink --main deepseek-ai/DeepSeek-V4.1-Flash codex
```

- `--main` goes before the tool name and works for every tool.
- The shortcuts (`tclaude`, `tcodex`, `topencode`, `tpi`) expand with the tool name first, so they
  cannot pin a model; use the full `togetherlink --main ID TOOL` form.
- A `--model` flag after `claude` is refused: "Together Link cannot forward Claude's --model flag
  because Claude aliases do not select gateway routes." Other tools may ignore it.

## Claude Code Tier Mapping

Inside a Claude Code session, `/model` shows Claude's tiers, and each tier runs a Together AI model
(live mapping from `togetherlink models`; snapshot above): Opus runs Kimi K3, Fable runs GLM 5.3,
Sonnet runs DeepSeek V4.1 Flash, and Haiku runs MiniMax M3.

TogetherLink sets the session model itself, so a `model` key in your own Claude Code
`settings.json` does not change it; use `--main` or `/model`. It also runs subagents on the
session's model by default (`CLAUDE_CODE_SUBAGENT_MODEL=inherit`) unless you set that variable
yourself.

Codex, OpenCode, Pi Code, and the desktop apps expose their own model menus, which can differ from
Claude Code's tiers.
