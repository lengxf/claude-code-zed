# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Rust workspace that integrates Claude Code CLI with the Zed editor. Two crates:

- **claude-code-extension** -- Zed extension (Rust compiled to WASM via `wasm32-wasip2`). Manages LSP server lifecycle, downloads platform-specific server binaries from GitHub releases.
- **claude-code-server** -- Native Rust binary bridging Zed (LSP over stdin/stdout) and Claude Code CLI (WebSocket + MCP protocol). Runs in `hybrid` mode by default (LSP + WebSocket concurrently, connected via Tokio broadcast channel).

Communication flow: `Zed <-> LSP (stdin/stdout) <-> claude-code-server <-> WebSocket <-> Claude Code CLI`

## Build Commands

```bash
cargo build                    # Build entire workspace
cargo build --release -p claude-code-server  # Release build for server only
cargo fmt                      # Format code
cargo clippy                   # Lint
```

### Makefile targets (server development)

```bash
make dev-build    # Release build server, copy to Zed extension dir
make dev-debug    # Debug build server, copy to Zed extension dir
make dev-test     # Build, deploy, and print test instructions
make status       # Show deployment status
make dev-clean    # Remove deployed binary
```

### Extension development

Install via Zed: `Cmd+Shift+P` -> "zed: install dev extension" -> select `claude-code-extension/` folder. Zed handles the WASM build automatically. Use `eprintln!()` for debug logging (appears in Zed's debug panel with `[EXTENSION]` prefix).

## Key Architecture Details

- **Server modes** (`main.rs`): `lsp`, `websocket`, or `hybrid` (default). Hybrid runs both concurrently in one process.
- **LSP server** (`lsp.rs`): Built on `tower-lsp`. Handles document sync, code actions, completions (e.g., `@claude explain`), selection tracking. Includes UTF-16 to UTF-8 position conversion via `char_pos_to_byte_pos()`.
- **WebSocket** (`websocket.rs`): Listens on localhost (default port 59792). Writes lock files to `~/.claude/ide/[port].lock` for Claude Code CLI discovery. Negotiates MCP protocol via `Sec-WebSocket-Protocol: mcp` header.
- **MCP handler** (`mcp.rs`): JSON-RPC handler implementing tools like `getCurrentSelection`, `getWorkspaceFolders`, `openFile`, `getDiagnostics`, `saveDocument`, etc.
- **Development mode** (`lib.rs`): `FORCE_DEVELOPMENT_MODE` constant (default `false`). Auto-detects dev mode when worktree path contains `claude-code-zed`.

## Testing

No automated tests. Manual testing workflow:
1. Install extension via Zed dev extension
2. Open a supported file in Zed
3. Run Claude Code CLI, use `/ide` to connect
4. Test selection sharing by selecting text

## Git Commit Convention

- Use emoji first to indicate commit type:
  - 🎉 `:tada:` - Initial commit or major feature
  - ✨ `:sparkles:` - New feature
  - 🐛 `:bug:` - Bug fix
  - 🔧 `:wrench:` - Configuration changes
  - 📝 `:memo:` - Documentation
  - 🚀 `:rocket:` - Performance improvements
  - 🎨 `:art:` - Code style/formatting
  - ♻️ `:recycle:` - Refactoring
  - 🔥 `:fire:` - Remove code/files
  - 📦 `:package:` - Add dependencies/submodules
