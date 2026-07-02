# 项目骨架（复制即用）

> **先 STEP-0 盘点，别整套硬铺**（手册 §7 STEP-0）：通用 MVP = `CHARTER · STATE · DECISIONS · JOURNAL`，**每个项目都要**；`threads/` 容器、`tasks/`、单源台账、文档治理只在 STEP-0 盘出"该 role 还没人填"时才补。已被治理的 repo 只补缺口。
> 下面是 **threads 容器（默认 profile）** 的多 thread 布局——其他容器（job-queue / kanban / solo）见手册「Profiles 矩阵」，SPEC/LOG/RESULT 语义不变。单 thread 项目把 `threads/T1/` 的内容平铺到根即可。
> 工单/回执模板见同目录 `TASK.md`、`RESULT.md`。

```
project/
  CHARTER.md            # 人 · 几乎不动 · 为什么立项
  JOURNAL.md            # 人 · 每天    · 剧情(给人看，不是流水账)
  STATE.md              # plan · 实时  · 交付看板(给调度看)
  DECISIONS.md          # 人 · 只追加  · 重大决策(ADR)
  LOCAL_RULES.md        # 可选 · 本地规则捕获点(手册 §9)，模板: templates/LOCAL_RULES.md
  threads/
    T1-<slug>/
      SPEC.md           # 人+plan · 稳定 · 这条线的长期契约
      LOG.md            # agent · 每~2h · 当前快照 + 工作流水
      tasks/
        TASK-01.md      # plan · 一次性 · 派给单个 agent 的工单
      scratch/          # agent · 随意  · 草稿/原始数据，可丢
```

---

## CHARTER.md（人 · 几乎不动）

```
# <项目名> · CHARTER
北极星: <一句话，成功长什么样>
为什么做: <动机 / 这个项目存在的理由>
non-goals: <明确不做什么>
铁约束: <资源 / 时间 / 不可逾越的边界>
built_against: manual v<X.Y>   ← 手册 §9，升级 submodule 时同步改
```

---

## JOURNAL.md（人 · 每天 · 给人看的剧情，不是流水账）

```
# <项目名> · JOURNAL

> 每天一条，high-level。隔一阵回来，读最近几条就重新上手。只记"剧情级"的事，琐事进 LOG。

## <date>
- 今天(high-level): <1–3 条，做成了什么 / 推进了什么，不记每个动作>
- 学到 / 判断变了: <关键发现、想法的改变>
- 下一步: <接下来要做什么、为什么是它>
- 悬而未决: <在纠结的取舍 / 等外部的事>
```

---

## STATE.md（plan 写 · 实时 · 交付看板，一屏读完）

```
# <项目名> · STATE 交付看板 · 更新 <date>

| thread | 状态 | 下一步 | 阻塞? | 等我拍板 | 更新 |
|---|---|---|---|---|---|
| T1-xxx | 进行中 | <...> | - | <要我决定啥> | <date> |
| T2-xxx | 阻塞 | <...> | 等 X | - | <date> |

## 待采纳队列
- [ ] <等你签字采纳的产物 / 路径>
```

---

## DECISIONS.md（人签 · 只追加 · 永不重写）

```
# DECISIONS（ADR）

## <date> · <决策标题>
- 定了: <what>
- 为什么: <why>
- 否决了: <备选方案 + 原因>
- 影响 thread: <list>
```

---

## threads/T*/SPEC.md（人+plan 共识 · 稳定 · 长期契约）

```
# T<n>-<slug> · SPEC
目标: <这条 thread 整体要达成什么>
文件边界: 只碰 <文件/模块>；绝不碰 <文件/模块>   ← 保证和别的 thread 正交
non-goals: <这条线不做什么>
验收 criteria: <可被没干活的人核对的清单>
逃逸条件: <发现什么就停下来上报，别自作主张>
```

---

## threads/T*/LOG.md（agent · 每~2h · 快照 + 流水）

```
# T<n>-<slug> · LOG

## 当前快照（被杀死后靠这段重建）
<一段：这条线现在到哪了、关键状态、紧接着要做什么>

## 工作流水（每~2h 追加，最新在上）
- <time> 干了啥 / 跑了啥 / 改了哪些文件 / 有无 escalation
- <time> ...
```
