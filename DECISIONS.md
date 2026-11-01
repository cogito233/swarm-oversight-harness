# DECISIONS（ADR · append-only）

> This repo's own decision log: every invariant change and every §9 promotion gets an entry. Append-only. (Entries before 2026-10-31 are in Chinese.)

## 2026-10-31 · v3.0.0 public release: English canon, plain terms, skills
- Decided: publish as SuperSwarm Harness; English MANUAL is the single canonical source, Chinese kept as a frozen translation in docs/zh/; replace metaphor terms with plain ones; package the manual as five Agent Skills with templates inside the skills that use them; installation is agent-run via INSTALL.md.
- Why: (1) outside readers could not parse the metaphor-heavy vocabulary; (2) skills are read by agents, which handle English at least as well, and reply in the user's language — so one English canon serves both audiences without two drifting copies (§8 single source); (3) "hand a repo path to an agent to install" is how users already adopt such tools; (4) DeepSeek's Super-agent (2026-10-31) made the runtime scheduling layer public and named long-term context and approval flows as open problems — this repo's focus.
- Rejected: keeping two full canonical copies (zh + en) — guaranteed drift; names `docspine` (unclear at a glance) and `project-harness` (an earlier repo with the same name and an overlapping single-agent scope exists); a runtime/service layer — out of scope, the harness stays docs + loop + gates.
- Affects: all downstream projects on upgrade (paths and terms; no protocol semantic change).

## 2026-07-26 · v2.0.0 从 一个长期运行的研究 benchmark 项目 吸收通用工作纪律
- 定了: 吸收五条跨 profile 成立的纪律：状态和建议不自动构成授权；修改、提交、启动、运行成功、独立验证、人工采纳分别报告；每个收口周期只有一条当前关键路径；重试保留旧失败且高影响重试重新确认范围；RESULT 先写结果与可复核证据，无法确认时写 `unknown`
- 为什么: 一个长期运行的研究 benchmark 项目 的长期运行实践证明，最大的普遍风险不是缺执行平台，而是 agent 活动冒充结果、建议扩张为授权、失败历史被重试覆盖、多条最高优先级争抢收口，以及无证据状态被推断为成功或失败。这五条可以直接降低人重建真实状态的认知负担
- 否决了: 复制 一个长期运行的研究 benchmark 项目 的 schema、engine、完整状态机、机器许可、HANDOFF / AUTHORIZATION / VERIFY 新模板、固定扩量阶梯和 SWE 专属对象；通用手册只吸收纪律，不吸收执行平台
- 影响 thread: 所有 profile 的 MANUAL 不变量；TASK / RESULT / PROJECT_SKELETON 模板。现有项目不全量回填历史记录，从下一次活跃更新开始采用

## 2026-07-02 · v1.1.0 认知减重：新词税预算制
- 定了: 手册可收的「新词税」按 ≈3–5 个预算管理。承重概念（拆分/收口/采纳/canon/单源台账）保留；装饰性术语（插座/插头、新颖性一刀、漏斗形健康指标、三根轴、§9 英文阶段名）删除或降级为普通措辞。新增「最小上手面」（3 概念 + 4 文档）与「开环 vs 闭环」定位段——控制论词汇是借用已知概念，零新词税
- 为什么: 命名概念已达 ~35 个，进入 BMAD 被公开嘲讽的区间（"Lots of Capitalized Words"）；成功框架的外露概念数普遍 ≤5（git、Scrum、GSD），超预算的框架死在 onboarding 而非理解
- 否决了: 删减系统本身——重的是接口不是系统；推论型概念转为「字典条目」按需遇见，不预付学习成本
- 影响 thread: 纯措辞变更，无协议语义改动

## 2026-07-02 · v1.0.0 结构化：版本化 + 双向协议
- 定了: 手册重命名 `MANUAL.md` 并加版本 frontmatter；新增 §9（分发与上游化：submodule pin 下行 + Capture→Nominate→Adjudicate→Redistribute 四段漏斗上行 + 五路由裁决表）；增设 `examples/`（判例库）、`templates/LOCAL_RULES.md`（项目侧捕获点）、`CHANGELOG.md`、本文件
- 为什么: (1) `00_` 数字前缀暗示不存在的兄弟文件，方法论仓库无阅读顺序可编码；(2) 项目本地规则（如「实验臂用名词命名」）缺少被主仓库吸收的显式通道，散在各项目 README 里 sweep 不到；(3) 无版本号则分发 / 回流协议无法严格执行
- 否决了: 手册拆多文件（单文件对人和 agent 的检索都更好，慢层不该长目录树）；`examples/` 无条件收所有 project-specific 知识（会堕化成规则坟场，故设「锚定」入场券——example 必须示范某条已有原则）
- 影响 thread: 所有以 submodule 引用本仓库的项目（升级为显式动作，无被动 breakage）
