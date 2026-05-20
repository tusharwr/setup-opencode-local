# OpenCode — Complete Setup Guide

> **Platforms:** Windows & macOS  
> **Last updated:** 2026-05-20

A step-by-step guide to install, configure, and use [OpenCode](https://opencode.ai) — the open-source AI coding agent. Covers CLI and Desktop app modes, free models, paid API keys, and real-world fixes for issues discovered during setup.

---

## Table of Contents

- [1. What is OpenCode?](#1-what-is-opencode)
- [2. Installation](#2-installation)
  - [Windows](#windows)
  - [macOS](#macos)
- [3. First Launch & Configuration](#3-first-launch--configuration)
- [4. Known Bug: `"client": "cli"` breaks the Desktop App](#4-known-bug-client-cli-breaks-the-desktop-app)
- [5. Plugin Install Error (Harmless)](#5-plugin-install-error-harmless)
- [6. OpenCode Free API Options](#6-opencode-free-api-options)
  - [OpenCode Zen API Endpoints](#opencode-zen-api-endpoints)
  - [Local Models via Ollama (Fully Free)](#local-models-via-ollama-fully-free)
- [7. Free Models: OpenCode Zen](#7-free-models-opencode-zen)
  - [How to connect Zen](#how-to-connect-zen)
  - [Available Free Models](#available-free-models)
  - [Set a Free Model as Default](#set-a-free-model-as-default)
- [8. Paid Models (Bring Your Own API Key)](#8-paid-models-bring-your-own-api-key)
- [9. Using CLI Mode](#9-using-cli-mode)
- [10. Using Desktop App Mode](#10-using-desktop-app-mode)
- [11. Troubleshooting](#11-troubleshooting)
  - [Windows-specific](#windows-specific)
  - [macOS-specific](#macos-specific)
  - [Both Platforms](#both-platforms)
- [12. Quick Reference](#12-quick-reference)

---

## 1. What is OpenCode?

OpenCode is an **open-source AI coding agent** that works in your terminal, desktop app, or IDE (VS Code, Cursor, Zed). It supports **75+ LLM providers**, has built-in LSP support, and runs any model — from free hosted models to paid API keys like Claude or GPT.

Key features:

- **Provider-agnostic** — use any model from any provider
- **Free Zen models** included (no API key needed beyond a free Zen account)
- **LSP enabled** — auto-loads language servers for better context
- **Multi-session** — run multiple agents in parallel
- **CLI + Desktop + IDE** — three interfaces, one config

---

## 2. Installation

OpenCode has two components you can install independently:

- **CLI / Terminal** — runs in the command line (TUI or non-interactive mode)
- **Desktop App (Beta)** — GUI app that runs a CLI sidecar in the background

### Windows

#### Option A: Desktop App (Recommended for GUI users)

1. Download the Windows installer from [opencode.ai/download](https://opencode.ai/download)
2. Run the `.exe` installer and follow the wizard
3. Launch OpenCode from the Start Menu

> **Prerequisite:** Microsoft Edge WebView2 Runtime is required. If you see a blank window, install it from [Microsoft's website](https://developer.microsoft.com/en-us/microsoft-edge/webview2/).

#### Option B: CLI via npm

```powershell
npm install -g opencode-ai
```

> **Note:** On Windows, npm may show `EPERM` errors about symlinks. Run the terminal **as Administrator** or enable Developer Mode in Windows Settings.

#### Option C: CLI via package managers

```powershell
# Chocolatey
choco install opencode

# Scoop
scoop bucket add extras
scoop install extras/opencode

# WinGet
winget install -e --id SST.opencode
```

#### Option D: CLI via install script (requires WSL/bash)

```bash
curl -fsSL https://opencode.ai/install | bash
```

---

### macOS

#### Option A: Desktop App (Recommended for GUI users)

1. Download the `.dmg` from [opencode.ai/download](https://opencode.ai/download) (choose Apple Silicon or Intel)
2. Open the DMG and drag OpenCode to Applications
3. Launch from Applications folder

> **Note:** macOS may block the app. Go to **System Settings → Privacy & Security** and click "Open Anyway."

#### Option B: CLI via Homebrew (Recommended)

```bash
brew install anomalyco/tap/opencode
```

#### Option C: CLI via npm

```bash
npm install -g opencode-ai
```

#### Option D: CLI via install script

```bash
curl -fsSL https://opencode.ai/install | bash
```

---

## 3. First Launch & Configuration

### Config File Location

OpenCode reads its configuration from a global JSON(C) file:

| Platform | Path |
|---|---|
| **Windows** | `%USERPROFILE%\.config\opencode\opencode.jsonc` |
| **macOS** | `~/.config/opencode/opencode.jsonc` |

Press `WIN+R` and paste `%USERPROFILE%\.config\opencode` to find it on Windows.

### Minimal Starting Config

```jsonc
{
  "$schema": "https://opencode.ai/config.json"
}
```

This is enough to start. Providers and models are configured interactively inside the app (see sections below).

---

## 4. Known Bug: `"client": "cli"` breaks the Desktop App

> **Platform:** Windows (confirmed) — potentially also macOS

### The Problem

If your config file contains the field `"client": "cli"`, the **Desktop app will fail** with:

> **"Choose an agent and model before sending a prompt."**

Even though the **CLI works perfectly fine** with the same config. This happens because:

1. The config file is **shared** between CLI and Desktop modes
2. Setting `"client": "cli"` forces the Desktop app's sidecar into CLI-only mode
3. The Desktop UI can't communicate properly with the sidecar to read your configured providers/models

### The Fix

**Step 1:** Remove the `"client": "cli"` line from your config.

**Before (broken):**
```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "client": "cli"     // <-- remove this
}
```

**After (fixed):**
```jsonc
{
  "$schema": "https://opencode.ai/config.json"
}
```

**Step 2:** Clear the OpenCode cache so it rebuilds without the bad config.

| Platform | Command |
|---|---|
| **Windows** | Delete `%USERPROFILE%\.cache\opencode` |
| **macOS** | Run `rm -rf ~/.cache/opencode` |

**Step 3:** Reset the Desktop app's saved storage.

| Platform | Path to delete |
|---|---|
| **Windows** | Delete everything inside `%APPDATA%\ai.opencode.desktop\` except the `opencode\` subfolder |
| **macOS** | Delete `~/Library/Application Support/ai.opencode.desktop/opencode.global.dat` and `opencode.settings` |

**Step 4:** Restart the Desktop app. Run `/connect` again to add your provider, and `/models` to select a model.

---

## 5. Plugin Install Error (Harmless)

> **Platform:** Windows & macOS

### The Problem

In the logs (`~/.local/share/opencode/log/`), you may see this warning:

```
WARN background dependency install failed
Cause: NpmInstallFailedError
cause: @opencode-ai/plugin: No matching version found for @opencode-ai/plugin@local.
```

This happens when OpenCode tries to auto-install plugins and encounters a reference to a non-existent local plugin. It's **harmless** — OpenCode continues to work normally.

### The Fix (Optional)

If the warning bothers you, check your config for a `plugin` key and remove any entries referencing `@opencode-ai/plugin` or `local` plugins.

---

## 6. OpenCode Free API Options

OpenCode offers several ways to use models for free:

### OpenCode Zen API Endpoints

Zen provides **REST API endpoints** you can call directly (not just through the OpenCode app). All endpoints use the same Zen API key.

| Model | Model ID | API Base URL | Auth |
|---|---|---|---|
| Big Pickle | `big-pickle` | `https://opencode.ai/zen/v1/chat/completions` | `x-api-key` header |
| DeepSeek V4 Flash Free | `deepseek-v4-flash-free` | `https://opencode.ai/zen/v1/chat/completions` | `x-api-key` header |
| MiniMax M2.5 Free | `minimax-m2.5-free` | `https://opencode.ai/zen/v1/chat/completions` | `x-api-key` header |
| Nemotron 3 Super Free | `nemotron-3-super-free` | `https://opencode.ai/zen/v1/chat/completions` | `x-api-key` header |
| All GPT models | `gpt-5.*` | `https://opencode.ai/zen/v1/responses` | `x-api-key` header |
| All Claude models | `claude-*` | `https://opencode.ai/zen/v1/messages` | `x-api-key` header |

List all available models: `GET https://opencode.ai/zen/v1/models`

> **Important:** Even free models require a Zen account with a payment method on file. Usage is billed at $0 for free models, but billing info is required.

### Local Models via Ollama (Fully Free)

For a **truly free, no-account, offline** option, run local models via [Ollama](https://ollama.com):

```bash
# Install Ollama, then pull a model
ollama pull qwen2.5-coder:7b

# Connect in OpenCode
# Run /connect in the TUI and select Ollama as the provider
```

This requires a machine with sufficient RAM/VRAM but has no API costs and works fully offline.

---

## 7. Free Models: OpenCode Zen

### How to Connect Zen

1. Go to **[opencode.ai/auth](https://opencode.ai/auth)** and sign in / create an account
2. Add billing details (required even for free models — but free models cost **$0**)
3. Copy your Zen API key
4. In OpenCode (CLI or Desktop), run the command:  
   **`/connect`**
5. Select **OpenCode Zen** from the provider list
6. Paste your API key
7. Run **`/models`** to see available models
8. Select a free model (look for "Free" in the name)

### Available Free Models

| Model | Model ID | SWE-bench | Context Window | Best For | Cost |
|---|---|---|---|---|---|
| **Big Pickle** | `big-pickle` | 72.0% | 200K | General coding, stealth model | Free |
| **DeepSeek V4 Flash Free** | `deepseek-v4-flash-free` | — | 128K | Fast general coding | Free |
| **MiniMax M2.5 Free** | `minimax-m2.5-free` | 80.2% | 200K | Strong coding, long context | Free |
| **Nemotron 3 Super Free** | `nemotron-3-super-free` | 52.0% | 1M | NVIDIA model, massive context | Free |

### Set a Free Model as Default

Add this to your config to skip the model selection step on startup:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/big-pickle"
}
```

> The format is always `opencode/<model-id>`. For example: `opencode/minimax-m2.5-free`.

---

## 8. Paid Models (Bring Your Own API Key)

If you have existing API keys for paid providers, you can use them in OpenCode.

### Using `/connect` (Recommended)

Just run `/connect` in the TUI, select your provider, and paste your API key. Supported providers include:

| Provider | Command | Environment Variable |
|---|---|---|
| **Anthropic (Claude)** | `/connect` → Anthropic | `ANTHROPIC_API_KEY` |
| **OpenAI (GPT)** | `/connect` → OpenAI | `OPENAI_API_KEY` |
| **Google Gemini** | `/connect` → Google | `GOOGLE_API_KEY` |
| **OpenRouter** | `/connect` → OpenRouter | `OPENROUTER_API_KEY` |
| **Azure OpenAI** | `/connect` → Azure | `AZURE_API_KEY` |
| **Together AI** | `/connect` → Together | `TOGETHER_API_KEY` |
| **Groq** | `/connect` → Groq | `GROQ_API_KEY` |

### Using Environment Variables

Set the environment variable in your shell profile (`~/.bashrc`, `~/.zshrc`, or via System Settings on Windows/macOS):

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

Then start OpenCode — it will auto-detect the credentials.

### Using Config File

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "anthropic": {
      "apiKey": "sk-ant-..."
    }
  },
  "model": "anthropic/claude-sonnet-4-6"
}
```

> **Security tip:** Never commit API keys to Git. Use environment variables instead of hardcoding them in config.

---

## 9. Using CLI Mode

The CLI (Terminal) mode runs OpenCode in your command prompt / terminal.

### Interactive TUI Mode

```bash
# Start in current directory
opencode

# Start with a specific model
opencode --model opencode/big-pickle

# Start with a specific agent
opencode --agent plan

# Continue last session
opencode --continue
```

### Non-Interactive (Scripting) Mode

```bash
# Run a single prompt
opencode run "Explain this codebase"

# Run with a model
opencode run --model opencode/big-pickle "Add error handling"

# Pipe input
echo "Find all unused variables" | opencode run
```

### Useful CLI Flags

| Flag | Description |
|---|---|
| `--model` | Specify the model to use |
| `--agent` | Specify the agent (build, plan, explore) |
| `--continue` / `-c` | Continue the last session |
| `--session` / `-s` | Continue a specific session by ID |
| `--dir` | Start in a specific directory |
| `--log-level DEBUG` | Show detailed debug logs |
| `--print-logs` | Print logs to stdout |

---

## 10. Using Desktop App Mode

### How it Works

The Desktop app runs a **CLI sidecar** (`opencode-cli serve`) in the background and connects to it from a GUI. This means:

- Config is shared between CLI and Desktop (same `opencode.jsonc`)
- Providers/models set in CLI are available in Desktop and vice versa
- The Desktop app auto-starts the sidecar when launched

### Common Desktop Issues

| Issue | Fix |
|---|---|
| **App shows "Connection Failed"** | Click the server name on home screen → **Clear** default server URL |
| **Blank / frozen window** | macOS: `OpenCode` menu → **Reload Webview** |
| **Blank window on Windows** | Install/update **Microsoft Edge WebView2 Runtime** |
| **"Choose an agent and model"** | Remove `"client": "cli"` from config (see [Section 4](#4-known-bug-client-cli-breaks-the-desktop-app)) |

---

## 11. Troubleshooting

### Windows-specific

| Problem | Solution |
|---|---|
| **Blank window on launch** | Install [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/en-us/microsoft-edge/webview2/) |
| **"Choose an agent and model" in Desktop app (CLI works)** | Remove `"client": "cli"` from config, clear cache and desktop storage ([see Section 4](#4-known-bug-client-cli-breaks-the-desktop-app)) |
| **npm install `EPERM` error** | Run terminal **as Administrator** or enable Developer Mode |
| **Desktop app runs slowly** | Use WSL2 for better performance — see [opencode.ai/docs/windows-wsl](https://opencode.ai/docs/windows-wsl) |
| **Can't find config file** | Press `WIN+R`, paste `%USERPROFILE%\.config\opencode` |
| **Can't find logs** | Press `WIN+R`, paste `%USERPROFILE%\.local\share\opencode\log` |

### macOS-specific

| Problem | Solution |
|---|---|
| **App blocked by macOS** | System Settings → Privacy & Security → "Open Anyway" |
| **Blank / frozen UI** | OpenCode menu → **Reload Webview** |
| **Can't find config file** | `~/.config/opencode/opencode.jsonc` |
| **Can't find logs** | `~/.local/share/opencode/log/` |
| **Orphan `opencode-cli` processes** | Run `pkill -9 -f opencode-cli` |

### Both Platforms

| Problem | Solution |
|---|---|
| **"Model not available" / `ProviderModelNotFoundError`** | Model IDs must follow `provider/model` format. Run `opencode models` to list available models |
| **`ProviderInitError`** | Corrupted config. Delete `~/.local/share/opencode` (or `%USERPROFILE%\.local\share\opencode` on Windows) and re-authenticate via `/connect` |
| **`AI_APICallError`** | Outdated provider packages. Clear cache at `~/.cache/opencode` (or `%USERPROFILE%\.cache\opencode`) and restart |
| **Plugin install warnings in logs** | Harmless — see [Section 5](#5-plugin-install-error-harmless) |
| **Need to reset everything** | 1. Delete config cache: `~/.cache/opencode` (or `%USERPROFILE%\.cache\opencode`)  
2. Delete data: `~/.local/share/opencode` (or `%USERPROFILE%\.local\share\opencode`)  
3. Delete desktop storage: `%APPDATA%\ai.opencode.desktop\*.dat` (Windows) or `~/Library/Application Support/ai.opencode.desktop/*.dat` (macOS)  
4. Restart and reconfigure |

---

## 12. Quick Reference

### Config File Template

Place this at `~/.config/opencode/opencode.jsonc` (macOS) or `%USERPROFILE%\.config\opencode\opencode.jsonc` (Windows):

```jsonc
{
  "$schema": "https://opencode.ai/config.json",

  // Default model (use "opencode/<model-id>" for Zen models)
  "model": "opencode/big-pickle",

  // Log level for debugging
  "logLevel": "INFO",

  // Auto-update behavior
  "autoupdate": "notify",

  // Disable to skip plugin loading
  // "plugin": []
}
```

### Essential Commands

| Command | What it does |
|---|---|
| `opencode` | Start interactive TUI mode |
| `opencode run "prompt"` | Run a single prompt non-interactively |
| `/connect` | Add a provider (Zen, Anthropic, OpenAI, etc.) |
| `/models` | List and select available models |
| `/agent` | Switch between agents (build, plan, explore) |
| `opencode models` | List all models from CLI |
| `opencode models --refresh` | Refresh the model cache |
| `opencode --log-level DEBUG` | Start with verbose debugging |

### Important File Paths

| What | Windows | macOS |
|---|---|---|
| Global config | `%USERPROFILE%\.config\opencode\opencode.jsonc` | `~/.config/opencode/opencode.jsonc` |
| Logs | `%USERPROFILE%\.local\share\opencode\log\` | `~/.local/share/opencode/log/` |
| Session data | `%USERPROFILE%\.local\share\opencode\project\` | `~/.local/share/opencode/project/` |
| Cache | `%USERPROFILE%\.cache\opencode\` | `~/.cache/opencode/` |
| Desktop storage | `%APPDATA%\ai.opencode.desktop\` | `~/Library/Application Support/ai.opencode.desktop/` |

---

> **Resources**
> - Official docs: [opencode.ai/docs](https://opencode.ai/docs)
> - Zen models: [opencode.ai/zen](https://opencode.ai/zen)
> - Download: [opencode.ai/download](https://opencode.ai/download)
> - GitHub: [github.com/anomalyco/opencode](https://github.com/anomalyco/opencode)
> - Discord: [opencode.ai/discord](https://opencode.ai/discord)
