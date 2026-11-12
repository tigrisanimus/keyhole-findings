# Stem casts: reusable prefix casts by truncation

## The problem

A cast is minted by running a prompt once and saving the
resulting key/value state. Minted naively, the cast contains the
mint prompt's *question* as well as its document — so it can only
ever serve that exact prompt again. A different question about
the same document diverges before the cast ends, and the loader
(correctly) refuses it: a cast serves only a prompt whose token
list begins with the cast's full token list.

## The construction

Given a minted full-prompt cast and two prompts that share a
document but ask different questions, define

**stem_len = min(LCP(mint prompt, prompt 2), LCP(mint prompt, prompt 3))**

— the longest common prefix, in tokens, across the uses the cast
must serve. The **stem cast** is the parent cast with each
layer's tensor truncated to `[:stem_len]`. Derivation is CPU-only
and takes seconds; no re-mint is needed. A constitutive check
confirmed all 43 layers of the stem are bit-equal to the parent
cast's corresponding slices.

In the measured instance: full cast 1,481 tokens; stem 1,454
tokens; the document-first re-use contract is *document verbatim
and first, then the question* (question-first compositions
diverge and miss, by the same prefix rule).

## Verification (one job, four arms)

1. **Full-cast anchor:** the original cast hits (1,481/1,481
   tokens, load 2.5 s) and reproduces the mint run's recorded
   greedy output.
2. **Stem on the mint prompt:** hits at 1,454/1,481 (load 1.8 s);
   the resulting distribution is **bit-identical** to the
   full-cast distribution — same greedy token, top-10 maximum
   relative delta 0.0.
3. **Re-use, a different question:** hits at 1,454/1,484. Wall
   214.5 s against a fresh-prefill estimate of 13,013.2 s —
   **98.4% of the re-read cost saved**, against a filed bar of
   ≥ 80%.
4. **Generality:** a third question, written after the mint,
   also hits (1,454/1,478). The stem serves unseen questions,
   not just the ones used to derive it.

**All gates pass; the stem cast was adopted as the shelf's live
entry for that document.** In practical terms, a ~3.6-hour
re-read became ~3.5 minutes of tail prefill. The full-prompt
parent cast remains on the shelf, superseded, not deleted.

## Why this works

Key/value state for a prefix depends only on that prefix. Any
truncation of a valid cast at a token boundary is therefore the
exact cast of the shorter prefix — there is nothing approximate
to verify beyond the slice equality and the end-to-end
distribution check, both of which were run. The contribution is
operational, not mathematical: choose the truncation point from
the *divergence boundary of the intended re-uses*, not from the
mint prompt, and a one-question artefact becomes a reusable
asset for a whole document.
