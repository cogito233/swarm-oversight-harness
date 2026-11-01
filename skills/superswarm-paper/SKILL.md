---
name: superswarm-paper
description: Pre-submission editing pass for research or benchmark papers written with coding agents. Use when the user says "pre-submission pass", "polish / edit my paper", "is this paper ready to submit", or asks agents to write, restructure, or tighten a research or benchmark paper.
---

# SuperSwarm Harness · Writing research / benchmark papers with coding agents

> **Where this fits**: an accept gate for the `research` profile. Once the work units (datasets, leaderboards, ablations) are accepted, use this pass to turn them into a submission. It does not replace the project loop; it is what "consolidate → accept" looks like when the output is a paper.
>
> **Scope**: only writing rules that hold for every paper. Nothing paper-specific (scores, model names, tickers) belongs here; those live in that paper's source files. Every example below is a one-line illustration, not a fact.
>
> Full rationale: [MANUAL](https://github.com/cogito233/superswarm-harness/blob/main/MANUAL.md).

One line: **write for the final paper, not the current snapshot.** Anything the final version would delete should not enter the shared draft now.

---

## Three core principles (everything else follows from them)

### 1. Write for the final paper

Every sentence you write should survive into the final version. Development mess stays out of the shared draft.

- **Missing data is a camera-ready scope statement, not a red `[TODO]`.** State scope in clean prose ("we report a preliminary slice of N models; the full matrix is left for camera-ready"). A reader who sees `[TODO]` marks the paper unfinished and starts doubting every claim.
- **Report the design, not the development log.** Bug-fix history, internal red-team rounds, harness refactors: these are how you got there, not the paper. Describe the system.
- **Frame the contribution at its final scale; label the current slice "preliminary".** The final scale is large (more models, more coverage); the current results are an honest slice of it. Say both, but position on the final scale.

### 2. Taste: novel vs. trivial

Separate what is genuinely new from narrow implementation choices dressed up as contributions.

- **Do not pad comparison tables with manufactured columns.** A column where only your row gets a ✓, for a trivial feature, reads as fake differentiation. Reviewers see it immediately.
- **Real novelty is often a combination, not a single axis.** Put the genuinely hard parts up front. If there is no single breakthrough, present the combination as the contribution.
- **Keep columns where competitors also earn a ✓.** That honesty is what makes "only we get all ✓" believable. A table made entirely of your unique columns looks defensive.

### 3. Debug the typeset PDF, not the source

Treat the paper as an artifact you render, screenshot, and iterate on visually.

- **Compile and read the PDF continuously** (a compile → read the PDF loop). Review the typeset result, not just the `.tex`. Overflowing tables, misplaced figures, broken equations only show up in the PDF.
- **Check every number, claim, and citation against the source of record before release.** Never let a reader find the first error. Anything a human reads as authoritative must be true.
- **Split large papers into per-section files** (`\input{sections/..}`) so several agents can each edit one section without overwriting each other. This is file-disjoint splitting applied to papers.
- **Build figures as artifacts too: HTML/CSS/SVG → headless screenshot → iterate (v1..vN).** Render with headless Chrome, read the screenshot, critique, revise. For clean technical diagrams, HTML gives more layout control than TikZ. **Do not use AI image generation for precise scientific figures** (it garbles text and numbers); at most use it for a background, then overlay exact labels. Watch the print page box (don't let a figure PDF get cropped or spill onto two pages). Use `figure*` / `\textwidth` (not `\columnwidth`) for wide figures; use `\resizebox` to keep wide tables inside the margins.
- **Split multi-agent editing by file.** Each parallel agent owns a disjoint set of section files, so there are no conflicts. **Cross-section edits** (a claim or caveat repeated across sections, or moving content to the appendix) must be done by a single agent, or you must assign one canonical home explicitly. Otherwise parallel agents will either all delete it (it ends up nowhere) or all keep it (it never gets removed). Read-only review agents, and non-paper figure or skill agents, can run safely alongside that one editing agent.

---

## Pre-submission checklist

Run this as an executable pass. If any item fails, do not submit.

### A. Final-paper framing and honesty
- [ ] **Zero red TODOs**: no `[TODO]` or red placeholder text in any shared version. Resolve it or move it to private notes.
- [ ] **Missing data is written as scope prose**, not placeholders.
- [ ] **No development log in the body** (bug history, red-team rounds, harness refactors).
- [ ] **The current slice is labeled "preliminary"**; the contribution is framed at final scale.
- [ ] **Show honesty, don't announce it**: delete defensive meta-text like "an honest result, not a defect", "we honestly note", "we do not claim". Collect real caveats in one clean **Limitations** section.
- [ ] **Overclaims leak upward; audit for them before submission.** Papers are often honest in Limitations but drop the caveats in the abstract, intro, and conclusion, where absolute claims survive. Run a **dedicated read-only overclaim + novelty audit pass**. Then propagate the body's caveats **up** into the abstract, intro, and conclusion, and **retire absolute words the evidence cannot support**: "calibrated" (needs a human or reference anchor), "invariant" (say "stable / preserved", scoped to the regime you actually tested), "best", "first / only". Scope every headline claim to its real basis (its n, a single run, a single condition). **Watch for near-circular validation**: an artifact distilled from the same source as a rubric cannot independently validate that rubric.

### B. Taste and argument
- [ ] **The comparison table is a feature matrix**: rows = systems, columns = meaningful axes, cells = ✓ / ✗ / partial. Standard shape, not prose in cells.
- [ ] **No manufactured differentiating columns**; keep columns where competitors legitimately get ✓, so "only we get all ✓" is credible.
- [ ] **Novelty claims rest on the genuinely hard parts or the combination**, not narrow implementation choices.
- [ ] **State ground-truth provenance up front**: when a benchmark grades quality rather than outcomes, say where the answer key comes from (e.g. "synthesized from ≥3 cross-checked human expert outputs"). "We benchmark against the best human work, not against the future."

### C. Writing habits to fix (from senior-editor critique)
- [ ] **Bold discipline**: top-venue papers rarely use bold. Use it only for contribution lead-ins and table headers / winners. Bold everywhere means nothing stands out. Let sentence structure carry the emphasis.
- [ ] **Cut the em-dash habit**: "main clause — aside — one more jab" is a first-draft rhythm. Turn about half of them into full stops.
- [ ] **Abstract discipline**: about 150 words. **No ground-level single numbers** (like "0.04 on X"). No implementation details. Introduce a named entity before using it (no bare lowercase model nicknames). Say only: what was built, the one-line core finding, and why it is novel.
- [ ] **Reduce neologisms and codenames; keep an academic register**: keep 3–4 load-bearing terms. Move internal item codes and internal nicknames to the appendix or drop them. The body stays in academic register.

### D. Consistency and structure
- [ ] **Consistency hygiene**: model-name casing, terminology, and citation rendering are uniform. Citations point to real third-party authors; **only self-citations are anonymized for blind review**.
- [ ] **Structure by genre**: a benchmark paper leads with **dataset + leaderboard**. Methodology and threat model / defenses are supporting, not co-headlines. **Do not give "Threat Model" its own top-level section**: fold the taxonomy into the metric motivation and the empirical audit into Experiments.
- [ ] **Explicit through-line between sections**: each section opens with a one-sentence bridge and a forward reference, so the paper doesn't read as randomly assembled parts.
- [ ] **Structural restraint**: ≤ 3 contributions. Merge Limitations into about 4 clean items, not 10 fragments. In a benchmark paper, Related Work can go at the end.

### E. Length and evaluation-reporting discipline
- [ ] **Floats drive length; the limit applies to the main body.** Cutting prose bottoms out quickly; total pages are really driven by **figures and tables**. To actually shorten, **cut or merge floats**, not just words. Measure **main-body pages** (excluding references and appendix; venue limits apply to the main body), not total PDF length.
- [ ] **Measure cross-section repetition with grep**: count how many times a key claim's signature phrase appears across sections (we once found one sentence repeated 27 times). Give every claim **one canonical home**. This is a measurable verbosity signal.
- [ ] **Do not trust any judge cell that hasn't passed a completeness check.** An LLM judge can emit empty or truncated verdict lists, and a "missing item counts as fail" backfill rule will **manufacture fake extreme scores** (we once hit a cell at −0.388 whose true value was ≈ +0.07). QC must flag empty or missing-layer outputs and re-judge them, verify completeness cell by cell, and use spot checks / case studies to catch such artifacts before trusting headline numbers. The authors must catch this first, not the readers.
- [ ] **Pin one canonical comparison set**: never compare means across different instance subsets (this repeatedly confuses readers, and you).
- [ ] **When judge invariance matters, report cross-judge consensus**, not a single same-vendor judge, as the "ground truth".
- [ ] **Failure analysis goes down to instance-level case studies** (broken down per model), not just aggregate per-layer tables.
- [ ] **Back up before any large regeneration / re-judging batch**: before a run that could overwrite already-analyzed results, snapshot the paper and data (sections / figures / PDF + score matrices + reports, as a **timestamped tar**). A bad batch must never be able to destroy **the headline state you have already analyzed**. Re-running is cheap; losing the analyzed headline is expensive.
- [ ] **Test a factor with a controlled re-run, not by overwriting the headline**: to test robustness to one variable (report language, judge vendor, reasoning effort), change only that factor on **the same instances** and write to **a different filename**, keeping the analyzed headline slice intact. Report the variant as a **controlled comparison**. Never overwrite the headline report to test a variant.

## Why this works (one line)

A paper is the final accepted output of the `research` profile, and **its readers are the verification gate**: a gate that never retries and discounts the whole paper after one error. So the whole point of this pass is to leave the reader **nothing from the development phase to grab onto**. Writing for the final paper stops you writing what will be deleted; taste stops you presenting trivial things as contributions; debugging the PDF stops the reader from finding the first error. Together they move a draft across the trust boundary into "trustworthy".

---

*General rules only: writing discipline that holds for every paper. Anything paper-specific (numbers, tickers, model names) lives in that paper's source files, not here.*
