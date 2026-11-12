# Serving observations

Three smaller measurements from running the system day to day.
Each is fenced to the machine and workload named.

## 1. Prompt caching engages, and is byte-exact

Question: does the local inference server's prompt cache actually
engage under the house's production launch lines, for both the
answerer post and the orchestrator post? Method: native
completion endpoint, temperature 0, cache on; cold / exact-repeat
/ shared-stem-variant probes; the server's own timing fields are
the evidence. Filed bar: a warmed repeat of a ≥ 500-token shared
prefix saves ≥ 50% of that prefix's cold processing time, with
outputs byte-identical.

- **Answerer post** (1,247-token prompt, ~1,243-token shared
  stem): exact repeat used 281 ms of prompt processing against
  13,849 ms cold — **98% saved** — output byte-identical. A
  shared-stem variant saved 78%; the server's matcher stopped
  mid-stem on a repeated-paragraph stem, and even that partial
  boundary cleared the bar by half again.
- **Orchestrator post** (a real routing-judgement prompt):
  exact repeat 1.08 s wall against 6.72 s cold, byte-identical;
  a new-message variant saved ~92% of prompt processing — in
  production the orchestrator effectively re-reads only the new
  message, at ~1.1–1.3 s per judgement instead of ~6.7 s.

**Verdict: PASS, both posts, no launch-line changes needed.**
Prefix reuse at this layer was already working; it is now
measured working.

## 2. The answer floor: 2,000 tokens survives a sweep

The answerer post serves under a minimum generation floor — a
budget the model may always spend before the answer is cut off.
The question: could a lower floor (1,200) buy throughput, or
would a higher one (3,200) buy reliability? Method: 18 tasks in
three classes (structured-short, reasoning, prose-long), paired
across floors, 72 dispatches total, production prompt shape,
confabulation/refusal strikes counted throughout.

- **Zero strikes at any floor, in either stage.**
- **1,200 adopted nowhere:** in the reasoning class it lost both
  arithmetic items to cap starvation — the model's deliberation
  consumed the whole budget and the answer never surfaced
  (finish reason: length). At 2,000 both delivered.
- **3,200 adopted nowhere:** two classes crossed the filed +20%
  throughput band on paper, and both crossings were ruled
  sampling noise rather than floor effects — no causal path
  (answers in those classes landed far from either cap, so the
  budget was never the constraint) and sign inconsistency
  (the same classes measured in the opposite direction at the
  lower floor in the other stage). A threshold crossed by noise
  does not convert noise into an adoption; the raw numbers are
  kept in the record so the ruling is auditable.
- The one real floor interaction cuts the other way: across both
  stages, arithmetic items starved 3 times at floors ≤ 2,000 and
  0 of 2 at 3,200 — but on the filed throughput metric 3,200 did
  not beat 2,000, so the band keeps 2,000. That is recorded as
  the known, priced cost of the settled floor: deliberation
  length is high-variance, and an occasional long draw will
  still eat a 2,000-token budget.

**Verdict: the production floor stays 2,000 tokens for every
class** — a figure originally set by feel, now the survivor of
a filed sweep, with its failure mode known rather than
hypothetical.

## 3. Free generation from the streamed checkpoint

Observations on the Resident (the 284B checkpoint) under this
harness, from a frame × regime test matrix:

- **Frame matters more than regime.** A bare chat frame (system
  markers, question, open assistant turn) produces real content
  under both greedy decoding and the adopted seeded sampling
  regime (plain temperature 1.0). A tool/function-call frame
  produced **no content under any regime tested**.
- **Greedy long-form frays.** In 256-token cells, degeneration
  onset came at ~token 85 on worked arithmetic and ~token 160
  on prose — in both cases after a complete, correct opening
  stretch (the worked solution's first ~85 tokens were right).
  Sampling extends coherence; it does not abolish the pattern.
- **Consequence, adopted as house rule:** exact arithmetic is
  routed to a deterministic evaluator, never trusted to a
  generated span. The model's method and early partial products
  are typically exact; its long threads are not.

These are observations on one checkpoint under one harness,
recorded because they shaped the system's design (the front
door's framing, the evaluator rule), not as claims about the
checkpoint family in general.
