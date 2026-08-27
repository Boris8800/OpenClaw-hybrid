<div align="center">

# OpenClaw Hybrid + MemPalace

**A single-file, cross-platform AI agent gateway with persistent memory**

macOS · Linux · Windows &nbsp;|&nbsp; Any local model server + any online API (OpenAI-compatible or Anthropic)

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/macOS%20%26%20Windows-ready-2ea44f)
![License](https://img.shields.io/badge/license-MIT-blue)

</div>

---

## What is it?

One file (`openclaw-one.sh`) is a hybrid agent gateway that:

- **Runs entirely locally** with any OpenAI-compatible model server (Ollama, LM Studio, vLLM, llama.cpp, GPT4All, etc.; default `qwen2.5-coder:14b`) for implementation and tools.
- **Optionally uses online models** as advisory supervisors/planners (OpenAI, Anthropic, DeepSeek, or any other OpenAI-compatible/Anthropic endpoint) — they plan and review, but never write files or run commands.
- **Remembers** — everything meaningful is stored in a local **MemPalace** (SQLite), searchable by relevance.
- **Gives the agent safe local tools** — file ops, terminal (policy-gated), web search, memory, diagnostics — behind a strict role/safety model.

The same file runs on **macOS/Linux** (`./openclaw-one.sh`) and **Windows** (`python openclaw-one.sh` or `openclaw-one.bat`) thanks to a sh+Python polyglot header.

---

## Quick start

```bash
# macOS / Linux
chmod +x openclaw-one.sh
./openclaw-one.sh install          # install checklist + safe auto-fix
./openclaw-one.sh chat "Hello"
./openclaw-one.sh repl

# Windows (PowerShell / CMD)
python openclaw-one.sh install
openclaw-one.bat chat "Hello"
```

**macOS users:** double-click **OpenClaw Installer.app** to run the install checklist in a Terminal window (right-click → *Open* the first time if macOS blocks it).

---

## Installation checklist

`install` (aliases: `doctor`, `setup`) runs a 10-step checklist and **auto-fixes safe issues** (creates the home folder, rebuilds a corrupt database, starts `ollama serve`, ensures data files and working roots exist).

> The local model server is **any OpenAI-compatible endpoint** — point `LOCAL_URL` at whatever server you use. Ollama is only the default/example; the auto-fix and troubleshooting steps below assume Ollama, but LM Studio, vLLM, llama.cpp, and GPT4All work identically.

```bash
./openclaw-one.sh doctor            # report only
./openclaw-one.sh doctor --auto     # also apply safe fixes
./openclaw-one.sh doctor --json     # machine-readable output
./openclaw-one.sh doctor --report   # write doctor-report.md
ALLOW_INSTALL=1 ./openclaw-one.sh install   # also install Homebrew packages (poppler, ollama)
```

### Prerequisites

| Requirement | Purpose | How to install |
| --- | --- | --- |
| Python 3.8+ | Runtime | [python.org](https://www.python.org) |
| A local model server | Local worker backend | Any OpenAI-compatible server — e.g. Ollama (`brew install ollama` then `ollama serve`), LM Studio, vLLM, llama.cpp |
| Local model | Default worker model | Pull it on your chosen server (Ollama: `ollama pull qwen2.5-coder:14b`) |
| Online API keys *(optional)* | Supervisors | Set `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, and/or `DEEPSEEK_API_KEY`, or configure any endpoint via `OPENCLAW_ONLINE_PROVIDERS` |
| `pdftotext` *(optional)* | PDF ingest | `brew install poppler` |

---

## Configuration

All configuration is via environment variables (safe defaults shown).

| Variable | Default | Description |
| --- | --- | --- |
| `OPENCLAW_HOME` | `~/openclaw` | Where data lives (`mempalace.sqlite3`, logs, state) |
| `LOCAL_MODEL` | `qwen2.5-coder:14b` | Local worker model |
| `LOCAL_URL` / `OLLAMA_BASE_URL` | `http://localhost:11434/v1` | Local OpenAI-compatible endpoint (Ollama, LM Studio, vLLM, llama.cpp, …) |
| `LOCAL_KEY` | — | Optional key for a local server that requires one |
| `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `DEEPSEEK_API_KEY` | — | Built-in online supervisors |
| `OPENCLAW_ONLINE_PROVIDERS` | `[]` | JSON list of extra OpenAI-/Anthropic-compatible models (see below) |
| `SUPERVISOR_ENABLED` | `1` | Turn online review on/off |
| `LOCAL_TERMINAL_POLICY` | `worker` | `safe` · `worker` · `unrestricted` |
| `LOCAL_WRITE_ENABLED` | `1` | Allow the agent to write files |
| `OPENCLAW_ALLOWED_ROOTS` | home + cwd | Paths the agent may touch |
| `SUPERVISOR_DAILY_CALLS` | `5` | Daily online-supervisor call budget |
| `ALLOW_INSTALL` | `0` | Permit `install` to run Homebrew installs |

View live config with `./openclaw-one.sh config` or `./openclaw-one.sh system`.

### Adding any online provider

Every model speaks the standard **OpenAI `/chat/completions`** (or **Anthropic `/messages`**) protocol, so any vendor can be wired in without code changes — keyed or keyless. Set `OPENCLAW_ONLINE_PROVIDERS` to a JSON array:

```bash
# Keyed provider (e.g. a gateway, OpenRouter, Groq, Together, a self-hosted server)
export OPENCLAW_ONLINE_PROVIDERS='[{
  "model": "provider-model-id",
  "base_url": "https://gateway.example.com/v1",
  "api_key_env": "MY_API_KEY",
  "provider": "myvendor",
  "kind": "openai_compatible",
  "weight": 4,
  "context_window": 32768
}]'

