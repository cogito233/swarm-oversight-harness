---
tier: accepted
manual_version: 3.0.0
verified_at: 2026-10-31
verifier: maintainer
decay: banner-date
---

# Swarm Oversight Harness · Operating Manual

> An agent harness runs one agent's loop. A project harness runs the project around many agents: what to work on, who does it, how results are verified, and what a human accepts.

> **Core claim**: AI execution is nearly free, so what lasts in a project is its documents. Every agent, including the planner, is replaceable. Your output is the number of results you accept per day, not the number of tokens burned. This manual covers only two things: which documents to keep and what goes in each, and how humans and agents run a loop around those documents.

> **General rules only**: this manual holds rules that do not change from project to project. Anything specific to one project lives in that project's CHARTER / SPEC / STATE. If this file needs frequent edits, something that does not belong here has leaked in.

> **Minimum to start**: you need only three ideas.
> 1. **Environment as memory, not context as memory.** What the project knows lives in the environment (the docs, git history, run outputs), not in any agent's context window or chat history. Because the docs hold the durable state, any agent can be killed and rebuilt by reading the environment.
> 2. **Two human gates.** A human steps in at exactly two points: upstream at the **spec gate**, where they write the SPEC and decide WHAT to build, and downstream at the **accept gate**, where they sign an output into the accepted tier.
> 3. **Accepted means trusted.** Once signed, content is trusted by default and is not re-checked.
>
> You also need four documents: CHARTER, STATE, DECISIONS and JOURNAL (defined in §2; checklist in §7). Treat every other term as a dictionary entry: look it up when you hit the problem it solves, not before.

> **Why this shape (open loop vs. closed loop)**: The pure closed-loop style gives the AI a verifiable target, ignores how it gets there, and lets it iterate until everything is green. It is the easiest style to start with, so it is the most overused. But the verifiable target is usually a proxy (tests pass), and a proxy can be gamed: the agent deletes the test or fakes completion. The loop then settles on the wrong result. Because you are outside the loop, your understanding of the system erodes every day, until you have to re-read the codebase daily to keep up. The pure open-loop style, where you direct every step yourself, is capped by how much you can understand. This manual uses two loops. Machines close the inner HOW loop with acceptance criteria, independent verification and a retry budget. Humans close the outer WHAT loop with the SPEC and acceptance. The docs serve as the dashboard that keeps you informed without re-reading the system.

---

## The loop at a glance

```
  ┌─► ① HUMAN · write/adjust SPEC ────► ② PLANNER · split into file-disjoint units ──┐
  │       (taste · direction · spec gate)                          TASK ×N │ isolated worktree
  │                                                                        ▼
  │                                                       ③ WORKERS · run in parallel
  │                                                                        │ RESULT ×N
  │       (accept gate)                                                    ▼ receipts
  └── ⑥ HUMAN · accept ◄── ⑤ PLANNER · consolidate + update STATE ◄── ④ PLANNER · independent verification
                              └──────── resync with the real repo ────────┘
  ═══════════════════════════════════════════════════════════════════════════════
  Documents (span the whole loop · agents die, docs do not):
     CHARTER ── JOURNAL (story for humans) ── STATE (dashboard) ── DECISIONS ── unit container*
     (*container = profile choice: threads (default) / job-queue / kanban / solo — SPEC·LOG·RESULT mean the same in all)
  Stop rule: retry budget exhausted / verification and execution keep fighting → escalate to a human to re-plan, or kill the direction
```

> **How to read it**: the human appears only at two corners, ① writing the SPEC (top left) and ⑥ accepting (bottom left). The steps in between (split, verify, consolidate) are the planner's loop, so you never merge N results by hand. Everyone reads and writes the same docs, which survive when agents are killed and rebuilt. Your output is the number of ⑥s you complete per day. The loop produces **work units**, and the **container** that holds them is a profile choice (threads by default, or job-queue, kanban or solo). Changing the container changes only how the work is split; the loop stays the same.

## Glossary (skim once)

