# CHANGELOG

> Versioning (MANUAL §9): invariant change = major; profile / skill / template change = minor; examples and wording = patch.
> Entries before v3.0.0 are in Chinese (the manual was Chinese-only until then).

## v3.0.0 · 2026-10-31 — first public release as SuperSwarm Harness

- **Renamed** `agent-workflow` → **SuperSwarm Harness** (`superswarm-harness`).
- **English is now the canonical manual** (`MANUAL.md`). The Chinese v2.0.0 originals are frozen in `docs/zh/` and may lag.
- **Plain vocabulary**: plan agent → planner; execution agent → worker; 收口 → consolidate; 采纳 → accept; tiers canon/working/scratch → accepted/working/draft; 单源台账 → source of record; 对齐 ground truth → resync; 断路器/cycle 预算 → retry budget + stall check; 判例 → examples.
- **Packaged as Agent Skills** (`skills/`): `superswarm-setup`, `superswarm-loop`, `superswarm-accept` (new), `superswarm-complexity-audit` and `superswarm-paper` (from `playbooks/`). Templates moved into the skills that use them.
- **New**: `INSTALL.md` (agent-run install), `README_zh.md`, `LICENSE` (MIT), architecture diagrams in `assets/`.
- Private project names anonymized.

Migration: projects pinned to v1/v2 keep working (protocol semantics unchanged from v2.0.0). On upgrade, `templates/` → `skills/superswarm-setup/templates/` and `skills/superswarm-loop/templates/`; `playbooks/*` → `skills/superswarm-*`; CHARTER line becomes `built_against: superswarm-harness v3.0`. Existing docs using old Chinese terms do not need rewriting; frontmatter `tier: canon|scratch` reads as `accepted|draft`.

## v2.0.0 · 2026-07-26

从 一个长期运行的研究 benchmark 项目 吸收跨 profile 成立的工作纪律，不复制执行平台。

- **授权边界**：状态、上下文、人的问题和 agent 建议都不自动构成新的执行授权；新增成本、不可逆副作用、扩量或越过人闸时须明确范围
- **阶段事实分离**：修改、提交、启动、运行成功、独立验证和人工采纳分别报告；无法确认时写 `unknown`
- **单关键路径**：每个收口周期只声明一个会解锁下一阶段的可验证结果，其他活动说明暂停是否影响关键路径
- **重试纪律**：保留旧失败与证据，说明新一轮改变了什么；高影响重试重新确认范围，同类失败重复时回查共同机制或 SPEC
- **结果优先回执**：TASK 写清执行边界和预期证据；RESULT 先报实际结果，再报改动、验收证据、失败与未知、上报事项和建议
- **没有新增平台层**：不新增 schema、engine、完整状态机、HANDOFF / AUTHORIZATION / VERIFY 模板或 SWE 专属对象

迁移：现有项目无需重铺骨架或回填全部历史记录；从下一次 TASK / RESULT / STATE 更新开始采用新模板。活跃 STATE 补一条当前关键路径，缺乏证据的运行或采纳状态改写为 `unknown`，失败后的新 cycle 保留旧失败证据。

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
