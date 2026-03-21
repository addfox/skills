# Addfox Reference

Detailed config, CLI, manifest behavior, and cross-platform notes for AI use.

---

## Config file and appDir

- **Config**: `addfox.config.ts`, `addfox.config.js`, or `addfox.config.mjs` at project root. Export default from `defineConfig` (from `addfox`).
- **appDir**: Default `"app"`. Entry paths and manifest auto-load are relative to appDir.

### Config Structure

```ts
// addfox.config.ts
import { defineConfig } from 'addfox';
import { pluginReact } from '@rsbuild/plugin-react';

const manifest = {
  manifest_version: 3,
  name: 'My Extension',
  version: '1.0.0',
  permissions: ['storage', 'activeTab']
};

export default defineConfig({
  // App directory (default: "app")
  appDir: 'app',
  
  // Output directory name under .addfox (default: "extension")
  outDir: 'extension',
  
  // Manifest configuration
  manifest: {
    chromium: manifest,
    firefox: { ...manifest }
  },
  
  // Rsbuild plugins
  plugins: [pluginReact()],
  
  // Override/extend Rsbuild config (optional)
  rsbuild: {
    // Object: deep-merged with base config
    // Or function: (base, helpers) => helpers.merge(base, overrides)
  }
});
```

### Config Fields

| Field | Type | Description |
|-------|------|-------------|
| `manifest` | `object \| { chromium, firefox }` | Extension manifest. Single object or split by browser. Can also be paths: `{ chromium: 'manifest.json' }`. |
| `plugins` | `RsbuildPlugin[]` | Rsbuild plugins array. Use function calls like `pluginReact()` or `vue()`. |
| `rsbuild` | `object \| function` | Override/extend Rsbuild config. Object merges deeply; function provides `(base, helpers)` for full control. |
| `entry` | `Record<string, EntryConfigValue>` | Custom entries. Key = name, value = path relative to appDir. Omit to auto-discover. Set to `false` to disable framework entry handling. |
| `appDir` | `string` | App directory. Default `"app"`. |
| `outDir` | `string` | Output directory under `.addfox`. Default `"extension"`. |
| `browserPath` | `BrowserPathConfig` | Browser executable paths for dev mode. `{ chrome, firefox, edge, brave, ... }` |
| `zip` | `boolean` | Create zip output. Default `true`. |
| `envPrefix` | `string[]` | Env var prefixes to expose. Default `['']` exposes all. Use `['PUBLIC_']` for safety. |
| `cache` | `boolean` | Cache chromium user data dir. Default `true`. |
| `hotReload` | `object \| boolean` | Hot reload options: `{ port?: number, autoRefreshContentPage?: boolean }`. Set to `false` to disable. |
| `debug` | `boolean` | Enable error monitor in dev. Default `false`. |
| `report` | `boolean \| object` | Enable Rsdoctor report. Default `false`. |

---

## Entry

### Discovery

When `entry` is omitted, framework scans appDir for directories/files matching reserved names.

### Reserved Names

Do not rename these entries:

| Entry | Output Path | HTML | Description |
|-------|-------------|------|-------------|
| `background` | `background/index.js` | ❌ | Service worker (MV3) |
| `content` | `content/index.js` | ❌ | Content script(s) |
| `popup` | `popup/index.html` | ✅ | Toolbar popup |
| `options` | `options/index.html` | ✅ | Options page |
| `sidepanel` | `sidepanel/index.html` | ✅ | Chrome side panel |
| `devtools` | `devtools/index.html` | ✅ | DevTools page |
| `offscreen` | `offscreen/index.html` | ✅ | Offscreen document |
| `sandbox` | `sandbox/index.html` | ✅ | Sandbox page |
| `newtab` | `newtab/index.html` | ✅ | New Tab override |
| `bookmarks` | `bookmarks/index.html` | ✅ | Bookmarks override |
| `history` | `history/index.html` | ✅ | History override |

### Entry Config Value

```ts
type EntryConfigValue = 
  | string                                    // Path relative to appDir
  | { src: string; html?: boolean | string }; // With HTML control

// Examples:
entry: {
  'my-script': 'scripts/custom.ts',           // Script only
  'my-page': {                                // With HTML
    src: 'pages/custom.ts',
    html: true                                // Auto-generate HTML
  },
  'my-page-with-template': {
    src: 'pages/custom.ts',
    html: 'templates/custom.html'             // Use template
  }
}
```

