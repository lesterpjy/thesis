# MSc Thesis Plan — PACE

Planning document for `docs/thesis/draft/msc_thesis.tex`. Not the thesis text — the
structure + narrative, to be reviewed/edited before drafting begins.

---

## 0. Ground rules, sources of truth, constraints

**Method name.** Use **PACE** (Posterior-Amortized Contrastive Estimation) throughout.
- ⚠️ `neurips/neurips_2026.tex` and `result_registry.md` use the OLD name **APCE** and the
  title "Amortizing Past the Saturation Wall." Treat their *prose* as drafts, not authority.

**⚠️⚠️ SINGLE SOURCE FOR ALL NUMBERS (hard rule, user-mandated 2026-06-07).**
`docs/amort_paper/pace_theory_summary.tex` is the **ONLY** source for every numerical result, metric,
statistic, and result table in the thesis. This includes `perstep_decomposition.tex`, which is `\input`
into the summary (line 713) and is therefore *part of* the single source.
- Every reported empirical/performance number — sPCE/sNMC bounds, Δ values, p-values, `L_escape,50`,
  ESS trajectories, ΔS_t, wall-clock times, diagnostic numbers (coverage, SBC KS, pick-stability,
  catastrophic rates) — and every result table (`tab:results`, `tab:results-rand`, `tab:ablation-grad`,
  `tab:escape`(+`-full`), `tab:dad-sweep`, `tab:ess-full`, `tab:raw-coverage`, `tab:sbc-perstep`,
  `tab:walltime`, `tab:config`) is transcribed from the summary, full stop.
- **ALL other documents are writing/context aid ONLY and must NEVER supply a numerical value or
  empirical result:** `result_registry.md` (provenance/trace only), `wall_time_analysis.md` (narrative
  framing only — the numbers live in summary `tab:walltime`), `neurips_2026.tex`, `theory_lecture_notes.md`,
  `guided-rewrite.tex`, `proposal/`. Use them for prose, structure, intuition, and citations — not data.
- If a number is needed but absent from the summary, do NOT pull it from another doc: flag it as a gap
  and ask, or leave a `% TODO(verify against summary)` placeholder. Never silently import.
- **One unavoidable exception — system *definitions* (not results):** the summary does not contain the
  ODE equations / rate constants / prior bounds / initial conditions, so the Systems appendix (App I)
  sources those from the simulator **code** (`design_amortization/systems/*.py`, etc.), cross-checked
  against the summary's `tab:config` for the meta-parameters (`d_θ`, `d_ξ`, `T`, `J`, `N_cand`, `C`,
  `L_eval`, `L_escape,50`). These are experimental *setup*, not empirical *results*.

**✅ Two sanctioned exceptions (user-approved 2026-06-07) — extract from `neurips_2026.tex` §exp-anchors.**
The summary does not report these figure-supporting numbers, so they may be taken from the NeurIPS draft:
1. **Cost-shift toy** (`fig4a_eig_convergence`, §4.2). 6-D Michaelis–Menten cascade, 4-OOM log-uniform
   prior; sweep `N_NPE` from `10^2` to `5×10^4`, fresh NPE per cell, 50 held-out targets, `L=2000`,
   `J=200`. Result: prior-IS sPCE stays pinned at the ceiling (~6 nat above true EIG) for all `N_NPE`;
   PACE descends to the true EIG between `N_NPE = 500` and `5000`.
2. **"% of rollouts at the ceiling"** (§4.3 anchors). CES: 76% (PACE) at `L_eval=10^5` → 41% (PACE) /
   4% (DAD) at `L_eval=10^7`. ASM1: 97% (PACE) pinned at `L=5000` (ceiling 8.52); `L_eval=5×10^4`
   ceiling 10.82. SEIR R=3: 97%/95% (PACE/random) at `L_eval=10^5` → 52%/36% at `L_eval=10^7`
   (ceiling 16.85).

   **⚠️ Entanglement caveat (critical):** in neurips these percentages appear in sentences that ALSO
   carry SUPERSEDED performance deltas (CES +1.06; ASM1 +0.95; SEIR +0.60-vs-random / appeared-tied).
   Take ONLY the ceiling percentages from neurips; the performance Δ's come from the summary — CES
   **+0.93** vs DAD, ASM1 **+0.89** vs random, SEIR **+7.31** vs DAD and **+0.62** vs random
   (`tab:results`: 14.78−14.16). Reconcile the two — never transcribe a neurips anchor sentence whole.

   These two are the ONLY sanctioned non-summary numbers. Everything else stays summary-only.

**⚠️ Superseded numbers NOT to reuse (old → authoritative):**
| Quantity | Old (neurips/registry) | Authoritative (summary) |
|---|---|---|
| CES PACE vs DAD | +1.06 | **+0.93** (definitive †) |
| ASM1 PACE vs random | +0.95 | **+0.89** |
| SEIR R=3 PACE vs DAD | +0.60 | **+7.31** (mean; DAD tail-pulled) |
| Monod PACE vs DAD | −0.56 | **−0.66** (DAD wins) |
| Src-loc p=5/p=8 | T_bed=15 truncation values | **T=30** values (p5 +1.07; p8 −0.34 ftk / +0.61 grad) |
| PK 1-comp T=3/T=5 | +0.13 / −0.07 (n=45) | **−0.01 / +0.07** (DAD-matched n=300, ties) |
| Step-DAD PK 1-comp | +0.75 | re-check; summary de-emphasizes Step-DAD |

