# agent-workflow

用「**文档脊柱 + 闭环**」推进 AI 驱动项目的个人操作手册 + 项目脚手架。

一句话：文档是脊柱，agent（含 plan）是可替换的肌肉；产能按「**每天签字采纳几份**」算，不按 token 算。人只站两端——上游生成 SPEC、下游裁决采纳，中间「拆分 → 验证 → 收口」交给 plan 闭环。

**为什么（开环 vs 闭环）**：把 AI 当纯闭环用——给个可验证目标、不管手段让它跑到绿——上手最快也最流行；但可验证的往往只是 proxy（测试通过），proxy 可被 game（删测试、假完成），而你在环外，对代码库的 grasp 每天流失，只好每天重读一遍。纯开环——事事亲手把关——受限于你的理解带宽。这套手册取双环：**HOW 的内环机器闭**（验收 criteria + 独立验证 + cycle 预算），**WHAT 的外环人闭**（SPEC + 采纳），**文档当仪表盘**——让你不重读系统也保持 grasp。

## 用它起一个新项目（3 步）

**1. 拿到脚手架**

```bash
# 独立 clone 来读 / 改
git clone git@github.com:cogito233/agent-workflow.git

# 或：作为子模块嵌进你自己的项目
git submodule add git@github.com:cogito233/agent-workflow.git agent-workflow
git submodule update --init --remote     # 协作者 clone 你的项目后，拉/更新子模块
```

**2. STEP-0 盘点 → 只补缺口**（手册 §7 STEP-0）。这套是**加法、可重入**：先拿 role 清单对照本 repo，**只创建缺失的**，已被治理的 repo 别硬套整套骨架。

- **通用 MVP（每个项目都要）**：`CHARTER` · `STATE` · `DECISIONS` · `JOURNAL`。
- **其余按 profile / 规模才补**：threads（=`templates/PROJECT_SKELETON.md` 的多 thread 布局）、单源台账、验证闸、文档治理（§8）、`templates/{TASK,RESULT}.md`、worktree —— 见手册「通用核心 + Profiles 矩阵」。socket × profile（research / experiments / product-eng / solo）查那张表选插头。

**3. 先填两样再开跑**：`CHARTER`（为什么 + 北极星）和第一个工作单元的 `SPEC`（目标 + 文件边界 + 验收 criteria + 逃逸条件）。然后照手册 §3 的闭环转：拆分 → 并行执行（容器=profile 选）→ 验证 → 收口 → 对齐 → 下一轮。

> 开工前对一遍 [`MANUAL.md`](MANUAL.md) §7 的 checklist。
> 拿不准「这活该不该上这套」，看手册 §1 的 misc-vs-project 表 + 「适用边界」。

## 文件

- **[`MANUAL.md`](MANUAL.md)** — 手册本体（版本化 canon）：一图流、通用核心 + Profiles 矩阵、术语表、文档体系、控制回路、审查/采纳（采纳 = 跨信任边界晋升）、断路器、产能、checklist、§8 文档治理 / 防漂移、§9 分发与上游化。
- **`templates/`**
  - `PROJECT_SKELETON.md` — 项目骨架（CHARTER / JOURNAL / STATE / DECISIONS + 各 thread 的 SPEC·LOG 内联模板）
  - `TASK.md` — 任务分发工单（plan → 执行 agent）
  - `RESULT.md` — 结果回执（执行 agent → plan）
  - `LOCAL_RULES.md` — 项目本地规则捕获点（每条标 `upstream-candidate` / `project-specific`，见手册 §9）
- **`playbooks/`** — profile 特定的收口闸 playbook（慢层，跨项目不变）
  - `paper-writing.md` — 用 code agent 写研究 / benchmark 论文的「投稿前编辑 pass」（research profile）。可调用入口：skill `paper-writing`
- **`examples/`** — 判例库：锚定已有原则的真实项目快照（描述性非规范性，契约见 `examples/README.md`）
- **`DECISIONS.md` / `CHANGELOG.md`** — 本仓库自身的 ADR 台账与版本史（手册按自己定义的规则治理）

## 版本与上游化（手册 §9）

- **下行**：项目 submodule pin 版本，CHARTER 记 `built against manual vX.Y`；升级 = 显式 `submodule update` + 读 CHANGELOG。
- **上行**：项目里长出的规则进该项目的 `LOCAL_RULES.md`（pattern 高度），标 `upstream-candidate` 的定期 sweep 回本仓库，人裁决分流（核心 / profile / playbook / 判例 / 留守），过闸记 ADR。

## 适用边界

服务的负载需满足：**拆得开 + 执行廉价 + 可验证 + 可回退 + 你掌握节奏**——软件、计算型研究（ML / benchmark / 数据 / 仿真）是主场。关键路径在外部协同、动作不可逆、或执行是慢物理过程时，它只管你自己掌控的那一截。misc 杂活别上这套。
