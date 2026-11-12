# Retrieval augmentation: a null result, pre-registered

## The candidate

The Lattice indexes the system's own past episodes. The proposal
under test: prepending the retriever's top-3 episodes to the
answerer model's prompt improves its answers on new, sibling
tasks. The test was pre-registered — design, bars, and kill
conditions filed before any test dispatch ran.

## Design

- **Substrate pin:** a 128-episode index, pinned by content hash
  for the duration of the test.
- **Disjointness:** the 24 test tasks were all new, in four
  strata (exact-answer arithmetic, prose advice judged on a
  filed rubric, factual/reasoning with exact answers, and
  consistency screens with seeded contradictions). No task could
  duplicate or near-duplicate an indexed episode's task; this was
  enforced by a token-overlap check at set-build time, with the
  check output filed with the results.
- **Arms:** OFF = the task plus neutral filler context matched in
  token count (±10%) to the retrieved block; ON = the task plus
  the retriever's top-3 block, unedited. Equal-token arms, so a
  longer-context effect cannot masquerade as a retrieval effect.
- **Endpoint:** delivered-and-correct per task (delivered =
  non-empty content inside the serving budget; correct = exact
  match, or the filed rubric for prose/screen strata), plus
  confabulation strikes counted under a pre-existing definition.
- **Bars:** mean paired improvement ≥ +10 pp with zero added
  strikes earns the right to argue adoption. **≤ 0 pp, or strikes
  worse by ≥ 2, closes the form as final.**

## Rung 0 — retrieval precision, measured separately

Before any benefit test, a precision gate: can the retriever
find the right family at all? **20/24 hits** against a filed bar
of ≥ 12 — precision proven. (One stratum was structurally
unmeasurable in this index version — the index held a single
episode of that family, so 2-of-top-3 was impossible — and was
counted as misses anyway; the bar was still met.) Precision is
not benefit, and the registration said so: this gate only earned
the benefit test.

## Rung 2 — the paired result (48 dispatches)

| Stratum | OFF | ON |
|---|---|---|
| Arithmetic | 6/6 | 6/6 |
| Prose advice | 6/6 | 6/6 |
| Factual/reasoning | 6/6 | 6/6 |
| Consistency screens | 5/6 | 5/6 |
| **Total** | **23/24** | **23/24** |

**Mean paired improvement: +0.0 pp. Confabulation strikes: 0 in
both arms, delta 0.**

The two screen failures were complementary — each arm lost a
different task — and both were budget exhaustions in the model's
reasoning channel (empty delivery at the full token budget), not
retrieval effects.

## Verdict

**≤ 0 pp → test-landed final for this retriever/consumer form.**
Retrieved episodes produced no measurable delivered-and-correct
benefit on this sibling set: both arms were already at ceiling
on three of four strata, and the fourth stratum's losses were
budget burns in both directions. This closes the tested form —
this retriever, this consumer — not the idea of an episode
index; a different pairing would be argued on its own filing.

Two method notes, recorded in the original result: the task set
as built carried no trap-item marking despite the registration
describing four trap items, so the strike arm is weaker evidence
than designed (no fabricated claim was delivered in either arm,
so no strike could attach regardless); and the run's actual cost
came in at 0.27× its estimate, because most dispatches stopped
early rather than burning the full budget — a calibration
lesson the programme kept.
