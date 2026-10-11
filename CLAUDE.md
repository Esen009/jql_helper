# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About this project

A single-file browser tool (`index.html`) that assembles Jira JQL queries from dropdowns. No build step, no dependencies, no server.

## Running it

Open `index.html` directly in a browser. There is nothing to install or build.

## Architecture

Everything lives in `index.html`, divided into three sequential sections:

**CSS** — colours are defined as CSS custom properties on `:root` and overridden in a `prefers-color-scheme: dark` media query. All other rules reference those variables, so theme changes are one-line edits.

**HTML** — the dropdown options are plain `<option>` elements. Adding or changing a filter choice means editing the relevant `<option>` block. Dropdown `id` attributes are the contract between HTML and JavaScript.

**JavaScript** — four responsibilities:
- `buildQuery()` reads each dropdown's `.value` and assembles a JQL string. Simple fields (project, issuetype, priority) are quoted in the output (`project = "ABC"`); fields whose options already contain complete JQL snippets (status, assignee, date) are inserted as-is.
- `render()` is the single update function — it rebuilds the query display and sets button disabled states. Call it after any state change.
- localStorage is used for two independent keys: `jqlHelperSite` (the Jira base URL) and `jqlHelperSaved` (the saved-query list as a JSON array). Every localStorage access is wrapped in try/catch to handle private-browsing environments where it throws.
- Saved queries store the full dropdown state as a plain object `{ name, state: { project, issuetype, … } }`. `setState()` restores dropdowns from that object and calls `render()`. `renderSavedList()` rebuilds the list DOM; it uses `esc()` to HTML-escape names before injecting them via `innerHTML`.

The `dropdowns` array (`["project", "issuetype", "status", "priority", "assignee", "date", "sort"]`) is the canonical list of filter ids. It drives change-event wiring, reset, `getState()`, and `setState()` — keep it in sync if dropdowns are added or removed.
