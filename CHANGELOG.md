# CHANGELOG

> 版本语义（§9）：不变量改动 = major；profile / playbook / 模板增改 = minor；判例与措辞 = patch。

## v1.0.0 · 2026-07-02

首个版本化 release。此前历史未版本化（init → 通用核心 + profiles 重构 → paper-writing playbook）。

- **重命名**：`00_operating_manual.md` → `MANUAL.md`（去掉暗示阅读顺序的数字前缀），头部加版本 frontmatter（自遵守 §8 契约）
- **新增 MANUAL §9**：分发与上游化——submodule pin 版本下行；四段漏斗上行；新颖性一刀 + 五路由裁决表；漏斗形健康指标
- **新增 `examples/`**：判例库（锚定入场券 + frontmatter 契约 + 防腐三条）
- **新增 `templates/LOCAL_RULES.md`**：项目侧本地规则捕获点（pattern 高度，incident 进 born_from）
- **新增 `DECISIONS.md` / `CHANGELOG.md`**：手册自身按它定义的规则治理

**迁移**：引用 `00_operating_manual.md` 的地方改指 `MANUAL.md`；各项目在 CHARTER 补一行 `built against manual v1.0.0`，并按 `templates/LOCAL_RULES.md` 建本地捕获点。
