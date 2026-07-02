# CHANGELOG

> 版本语义（§9）：不变量改动 = major；profile / playbook / 模板增改 = minor；判例与措辞 = patch。

## v1.1.0 · 2026-07-02

认知减重 + 定位故事。新词税预算有限（≈3–5 个），只花在承重概念上。

- **新增「最小上手面」**（MANUAL 头部）：3 个概念（脊柱 / 两道闸 / canon）+ 4 份文档即可开工，其余术语降级为字典条目，撞到再查
- **新增「开环 vs 闭环」定位段**（MANUAL + README）：纯闭环 game proxy、丢 grasp；纯开环限于人的带宽；本手册 = 双环（HOW 机器闭 / WHAT 人闭 / 文档当仪表盘）
- **删除装饰性术语**：插座/插头 → 可变点/取值；「新颖性一刀」→ 直述「先问新旧」；「漏斗形健康指标」→ 一句自检；「三根轴」→ 三条；INBOX 降级为普通文件；§9 阶段名改中文（捕获/提名/裁决/回流）

迁移：纯措辞变更，无协议语义改动，引用旧术语的项目文档无需跟改。

## v1.0.0 · 2026-07-02

首个版本化 release。此前历史未版本化（init → 通用核心 + profiles 重构 → paper-writing playbook）。

- **重命名**：`00_operating_manual.md` → `MANUAL.md`（去掉暗示阅读顺序的数字前缀），头部加版本 frontmatter（自遵守 §8 契约）
- **新增 MANUAL §9**：分发与上游化——submodule pin 版本下行；四段漏斗上行；新颖性一刀 + 五路由裁决表；漏斗形健康指标
- **新增 `examples/`**：判例库（锚定入场券 + frontmatter 契约 + 防腐三条）
- **新增 `templates/LOCAL_RULES.md`**：项目侧本地规则捕获点（pattern 高度，incident 进 born_from）
- **新增 `DECISIONS.md` / `CHANGELOG.md`**：手册自身按它定义的规则治理

**迁移**：引用 `00_operating_manual.md` 的地方改指 `MANUAL.md`；各项目在 CHARTER 补一行 `built against manual v1.0.0`，并按 `templates/LOCAL_RULES.md` 建本地捕获点。
