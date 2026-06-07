# Notes — Ivanova et al. 2021, *Implicit Deep Adaptive Design (iDAD)*

- **Venue / id:** NeurIPS 2021. arXiv:2111.02329.
- **Authors:** Ivanova, Foster, Kleinegesse, Gutmann, Rainforth.
- **Role in lineage:** Extends DAD to **implicit / likelihood-free** models (simulator
  with no tractable density). Introduces **sLACE**, the likelihood-free sequential
  learned-proposal bound.
- **sLACE relevance:** **Appendix B, Proposition 4 — stated, proven, with a training
  recipe, but NOT evaluated.** The experiments use the prior-as-proposal special case
  (InfoNCE). One notch past DAD's sACE, still unimplemented as a *trained proposal*.

## Core method
Replaces the explicit-likelihood critic with a **learned critic `U_ψ(h_T, θ)`** so the
policy can be trained without evaluating `p(y|θ,ξ)`. Two bounds are used in practice:
- **InfoNCE** (self-normalized; bounded by `log(L+1)`),
- **NWJ** (unbounded but high variance).

**Experiments are all "iDAD (InfoNCE)"** — see the results tables. InfoNCE uses the
**prior** as the contrastive distribution.

## sLACE — sequential Likelihood-free ACE (Appendix B, Proposition 4)
> *Direct evidence for the prior-art question.*

The appendix derives the likelihood-free generalization of sACE:
```
L_sLACE(π,U,q;L) = E[ log  U(h_T,θ_0) / ( (1/(L+1)) Σ_{ℓ=0}^L U(h_T,θ_ℓ)p(θ_ℓ)/q(θ_ℓ;h_T) ) ]  ≤  I_T(π)
```
with a full proof, plus the tightness results (tight as `L→∞` for the optimal critic;
tight for **any L** if `q(θ;h_T)=p(θ|h_T)`).

**Training recipe (one paragraph, App. B):**
> "In practice, we parameterize the policy, the critic and the density of the proposal
> distribution by neural networks `π_φ, U_ψ, q_ζ` and optimize `L_sLACE` with respect
> to ... `φ, ψ, ζ` with SGA ... If ... we fix `q_ζ(θ;h_T) = p(θ)` instead of training
> it, then **we recover the InfoNCE bound**."

**The tell:** the evaluated method *is* the `q = prior` special case (InfoNCE). The
paper writes down how you'd train `q_ζ` but does not report doing so. So sLACE is
"here is an extension, and here is how you'd train it" — **not** "here is an extension
we evaluated." Recipe-level, unevaluated.

Also notes the practical costs of the learned-proposal version: "introducing another
set of optimizable parameters ζ" and that "the sLACE bound assumes that we know the
prior `p(θ)` and can evaluate its density."

## How this differs from PACE
| | iDAD / sLACE | PACE |
|---|---|---|
| Proposal `q` | jointly trained inside the BED objective (when trained at all; in practice fixed to prior = InfoNCE) | one-time **NPE trained independently via SBI**, frozen for design |
| Target | total trajectory MI | per-step EIG **given history** |
| Likelihood | likelihood-free (learned critic) | explicit likelihood (kinetic ODE simulator) |
| Deployment | policy forward pass | forward-only per-step acquisition (CEM), no policy |

## For the draft
- iDAD is the natural cite for "the learned-proposal sequential bound exists, but was
  left at recipe level and evaluated only in its prior-proposal (InfoNCE) special case."
- iDAD's InfoNCE = sLACE with `q=p(θ)` is also a clean way to state that **the
  evaluated sequential methods all use prior contrastives** — which is precisely the
  regime PACE's saturation lemma indicts.
- iDAD is implicit-likelihood; PACE is explicit-likelihood. If a reviewer pushes the
  sLACE comparison, the likelihood-free vs SBI-NPE distinction plus the
  target/training differences above are the response.
