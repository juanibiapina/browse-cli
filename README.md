# Browse

**The CLI for AI agents to control Chrome. Zero config, agent-agnostic, battle-tested.**

[![npm version](https://img.shields.io/npm/v/@juanibiapina/browse-cli?style=for-the-badge)](https://www.npmjs.com/package/@juanibiapina/browse-cli)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Linux-blue?style=for-the-badge)]()

```bash
browse go "https://example.com"
browse read
browse click e5
browse snap
```

## Why Browse

Browser automation for AI agents is harder than it looks. Most tools require complex setup, tie you to specific AI providers, or break on real-world pages.

Browse takes a different approach:

**Agent-Agnostic** - Pure CLI commands over Unix socket. Works with Claude Code, GPT, Gemini, Cursor, custom agents, shell scripts - anything that can run commands.

**Zero Config** - Install the extension, run commands. No relay processes, no subscriptions.

**Battle-Tested** - Built by reverse-engineering production browser extensions and methodically working through agent-hostile pages like Discord settings. Falls back gracefully when CDP fails.

**Smart Defaults** - Screenshots auto-resize to 1200px (saves tokens). Actions auto-capture screenshots (saves round-trips). Errors on restricted pages warn instead of fail.

**Network Capture** - Automatically logs all network requests while active. Filter, search, and replay API calls without manually setting up request interception.

## Installation

### Quick Start

```bash
# 1. Install globally
npm install -g @juanibiapina/browse-cli

# 2. Load extension in Chrome
#    - Open chrome://extensions
#    - Enable "Developer mode"
#    - Click "Load unpacked"
#    - Paste the path from: browse extension-path

# 3. Install native host (copy extension ID from chrome://extensions)
browse install <extension-id>

# 4. Restart Chrome and test
browse tab.list
```

### Multi-Browser Support

```bash
browse install <extension-id>                    # Chrome (default)
browse install <extension-id> --browser brave    # Brave
browse install <extension-id> --browser all      # All supported browsers
```

Supported: `chrome`, `chromium`, `brave`, `edge`, `arc`

### Uninstall

```bash
browse uninstall                  # Chrome only
browse uninstall --all            # All browsers + wrapper files
```

### Development Setup

```bash
git clone https://github.com/juanibiapina/browse-cli.git
cd browse-cli
npm install
npm run build
# Then load dist/ as unpacked extension
```

## Usage

```bash
browse <command> [args] [options]
browse --help                    # Basic help
browse --help-full               # All 50+ commands
browse <command> --help          # Command details
browse --find <query>            # Search commands
```

### Navigation

```bash
browse go "https://example.com"
browse back
browse forward
browse tab.reload --hard
```

### Reading Pages

```bash
browse read                           # Accessibility tree + visible text content
browse read --no-text                 # Accessibility tree only (no text)
browse read --depth 3                 # Limit tree depth (smaller output)
browse read --compact                 # Remove empty structural elements
browse read --depth 3 --compact       # Both (60% smaller output)
browse page.text                      # Raw text content only
browse page.state                     # Modals, loading state, scroll position
```

Element refs (`e1`, `e2`, `e3`...) are stable identifiers from the accessibility tree - semantic, predictable, and resilient to DOM changes.

**Note:** `browse read` returns only elements **visible in the current viewport** to minimize output size for LLMs. Refs remain valid for off-screen elements - you can still `click e5` even if `e5` isn't in the current output. Scroll to see more elements, or use `scroll.to --ref e5` to bring an element into view.

### Semantic Locators

Find and interact with elements by role, text, or label - no refs or selectors needed:

```bash
# By ARIA role
browse locate.role button --name "Submit"           # Find button
browse locate.role button --name "Submit" --action click  # Find and click
browse locate.role textbox --action fill --value "hello"  # Find and fill
browse locate.role link --all                       # List all links

# By text content  
browse locate.text "Sign In" --action click         # Click element with text
browse locate.text "Accept" --exact                 # Exact match only

# By form label
browse locate.label "Email" --action fill --value "test@example.com"
```

### Iframe Support

Work with content inside iframes:

```bash
browse frame.list                     # List all frames
browse frame.switch --index 0         # Switch to first iframe
browse frame.switch --name "payment"  # Switch by frame name
browse frame.switch --selector "#checkout-frame"  # Switch by CSS selector

# Now all commands target the iframe
browse read                           # Read iframe content
browse click e5                       # Click in iframe
browse locate.role button --action click

browse frame.main                     # Return to main page
```

### Interaction

```bash
browse click e5                       # Click by element ref
browse click --selector ".btn"        # Click by CSS selector
browse click 100 200                  # Click by coordinates
browse type "hello" --submit          # Type and press Enter
browse type "email@example.com" --ref e12  # Type into specific element
browse key Escape                     # Press key
browse scroll down                    # Scroll down
browse scroll down --amount 5         # Scroll down more
browse scroll.bottom                  # Scroll to bottom
```

### Forms

Select options in dropdown menus:

```bash
browse select e5 "US"                         # Select by value
browse select "#country" "US"                 # Select by CSS selector
browse select e5 "opt1" "opt2"                # Multi-select
browse select e5 --by label "United States"   # Select by visible text
browse select e5 --by index 0                 # Select first option
```

### Element Inspection

Get computed styles from elements:

```bash
browse element.styles e5              # Get styles by ref
browse element.styles ".header"       # Get styles by CSS selector (can return multiple)
```

Returns font, color, background, border, padding, and bounding box for design debugging.

### Screenshots

Screenshots auto-save to `/tmp` by default (optimized for AI agents):

```bash
browse screenshot                             # Auto-saves to /tmp/browse-snap-*.png
browse screenshot --output /tmp/shot.png      # Save to specific path
browse screenshot --full --output /tmp/hd.png # Full resolution (skip resize)
browse screenshot --annotate                  # With element labels
browse screenshot --fullpage                  # Entire page
browse screenshot --no-save                   # Return base64 + ID only (no file)
browse snap                                   # Alias for screenshot
```

To disable auto-save globally, set `autoSaveScreenshots: false` in `browse.json`.

Actions like `click`, `type`, and `scroll` automatically capture a screenshot after execution - no extra command needed.

### Tabs

```bash
browse tab.list
browse tab.new "https://example.com"
browse tab.switch 123
browse tab.close 123
browse tab.name "dashboard"           # Name current tab
browse tab.switch "dashboard"         # Switch by name
browse tab.group --name "Work" --color blue
```

### Window Isolation

Keep using your browser while the agent works in a separate window:

```bash
# Create isolated window for agent
browse window.new "https://example.com"
# Returns: Window 123456 (tab 789)

# All subsequent commands target that window
browse click e5 --window-id 123456
browse read --window-id 123456
browse tab.new "https://other.com" --window-id 123456

# Or manage windows directly
browse window.list                    # List all windows
browse window.list --tabs             # Include tab details
browse window.focus 123456            # Bring window to front
browse window.close 123456            # Close window
```

### Device Emulation

Test responsive designs and mobile layouts:

```bash
browse emulate.device --list                    # Show available devices
browse emulate.device "iPhone 14"               # Emulate iPhone 14
browse emulate.device "Pixel 7"                 # Emulate Pixel 7
browse emulate.device reset                     # Return to desktop

# Custom viewport
browse emulate.viewport --width 375 --height 812
browse emulate.viewport --width 1920 --height 1080 --scale 2

# Touch emulation
browse emulate.touch                            # Enable touch
browse emulate.touch --enabled false            # Disable touch
```

Available devices: iPhone 12-14 (Pro/Max), iPhone SE, iPad (Pro/Mini), Pixel 5-7 (Pro), Galaxy S21-S23, Galaxy Tab S7, Nest Hub (Max).

### Performance Tracing

Capture performance metrics and traces:

```bash
browse perf.metrics                   # Current performance metrics
browse perf.start                     # Start tracing
browse perf.stop                      # Stop and get trace data
```

### Waiting

```bash
browse wait 2                         # Wait 2 seconds
browse wait.element ".loaded"         # Wait for element
browse wait.network                   # Wait for network idle
browse wait.url "/dashboard"          # Wait for URL pattern
```

### Other

```bash
browse js "return document.title"     # Execute JavaScript
browse search "login"                 # Find text in page
browse cookie.list                    # List cookies
browse zoom 1.5                       # Set zoom to 150%
browse console                        # Read console messages
browse network                        # Read network requests
```

### Network Capture

Browse automatically captures all network requests while active. No explicit start needed.

```bash
# Overview (token-efficient for LLMs)
browse network                          # Recent requests, compact table
browse network --urls                   # Just URLs (minimal output)
browse network --format curl            # As curl commands

# Filtering
browse network --origin api.github.com  # Filter by origin/domain
browse network --method POST            # Only POST requests
browse network --type json              # Only JSON responses
browse network --status 4xx,5xx         # Only errors
browse network --since 5m               # Last 5 minutes
browse network --exclude-static         # Skip images/fonts/css/js

# Drill down
browse network.get r_001                # Full request/response details
browse network.body r_001               # Response body (for piping to jq)
browse network.curl r_001               # Generate curl command
browse network.origins                  # List captured domains

# Management
browse network.clear                    # Clear captured data
browse network.stats                    # Capture statistics
```

Storage location: `/tmp/browse/` (override with `--network-path` or `BROWSE_NETWORK_PATH` env).
Auto-cleanup: 24 hours TTL, 200MB max.

## Global Options

```bash
--tab-id <id>      # Target specific tab
--window-id <id>   # Target specific window (isolate agent from your browsing)
--json             # Output raw JSON
--soft-fail        # Warn instead of error (exit 0) on restricted pages
--no-screenshot    # Skip auto-screenshot after actions
--full             # Full resolution screenshots (skip resize)
--network-path <path>  # Custom path for network logs (default: /tmp/browse, or BROWSE_NETWORK_PATH env)
```

## Socket API

For programmatic integration, send JSON to `/tmp/browse.sock`:

```bash
echo '{"type":"tool_request","method":"execute_tool","params":{"tool":"tab.list","args":{}},"id":"1"}' | nc -U /tmp/browse.sock
```

### Protocol Reference

**Request:**
```json
{
  "type": "tool_request",
  "method": "execute_tool",
  "params": {
    "tool": "click",
    "args": { "ref": "e5" }
  },
  "id": "unique-request-id",
  "tabId": 123,
  "windowId": 456
}
```

**Success Response:**
```json
{
  "type": "tool_response",
  "id": "unique-request-id",
  "result": {
    "content": [{ "type": "text", "text": "Result message" }]
  }
}
```

**Error Response:**
```json
{
  "type": "tool_response",
  "id": "unique-request-id",
  "error": {
    "content": [{ "type": "text", "text": "Error message" }]
  }
}
```

## Command Groups

| Group | Commands |
|-------|----------|
| `window.*` | `new`, `list`, `focus`, `close`, `resize` |
| `tab.*` | `list`, `new`, `switch`, `close`, `name`, `unname`, `named`, `group`, `ungroup`, `groups`, `reload` |
| `scroll.*` | `top`, `bottom`, `to`, `info` |
| `page.*` | `read`, `text`, `state` |
| `locate.*` | `role`, `text`, `label` |
| `element.*` | `styles` |
| `frame.*` | `list`, `switch`, `main`, `js` |
| `wait.*` | `element`, `network`, `url`, `dom`, `load` |
| `cookie.*` | `list`, `get`, `set`, `clear` |
| `bookmark.*` | `add`, `remove`, `list` |
| `history.*` | `list`, `search` |
| `dialog.*` | `accept`, `dismiss`, `info` |
| `emulate.*` | `network`, `cpu`, `geo`, `device`, `viewport`, `touch` |
| `perf.*` | `start`, `stop`, `metrics` |
| `network.*` | `get`, `body`, `curl`, `origins`, `clear`, `stats`, `export`, `path` |

## Aliases

| Alias | Command |
|-------|---------|
| `snap` | `screenshot` |
| `read` | `page.read` |
| `find` | `search` |
| `go` | `navigate` |

## How It Works

```
CLI (browse) -> Unix Socket -> Native Host -> Chrome Extension -> CDP/Scripting API
```

Browse uses Chrome DevTools Protocol for most operations, with automatic fallback to `chrome.scripting` API when CDP is unavailable (restricted pages, certain contexts). Screenshots fall back to `captureVisibleTab` when CDP capture fails.

## Limitations

- Cannot automate `chrome://` pages or the Chrome Web Store (Chrome restriction)
- First CDP operation on a new tab takes ~100-500ms (debugger attachment)
- Some operations on restricted pages return warnings instead of results

## Linux Support (Experimental)

Browse should work on Linux with Chromium. Not yet tested in production.

```bash
# Install dependencies
sudo apt install chromium-browser nodejs npm imagemagick

# For headless server: add Xvfb + VNC
sudo apt install xvfb tigervnc-standalone-server

# Install Browse and native host
npm install -g @juanibiapina/browse-cli
browse install <extension-id> --browser chromium
```

**Notes:**
- Use Chromium (no official Chrome for Linux ARM64)
- Screenshot resize uses ImageMagick instead of macOS `sips`
- Headless servers need Xvfb + VNC for initial login setup

## AI Agent Integration

Browse includes a skill file for AI coding agents like [Pi](https://github.com/badlogic/pi-mono):

```bash
# Symlink for auto-updates
ln -s "$(pwd)/skills/browse" ~/.pi/agent/skills/browse

# Or copy
cp -r skills/browse ~/.pi/agent/skills/
```

See [`skills/README.md`](skills/README.md) for details.

## Development

```bash
npm run dev       # Watch mode
npm run build     # Production build
```

After changes:
- **Extension** (`src/`): Reload at `chrome://extensions`
- **Host** (`native/`): Restart `node native/host.cjs`

## Acknowledgments

Browse is a fork of [surf-cli](https://github.com/nicobailon/surf-cli) by Nico Bailon. This fork is independently maintained and not affiliated with or endorsed by the original project.

## License

MIT
