# Keyhole — research findings

Keyhole is a personal, fully local AI system. Its centrepiece is a
284B-parameter mixture-of-experts checkpoint (~13B active parameters
per token) that is far too large for the machine it runs on, so a
custom engine streams the checkpoint's weights through the GPU as
needed. The hardware is a single consumer Windows PC: an RTX 4060
with 8 GB of VRAM and 16 GB of system RAM. Nothing in the system
depends on a cloud service.

Around the big model — called **the Resident** in the project's own
vocabulary — sit smaller local models in defined posts (a fast
answerer, an orchestrator), a single web front door that routes
requests between them, and two persistence mechanisms that this
repository's findings are mostly about:

- **Casts** — exact saved states of the Resident's key/value cache
  for a text prefix. Resuming a cast skips re-reading a long
  document the Resident has already processed. Casts live on a
  shelf, are integrity-checked by hash before use, and fail closed
  (fall back to a full prefill) on any inconsistency.
- **The Lattice** — an index of the system's own past episodes,
  retrieved as context for new work.

## Method

Every result here was produced under the same discipline:

1. **Predictions and bars are filed before a run.** The threshold a
   candidate must clear — and the threshold that kills it — are
   written down in advance and not moved afterwards. A re-test is a
   new filing, never a quiet edit of the old one.
2. **Verdicts are read from the run's own artefacts**, not from
   memory or from a console scrollback.
3. **Bit-identity is the correctness gate.** A faster path is only a
   candidate if its outputs are bit-identical to the path it would
   replace, checked on the actual serving workload.
4. **Negative results are results.** Two of the six findings below
   are pre-registered candidates that were measured, failed their
   filed bars, and were closed. They are published in the same form
   as the passes.

Figures are quoted with their denominators. Where two numbers in
the underlying records describe different populations (for example
a format bake-off's store versus the later deployed store), the
findings say so rather than merging them.

## Contents

| # | Finding | One-line summary |
|---|---------|------------------|
| 01 | [Cast persistence, 2×2 factorial](findings/01-cast-persistence-factorial.md) | Bulk transfer dominates cast resume time; delta encoding's prize is bytes, not speed; the two prizes substitute, they do not stack. |
| 02 | [The cast-delta format](findings/02-cast-delta-format.md) | Per-layer XOR deltas against an acyclic parent forest cut the cast shelf from 477.2 MiB to 186.6 MiB, 12/12 casts bit-exact. |
| 03 | [Hot-set pinning: a negative result](findings/03-hot-set-negative-result.md) | Even an oracle-chosen pinned hot set of experts saves under 3% of streamed bytes — expert use is broad, not hot. |
| 04 | [Retrieval augmentation: a null result](findings/04-lattice-pair-null-result.md) | Retrieval precision was proven (20/24), but paired delivered-and-correct outcomes improved by +0.0 pp — the form was closed. |
| 05 | [Stem casts by truncation](findings/05-priming-stem-casts.md) | Truncating a minted cast at the divergence boundary yields a reusable prefix cast, bit-identical, saving 98.4% of re-read cost. |
| 06 | [Serving observations](findings/06-serving-observations.md) | Prompt caching verified byte-identical; a 2,000-token answer floor survives a 72-dispatch sweep; free-generation behaviour of the streamed checkpoint, as measured. |

## Status

Published November 2026.
All measurements were taken on the single machine described above,
on the workloads named in each finding; claims are fenced to what
was measured.
