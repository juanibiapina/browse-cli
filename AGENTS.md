# surf-cli Development Guide

## Add CLI Command

1. Add to `TOOLS` in `native/cli.cjs` (args, opts, examples)
2. Add handler in `src/service-worker/index.ts` `handleMessage()` switch
3. CDP op? Add method in `src/cdp/controller.ts`
4. DOM interaction? Add handler in `src/content/accessibility-tree.ts`

## Add CDP Operation

1. Add method to `CDPController` class in `src/cdp/controller.ts`
2. Use `this.send(tabId, "Domain.method", params)`
3. Handle events in `handleCDPEvent()` if needed

## Core Files

**src/service-worker/index.ts** (~2500L) - Central message router, CDP ops, screenshot cache, tab registry
- Message types: EXECUTE_CLICK, EXECUTE_TYPE, READ_PAGE, EXECUTE_SCREENSHOT
- Screenshot cache: generateScreenshotId(), cacheScreenshot(), getScreenshot()
- Tab names: tabNameRegistry Map<name,tabId>

**src/cdp/controller.ts** (~1000L) - CDP wrapper, CDPController class
- Mouse: click(), rightClick(), doubleClick(), hover(), drag()
- Keyboard: type(), pressKey(), pressKeyChord()
- Screenshots: captureScreenshot(), captureRegion()
- Network/Console: enableNetworkTracking(), getNetworkRequests(), getResponseBody(), subscribeToNetwork(), enableConsoleTracking(), getConsoleMessages()
- Emulation: emulateNetwork(), emulateCPU(), emulateGeolocation()

**src/content/accessibility-tree.ts** (~1900L) - Content script, generates a11y tree YAML, element interactions
- Handlers: GENERATE_ACCESSIBILITY_TREE, CLICK_ELEMENT, FORM_INPUT, GET_ELEMENT_COORDINATES, WAIT_FOR_ELEMENT, WAIT_FOR_URL
- Element refs: e1, e2... in window.__piRefs for stable references

**native/cli.cjs** (~2100L) - CLI parser, socket client
- TOOLS: command definitions with args/opts
- ALIASES: shortcut to command map
- AUTO_SCREENSHOT_TOOLS: commands that auto-capture
- parseArgs(), sendRequest()

**native/host.cjs** (~1500L) - Socket server, AI integration
- handleToolRequest(): main dispatcher
- queueAiRequest(): AI request serialization
- AI clients: chatgptClient, geminiClient, perplexityClient, grokClient

**native/host-helpers.cjs** - Tool mapping and formatting
- mapToolToMessage(): tool to extension message converter
- mapComputerAction(): computer tool action handler
- formatToolContent(): response formatting

**src/native/port-manager.ts** - Extension to native messaging, auto-reconnect, request/response tracking

## Debug

- Extension logs: chrome://extensions -> Service Worker -> Console
- Host logs: `/tmp/surf-host.log`
- CLI raw output: `--json` flag

## Gotchas

- CDP attach takes 100-500ms first time per tab
- chrome:// pages are restricted (no automation possible)
- Content script needs page refresh on existing tabs after extension reload
- Screenshots auto-resize to 1200px unless `--full` flag is used