- **Split**: the planner cuts one large job into N small jobs that touch different files and sends them to N workers in parallel.
- **Consolidate**: the reverse of a split. When N workers finish, the planner collects their results, merges them, resolves conflicts, and writes one update back to the docs. Without this step you merge by hand, which defeats the purpose.
- **Accept**: you sign an output across the **trust boundary**, moving it from untrusted/draft to trusted/accepted. Where the boundary sits depends on the profile (table below). Merging code into main is one special case, not the definition. Acceptances per day are your real output.

  | profile | trust boundary (= accept) |
  |---|---|
  | research | working finding → accepted (a numbers owner fixes it in the VERDICT) |
  | experiments | candidate run → current-best (registry) |
  | product-eng | branch → main (merge) |
- **Reality**: what the code, repo and results actually look like right now, as opposed to what the planner thinks they look like (often a stale picture).
- **Resync**: before starting the next round, the planner re-reads the actual state (runs tests, reads diffs, greps) so that its picture matches reality. Do not command a moved army with an old map.
- **Escalate**: when a worker suspects the spec is wrong or hits something unexpected, it does not improvise. It stops and hands the question back to the human or the planner.

## Universal core (MVP) + profile matrix

This manual is a universal core plus thin profiles. The core never changes, and a few variable points take a different value in each profile. Two things come first.

**Universal MVP (every project keeps these four, whatever its profile)**:

- **CHARTER** — why + goal + hard constraints
- **STATE** — live dashboard + acceptance queue
- **DECISIONS** — append-only ADR log; keeps decisions from flip-flopping
- **JOURNAL** — the story, written for humans

Everything else (threads / source-of-record files / REDTEAM_LOG / DOC_SPEC / templates / worktrees) depends on profile and scale. Add a piece only when the §7 STEP 0 inventory finds that nothing in the repo fills its role yet.

