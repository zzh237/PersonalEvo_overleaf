# PersonalEvo v5 Revision Report

## Revision diagnosis

Version 4 placed Experiments before the problem formulation, method, and theory. It also defined the Hyper-Ridge estimator and algorithm in the preliminary section, repeated them in a nominal Method section, and repeated the same equations and algorithm again in Theory. The only complete proof in the main paper was the short Hyper-Ridge dominance proof; the other main-text proofs were already labeled as proof sketches.

Version 5 restores the reading order

> Introduction -> Related Work -> Preliminaries -> Method -> Theory -> Experiments -> Conclusion.

It removes the duplicate estimator equations, duplicate Hyper-Ridge algorithm, and redundant Method summary. The theorem statements and proof sketches remain in the main paper. The complete dominance proof now appears in the Hyper-Ridge appendix.

## Evidence-labeled lineage

- **Verified:** `iclr2026_v5.tex` was created from `iclr2026_v4.tex`; v4 is the Overleaf repository commit `eaeb313`.
- **Verified:** v3 and v4 have the same theorem and label inventory. The principal v4 change was moving Experiments before the technical development.
- **Verified:** the clean experiment package entered the outer code repository in commits `199828e` and `934f89f`.
- **Verified:** the clean package implements `run_experiments.py -> summary.json/traces.npz/manifest.json -> plot_from_artifacts.py`.
- **Inferred:** the later five-curve Figure 2 raster was produced by an uncommitted or temporary plotting path; commit `1767cbd` contains only the raster.
- **Unknown:** the exact temporary source and trace artifacts used to create that later Figure 2 raster.

## Main claim ledger

| Claim | Target and observation model | Assumptions | Dependency | Status |
|---|---|---|---|---|
| Exact four-term decomposition | Population-average mean reward along one user's online trajectory | Finite shared deterministic initial archive; bounded measurable rewards | Appendix A identities | Proved |
| Personalized-routing dominance | Mean reward exceeds the best static initial candidate | Same assumptions as the decomposition | Exact decomposition | Proved; full proof in Appendix J.1 |
| LinUCB plug-in bound | Finite-time control of exploration cost | Predictable archive, linear model, bounded features, sub-Gaussian noise, stated confidence multiplier | Standard self-normalized linear-bandit result plus paper-specific reduction | Conditional/imported and explicitly scoped |
| First-useful-specialist time | Probability of discovering an improving candidate | Per-attempt repair probability \(q_\epsilon\) | Geometric stopping argument | Proved conditionally; not empirically tested |
| Known-\(\Omega\) Hyper-Ridge bound | Pathwise and expected exploration cost | Known positive-definite geometry, predictable births, bounded ellipsoid and effective dimension | Self-normalized concentration and elliptical potential lemmas | Proved; full derivation in Appendix K |
| Calibration-split Hyper-Ridge bound | Estimated-covariance exploration cost | Independent fresh calibration candidates and explicit diversity conditions | Covariance concentration and frozen-estimate confidence theorem | Proved under stated split; does not cover periodic updates |
| Nine-benchmark headline results | Base-router reward, ablations, and observed-reward residual | Historical benchmark protocol | Legacy artifacts and figures | Numerically retained; figure provenance remains partial |

## Gap and provenance memo

### Legacy figures

1. **Claimed implication:** the historical Figure 1/2 generator provides an end-to-end reproducible path for the paper figures.
2. **Missing step:** Figure 1 inserts PAHF Embodied and PrefEval as Python literals; Figure 2 combines a frozen synthetic artifact with live benchmark execution, while the committed five-curve raster is not reproduced by the tracked script.
3. **Correct statement:** the exact rasters, known inputs, generator, and hashes are registered, but exact safe regeneration is unavailable.
4. **Affected claims:** only provenance and reproducibility claims. No numerical result or caption value was changed in v5.
5. **Repair applied:** the outer repository now contains `reporting/figures/provenance.json` with the correct paper-relative paths and explicit `partial` status.
6. **Remaining repair:** recover the historical temporary traces or replace the figures only after validating the clean protocol as a scientifically equivalent experiment.

### Theory-to-experiment boundary

The saved headline tables report \(m_t-r_t\), while the decomposition theorem uses \(m_t-\mu(U,a_t)\). Reward clipping prevents identifying these quantities without the selected-arm pre-clipping means. The paper already states this limitation, and v5 preserves it. The periodic online covariance estimator also remains outside the proved known-\(\Omega\) and calibration-split guarantees.

## Proof movement map

| Main-text result | Main text in v5 | Full proof |
|---|---|---|
| Exact decomposition | Statement and proof sketch | Appendix A |
| Known-\(\Omega\) exploration bound | Statement and proof sketch | Appendix K |
| Hyper-Ridge dominance corollary | Statement only | Appendix J.1 |
| Calibration-split theorem | Referenced summary | Appendix L |

## Validation

- Static label audit: 535 labels, 431 references, no duplicate labels, no missing references.
- LaTeX: `latexmk -pdf -interaction=nonstopmode -halt-on-error iclr2026_v5.tex` succeeds.
- Final log: no undefined references, multiply defined labels, overfull boxes, or LaTeX errors.
- PDF: 85 pages total; main text ends on page 18 and the appendix starts on page 19.
- Main-section starts: Related Work page 3, Preliminaries page 4, Method page 6, Theory page 9, Experiments page 14, Conclusion page 18.
- Reporting audit: 0 errors; 2 expected partial-provenance warnings for the legacy main figures.
- Experiment-package tests: 8 passed.
- Frozen-artifact plotting: two independent renders produce byte-identical PNG and PDF figures.

## Unresolved work

- The clean five-seed paper experiment was not launched in this reporting revision. Its benchmark constructions differ from the historical protocol, so its outputs cannot silently replace the current paper figures.
- Exact end-to-end regeneration of the two historical main figures remains unavailable.
- The main paper is 18 pages before appendices. The restored scientific order is correct, but venue-length compression remains an authorial editing task.
- The periodic covariance estimator still lacks a matching theorem and a feature-aware nine-benchmark evaluation.
