# agent-workflow

用「**文档脊柱 + 闭环**」推进 AI 驱动项目的个人操作手册 + 项目脚手架。

## 内容

- **[`00_operating_manual.md`](00_operating_manual.md)** — 手册本体：一图流、术语表、文档体系（每个文件装什么）、控制回路、审查/采纳、断路器、产能、开工 checklist。
- **`templates/`** — 复制即用的空白模板：
  - `PROJECT_SKELETON.md` — 项目骨架（CHARTER / JOURNAL / STATE / DECISIONS + 各 thread 的 SPEC·LOG 内联模板）
  - `TASK.md` — 任务分发工单（plan → 执行 agent）
  - `RESULT.md` — 结果回执（执行 agent → plan）

## 一句话

文档是脊柱，agent（含 plan）是可替换的肌肉；产能按「**每天签字采纳几份**」算，不按 token 算。人只站两端：上游生成 SPEC、下游裁决采纳，中间的拆分→验证→收口交给 plan 闭环。

## 适用边界

它服务的负载需满足：**拆得开 + 执行廉价 + 可验证 + 可回退 + 你掌握节奏**——软件、计算型研究（ML / benchmark / 数据 / 仿真）是主场。关键路径在外部协同、动作不可逆、或执行是慢物理过程时，它只管你自己掌控的那一截。misc 杂活别上这套。
