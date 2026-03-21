# Manifest fields (MV3)

Reference for common manifest fields when using Addfox. Align with [Chrome Extension API / manifest](https://developer.chrome.com/docs/extensions/reference/manifest/) and WebExtensions.

## Required / common

- **manifest_version**: Use `3` (MV3). Addfox defaults to MV3 for Chromium; Firefox can use 2 or 3 per target.
- **name**: Extension name (short).
- **version**: Semver string (e.g. `"1.0.0"`).
- **description**: Optional but recommended for store listing.

## Background

- **background.service_worker** (MV3): Path to service worker script. Addfox injects `background/index.js` when using reserved `background` entry.
- **background.scripts** (MV2, e.g. Firefox): Array of script paths. Use in `manifest.firefox` when targeting MV2.

## UI surfaces

- **action**: Toolbar icon and popup. `default_popup`: path to HTML (e.g. `popup/index.html`). `default_icon`: optional icons.
- **options_ui**: Options page. `page`: path (e.g. `options/index.html`).
- **side_panel** (Chrome): Side panel. `default_path`: path (e.g. `sidepanel/index.html`).
- **sidebar_action** (Firefox): Use in `manifest.firefox` for sidebar.
- **devtools_page**: DevTools tab. Path (e.g. `devtools/index.html`).

## Content scripts

- **content_scripts**: Array of `{ matches, js?, css?, run_at?, all_frames?, ... }`.
  - For the built-in content entry, do not write source paths; the framework fills the built js/css paths. Declare `matches` and other options as needed.
  - **matches**: e.g. `["<all_urls>"]` or specific patterns; minimize scope when possible.

## Permissions

- **permissions**: List of permission strings (e.g. `storage`, `activeTab`, `scripting`).
- **host_permissions** (MV3): Optional host patterns; use only when needed. Prefer specific patterns over `<all_urls>`.

See [permissions.md](permissions.md) for usage scenarios.

## Icons and assets

- **icons**: Optional; typically 16, 48, 128. Paths relative to extension root (output dir).
- **web_accessible_resources**: If content script or page needs to load extension resources by URL, list them here.

## Other

- **host_permissions**, **optional_permissions**: See Chrome manifest docs.
- **content_security_policy**: Addfox/Rsbuild output is MV3-compliant; override only if required.
