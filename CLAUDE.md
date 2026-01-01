# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Slash Search is a browser extension that focuses the search box when pressing '/' on any website, similar to Google's native behavior. It uses Manifest V3 and runs as a content script on all URLs except sites with native '/' support (GitHub, Twitter, Gmail, YouTube, etc.).

## Development

**No build step required.** Edit files directly and reload the extension:
- Chrome/Edge: `chrome://extensions` → Enable Developer mode → Load unpacked
- Firefox: `about:debugging#/runtime/this-firefox` → Load Temporary Add-on

## Architecture

The entire extension is a single content script (`src/content.js`) with three layers:

1. **Site Adapters (`SITE_ADAPTERS` array)**: Site-specific selectors for reliable detection. Each adapter has:
   - `test(hostname)`: Returns true if this adapter handles the site
   - `selectors[]`: CSS selectors to try in order, OR
   - `find()`: Custom function for complex cases (e.g., clicking to reveal hidden search, shadow DOM traversal)

2. **Generic Heuristics (`candidateSelectors` + `findGeneric()`)**: Fallback for sites without adapters. Uses semantic selectors (`[role='search']`, `input[type='search']`) and scoring to pick the best match. Includes international support (Chinese/Japanese/Korean search terms).

3. **Shadow DOM Support (`queryAllDeep()`, `pickFirstVisibleDeep()`)**: Traverses open shadow roots for web component-based sites.

## Adding a Site Adapter

Add new adapters to `SITE_ADAPTERS` in `src/content.js`:

```javascript
// Simple case: just selectors
{
  test: (h) => /(^|\.)example\.com$/i.test(h),
  selectors: ["#search-input", "input[name='q']"]
}

// Complex case: custom find function
{
  test: (h) => /(^|\.)example\.com$/i.test(h),
  find: () => {
    // Click to reveal hidden search, then return input
    const opener = document.querySelector("button.search-toggle");
    if (opener) opener.click();
    return document.querySelector("input#search");
  }
}
```

## Key Functions

- `isVisible(el)`: Checks display, visibility, opacity, dimensions, and disabled/hidden attributes
- `isSearchyInput(el)`: Validates input type (excludes password fields)
- `scoreCandidate(el)`: Ranks inputs by semantic indicators, position, and size
- `focusAndSelect(el)`: Focuses input without scroll jump, selects existing text

## Excluded Sites

Sites in `manifest.json` `exclude_matches` have native '/' support and are ignored by this extension.