### HTML Templates

For popup/options/etc., either:
1. **Omit HTML** — framework generates it automatically
2. **Provide template** with entry marker:

```html
<!-- app/popup/index.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Popup</title>
</head>
<body>
  <div id="root"></div>
  <script data-addfox-entry src="./index.tsx"></script>
</body>
</html>
```

---

## Manifest

### Built-in Entries

**Do not write source file paths for built-in entries in the manifest.** The framework injects the correct output paths.

```ts
// ✅ Correct - framework fills paths
const manifest = {
  manifest_version: 3,
  background: { service_worker: 'background/index.js' },
  content_scripts: [{ matches: ['<all_urls>'], js: ['content/index.js'] }],
  action: { default_popup: 'popup/index.html' },
  options_ui: { page: 'options/index.html' },
  side_panel: { default_path: 'sidepanel/index.html' },
  devtools_page: 'devtools/index.html'
};
```

### Browser Split

Use `chromium` and `firefox` branches for browser-specific fields:

```ts
const baseManifest = {
  manifest_version: 3,
  name: 'My Extension',
  version: '1.0.0'
};

export default defineConfig({
  manifest: {
    chromium: {
      ...baseManifest,
      minimum_chrome_version: '88'
    },
    firefox: {
      ...baseManifest,
      browser_specific_settings: {
        gecko: {
          id: 'myextension@example.com',
          strict_min_version: '109.0'
        }
      }
    }
  }
});
```

Build target via CLI:
```bash
addfox dev -b chrome
addfox build -b firefox
```

### Manifest Path Config

You can also specify manifest file paths:

```ts
export default defineConfig({
  manifest: {
    chromium: 'manifest/manifest.json',      // Relative to appDir
    firefox: 'manifest/manifest.firefox.json'
  }
});
```

Auto-discovery (if `manifest` omitted): looks for `manifest.json`, `manifest.chromium.json`, `manifest.firefox.json` in appDir.

### Common Manifest Fields

```ts
const manifest = {
  manifest_version: 3,
  name: 'Extension Name',
  version: '1.0.0',
  description: 'Extension description',
  icons: {
    '16': 'icons/icon16.png',
    '48': 'icons/icon48.png',
    '128': 'icons/icon128.png'
  },
  
  // Permissions
  permissions: [
    'storage',
    'activeTab',
    'scripting',
    'alarms',
    'notifications',
    'contextMenus',
    'offscreen'
  ],
  host_permissions: ['*://*.example.com/*'],
  optional_permissions: ['tabs', '<all_urls>'],
  
  // Entry points (paths filled by framework)
  action: {
    default_popup: 'popup/index.html',
    default_icon: { '16': 'icons/icon16.png' }
  },
  background: {
    service_worker: 'background/index.js'
  },
  options_ui: {
    page: 'options/index.html',
    open_in_tab: true
  },
  side_panel: {
    default_path: 'sidepanel/index.html'
  },
  devtools_page: 'devtools/index.html',
  
  // Content scripts
  content_scripts: [{
    matches: ['<all_urls>'],
    js: ['content/index.js'],
    css: ['content/styles.css'],
    run_at: 'document_idle',
    all_frames: false
  }],
  
  // Web accessible resources
  web_accessible_resources: [{
    resources: ['assets/*', 'injected.js'],
    matches: ['<all_urls>']
  }]
};
```

---

## CLI

### Commands

| Command | Purpose | Options |
|---------|---------|---------|
| `addfox dev` | Watch + HMR | `-b, --browser <browser>`, `-c, --cache`, `-p, --persist`, `--debug`, `-r, --report` |
| `addfox build` | Production build | `-b, --browser <browser>`, `-r, --report` |

### Browser Options

`-b, --browser` accepts: `chromium`, `chrome`, `firefox`, `edge`, `brave`, `vivaldi`, `opera`, `arc`, `yandex`, `browseros`, `santa`, `custom`

### Output

Default locations:
- Output root: `.addfox/`
- Built extension: `.addfox/extension/` (or `outDir` if customized)
- Zip package: `.addfox/extension.zip`
- Error log: `.addfox/error.md` (when errors occur with `--debug`)
- Build metadata: `.addfox/meta.md`
- Rsdoctor report: `.addfox/report/` (with `-r` flag)

### Examples

