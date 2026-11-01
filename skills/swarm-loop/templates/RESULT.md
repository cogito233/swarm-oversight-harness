# RESULT <task-id>

status: done | partial | blocked | escalated

> `done` only means the worker has delivered this round. It does not mean committed, launched, ran OK, independently verified, or accepted. Any stage without supporting evidence is `unknown`.

## Result (facts first)
- Reached: <actual end state and outputs>
- Not reached: <unfinished or unknown; "none" if none>

## Changes and outputs
- Files touched: <list>
- Branch / worktree: <name>
- Output paths: <list>

## Acceptance criteria self-check (⚠️ not authoritative; for the independent verifier's reference only)
- [x] <criterion> — check: <command or method>; observed: <result / exit status>; evidence: <path>
- [ ] <criterion> — why it is not met or cannot be confirmed: <facts>

## Failures, assumptions, unknowns
- Failures: <failures from this round and relevant earlier cycles, with evidence; "none" if none>
- Changed vs. last round: <fill in on a retry; "n/a" otherwise>
- Assumptions: <list>
- unknown: <list; never report an unknown as a success or a failure>

## Escalations
- <SPEC contradictions, boundary risks, surprising results, anything the human / planner must decide; "none" if none>

## Suggested next step (a suggestion, not a fact or an authorization)
- <suggestion + reason>
