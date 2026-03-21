# addfox-skills

独立 skill 库，供 AI 通过 [Addfox](https://github.com/addfox/addfox) 框架开发浏览器插件时使用。使用 [skills CLI](https://github.com/vercel-labs/skills) 通过 **`npx skills add`** 即可添加本库中的 skills。

[English](README.md)

## 添加使用

在项目根目录执行：

```bash
# 添加本库全部 skills 到当前项目（Cursor 为 .cursor/skills/ 或 .agents/skills/）
npx skills add addfox/skills

# 只添加指定 skill
npx skills add addfox/skills --skill migrate-to-addfox
npx skills add addfox/skills --skill addfox-best-practices
npx skills add addfox/skills --skill extension-functions-best-practices
npx skills add addfox/skills --skill addfox-debugging
npx skills add addfox/skills --skill addfox-testing

# 先列出可用的 skills 再决定安装
npx skills add addfox/skills --list
```

也可使用完整 GitHub URL：

```bash
npx skills add https://github.com/addfox/skills
```

## Skills 列表

| Skill | 用途 |
|-------|------|
| **migrate-to-addfox** | 从已有项目迁移到 Addfox：支持 WXT、Plasmo、Extension.js、无框架（vanilla）四种来源。 |
| **addfox-best-practices** | 使用 Addfox 开发插件时的最佳实践：入口、配置、manifest、权限、跨平台、框架与样式、消息通讯。 |
| **extension-functions-best-practices** | 实现扩展功能时的实践：视频/音频、录制、截图、AI、登录、翻译；推荐轻量库（如 Mediabunny）与优质开源扩展参考。 |
| **addfox-debugging** | 报错或需要调试时使用：优先查看 addfox/rsbuild 终端输出与 `.addfox/error.md`；用 entry、location、message、stack、front-end-framework 定位并排查；可查 `.addfox/meta.md` 确认插件基本信息。 |
| **addfox-testing** | 插件单元测试与 e2e 测试：如何配置 Rstest 的单元/e2e、适用场景、依赖安装、文件命名规范，以及按前端框架（React/Vue/Svelte/Solid 等）使用的第三方测试库。 |

## 目录结构

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

## 参考

- [skills CLI](https://github.com/vercel-labs/skills) — 使用 `npx skills add` 添加任意符合规范的 skill 仓库。
- [Remotion skills](https://github.com/remotion-dev/skills/tree/main/skills/remotion) — skill 与 rules 分目录的参考。
- [rstackjs/agent-skills](https://github.com/rstackjs/agent-skills/tree/main/skills) — 多 skill 分目录与 references 结构参考。
