# Project Context

- **Owner:** Songbo Wang
- **Project:** Copilot-Log-Reviewer — a zero-dependency, client-side web app for visualizing Copilot Adapter log files (JSONL). Runs entirely in the browser with strict CSP.
- **Stack:** Vanilla JS (ES modules), HTML, CSS. No bundler, no server. Three core modules: app.js (orchestrator), parser.js (data processing), renderer.js (DOM generation).
- **Created:** 2026-04-23

## Learnings

<!-- Append new learnings below. Each entry is something lasting about the project. -->

### 2026-04-23 — LRU Cache Implementation (PR #44, issue #18)

Implemented an LRU cache class in `js/parser.js` to optimize memory usage for the tools cache. The cache now has a 200-item limit and evicts the oldest 10% (20 items) when full. The implementation uses JavaScript Map's insertion order guarantee to track recency — on each `get()`, the key is deleted and re-inserted to move it to most recent position. The `LRUCache` class is exported for potential reuse in future features and provides a drop-in replacement for the plain Map API (`get`, `set`, `has`, `clear`, `size`). All existing cache consumers in `app.js` and `renderer.js` work without modification because they use the abstraction layer (`getToolsFromCache()`).