```bash
# Development with Chrome
addfox dev -b chrome

# Development with Firefox, no cache
addfox dev -b firefox --no-cache

# Production build for Chrome
addfox build -b chrome

# Build with bundle analysis
addfox build -r

# Debug mode with error monitoring
addfox dev --debug
```

---

## Cross-platform

### webextension-polyfill (Strongly Recommended)

**强烈推荐**使用 `webextension-polyfill` 实现跨浏览器兼容：

```bash
npm install webextension-polyfill
npm install -D @types/webextension-polyfill
```

```ts
import browser from 'webextension-polyfill';

// Works on both Chrome and Firefox
browser.runtime.sendMessage({ type: 'ping' });
browser.storage.local.get('settings');
browser.tabs.query({ active: true });
```

### Browser Differences

| Feature | Chrome | Firefox |
|---------|--------|---------|
| Manifest | MV3 | MV2/MV3 |
| Background | `service_worker` | `scripts` (MV2) / `service_worker` (MV3) |
| Action | `chrome.action` | `browser.browserAction` |
| Storage | `chrome.storage` | `browser.storage` |

### Testing Both Platforms

```bash
# Chrome
addfox dev -b chrome
addfox build -b chrome

# Firefox
addfox dev -b firefox
addfox build -b firefox
```

---

## Framework Plugins

### Supported Frameworks

Install corresponding plugin:

```bash
# React
npm install -D @rsbuild/plugin-react

# Vue
npm install -D @addfox/rsbuild-plugin-vue

# Preact
npm install -D @rsbuild/plugin-preact

# Svelte
npm install -D @rsbuild/plugin-svelte

# Solid
npm install -D @rsbuild/plugin-solid
```

### Configuration

```ts
import { pluginReact } from '@rsbuild/plugin-react';
import vue from '@addfox/rsbuild-plugin-vue';

export default defineConfig({
  plugins: [pluginReact()]  // or vue(), pluginPreact(), pluginSvelte(), pluginSolid()
});
```

---

## Styles

### Tailwind CSS v4

```bash
npm install -D tailwindcss @tailwindcss/postcss postcss
```

```js
// postcss.config.mjs
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

```css
/* app/popup/index.css */
@import 'tailwindcss';
```

### Tailwind CSS v3

```bash
npm install -D tailwindcss postcss autoprefixer
```

```js
// postcss.config.mjs
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

```js
// tailwind.config.js
export default {
  content: ['./app/**/*.{js,ts,jsx,tsx}'],
  theme: { extend: {} },
  plugins: [],
};
```

### UnoCSS

```bash
npm install -D unocss @unocss/postcss
```

```js
// postcss.config.mjs
export default {
  plugins: {
    '@unocss/postcss': {},
  },
};
```

### Sass/Less

Rsbuild has built-in support:

```bash
npm install -D sass
```

```scss
// app/popup/index.scss
$primary: #3b82f6;
.button { background: $primary; }
```

### CSS Scoping

Content scripts should use scoped styles:

```css
.my-extension-button { ... }
.my-extension-popup { ... }
```

---

## Content UI (Injecting DOM in Pages)

Use **`@addfox/utils`** helpers instead of manual shadow root creation.

### Three Isolation Levels

| Helper | Wrapper | Isolation | Best For |
|--------|---------|-----------|----------|
| `defineShadowContentUI` | Shadow DOM | Style only | Most content UI |
| `defineIframeContentUI` | iframe | Full (JS+CSS) | Full isolation from page |
| `defineContentUI` | Plain element | None | No isolation needed |

### Usage

```ts
import { defineShadowContentUI } from '@addfox/utils';

const mountUI = defineShadowContentUI({
  name: 'my-content-ui',
  target: 'body',
  attr: {
    style: 'position:fixed;bottom:16px;right:16px;z-index:2147483647;'
  },
  injectMode: 'append'
});

function init() {
  const root = mountUI();
  createRoot(root).render(<App />);
}

if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', init);
} else {
  init();
}
```

See [rules/content-ui.md](rules/content-ui.md) for detailed API reference.

---

## Rsbuild Configuration

### Object Form (Deep Merge)

```ts
export default defineConfig({
  rsbuild: {
    source: {
      alias: {
        '@': './app',
      },
      define: {
        'process.env.API_URL': JSON.stringify('https://api.example.com')
      }
    },
    output: {
      copy: [
        { from: './public', to: '.' }
      ],
      assetPrefix: './'
    }
  }
});
```

