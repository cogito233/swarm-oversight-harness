# agent-workflow

用「**文档脊柱 + 闭环**」推进 AI 驱动项目的个人操作手册 + 项目脚手架。

一句话：文档是脊柱，agent（含 plan）是可替换的肌肉；产能按「**每天签字采纳几份**」算，不按 token 算。人只站两端——上游生成 SPEC、下游裁决采纳，中间「拆分 → 验证 → 收口」交给 plan 闭环。

## 用它起一个新项目（3 步）

**1. 拿到脚手架**

```bash
# 独立 clone 来读 / 改
git clone git@github.com:cogito233/agent-workflow.git

# 或：作为子模块嵌进你自己的项目
git submodule add git@github.com:cogito233/agent-workflow.git agent-workflow
git submodule update --init --remote     # 协作者 clone 你的项目后，拉/更新子模块
```

**2. 拷模板、搭骨架**（照 `templates/PROJECT_SKELETON.md`），在你项目根建：

```
CHARTER.md   JOURNAL.md   STATE.md   DECISIONS.md
threads/T1-<slug>/{SPEC.md, LOG.md}
```

`templates/TASK.md`、`RESULT.md` 留着，每次分发工单 / 收回执时复制一份。

**3. 先填两样再开跑**：`CHARTER`（为什么 + 北极星）和 `T1/SPEC`（目标 + 文件边界 + 验收 criteria + 逃逸条件）。然后照手册 §3 的闭环转：拆分 → 并行隔离执行 → 验证 → 收口 → 对齐 → 下一轮。

> 开工前对一遍 [`00_operating_manual.md`](00_operating_manual.md) 末尾的 checklist。
> 拿不准「这活该不该上这套」，看手册 §1 的 misc-vs-project 表 + 「适用边界」。

## 文件

- **[`00_operating_manual.md`](00_operating_manual.md)** — 手册本体：一图流、术语表、文档体系（每个文件装什么）、控制回路、审查/采纳、断路器、产能、checklist。
- **`templates/`**
  - `PROJECT_SKELETON.md` — 项目骨架（CHARTER / JOURNAL / STATE / DECISIONS + 各 thread 的 SPEC·LOG 内联模板）
  - `TASK.md` — 任务分发工单（plan → 执行 agent）
  - `RESULT.md` — 结果回执（执行 agent → plan）

## 适用边界

服务的负载需满足：**拆得开 + 执行廉价 + 可验证 + 可回退 + 你掌握节奏**——软件、计算型研究（ML / benchmark / 数据 / 仿真）是主场。关键路径在外部协同、动作不可逆、或执行是慢物理过程时，它只管你自己掌控的那一截。misc 杂活别上这套。
