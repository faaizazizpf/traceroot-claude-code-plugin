# traceroot-claude-code-plugin

Observability plugin for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that sends per-turn traces to [TraceRoot](https://traceroot.ai).

Each conversation turn becomes a trace in TraceRoot showing the LLM call, tool invocations, token usage, and any nested sub-agents — all grouped by session.

---

## Installation

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

---

## Configuration

Set the following environment variables (or add them to your shell profile):

| Variable | Required | Description |
|---|---|---|
| `TRACEROOT_API_KEY` | Yes | Your TraceRoot API key. Find it at [app.traceroot.ai](https://app.traceroot.ai) → Settings → API Keys. |
| `TRACEROOT_HOST_URL` | No | TraceRoot ingest endpoint. Defaults to the hosted service. Set this only when self-hosting. |
| `TRACEROOT_PLUGIN_CAPTURE_CONTENT` | No | Defaults to `true`. Set to `false` (or `0`, `no`, `off`) to send metadata only: no prompts, replies, thinking, or tool inputs/outputs. |

Example (add to `~/.zshrc` or `~/.bashrc`):

```bash
export TRACEROOT_API_KEY="tr-..."
# Optional: only needed for self-hosted deployments
# export TRACEROOT_HOST_URL="https://your-traceroot.example.com"
```

Restart Claude Code (or open a new terminal session) after setting the variables.

---

## Privacy

When `TRACEROOT_API_KEY` is set, the plugin sends to `TRACEROOT_HOST_URL`:
prompts, assistant replies, thinking, and tool inputs/outputs (each truncated to
`TRACEROOT_PLUGIN_MAX_CHARS`, default 20000), plus git repo/ref, model, token counts and timings.
No redaction is applied.

To send metadata only, set `TRACEROOT_PLUGIN_CAPTURE_CONTENT=false`. Model, token counts,
timings, tool names, session id and git repo/ref are still sent; conversation content and
tool error messages are not.

If `TRACEROOT_API_KEY` is not set, the plugin sends nothing.
