<h1 align="center">SuperSwarm Harness</h1>

<p align="center"><strong>让 agent 集群去干活，人只守两道闸。</strong><br/><em>Environment as Memory，而不是 Context as Memory。</em></p>

<p align="center">
  <a href="README.md">English</a> | <a href="README_zh.md">中文</a>
</p>

Agent harness 管的是一个 agent 怎么跑。**SuperSwarm Harness 管的是围着多个 agent 的整个项目**：做什么、谁来做、结果怎么验证、什么东西要由人签字。它由三样东西组成：一套纯 Markdown 文档、一个 planner/worker 闭环、一组 Agent Skills。它没有运行时，也不是服务，Claude Code、Codex 或任何能读文件的 coding agent 都能用。

<p align="center">
  <img src="assets/loop.svg" alt="闭环：SPEC 闸 → 拆分 → workers → 独立验证 → 汇总 → 验收闸" width="820" />
</p>

> [!IMPORTANT]
> **让你的 agent 来安装**，把下面这句贴给 Claude Code / Codex / Cursor：
> ```
> Install SuperSwarm Harness from https://github.com/cogito233/superswarm-harness — follow its INSTALL.md.
> ```
> **装好后试试：**
> ```
> 在这个仓库里 set up superswarm。
> ```
> ```
> 对 T1 跑一轮 superswarm loop：拆分、派活、验证、汇总。
> ```
> ```
> 验收队列里有什么？我签字前先 red-team 排第一的那个。
> ```

## 核心原则：Environment as Memory

过去的做法是 **Context as Memory**：项目状态放在 agent 的上下文窗口和聊天记录里，靠压缩、总结来续命。几周后就会断，一旦同时跑多个 agent，马上就断。

SuperSwarm Harness 的做法是 **Environment as Memory**：项目知道的一切都在仓库这个环境里，包括 CHARTER、STATE、DECISIONS，每条线各自的 SPEC 和 LOG，git 历史和运行产物。每个 agent 每轮开工先读环境，收工写回环境。好处有三个：agent 可以随时替换；并行的 worker 看到的是同一份事实；你隔两周回来，不用重读代码。

## 为什么

Agent 很便宜，同时跑十个也不难。难的是跑完以后，**你**还得知道哪些是真的：

- **瓶颈是人的注意力，不是模型。** 一个人能同时盯住的工作线大约只有 3–5 条。
- **proxy 会被 game。** "测试通过"可能是测试被删了。让 AI "迭代到绿"这种纯闭环会收敛到错误的目标，而你在环外，看不到。
- **上下文会腐烂。** 几周之后，不管是人还是 agent，都没法从聊天记录里重建"为什么现在是这个样子"。

解法是双环：

| 环 | 谁来闭 | 怎么闭 |
|---|---|---|
| **HOW**（内环，怎么做） | 机器 | 验收标准 + 独立验证 + 重试预算 |
| **WHAT**（外环，做什么） | 人 | 写 SPEC（SPEC 闸）+ 验收产物（验收闸） |

**文档就是仪表盘。** CHARTER、STATE、DECISIONS、JOURNAL，加上每条工作线各自的 SPEC 和 LOG，起两个作用：任何 agent 被杀掉，都能从文档重建；你不用重读整个项目，也能掌握它的状况。产能按**每天验收几份**算，不按烧了多少 token 算。

<p align="center">
  <img src="assets/architecture.svg" alt="管理层级：人的两道闸、planner、workers、承载状态的文档" width="900" />
</p>

## 里面有什么

| Skill | 什么时候用 |
|---|---|
| [`superswarm-setup`](skills/superswarm-setup/SKILL.md) | 起新项目，或者给已有仓库补上这套流程。它先盘点已有文档，只创建缺的那些。 |
| [`superswarm-loop`](skills/superswarm-loop/SKILL.md) | 跑一轮 planner 闭环：核对现状 → 定一条关键路径 → 按文件拆成互不重叠的任务 → workers 并行 → 独立验证 → 汇总进 STATE。 |
| [`superswarm-accept`](skills/superswarm-accept/SKILL.md) | 你签字之前用：核对证据、red-team，给你一屏的简报。agent 永远不替你验收。 |
| [`superswarm-complexity-audit`](skills/superswarm-complexity-audit/SKILL.md) | 系统"能跑"但你得一直盯着的时候用：判断哪些复杂度是必要的、偶然的、过早泛化的，哪些是控制面债务。 |
| [`superswarm-paper`](skills/superswarm-paper/SKILL.md) | 用 coding agent 写研究或 benchmark 论文时，做投稿前的编辑 pass。 |

完整说明见 **[MANUAL.md](MANUAL.md)**（英文正本），原始中文版见 [docs/zh/](docs/zh/)。

## 最承重的几条规则

1. **人只守两道闸**：写 SPEC，验收产物。中间全部归 planner。
2. **干活的不给自己盖章**：验证是独立工序，由别人来做。
3. **六个事实分开报**：修改、提交、启动、运行成功、独立验证、验收。没有证据的一律写 `unknown`。
4. **每轮只有一条关键路径**：agent 多没关系，终点不能多。
5. **HOW 问题使劲试，WHAT 问题立刻停**：执行层的 bug 在预算内重试；发现计划本身错了，马上上报。
6. **建议不等于授权**：任务不能自己创造或扩大人给的授权。

## 和其他方案的关系

| | 关注点 |
|---|---|
| Agent harness（Claude Code、Codex、DeepSeek Harness…） | 把单个 agent 的循环跑好：工具、上下文、沙箱 |
| 运行时多 agent 系统（如 DeepSeek 2026-10 的 Super-agent） | 调度：常驻 Router + 并行 Worker + 事件驱动 |
| 单 agent 仓库护栏 | 让一个 agent 跨 session 恢复上下文；完成前必须验证 |
| **SuperSwarm Harness** | **项目层：什么能被信任、谁来签字，以及能撑几个月的状态** |

这几类是互补的。Super-agent 的作者自己把"长期上下文恢复"和"审批流程设计"列为未解问题，这正是本仓库要解决的部分。*与 DeepSeek 无关联。*

## 适用边界

适合**拆得开、执行便宜、可验证、可回退、节奏由你掌握**的工作，主场是软件和计算型研究（ML、benchmark、数据、仿真）。一锤子的杂活别用，也就是你已经知道答案、结果只有一路的那种。

## 来历

这套东西 2026 年 6 月起是我的私人操作手册，在长期的研究 benchmark 项目和 ML 实验项目里实际用过。v3.0.0 是第一个公开版本：改成英文正本，用词简化，封装成 Agent Skills。

## License

MIT
