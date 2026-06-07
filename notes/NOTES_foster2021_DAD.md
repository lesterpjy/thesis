# Notes — Foster et al. 2021, *Deep Adaptive Design (DAD)*

- **Venue / id:** ICML 2021. arXiv:2103.02438.
- **Authors:** Foster, Ivanova, Malik, Rainforth.
- **Role in lineage:** Lifts contrastive-EIG BOED from single-experiment to a fully
  **amortized sequential policy**. The headline method PACE benchmarks against.
- **sACE relevance:** the sequential learned-proposal bound is **Appendix B, Theorem 5
  — stated and proven, never implemented or evaluated.**

## Core method
Train a **design network (policy) `π_φ(h_{t-1})`** end-to-end to map an experiment
history to the next design. At deployment the policy is a **single forward pass
(~ms)** — no per-step optimization. The expensive part is **offline training**.

- A permutation-invariant encoder builds a representation `R(h_T)` of the history
  (sum-pooled per-step embeddings) — the architectural ancestor of PACE's
  set-equivariant NPE conditioner.
- Trained by maximizing **sPCE** over simulated trajectories.

## sPCE — sequential Prior Contrastive Estimation (the training objective)
For a trajectory `(ξ_{1:T}, y_{1:T})` with `θ_0, θ_{1:L} ~ p(θ)`:

```
Î_sPCE = (1/N) Σ_n log [ Π_t p(y_t|θ_0,ξ_t) / ( (1/(L+1)) Σ_{ℓ=0}^L Π_t p(y_t|θ_ℓ,ξ_t) ) ]
```

- Contrastives `θ_ℓ ~ p(θ)` (**prior**). Bounds the **total trajectory MI**
  `I(θ; y_{1:T} | ξ_{1:T})`, not per-step EIG given history.
- Ceiling `log(L+1)`. **This is exactly the saturation ceiling PACE's §3 analyzes:**
  on informative systems the prior contrastives miss the posterior region, the bound
  pins to `log(L+1)`, and the gradient vanishes — DAD's training loses signal.

## sACE — the learned-proposal sequential bound (Appendix B, Theorem 5)
> *Direct quotes — this is the crux of the prior-art question.*

**Main text (§4.1, after Eq. 15), hedged and explicitly deferred:**
> "If using a sufficiently large L proves problematic ... one can further tighten these
> bounds for a fixed L by introducing an amortized proposal q(θ; h_T) for the
> contrastive samples ... the proposal and the design network can then be trained
> simultaneously with a single unified objective, in a manner similar to a VAE ... The
> resulting more general class of bounds are described in detail in Appendix B and
> **may offer further improvements** for the DAD approach. **We focus on training with
> sPCE here in the interest of simplicity** of both exposition and implementation."

**Appendix B, Theorem 5:** states the **sACE lower bound** (Eq. 91) and the **sVNMC
upper bound** (Eq. 92), with proofs. The proposal `q(θ; h_T)`:
> "can be used to approximate the posterior ... One example ... would be an amortized
> variational approximation to the posterior ... It would be possible to share the
> representation `R(h_T)` ... between the design network and the inference network."

**Assessment:** bound + proof only. **No algorithm, no experiment, no evaluation.** The
proposal, where mentioned, is conditioned on the **full trajectory `h_T`** and intended
to be **trained jointly with the policy** to tighten the **trajectory-level** bound.
This is "here is something you could do" — a flag, explicitly not pursued.

## What DAD does NOT do (the gaps PACE fills)
- Never trains a learned proposal — all reported results use prior-contrastive sPCE.
- Targets total trajectory MI, not per-step EIG **given history**.
- Requires **backprop through the simulator** (adjoint) during policy training — the
  cost PACE's forward-only §5 contrasts against; on stiff ODEs (ASM1) and high
  `L_escape` systems (SEIR) this is what PACE argues is infeasible.
- Limited to low-D per-step designs (≈1–2D in published benchmarks).

## For the draft
DAD is the right primary baseline and the right place to anchor the saturation story
(sPCE ceiling = `log(L+1)`). When conceding prior art, cite **Thm. 5 + the deferral
sentence**: it shows the learned-proposal sequential bound was foreseen but left
unevaluated, which is the honest framing.
