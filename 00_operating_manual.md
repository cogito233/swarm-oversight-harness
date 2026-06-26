# Agent 工作流操作手册

> **核心命题**：AI 执行近乎免费，所以项目的持久实体是**一套文档**，agent（包括 plan agent）是可随时替换的肌肉。你的真实产能不看烧了多少 token，看**每天签字采纳了几份**。这份手册只讲两件事：**放哪些文档、各装什么**，以及**人和 agent 怎么绕着这些文档转一个闭环**。

> **本手册属于慢层**：只写跨项目不变的规则，不写任何单个项目的具体内容（那些活在各项目的 CHARTER/SPEC/STATE 里）。如果它需要频繁改，说明混进了不该在这里的东西。

---

## 一图流

```
  ┌─► ① 人 · 生成/调整 SPEC ───────► ② PLAN · 拆分正交单元 ──────┐
  │       (taste · 方向 · 上游闸)                      TASK ×N │ 隔离 worktree
  │                                                            ▼
  │                                           ③ 执行 AGENTS · 并行干活
  │                                                            │ RESULT ×N
  │       (采纳 · 下游闸)                                       ▼ 回执 envelope
  └── ⑥ 人 · 裁决采纳 ◄── ⑤ PLAN · 收口 收敛+更新 STATE ◄── ④ PLAN · 独立验证
                              └─ 对齐 ground truth ─┘
  ════════════════════════════════════════════════════════════════════
  文档体系 (贯穿全程 · agent 可死它不死)：
     CHARTER ── JOURNAL(人读叙事) ── STATE(看板) ── DECISIONS ── 工作单元容器*
     (*容器=profile选: threads默认 / job-queue / kanban / solo —— SPEC·LOG·RESULT 语义不变)
  断路器：cycle 预算耗尽 / 验证与执行反复打架 → 升级到人重做计划，或杀掉方向
```

> **读法**：人只在**两个角**出现——左上 ① 生成 SPEC，左下 ⑥ 裁决采纳。中间「拆分 → 验证 → 收口」全是 plan agent 的闭环，**绝不让你亲手去拼 N 路结果**。所有人围着同一套文档读写：agent 可被杀死重建，文档不死。产能 = 每天走完几个 ⑥（签字采纳），不是烧了多少 token。闭环喷出的是**工作单元**；装它们的**容器是 profile 选择**（threads 默认 / job-queue / kanban / solo），换容器只改 fan-out 形状，闭环本身不变。

## 术语表（先扫一眼）

- **拆分**：plan 把一件大事切成 N 份正交的小活，并行发给 N 个执行 agent。
- **收口**：反向。N 个 agent 干完，plan 把结果收集、合并、消解冲突，归并成一份更新写回文档。没有它，你就得亲手拼——那就白干了。
- **采纳**：你**签字**把一个产物**跨过信任边界**——从 untrusted/draft 抬成 trusted/canonical。边界因 profile 而异（见下表），「merge 代码进 main」只是其中一个特例，**不是定义**。**每天采纳几份 = 真实产能。**

  | profile | 信任边界（= 采纳） |
  |---|---|
  | research | working finding → canon（口径 agent 固化进 VERDICT） |
  | experiments | candidate run → current-best（registry） |
  | product-eng | branch → main（merge） |
- **ground truth**：代码/repo **此刻实际**长什么样（区别于 plan **以为**的样子，那个常常过时）。
- **对齐 ground truth**：plan 开下一轮前重新读一遍真实现状（跑测试、看 diff、grep），把认知和现实重新对上——别拿旧地图指挥已经移动过的部队。
- **上报（escalation）**：执行 agent 撞到「spec 好像错了 / 意外情况」时不自作主张，停下来把问题抛回给人或 plan。

## 通用核心 (MVP) + Profiles 矩阵

本手册是 **通用核心 + 薄 profile**：核心不变，几个"插座"按 profile 换插头。先认两件事——

**通用 MVP（每个项目都要这 4 份，不分 profile）**：

- **CHARTER** — 为什么 + 北极星 + 铁约束
- **STATE** — 实时看板 + 待采纳队列
- **DECISIONS** — append-only ADR，治"翻烧饼"
- **JOURNAL** — 给人读的叙事

其余一切（threads / 单源台账文件 / REDTEAM_LOG / DOC_SPEC / templates / worktree）都是 **profile / 规模相关**，只在 §7 STEP-0 盘点出"这个 role 还没人填"时才补。

**Profiles 矩阵**（每行一个"插座"，每列一个 profile，格子里是该装的插头）：

