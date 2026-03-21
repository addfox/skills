---
name: migrate-to-addfox
description: Migrate from other extension frameworks (WXT, Plasmo, Extension.js/CRXJS, vanilla) to Addfox. Use when converting an existing extension project to Addfox or when the user mentions migration from another framework.
---

# Migrate to Addfox

Apply when migrating from other extension frameworks (WXT, Plasmo, Extension.js/CRXJS, vanilla) to Addfox.

## When to use

Use this skill when:
- Converting an existing extension from WXT, Plasmo, Extension.js, or vanilla to Addfox.
- User mentions migration from another framework.
- Porting entry structure, manifest, or config from another tool.
- Need to map concepts between frameworks.
- **Migrating specific extension features** (video, AI, etc.) that require implementation guidance.

---

## 1. Migration principles

1. **Preserve original structure first** — Use file-based entries under `app/` that mirror the original source layout to minimize change.
2. **Keep the smallest change first** — Migrate config and manifest, ensure it builds, then refactor.
3. **Validate at each step** — Run `addfox build` after each major change; confirm extension loads before proceeding.
4. **Use framework-specific references** — Consult the corresponding reference file in `references/` folder for detailed mapping.
5. **Feature-specific guidance** — When migrating features (video download, AI, etc.), also consult extension-functions-best-practices skill.

---

## 2. Common migration scenarios

| Source Framework | Key Mapping | Reference |
|------------------|-------------|-----------|
| **WXT** | `entrypoints/` → `app/`; `wxt.config.ts` → `addfox.config.ts` | [references/wxt.md](references/wxt.md) |
| **Plasmo** | `contents/` → `app/content/`; `popup.tsx` → `app/popup/index.tsx` | [references/plasmo.md](references/plasmo.md) |
| **Extension.js/CRXJS** | Vite + manifest → `addfox.config.ts` + `app/` | [references/extension-js.md](references/extension-js.md) |
| **Vanilla** | Manual manifest + scripts → `app/` + auto-discovery | [references/no-framework.md](references/no-framework.md) |

### 2.1 Entry Mapping Summary

| Entry Type | WXT | Plasmo | Extension.js | Addfox |
|------------|-----|--------|--------------|-------|
| Background | `entrypoints/background.ts` | `background.ts` | `src/background.js` | `app/background/index.ts` |
| Content | `entrypoints/content.ts` | `contents/*.ts` | `src/content.js` | `app/content/index.ts` |
| Popup | `entrypoints/popup/` | `popup.tsx` | `src/popup/` | `app/popup/index.tsx` |
| Options | `entrypoints/options/` | `options.tsx` | `src/options/` | `app/options/index.tsx` |
| Sidepanel | `entrypoints/sidepanel/` | `sidepanel.tsx` | N/A | `app/sidepanel/index.tsx` |

---

## 3. Quick migration steps (all frameworks)

### Step 1: Create addfox.config.ts

```ts
// addfox.config.ts
import { defineConfig } from 'addfox';
import { pluginReact } from '@rsbuild/plugin-react'; // or vue from '@addfox/rsbuild-plugin-vue'

const manifest = {
  manifest_version: 3,
  name: 'My Extension',
  version: '1.0.0',
  description: 'Migrated extension',
  permissions: ['storage', 'activeTab'],
  host_permissions: ['<all_urls>']
};

export default defineConfig({
  manifest: { chromium: manifest, firefox: { ...manifest } },
  plugins: [pluginReact()]
});
```

### Step 2: Create app directory structure

```bash
mkdir -p app/{background,content,popup,options}
```

### Step 3: Port entry files

Copy and adapt source files:

```ts
// app/background/index.ts
// Original: WXT background.ts or Plasmo background.ts

import browser from 'webextension-polyfill';

browser.runtime.onInstalled.addListener(() => {
  console.log('Extension installed');
});

// Port your message handlers
browser.runtime.onMessage.addListener(async (message, sender) => {
  // ... original logic
});
```

### Step 4: Install dependencies

```bash
# Remove old framework
npm uninstall wxt plasmo @crxjs/vite-plugin

# Install Addfox
npm install -D addfox
npm install webextension-polyfill

# Install framework plugin if using React/Vue/etc.
npm install -D @rsbuild/plugin-react  # or @addfox/rsbuild-plugin-vue
```

### Step 5: Update package.json scripts

```json
{
  "scripts": {
    "dev": "addfox dev -b chrome",
    "build": "addfox build"
  }
}
```

### Step 6: Build and validate

```bash
npm run build
# Check .addfox/extension/ for output
# Load in chrome://extensions/ (Chrome) or about:debugging (Firefox)
```

---

## 4. Framework-specific guides

### 4.1 WXT to Addfox

**Key differences:**
- WXT uses `entrypoints/`; Addfox uses `app/`
- WXT auto-generates manifest; Addfox uses explicit config with `manifest` field
- WXT has its own dev server; Addfox uses Rsbuild
- WXT uses `@wxt-dev/module-*`; Addfox uses `@rsbuild/plugin-*` or `@addfox/rsbuild-plugin-*`

**Mapping:**
```
entrypoints/background.ts   → app/background/index.ts
entrypoints/content.ts      → app/content/index.ts
entrypoints/popup/          → app/popup/
entrypoints/options/        → app/options/
```

See [references/wxt.md](references/wxt.md) for full details.

### 4.2 Plasmo to Addfox

