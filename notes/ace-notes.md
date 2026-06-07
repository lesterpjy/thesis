# Literature — contrastive-EIG lineage for PACE

This folder collects the prior-art papers most relevant to PACE's contribution and
positioning, with detailed per-paper notes. All four papers are from the
Foster/Rainforth (Oxford) line of work and trace the lineage of the contrastive
EIG estimator that PACE builds on.

## Files

| PDF | Notes | What it is |
|---|---|---|
| `foster2019_VBOED_ACE.pdf` | `NOTES_foster2019_VBOED.md` | Foster et al., NeurIPS 2019. Origin of variational EIG estimators (posterior/BA, marginal, VNMC, implicit-likelihood). |
| `foster2020_unified_PCE_ACE.pdf` | `NOTES_foster2020_unified.md` | Foster et al., AISTATS 2020. **Origin of PCE and ACE by name.** ACE (learned proposal) is a headline, **evaluated** method — but **static / single-experiment**. |
| `foster2021_DAD.pdf` | `NOTES_foster2021_DAD.md` | Foster et al., ICML 2021. Deep Adaptive Design: amortized sequential policy trained on sPCE. **sACE (sequential, learned proposal) is Appendix B only — bound+proof, not evaluated.** |
| `ivanova2021_iDAD.pdf` | `NOTES_ivanova2021_iDAD.md` | Ivanova et al., NeurIPS 2021. Implicit/likelihood-free DAD. Experiments use InfoNCE (prior proposal). **sLACE (sequential, learned proposal) is Appendix B only — bound+proof+recipe, not evaluated.** |

## The lineage in one paragraph (relevant to PACE's prior art)

The learned-proposal contrastive EIG estimator is **not** merely an appendix flag in
general: in the **static, single-experiment** setting it is **ACE** (Foster 2020), a
fully-developed and empirically-evaluated headline method. What was left undone is the
**sequential** learned-proposal estimator. DAD's sACE (App. B, Thm. 5) and iDAD's
sLACE (App. B, Prop. 4) state and prove the sequential bounds but **neither is
implemented or evaluated** — DAD explicitly defers it ("we focus on sPCE here ... for
simplicity"), and iDAD's experiments use the prior-as-proposal special case (InfoNCE).

## What this means for PACE's novelty claim

Three things separate PACE from the closest prior art, none of which appear in the
sACE/sLACE appendices:

1. **Target.** sACE/sLACE bound the **total trajectory MI** (prior-marginal
   denominator, measure p(θ)). PACE targets the **per-step EIG given history**
   (posterior-predictive denominator, measure p(θ | h_{t-1})). This retargeting — not
   the estimator algebra — is the non-obvious conceptual move; neither DAD nor iDAD
   makes it (they stay at trajectory MI).
2. **Proposal training.** sACE/sLACE train the proposal **jointly with the design
   policy inside the BED objective**. PACE uses a **one-time NPE trained independently
   via SBI** (forward KL / max-likelihood on simulated pairs), decoupled from any BED
   loop. (Klein 2026 / Zaballa 2025 instead retrain the posterior online each step.)
3. **Saturation diagnosis.** The L_escape ≈ exp(Λ) threshold and its empirical map are
   absent from all four papers.

**Honest positioning:** concede the bound-level overlap loudly (static ACE is
evaluated prior art; sequential sACE/sLACE are proven-but-unevaluated appendix
bounds), then rest PACE's claim on the retargeting (1) + the saturation theory (3),
with the estimator as the vehicle, not the headline. "Getting it to train" (2) is real
but is the kind of contribution reviewers discount in isolation.

## Provenance
- DAD / iDAD PDFs from arXiv:2103.02438 / 2111.02329.
- Foster 2020 from arXiv:1911.00294; Foster 2019 from arXiv:1903.05480.
- Notes written from direct reads of the appendices and estimator sections (June 2026).
- Not yet downloaded (titles unconfirmed in our notes): Klein et al. 2026, Zaballa &
  Hui 2025 (the two online-retrained-NPE comparators). Worth adding once identified.
