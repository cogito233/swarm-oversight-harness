# LOCAL_RULES (local rules · upstreaming capture point)

> Role: see MANUAL §9, "capture". Rules that grow while the project runs go **here and only here**, not into READMEs or prose. Rules scattered elsewhere can't be swept, so they never get upstreamed.
> Write each rule at **pattern level** (a transferable class), not as this one incident. The incident itself goes in `born_from` as provenance.

## Entry format

```
## <the rule in one sentence, pattern level>
- scope: project-specific | upstream-candidate
- why: <what concretely breaks when it is violated>
- born_from: <the incident that triggered it (incidents go here, not in the rule name)>
- status: active | promoted@vX.Y | rejected
```

---

## Example entry (delete freely)

## Name experiment arms with content-bearing nouns, never position letters or pronouns
- scope: upstream-candidate
- why: Position references like "arm A / the third group / it" only make sense in the context where they were written. For a different agent, or for you a week later, every reference is lost. Anything that depends on the context in the room fails a cold rebuild.
- born_from: a three-arm experiment LOG said "arms ABCD are all launched", and no reader could reconstruct what each arm was
- status: active
