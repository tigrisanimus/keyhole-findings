# Hot-set pinning: a negative result

## The candidate

The Resident's checkpoint is a mixture-of-experts model: 43
layers × 256 experts = an 11,008-expert pool, each expert
25.2 MB in its stored (FP8) form. Because the model is streamed,
an obvious optimisation is to **pin a hot set** — keep the most
frequently used experts resident and stream only the rest —
against the incumbent policy of ordinary least-recently-used
retention under the same total budget.

## Method

A $0 offline replay over a recorded routing trace: 6 prompts,
127 generated tokens, 32,766 expert-selection events. Two pinning
arms, both judged against a same-budget LRU:

- **Oracle** — the hot set chosen from global frequencies over
  the whole trace. This is an upper bound no real deployment can
  have, since it peeks at the future.
- **Causal** — the hot set chosen from the first half of the
  trace, judged on the second half.

Filed bars, set in advance: at the real budget (the machine's
16 GB RAM / 8 GB VRAM envelope, with casts and KV state
competing), projected byte saving **≥ 15%** *and* projected
speed ceiling **≥ 1.10×**. Both, or the candidate closes.

## Result

| Budget | Oracle saving | Oracle ceiling | Causal saving | Causal ceiling |
|---|---|---|---|---|
| ≈8 GB (325 experts) | 2.8% | 1.029× | 1.1% | 1.011× |
| ≈16 GB (650 experts) | 0.4% | 1.004× | 0.0% | 1.000× |

Both arms, both budget points, fall short of both bars by a
factor of 5–30. Pinning only approaches the bars at budgets far
below the real one, where the incumbent is starved — not the
filed comparison.

## The quantity underneath

The trace touches **6,472 distinct (layer, expert) keys — 59% of
the entire expert pool — in 127 tokens**. Each token selects
~258 experts, ≈ 6.4 GB of weights. Expert use on this substrate
is **broad, not hot**: there is no small set to pin. The bytes
wall for a streamed MoE of this shape is the breadth of expert
use, and breadth is not a residency problem.

## Verdict and caveat

**Closed, test-landed final.** Even with oracle knowledge of the
future, a pinned hot set cannot beat ordinary retention by the
filed margins at the real budget, so no live benchmark was
filed. The caveat stands on record — the trace is small
(6 prompts, 127 tokens) — but the oracle arm's margin (5–30×
under the bars) leaves no plausible rescue in a larger trace.
