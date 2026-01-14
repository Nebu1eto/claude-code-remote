# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Claude Code hooks integration with Telegram Bot, available in both Python and Rust versions:
- **Python version** (`claude_code_telegram/`): Full-featured, ~50MB binary
- **Rust version** (`rust/`): Lightweight, ~4MB binary

Features:
- Intercept Claude Code permission requests via hooks
- Send notifications to users via Telegram (includes hostname for multi-machine setups)
- Receive user decisions (approve/deny/always allow) through Telegram inline keyboards
- Respond back to Claude Code with the user's decision

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Claude Code    │────▶│  Hook Handler    │────▶│  Telegram Bot   │
│  (Permission    │     │  (Python)        │     │  API            │
│   Request)      │     │                  │     │                 │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                │                        │
                                │                        ▼
                                │                 ┌─────────────────┐
                                │                 │  User Device    │
                                │                 │  (Telegram App) │
                                │                 └─────────────────┘
                                │                        │
                                ▼                        ▼
                        ┌──────────────────────────────────────────┐
                        │  Decision Handler (receives callback)    │
                        │  Returns: allow/deny to Claude Code      │
                        └──────────────────────────────────────────┘
```

## Package Structure

```
claude_code_telegram/
├── __init__.py       # Package exports
├── always_allow.py   # Always-allow preference manager
├── config.py         # Configuration management
├── hook_handler.py   # Claude Code hook script
└── bot.py            # Telegram bot commands
```

## Claude Code Hook Integration

Claude Code hooks are configured in `~/.claude/settings.json` or project's `.claude/settings.json`:
```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": {
          "tools": ["Bash", "Edit", "Write"]
        },
        "hooks": [
          {
            "type": "command",
            "command": "claude-code-telegram hook"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": {},
        "hooks": [
          {
            "type": "command",
            "command": "claude-code-telegram stop"
          }
        ]
      }
    ]
  }
}
```

The hook script receives JSON via stdin with the permission request details and must output a JSON response.

## Development Commands

### Python

```bash
# Install package in development mode
uv sync --all-extras

# Run CLI commands
claude-code-telegram hook   # Permission request handler
claude-code-telegram stop   # Job completion handler
claude-code-telegram bot    # Long-running Telegram bot

# Run tests
make test

# Run single test
.venv/bin/python -m pytest tests/test_file.py::test_function -v

# Type checking
make lint

# Format code
make format

# Build self-executable binary (requires: pip install pex)
make build-scie        # ~50MB, bundled Python
make build-scie-lazy   # ~5MB, fetches Python on first run
```

### Rust

```bash
cd rust

# Build release binary (~4MB)
cargo build --release

# Run tests
cargo test

# Run CLI commands
./target/release/claude-code-telegram hook
./target/release/claude-code-telegram stop
./target/release/claude-code-telegram bot
```

## Configuration

The hook loads configuration in this priority order:
1. JSON file at `~/.claude/telegram_hook.json` (recommended)
2. Environment variables (fallback)

### JSON Configuration (Recommended)

Create `~/.claude/telegram_hook.json`:
```json
{
  "telegram_bot_token": "your_bot_token",
  "telegram_chat_id": "your_chat_id"
}
```

### Environment Variables (Fallback)

Store in `~/.claude/.env` (never commit):
- `TELEGRAM_BOT_TOKEN`: Bot token from @BotFather
- `TELEGRAM_CHAT_ID`: Target chat ID for notifications

### Data Files

- `~/.claude/always_allow.json`: Stores always-allow tool preferences

## Dependencies

- `python-telegram-bot`: Telegram Bot API wrapper
- `python-dotenv`: Environment variable management
- `pytest`: Testing framework
- `mypy`: Static type checking
- `ruff`: Linting and formatting
- `pex`: Build self-executable binaries (dev only)
