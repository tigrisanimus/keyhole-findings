# Cast persistence: format × restore path, a 2×2 factorial

## Question

A cast resume has two independent design choices: the **storage
format** (plain, one self-contained payload per cast, versus a
delta format in which a cast is stored as the difference from a
parent cast) and the **restore path** (per-row, restoring the
cached state row by row into the engine, versus bulk, transferring
each layer's state in a single operation). Which choice costs the
time, and do the two compose?

## Design

- Workload: resuming one production cast — a 1,454-token prefix
  cast (a "stem cast"; see finding 05) for a 1,465-token prompt.
- Arms: A = plain + per-row; B = delta + bulk (a composed loader
  built for this test, transplanting the bulk restore block into
  the delta-capable loader); C = delta + per-row (the loader in
  production at the time); D = plain + bulk.
- 3 repetitions per arm, arm order rotated; the timed wall is the
  loader's full resume call, cast selection included.
- Identity leg: each arm, after resuming, generates 11 further
  steps, compared exactly (greedy choice and full top-10) against
  the pre-registered baseline output.

## Result

| Arm | Format × restore | Reps (s) | Median |
|-----|------------------|----------|--------|
| A | plain + per-row | 2.003 / 2.606 / 2.634 | **2.606** |
| B | delta + bulk | 1.100 / 1.217 / 0.993 | **1.100** |
| C | delta + per-row | 2.481 / 3.000 / 3.247 | **3.000** |
| D | plain + bulk | 0.302 / 0.335 / 0.460 | **0.335** |

**All four arms resumed at the correct token and produced
bit-identical output.** The ordering is not a median artefact: D
is fastest in every rep, B beats C in every rep, and the B−D gap
is positive in every rep.

## The filed prediction, scored as filed

The prediction written down before the run, in bounded form:
**B ≈ D within 0.15 s, and both beat C by ≥ 1.0 s** (with a third
clause expecting the plain per-row arm, A, to be slowest).

**Verdict: partially refuted.**

- Both-beat-C: **confirmed**, with margins of 1.900 s (B) and
  2.665 s (D) on the medians.
- B ≈ D within 0.15 s: **refuted.** The gap is 0.765 s, and the
  rep distributions are disjoint.
- A slowest: **refuted.** C — the production configuration — is
  the slowest arm.

## Decomposition

Timed sub-legs explain where the walls go. On the delta path the
loader reads *fewer* bytes and pays a *cheaper* read+hash leg
than on the plain path (stem delta payload 21,515 B; its parent
119,521,803 B, read+hash 0.064 s; plain payload 128,060,363 B,
read+hash 0.101 s). The codec term is therefore **compute, not
bytes**: the parent's load and decompression chain.

Fitting the standard two-term form (an object-count term for the
per-row restore, plus a transfer/codec term):

- The object-count term measures **2.271 s (plain) / 1.900 s
  (delta)**, implying **36 / 30 µs per restored row**.
- The additive form does **not** close. Predicting B from the
  other three arms gives 0.729 s against a measured 1.100 s —
  an **interaction residual of +0.371 s**. The delta penalty
  grows once bulk transfer removes the per-row term that was
  masking it: +0.394 s under per-row restore (C − A), +0.765 s
  under bulk (B − D).

## The headline

Bulk transfer is the dominant term in cast resume time. Delta
encoding is a **net time cost at either restore path** on this
workload class — its prize is bytes (finding 02), and it may not
be quoted as a speed technique. The two prizes **substitute;
they do not stack to parity**: the composed delta+bulk path takes
the bulk prize against production (3.000 s → 1.100 s) but does
not reach plain+bulk (0.335 s).

## A settled estimate, corrected by this measurement

An earlier figure in the programme — that the delta format would
save 1.61–2.31 s per resume — came from an unpaired comparison
against a production load *range*. The factorial's paired per-row
arms settle that estimand directly: **−0.394 s per resume**
(C 3.000 vs A 2.606). The estimate was wrong in sign, and the
derived sessions-to-payback figure for the delta adoption was
discharged on that basis: on speed alone, there is no payback;
the adoption's return is the byte saving.

## Scope

One cast class (a 1,454-token, depth-1 delta child with a
~120 MB parent), one machine, one sitting. The decomposition and
the substitution claim are stated for that class; deeper chains
and other parent sizes are not measured here.
