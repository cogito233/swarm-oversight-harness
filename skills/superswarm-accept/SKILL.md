---
name: superswarm-accept
description: Prepare a human acceptance decision in SuperSwarm Harness — check the evidence behind an output, run an adversarial red-team pass on it (or on a SPEC), and present a short accept / reject / revise brief. The agent never accepts on the human's behalf. Use when the user says "is this ready to accept/merge", "review before I sign off", "red-team this spec/result", "what's in the accept queue".
---

# SuperSwarm Harness · Accept gate

Acceptance moves an output across the trust boundary: draft → accepted. By profile that means branch → main, run → current best, or finding → accepted conclusion. Accepted content is trusted by default afterwards, so this gate is where the human's attention is worth spending. Your job is to make the decision fast and well-informed — **not to make it**.

Full rationale: [MANUAL.md](https://github.com/cogito233/superswarm-harness/blob/main/MANUAL.md) §4.

## 1 · Collect the candidate

From STATE's accept queue (or what the user points at): the output, its SPEC/TASK, the RESULT, and the verification evidence.

## 2 · Check the evidence, not the story

- Does each acceptance criterion have evidence a non-worker can re-check (command + observed output + path)? Re-run cheap checks yourself.
- Are the six stage facts stated separately (edited / committed / launched / ran OK / verified / accepted)? Anything inferred is `unknown`.
- Was verification done by someone other than the worker?
- Could the proxy have been gamed — deleted or weakened tests, skipped cases, hard-coded outputs, a judge from the same source as the generator?
- Are headline numbers taken from the source of record, not copied?

## 3 · Red-team

Spawn (or act as) an agent with **no stake** in the work. Its only job: list concrete ways this output — or the SPEC behind it — is wrong, overclaimed, or will break later. Exposure is the agent's job; the verdict stays with the human. Keep the 3 strongest findings.

## 4 · Brief the human (one screen)

```
Candidate: <what>          Profile gate: <tests / eval / red-team>
Criteria:  <n/m met, with evidence links>      Unknown: <list or none>
Strongest objections: 1) … 2) … 3) …
Reversibility: <cheap to undo? irreversible?>
Recommendation: accept | revise (what) | reject (why)
```

Depth follows reversibility: an irreversible change gets a deep check; an easily reverted change gets a light one. Occasionally sample raw code/data in depth to keep calibrating trust in the verifier.

## 5 · After the human decides

Only after an explicit human decision: record it (merge, promote the run, mark the finding `tier: accepted`), add a DECISIONS entry if it was a real decision, and update STATE. If rejected, the output simply stays outside the boundary — nothing accepted is touched.

## Never

- Accept, merge, or promote on your own judgment.
- Present the planner's summary as the evidence.
- Hide an unknown to make the brief look clean.