| 插座 | research | experiments | product-eng | solo |
|---|---|---|---|---|
| 执行协调（容器） | threads | job-queue | threads+worktree | none |
| 采纳边界 | finding→canon | run→current-best | branch→main | n/a |
| 单源台账 | build NUMBERS | native ABLATION+DATA | metrics doc | n/a |
| 验证闸 | red-team | eval-score 回归 | tests/CI | none |
| freshness decay | git-edit-decay | banner-date | git-edit-decay | none |
| 物理隔离 | skip（per-thread LOG） | per-job dir | worktree | n/a |

> 两次真实落地校准了这张表：**swe-chains**（research）、**midtrain_exps**（experiments）。profile 的具体内容是**插头（命名 + 一行）**，不进手册正文；项目特定的东西活在 CHARTER/SPEC。

## 1. 什么时候用这套（以及什么时候千万别用）

AI 执行力极强，但会**过冲**、产生大量中间**垃圾**、状态**每天剧变** → 你每天得重新读懂一遍项目。根因不是 AI 太能产，是缺一套**信息分层 + 闭环**，让你稳定待在「看一眼现状、拍板采纳」的位置，而不被 N 路结果倒灌回去做人肉拼装。

但这套**只服务"推进 project"，不服务"misc 杂活"**：

| | 推进 project | misc 杂活 |
|---|---|---|
| 形态 | 有组织的探索 + 执行，多步、会迭代 | infra 已就位，一锤子小活 |
| 状态 | 变得比你脑子记得住的快 | 慢；你脑子里的状态一直是最新的 |
| 有没有「计划可能错」 | 有 | 没有（你已知答案） |
| 要不要把多路结果收口 | 要 | 不要（就一路） |
| 怎么办 | 上整套文档 + 闭环 | 让 agent 直接干，**别上工作流** |

判别一句话：**这活需要一个"可能会错的计划"、还要把多个 agent 的结果收口吗？** 否 → misc（你的脑子就是它的 STATE，上这套纯属额外负担）；是 → project。注意会变：一个 misc 一旦攒出不可逆后果或"计划可能错"，就**就地升级**成 project。

**这套是加法、可重入**：落地不是"铺整套骨架"，而是**先盘点已有 role、只补缺口**（清单 + 流程见 §7 STEP-0）。已被治理的 repo 严禁硬套全套 skeleton——补它缺的那几个 role 就行。

**适用边界（不是万能）**：这套假设你这部分工作是「**拆得开 + 执行廉价 + 可验证 + 可回退 + 你掌握节奏**」——软件、计算型研究（ML / benchmark / 数据 / 仿真）是它的主场。一旦关键路径挪到**外部**（等人、等合作方、等审批、等真实世界事件）、动作**不可逆**、或执行是**慢物理过程**（湿实验、制造），它就只能管你**自己是关键路径、且试错便宜**的那一截，管不了被别人或物理世界卡住的部分——那些部分不归它管，不代表它失效。

## 2. 文档体系：每个文件到底装什么（本手册的核心）

文档按 **谁写 × 多快变** 分层。**越往上：越是人的、越慢、越可信、越不该乱改；越往下：越是机器的、越快、越能丢。** 你的注意力只该花在最上面两层。

```
project/
  CHARTER.md            # 人写  · 几乎不动 · 为什么立项
  JOURNAL.md            # 人写  · 每天    · 剧情(给人看)
  STATE.md              # plan写 · 实时    · 交付看板(给调度看)
  DECISIONS.md          # 人签  · 只追加  · 重大决策(ADR)
  threads/
    T1-<slug>/
      SPEC.md           # 人+plan · 稳定  · 这条线的长期契约
      LOG.md            # agent  · 每~2h · 工作流水(给机器收口/重建)
      tasks/            # 可选：只有这条线要拆给多个 agent 时才建
        TASK-01.md      # plan   · 一次性 · 派给单个 agent 的工单
      scratch/          # agent  · 随意   · 非代码草稿/原始数据，可丢
```

