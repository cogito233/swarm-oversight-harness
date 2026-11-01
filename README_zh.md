<h1 align="center">Swarm Oversight Harness</h1>

<p align="center"><strong>用一屏信息，监督一个长期项目里的许多 coding agent。</strong></p>

<p align="center">
  <a href="https://cogito233.github.io/blog/human-in-the-loop-oversight/">博客（为什么这样做）</a> · <a href="README.md">English</a> · <a href="MANUAL.md">手册</a>
</p>

<p align="center">
  <img src="assets/oversight-layers.svg" alt="三层：你只看一屏（章程、预算、各条线）并做两类决定；中间是任务文件和 planner；下面是许多短命的 agent。你不直接盯着 agent。" width="560" />
</p>

Swarm Oversight Harness 是一套 Markdown 约定加一组 Agent Skills，用来让许多 agent 在同一个项目上跑几个月。你只看一屏，只做两类决定。planner agent 负责维护仓库里的任务文件，worker agent 接一个任务、做完自己那部分就退出。它没有运行时，也不是服务，Claude Code、Codex 或任何能读文件的 agent 都能用。

## 安装

把这句贴给 Claude Code、Codex 或 Cursor：

```
Install Swarm Oversight Harness from https://github.com/cogito233/swarm-oversight-harness — follow its INSTALL.md.
```

装好后可以试：

```
在这个仓库里 set up swarm oversight。
推进 T1：让 planner 拆分、派活、验证，再更新 STATE。
验收队列里有什么？我签字前先 red-team 排第一的那个。
```

## 怎么运作

1. **Setup** 只补项目缺的文件：`CHARTER`（问题和假设）、`STATE`（每条线一行，含预算）、`DECISIONS`、`JOURNAL`。
2. **你批准任务**：planner 在 `TASK.md` 里起草目标、可以改哪些文件、验收检查和预算，由你批准。
3. **agent 围绕任务干活，而不是靠自己的上下文**：往任务的 `LOG` 里追加记录，写出 `RESULT`。启动长实验的 agent 写好交接后直接退出，不轮询。
4. **验证者重跑检查**，planner 更新 `STATE`。
5. **你验收结果**，之后的工作以它为基础。

为什么这样组织，以及一屏视图的完整例子，见[博客](https://cogito233.github.io/blog/human-in-the-loop-oversight/)。

## 里面有什么

| Skill | 什么时候用 |
|---|---|
| [`swarm-setup`](skills/swarm-setup/SKILL.md) | 起新项目，或给已有仓库补上这套约定 |
| [`swarm-loop`](skills/swarm-loop/SKILL.md) | 让 planner 跑一轮：核对现状、拆分、派活、验证、更新 STATE |
| [`swarm-accept`](skills/swarm-accept/SKILL.md) | 你签字之前：核对证据、red-team、一屏简报 |

[`extras/`](extras/) 里另有两个可选 skill：复杂度审计和论文投稿前编辑，都是在同样的项目里长出来的。完整规则和各种 profile（`research`、`experiments`、`product-eng`、`solo`）见 [MANUAL.md](MANUAL.md)，模板在 `skills/*/templates/`。

## 核心规则

- 干活的 agent 不给自己盖章，由独立的验证者重跑检查。
- 修改、提交、启动、运行成功、验证、验收分开报告，没有证据的一律写 `unknown`。
- 执行层的问题在预算内重试；计划和现实矛盾，或任务快要超预算，就交回给你。
- 日志只追加；过期文档要标成过期，不能默默继续信。

## 局限

- 这是一套约定，代码层面没有强制执行。执行层的强制可以看 [agentsh](https://github.com/canyonroad/agentsh) 这类工具。
- 它来自一个人大约四个月的研究项目实践，不是对照研究。
- 不适合一次性的杂活。

## 相关工作

[Anthropic：长时运行 agent 的 harness](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) ·
[DeepSeek Super-agent](https://github.com/deepseek-ai/dsh-superagent-prototype) ·
[Agensh](https://arxiv.org/abs/2609.26781) ·
[Evolving Agent Teams](https://www.alphaxiv.org/abs/2610.36848) ·
[GitSwarm](https://arxiv.org/abs/2610.04862) ·
[Park & Choi：progress mirage](https://arxiv.org/abs/2607.25152)。
BibTeX 见 [docs/references.bib](docs/references.bib)。与以上项目均无关联。

## 引用

```bibtex
@software{lyu2026swarmoversight,
  title  = {Swarm Oversight Harness},
  author = {Lyu, Zhiheng},
  year   = {2026},
  url    = {https://github.com/cogito233/swarm-oversight-harness}
}
```

MIT 协议。版本历史见 [CHANGELOG.md](CHANGELOG.md) 和 [DECISIONS.md](DECISIONS.md)。
