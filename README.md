# circuit

**Write your thinking down. Circuit works on it while you're not.**

Most AI tools forget you the moment the conversation ends. Circuit doesn't. Every note you write — a question, a principle, a why-we-do-this — accumulates into context that quietly shapes every response you get back.

The further you get, the more it knows. The more it knows, the less you have to re-explain.

---

## Install

**macOS**
```sh
brew install mmadhugiri/circuit/circuit
```

**Linux**
```sh
# amd64
curl -L https://github.com/mmadhugiri/circuit-dist/releases/latest/download/circuit-linux-amd64 -o circuit
chmod +x circuit
sudo mv circuit /usr/local/bin/

# arm64
curl -L https://github.com/mmadhugiri/circuit-dist/releases/latest/download/circuit-linux-arm64 -o circuit
chmod +x circuit
sudo mv circuit /usr/local/bin/
```

Or download any binary directly from the [releases page](https://github.com/mmadhugiri/circuit-dist/releases).

---

## Quickstart

```sh
# One-time setup — creates .env and working directories
circuit init

# Write a note, get a response immediately
circuit -n "I keep procrastinating on the hard thing first thing in the morning"

# Headless mode — runs automatically in the background
circuit daemon

# UI mode — dashboard at http://localhost:4242
circuit serve
```

---

## How it works

You write things down. Circuit responds — with context from everything you've written before, what's happening in the world right now, and patterns it has noticed over time.

It runs on a heartbeat. You don't ask. It works.

Over time, daily notes consolidate into weekly patterns, weekly patterns into monthly arcs. The responses get sharper. The repetition drops away.

---

## Commands

| Command | What it does |
|---------|-------------|
| `circuit init` | Interactive setup — creates `.env` and directories |
| `circuit -n "..."` | Write a note, get a response immediately |
| `circuit daemon` | Headless mode — runs automatically, no UI |
| `circuit serve` | Dashboard at `http://localhost:4242` — runs automatically |
| `circuit beat` | One manual heartbeat, for cron or scripting |
| `circuit status` | Show what's running and recent activity |
| `circuit --version` | Print version |

---

## Providers

Set `CIRCUIT_PROVIDER` in `.env`. The default is `claude-cli` — uses your Claude subscription, no API key needed.

| Provider | Value | Key needed |
|----------|-------|------------|
| Claude CLI | `claude-cli` | — (uses your subscription) |
| Anthropic API | `anthropic` | `ANTHROPIC_API_KEY` |
| OpenAI | `openai` | `OPENAI_API_KEY` |
| xAI | `xai` | `XAI_API_KEY` |
| Gemini | `gemini` | `GEMINI_API_KEY` |
| Ollama (local) | `ollama` | — |

---

## License

MIT