**Hard constraints (from template README + stubs):**
- **35-page limit**, counted **from Introduction → Conclusion**; references + appendix excluded.
- Thesis must be **self-contained without the appendix** (appendix = "bonus / read more").
- Keep the 6-chapter skeleton: Introduction · Related Work · Methodology · Results · Discussion · Conclusion (+ Appendix). **Do not add/remove chapters.**
- **Single column** (see §2 for the mechanical edits).

**Memory-flagged guardrails (apply when writing):**
- N<45 sPCE claims are unreliable; only report n≥300 multi-seed paired results in the main body
  (`mc_noise_retractions`). PK-1comp/PI3K had n=45 PACE cells — the summary's n=300 DAD-matched
  reruns are the ones to cite.
- Figures: simulated-data-grounded, correctness-first, YlGnBu palette, **no titles** (caption carries
  meaning) (`figure_preferences`).
- SEIR ftk-vs-DAD comparison is unpaired-in-θ in one table — preserve the summary's caveat wording.

---

## 1. The narrative spine (the thing to foreground)

The user's priority: the **four contributions AND the mathematical connection between them** must be
central (the adviser's `guided-rewrite.tex` under-emphasized this). The spine below is the connective
tissue — one mathematical object playing four roles.

**One quantity, four roles.** The per-contrastive likelihood ratio
`r_ℓ = p(y|θ_ℓ)/p(y|θ_0)`, whose prior expectation is `E[r_ℓ] ~ e^{-Λ}` with
`Λ = ½ Σ log(1+λ_i)` (prior-scaled Fisher information = posterior↔prior volume ratio), is the
single object that threads all four contributions:

1. **Saturation (C1).** sPCE `= log(L+1) − log(1+Σr_ℓ)`. When the posterior is a tiny fraction of
   the prior, `Σr_ℓ → 0`, sPCE pins at `log(L+1)`, and its **gradient vanishes at the same rate**.
   Escape needs `L ≳ e^Λ` → infeasible on stiff/broad-prior systems. *(driver = `r`/`Λ`)*

2. **PACE + cost transfer (C2).** Draw contrastives from a one-time NPE posterior `q_φ` instead of
   the prior; SNIS weights `w ∝ p(θ)p(h|θ)/q_φ` retarget to the posterior. The likelihood factor
   `p(h|θ)` is *exactly what cancels* the divergence that doomed prior contrastives. But as the
   posterior tightens, `χ²(p‖q_φ)` grows and **ESS collapses** — the *same* concentration that caused
   saturation, now relocated from policy training to amortized inference. *(same cause, moved)*

3. **Validity under collapse (C3).** As `ESS→1`, PACE algebraically reduces to a weighted sum of the
   *same* ratios `r_{k←dom}`; **common-mode cancellation** (the divergent weight scale enters only
   design-independent terms and cancels) preserves the design ranking, and the criterion **interpolates
   smoothly to a Fisher-information / D-optimal criterion** `½ tr(F Σ_post)` — i.e. `Λ` from C1
   *reappears as the design objective*. *(same object becomes the criterion)*

4. **Empirics (C4).** Across 12 systems, the **crossover (PACE vs DAD) is predicted by `L_escape`
   (the C1 quantity), NOT by ESS (the C2/C3 quantity).** ESS collapse is survivable; saturation of the
   training signal is decisive. *(loop closes: C1's object is the operative predictor)*

**This synthesis gets its own named subsection** at the end of Methodology ("§3.x One quantity, four
roles") and is previewed in the Introduction. It is the thesis's intellectual signature.

**Hero figure for the spine:** `fig_a1_hero.pdf` (3 panels) is the graphical abstract — (a) sPCE
flatlines at `log(L+1)` while PACE tracks true MI past `t_sat`; (b) post-saturation, sPCE scores all
designs flat while PACE still ranks them; (c) PACE yields the tighter final posterior at `θ*`.

---

## 2. Global mechanics

**Single-column conversion (mechanical):**
- `msc_thesis.tex:58` — delete `\twocolumn`. (Class defaults to one column; the `\if@twocolumn`
  machinery at cls:82–105 is just `\chapter` plumbing, harmless.)
- Abstract `adjustwidth{4cm}{4cm}` (msc_thesis.tex:43) — reduce to ~{1cm}{1cm} or remove for a
  single column.
- `sections/4-results.tex` — change the demo `figure*` (spanning) to `figure`.
- Appendix `7-appendix.tex` — leave the `\onecolumn` note alone; whole doc is single column now.
- Everything else (lists of figs/tables, natbib, amsthm) is column-agnostic.

**Body-vs-appendix policy (the core compression strategy for 35pp):**
- **Main body** = prose + the *load-bearing* equations/lemmas/figures + the n≥300 result tables.
  State each theorem/lemma and give the one-line intuition; cite the appendix for the proof.
- **Appendix** = every step-by-step derivation, the full 12-system specs, all config/diagnostic/
  wall-time tables, the per-step and ESS trajectory tables. This is a near 1:1 lift of the
  `pace_theory_summary.tex` appendix (§A–I there → App. A–I here).
- The promotion the user asked for ("appendix sections of the summary → thesis body") is satisfied by
  moving the *conceptual content* (saturation mechanism, PACE definition, low-ESS reasoning, NPE/SBI
  setup) into the body as **exposition**, while the *derivations* stay in the appendix.

**Page budget (single column, 11pt, target ≤35):**
| Chapter | Pages | Notes |
|---|---|---|
| 1 Introduction | 4–5 | incl. hero figure + contributions + RQs |
| 2 Related Work | 3–4 | three-family organization |
| 3 Methodology | 13–15 | preliminaries + C1 + C2 + C3 + optimizer + eval + synthesis |
| 4 Results | 7–9 | C4: suite, anchors, cross-system, per-step/ESS, diagnostics |
| 5 Discussion | 2–3 | when-to-use, limitations, ethics |
| 6 Conclusion | 1–1.5 | answer each RQ |
| **Total** | **30–37** | trim Methodology first if over (push more derivation to appendix) |

**Budget note (post-decisions, 2026-06-07):** page length is explicitly NOT a constraint right now
(user: "handle page length later if we're over"). Preliminaries (§3.1) and Diagnostics (§4.6 + App F)
are being *expanded*, not trimmed. Two full derivations live in the body (§3.2.1 saturation form +
§3.4.1 ESS→1 limit) plus the preserved background derivations of §3.1. The 35-page target is recorded
for a later compression pass; do not drop content to hit it now. When we do compress, levers in order:
push background derivations → a dedicated appendix or tighten; Related Work → 3 pp; Discussion → 2 pp.

## 2b. Unified notation, symbols, conventions (REQUIREMENT)

A single notation must be used everywhere; the source docs disagree and must be reconciled to ONE
convention. Three homes, all consistent:
1. **Frontmatter `sections/0-list-symbols.tex`** — the authoritative List of Symbols (align table) +
   List of Abbreviations. Every symbol used in the thesis appears here once.
2. **In-body §3.1.0** — a compact working-notation table introduced before first use (body is
   self-contained without frontmatter).
3. **This spec** — the canonical convention to write against.

**Canonical convention (resolves cross-doc clashes):**
- History `h_t = {(ξ_i, y_i)}_{i=1}^t`, `h_0 = ∅`; params `θ`, prior `p(θ)`; design `ξ_t`; obs `y_t`.
- Per-step EIG `I_{h_{t-1}}(ξ_t)`; posterior predictive `p(y_t | ξ_t, h_{t-1})`.
- **Contrastives (sPCE/sNMC eval, DAD training):** index `ℓ = 0..L`, `θ_0` data-generating,
  `r_ℓ = p(y|θ_ℓ)/p(y|θ_0)`. Always distinguish `L_train` (DAD) vs `L_eval` (scoring).
- **PACE particles:** index `j,k = 1..J` from `q_φ`; weights `w_j`, normalized `w̃_j`; `ESS`; `M`
  inner obs draws; `N_cand`, `C` (CEM rounds), `ρ` (elite frac).
- Fisher/Lindley: `Λ`, eigenvalues `λ_i`, `F`, `Σ_0`, `Σ_post`; threshold `L_escape,50`.
- Per-step: `S_t` = sPCE on length-`t` prefix; `ΔS_t = S_t − S_{t-1}`.
- Low-ESS decomposition: `Ψ_j`, `Φ̄(ξ)`, weight entropy `H(w̃)`.
- Diagnostics: `χ²`, `D_KL`, PSIS `k̂`, SBC KS-distance.
- Conventions: information in **nats** (log base e); **sPCE = lower bound ↑**, **sNMC = upper bound ↓**;
  `†` = definitive ordering (one bound clears the other); main-body results are `n=300` paired unless
  noted; figures YlGnBu, no titles.
- **Reconciliations:** lecture-notes `l`/`δ_l` → use `ℓ`/`δ_ℓ`; lecture-notes "PCE (upper bound)" is the
  per-step analogue of what we call **sNMC** — present it as sNMC; neurips `\hist` → `h`; "APCE" → PACE.

---

## 2c. Writing style (REQUIREMENT, user 2026-06-07)

Applies to all thesis prose: body, appendix, captions, abstract, and research-question statements.
1. Plain, straightforward academic English. Precise and scientific. The goal is to get the message
   across, not to impress.
2. No fancy or ornate diction. Prefer common words over elaborate ones.
3. No em-dashes. Use commas, colons, parentheses, or separate sentences instead. (En-dashes only for
   numeric ranges, e.g. 3.6-12.8 min.)
4. Simple sentence structure. Short to medium sentences, one idea each. Avoid long subordinate-clause
   chains and nested parentheticals.
5. No hype or rhetorical flourish. Avoid words like "strikingly", "dramatically", "remarkably".
6. Active, direct voice where natural. Define each term on first use. Keep terminology consistent with
   the §2b notation.
7. The source docs use colorful or dense phrasing in places (e.g. "forward-only thesis in miniature",
   "Rube Goldberg", "two rulers", "the saturation wall"). Do NOT copy that phrasing into body prose;
   rewrite the idea plainly. Such labels may appear at most as a short figure or section name, not in
   running text. This is a phrasing directive only; it does not change any claim or number (§0 still governs).

**Figures (all already in `docs/thesis/draft/figures/` unless noted):**
| Figure | File | Chapter / Section | Role |
|---|---|---|---|
| Hero (3-panel a/b/c) | `fig_a1_hero.pdf` | 1 Introduction | Graphical abstract of the spine |
| L_escape heatmap | `fig_a1_heatmap.pdf` | 3 Methodology §saturation | `L_esc~e^Î` grows per step; DAD-solves axis = infeasibility |
| Prior-vs-posterior contrastives | `fig_a1_contraction.pdf` | 3 Methodology §PACE | Why posterior contrastives don't waste (source-loc 2D, t=2/6/10) |
| L-scan transitions | `fig3_lscan_transitions.pdf` | 4 Results §saturation-suite | Empirical escape across 5 representative systems |
| Cost-shift convergence | `fig4a_eig_convergence.pdf` | 4 Results §estimator-validation | PACE→true EIG as N_NPE↑; sPCE pinned (toy) |
| Anchor rollout deficits | `fig4b_anchors_violin.pdf` | 4 Results §anchors | ASM1 + SEIR per-rollout deficit distributions |
| Per-step ΔS_t + ESS | `outputs/paper_figures/b7_perstep_eig_ess/fig_b7_*_{CES,SEIR_R_3}.pdf` | 4 Results §per-step | ΔS_t decays while ESS collapses, PACE keeps lead (copy 2 into figures/) |

Optional: a small schematic of the PACE pipeline (NPE once → particles+weights → forward-only CEM)
in Methodology if space allows — would need to be made; not strictly required.

**Tables (authoritative source = `pace_theory_summary.tex`):**
| Table | Chapter | Body or App | Notes |
|---|---|---|---|
| `tab:results` (PACE-ftk vs DAD vs random, 8 sys) | 4 Results | **Body** | headline; keep † definitive markers |
| `tab:results-rand` (ASM1, Goodwin) | 4 Results | **Body** | no-DAD-comparator systems |
| `tab:ablation-grad` (grad vs ftk) | 4 Results | **Body** (compact) | isolates optimizer; PI3K story |
| `tab:dad-sweep` (SEIR L-sweep, catastrophic rate) | 4 Results | **Body** (compact) | robustness headline |
| `tab:escape` (5-system thresholds) | 3 Methodology | **Body** (compact) | full `tab:escape-full` → App |
| `tab:perstep-dS` | — | **App E** | summarize 3 case studies in body prose |
| `tab:ess-full` | — | **App E** | summarize in body prose |
| `tab:raw-coverage`, `tab:sbc-perstep` | — | **App F** | cite key numbers (raw≥70–90%, IS collapses) in body |
| `tab:walltime` | — | **App G** | compact 3-row version may go in Discussion |
| `tab:config`, eval-estimator defns, system specs | — | **App H** | |

---

## 4. Chapter-by-chapter outline

### Chapter 1 — Introduction  (target 4–5 pp)
Goal: motivate → name the technical obstacle → state the 4 contributions + their connection → RQs.

- **1.1 Motivation: timed-input / sequential experiments on stiff scientific systems.**
  Open with the adviser's strongest hook (sequential BED is most valuable where each measurement is
  expensive and priors are broad: wastewater ASM1, cell-free biosynthesis, epidemics, PK, bioprocess;
  *when* you intervene matters, not just *how much* — cite Jakstaite2026 timed-batch, van Sluijs2024).
  Source: `guided-rewrite.tex` §1; `neurips_2026.tex` §1 ¶1; proposal §1.1–1.3 (cell-free framing).
  **DECISION POINT A** (framing emphasis) — see §7.
- **1.2 The obstacle: prior-contrastive saturation.** Amortized policy BED (DAD & descendants) trains
  against the prior-contrastive sPCE bound, capped at `log(L+1)`; on high-information systems the
  signal — value *and gradient* — vanishes, and the escape budget grows like `e^Λ`. Per-trajectory
  SBI methods avoid this but need simulator adjoints, unstable/infeasible on stiff ODEs. → the gap.
  Source: summary §1, §intro; neurips §1 ¶3–4.
- **1.3 Contributions.** The four (verbatim spine from §1 above), each one sentence, explicitly noting
  the connecting object (`r`/`Λ`/`χ²`). Forward-reference the synthesis section.
- **1.4 Research questions** (see §6). Main RQ + RQ1–RQ4 mapped to the four contributions.
- **1.5 Thesis outline.** One paragraph.
- **Figure:** `fig_a1_hero.pdf` as the overview/graphical abstract.

### Chapter 2 — Related Work  (target 3–4 pp)
Per template: *announce the gap, end with the gap; not a laundry list.* The adviser flagged the
NeurIPS related-work as "too much short-paragraph form" — fix by leading with the closest work, then a
thematic sweep. Organize by **what cost each method pays for the saturation diagnosis** (neurips §6
"three families" is the cleanest scaffold).

- **2.1 Closest work (named, contrasted explicitly):** DAD/iDAD/Step-DAD (Foster, Ivanova, Hedman);
  Klein 2026 (per-trajectory SBI-BOED with simulator gradients). State precisely how PACE differs:
  forward-only, one-time SBI proposal, posterior contrastives. *This is the head-to-head.*
- **2.2 Family A — methods paying the per-step contrastive-escape cost** (all prior-IS-sPCE-trained:
  DAD, iDAD, Step-DAD, JADAI, RL-BOED, vsOED, ALINE/TNDP, value-based). Lemma applies uniformly.
  Note Foster's appendix sACE/sLACE *anticipates* the SBI-side variant and defers it → PACE realizes it.
- **2.3 Family B — posterior-guided / per-trajectory SBI design** (Klein, Iollo, Bharti, Zaballa);
  contrast: they retrain a posterior per step; PACE trains once.
- **2.4 Family C — optimization through stiff simulators** (adjoint/vanishing-gradient pathologies;
  Kim, Pal, Kidger) → motivates forward-only.
- **2.5 Thematic context** (one tight paragraph): SBI/NPE (Cranmer, Greenberg, Durkan); contrastive MI
  bounds + the `log N` ceiling (McAllester & Stratos, Poole); timed-input/autonomous experimentation
  (van Sluijs, Jakstaite). End by restating the gap.
- Source: neurips §6; summary citations; proposal §2.1–2.6 (reuse SBI/BOED survey paragraphs + ~90
  refs in `proposal/bibentries.bib`); literature memory note.

### Chapter 3 — Methodology  (target 13–15 pp) — the heart; absorbs the "knowledge setup"
Structure: preliminaries → C1 → C2 → C3 → optimizer → evaluation → synthesis. (Folding preliminaries
into Methodology, rather than a new Background chapter, respects "don't change the structure";
template explicitly allows setup as Methodology subsections.)

- **3.1 Background & preliminaries.**
  **Body-vs-appendix principle (user, refined 2026-06-07):** novelty decides depth.
  (i) Do NOT re-derive textbook material (Bayes' rule, KL/MI definitions, flow/SGD mechanics) — assume
  an informed reader. (ii) For *standard but non-obvious* results (entropy telescoping → trajectory MI /
  DAD Thm 1; PCE/sPCE/sNMC bounding-property proofs; sNMC Jensen) — **state the result + one-line
  intuition in the body, full derivation → App A.** There is no reason to spend body pages re-deriving
  established results. (iii) Reserve full in-body derivations for the *novel* contributions (saturation
  form §3.2.1, ESS→1 limit §3.4.1, and the C2/C3 reasoning). Primary derivation source for App A:
  `theory_lecture_notes.md` Parts I–II; cross-check summary §setup + neurips §2.
  - **3.1.0 Notation & conventions** — the compact working-symbol table (see §2b).
  - **3.1.1 Sequential BED & EIG.** Problem setup; information gain as KL; per-step EIG as conditional
    MI `I_{h_{t-1}}(ξ_t)` (Eq. step-eig); prior- vs posterior-predictive denominator. Myopic vs
    lookahead and why amortize — *briefly*. [LN I.1–I.5]
  - **3.1.2 Policy methods & the trajectory MI.** Policy idea (history→design network) → trajectory
    simulation. **State** the total-EIG-of-a-policy result (trajectory MI, DAD Thm 1) and that the
    trajectory marginal `p(h_T|π)` is intractable; **entropy-telescoping derivation → App A** (standard
    result, Foster 2021). [LN II.1–II.6]
  - **3.1.3 Contrastive estimation: PCE, sPCE, sNMC.** **State** the estimators and the key facts the
    rest of the thesis leans on: the `log(L+1)` ceiling, sPCE = lower bound ↑, sNMC = upper bound ↓,
    bracket `E[sPCE] ≤ I ≤ E[sNMC]` with both → I as L→∞. The `log(L+1)` ceiling argument is one line
    (denominator ≥ `p(y|θ_0)/(L+1)`) and can sit in the body since it directly motivates C1; the
    **full bounding proofs (InfoNCE lower bound; Jensen upper bound) → App A.** [LN II.7–II.9 + summary
    app:eval-estimators]
  - **3.1.4 sPCE as the policy training signal.** The trajectory sPCE bound is what DAD (and all policy
    methods) maximize, at `O(L·T)` per gradient step — the hook into C1. (State; cost detail in C1.)
    [LN II.10–II.11]
  - **3.1.5 Simulation-based inference & NPE.** What NPE learns, why amortize once, set-equivariant
    history conditioning, variable-length training (concise; full architecture → App J). [summary
    §setup, App I]
- **3.2 Contribution 1 — Contrastive saturation** (~3 pp).
  - 3.2.1 Saturation form — **full derivation in body** (Lemma: `sPCE = log(L+1) − log(1+S)`; the
    proof is elementary and load-bearing, and is the novel C1 lens — body is right). Gradient vanishing:
    state the result + intuition; full bound → App D (gradient-vanishing).
  - 3.2.2 Why prior contrastives fail: `E[r_ℓ]~e^{-Λ}`, escape needs `L≳e^Λ` (state via Laplace; full
    derivation → App B). Define empirical `L_escape,50`.
  - 3.2.3 Mechanisms that worsen it (broad priors, high-dynamic-range maps, non-Gaussian noise) —
    prose list; derivations → App C.
  - **Table** `tab:escape` (compact). **Figure** `fig_a1_heatmap.pdf`.
  - Source: summary §saturation; lecture-notes Part III (rich intuition: "wide prior + sensitive
    simulator → posterior is a tiny fraction of prior → … → DAD can't learn").
- **3.3 Contribution 2 — PACE estimator & cost transfer** (~3 pp).
  - 3.3.1 Definition (Eq. weights, Eq. pace); each particle is generator + contrastive; SNIS targets
    the posterior predictive. Relation to sACE/sLABEL (one paragraph).
  - 3.3.2 Consistency (Theorem, state C1–C3; proof → App). IS-weight identity: a *perfect* NPE gives
    ESS=J at any concentration (Eq. w-identity) — so collapse ⇒ benign mismatch, not failure.
  - 3.3.3 Cost transfer / ESS degradation: `ESS≈J/(1+χ²)`; the burden moves from policy training to
    amortized inference; `N~exp(KL)` for value estimation is infeasible → but design needs *rankings*.
  - **Figure** `fig_a1_contraction.pdf` (prior vs posterior contrastives).
  - Source: summary §pace; lecture-notes Part IV (cost-shift intuition, χ² cancellation).
- **3.4 Contribution 3 — Design selection under low ESS** (~2.5 pp).
  - 3.4.1 Decomposition `Î = H(w̃) − Φ̄(ξ)` → argmax depends only on `Φ̄`; **ESS→1 limit derived in
    full in body** (reduces to a weighted sum of likelihood ratios — the second load-bearing
    derivation). Pairwise/KL-discrimination special case + longer algebra → App E.
  - 3.4.2 Common-mode cancellation: divergent weight scale enters only design-independent terms.
  - 3.4.3 Smooth interpolation (multi-particle EIG ↔ dominant-particle LR-test, no mode switch) and the
    KL/Fisher connection `→ ½ tr(F Σ_post)` (D-optimal) — *this is where Λ returns as the criterion*.
  - 3.4.4 Failure modes: identifiability plateau (Goodwin) and NPE mode collapse (diagnosable).
  - Source: summary §low-ess; lecture-notes Part V ("Rube Goldberg" objection+response is good defense).
- **3.5 Forward-only design optimization** (~1.5 pp). Why no simulator backprop; the CEM algorithm
  (Alg. ftk); per-step forward-solve cost `J(1+M(1+C)N_cand)`; when gradient ascent helps/hurts
  (preview PI3K). Source: summary §forward-only.
- **3.6 Evaluation protocol** (~1 pp). sPCE (lower) + sNMC (upper) bracket the true EIG; "definitive
  ordering" rule; what is bounded (prior-contrastive per-step IG, shared scale across methods);
  per-step chain-rule ΔS_t (telescopes to trajectory sPCE); diagnostics used (ESS, PSIS, SBC, raw
  coverage, bootstrap pick-stability). Systems suite overview (12 systems, one line each; full specs →
  App I). Source: summary §eval-estimators, perstep_decomposition.tex.
- **3.7 Synthesis — one quantity, four roles** (~0.75 pp). The §1 spine made explicit and tight: how
  `r`/`Λ`/`χ²` link C1→C2→C3 and set up C4's predictor. **The user's centerpiece.**

### Chapter 4 — Results  (target 7–9 pp) — Contribution 4
> **Guardrail:** every number in this chapter must appear in `pace_theory_summary.tex`, with EXACTLY two
> sanctioned exceptions from `neurips_2026.tex` §exp-anchors (see §0): the cost-shift toy numbers (§4.2)
> and the "% of rollouts at the ceiling" stats (§4.3). For the latter, take only the percentages from
> neurips — the performance deltas still come from the summary (CES +0.93, ASM1 +0.89, SEIR +7.31/+0.62).
> Any other figure-supporting number not in the summary → keep prose qualitative or `% TODO(verify)`.
- **4.1 Saturation across the 12-system suite.** L-scan + `L_escape,50` ranges; Λ as reference not
  predictor (Laplace breaks on Hill/SEIR). **Figure** `fig3_lscan_transitions.pdf`. Compact escape
  table or point to §3.2's.
- **4.2 Estimator validation (cost-shift toy).** PACE→true EIG as N_NPE grows; prior-sPCE pinned.
  **Figure** `fig4a_eig_convergence.pdf`. Numbers (6-D MM cascade; N_NPE 10²–5×10⁴; descends to true
  EIG between 500–5000; ~6 nat ceiling gap) from neurips §exp-anchors — **sanctioned exception (§0).**
- **4.3 Anchor systems (where saturation bites).** CES (+0.93 vs DAD, definitive), SEIR R=3 (+7.31,
  0/300 catastrophic vs DAD 0.7–6.3%), ASM1 (+0.89 vs random, DAD infeasible), Monod (−0.66, DAD wins,
  honest counter), Goodwin (−0.48 counter-case). **Tables** `tab:results` + `tab:results-rand` +
  `tab:dad-sweep`. **Figure** `fig4b_anchors_violin.pdf`. Use sPCE/sNMC brackets to mark definitive
  orderings. The "% at ceiling" stats (CES 76%→41%/DAD 4%; ASM1 97% pinned; SEIR 97%/95%→52%/36%) are
  the **sanctioned neurips exception (§0)** — pair them with the summary deltas above, NOT the neurips
  deltas they're written next to.
- **4.4 Cross-system benchmark & optimizer ablation.** Source-loc p=2/5/8, PK 1/8-comp, PI3K; crossover
  tracks `L_escape`, not d_θ or ESS. PACE-grad vs ftk (`tab:ablation-grad`): gradient helps only where
  the simulator gradient is reliable (src-loc p=8) and *hurts* on stiff PI3K — "forward-only thesis in
  miniature."
- **4.5 Per-step information & the ESS decoupling.** ΔS_t decays with posterior concentration, not ESS;
  three case studies (SEIR gap grows as ESS→1; PI3K keeps high ESS yet favors DAD; CES 48% of info at
  ESS≤5). **Figure** `fig_b7_*_{CES,SEIR_R_3}`. "ESS does not predict performance" — and we deliberately
  do NOT report a single cross-system correlation (not sign-stable). (full tables → App F.)
- **4.6 Diagnostics: why trust the results despite ESS collapse** (NOT brief — this is the evidentiary
  core; user-flagged). Frame it as a defense: ESS→1 *looks* alarming, so each diagnostic must rule out
  a specific failure mode. Open with the perfect-NPE identity (a perfect NPE keeps ESS=J at *any*
  posterior concentration, Eq. w-identity) ⇒ collapse implies a *small, benign* proposal mismatch
  amplified by concentration, not proposal failure. Then five diagnostics, each tied to a question:
  1. **Does the design pick survive collapse?** Bootstrap pick-stability — 45% top-1 agreement, ≈23×
     the `1/N_cand` chance baseline ⇒ common-mode cancellation (C3) holds operationally. (Tell the
     honest story from LN VII.4: predicted ≥80%, observed 45%, reinterpreted as a biased-but-noisy
     per-step pick that aggregates to consistent wins.)
  2. **Is it the NPE or the IS resampling that breaks?** Raw NPE coverage (before reweighting) stays
     70–90% at the 90% level through the final step, while IS-resampled coverage collapses to ~30–49%
     ⇒ the collapse is an IS-resampling artifact, not a miscovering proposal.
  3. **Is the proposal calibrated at the *deployment* histories (not just on average)?** The
     marginal-vs-conditional calibration subtlety: SBC is average-case over the prior predictive;
     raw-coverage-at-deployment-histories is the conditional evidence SBC can't give. Explain this
     carefully — it is the crux of the trust argument.
  4. **Is calibration stable across steps?** Per-step SBC (raw) flat/improving even where ESS<10; IS-SBC
     degrades (again, the IS artifact).
  5. **Is there gross mode collapse?** PSIS Pareto-`k̂` and raw coverage as mode-collapse detectors;
     our values rule it out (SBC KS ∈ [0.13,0.18], coverage ≥70%).
  Also state the smooth-interpolation evidence (>100× ESS/J range traversed without mode-switching).
  Present headline numbers in body; full protocols/tables → App G (expanded beyond the summary).
- Source: summary §diagnostics + LN Part VII (richer exposition); numbers per §0 hierarchy.

### Chapter 5 — Discussion  (target 2–3 pp)
- **5.1 When to use PACE.** Broad-prior, high-information, stiff, low/moderate-dim continuous design;
  honest about where learned policies win (unsaturated, high-dim combinatorial src-loc).
- **5.2 Practical feasibility.** NPE 3.6–12.8 min once; per-step <4 s on 11/12; DAD training 7 min–16 h
  –infeasible. The ms-vs-seconds latency gap is irrelevant at scientific timescales. Compact wall-time
  numbers (full `tab:walltime` → App H). **Numbers from summary `tab:walltime` + §forward-only ONLY;**
  `wall_time_analysis.md` is narrative framing only, not a number source.
- **5.3 Limitations** (template asks for reproducibility/scalability/generalizability/validity):
  forward-only optimizer doesn't scale to high-dim combinatorial; PACE values not calibrated EIG under
  collapsed ESS (rankings only); NPE mode-collapse risk on strongly multimodal posteriors; Laplace Λ is
  qualitative on stiff systems; some PACE cells were n=45 (now superseded by n=300 DAD-matched).
- **5.4 Ethics / broader impact** (template requires a mention): efficient design reduces wet-lab/
  animal/compute cost; dual-use caution for epidemic-intervention design; misspecification honesty.

### Chapter 6 — Conclusion  (target 1–1.5 pp)
Answer each RQ in order; restate gap→contribution; qualify with limitations; future work (learned
proposal + posterior-contrastive scoring; y-conditional inner proposals; explicit timed-input
biochemical benchmarks). Source: summary §summary, neurips §discussion.

---

## 5. Appendix map (the "bonus" — unlimited)
- **App A — Standard background & estimator derivations (for completeness, not novel).** These are
  textbook/established results stated in the body and derived here: trajectory-MI via entropy
  telescoping (DAD Theorem 1, Foster 2021); PCE/sPCE/sNMC bounding properties (`log(L+1)` ceiling,
  InfoNCE lower-bound interpretation, Jensen upper-bound argument for sNMC); the per-step chain-rule
  telescoping `Σ_t ΔS_t = sPCE_T`; extended sPCE/sNMC bias notes. (LN II.4, II.7–II.9 + summary
  app:eval-estimators + perstep_decomposition.tex)
- **App B** Linearized EIG (Lindley/Laplace) derivation + escape threshold `e^Λ`. (summary App A; LN App A)
- **App C** Mechanisms inflating `L_escape` (broad priors / high-DR maps / non-Gaussian noise). (App B)
- **App D** Full gradient-vanishing derivation. (App C; LN III.2b)
- **App E** PACE decomposition, ESS→1 limit, KL-discrimination / Fisher criterion. (App D; LN V.4–V.6)
  — note: the *novel* core (saturation form, ESS→1 limit) is in the BODY; this appendix holds the
  longer algebra and the pairwise/KL special case.
- **App F** Supplementary tables: full escape table, DAD L-sweep, ESS trajectories, per-step ΔS_t. (App E + perstep_decomposition.tex)
- **App G — Diagnostics, expanded** (NOT a terse lift; user-flagged). For each of the five diagnostics:
  full protocol, the failure mode it controls for, the result table, and a careful reading of *why it
  licenses trusting the picks under low ESS*. Bootstrap pick-stability; raw-vs-IS coverage; per-step
  raw-vs-IS SBC; the marginal-vs-conditional calibration argument (the key subtlety); ESS-regime
  interpolation; PSIS `k̂` mode-collapse check. (summary App F + LN Part VII; expand the explanation.)
- **App H** Wall-clock cost comparison. (numbers from summary `tab:walltime`/App G ONLY;
  `wall_time_analysis.md` = narrative aid, not a number source.)
- **App I — Systems: full specifications (NEW dedicated appendix; user-flagged).** Every system used
  must be explicitly defined. One subsection per system, covering EVERY system that appears in any
  table/figure — the 12 evaluated (CES, Monod, Goodwin, ASM1, SEIR R=3, PI3K, PK 1-comp, PK 8-comp,
  source-loc p=2/5/8) PLUS the saturation-table-only ones (DC motor, Haldane, PK transit, Repressilator,
  SEIR R=5, SEIR R=8). For each: literature source/citation; `d_θ/d_ξ/d_y`; the ODE system (or analytic
  forward map for CES / source-loc); fixed constants; parameter priors; initial conditions; observation
  & noise model; design space; horizon `T`; and the per-method config (`J`, `N_cand`, `C`, optimizer,
  `L_train`/`L_eval`, `L_escape,50`) — fold in `tab:config`.
  **Source of equations:** extract from code `design_amortization/systems/{asm1,goodwin,pi3k,pk_1comp,
  pk_8comp,seir_multiregion,source_location,base}.py` and `experiments/dad_pytorch/{ces_pyro,monod_motor,
  goodwin_pyro,asm1_pyro,pi3k}.py`; cross-check each against its literature source (Henze ASM1;
  Foster CES; vsOED/Shen source-loc; Monod; Goodwin; PI3K). See `benchmark_systems` memory.
- **App J** NPE architecture & training (set-equivariant encoder, NSF, hyperparameters, data-gen
  posterior-correctness argument). (summary App I)
- (Optional) **App K** sACE/sLACE relationship in full. (LN IV.5b)

---

## 6. Research questions (draft — map 1:1 to contributions)
- **Main RQ.** Can sequential Bayesian experimental design be made *feasible and effective* on stiff,
  broad-prior dynamical systems where amortized policy methods fail?
- **RQ1 (diagnosis / C1).** When and why does the prior-contrastive sPCE objective — the training
  signal common to all amortized-policy BED — fail, and can the failure be quantified a priori?
- **RQ2 (estimator / C2).** Can a one-time simulation-based posterior estimator supply contrastives
  that escape saturation, and what new cost does this introduce?
- **RQ3 (validity / C3).** Does design selection remain valid when the importance weights collapse
  (ESS→1)?
- **RQ4 (empirical / C4).** Across a broad system suite, when does forward-only posterior-contrastive
  design match or beat amortized policies, and what predicts the crossover?

(Replaces the obsolete proposal RQs about α-activation mechanism identification.)

---

## 7. Decisions (LOCKED 2026-06-07)

- **A/B — Positioning: Problem-first, method-centric.** Introduction opens with the timed-input /
  stiff-system motivation (adviser's hook), pivots to the saturation obstacle, then the 4-contribution
  spine. Thesis stays a methods thesis about PACE; biochemical + other systems are the proving ground.
  Domain motivation is *light throughout* (no heavy cell-free/MAPK foregrounding; the α-mechanism
  proposal work is out of scope for results).
- **C — Theory depth: 1–2 key derivations in body.** Saturation form (§3.2.1) and the ESS→1 limit
  (§3.4.1) are derived in full in the body; all other derivations (gradient-vanishing, Laplace/`e^Λ`,
  KL-discrimination, PACE decomposition algebra) stay in the appendix.
- **D — Per-step in body.** §4.5 ΔS_t / ESS-decoupling + the b7 figures (CES, SEIR R=3) stay in Results.

**Other things to verify before/while writing:**
- **Single-source compliance is non-negotiable (see §0).** Before stating any number, confirm it is in
  `pace_theory_summary.tex`. The ONLY exceptions are the two sanctioned neurips-anchors items (§4.2
  cost-shift toy; §4.3 %-at-ceiling) recorded in §0 — and for those, only the percentages/toy-config
  come from neurips while the performance deltas come from the summary (watch the entanglement caveat).
  Any other not-in-summary number → qualitative prose or `% TODO(verify against summary)`.
- The per-step ΔS_t table is `perstep_decomposition.tex` (PART of the summary): transcribe it as-is. If
  its PK/src-loc grad rows are known-stale (registry note), that is a fix to make IN THE SUMMARY first —
  do NOT re-derive numbers from `outputs/` for the thesis.
- Confirm final Step-DAD treatment using the summary only (registry shows inconsistent Δ across older
  tables; the summary de-emphasizes Step-DAD — follow the summary).
- Assemble a real `references.bib` (draft stub has 2 entries; harvest from `proposal/bibentries.bib`
  + the paper .tex bibitems). [citations are not "numbers" — sourcing bib entries broadly is fine.]
- Decide bibstyle: template default is `ACM-Reference-Format`; summary/neurips use `plainnat`.