**Key differences:**
- Plasmo uses inline exports; Addfox uses standard entry files
- Plasmo has `contents/` directory; Addfox uses `app/content/`
- Plasmo has `~` path alias; Addfox uses standard relative imports or configure in `rsbuild`
- Plasmo auto-discovers; Addfox uses explicit `entry` or standard `app/` structure

**Mapping:**
```
background.ts               → app/background/index.ts
contents/*.ts               → app/content/index.ts (or multiple entries)
popup.tsx                   → app/popup/index.tsx
options.tsx                 → app/options/index.tsx
```

See [references/plasmo.md](references/plasmo.md) for full details.

### 4.3 Extension.js/CRXJS to Addfox

**Key differences:**
- Extension.js uses Vite; Addfox uses Rsbuild
- CRXJS has custom manifest handling; Addfox has explicit config with `manifest` field
- Vite plugins don't work directly; use Rsbuild plugins
- `rsbuild` field (not `rsbuildConfig`) for build overrides

**Mapping:**
```
manifest.json               → addfox.config.ts manifest field
src/background.js           → app/background/index.ts
src/content.js              → app/content/index.ts
src/popup/                  → app/popup/
```

See [references/extension-js.md](references/extension-js.md) for full details.

### 4.4 Vanilla to Addfox

**Key differences:**
- No build system → Rsbuild bundling
- Manual manifest → Config-based manifest
- Inline scripts → Entry files

**Mapping:**
```
manifest.json               → addfox.config.ts
background.js               → app/background/index.ts
content.js                  → app/content/index.ts
popup.html + popup.js       → app/popup/index.html + index.ts
```

See [references/no-framework.md](references/no-framework.md) for full details.

---

## 5. Migrating specific features

When migrating extensions with specific features, use the **extension-functions-best-practices** skill for implementation details:

| Feature | Migration Tasks | Reference |
|---------|-----------------|-----------|
| **Video Download** | Background fetch, media sniffing | extension-functions-best-practices (Video) |
| **AI Sidebar** | Sidepanel entry, AI SDK setup | extension-functions-best-practices (AI) |
| **Screenshot** | activeTab permission, capture API | extension-functions-best-practices (Image) |
| **Password Manager** | Secure storage, autofill | extension-functions-best-practices (Password Manager) |
| **Web3 Wallet** | Content injection, offscreen | extension-functions-best-practices (Web3) |
| **Theme/Styling** | CSS injection, dark mode | extension-functions-best-practices (Theme) |

### 5.1 Example: Migrating a Video Download Extension

```ts
// addfox.config.ts
import { defineConfig } from 'addfox';

const manifest = {
  manifest_version: 3,
  name: 'Video Downloader',
  permissions: ['activeTab', 'downloads', 'webRequest', 'storage'],
  host_permissions: ['<all_urls>']
};

export default defineConfig({
  manifest: { chromium: manifest, firefox: { ...manifest } },
  entry: {
    // Custom entry for media sniffer
    'media-sniffer': 'entries/media-sniffer.ts'
  }
});
```

For media sniffing implementation, see extension-functions-best-practices Video section.

### 5.2 Example: Migrating an AI Sidebar Extension

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

See extension-functions-best-practices AI section for SDK integration (Vercel AI SDK, LangChain).

---

## 6. Common migration issues

### 6.1 Build Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `Cannot find module` | Missing dependencies | `npm install` all required packages |
| `Entry not found` | Wrong path in `entry` | Check relative paths from project root |
| `rsbuild is not a function` | Using `rsbuildConfig` instead of `rsbuild` | Change to `rsbuild: { ... }` |
| TypeScript errors | Different tsconfig | Align strict settings with original |
| Manifest validation | Invalid fields | Check against MV3 schema |

### 6.2 Runtime Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Extension not loading | Wrong output path | Load `.addfox/extension/`, not source |
| Content script not injecting | Wrong `matches` | Update host patterns |
| Messaging broken | No `from` field | Add `from` to all messages |
| Storage data lost | Different storage key | Migrate data if key changed |
| Polyfill not working | Missing import | Add `import browser from 'webextension-polyfill'` |

### 6.3 Feature-specific Issues

| Feature | Common Issue | Solution |
|---------|--------------|----------|
| Video download | webRequest blocking changed | Use declarativeNetRequest in MV3 |
| AI integration | CORS in content script | Use background proxy |
| Screenshot | activeTab required | Request on user gesture |
| Content UI | Shadow DOM isolation | Use `@addfox/utils` helpers |

---

## 7. Post-migration checklist

- [ ] `addfox.config.ts` created with correct `manifest` format (not `rsbuildConfig`)
- [ ] Using `@rsbuild/plugin-*` or `@addfox/rsbuild-plugin-*` for framework plugins
- [ ] `app/` directory with entry files matching original structure
- [ ] Original dependencies reinstalled (minus old framework)
- [ ] `webextension-polyfill` added for cross-browser compatibility
- [ ] Framework plugin installed if using React/Vue/etc.
- [ ] `package.json` scripts updated to use `addfox`
- [ ] Build succeeds: `addfox build`
- [ ] Extension loads in Chrome/Firefox
- [ ] All original features working
- [ ] **Feature implementations reviewed in extension-functions-best-practices**

---

## 8. Additional resources

- Framework references: `references/` folder
- Addfox best practices: [addfox-best-practices skill](../addfox-best-practices/SKILL.md)
- Feature implementations: [extension-functions-best-practices skill](../extension-functions-best-practices/SKILL.md)
- Debugging: [addfox-debugging skill](../addfox-debugging/SKILL.md)
- Testing: [addfox-testing skill](../addfox-testing/SKILL.md)
