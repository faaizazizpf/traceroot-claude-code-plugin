# traceroot-claude-code-plugin

Observability plugin for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that sends per-turn traces to [TraceRoot](https://traceroot.ai).

Each conversation turn becomes a trace in TraceRoot showing the LLM call, tool invocations, token usage, and any nested sub-agents — all grouped by session.

---

## Installation

### Prerequisite: Claude Code

Install [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and confirm `claude --version` works before continuing.

**Windows (PowerShell):**

```powershell
irm https://claude.ai/install.ps1 | iex
```

If `claude` is not recognized afterwards, add `%USERPROFILE%\.local\bin` to your user `PATH`
(see [Troubleshooting](#troubleshooting)), then fully restart your terminal or IDE.

### Plugin marketplace (recommended)

```bash
claude plugin marketplace add traceroot-ai/traceroot-claude-code-plugin
claude plugin install traceroot-observability@traceroot-claude-code-plugin
```

Or, from within Claude Code using the `/plugin` command:

```
/plugin marketplace add traceroot-ai/traceroot-claude-code-plugin
```

### Manual installation

Clone the repo and register the hook directory with Claude Code:

```bash
git clone https://github.com/traceroot-ai/traceroot-claude-code-plugin.git ~/.claude/plugins/traceroot
# then add the hooks path to your Claude Code config
```

On Windows (PowerShell) :

```powershell
git clone https://github.com/traceroot-ai/traceroot-claude-code-plugin.git "$env:USERPROFILE\.claude\plugins\traceroot"
# then add the hooks path to your Claude Code config
```

---

## Configuration

Set the following environment variables (or add them to your shell profile). Each variable can also be supplied as `CLAUDE_PLUGIN_OPTION_<NAME>` (for example, `CLAUDE_PLUGIN_OPTION_TRACEROOT_API_KEY`).

| Variable | Required | Description |
|---|---|---|
| `TRACEROOT_API_KEY` | Yes | Your TraceRoot API key. Find it at [app.traceroot.ai](https://app.traceroot.ai) → Settings → API Keys. |
| `TRACEROOT_HOST_URL` | No | TraceRoot ingest endpoint. Defaults to the hosted service. Set this only when self-hosting. |
| `TRACEROOT_PLUGIN_MAX_CHARS` | No | Maximum characters captured per content field (prompts, replies, thinking, tool inputs/outputs). Defaults to `20000`. |
| `TRACEROOT_PLUGIN_CAPTURE_CONTENT` | No | Set to `0`, `false`, `no`, or `off` to stop sending prompt/response content. Defaults to `true`. See [Privacy](#privacy). |

Example (add to `~/.zshrc` or `~/.bashrc`):

```bash
export TRACEROOT_API_KEY="tr-..."
# Optional: only needed for self-hosted deployments
# export TRACEROOT_HOST_URL="https://your-traceroot.example.com"
```

**Windows (PowerShell), persistent for your user:**

```powershell
[Environment]::SetEnvironmentVariable("TRACEROOT_API_KEY", "tr-...", "User")
# Optional: only needed for self-hosted deployments
# [Environment]::SetEnvironmentVariable("TRACEROOT_HOST_URL", "https://your-traceroot.example.com", "User")
```

**Windows (PowerShell), current session only:**

```powershell
$env:TRACEROOT_API_KEY = "tr-..."
```

Restart Claude Code (or open a new terminal session) after setting the variables.
On Windows, restart your IDE (e.g. PyCharm, VS Code) too, since it reads environment variables at launch.

---

## Platform support

Session state is stored in `~/.claude/state/traceroot/<session_id>/` and guarded by a file lock
(`fcntl` on macOS/Linux, `msvcrt` on Windows). Windows support is new and lightly tested; please
open an issue if you hit locking problems.

---

## Privacy

When `TRACEROOT_API_KEY` is set, the plugin sends the following to `TRACEROOT_HOST_URL`:
prompts, assistant replies, thinking, and tool inputs/outputs (each truncated to
`TRACEROOT_PLUGIN_MAX_CHARS`, default 20000), plus git repo/ref, model, token counts,
and timings. No redaction is applied.

To send metadata only (model, token counts, timings, tool names), set
`TRACEROOT_PLUGIN_CAPTURE_CONTENT=false`. Prompts, replies, thinking, tool
inputs/outputs, and tool error messages are then omitted from exported spans.

If `TRACEROOT_API_KEY` is not set, nothing is sent.

---

## Troubleshooting

### `claude` is not recognized (Windows)

The installer places `claude.exe` in `%USERPROFILE%\.local\bin`, which may not be on your `PATH`.

```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";$env:USERPROFILE\.local\bin", "User")
```

Then fully close and reopen your terminal or IDE. To verify, run `claude --version`.
