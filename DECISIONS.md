# DECISIONS（ADR · append-only）

> 本仓库自己的决策台账：手册不变量的改动、§9 晋升闸的每次过闸都记一条。永不重写。

## 2026-07-02 · v1.0.0 结构化：版本化 + 双向协议
- 定了: 手册重命名 `MANUAL.md` 并加版本 frontmatter；新增 §9（分发与上游化：submodule pin 下行 + Capture→Nominate→Adjudicate→Redistribute 四段漏斗上行 + 五路由裁决表）；增设 `examples/`（判例库）、`templates/LOCAL_RULES.md`（项目侧捕获点）、`CHANGELOG.md`、本文件
- 为什么: (1) `00_` 数字前缀暗示不存在的兄弟文件，方法论仓库无阅读顺序可编码；(2) 项目本地规则（如「实验臂用名词命名」）缺少被主仓库吸收的显式通道，散在各项目 README 里 sweep 不到；(3) 无版本号则分发 / 回流协议无法严格执行
- 否决了: 手册拆多文件（单文件对人和 agent 的检索都更好，慢层不该长目录树）；`examples/` 无条件收所有 project-specific 知识（会堕化成规则坟场，故设「锚定」入场券——example 必须示范某条已有原则）
- 影响 thread: 所有以 submodule 引用本仓库的项目（升级为显式动作，无被动 breakage）