### Function Form (Full Control)

```ts
export default defineConfig({
  rsbuild: (base, helpers) => {
    return helpers.merge(base, {
      source: {
        define: {
          __VERSION__: JSON.stringify('1.0.0')
        }
      }
    });
  }
});
```

### Common Options

| Option | Description |
|--------|-------------|
| `source.alias` | Path aliases |
| `source.define` | Global constants |
| `output.copy` | Static asset copying |
| `output.assetPrefix` | Asset URL prefix |
| `server.port` | Dev server port |
| `dev.hmr` | HMR configuration |

---

## Lifecycle Hooks

Configure `hooks` in `defineConfig`:

| Hook | When |
|------|------|
| `afterCliParsed` | After CLI args parsed |
| `afterConfigLoaded` | After config and entries resolved |
| `beforeRsbuildConfig` | Before Rsbuild config built |
| `beforeBuild` | Before build runs |
| `afterBuild` | After build finishes (build only) |

```ts
export default defineConfig({
  hooks: {
    beforeBuild: (ctx) => {
      console.log('Building for', ctx.browser);
    },
    afterBuild: (ctx) => {
      console.log('Build completed');
    }
  }
});
```

---

## Feature Implementation

When implementing specific extension features, combine Addfox with guidance from extension-functions-best-practices:

| Feature Category | Addfox Setup | Detailed Implementation |
|-----------------|-------------|------------------------|
| **Video Enhancement** | `app/content/` entry | [extension-functions-best-practices/rules/video.md](../extension-functions-best-practices/rules/video.md) |
| **Video Download** | `entry` + background | [extension-functions-best-practices/rules/video.md](../extension-functions-best-practices/rules/video.md) |
| **AI Sidebar** | `app/sidepanel/` entry | [extension-functions-best-practices/rules/ai.md](../extension-functions-best-practices/rules/ai.md) |
| **Screenshot** | `activeTab` permission | [extension-functions-best-practices/rules/image.md](../extension-functions-best-practices/rules/image.md) |
| **Page Translation** | `app/content/` entry | [extension-functions-best-practices/rules/translation.md](../extension-functions-best-practices/rules/translation.md) |
| **Password Manager** | `storage` + `scripting` | [extension-functions-best-practices/rules/password-manager.md](../extension-functions-best-practices/rules/password-manager.md) |
| **Web3 Wallet** | `content` + `offscreen` | [extension-functions-best-practices/rules/web3.md](../extension-functions-best-practices/rules/web3.md) |

### Example: AI Sidebar Extension

```ts
// addfox.config.ts
import { defineConfig } from 'addfox';
import { pluginReact } from '@rsbuild/plugin-react';

const manifest = {
  manifest_version: 3,
  name: 'AI Assistant',
  permissions: ['sidePanel', 'activeTab', 'storage', 'scripting']
};

export default defineConfig({
  manifest: { chromium: manifest, firefox: { ...manifest } },
  plugins: [pluginReact()]
});
```

For AI SDK integration (Vercel AI SDK, LangChain.js, etc.), see extension-functions-best-practices AI section.

---

## Store and Policies

### Chrome Web Store

- [Program Policies](https://developer.chrome.com/docs/webstore/program-policies/)
- Single purpose requirement
- Permissions must be explained
- Privacy policy required if handling user data

### Firefox Add-ons

- [Add-on Policies](https://extensionworkshop.com/documentation/publish/add-on-policies/)
- Same principles: minimal permissions, clear description
- Privacy policy when needed

### Best Practices

- Request minimal permissions
- Document all permission usage
- Provide clear, accurate description
- Include privacy policy for data-handling extensions

---

## Related Resources

- [SKILL.md](SKILL.md) — Main best practices
- [rules/content-ui.md](rules/content-ui.md) — Content UI API details
- [rules/manifest-fields.md](rules/manifest-fields.md) — Manifest field reference
- [rules/permissions.md](rules/permissions.md) — Permission patterns
- [rules/messaging.md](rules/messaging.md) — Messaging patterns
- [addfox-debugging skill](../addfox-debugging/SKILL.md) — Debugging guide
- [addfox-testing skill](../addfox-testing/SKILL.md) — Testing guide
- [migrate-to-addfox skill](../migrate-to-addfox/SKILL.md) — Migration guide
- [extension-functions-best-practices skill](../extension-functions-best-practices/SKILL.md) — Feature implementations
