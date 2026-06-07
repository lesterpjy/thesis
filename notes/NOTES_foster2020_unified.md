# Notes — Foster et al. 2020, *A Unified Stochastic Gradient Approach to Designing Bayesian-Optimal Experiments*

- **Venue / id:** AISTATS 2020. arXiv:1911.00294.
- **Authors:** Foster, Jankowiak, O'Meara, Teh, Rainforth.
- **Role in lineage:** **Origin of PCE and ACE by name.** Critically for PACE's prior
  art: ACE (the learned-proposal contrastive estimator) is here a **headline,
  empirically-evaluated method** — but in the **static / single-experiment** setting.

## Core idea
A single, unified stochastic-gradient objective that **jointly optimizes the design ξ
and the variational/proposal parameters φ** in one stage (no inner optimization loop).
The design is optimized by gradient ascent on a variational EIG bound.

## ACE — Adaptive Contrastive Estimation (Eq. 11)
Built by fixing the failure mode of VNMC's upper bound: VNMC under-estimates `p(y|ξ)`
when the L inner samples `θ_{1:L}` all miss the high-mass region. ACE **adds the
data-generating sample `θ_0` into the denominator sum**, turning the upper bound into a
**lower bound** that can be jointly maximized in `(ξ, φ)`:

```
I_ACE(ξ, φ, L) = E[ log  p(y|θ_0, ξ) / ( (1/(L+1)) Σ_{ℓ=0}^L p(θ_ℓ)p(y|θ_ℓ,ξ)/q_φ(θ_ℓ|y) ) ]
```
expectation over `p(θ_0)p(y|θ_0,ξ) q(θ_{1:L}|y)`. `θ_{1:L}` are the **contrastive
samples**, drawn from the **learned proposal** `q_φ(θ|y)`.

**Theorem 1 properties:**
1. Valid **lower bound**; error term is an expected KL `≥ 0`.
2. `lim_{L→∞} I_ACE = I(ξ)` (consistent).
3. **Monotonically increasing in L.**
4. If `q_φ(θ|y) = p(θ|y,ξ)` (perfect proposal), then `I_ACE = I(ξ)` for **all L** — no
   contrastives needed. *(This is exactly the identity PACE reuses: a perfect NPE
   ⇒ ESS = J, no IS correction.)*

The paper notes ACE "has not previously appeared in the BOED literature."

## PCE — Prior Contrastive Estimation (Eq. 12)
The special case of ACE where the proposal is fixed to the **prior** `p(θ)` instead of
a learned `q_φ`. Cheaper (no φ to learn), only tight as `L → ∞`, effective when prior
≈ posterior. **This is the bound DAD's sPCE is the sequential version of.**

The static analogue of PACE's saturation story lives here: PCE relies entirely on
contrastives, so it degrades exactly when the prior is a poor proposal for the
posterior-conditional marginal.

## Gradient estimation (Sec. 3.6) & iterated design (Sec. 3.7)
- **Reparameterized gradients** of `I_ACE` w.r.t. ξ (Eq. 18), with Rao-Blackwellization
  for discrete `y`. This is the backprop-through-simulator machinery that DAD inherits
  and that PACE's §5 contrasts against (forward-only CEM).
- **Likelihood-free ACE** is sketched here (precursor to iDAD's sLACE).
- **"Iterated experimental design with ACE":** a **greedy/myopic** sequential loop —
  run ACE, update the posterior, repeat — *not* an amortized policy. This is the
  closest the 2020 paper comes to the sequential setting, and it is greedy, not a
  trained policy and not history-amortized.

## Bottom line for PACE positioning
- **ACE with a learned proposal is evaluated prior art** — so PACE cannot claim the
  *idea* of a learned-proposal contrastive EIG estimator as novel. Concede this.
- **But it is static.** The 2020 ACE proposal `q_φ(θ|y)` conditions on a single
  outcome, is trained jointly with the design inside the EIG objective, and targets the
  single-experiment EIG. PACE's setting (sequential, per-step EIG **given history**,
  one-time SBI-NPE proposal, forward-only) is materially different, and the sequential
  learned-proposal estimator that would be the true precedent (sACE) is only stated as
  an unevaluated bound in DAD's appendix.
