# Notes — Foster et al. 2019, *Variational Bayesian Optimal Experimental Design*

- **Venue / id:** NeurIPS 2019. arXiv:1903.05480 (v3, Jan 2020).
- **Authors:** Foster, Jankowiak, Bingham, Horsfall, Teh, Rainforth, Goodman (Oxford / Uber AI / Stanford).
- **Role in lineage:** The foundational paper. Introduces the *variational* approach to
  EIG estimation that everything downstream (PCE/ACE, DAD, iDAD, and PACE's evaluation
  estimators) builds on. Does **not** yet use the names "PCE"/"ACE" — those are coined
  in Foster 2020.

## Problem
EIG (Eq. 1) is doubly intractable: the posterior `p(θ|y,d)` and the marginal `p(y|d)`
are both unavailable. Classical nested Monte Carlo (NMC) is `O(N·M)` and converges at
`O(T^{-1/3})`. The paper replaces inner integration with *amortized variational
inference*, learning a network once and reusing it across outcomes `y`.

## The estimator taxonomy (the lasting contribution)
1. **Variational posterior `µ̂_post` (Barber–Agakov / "BA" bound).** Learn
   `q_φ(θ|y,d) ≈ p(θ|y,d)`; gives a **lower bound** `EIG ≥ L_post`, tight iff the
   variational family contains the true posterior. This is the bound DAD's "BA bound"
   refers to.
2. **Variational marginal `µ̂_marg`.** Learn `q_m(y|d) ≈ p(y|d)`; gives an **upper
   bound**. Attractive when θ is high-dimensional (estimate the marginal instead of
   the posterior).
3. **Variational NMC `µ̂_VNMC`.** Combines a learned proposal with nested MC to remove
   the bias of (1)/(2): asymptotically consistent **even when the variational family
   does not contain the target**. Converges `O(T^{-1/2})` when the family does contain
   it. This is the direct ancestor of the ACE/sVNMC machinery.
4. **Implicit-likelihood variants.** Two of the estimators work without an explicit
   likelihood — the seed of iDAD's likelihood-free direction.

## Why it matters for PACE
- The **sPCE lower bound** and **sNMC upper bound** that PACE uses to *evaluate* EIG
  (the `[sPCE, sNMC]` brackets in the results table) are the sequential descendants of
  the lower/upper bound pair introduced here (posterior-style lower, NMC-style upper).
- Establishes the central idea PACE depends on: **amortize the inner inference once,
  reuse it across many outcome draws.** PACE's NPE is exactly an amortized posterior
  network `q_φ(θ|h)` of the BA/`µ̂_post` type — but trained by SBI rather than by
  maximizing an EIG bound.

## What it does NOT do
- Single-experiment (static) only; no sequential policy, no history conditioning.
- No contrastive-sample bound by that name yet (no PCE/ACE).
