# examples/ · 判例库

> **定位**：MANUAL / profiles 是成文法（规范性），这里是判例（描述性）。example 不是知识的落点，是**原则的证据**——它让抽象原则可信、可抄、可教。

## 入场券（缺一不进）

1. **锚定**：必须显式标注它示范哪条原则（frontmatter `demonstrates:`）。找不到锚点的候选恰恰是最有价值的信号，二选一：
   - 它其实是一条还没被提炼的原则 → 回 MANUAL §9 晋升闸，走「全新原则」路线；
   - 它谁也不示范 → 噪音，留在来源项目的 `LOCAL_RULES.md`，不上游。
2. **快照自包含**：脱敏、无外部依赖，一个不了解来源项目的人能独立读懂。

## frontmatter 契约（每个 example 顶部必带）

```
tier:         example
demonstrates: MANUAL §2③ / profiles/experiments#单源台账   # 锚定的原则，必填，可多条
snapshot_of:  <project> @ <date 或 commit>                 # 来源快照
quarantine:   true                                         # 判例是历史快照，豁免 freshness（§8）
```

## 防腐三条

1. **描述性，非规范性**——任何地方禁止「照 example X 做」当规则引用；想这么写，说明该原则还没被提炼，回晋升闸。
2. **每原则 ≤3 判例**——超了说明原则表述太模糊，该修原则，不是堆例子。
3. **原则端反挂证据**——在被示范原则所在小节的末尾加一行 `> 判例: [examples/<slug>.md](../examples/<slug>.md)`；裁决新候选时先查判例（「这类规则上次怎么分流的」），让晋升闸随使用越来越一致。