# Keyless local/LAN server (LM Studio, vLLM, llama.cpp on your network)
export OPENCLAW_ONLINE_PROVIDERS='[{
  "model": "local-hosted-model",
  "base_url": "http://192.168.1.50:8000/v1",
  "no_auth": true,
  "kind": "openai_compatible"
}]'
```

- `kind` is `openai_compatible` (default) or `anthropic`.
- Omit `api_key_env`/`api_key` and set `no_auth: true` for servers that need no authentication.

---

## Usage examples

```bash
# Agent mode — inspect, edit, test locally
./openclaw-one.sh agent "refactor src/main.py and run the tests"

# Chat
./openclaw-one.sh chat "Explain this repo"
./openclaw-one.sh repl                # interactive loop (type /exit)

# Memory (MemPalace)
./openclaw-one.sh memory add "Deployment uses azd up" --category notes --tags deploy
./openclaw-one.sh memory search "deploy"
./openclaw-one.sh memory stats
./openclaw-one.sh memory export --path backup.json
./openclaw-one.sh memory prune --dry-run

# To-dos
./openclaw-one.sh todo add "Ship v1.4" --priority 2
./openclaw-one.sh todo list
./openclaw-one.sh todo done <id>

# Time utilities
./openclaw-one.sh calendar --year 2026 --month 8
./openclaw-one.sh timer 5m
./openclaw-one.sh stopwatch

# Web & research
./openclaw-one.sh search "OpenClaw docs"
./openclaw-one.sh info "What is OpenClaw?"
./openclaw-one.sh ingest ./report.pdf

# Health & maintenance
./openclaw-one.sh system
./openclaw-one.sh report --path report.html
./openclaw-one.sh backup
./openclaw-one.sh monitor            # live dashboard (macOS)
```

---

## Agent tools

The local worker can call these tools through a policy-gated dispatcher (see `./openclaw-one.sh tools`):

**Files & text:** `list_files`, `read_file`, `write_file`, `delete_file`, `move_file`, `grep_files`, `replace_in_file`, `line_range`, `hex_dump`, `base64_tool`, `sort_unique`, `count_occurrences`, `json_format`, `json_yaml`, `file_checksum`, `text_stats`, `trash_file`, `find_files`, `rename_files`, `dir_size`, `touch_file`, `dedupe_files`, `package_files`

**Terminal:** `run_command` (allowlisted), `terminal` (policy-gated, safe by default)

**Web & network:** `web_search`, `fetch_url`, `download_file`, `fetch_json`, `url_links`, `http_status`, `dns_resolve`, `tls_cert`, `port_probe`

**Memory:** `memory_search`, `memory_add`, `ingest_file`, `memory_growth`, `activity_summary`, `sql_query` (read-only)

**Productivity:** `todo_add`, `todo_list`, `todo_done`, `note_append`, `reminder`, `calendar_view`, `current_time`, `uuid`

**System:** `system_info`, `env_summary`, `model_capabilities`, `task_state`, `doctor_check`, `html_report`

Run any tool directly: `./openclaw-one.sh tool grep_files --args '{"pattern":"TODO","path":"."}'`

---

## Safety model

- **Roles are separated:** the online supervisor can only *plan* and *review*; it can never read/write repositories, run commands, or implement code (enforced + logged).
- **The local worker is the only implementation authority**, and even it is gated:
  - `terminal` uses a bin allowlist, blocks dangerous patterns (`rm -rf`, `sudo`, shell metacharacters, etc.) and a `safe`/`worker`/`unrestricted` policy ladder.
  - `write`/`delete`/`move`/`trash`/`rename` require the `execute_commands` role.
  - All file access is confined to `OPENCLAW_ALLOWED_ROOTS`.
- **URLs are validated** — private/loopback/internal addresses are blocked unless explicitly allowed.
- **Web content is treated as untrusted data**, never as instructions.
- **Search/ingest results are logged** for auditability (`activity.jsonl`).

---

## Project layout

```
openclaw-one.sh            # the entire program (polyglot shell+python)
openclaw-one.bat           # Windows launcher
OpenClaw Installer.app     # macOS double-click installer
README.md
```

Data written to `OPENCLAW_HOME` (default `~/openclaw`):
`mempalace.sqlite3` (memories + cache), `activity.jsonl` (audit log), `health.json`, `task_state.json`, `learning_state.json`, `todos.json`.

---

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Local server connection refused | Start your model server (Ollama: `ollama serve`), then `./openclaw-one.sh doctor --auto` |
| Model not found | Pull it on your server (Ollama: `ollama pull qwen2.5-coder:14b`); for other servers make sure `LOCAL_MODEL` matches a served model id |
| Use a non-Ollama server | Point `LOCAL_URL` at it (e.g. `http://localhost:8000/v1`) and set `LOCAL_MODEL` to a served model |
| Keyless online server not used | Add it to `OPENCLAW_ONLINE_PROVIDERS` with `no_auth: true` |
| PDF ingest fails | `brew install poppler` |
| Supervisor skipped / budget | Online daily call/token budget hit; check `system` |
| Agent can't write files | Set `LOCAL_WRITE_ENABLED=1`, ensure path is in `OPENCLAW_ALLOWED_ROOTS` |
| macOS "unidentified developer" on the app | Right-click → **Open** once |

For anything else, run `./openclaw-one.sh doctor` and include its output.

---

## License

MIT
