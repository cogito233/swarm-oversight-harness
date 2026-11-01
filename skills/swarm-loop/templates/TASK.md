# TASK <id>: <title>

thread: T<n>-<slug>
isolation: worktree / branch <name>
owner-agent: <filled in at dispatch>

## Goal
<one sentence: what done looks like>

- Critical path: <quote the single project-level critical path from STATE>
- How this task serves it: <what this deliverable unlocks; if it is off the critical path, why it is being done now>
- Would pausing this task affect the critical path: <yes / no + reason>

## Read-only references
- thread SPEC: ../SPEC.md
- related files / docs: <list>

## Execution boundary (independence + authorization scope)
- Authorization basis: <the human's original instruction / SPEC / DECISIONS path and exact entry; a TASK cannot create or widen authorization>
- MAY touch: <explicit files / modules>
- MUST NOT touch: <list>
- May do directly: <actions the authorization basis clearly covers>
- Must escalate first: <actions the authorization basis does not cover, or that add cost, have irreversible side effects, scale up, or cross a human gate; "none" if none>

## Non-goals
- <what not to do>

## Rejected alternatives (do not reinvent them)
- <option X — because Y>

## Acceptance criteria (verifiable by someone who did not do the work)
- [ ] <check 1> — how to verify: <command / method>; expected evidence: <result or artifact>
- [ ] <check 2> — how to verify: <command / method>; expected evidence: <result or artifact>

## Stop / escalation conditions
- If you find any of the following, stop and escalate instead of deciding alone: <criteria; usually signs that the SPEC is wrong>
- Retry budget: <N cycles>; once it is exceeded, escalate to a human
- Retry discipline: <whether you may retry without asking while within budget; which earlier failures must be kept; when to reconfirm scope>
