# Planned experiments (deferred — to run after the writing pass)

Status: specs only. Do NOT run yet; we are focused on writing. When ready, run on
Snellius (A100) and fold results into the thesis sections noted below. All runs follow
the established conventions: `n=300` paired rollouts (3 `θ_true` seeds × 100), report
the `[sPCE, sNMC]` bracket at the per-system `L_eval` of `tab:config`, paired tests,
PACE-ftk unless stated. Numbers, once produced, go through the same single-source
discipline (land in `pace_theory_summary.tex` first, then transcribe).

Infrastructure to reuse (monorepo, not the thesis repo): NPE training + the Stage-D
runner `scripts/runner/run_bed.py`; sNMC re-scoring `scripts/analyze/rescore_snmc.py`;
system definitions `src/models/*.py`; SLURM templates `scripts/snellius/*.sbatch`;
diagnostics utilities used for the coverage / SBC / PSIS / pick-stability tables.

---

## A. Quality-vs-budget sweeps  → thesis §"Computational cost and the budget trade-off"

Goal: turn the cost-transfer claim (contribution 2) into a quantitative budget study.
We already have three fragments (now consolidated in the new Results section): the
cost-shift toy `N_NPE` curve on the 6-D MM cascade (Fig 4a), the DAD L-sweep on SEIR
(`tab:dad-sweep`), and the wall-clock table. The three sweeps below fill the untested
axes.

### A1. PACE quality vs NPE training budget on a *real* anchor  (highest value)
- **Why:** the only PACE quality-vs-`N_NPE` curve is on the toy; show the phase
  transition is not a toy artifact.
- **Systems:** CES (clean static, where PACE beats DAD definitively) and SEIR R=3 (stiff,
  saturated). Optionally ASM1.
- **Vary:** `N_NPE ∈ {1e2, 3e2, 1e3, 3e3, 1e4, 3e4, 5e4}` (simulation budget for NPE
  training). Train a fresh NPE per cell; everything else fixed at the `tab:config` values.
- **Measure:** PACE sPCE/sNMC at the system's `L_eval`; for reference, prior-sPCE at the
  same rollouts (should stay pinned at the ceiling). Plot sPCE vs `N_NPE` with the true
  EIG / DAD line for context.
- **Hypothesis:** PACE sPCE rises with `N_NPE` then plateaus at some `N* ` (expect a few
  thousand, as on the toy); prior-sPCE flat at `log(L+1)`.
- **Thesis slot:** main figure of the new budget section (companion to Fig 4a, real
  systems). Roughly 7 cells × (NPE train + 300-rollout eval) per system.

### A2. PACE quality and cost vs particle count J  (cheap; reassurance)
- **Why:** show the default `J=1000` is not over-tuned and expose the quality/cost
  frontier in `J`.
- **Systems:** CES, PI3K, ASM1 (one cheap, one mid, one stiff).
- **Vary:** `J ∈ {100, 200, 500, 1000, 2000}` at deployment (re-run PACE BED per `J`).
- **Measure:** sPCE/sNMC and median per-step wall-time vs `J`.
- **Hypothesis:** sPCE saturates by `J≈1000`; per-step time grows roughly linearly in
  `J` (cost is `J(1+M(1+C)N_cand)` solves).
- **Thesis slot:** a small table/figure in the budget section + a sentence justifying the
  `J` defaults.

### A3. PACE quality vs CEM search budget (N_cand, C)  (cheap)
- **Why:** justify the per-system `N_cand`/`C` choices, especially ASM1's `N_cand=50, C=0`.
- **Systems:** ASM1 (the outlier) and one non-stiff (e.g., source-loc p=2).
- **Vary:** `N_cand ∈ {50, 200, 1000, 5000}`, `C ∈ {0,1,2}`.
- **Measure:** sPCE/sNMC and per-step wall-time.
- **Hypothesis:** design quality saturates in the search budget; the chosen settings are
  on the plateau, so ASM1's small `N_cand` is not leaving performance on the table.
- **Thesis slot:** sentence + small table in the budget section (or appendix).

---

