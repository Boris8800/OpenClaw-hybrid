# OpenClaw Hybrid + MemPalace

A single-file, cross-platform agent gateway with persistent memory (MemPalace). Runs locally with Ollama and can use optional online supervisors (OpenAI / Anthropic / DeepSeek).

## Quick start

```bash
# macOS
chmod +x openclaw-one.sh
./openclaw-one.sh install          # install checklist + auto-fix
./openclaw-one.sh chat "Hello"
./openclaw-one.sh repl

# Windows
python openclaw-one.sh install
openclaw-one.bat chat "Hello"
```

You can also double-click the **OpenClaw Installer.app** (macOS) to run the install checklist in a Terminal window.

## Prerequisites

- Python 3.8+
- Ollama (local model): `ollama serve` then `ollama pull qwen2.5-coder:14b`
- Optional online supervisors: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `DEEPSEEK_API_KEY`
- Optional PDF ingest (macOS): `brew install poppler`

## Commands

| Command | Description |
| --- | --- |
| `doctor` / `install` / `setup` | Install checklist + auto-troubleshoot (`--auto`, `--json`, `--report`) |
| `agent "<goal>"` | Run the general-purpose hybrid agent |
| `chat "<text>"` | One-off chat |
| `repl` | Interactive chat loop |
| `memory add/search/list/...` | MemPalace memory ops (add, search, list, forget, stats, export, prune, dedupe) |
| `calendar` / `clock` / `timer` / `stopwatch` | Time utilities |
| `todo add/list/done` | To-do list stored locally |
| `report` | Generate an HTML status report |
| `system` / `version` | System health report / version |
| `backup` | Archive the database + data files |
| `monitor` | Live browser activity dashboard (macOS) |
| `search` / `info` / `ingest` | Web search, page info, file/image ingest |
| `tool <name> --args {...}` | Run any registered agent tool directly |

See `doctor` (or `openclaw-one.sh --help`) for the full tool and command list.
