# Claude Code Telegram (Rust)

A Rust implementation of the Claude Code Telegram integration for smaller binaries and faster startup.

## Features

- **Permission request notifications** via Telegram with inline keyboards
- **Always Allow** feature to auto-approve trusted tools
- **Job completion notifications** when Claude Code finishes
- **Multi-machine support** with hostname display

## Binary Size

~4 MB (vs ~50 MB for Python scie version)

## Building

```bash
# Development build
cargo build

# Release build (optimized)
cargo build --release

# Run tests
cargo test
```

## Usage

```bash
# Permission request handler (used by PermissionRequest hooks)
claude-code-telegram hook

# Job completion notifications (used by Stop hooks)
claude-code-telegram stop

# Long-running Telegram bot (/start, /help, /status)
claude-code-telegram bot
```

## Configuration

Create `~/.claude/telegram_hook.json`:

```json
{
  "telegram_bot_token": "your_bot_token",
  "telegram_chat_id": "your_chat_id"
}
```

## Claude Code Hook Configuration

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PermissionRequest": [{
      "matcher": {"tools": ["Bash", "Edit", "Write"]},
      "hooks": [{"type": "command", "command": "claude-code-telegram hook"}]
    }],
    "Stop": [{
      "matcher": {},
      "hooks": [{"type": "command", "command": "claude-code-telegram stop"}]
    }]
  }
}
```

## Cross-Compilation Targets

- `x86_64-unknown-linux-musl` (Linux x86_64, static)
- `x86_64-apple-darwin` (macOS Intel)
- `aarch64-apple-darwin` (macOS Apple Silicon)
