# TogetherLink CLI Reference

Commands and flags as of CLI v0.9.82 (2026-10-13). TogetherLink is beta and updates itself; confirm
with `togetherlink --help` and `togetherlink image --help`. Source of truth for behavior:
[Together Link docs](https://docs.together.ai/docs/togetherlink).

## Contents

- [Command Shape](#command-shape)
- [Install](#install)
- [Launch Commands](#launch-commands)
- [Global Flags and API Keys](#global-flags-and-api-keys)
- [Account and Information Commands](#account-and-information-commands)
- [Desktop Integrations](#desktop-integrations)
- [Image Commands](#image-commands)
- [Codex Security](#codex-security)
- [Headless Pattern](#headless-pattern)
- [Troubleshooting](#troubleshooting)

## Command Shape

```text
togetherlink [global flags] TOOL [arguments passed through to the tool]
```

`tlink` is short for `togetherlink`. Global flags must come before the tool name; everything after
it passes through to the tool. TogetherLink installs no local proxy or daemon: each agent connects
directly to the hosted gateway.

## Install

```bash
curl -fsSL https://link.together.ai/install | bash
```

- Requires macOS or Linux, Bash, and curl. The installer also installs Bun if needed (on Linux that
  needs `unzip`) and adds the commands to `~/.local/bin`.
- If `togetherlink` is not found in a new terminal, load the startup file the installer updated
  (`source ~/.zshrc` or `source ~/.bashrc`).
- TogetherLink does not install the agents. It supports Claude Code, Codex, OpenCode 2, Pi Code
  0.80.8 or newer (Node.js 22.19 or newer), Claude Desktop, and ChatGPT Desktop.

## Launch Commands

| Command | Shortcut | Launches |
|---|---|---|
| `togetherlink` | `tlink` | Interactive launcher (prompts for a key if none is configured) |
| `togetherlink claude [args]` | `tclaude` | Claude Code |
| `togetherlink codex [args]` | `tcodex` | Codex CLI |
| `togetherlink opencode [args]` | `topencode` | OpenCode 2 |
| `togetherlink pi [args]` | `tpi` | Pi Code (`togetherlink picode` also works) |
| `togetherlink claude-desktop` | `tclaude-desktop` | Claude Desktop, Cowork, and Code (beta, macOS and Linux) |
| `togetherlink chatgpt` | none | ChatGPT Desktop on its own profile (beta, macOS and Linux; alias `chatgpt-desktop`) |

Terminal agents get a temporary per-launch configuration that is removed when the session ends;
your normal configuration files are never modified. Every session starts with a routing banner such
as `Claude Code › Auto` and prints a cost summary when it exits.

## Global Flags and API Keys

| Flag | Purpose |
|---|---|
| `--main MODEL_ID` | Pin the session to one model instead of `auto` (see models.md) |
| `--api-key KEY` | Use this Together API key for one run |

API keys are read in this order: `--api-key` before the command, then the saved
`~/.togetherlink/config.json`, then an exported `TOGETHER_API_KEY`. A saved key therefore wins over
an exported one. Project `.env` files are not read.

## Account and Information Commands

| Command | Purpose |
|---|---|
| `togetherlink configure` | Add or replace API keys (validates the Together key; optionally asks for an Anthropic key) |
| `togetherlink usage [--last 7d]` | Spend tracked by the cloud gateway across sessions, tokens and images |
| `togetherlink models` | Models, Claude tiers, context, and prices (alias `togetherlink list`) |
| `togetherlink update` | Update to the latest release (it also checks for updates on its own) |
| `togetherlink --version` | Installed version |

Claude Code's own cost figures (`total_cost_usd` in `--output-format json`, `/cost`) use Anthropic
prices and are wrong under TogetherLink; `togetherlink usage` and the session receipt show actual
spend.

## Desktop Integrations

Each desktop app gets a persistent profile owned by TogetherLink that stores the Together API key.

| Command | Effect |
|---|---|
| `togetherlink claude-desktop` | Configure Claude Desktop, Cowork, and Code to use Together AI models |
| `togetherlink claude-desktop off` | Back to official Claude; keep the TogetherLink profile |
| `togetherlink claude-desktop reset [--yes]` | Remove the TogetherLink profile and its imported copies |
| `togetherlink chatgpt` | Launch ChatGPT Desktop on the dedicated `~/.codex-togetherlink` profile |
| `togetherlink chatgpt off` | Relaunch on the normal profile |
| `togetherlink chatgpt reset [--yes]` | Delete the dedicated profile, its local tasks, and settings (`chatgpt restore` is an alias) |
| `togetherlink chatgpt preview [dir]` | Write a test Responses config |

- If Claude is running, TogetherLink asks before quitting and restarting it. On macOS, Claude
  Desktop must be installed in `/Applications` or `~/Applications`.
- Reset asks for confirmation; `--yes` confirms in scripts and lets TogetherLink quit and reopen a
  running app.
- TogetherLink reads but never rewrites the normal `~/.codex` home. Native web search is disabled
  in the ChatGPT Desktop profile.

## Image Commands

```text
togetherlink image generate --prompt TEXT [--aspect RATIO] [--quality auto|draft|standard|best]
                            [--seed INT] [--out PATH] [--force] [--json]
togetherlink image edit --image PATH [--image PATH ...] --prompt TEXT [--aspect RATIO]
                        [--quality auto|draft|standard|best] [--seed INT] [--out PATH] [--force] [--json]
```

| Flag | Meaning |
|---|---|
| `--prompt` | Text prompt (required) |
| `--image` | Input image for `edit`; repeat for several references |
| `--aspect` | `1:1`, `16:9`, `9:16`, `4:3`, `3:4`, `3:2`, `2:3`, or `1.91:1`; without it the model chooses (usually square) |
| `--quality` | `auto` (default), `draft`, `standard`, or `best` |
| `--seed` | Integer seed for repeatable output |
| `--out` | Output path; default `output/imagegen/output.png` |
| `--force` | Overwrite an existing output file |
| `--json` | Machine-readable result |

The image model is selected remotely, and exact pixel sizes vary by model, so resize or crop when a
precise size matters. Image spend appears in `togetherlink usage`, separately from token spend.

## Codex Security

```bash
npm install -g @openai/codex-security
togetherlink codex-security scan [path]
```

It sets the gateway for that process and defaults to GLM 5.3 at high effort; `--model` and
`--effort` override either.

## Headless Pattern

Each tool's own non-interactive flags pass through:

```bash
togetherlink claude -p "TASK" --output-format json < /dev/null
togetherlink codex exec "TASK"
togetherlink opencode run "TASK"
togetherlink pi --mode json -p "TASK"
togetherlink --main deepseek-ai/DeepSeek-V4.1-Flash claude -p "TASK" < /dev/null
```

Always append `< /dev/null` to headless Claude Code runs: a process that inherits an open stdin
blocks indefinitely.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `togetherlink: command not found` | Open a new terminal or `source ~/.zshrc` / `~/.bashrc` |
| No API key configured | `togetherlink configure`, or export `TOGETHER_API_KEY` (a saved key wins over it) |
| Agent not installed | TogetherLink does not install agents; run the install command printed in the error |
| OpenCode 1 installed | `npm uninstall -g opencode-ai`, then `npm install -g @opencode/cli@latest` |
| Pi Code too old | `pi update self`, or `npm install -g --ignore-scripts @earendil-works/pi-coding-agent` (needs Node.js 22.19+) |
| Claude `--model` rejected | Use `togetherlink --main ID claude -- CLAUDE_ARGS` |
| Headless Claude Code run hangs | Append `< /dev/null` |
| JSON output reports a high cost | `total_cost_usd` uses Anthropic prices; read `togetherlink usage` |
| No co-author trailer on commits and PRs | TogetherLink blanks Claude Code's commit and PR attribution; add one yourself if needed |
| Auto router never uses Claude Opus | Add an Anthropic key with `togetherlink configure`; only Claude sessions use it |
| Desktop app still on TogetherLink models | Run the matching `off` command |
| Restore the original setup | Launch the tool directly; desktop apps need `off` (or `reset`) |

Report bugs at https://github.com/Nutlope/togetherlink/issues.