**Profile matrix** (rows are variable points, columns are profiles, and each cell is that profile's value):

| variable point | research | experiments | product-eng | solo |
|---|---|---|---|---|
| execution container | threads | job-queue | threads+worktree | none |
| accept boundary | finding→accepted | run→current-best | branch→main | n/a |
| source of record | build NUMBERS | native ABLATION+DATA | metrics doc | n/a |
| verification gate | red-team | eval-score regression | tests/CI | none |
| freshness decay | git-edit-decay | banner-date | git-edit-decay | none |
| physical isolation | skip (per-thread LOG) | per-job dir | worktree | n/a |

> This table was calibrated on two real deployments: a long-running research benchmark project (research) and an ML training-experiments repo (experiments). A profile value needs only a name and one line, and it does not go into the manual body. Project-specific content lives in CHARTER/SPEC.

## 1. When to use this (and when not to)

AI execution is powerful, but it overshoots, produces a lot of intermediate junk, and changes the project state so much each day that you have to re-learn the project daily. The volume of output is not the root cause. The root cause is the lack of layered information and a closed loop. With both in place, you only glance at the current state and sign off; without them, you are flooded by N results that you must stitch together by hand.

But this harness is for advancing a project, not for misc chores:

| | advancing a project | misc chore |
|---|---|---|
| shape | organized exploration + execution, multi-step, iterative | infra is in place, one-shot small job |
| state | changes faster than you can remember | slow; your head is always up to date |
| "the plan might be wrong"? | yes | no (you already know the answer) |
| need to consolidate many results? | yes | no (only one path) |
| what to do | full docs + loop | let the agent do it; no workflow |

One-line test: **does this job need a plan that might be wrong, and do several agents' results need consolidating?** If not, it is a misc chore: your head is its STATE, and this harness is pure overhead. If so, it is a project. The answer can change. Once a misc job starts to carry irreversible consequences, or its plan might turn out wrong, upgrade it to a project on the spot.

**Adoption is additive and re-entrant.** You do not lay down the whole skeleton. You take an inventory of the roles that already exist and fill only the gaps (checklist and procedure in §7 STEP 0). Never force the full skeleton onto a repo that already has governance; add only the roles it lacks.

**Scope (not universal)**: the harness assumes that the work it manages can be split, is cheap to execute, can be verified, can be reversed, and moves at your pace. Software and computational research (ML, benchmarks, data, simulation) fit well. If the critical path moves outside your control (waiting on people, partners, approvals or real-world events), if actions are irreversible, or if execution is a slow physical process (wet lab, manufacturing), the harness can manage only the slice where you are the critical path and trial and error is cheap. The parts blocked by other people or by the physical world are outside its scope, and that is not a failure of the harness.

## 2. The document set: what goes in each file (the core of this manual)

Documents are layered by who writes them and how fast they change. Higher layers are written more by humans, change more slowly, are more trusted, and should not be edited casually. Lower layers are written more by machines, change faster, and are more disposable. Your attention belongs only on the top two layers.

```
project/
  CHARTER.md            # human   · almost never changes · why the project exists
  JOURNAL.md            # human   · daily               · the story (for people)
  STATE.md              # planner · live                · delivery dashboard (for scheduling)
  DECISIONS.md          # human signs · append-only     · major decisions (ADR)
  threads/
    T1-<slug>/
      SPEC.md           # human+planner · stable        · this thread's long-term contract
      LOG.md            # agent   · every ~2h           · work log (for consolidation / rebuild)
      tasks/            # optional: only when this thread is split across several agents
        TASK-01.md      # planner · one-off             · work order for one worker
      scratch/          # agent   · anything            · non-code drafts / raw data, disposable
```

| file | who writes | how often | contents |
|---|---|---|---|
| `CHARTER.md` | human | almost never | why the project exists, the **goal** (what success looks like), non-goals, hard constraints |
| `JOURNAL.md` | human (planner may draft, you edit) | daily | the high-level **story**: what got done, what was learned, what comes next, and the current key judgments and trade-offs. Not a log of every action |
| `STATE.md` | planner writes, you skim | live | **delivery dashboard**: one table with each thread's stage, next step, blocker, and what it needs you to decide. Fits on one screen |
| `DECISIONS.md` | human signs | append-only | major decisions: what was decided, why, which alternatives were rejected (ADR, never rewritten) |
| `threads/T*/SPEC.md` | human + planner agreement | stable | the thread's **long-term contract**: goal, file boundary, acceptance criteria, escape conditions |
| `threads/T*/LOG.md` | worker / planner | appended every ~2h | "what I did, ran, and changed this round", with **a current snapshot at the top** (used to rebuild after a kill) |
| `threads/T*/tasks/TASK-*.md` | planner | one-off | a concrete piece cut from the SPEC for **one** worker |
| `threads/T*/scratch/` | worker | anything | **non-code** drafts, raw data, one-off experiment outputs. Disposable, regenerable |
| code (main repo) | worker | high | real code changes on an **isolated branch/worktree** in the main repo; **accept = merge into main**, not into scratch |
| **source of record** (a role, not a fixed filename) | planner / numbers owner | as results arrive | **each headline number or metric definition lives in exactly one place, and everything else links to it**. Research: a NUMBERS file you build; experiments: the native ABLATION + DATA registry; product-eng: a metrics doc. This is the "no duplication" rule of §8. Do not insist on the name `NUMBERS.md`; use whatever the repo already has |

Three things that are easy to mix up:

**① Three "progress" documents for three different readers:**
- `JOURNAL` is the story for you. It is written daily at a high level: where we are, what comes next, and how you judge it. After a break, reading it brings you back up to speed. It is filtered on purpose and leaves out trivia.
- `STATE` is the dashboard for scheduling. It is live and kept to a minimal table: where each thread is stuck, and what needs your decision.
- `LOG` is the raw record for machines. Every ~2h the agent writes down what it did. The planner uses it to consolidate and to rebuild a thread; you almost never read it.

Each layer down is faster, more detailed, and less of your concern. That is what layering means.

**② By default one thread is one task, and the SPEC is its work order. Do not add layers early:**
- A thread is usually handled by **one agent from start to finish**. The agent works on its own isolated branch, and the thread closes when the work is done. In that case the `SPEC` is both the contract and the work order, and no separate TASK is needed.
- Only a **fat thread** (split across several parallel agents, or run over many rounds) grows a `tasks/` layer. Its `SPEC` stays stable and says where the thread is going; its `TASK`s change each round and say who does which piece now.
- So `tasks/` is optional, and most threads have only `SPEC + LOG`.

**③ The unit container is a profile choice, and physical isolation is used only when needed.** The options are `threads` (default), `job-queue` (many similar runs, e.g. a training job queue), `kanban`, and `solo` (flattened into the root). SPEC, LOG and RESULT mean the same in every container; only the shape of the work split changes.

Every split must still produce **file-disjoint** units, each touching different files or modules. Physical isolation (a worktree or branch), however, is required only when units share the same mutable artifact:
- product-eng changes code in the main repo, so it uses worktrees, and `accept = merge branch into main`. A failed attempt drops its branch, and main is untouched (see §5).
- If units are file-disjoint, or each one writes to its own per-job directory (typical for training experiments), they never collide in code. Their only collision point is a shared LOG or registry. Give each unit its own LOG (a **per-unit LOG**) and no worktree is needed.

**If the work does not split cleanly (two units would edit the same mutable files), do not make it two threads.**

**Acceptance pipeline**: to accept is to promote an output across the trust boundary, from untrusted/draft to trusted/accepted. Where the boundary sits depends on the profile (see the glossary table). Merging a thread branch into main is only the product-eng case. In research, acceptance moves a working finding to accepted; in experiments, it moves a candidate run to current-best. In every case, you gate the promotion. Each output carries frontmatter at the top (contract in §8), so you can tell at a glance whether to trust it. §8 governs trust and verification status.

Templates: `skills/swarm-setup/templates/` (PROJECT_SKELETON, LOCAL_RULES) and `skills/swarm-loop/templates/` (TASK, RESULT).

## 3. Roles and the loop

**Planner (the hub; can be killed and rebuilt from SPEC + STATE)**: the planner has four jobs. It splits work into file-disjoint units, verifies results independently, consolidates them, and **resyncs** every round before starting the next. It must be able to run tests, read diffs, and grep. "Does not execute" means it does not write code; it can still run commands. It is disposable, and every cycle can start with a fresh planner.

**Workers (replaceable)**: each worker is one agent that takes one TASK covering one file-disjoint piece, isolates itself as needed (see §2③), and hands back a RESULT receipt when done. A worker never certifies its own work as passing.

**Human (two modes; do not let the second crowd out the first)**: upstream, the human **generates**, putting taste, direction, and rewrites as drastic as "kill this whole direction" into the SPEC. Downstream, the human **gates** by signing at the accept gate. A human who leans too hard on the system turns into a bureaucrat who only signs and stops generating.

The loop (a cycle, not a pipeline):

```
meet SPEC → split into file-disjoint units → run in parallel (container = profile: threads/job-queue/kanban/solo) → verify independently → consolidate + update STATE → resync → next round
```

Generation is unbounded; consolidation is the bottleneck. The planner must do the right half of the loop (consolidation), or that work falls back on you.

Each consolidation cycle declares exactly one current **critical path**: the single verifiable result that unlocks the next stage. Several file-disjoint TASKs can still run in parallel on the critical path. Every other activity must state why it is running now and whether pausing it would affect the critical path. The rule limits competing end goals, not the number of agents.

## 4. Review and acceptance: the human holds two gates

**Verification and acceptance are two different acts, done by two different actors:**

| what is reviewed | who | why |
|---|---|---|
| did the execution **meet the SPEC** | planner / independent verifier ✅ | it did not do the work, so it has no stake in grading itself |
| **is the SPEC itself right** | **human** ✅ | hard to reverse; taste and direction live here; cannot be outsourced |

**How the verification gate shows that execution met the SPEC is a profile choice.** One invariant always holds: the verifier is never the worker.

| profile | verification gate |
|---|---|
| research | red-team (look for holes in the SPEC or conclusions; record them in REDTEAM_LOG) |
| experiments | eval-score regression (a new run must beat current-best to pass) |
| product-eng | tests / CI |

Red-teaming a SPEC splits again into two acts: exposing problems and judging them. Finding holes in the SPEC and listing failure modes goes to an agent with no stake in the SPEC, which is cheap, runs in parallel, and avoids author bias. The verdict on whether the SPEC is good enough belongs only to the human. Do not let "the human decides" turn into "the human decides from the planner's one-sided account". Without a contrarian agent acting as prosecutor, the holes never reach the table, and you are biased too, because you also approved the direction. The agent puts candidate holes on the table, and you rule only on the strongest few.

**Edited, committed, launched, ran OK, independently verified, and accepted are six different facts**, and an earlier one does not prove a later one. Report only the stage the evidence supports, and write `unknown` for anything you cannot confirm. Never advance a status because a file exists, a process is alive, or an agent says so.

Human review sits at the seams between layers: it reviews the act of acceptance, not every line of content. Its depth follows irreversibility. Irreversible decisions get a deep review, and anything that can be rolled back gets none. Add **spot-check calibration**: now and then, pick something at random and read deep into the code or raw data to calibrate how much you trust the verifier. Spot-check more in unfamiliar territory and less on well-trodden paths.

## 5. How hard to try, when to roll back, when to pull in a human

One rule: **push hard on "how" problems; stop immediately on "what/why" problems.**

| what you hit | signal | response |
|---|---|---|
| **HOW** (execution level) | criteria not met, a bug, tests red | **push hard**: retry and iterate up to the retry budget. Do not call the human at every step |
| **WHAT** (plan level) | the goal is wrong / the approach is impossible / the spec contradicts reality | **stop, escalate, and roll back immediately**. Pushing through a wrong spec is overshoot; the harder you try, the deeper the hole |
| the problem itself is ill-posed | re-planning still does not converge | kill this direction |

"Roll back when the result does not match expectations" is only half right. If the result misses the **criteria** (the work is not done yet), do not roll back; keep trying. If it breaks the **premise** (the expectation itself was wrong), roll back immediately.

**The human reviews in only two places**: ① every **acceptance**, because you must look before anything crosses the trust boundary (in product-eng, before a merge into main); and ② every **escalation**, when an agent hits a WHAT problem and pulls you in. You do not watch execution in between.

**Why rollback is cheap**: if an attempt fails or the direction is wrong, you **do not accept** it. The output stays outside the trust boundary, and the accepted state is untouched. In product-eng you drop the branch and main is unchanged; in experiments you discard the run and current-best is unchanged; in research the finding never becomes accepted. That is why you can let agents try freely: a failure costs nothing to roll back. Give every work unit a **retry budget and a stall check** so it does not burn forever on a wrong plan.

State records, context, a human's questions, and an agent's suggestions **never grant new authorization to act on their own**. A TASK written by the planner cannot create or widen the human's authorization either. A TASK may only restate authorization that already exists in the human's original instruction, the SPEC, or DECISIONS, and it must cite that source. If no source can be found, stop and escalate. New cost, irreversible side effects, scaling up, and crossing an established human gate also require you to reconfirm the **authorization scope**.

Before retrying, diagnose the cause from the failure evidence and keep the record of the previous failure. The new round must state what it changed from the last one, and a later success never overwrites an earlier failure. If a retry adds cost or external side effects, or crosses a human gate, reconfirm scope first. When the same kind of failure keeps repeating, stop adding attempts mechanically and go back to the shared mechanism or the SPEC.

## 6. Output: only two numbers limit you

The number of agents you can run is never the bottleneck, because enough tokens solve it. So do not use agents run or tokens burned as a KPI; that is a vanity metric. Two numbers actually limit you:

- **Acceptances per day (the KPI)**: each acceptance (promoting an output across the trust boundary; see the glossary) needs your deep judgment, so this is the real ceiling on output. The right rate varies by project, so calibrate your own. *Software projects often land around ≈20–40 acceptances per day; treat this as a reference point, not a target.*
- **Work units you can watch at once**: this number is capped by your attention and working memory. Within one unit you can split the work across many parallel agents cheaply, because they share context. Each additional unit, however, takes its own slice of your head. For software projects the cap is often *≈3–5* threads; this is also an empirical number, so calibrate it per project.

To run more agents, make **verification trustworthy** instead of adding tokens. Once verification is trustworthy, you read conclusions instead of code.

## 7. Start-up checklist

- [ ] **STEP 0 · inventory what exists, then fill only the gaps**: check each role on the list (CHARTER / JOURNAL / STATE / DECISIONS / source of record / verification gate / §8 doc governance / LOCAL_RULES §9) against this repo and **create only the missing roles**. Never force the whole skeleton onto a repo that already has governance; fill the gaps instead of tearing it down and rebuilding.
- [ ] Do CHARTER (why), JOURNAL (story) and the STATE dashboard all exist as separate documents?
- [ ] Does every thread have its own SPEC and LOG?
- [ ] Are work units file-disjoint, so that no two units edit the same files? Is isolation used only as needed (a worktree only when units share a mutable artifact, otherwise a per-unit LOG; see §2③)?
- [ ] Does each thread SPEC (the work order; only fat threads split into TASKs) state the goal, file boundary, acceptance criteria, escape conditions and stop conditions?
- [ ] Is verification a separate step, rather than the worker saying "I passed"? Are execution, verification and acceptance status reported separately, each with evidence?
- [ ] Do objections to the SPEC come from outside the planner (the human, or an agent with no stake)?
- [ ] Can the planner be rebuilt from SPEC + STATE, with no key information living only in one agent's head?
- [ ] Is there only one critical path right now, and does every other activity say whether pausing it would affect that path?
- [ ] Does every thread have a retry budget and a stop rule? Do retries keep earlier failures, and do high-impact retries reconfirm scope?
- [ ] Is every status that cannot be confirmed written as `unknown` instead of being upgraded by inference?
- [ ] Do I count today's output in **acceptances**, not tokens?

## 8. Doc governance / drift prevention

The docs are the project's durable state (see the core claim), so rotten docs mean a rotten project. Governance must keep three properties at once:

- **① Correct (freshness)**: what each doc says matches reality, and nothing is stale.
- **② Useful (reachability)**: every doc can be reached from an entry point. An orphan doc rots: nobody reads it, so nobody fixes it, so it misleads.
- **③ No duplication (single source)**: organize docs into role folders. Each fact lives in exactly one source of record, and everything else links to it (see the source-of-record role in §2 and the KPI definitions in §6).

**Frontmatter contract** (required at the top of authoritative / control docs; machine-checkable):

```
tier:         accepted | working | draft         # trust tier
verified_at:  <date>                             # last time a human confirmed it true
verifier:     <who>
covers:       <which code/data/artifacts this doc claims to reflect>   # for decay checks
numbers_from: <path to the source of record for headline numbers>    # required when citing numbers; never copy them inline
quarantine:   <true exempts it from freshness; explicitly marks "known stale / archived">
```

**Linter and the human-truth rule**: run a linter that checks the three properties. **If an authority or control doc (one that humans make decisions from) is neither 🟢 (fresh) nor quarantined, the linter exits non-zero.** The reason is that whatever humans treat as authoritative must be true. Quarantine a doc explicitly if you must, but never let it silently lie.

**The decay signal must be pluggable.** Choose a plugin with the `decay:` field, because the events that make a doc stale differ by profile:

| plugin | how it decides staleness | fits |
|---|---|---|
| `git-edit-decay` | a file listed in `covers` has a commit **later than** `verified_at` | invalidating events **live in git** (code repos) |
| `banner-date` | parse dates in the prose + a TTL; `verified_at` maintained by hand | no trackable commits; relies on periodic human review |
| `external-results-hook` | a script checks gitignored / external storage (new eval scores, new checkpoints) for anything that makes the doc stale | results do not land in git |

**Key trap**: `git-edit-decay` assumes that every event that invalidates a doc shows up in git. When results land in gitignored or external storage (typically eval scores and checkpoints in a training repo), git never sees a matching commit, so the plugin **never fires and fails silently**. Such profiles must use `banner-date` or `external-results-hook`.

## 9. Distribution and upstreaming: the two-way protocol between this repo and projects

This repo is itself a project. MANUAL is its accepted content, and the local rules that grow in running projects are its working findings. Promoting a rule into this repo is also an acceptance across the trust boundary: a human gates it, and it is recorded as an ADR in this repo's `DECISIONS.md`.

**Downstream (distribution)**: projects reference this repo as a git submodule and pin a version. They instantiate only the templates and **do not fork the manual**. The MANUAL header carries a version number. A change to an invariant bumps the major version; new profiles, playbooks or templates bump the minor version; examples and wording bump the patch version. A project records one line in its CHARTER, `built_against: swarm-oversight-harness v<X.Y>`. Upgrading is an explicit act (`submodule update`, then read the CHANGELOG migration notes), so when the manual changes you can see at once which projects lag behind.

**Upstream (learning), four steps: capture → nominate → decide → return**:

1. **Capture (inside the project)**: every local rule that grows while the project runs **goes into the project's `LOCAL_RULES.md`** (template: `skills/swarm-setup/templates/LOCAL_RULES.md`), not scattered across the README or other prose. Each entry is tagged `project-specific` or `upstream-candidate`. **Record the pattern, not the incident**: write "name experiment arms with content-bearing nouns, never positional letters — positional references lose meaning across contexts", not "don't write ABCD again". The incident goes into the entry's `born_from` field as provenance.
2. **Nominate (sweep)**: by default, the sweep runs **when a project's submodule is upgraded**, so the downstream step carries the upstream one and no new schedule is needed. A governance job can also run it periodically. Scan that project's LOCAL_RULES for `upstream-candidate` entries and record them in this repo's `INBOX.md`, the queue of pending decisions (create it on the first sweep). Deduplicate against **every candidate ever decided**, including rejected ones (using the status written back in step 4), not just against the current rules.
3. **Decide (human)**: first ask whether the candidate is new or old (below), then consult the routing table. Each decision at the gate is recorded as one ADR: which project the candidate came from, why, and where it landed.
4. **Return**: bump the version and update the CHANGELOG. **Write the decision back to the source project's LOCAL_RULES** (`promoted@vX.Y` / `rejected`, plus a one-line reason). Without this step, the same entry is nominated again and again and the process never converges. The project picks up the new version on its next `submodule update`, which closes the loop.

**First ask new vs. old (then consult the routing table)**: is the candidate a **genuinely new principle**, or **a new instance of an existing one**? An instance goes into `examples/` or becomes a template checklist item, **never a new rule clause**. The "no duplication" rule of §8 binds the manual itself: the same principle may not reappear in different clothes.

**Routing table (ask in order; first match wins)**:

| test | lands in | expected frequency |
|---|---|---|
| an invariant that holds across all profiles? | MANUAL body | very rare |
| holds only for one kind of project? | a row in the profile matrix (only extract to `profiles/<x>.md` if it exceeds one line) | main landing spot |
| a multi-step method invoked on demand? | a skill under `skills/`, or under `extras/` if it is optional (e.g. `swarm-paper`, `swarm-complexity-audit`) | occasional |
| a perfect instance of an existing principle? | `examples/` + anchor (contract in `examples/README.md`) | common |
| none of the above? | stays in the project's LOCAL_RULES (**the default home, not a failure**) | most |

**Self-check**: if the MANUAL body grows faster than `examples/`, the gate is too loose. Fix the gate first, then trim the rules.

---

*This manual follows its own rules. It holds only general rules and invariants, it keeps profile-specific content to a name and one line, and it leaves anything project-specific to that project's CHARTER/SPEC/STATE. The manual itself evolves through the §9 promotion gate, with each change recorded in `DECISIONS.md` and versioned in `CHANGELOG.md`.*
