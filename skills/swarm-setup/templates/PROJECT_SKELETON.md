# Project skeleton (copy and fill)

> **Inventory first, don't lay down the whole tree** (MANUAL §7, STEP 0). The universal minimum is `CHARTER · STATE · DECISIONS · JOURNAL`, and every project needs it. Add `threads/`, `tasks/`, a source of record, or doc governance only when STEP 0 shows that role is not yet covered. In a repo that is already governed, fill only the gaps.
> The layout below is the **threads container (the default)**. For other containers (job-queue / kanban / solo) see the Profiles table in the MANUAL; SPEC / LOG / RESULT mean the same in all of them. A single-thread project can flatten `threads/T1/` into the root.
> Task and result templates: `TASK.md` and `RESULT.md` in the `swarm-loop` skill.

```
project/
  CHARTER.md            # human   · almost never changes · why the project exists
  JOURNAL.md            # human   · daily                · the story (for people, not a log)
  STATE.md              # planner · live                 · delivery board (for scheduling)
  DECISIONS.md          # human   · append-only          · major decisions (ADR)
  LOCAL_RULES.md        # optional · local rule capture (MANUAL §9); template: LOCAL_RULES.md
  threads/
    T1-<slug>/
      SPEC.md           # human + planner · stable  · long-term contract for this thread
      LOG.md            # agent           · ~every 2h · current snapshot + work log
      tasks/
        TASK-01.md      # planner         · one-off · work order for one worker
      scratch/          # agent           · anything · drafts / raw data, disposable
```

---

## CHARTER.md (human · almost never changes)

```
# <project> · CHARTER
goal: <one sentence: what success looks like>
Why: <motivation / reason this project exists>
Non-goals: <what we explicitly do not do>
Hard constraints: <resources / time / lines that cannot be crossed>
Budget: <total GPU-h> GPU-h + $<total> API; per-thread caps in STATE. Over a thread's cap → ask.
built_against: swarm-oversight-harness v<X.Y>   ← MANUAL §9; update when you upgrade the submodule
```

---

## JOURNAL.md (human · daily · the story, not a log)

```
# <project> · JOURNAL

> One entry per day, high level. When you come back after a break, the last few entries get you up to speed. Story-level only; details go to LOG.

## <date>
- Results / current trusted conclusions: <1–3 actual results, not agent activity>
- Failures / unknown: <blockers, unconfirmed states; "none" if none>
- Learned / judgment changed: <key findings, changes of mind>
- Next: <what comes next and why this>
- Needs a human decision: <trade-offs, authorization scope, external items; "none" if none>
```

---

## STATE.md (planner writes · live · delivery board, fits one screen)

```
# <project> · STATE · updated <date>

Critical path: <which verifiable result of which thread unlocks the next stage>

| thread | current stage | latest evidence | budget used / cap | next | blocked / needs decision | updated |
|---|---|---|---|---|---|---|
| T1-xxx | <working / awaiting verification / awaiting acceptance; `unknown` if it cannot be confirmed> | <path / check result supporting the stage; if unconfirmed, what evidence is missing> | <e.g. 40/100 GPU-h or $90/$200> | <...> | <what a human must decide; "-" if nothing> | <date> |
| T2-xxx | blocked | <failure evidence> | <used>/<cap> | <...> | waiting on X | <date> |

## Acceptance queue
- [ ] <output / path waiting for your sign-off>
```

---

## DECISIONS.md (human signs · append-only · never rewritten)

```
# DECISIONS (ADR)

## <date> · <decision title>
- Decided: <what>
- Why: <why>
- Rejected: <alternatives + reasons>
- Affects threads: <list>
```

---

## threads/T*/SPEC.md (human + planner agree · stable · long-term contract)

```
# T<n>-<slug> · SPEC
Goal: <what this thread must achieve overall>
File boundary: only touch <files/modules>; never touch <files/modules>   ← keeps threads independent
Non-goals: <what this thread does not do>
Acceptance criteria: <a checklist someone who did not do the work can verify>
Escalation conditions: <what you find that means stop and escalate, don't decide alone>
```

---

## threads/T*/LOG.md (agent · ~every 2h · snapshot + work log)

```
# T<n>-<slug> · LOG

## Current snapshot (used to rebuild after the agent is killed)
<one paragraph: where this thread is, key state, what happens next>

## Work log (append ~every 2h, newest first)
- <time> what was done / run / which files changed / any escalation
- <time> ...
```
