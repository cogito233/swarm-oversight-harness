# LOCAL_RULES（本地规则 · 上游化捕获点）

> 角色见手册 §9 Capture 段。项目运行中长出的规则**只进这里**，不散进 README / 散文——散出去的规则 sweep 不到，永远上不了游。
> 每条必须记在 **pattern 高度**（可迁移的类），不是这次的事故；事故本身进 `born_from` 做 provenance。

## 条目格式

```
## <规则一句话，pattern 高度>
- scope: project-specific | upstream-candidate
- why: <违反它时具体坏在哪>
- born_from: <触发它的事故（incident 记这里，不进规则名）>
- status: active | promoted@vX.Y | rejected
```

---

## 示例条目（可删）

## 实验臂一律用承载内容的名词命名，禁用位置字母 / 代词
- scope: upstream-candidate
- why: 「臂 A / 第三组 / 它」这类位置引用只在写下时的 context 里成立；换一个 agent、隔一周回来读，指称全部丢失——任何依赖在场上下文的表述都过不了冷重建
- born_from: 三臂实验的 LOG 里出现「ABCD 臂都已拉起」，读者完全无法重建每臂是什么
- status: active
