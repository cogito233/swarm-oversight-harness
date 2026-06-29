# Playbook · 用 code agent 写研究 / benchmark 论文

> **定位**：`research` profile 的一个**收口闸**——当工作单元（数据集、leaderboard、消融）已采纳进 canon，要把它们写成一篇投稿论文时，用这套做"投稿前编辑 pass"。它不替代手册的闭环，它是闭环右半边（收口 → 采纳）在「论文」这个产物上的具体形态。
>
> **不变量**：本 playbook 只写**跨论文不变**的写作纪律，不写任何单篇论文的具体内容（κ 值、ticker、模型名活在那篇的源文件里）。下面凡举例都是一行示意，不是事实。

一句话：**为终局那篇写，不为当前快照写**——任何终稿会删的东西，现在就别写进共享草稿。

---

## 三条核心（其余都是它们的推论）

### 1. 终局思维（write for the final paper）

你写的每一句都该是终稿留得下的。开发期的脏东西不进共享草稿。

- **缺的数据 = 相机就绪的 scope 声明，不是红 `[TODO]`**。用干净的散文交代范围（"本文报告 N 个模型的初步切片；完整矩阵留待 camera-ready"），而不是在正文里插红色待办——读者看到 `[TODO]` 会立刻给"未完成"盖章，并连带怀疑你每一个结论。
- **报告设计，不报告开发日志**。bug 修复史、内部 red-team 轮次、harness 重构——这些是肌肉的抽搐，不是论文。报告**系统的设计**，不是你怎么把它调出来的。
- **把贡献摆成终局，当前切片诚实标"preliminary"**。终局是大（更多模型 / 更多标的 / 更大覆盖），当前是它诚实的一小块。两者都说，但定位在终局。

### 2. taste：novel vs trivial（别 report trivial 结果）

有品味地分清"真新"和"被包装成贡献的窄实现选择"。

- **不要用制造出来的差异化列灌水对比表**。一列里只有你这行打 ✓、那个 ✓ 还是个 trivial 特性——这读起来就是假差异化，审稿人一眼看穿。
- **真正的 novelty 常是组合，不是任何单一轴**。把真正难的部分放到台前；如果没有单点突破，就老实把"组合"作为贡献。
- **保留竞品也合法得 ✓ 的列**（诚实），这样"只有我们这行全 ✓"的论断才站得住——一张全是你独有列的表，等于自证心虚。

### 3. 交互式调试 artifact（debug the typeset PDF, not the source）

把论文当成一个要 render → 截图 → 视觉迭代的 artifact，不是一坨 source。

- **持续 render 并截图 PDF，审排版后的结果**，不是只看 `.tex` 源码。表格溢出、图错位、公式断行——只有看 PDF 才发现。（复用 `latex-qa` skill 的 compile→read-PDF 循环。）
- **每个数字 / 论断 / 引用，发版前对一遍 ground truth**。绝不让读者第一个抓到错误（这是项目级铁律：人类看到的权威面必须真实）。
- **大论文拆成 per-section 文件**（`\input{sections/..}`），让多个 code agent 各编一节、互不覆盖——这就是手册「文件正交拆分」落在论文上的形态。
- **图表也当 artifact 建：HTML/CSS/SVG → headless 截图 → 视觉迭代（v1..vN）**。用 headless Chrome render，读截图，挑刺，再改。干净的技术线稿 / "PPT 级"示意图用 HTML 比 TikZ 更可控、layout 更好掌握。**别用 AI 图像生成画精确科学图**（它会把文字 / 数字搞乱）——只用它生成示意底图，再把精确标签叠上去。注意 print page-box（别让图 PDF 被裁或溢成两页）；宽图用 `figure*` / `\textwidth`（不是 `\columnwidth`）保可读；宽表用 `\resizebox` 防出血到页边。
- **多 agent 编论文按文件分工**：每个并行子 agent 拥有不相交的 section 文件（天然无冲突）。但**跨节编辑**——一个 claim / caveat 在多节重复、或把内容挪进附录——必须单手做，或显式指派一个 "canonical home"，否则并行 agent 会**全删**（落得没处放）或**全留**（删不掉）。只读 review agent、以及非论文的图 / skill agent，可与那一个编辑 agent 安全并行。

---

## 投稿前编辑 pass · checklist

把下面当一遍可执行的过稿流程。一条没过就别投。

### A. 终局 & 诚实
- [ ] **零红 TODO**：任何共享版本里没有 `[TODO]` / 红字待办。要么解决，要么移进私有笔记。
- [ ] **缺数据写成 scope 散文**，不是占位符。
- [ ] **正文不暴露开发日志**（bug 史 / red-team 轮次 / harness 重构）。
- [ ] **当前切片标 "preliminary"**，贡献定位在终局规模。
- [ ] **展示诚实，不宣称诚实**：删掉正文里 "an honest result, not a defect" / "we honestly note" / "we do not claim" 这类防御性 meta；把真正的 caveat 收进一个干净的 **Limitations** 节。
- [ ] **overclaim 向下传染，投稿前专审一遍**：论文通常在 Limitations 里诚实，却在 abstract / intro / conclusion 把 caveat 丢掉——绝对断言恰恰在这些顶层位置存活。跑一个**专门的只读 overclaim + novelty 审计 pass**；然后把正文的 caveat **往上传**到 abstract / intro / conclusion，并**退役证据够不着的绝对词**："calibrated"（需要人类 / 参照锚点才能说）、"invariant"（改成 "stable / preserved"，并 scope 到你真测过的 regime）、"best"、"first / only"。每个 headline claim 都 scope 到它真实的 n / 单次 run / 单条件基础上。**警惕近似循环验证**：一个从 rubric 同源蒸馏出来的 artifact，不能反过来独立验证那条 rubric。

