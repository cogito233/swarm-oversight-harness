# examples/

> **What this is**: MANUAL and the profiles are the rules (normative). Examples are real snapshots that show a rule in action (descriptive). An example is not where knowledge lives; it is **evidence for a rule**. It makes an abstract rule believable, copyable, and teachable.

## Entry requirements (both required)

1. **Anchored**: it must say which rule it demonstrates (frontmatter `demonstrates:`). A candidate that has no anchor is the most informative kind, because it is one of two things:
   - a rule nobody has written down yet → send it through MANUAL §9 as a new principle;
   - an example of nothing → it is noise. Leave it in the source project's `LOCAL_RULES.md`.
2. **Self-contained**: de-identified and with no external dependencies. Someone who doesn't know the source project can read it on its own.

## Frontmatter (required on every example)

```
tier:         example
demonstrates: MANUAL §2③ / profiles/experiments#source-of-record   # anchored rule(s), required
snapshot_of:  <project> @ <date or commit>                          # source snapshot
quarantine:   true                                                  # historical snapshot, exempt from freshness (§8)
```

## Three rules against rot

1. **Descriptive, not normative.** Never cite "do it like example X" as a rule. If you want to, the rule hasn't been written yet; send it through the §9 gate.
2. **At most 3 examples per rule.** If a rule needs more, its wording is too vague. Fix the rule instead of adding examples.
3. **Link back from the rule.** At the end of the section holding the rule, add `> Example: [examples/<slug>.md](../examples/<slug>.md)`. When deciding a new candidate, check the existing examples first ("how did we route this kind of rule last time?") so the gate gets more consistent with use.
