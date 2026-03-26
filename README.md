# addfox-skills

A standalone skill library for AI agents building browser extensions with the [Addfox](https://github.com/addfox/addfox) framework. Use the [skills CLI](https://github.com/vercel-labs/skills) to add these skills with **`npx skills add`**.

**[Chinese Documentation (English version)](README-zh_CN.md)**

## Install

From your project root, run:

```bash
# Add all skills from this repo (e.g. to .cursor/skills/ or .agents/skills/)
npx skills add addfox/skills

# Add only specific skills
npx skills add addfox/skills --skill migrate-to-addfox
npx skills add addfox/skills --skill addfox-best-practices
npx skills add addfox/skills --skill extension-functions-best-practices
npx skills add addfox/skills --skill addfox-debugging
npx skills add addfox/skills --skill addfox-testing

# List available skills before installing
npx skills add addfox/skills --list
```

Or use a full GitHub URL:

```bash
npx skills add https://github.com/addfox/skills
```

## Skills

| Skill | Description |
|-------|-------------|
| **migrate-to-addfox** | Migrate existing projects to Addfox from WXT, Plasmo, Extension.js, or vanilla (no framework). |
| **addfox-best-practices** | Best practices for building extensions with Addfox: entry, config, manifest, permissions, cross-platform, frameworks/styles, messaging. |
| **extension-functions-best-practices** | Practices for extension features: video/audio, recording, screenshots, AI, login, translation; recommends lightweight libs (e.g. Mediabunny) and open-source extensions. |
| **addfox-debugging** | Debug build and runtime errors: use addfox/rsbuild terminal output, `.addfox/error.md`, and `.addfox/meta.md`; use entry, location, message, stack, and front-end-framework to locate and fix issues. |
| **addfox-testing** | Unit and e2e testing with Rstest: how to configure rstest, when to use unit vs e2e, dependencies (e.g. @rstest/core, playwright, @rstest/browser), file naming, and framework-specific libs (React, Vue, Svelte, Solid). |

## Structure

```
.
├── package.json
├── README.md
├── README-zh_CN.md
└── skills/
    ├── migrate-to-addfox/
    │   ├── SKILL.md
    │   └── references/
    ├── addfox-best-practices/
    │   ├── SKILL.md
    │   ├── reference.md
    │   └── rules/
    ├── extension-functions-best-practices/
    │   ├── SKILL.md
    │   └── reference.md
    ├── addfox-debugging/
    │   ├── SKILL.md
    │   └── reference.md
    └── addfox-testing/
        ├── SKILL.md
        └── reference.md
```

## References

- [skills CLI](https://github.com/vercel-labs/skills) — Add any skill repo with `npx skills add`.
- [Remotion skills](https://github.com/remotion-dev/skills/tree/main/skills/remotion) — Skill + rules layout reference.
- [rstackjs/agent-skills](https://github.com/rstackjs/agent-skills/tree/main/skills) — Multi-skill repo structure reference.
