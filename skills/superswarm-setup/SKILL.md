---
name: superswarm-setup
description: Set up SuperSwarm Harness in a repo — inventory which project docs already exist, pick a profile, and create only the missing ones (CHARTER, STATE, DECISIONS, JOURNAL, threads). Use when the user says "set up superswarm", "start a new project with this workflow", "add the harness to this repo", "bootstrap CHARTER/STATE", or hands you a project path to organize for multi-agent work.
---

# SuperSwarm Harness · Setup

Goal: after this skill runs, any agent (or the human, after two weeks away) can rebuild the project's real state from its docs alone. You **add what is missing**; you never re-lay a structure the repo already has.

Full rationale: [MANUAL.md](https://github.com/cogito233/superswarm-harness/blob/main/MANUAL.md) §2 (docs), §7 (checklist), §8 (doc hygiene).

## Step 0 · Is this a project at all?

Ask one question: *does this work need a plan that could be wrong, and results from several agents merged back together?*

- **No** → it is a one-off task. Tell the user the harness would be overhead, and just do the task.
- **Yes** → continue.

## Step 1 · Inventory (read-only)

Read the repo's entry docs first (README, AGENTS.md / CLAUDE.md, any docs/ index). Do not scan the whole tree. For each **role** below, find the file that already fills it — under any name:

| Role | What it holds | Needed |
|---|---|---|
| CHARTER | why, goal, non-goals, hard limits | always |
| STATE | one-screen board: each thread's stage, evidence, next step, what waits on the human; the one **critical path**; the accept queue | always |
| DECISIONS | append-only decision log (what, why, rejected options) | always |
| JOURNAL | human-readable story: results, unknowns, what changed in our thinking, next | always |
| threads/ (SPEC + LOG per thread) | one workstream each: contract + running log with a snapshot on top | when >1 workstream |
| source of record | the single place headline numbers live | when numbers are reported |
| verification gate | tests/CI, eval regression, or red-team | when outputs get accepted |
| LOCAL_RULES | project rules that may later go upstream | optional |

Report the inventory as a table: role → existing file (or **missing**) → note.

## Step 2 · Pick a profile

| | research | experiments | product-eng | solo |
|---|---|---|---|---|
| work container | threads | job queue | threads + worktree | none |
| what "accept" means | finding → accepted conclusion | run → current best | branch → main | n/a |
| verification gate | red-team | eval-score regression | tests / CI | none |
| isolation | per-thread LOG | per-job dir | git worktree | n/a |

Propose one; the user confirms. Mixed repos may use a different profile per thread.

## Step 3 · Create only the gaps

Use `templates/PROJECT_SKELETON.md` (next to this file). Rules:

- Never overwrite or rename an existing file that fills a role. If a role is half-filled, propose an edit, don't replace.
- CHARTER and the first SPEC need the **human's** content (goal, non-goals, acceptance criteria). Draft them from what the user said and mark open fields `<TODO: human>`; do not invent goals.
- Add one line to CHARTER: `built_against: superswarm-harness v<version>`.
- Put a frontmatter block on STATE and other authority docs (`tier`, `verified_at`, `verifier`, `covers`) — see MANUAL §8.

## Step 4 · Hand off

End with:
1. the inventory table (before → after),
2. the fields that still wait on the human,
3. the next command: "run the superswarm-loop skill on thread T1".

## Do not

- Create docs for roles the project does not need yet.
- Write private or sensitive data into files that will be committed.
- Start executing work — setup ends at a ready STATE.