| 文件 | 谁写 | 多久动 | 装什么（具体） |
|---|---|---|---|
| `CHARTER.md` | 人 | 几乎不动 | 为什么做这个项目、**北极星**（成功长什么样）、non-goals、不可逾越的铁约束 |
| `JOURNAL.md` | 人（plan 可起草，你改） | 每天 | high-level **剧情**：今天做了啥、学到啥、下一步要做啥、当下的关键判断/取舍。**不是流水账** |
| `STATE.md` | plan 写、你扫 | 实时 | **交付看板**：一张表，每条 thread 的状态/下一步/阻塞/等你拍什么板。一屏读完 |
| `DECISIONS.md` | 人签 | 只追加 | 重大决策：定了啥、为什么、否决了哪些备选（ADR，永不重写） |
| `threads/T*/SPEC.md` | 人+plan 共识 | 稳定 | 这条 thread 的**长期契约**：目标、文件边界、验收 criteria、逃逸条件 |
| `threads/T*/LOG.md` | 执行/plan agent | 每~2h 追加 | "我这轮干了啥、跑了啥、改了哪些文件"；**顶部一段当前快照**（用于杀死后重建） |
| `threads/T*/tasks/TASK-*.md` | plan 写 | 一次性 | 从 SPEC 拆出来、派给**单个**执行 agent 的一块具体活 |
| `threads/T*/scratch/` | 执行 agent | 随意 | **非代码**草稿、原始数据、一次性实验产物。可丢、可重生成 |
| 代码（主库） | 执行 agent | 高频 | 在主库**隔离分支/worktree**上改真代码；**采纳 = 合进 main**，不写进 scratch |
| **单源台账**（role，非固定文件名） | plan / 口径 agent | 随结果 | **headline 数字 / 口径只存一处，别处一律链接**——research: 自建 NUMBERS；experiments: 原生 ABLATION + DATA registry；product-eng: metrics 文档。对应 §8 轴③「不冗余」。别强制叫 `NUMBERS.md`，用 repo 原生那份 |

三个容易混的地方，单独讲清：

**① 三个"进度"文档别搞混——它们是写给三种不同读者的：**
- `JOURNAL`（给**你**看的剧情）：每天 high-level，"走到哪、下一步、我怎么判断"。你隔一阵回来，**读它就重新上手**。刻意筛过，不记琐事。
- `STATE`（给**调度**看的仪表盘）：实时、极简一张表，"现在每条 thread 卡在哪、等我拍什么板"。
- `LOG`（给**机器**用的原始流水）：每~2h agent 写"我具体干了啥"。供 plan 收口、供重建 thread。你基本不用读。

从上到下越来越快、越来越细、越来越不用你管——这就是信息分层。

**② 默认 thread = task，SPEC 就是工单——别急着分层：**
- 一条 thread 通常**一个 agent 一把梭**：在自己的隔离分支上开干、干完关掉。这种情况 `SPEC` 既是契约也是工单，**不需要单独的 TASK**。（你说得对：开一个、做完关掉，就一份文档。）
- 只有**胖 thread**（一条线还要拆给多个并行 agent，或要跑很多轮）才长出 `tasks/` 这一层：`SPEC` 稳定（这条线要去哪），`TASK` 一次次变（这轮派谁干哪块）。
- 所以 `tasks/` 是**可选**的，绝大多数 thread 只有 `SPEC + LOG`。

**③ 工作单元的容器是 profile 选择，物理隔离按需上**——单元的容器形态可插拔：`threads`（默认）｜`job-queue`（大量同构 run，如训练 jobq）｜`kanban`｜`solo`（拍平到根）。SPEC/LOG/RESULT 语义在任何容器里都成立，变的只是 fan-out 形状。

拆分仍要让单元**文件正交**（碰不同文件/模块），但**物理隔离（worktree/分支）只在单元间共享同一份可变产物时才必需**：
- product-eng 在主库改代码 → 用 worktree，`采纳 = 分支合进 main`，试砸丢分支 main 毫发无损（见 §5）。
- 若单元**文件不相交**、或各写**独立 per-job 目录**（典型：训练实验），代码上根本不撞，真正的碰撞点是**共享的 LOG / registry**——用 **per-unit LOG**（每单元自己一份）化解即可，**不必上 worktree**。

**拆不干净（抢同一批可变文件）就别开成两条线。**

**采纳管线**：采纳 = 把产物**跨过信任边界**晋升（untrusted/draft → trusted/canonical），边界因 profile 而异（见术语表那张表）。「把 thread 分支 merge 进 main」只是 **product-eng 的特例**——research 里是 working finding → canon，experiments 里是 candidate run → current-best。无论哪种，**晋升动作都由你 gate**。产物顶部带 frontmatter（契约见 §8），一眼知道能不能信；能否信 / 是否已验证由 §8 文档治理那套统一管。

## 3. 角色与闭环

**Plan agent（中枢，可被杀死后从 SPEC+STATE 重建）**，戴四顶帽子：拆分正交单元 → 收口合并 → 独立验证 → 每轮**对齐 ground truth** 再开下一轮。它必须能跑测试/看 diff/grep——"不执行"指不写代码，不是不许运行任何东西。把它当牲口不当宠物：每个 cycle 都可以换一个全新的 plan。

