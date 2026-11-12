# The cast-delta format

## The format

A cast is the saved key/value state of a long prefix — on this
system, routinely over 100 MB per cast. Many casts share most of
their content (the same documents, the same scaffolding), so the
shelf stores them as a forest:

- **Roots** are stored self-encoded (zstd, layer-major layout).
- **Children** are stored as per-layer XOR deltas against one
  parent cast, compressed with zstd at level 9.
- Parents are assigned greedily by measured saving, subject to
  the forest being **acyclic by construction** — every chain
  terminates at a root and decodes by resolving parents
  recursively.

## A structural finding from the bake-off

The first size estimate for this design was unachievable by *any*
decodable format, and the bake-off found out why: pricing mutual
deltas lets two casts be each other's parent, so a span shared by
a whole family is counted as stored nowhere. A decodable forest
must pay one real encoding of each shared family. On this
population that honesty cost ~38 MB and capped the true ratios at
**2.92× on the census subset and 2.35× on the full store**,
against the self-encoded baseline — real savings, honestly
denominated.

## Measured results (12 casts, 500.4 MB raw)

Format bake-off, all ten candidate formats bit-exact on
round-trip; the winner, delta-layer-9:

| Quantity | Value |
|---|---|
| Bake-off store total | 195.49 MB (195,486,678 B) |
| Self-encoded (zstd-3) store, same population | 459.44 MB |
| **Store saving (bake-off population)** | **263.95 MB** |
| Census subset | 75.98 MB vs 222.20 MB self-encoded |
| Mint overhead (writing a delta cast vs a plain save) | 1.039× |

Deployed conversion of the live shelf, a separate measurement on
a separate population: **477.2 MiB → 186.6 MiB** (payloads
500,386,820 B → 195,693,316 B). The bake-off and deployed totals
are close but are different quantities and are quoted separately
here, as in the underlying records.

- **Exactness:** 12/12 casts verified bit-exact through chain
  decode (per-layer tensor equality), both in the bake-off
  round-trip and in the deployed conversion.
- **Orphan bound K = 4:** computed from the deployed parent
  forest and later recount-verified. Losing the worst single
  parent file orphans 4 dependent casts (5 files undecodable if
  the lost file itself is counted). All other parents orphan at
  most 1.

## Integrity semantics

Each cast carries a manifest recording its token count, a hash of
its token ids, a SHA-256 of its payload bytes, and a **guard
key** derived from the generator build and the serving shape.
The loader:

- serves a cast only when the manifest's guard key matches the
  current build **and** the cast's token list is an exact prefix
  of the incoming prompt (longest matching prefix wins);
- verifies the payload hash **before** restoring any state —
  for a delta cast, every parent in the chain is verified the
  same way;
- **fails closed**: any inconsistency — wrong guard, hash
  mismatch, broken chain — means no cast is served and the
  prompt is prefilled from scratch. A corrupt shelf can cost
  time; it cannot change an answer.

## What the format is for, after the factorial

Finding 01 settled the division of labour: this format's return
is **capacity** — a shelf roughly 2.5× smaller, growing slower —
not resume speed. On the measured class it is slightly *slower*
to resume from than plain storage under the same restore path,
and that cost is the honest price of the bytes.
