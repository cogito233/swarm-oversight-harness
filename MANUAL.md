---
tier: accepted
manual_version: 3.0.0
verified_at: 2026-10-31
verifier: maintainer
decay: banner-date
---

# SuperSwarm Harness · Operating Manual

> **An agent harness runs one agent's loop. A project harness runs the project around many agents: what to work on, who does it, how results are verified, and what a human accepts.**

> **Core claim**: AI execution is nearly free, so the durable thing in a project is **a set of documents**. Agents — including the planner — are replaceable. Your real output is not tokens burned; it is **how many results you accept per day**. This manual covers two things only: **which documents to keep and what goes in each**, and **how humans and agents run a loop around those documents**.

> **General rules only**: this manual holds rules that do not change from project to project. Anything specific to one project lives in that project's CHARTER / SPEC / STATE. If this file needs frequent edits, something that does not belong here has leaked in.

> **Minimum to start**: three ideas are enough.
> 1. **Environment as memory, not context as memory.** What the project knows lives in the environment (the docs, git history, run outputs), not in any agent's context window or chat history. Docs are the durable state; agents are replaceable. Any agent can be killed and rebuilt by reading the environment.
> 2. **Two human gates.** A human appears in exactly two places: upstream at the **spec gate** (write the SPEC — decide WHAT) and downstream at the **accept gate** (sign an output into the accepted tier).
> 3. **Accepted means trusted.** Once signed, content is trusted by default and not re-checked.
>
> Plus 4 documents: CHARTER / STATE / DECISIONS / JOURNAL (see §7). **Every other term is a dictionary entry — look it up when you hit the problem it solves. No need to learn it up front.**

> **Why this shape (open loop vs closed loop)**: The pure closed-loop style — give the AI a verifiable target, ignore how it gets there, let it iterate until green — is the easiest to start and so the most overused. But what is verifiable is usually a proxy (tests pass), and a proxy can be gamed (delete the test, fake completion). The loop settles on the wrong result, and because you are outside the loop, your understanding of the system erodes daily until you have to re-read the codebase every day. The pure open-loop style — personally directing every step — is capped by your bandwidth to understand. This manual uses two loops: **machines close the inner HOW loop** (acceptance criteria + independent verification + retry budget); **humans close the outer WHAT loop** (SPEC + acceptance); **the docs are the dashboard** that keep you informed without re-reading the system.

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

> **How to read it**: the human appears only at **two corners** — ① writing the SPEC (top left) and ⑥ accepting (bottom left). The middle — split → verify → consolidate — is the planner's loop. **You never hand-merge N results yourself.** Everyone reads and writes the same docs: agents can be killed and rebuilt; the docs survive. Output = how many ⑥s you complete per day, not tokens burned. The loop emits **work units**; the **container** that holds them is a profile choice (threads by default / job-queue / kanban / solo). Changing the container changes only the shape of the work split; the loop itself does not change.

## Glossary (skim once)

- **Split**: the planner cuts one large job into N small jobs that touch different files and sends them to N workers in parallel.
- **Consolidate**: the reverse. When N workers finish, the planner collects their results, merges them, resolves conflicts, and writes one update back to the docs. Without this, you merge by hand — and the whole point is lost.
- **Accept**: you **sign** an output **across the trust boundary** — from untrusted/draft to trusted/accepted. The boundary depends on the profile (table below). "Merge code into main" is one special case, **not the definition**. **Acceptances per day = real output.**

  | profile | trust boundary (= accept) |
  |---|---|
  | research | working finding → accepted (a numbers owner fixes it in the VERDICT) |
  | experiments | candidate run → current-best (registry) |
  | product-eng | branch → main (merge) |
- **Reality**: what the code/repo/results **actually** look like right now (as opposed to what the planner **thinks** they look like, which is often stale).
- **Resync**: before starting the next round, the planner re-reads the actual state (run tests, read diffs, grep) so its picture matches reality. Do not command a moved army with an old map.
- **Escalate**: when a worker hits "the spec looks wrong / something unexpected", it does not improvise. It stops and hands the question back to the human or planner.

## Universal core (MVP) + profile matrix

This manual is **a universal core + thin profiles**: the core never changes; a few variable points take a value per profile. Two things first —

**Universal MVP (every project keeps these 4, regardless of profile)**:

- **CHARTER** — why + goal + hard constraints
- **STATE** — live dashboard + acceptance queue
- **DECISIONS** — append-only ADR log; stops decisions from being reversed back and forth
- **JOURNAL** — the story, written for humans

Everything else (threads / source-of-record files / REDTEAM_LOG / DOC_SPEC / templates / worktrees) depends on **profile and scale**. Add it only when the §7 STEP-0 inventory finds "nobody fills this role yet".

**Profile matrix** (each row a variable point, each column a profile, each cell that profile's value):

| variable point | research | experiments | product-eng | solo |
|---|---|---|---|---|
| execution container | threads | job-queue | threads+worktree | none |
| accept boundary | finding→accepted | run→current-best | branch→main | n/a |
| source of record | build NUMBERS | native ABLATION+DATA | metrics doc | n/a |
| verification gate | red-team | eval-score regression | tests/CI | none |
| freshness decay | git-edit-decay | banner-date | git-edit-decay | none |
| physical isolation | skip (per-thread LOG) | per-job dir | worktree | n/a |

> Two real deployments calibrated this table: a long-running research benchmark project (research) and an ML training-experiments repo (experiments). A profile value needs only **a name + one line**; it does not go into the manual body. Project-specific content lives in CHARTER/SPEC.

## 1. When to use this (and when not to)

AI execution is very strong, but it **overshoots**, produces lots of intermediate **junk**, and the **state changes drastically every day** → you have to re-learn the project daily. The root cause is not that AI produces too much; it is the lack of **layered information + a closed loop** that keeps you in the "glance at the current state, sign off" position instead of being flooded by N results you must stitch together by hand.

But this **serves "advancing a project", not "misc chores"**:

| | advancing a project | misc chore |
|---|---|---|
| shape | organized exploration + execution, multi-step, iterative | infra is in place, one-shot small job |
| state | changes faster than you can remember | slow; your head is always up to date |
| "the plan might be wrong"? | yes | no (you already know the answer) |
| need to consolidate many results? | yes | no (only one path) |
| what to do | full docs + loop | let the agent just do it — **no workflow** |

One-line test: **does this job need a plan that might be wrong, and do multiple agents' results need consolidating?** No → misc (your head is its STATE; this harness is pure overhead). Yes → project. It can change: once a misc job accumulates irreversible consequences or "the plan might be wrong", **upgrade it to a project on the spot**.

**This is additive and re-entrant**: adopting it does not mean laying down the whole skeleton. It means **inventory the roles that already exist, then fill only the gaps** (checklist + procedure in §7 STEP-0). Never force the full skeleton onto a repo that is already governed — add only the roles it lacks.

**Scope (not universal)**: this assumes the part of the work it manages is **splittable + cheap to execute + verifiable + reversible + paced by you** — software and computational research (ML / benchmarks / data / simulation) are home ground. Once the critical path moves **outside** (waiting on people, partners, approvals, real-world events), actions are **irreversible**, or execution is a **slow physical process** (wet lab, manufacturing), it can only manage the slice where **you are the critical path and trial-and-error is cheap**. It does not manage the parts blocked by others or by the physical world — that is outside its scope, not a failure.

## 2. The document set: what goes in each file (the core of this manual)

Documents are layered by **who writes × how fast it changes**. **Higher up: more human, slower, more trusted, less to be casually edited. Lower down: more machine, faster, more disposable.** Your attention belongs only on the top two layers.

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
| `JOURNAL.md` | human (planner may draft, you edit) | daily | the high-level **story**: what got done, what was learned, what is next, current key judgments/trade-offs. **Not a log of every action** |
| `STATE.md` | planner writes, you skim | live | **delivery dashboard**: one table — each thread's stage / next step / blocker / what it needs you to decide. Readable on one screen |
| `DECISIONS.md` | human signs | append-only | major decisions: what was decided, why, which alternatives were rejected (ADR, never rewritten) |
| `threads/T*/SPEC.md` | human + planner agreement | stable | the thread's **long-term contract**: goal, file boundary, acceptance criteria, escape conditions |
| `threads/T*/LOG.md` | worker / planner | appended every ~2h | "what I did, ran, and changed this round"; **a current snapshot at the top** (for rebuild after a kill) |
| `threads/T*/tasks/TASK-*.md` | planner | one-off | a concrete piece cut from the SPEC for **one** worker |
| `threads/T*/scratch/` | worker | anything | **non-code** drafts, raw data, one-off experiment outputs. Disposable, regenerable |
| code (main repo) | worker | high | real code changes on an **isolated branch/worktree** in the main repo; **accept = merge into main**, not into scratch |
| **source of record** (a role, not a fixed filename) | planner / numbers owner | as results arrive | **each headline number / metric definition lives in exactly one place; everywhere else links to it** — research: a NUMBERS file you build; experiments: the native ABLATION + DATA registry; product-eng: a metrics doc. This is §8's "no duplication". Do not force the name `NUMBERS.md`; use whatever the repo already has |

Three things that are easy to mix up:

**① Three "progress" documents for three different readers:**
- `JOURNAL` (the story **for you**): daily, high-level — "where we are, what is next, how I judge it". Come back after a break and **reading it gets you back up to speed**. Deliberately filtered; no trivia.
- `STATE` (the dashboard **for scheduling**): live, a minimal table — "where each thread is stuck, what needs my decision".
- `LOG` (raw records **for machines**): every ~2h the agent writes what it did. Used by the planner to consolidate and to rebuild a thread. You basically never read it.

Faster, finer, and less of your business as you go down — that is what layering means.

**② By default a thread = a task, and the SPEC is the work order — do not add layers early:**
- A thread is usually **one agent, start to finish**: it works on its own isolated branch and closes when done. Then the `SPEC` is both contract and work order; **no separate TASK is needed**.
- Only a **fat thread** (split across several parallel agents, or many rounds) grows a `tasks/` layer: the `SPEC` stays stable (where this thread is going); `TASK`s change each round (who does which piece now).
- So `tasks/` is **optional**. Most threads have only `SPEC + LOG`.

**③ The unit container is a profile choice; physical isolation only when needed** — `threads` (default) | `job-queue` (many homogeneous runs, e.g. a training job queue) | `kanban` | `solo` (flattened into the root). SPEC/LOG/RESULT mean the same in every container; only the shape of the work split changes.

Splits must still make units **file-disjoint** (touching different files/modules), but **physical isolation (worktree/branch) is required only when units share the same mutable artifact**:
- product-eng changes code in the main repo → use worktrees; `accept = merge branch into main`; a failed try just drops the branch and main is untouched (see §5).
- If units are **file-disjoint**, or each writes an **independent per-job directory** (typical: training experiments), they never collide in code. The real collision point is the **shared LOG / registry** — solve it with a **per-unit LOG** (one per unit). **No worktree needed.**

**If it does not split cleanly (both units fight over the same mutable files), do not make it two threads.**

**Acceptance pipeline**: accept = promote an output **across the trust boundary** (untrusted/draft → trusted/accepted); the boundary depends on the profile (see the glossary table). "Merge a thread branch into main" is only the **product-eng special case** — in research it is working finding → accepted; in experiments it is candidate run → current-best. In every case, **you gate the promotion**. Outputs carry frontmatter at the top (contract in §8) so you can tell at a glance whether to trust them; trust and verification status are governed by §8.

Templates: `skills/superswarm-setup/templates/` (PROJECT_SKELETON, LOCAL_RULES) and `skills/superswarm-loop/templates/` (TASK, RESULT).

## 3. Roles and the loop

**Planner (the hub; can be killed and rebuilt from SPEC + STATE)** wears four hats: split into file-disjoint units → consolidate → verify independently → **resync** every round before starting the next. It must be able to run tests, read diffs, and grep — "does not execute" means it does not write code, not that it runs nothing. It is disposable: every cycle can start with a fresh planner.

**Workers (replaceable)**: take one TASK, isolate as needed (see §2③), one agent per file-disjoint piece, and hand back a RESULT receipt when done. **A worker never certifies its own work as passing.**

**Human (two modes — do not let the second eat the first)**: upstream **generate** — inject taste, direction, and "kill this whole direction" rewrites into the SPEC; downstream **gate** — sign at the accept gate. An over-systematized human degrades into a bureaucrat who only signs, and loses the generating.

The loop (a cycle, not a pipeline):

```
meet SPEC → split into file-disjoint units → run in parallel (container = profile: threads/job-queue/kanban/solo) → verify independently → consolidate + update STATE → resync → next round
```

Generation is unbounded; **consolidation is the bottleneck**. The right half of the loop (consolidation) must be done by the planner, or it falls back on you.

Each consolidation cycle declares **exactly one current critical path**: the one verifiable result that unlocks the next stage. Inside the critical path, several file-disjoint TASKs can still run in parallel; every other activity must state why it exists now and whether pausing it would affect the critical path. The limit is on competing end goals, not on the number of agents.

## 4. Review and acceptance: the human holds two gates

**Verification ≠ acceptance. Two different acts, two different actors:**

| what is reviewed | who | why |
|---|---|---|
| did the execution **meet the SPEC** | planner / independent verifier ✅ | it is not the agent that did the work; no self-grading interest |
| **is the SPEC itself right** | **human** ✅ | hard to reverse; taste and direction live here; cannot be outsourced |

**How the verification gate proves execution met the SPEC is a profile choice** (the invariant holds: **verifier ≠ worker**):

| profile | verification gate |
|---|---|
| research | red-team (look for holes in the SPEC / conclusions, record in REDTEAM_LOG) |
| experiments | eval-score regression (a new run must beat current-best to pass) |
| product-eng | tests / CI |

One more cut: red-teaming a SPEC splits again — **exposing ≠ judging**. "Find holes in the SPEC, list failure modes" goes to an agent with no stake in the SPEC (cheap, parallel, no author bias). But the **verdict** "is this SPEC good enough" belongs only to the human. Do not let "the human decides" collapse into "the human decides based on the planner's one-sided story" — without a contrarian agent acting as prosecutor, the holes never reach the table (you approved the direction too, so you are biased as well). **The agent puts candidate holes on the table; you rule only on the strongest few.**

**Edited, committed, launched, ran OK, independently verified, and accepted are six different facts**; an earlier one does not prove a later one. Report only the stage the evidence supports. If it cannot be confirmed, write `unknown`. Never advance a status because a file exists, a process is alive, or an agent says so.

Human review sits **at the seams between layers** (it reviews the act of acceptance, not every line of content), and its depth follows **irreversibility** (deep review for irreversible decisions; none for what can be rolled back). Add **spot-check calibration**: occasionally read deep into the code/raw data at random to calibrate how much you trust the verifier — more spot checks in unfamiliar territory, fewer on well-trodden paths.

## 5. How hard to try, when to roll back, when to pull in a human

One rule: **push hard on "how" problems; stop immediately on "what/why" problems.**

| what you hit | signal | response |
|---|---|---|
| **HOW** (execution level) | criteria not met, a bug, tests red | **push hard**: retry / iterate, capped by the retry budget. Do not call the human at every step |
| **WHAT** (plan level) | the goal is wrong / the approach is impossible / the spec contradicts reality | **stop, escalate, roll back immediately**. Forcing a wrong spec = overshoot; the more you try, the deeper the hole |
| the problem itself is ill-posed | re-planning still does not converge | kill this direction |

"Roll back when it does not match expectations" is only half right: if what is violated is the **criteria** (not done yet) → do not roll back, keep trying; if what is violated is the **premise** (the expectation itself was wrong) → roll back immediately.

**The human reviews in only two places**: ① every **acceptance** (you must look before anything crosses the trust boundary; in product-eng, before merge into main); ② when an agent **escalates** (it hit a WHAT problem and pulled you in). You do not watch execution in between.

**Why rollback is cheap**: if a try fails or the direction is wrong → **do not accept**. The unpromoted output stays outside the trust boundary and the accepted truth is untouched — in product-eng, "drop the branch, main unchanged"; in experiments, "discard the run, current-best unchanged"; in research, "the finding does not become accepted". That is why you can "let it try freely; failure costs nothing to roll back". Give every work unit a **retry budget + stall check** so it does not burn forever on a wrong plan.

State records, context, a human's questions, and an agent's suggestions **never by themselves grant new authorization to act**; a TASK written by the planner cannot create or widen the human's authorization either. A TASK may only restate, with its source, authorization that already exists in the human's original instruction, the SPEC, or DECISIONS; if no source can be found, stop and escalate. New cost, irreversible side effects, scaling up, or crossing an established human gate also require reconfirming the **authorization scope**.

Before retrying, diagnose the cause from the failure evidence and keep the previous failure; the new round must state what it changed relative to the last one — a later success never overwrites an earlier failure. If a retry adds cost, external side effects, or crosses a human gate, reconfirm scope first; when the same kind of failure repeats, stop mechanically adding attempts and go back to the shared mechanism or the SPEC.

## 6. Output: only two numbers limit you

How many agents you want to run **is never the bottleneck** (enough tokens solve it), so do not use "agents run / tokens burned" as a KPI — that is a vanity metric. Two numbers actually limit you:

- **Acceptances per day = the KPI**: each acceptance (= promoting an output across the trust boundary; see glossary) needs your deep judgment; this is the real ceiling on output. **This is the KPI; the right rate varies by project — calibrate your own.** *Software-type acceptances often land around ≈20–40/day — a reference point, not a target.*
- **The number of work units you can watch at once has a ceiling**: attention / working-memory limit. **Within one unit** you can split across many parallel agents (shared context, cheap); but **each different unit** takes its own slice of your head. For software projects this is often *≈3–5* (threads) — also an empirical number; calibrate per project.

To run more agents, make **verification trustworthy** (once it is, you read conclusions, not code) — not more tokens.

## 7. Start-up checklist

- [ ] **STEP 0 · inventory what exists → fill only the gaps**: walk the role list (CHARTER / JOURNAL / STATE / DECISIONS / source of record / verification gate / §8 doc governance / LOCAL_RULES §9) against this repo and **create only the missing roles**. **Never force the whole skeleton** onto an already-governed repo — fill gaps; do not tear down and relay.
- [ ] CHARTER (why) + JOURNAL (story) + STATE dashboard exist and are not mixed?
- [ ] Every thread has its own SPEC and LOG?
- [ ] Work units are file-disjoint, not fighting over files? Isolation only as needed (worktree only when sharing the same mutable artifact; otherwise a per-unit LOG; see §2③)?
- [ ] The thread SPEC (= work order; only fat threads split into TASKs) states: goal + file boundary + acceptance criteria + escape conditions + stop conditions?
- [ ] Verification is a separate step, not the worker saying "I passed"? Execution, verification, and acceptance status are reported separately with evidence?
- [ ] Objections to the SPEC come from outside the planner (the human, or an agent with no stake)?
- [ ] The planner can be rebuilt from SPEC + STATE (no key information lives only in one agent's head)?
- [ ] There is only one critical path right now, and every other activity says whether pausing it affects the critical path?
- [ ] Every thread has a retry budget + stop rule? Retries keep earlier failures; high-impact retries reconfirm scope?
- [ ] Status that cannot be confirmed is written as `unknown`, not upgraded by inference?
- [ ] I count today's output in **acceptances**, not tokens?

## 8. Doc governance / drift prevention

The docs are the project's durable state (core claim), so **rotten docs = a rotten project**. Governance must hold three properties at once:

- **① Correct (freshness)**: what the doc says matches reality; nothing stale.
- **② Useful (reachability)**: every doc is reachable from an entry point; **an orphan doc will rot** (nobody reads it → nobody fixes it → it misleads).
- **③ No duplication (single source)**: organize by **role folders**; each fact lives in exactly one **source of record**, everywhere else links to it (see the source-of-record role in §2 and the KPI definitions in §6).

**Frontmatter contract** (required at the top of authoritative / control docs; machine-checkable):

```
tier:         accepted | working | draft         # trust tier
verified_at:  <date>                             # last time a human confirmed it true
verifier:     <who>
covers:       <which code/data/artifacts this doc claims to reflect>   # for decay checks
numbers_from: <path to the source of record for headline numbers>    # required when citing numbers; never copy them inline
quarantine:   <true exempts it from freshness; explicitly marks "known stale / archived">
```

**Linter + the human-truth rule**: run a linter that checks the three properties; **if an authority / control doc (a surface humans make decisions from) is neither 🟢 (fresh) nor quarantined → non-zero exit code**. Reason: what humans see as authoritative **must be true**. Explicitly quarantine it if you must, but never let it silently lie.

**The decay signal must be pluggable** — choose a plugin with the `decay:` field, because "what event makes a doc stale" differs by profile:

| plugin | how it decides staleness | fits |
|---|---|---|
| `git-edit-decay` | a file listed in `covers` has a commit **later than** `verified_at` | invalidating events **live in git** (code repos) |
| `banner-date` | parse dates in the prose + a TTL; `verified_at` maintained by hand | no trackable commits; relies on periodic human review |
| `external-results-hook` | a script checks gitignored / external storage (new eval scores, new checkpoints) for anything that makes the doc stale | results do not land in git |

**Key trap**: `git-edit-decay` assumes "every event that invalidates a doc is in git". When results land in **gitignored / external storage** (typical: eval scores and checkpoints in a training repo), git never gets a matching commit → the plugin **never fires and fails silently**. Such profiles must use `banner-date` or `external-results-hook`.

## 9. Distribution and upstreaming: the two-way protocol between this repo and projects

This repo is itself a project: MANUAL is its accepted content, and local rules that grow in running projects are working findings. Promoting them into this repo also goes through "accept = cross the trust boundary" — gated by a human and recorded as an ADR (in this repo's `DECISIONS.md`).

**Downstream (distribution)**: projects reference this repo as a git submodule and pin a version. They instantiate only the templates and **do not fork the manual**. The MANUAL header carries a version number: invariant changes = major; profile / playbook / template additions = minor; examples and wording = patch. A project records one line in its CHARTER, `built against manual vX.Y`; upgrading is an explicit act (`submodule update` + read the CHANGELOG migration notes) — so when the manual evolves, you can immediately see which projects lag behind.

**Upstream (learning), four steps: capture → nominate → decide → return**:

1. **Capture (inside the project)** — local rules that grow while running **all go into the project's `LOCAL_RULES.md`** (template: `skills/superswarm-setup/templates/LOCAL_RULES.md`), not scattered across README / prose. Each entry is tagged `project-specific` or `upstream-candidate`. **Record the pattern, not the incident**: write "name experiment arms with content-bearing nouns, never positional letters — positional references lose meaning across contexts", not "don't write ABCD again". The incident goes into the entry's `born_from` field as provenance.
2. **Nominate (sweep)** — **default trigger = when upgrading a project's submodule** (the downstream act carries the upstream one; no new cadence; a governance job can also run it periodically): scan that project's LOCAL_RULES for `upstream-candidate` entries and record them in this repo's `INBOX.md` (the pending-decision queue; create it on the first sweep). Deduplicate against **every candidate ever decided** (including rejected ones, via the status written back in step 4), not just current rules.
3. **Decide (human)** — first ask new vs. old (below), then consult the routing table. Each passage through the gate records one ADR: which project it came from, why, where it landed.
4. **Return** — bump the version + CHANGELOG; **write the decision back to the source project's LOCAL_RULES** (`promoted@vX.Y` / `rejected` + one-line reason) — without this step the same entry gets nominated again and again and never converges. The project picks up the new version on its next `submodule update`. Loop closed.

**First ask new vs. old (then consult the routing table)**: is the candidate a **genuinely new principle**, or **a new instance of an existing one**? If an instance → it goes to `examples/` or a template checklist item, **never a new rule clause** — §8's "no duplication" binds the manual itself: the same principle may not reappear in different clothes.

**Routing table (ask in order; first match wins)**:

| test | lands in | expected frequency |
|---|---|---|
| an invariant that holds across all profiles? | MANUAL body | very rare |
| holds only for one kind of project? | a row in the profile matrix (only extract to `profiles/<x>.md` if it exceeds one line) | main landing spot |
| a multi-step method invoked on demand? | a skill under `skills/` (e.g. `superswarm-paper`, `superswarm-complexity-audit`) | occasional |
| a perfect instance of an existing principle? | `examples/` + anchor (contract in `examples/README.md`) | common |
| none of the above? | stays in the project's LOCAL_RULES (**the default home, not a failure**) | most |

**Self-check**: if the MANUAL body grows faster than `examples/`, the gate is too loose — fix the gate first, then trim the rules.

---

*This manual follows its own rules: it holds only general rules and invariants; profile-specific content is kept to **a name + one line**; anything project-specific belongs in that project's CHARTER/SPEC/STATE, never here. The manual itself evolves through the §9 promotion gate: recorded in `DECISIONS.md`, versioned in `CHANGELOG.md`.*