**执行 agents（可替换的肌肉）**：接一张 TASK 工单，按需隔离（隔离与否见 §2③）、一个 agent 一块正交活，干完交一份 RESULT 回执。**永远不许自己给自己盖"达标"章。**

**人（两种模式，别让第二种吃掉第一种）**：上游**生成**——在 SPEC 注入品味、方向、"砍掉整个方向"的重构；下游**把关**——在采纳闸签字。过度系统化的人会退化成只会签字的官僚，丢了生成。

闭环（是回路不是流水线）：

```
达成 SPEC → 拆分正交单元 → 并行执行(容器=profile: threads/job-queue/kanban/solo) → 独立验证 → 收口+更新 STATE → 对齐 ground truth → 下一轮
```

生成无界，**收口才是瓶颈**。闭环右半边（收口）必须由 plan 做，否则又落回你身上。

## 4. 审查与采纳：人就守两道闸

**验证 ≠ 采纳，是两件事、两个主语：**

| 审什么 | 谁来 | 为什么 |
|---|---|---|
| 执行**达没达到 SPEC** | plan / 独立验证方 ✅ | 它不是干活那个 agent，没有自评利益 |
| **SPEC 本身对不对** | **人** ✅ | 单向门，品味和方向所在，不可外包 |

**验证闸"怎么证明执行达到了 SPEC"是 profile 选择**（不变量不变：**验证者 ≠ 干活者**）：

| profile | 验证闸类型 |
|---|---|
| research | red-team（对 SPEC / 结论找洞，记 REDTEAM_LOG） |
| experiments | eval-score 回归（新 run 必须打过 current-best 才算达标） |
| product-eng | tests / CI |

注意一刀：红 SPEC 这件事还要再劈——**暴露 ≠ 判决**。"找 SPEC 的洞、列失败模式"交给一个对 SPEC 没立场的 agent（便宜、可并行、没作者偏见）；但"这 SPEC 行不行"的**判决只能是人**。别让"人来判断"塌成"人对着 plan 的一面之词判断"——没有一个唱反调的 agent 当公诉人，洞根本不会被摆上台（你自己也批准过方向、同样有偏见）。**agent 把候选漏洞摆出来，你只对最硬的几条拍板。**

人 review 落在**层与层的接缝**（审的是"采纳"这个动作，不是逐行扫内容），深度跟**不可逆性**走（单向门深审，可回滚的不审）。外加**抽样校准**：偶尔随机深读到代码/原始数据，校准你"对验证方的信任"——项目进了陌生地带就多抽，熟路上就少抽。

## 5. 试多狠、何时回退、何时拉人进来

一句判据：**「怎么做」的问题使劲试，「做什么/为什么」的问题立刻停。**

| 撞到的是 | 信号 | 处置 |
|---|---|---|
| **HOW**（执行层） | 没过 criteria、有 bug、测试红 | **使劲试**：retry / 迭代，受 cycle 预算封顶。别动不动叫人 |
| **WHAT**（计划层） | 发现目标错了 / approach 不可能 / spec 跟现实矛盾 | **立刻停、上报、回退**。硬怼错 spec = 过冲，越试坑越深 |
| 问题本身病态 | 重做计划还是不收敛 | 杀掉这个方向 |

「不符合预期就回退」只对一半：违背的是 **criteria**（还没做到）→ 别退，接着试；违背的是**前提**（预期本身就错）→ 立刻退。

**人只在两处 review**：① 每次**采纳**（晋升过信任边界前你必看；product-eng 里即 merge 进 main 前）；② agent **上报**时（它撞到 WHAT 问题、把你拉进来）。中间的执行过程你不盯。

**回退为什么便宜**：试砸了或方向错了 → **不采纳**，未晋升的产物就停在信任边界外，canonical 真相毫发无损——product-eng 里是"丢掉分支、main 不动"，experiments 里是"弃用该 run、current-best 不变"，research 里是"finding 不进 canon"。正因如此你才敢「放手让它试，错了零成本回退」。给每个工作单元设 **cycle 预算 + 空转检测**，别在错的计划上无限烧。

## 6. 产能：只有两个数卡你

你想跑多少 agent **从来不是瓶颈**（token 给够就行），所以别拿"跑了多少 agent / 烧了多少 token"当 KPI——那是虚荣指标。真正卡你的是两个数：