## B. Misspecification robustness  → thesis §"Robustness to misspecification" (new)

Goal: test the frozen, one-time NPE under model error. Two outcomes are both useful:
graceful degradation (a robustness result) or a detectable failure (the trust
diagnostics — PSIS `k̂`, raw coverage, per-step SBC — double as misspecification
detectors, making the stated NPE-mode-collapse failure mode concrete). Run B1 first.

### B1. Prior / observation-noise shift  (run first; low risk)
- **Why:** the safe robustness extension; the IS weights use the correct likelihood, so
  PACE should retarget as long as the NPE has support in the shifted region. Extends the
  SEIR robustness anchor (0/300 catastrophic) to "even under shift."
- **Systems:** SEIR R=3 (robustness anchor) and CES.
- **Two shift types:**
  - *Prior shift:* train NPE on the nominal prior `p(θ)`; sample `θ_true` from a shifted
    prior `p'(θ)` (e.g. shift each log-mean by +1 and +2 prior SDs; or widen σ by 1.5–2×).
  - *Noise shift:* train NPE assuming the nominal noise scale; deploy with the noise
    scale ×1.5 and ×2 (the likelihood in the weights is then mismatched).
- **Vary:** shift magnitude (a small grid, to get a degradation curve), and compare PACE
  (frozen NPE) vs DAD (policy trained on nominal) vs random on the shifted DGP.
- **Measure:** sPCE/sNMC, catastrophic-rollout rate, and the diagnostics (raw NPE
  coverage of the shifted `θ_true`, PSIS `k̂`, SBC) as a function of shift.
- **Hypothesis:** PACE degrades gracefully and stays non-catastrophic where the DAD
  policy degrades more; raw coverage drops smoothly with shift, flagging the regime.
- **Thesis slot:** a robustness subsection; one figure (sPCE / coverage vs shift) + a
  small table. Reuses existing NPE checkpoints + `run_bed.py` with a modified `θ_true`
  sampler / noise; no retraining for the prior-shift case.

### B2. Structural model gap  (run second; ambitious; ties to the project's roots)
- **Why:** closes the loop on the thesis's motivation (stiff biochemical systems; the
  original RSI/AMI misspecification framing). The NPE's training model omits a mechanism
  present in the data-generating process, so the likelihood in PACE's weights is wrong.
- **System + gap (pick one to start):**
  - SEIR R=3 **+ waning immunity** (add an `R→S` flow at rate `ω`); NPE trained on the
    no-waning model. Tunable severity via `ω` (0 → large) gives a degradation curve.
  - ASM1 **+ an unmodeled inhibition** process; NPE trained on standard ASM1.
  - PI3K / MAPK **omit a reaction** (cf. the okadaic-acid / PP2A example from the
    proposal): a clean structural-absence case.
  - Recommended start: SEIR + waning (cheapest simulator change, tunable severity).
- **Vary:** the misspecification severity (e.g. `ω ∈ {0, small, …, large}`).
- **Measure:** PACE sPCE/sNMC and selected designs vs severity; crucially, whether the
  diagnostics (PSIS `k̂ → ∞`, raw coverage → 0, SBC KS → 0.5) DETECT the gap before the
  designs become unreliable.
- **Hypothesis / two acceptable outcomes:** (i) graceful degradation (robustness result),
  or (ii) PACE breaks but the diagnostics flag it (detection result) — either is
  reportable, and (ii) makes the NPE-mode-collapse failure mode concrete.
- **Thesis slot:** second figure of the robustness subsection; possibly a short
  discussion connecting to RSI/AMI. More work: new/extended simulators in `src/models/`,
  NPE retrain on the base model, eval on the extended DGP.

---

## Notes
- Keep the cost-transfer / budget section anchored on existing results (toy, wall-clock,
  DAD sweep); the A-sweeps augment it.
- For misspecification, reuse the diagnostics machinery already built for the coverage /
  SBC / PSIS tables — the point is that those same diagnostics detect misspecification.
- Slot markers are left as `% TODO(planned: ...)` LaTeX comments at the relevant places
  in `sections/4-results.tex` so the structure is ready for the numbers.
