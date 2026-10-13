# Together AI: TogetherLink

TogetherLink (`togetherlink`, alias `tlink`; beta) runs coding agents and desktop apps you already
have on models hosted by Together AI: Claude Code, Codex, OpenCode, Pi Code, Claude Desktop and
Cowork, and ChatGPT Desktop. Each tool connects directly to TogetherLink's hosted gateway; nothing
runs locally and normal config files are not modified.

Hand-offs: calling Together models from the user's own code is `domains/chat-completions.md`;
generating images from code with a chosen model is `domains/images.md`.

## Workflow

1. Install if needed: `curl -fsSL https://link.together.ai/install | bash` (macOS or Linux). It does
   not install the agents themselves.
2. Key: `togetherlink configure` saves and validates the Together API key, and optionally an
   Anthropic key for the auto router. It is interactive, so ask the user to run it, or use an
   exported `TOGETHER_API_KEY` in scripts and CI.
3. Launch: `tclaude`, `tcodex`, `topencode`, `tpi`, `tclaude-desktop`, or `togetherlink chatgpt`.
   Arguments after the tool name pass through to it.
4. Pick the model: sessions default to `auto`; pin one with `togetherlink --main MODEL_ID TOOL`.
   Run `togetherlink models` for the live list, Claude tiers, and prices.
5. Check spend with `togetherlink usage --last 7d`.
6. To stop using it: launch the tool directly; desktop apps need `togetherlink claude-desktop off`
   or `togetherlink chatgpt off` (`reset` also deletes the profile).

## Rules

- Put `--main` and `--api-key` before the tool name; everything after it goes to the tool. The
  shortcuts (`tclaude` and the rest) cannot pin a model.
- Never pin with the agent's own `--model`: Claude Code's is refused because Claude aliases do not
  select gateway routes. Use `togetherlink --main ID claude -- CLAUDE_ARGS`. A `model` key in
  Claude Code's `settings.json` does not change the session model either.
- Inside Claude Code, `/model` tiers map to Together models (check `togetherlink models`; on
  2026-10-13 Opus ran Kimi K3, Fable GLM 5.3, Sonnet DeepSeek V4.1 Flash, Haiku MiniMax M3).
- The auto router sends hard requests to Claude Opus only in Claude Code and Claude Desktop sessions
  with an Anthropic key; every other tool always stays on Together models.
- Keys are read from `--api-key`, then the saved `~/.togetherlink/config.json`, then
  `TOGETHER_API_KEY`, so a saved key overrides an exported one. Project `.env` files are ignored.
- Headless Claude Code runs must end with `< /dev/null`, or they block on stdin forever.
- Report spend from `togetherlink usage` or the session receipt. Claude Code's `total_cost_usd` and
  `/cost` use Anthropic prices and are wrong here; do not quote prices from memory.
- Image generation: Claude Code, Claude Desktop, and ChatGPT Desktop sessions include an image
  skill; elsewhere use `togetherlink image generate` or `image edit`.
- TogetherLink blanks Claude Code's commit and PR attribution, so add a co-author trailer yourself
  if one is required.
- It is beta and updates itself: confirm commands with `togetherlink --help` before scripting them.

## Open next

| File | Lines | Contains | Open when |
|---|---|---|---|
| [references/togetherlink/cli-reference.md](references/togetherlink/cli-reference.md) | 170 | install, every launch, account, desktop, image, and Codex Security command; key order; headless patterns per tool; troubleshooting | a command, flag, desktop integration, or error |
| [references/togetherlink/models.md](references/togetherlink/models.md) | 61 | model and price snapshot, auto router rules, pinning, Claude tier mapping, subagent behavior | choosing or pinning a model, or explaining which model served a request |

## Docs

- [Together Link](https://docs.together.ai/docs/togetherlink)
- [Together Link on GitHub](https://github.com/Nutlope/togetherlink/issues) (issues and feedback)
