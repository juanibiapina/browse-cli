# Architecture

surf-cli is a browser automation system that enables programmatic control of Chrome through a command-line interface. It's designed for AI agents and developers who need to interact with web pages.

## System Components

The system consists of three main components that communicate through well-defined protocols:

```
┌─────────────┐     Unix Socket      ┌──────────────┐    Native Messaging    ┌──────────────────┐
│             │    /tmp/surf.sock    │              │    (Chrome protocol)   │                  │
│     CLI     │ ──────────────────── │  Native Host │ ────────────────────── │ Chrome Extension │
│  (cli.cjs)  │                      │  (host.cjs)  │                        │ (service-worker) │
│             │ <─────────────────── │              │ <───────────────────── │                  │
└─────────────┘      JSON response   └──────────────┘                        └────────┬─────────┘
                                                                                      │
                                                                             ┌────────┴────────┐
                                                                             │  CDP Controller │
                                                                             │  Content Scripts│
                                                                             └─────────────────┘
```

### CLI (`native/cli.cjs`)

The command-line interface that users and AI agents interact with directly.

- Parses commands and arguments
- Connects to the native host via Unix socket
- Formats and displays responses
- Supports command aliases (e.g., `snap` → `screenshot`, `go` → `navigate`)

### Native Host (`native/host.cjs`)

A Node.js process that bridges the CLI and Chrome extension.

- Listens on Unix socket `/tmp/surf.sock`
- Translates CLI commands to extension messages via `mapToolToMessage()`
- Communicates with Chrome using the native messaging protocol

### Chrome Extension (`src/`)

A Manifest V3 extension that performs the actual browser automation.

- **Service Worker** (`src/service-worker/`) - Central message router
- **CDP Controller** (`src/cdp/`) - Chrome DevTools Protocol wrapper
- **Content Scripts** (`src/content/`) - DOM interaction and accessibility tree
- **Port Manager** (`src/native/`) - Native messaging connection

## Communication Flow

### Request Flow

1. User runs CLI command: `surf click e5`
2. CLI connects to socket and sends JSON request
3. Native host transforms request via `mapToolToMessage()`
4. Message sent to extension via native messaging (length-prefixed JSON)
5. Service worker routes message to appropriate handler
6. CDP controller or content script executes action
7. Response flows back through the same path

### Message Protocol

All communication uses JSON messages with a consistent structure:

```json
{
  "type": "click",
  "ref": "e5"
}
```

Responses include success/error status and relevant data.

## Key Concepts

### Element References

The accessibility tree content script generates stable element identifiers:

- Elements are labeled `e1`, `e2`, `e3`, etc.
- References persist until page navigation
- Used for clicking, typing, and other interactions
- Run `surf read` to see the current accessibility tree with refs

### Dual Execution Paths

Browser operations can execute via two mechanisms:

1. **CDP (Chrome DevTools Protocol)** - Low-level browser control
   - Used for: navigation, screenshots, network interception, JavaScript execution
   - More powerful but requires debugger attachment

2. **Content Scripts** - JavaScript injected into pages
   - Used for: accessibility tree, click simulation, form filling
   - Works without debugger, handles dynamic content

The system automatically falls back from CDP to content scripts when needed.

### Network Capture

The extension automatically captures network requests:

- Stored in `~/.surf/network/` with timestamps
- Accessible via `surf network` commands
- Useful for debugging and monitoring API calls

## Directory Structure

```
surf-cli/
├── native/                 # Node.js native host (CommonJS)
│   ├── cli.cjs             # CLI entry point
│   ├── host.cjs            # Native host server
│   ├── protocol.cjs        # Message encoding
│   └── host-helpers.cjs    # Tool-to-message mapping
│
├── src/                    # Chrome extension (TypeScript)
│   ├── service-worker/     # Background script, message routing
│   ├── cdp/                # Chrome DevTools Protocol controller
│   ├── content/            # Content scripts (a11y tree, indicators)
│   ├── native/             # Native messaging port management
│   └── utils/              # Shared utilities
│
├── dist/                   # Built extension (load in Chrome)
├── scripts/                # Install/uninstall native host
├── skills/                 # AI agent skill definitions
└── test/                   # Test suite (vitest)
```

## Technology Stack

| Layer | Technology |
|-------|------------|
| CLI Runtime | Node.js (CommonJS) |
| Extension | Chrome MV3, TypeScript |
| Browser Control | Chrome DevTools Protocol |
| IPC | Unix Sockets, Native Messaging |
| Build | Vite, TypeScript |
| Testing | Vitest |

## Related Documentation

- `README.md` - User guide with all commands
- `.claude/skills/surf-codebase/SKILL.md` - AI agent quick reference
- `skills/README.md` - AI skill integration guide