### B. 品味 & 论证
- [ ] **对比表是 feature matrix**：行=系统，列=有意义的轴，格子 ✓/✗/partial——标准长相，不是散文格子。
- [ ] **没有制造出来的差异化列**；保留竞品也合法 ✓ 的列，"只有我们全 ✓"才可信。
- [ ] **novelty 论断落在真难的部分 / 组合上**，不是窄实现选择。
- [ ] **ground-truth provenance 前置**：当 benchmark 评的是"质量不是结果"，明说答案键来自哪（如"综合 ≥3 份交叉核对的人类专家产物"）——"我们对标最好的人类工作，不对标未来"。

### C. 写作工艺 tics（来自资深编辑 critique）
- [ ] **bold 纪律**：顶会论文极少用粗体——只在贡献 lead-in、表头 / winner 用。满屏粗体 = 没有重点。让句子结构承重。
- [ ] **戒 em-dash 瘾**："主句—插一句—再补一刀"的节奏是初稿信号。把约一半改成句号。
- [ ] **abstract 纪律**：约 150 词；**不放地面级单个数字**（如"在 X 上 0.04"）；不放实现细节；命名实体前先引入它（别甩裸的小写模型昵称）。只说：建了什么 + 一句话核心发现 + 为何 novel。
- [ ] **降 neologism / codename 负载 + register**：留 3–4 个真正承重的术语；内部 item code / 萌名下放附录；正文保持学术语域（正文不出现 "soul item" / "kitchensink" 这类词）。

### D. 一致性 & 结构
- [ ] **一致性卫生**：模型名大小写、术语命名、引用渲染统一；引用是真实第三方作者，**只有自引为盲审匿名**。
- [ ] **按体裁结构**：benchmark 论文以 **数据集 + leaderboard** 为头版；方法论、威胁模型 / 防御是配角，不是并列头条。**别给 "Threat Model" 单开顶级节**——把分类法折进 metric 动机，把实证审计折进 Experiments。
- [ ] **节间有显式 through-line**：每节用 1 句 bridge + forward reference 接上下文，别让论文读成随机拼接的零件。
- [ ] **结构克制**：contributions ≤ 3；Limitations 合并成 ~4 条干净的，不是碎成 10 条；benchmark 论文的 Related Work 可以放末尾。

### E. 长度 & 评测报告纪律
- [ ] **长度由 float 主导，限制卡的是正文**：砍散文很快触底；总页数其实由**图 + 表**驱动。要真缩短就**砍 / 合并 float**，不是只抠字。量「**正文页数**」（不含 references + appendix——venue 限制只卡正文），别量 PDF 总长。
- [ ] **跨节重复用 grep 量**：数一个关键 claim 的标题短语在各节出现几次（我们有过一句复现 27 次），每个 claim 给**唯一 canonical home**——这是可测量的啰嗦信号。
- [ ] **没过 completeness check 的 judge 格子一律不信**：LLM judge 可能吐空 / 截断的 verdict list；一条"缺项回填为 fail"的规则就会**制造出假的极端分**（我们碰到一个 −0.388 的格子，真值其实 ≈ +0.07）。QC 必须标记空 / 缺整层的打分输出并重判、逐格验完整性，并在信 headline 数字前用 spot-check / case study 抓这种 artifact。作者要先抓到，不是读者。
- [ ] **钉死一个 canonical 对比篮子**：绝不跨不同 instance 子集比均值（这会反复把读者、连同你自己一起搞糊涂）。
- [ ] **judge 不变性下，报跨 judge 共识**，而不是某个同厂 judge 的单点结果，作为"ground truth"。
- [ ] **failure analysis 落到 instance 级 case study**（按模型逐一拆），不只摆 aggregate 的 per-layer 表。
- [ ] **任何大规模重生成 / 重判 batch 前先备份**：在一次可能覆盖掉已分析结果的 run 之前，snapshot 论文 + 数据（sections / figures / PDF + 分数矩阵 + 报告的**时间戳 tar**）。一个坏 batch 永远不能有能力弄丢你**已经分析过的 headline state**——重跑很便宜，丢掉已分析的 headline 很贵。
- [ ] **测一个因子用受控 re-run，不用覆盖 headline**：要测对某个变量（报告语言 CN vs EN、judge 厂商、reasoning effort）的鲁棒性时，对**同一批 instance** 只变那一个因子、写到一个**不同的文件名**，从而保住已分析的 headline 切片；把变体作为**受控对比**来报。绝不为了测变体而覆盖 headline 报告。

## 为什么这套有效（一句话原理）

论文是 `research` profile 的最终采纳物，**读者就是验证闸**——而且是个不会重试、抓到一个错就整篇打折的验证闸。所以投稿前这一 pass 的全部目的是：**让读者在终稿里找不到任何"开发期残留"的把柄**。终局思维管"别写会被删的"，taste 管"别把 trivial 当贡献"，交互式调试管"别让读者第一个发现错"——三者合起来就是把一篇草稿抬过"可信"这条信任边界。

---

*本 playbook 属慢层：只含跨论文不变的写作纪律。任何单篇论文的具体内容（数字 / 标的 / 模型名）活在那篇的源文件里，不写进这里。*
