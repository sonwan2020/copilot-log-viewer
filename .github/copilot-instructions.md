# Copilot Instructions

## Project Overview

Zero-dependency, client-side web application for visualizing Copilot Adapter log files (JSONL format). Runs entirely in the browser — no server, no bundler, no build step.

## Development

Open `index.html` directly in a browser. No build, no package manager, no tests. Deployed via GitHub Pages: push to `main` triggers `deploy-pages.yml`.

## Architecture

Three ES modules loaded via `<script type="module">`, plus vendored highlight.js (loaded as a global script):

- **`js/app.js`** — Orchestrator: owns the centralized `state` object, file I/O (drag-drop + streaming/legacy FileReader), event delegation, theme toggle, search/filter logic, search match navigation.
- **`js/parser.js`** — Data processing: JSONL parsing (streaming via ReadableStream + FileReader fallback), SSE response parsing, content normalization. All pure/exported functions.
- **`js/renderer.js`** — DOM generation: renders 6 tabs (Messages, System, Tools, Request, Response, Raw), custom markdown engine, syntax highlighting via `hljs` global, interactive elements.

**Data flow:** File drop → `parseLogFile[Streaming]()` → `state.entries` (each entry gets `_index`) → `applyFilters()` → `renderEntryList()` + `render*Tab()`.

## Critical Conventions

### Strict Content Security Policy

The CSP is `default-src 'none'; style-src 'self'; script-src 'self'; img-src 'none'; connect-src 'none'`. This means:
- No inline styles or scripts — all dynamic content via DOM APIs
- No external resources of any kind
- User text must always be set via `.textContent`; use `escapeHtml()` for any `.innerHTML` usage
- highlight.js is vendored locally, guarded with `typeof hljs === 'undefined'` check

### Lazy DOM Rendering

Performance-critical pattern used throughout `renderer.js`. Message bodies, tool result content, system prompts, and thinking blocks are NOT rendered until first expand. Each uses a `bodyRendered` flag to avoid re-rendering. **New content must follow this pattern** — render expensive DOM trees on first toggle, not on initial page paint.

### Plain Text Toggle Pattern

Use `createLazyToggleWrapper(text)` from `renderer.js`. Starts as plain text `<pre>`, builds markdown view lazily on first toggle. Uses `.md-toggle-wrapper` / `.md-toggle-btn` / `.plain-text-view` CSS classes.

### Tools Caching

Tools arrays are deduplicated using SHA-256 hashing (Web Crypto API) in `parser.js`. Entries store a `_toolsCacheId` reference instead of the full tools array. Both `app.js` and `renderer.js` have a `getTools(entry)` helper that resolves cached or inline tools transparently.

### Tool Use/Result Linking

Messages tab builds a `toolUseMap` (`Map<id, {name, input}>`) from `tool_use` blocks, so `tool_result` blocks can display linked tool name and input.

### Inline JSON Links

Request tab uses `createLinkedJsonView()` with `__LINK_*__` placeholder substitution to embed clickable links inside JSON text that open content viewers or switch tabs.

### CSS Theming

All colors use CSS custom properties in `:root` (light) and `[data-theme="dark"]`. Theme persists in `localStorage('logs-reviewer-theme')`. Highlight.js theme CSS is swapped by toggling `disabled` on `<link>` elements. Responsive breakpoint at 768px.

## Log File Format

JSONL: one JSON object per line. Each entry contains:
- `timestamp`, `streaming`
- `anthropicRequest` (model, messages, system, tools, max_tokens, temperature)
- `openaiRequest` (OpenAI-format equivalent)
- `copilotResponse` (SSE text: `data: {...}\n` lines)
