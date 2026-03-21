# addmo-skills

A standalone skill library for AI agents building browser extensions with the [Addmo](https://github.com/addmo-build/addmo) framework. Use the [skills CLI](https://github.com/vercel-labs/skills) to add these skills with **`npx skills add`**.

**[中文说明](README-zh_CN.md)**

## Install

From your project root, run:

```bash
# Add all skills from this repo (e.g. to .cursor/skills/ or .agents/skills/)
npx skills add addmo-dev/skills

# Add only specific skills
npx skills add addmo-dev/skills --skill migrate-to-addmo
npx skills add addmo-dev/skills --skill addmo-best-practices
npx skills add addmo-dev/skills --skill extension-functions-best-practices
npx skills add addmo-dev/skills --skill addmo-debugging
npx skills add addmo-dev/skills --skill addmo-testing

# List available skills before installing
npx skills add addmo-dev/skills --list
```

Or use a full GitHub URL:

```bash
npx skills add https://github.com/addmo-dev/skills
```

## Skills

| Skill | Description |
|-------|-------------|
| **migrate-to-addmo** | Migrate existing projects to Addmo from WXT, Plasmo, Extension.js, or vanilla (no framework). |
| **addmo-best-practices** | Best practices for building extensions with Addmo: entry, config, manifest, permissions, cross-platform, frameworks/styles, messaging. |
| **extension-functions-best-practices** | Practices for extension features: video/audio, recording, screenshots, AI, login, translation; recommends lightweight libs (e.g. Mediabunny) and open-source extensions. |
| **addmo-debugging** | Debug build and runtime errors: use addmo/rsbuild terminal output, `.addmo/error.md`, and `.addmo/meta.md`; use entry, location, message, stack, and front-end-framework to locate and fix issues. |
| **addmo-testing** | Unit and e2e testing with Rstest: how to configure rstest, when to use unit vs e2e, dependencies (e.g. @rstest/core, playwright, @rstest/browser), file naming, and framework-specific libs (React, Vue, Svelte, Solid). |

## Structure

```
.
├── package.json
├── README.md
├── README-zh_CN.md
└── skills/
    ├── migrate-to-addmo/
    │   ├── SKILL.md
    │   └── references/
    ├── addmo-best-practices/
    │   ├── SKILL.md
    │   ├── reference.md
    │   └── rules/
    ├── extension-functions-best-practices/
    │   ├── SKILL.md
    │   └── reference.md
    ├── addmo-debugging/
    │   ├── SKILL.md
    │   └── reference.md
    └── addmo-testing/
        ├── SKILL.md
        └── reference.md
```

## References

- [skills CLI](https://github.com/vercel-labs/skills) — Add any skill repo with `npx skills add`.
- [Remotion skills](https://github.com/remotion-dev/skills/tree/main/skills/remotion) — Skill + rules layout reference.
- [rstackjs/agent-skills](https://github.com/rstackjs/agent-skills/tree/main/skills) — Multi-skill repo structure reference.
