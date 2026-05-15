# Coda Phase-0 Prototype

A minimal, single-file coding agent that validates the core ReAct loop against LiteLLM tool-use. This is a throwaway prototype — Phase 1 will reimplement everything behind proper Protocol contracts.

## Features

- **ReAct Agent Loop** — read → reason → act cycle with configurable max turns
- **Three Tools** — `read_file`, `write_file`, `run_shell`
- **Streaming Output** — token-by-token LLM response via LiteLLM
- **Human-in-the-loop Approval** — colored diff preview for writes, suspicious command highlighting for shell
- **Session-scoped Trust** — approve once, or trust for the entire session (`y` / `n` / `a`)
- **Interactive REPL** — multi-turn conversation mode, or single-shot via CLI argument
- **Model Agnostic** — works with any LiteLLM-supported provider (Anthropic, OpenAI, Gemini, …)

## Quick Start

```bash
# 1. Set your API key
export ANTHROPIC_API_KEY=sk-ant-...

# 2. Single-shot mode
uv run python prototype.py "explain what src/main.py does"

# 3. Interactive REPL mode
uv run python prototype.py
```

## Configuration

The agent reads `.coda/config.yaml` in the project root for the default model:

```yaml
provider: anthropic/claude-sonnet-4-5-20250929
```

Command-line flags and environment variables override this:

| Source | Flag / Env | Example |
|--------|-----------|---------|
| CLI flag | `--model` | `--model openai/gpt-4o` |
| Env var | `CODA_MODEL` | `export CODA_MODEL=openai/gpt-4o` |
| CLI flag | `--root` | `--root ~/projects/my-app` |
| Env var | `CODA_ROOT` | `export CODA_ROOT=~/projects/my-app` |
| CLI flag | `--max-turns` | `--max-turns 20` (default: 15) |

## How It Works

```
User prompt
    │
    ▼
┌─────────────┐
│  LLM (LLM)  │◄── stream tokens to terminal
└─────┬───────┘
      │ tool_call?
      ▼
┌──────────────┐    ┌─────────────────────────────┐
│ Execute Tool │◄──►│ read_file / write_file /     │
│              │    │ run_shell                    │
└──────┬───────┘    └─────────────────────────────┘
       │
       ▼
  Tool result → back to LLM (next turn)
```

Each turn the LLM can:
1. Respond with plain text (conversation ends)
2. Call one or more tools, whose results feed into the next turn

## Safety

| Mechanism | Detail |
|-----------|--------|
| Path sandboxing | All file access is resolved and verified to stay within the project root |
| Shell blocklist | `sudo`, `su`, `rm -rf /`, `mkfs`, fork bombs are blocked by policy |
| Suspicious highlighting | `git push`, `rm`, `curl`, etc. are flagged in red |
| Human approval | Every `write_file` and `run_shell` requires explicit user consent |
| Size limits | File reads capped at 50 KB, shell output at 20 KB |
| Shell timeout | Commands are killed after 60 seconds |

## Requirements

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) (package manager)
- An API key for your chosen LLM provider

## Project Structure

```
.
├── prototype.py      # The entire agent (~520 lines)
├── pyproject.toml    # Dependencies: litellm, rich, pyyaml
├── uv.lock
└── README.md
```

## License

Prototype / experimental — not intended for production use.