- **采纳数/天 = KPI**：每采纳一份（= 把产物跨过信任边界晋升，见术语表）都要你深度判断，这是真实产能上限。**这才是 KPI；具体速率因项目而异，你自己校准**——*软件型采纳常落在 ≈20–40/天 量级，仅供参照，别当硬指标*。
- **同时盯的工作单元数有上限**：注意力/工作记忆上限。**一个单元内**可以拆很多 agent 并行（共用一个 context，便宜）；但**不同单元**各占你一块脑子。软件型项目这个数常在 *≈3–5*（threads），同样是经验值、按项目校准。

想多带 agent，靠的是把**验证做可信**（可信了你就只看结论、不看代码），不是加 token。

## 7. 开工 checklist

- [ ] **STEP 0 · 盘点已有 → 只补缺口**：先拿 role 清单（CHARTER / JOURNAL / STATE / DECISIONS / 单源台账 / 验证闸 / §8 文档治理）逐项对照本 repo，**只创建缺失的 role**。已被治理的 repo **严禁硬套整套骨架**——补缺即可，别推翻重铺。
- [ ] 有 CHARTER（为什么）+ JOURNAL（剧情）+ STATE 看板，三者不混？
- [ ] 每条 thread 有独立 SPEC 和 LOG？
- [ ] 工作单元拆到了文件正交、互不抢文件？隔离按需（共享同一可变产物才上 worktree，否则 per-unit LOG 即可，见 §2③）？
- [ ] thread 的 SPEC（=工单；胖 thread 才另拆 TASK）写了：目标 + 文件边界 + 验收 criteria + 逃逸条件 + 停止条件？
- [ ] 验证是独立工序，不是干活的 agent 自己说"我达标了"？
- [ ] SPEC 的反方意见来自 plan 之外（人，或一个没立场的 agent）？
- [ ] plan 能从 SPEC+STATE 重建（没有关键信息只活在某个 agent 脑子里）？
- [ ] 每条线有 cycle 预算 + 断路器？
- [ ] 今天的产出我用**采纳数**计，不用 token 数计？

## 8. 文档治理 / 防漂移

文档是项目的持久实体（核心命题），所以**文档烂 = 项目烂**。治理沿三根轴，缺一不可：

- **① 事实对（freshness）**：文档说的和 ground truth 一致，没说过时的话。
- **② 有用（reachability）**：每篇都从入口可达；**孤儿文档 = 必腐烂**（没人读 → 没人修 → 误导）。
- **③ 不冗余（single-source）**：按 **role 文件夹**组织；同一事实**单源台账**只存一处，别处一律链接（呼应 §2 单源台账 role + §6 KPI 口径）。

**frontmatter 契约**（权威 / 控制类文档顶部必带，机器可校验）：

```
tier:         canon | working | scratch         # 信任层级
verified_at:  <date>                            # 最后一次人核对为真
verifier:     <who>
covers:       <这篇声称反映哪些代码/数据/产物>    # 供 decay 判定
numbers_from: <headline 数字的单源台账路径>       # 引别处的数必填，禁就地抄
quarantine:   <true 时豁免 freshness，显式标注"已知过期/存档">
```

**linter + 人类真相铁律**：跑一个 linter 校验三根轴；**authority / control 类文档（人类据以决策的权威面）若既非 🟢(fresh) 又未 quarantine → 退出码 ≠ 0**。理由：人类看到的权威面**必须真实**，宁可显式 quarantine，也绝不许默默撒谎。

**decay 信号必须可插拔**——用 `decay:` 旋钮选插件，因为"什么算让文档过期的事件"随 profile 而变：

| 插件 | 怎么判过期 | 适用 |
|---|---|---|
| `git-edit-decay` | `covers` 列的文件有 commit **晚于** `verified_at` | 失效事件**在 git 里**（代码型 repo） |
| `banner-date` | 解析 prose 里的日期 + TTL，手动维护 `verified_at` | 没有可追踪 commit、靠人定期复核 |
| `external-results-hook` | 跑脚本查 gitignored / 外部存储（新 eval 分数、新 checkpoint）是否使文档过期 | 结果不落 git |

**关键陷阱**：`git-edit-decay` 假设"让文档失效的事件都在 git 里"。当结果落在 **gitignored / 外部存储**（典型：训练 repo 的 eval 分数、checkpoint），git 里永远没有对应 commit → 这个插件**永远不触发、静默失效**。这类 profile 必须改用 `banner-date` 或 `external-results-hook`。

---

*本手册自身遵守它定义的规则：它在慢层、只含不变量；profile 特定的东西是**插头（命名 + 一行）**不是手册正文，任何项目特定内容都属于那个项目的 CHARTER/SPEC/STATE，不许写进这里。*
