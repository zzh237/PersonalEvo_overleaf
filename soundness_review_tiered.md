# Soundness-Only Tiered Review

## Scope

This review assesses only scientific and technical soundness. It does not score
novelty, presentation, venue compliance, or writing quality.

I checked the current `latex/iclr2026.tex`, the central proofs in its appendix,
the experiment implementations under `src/personal_evo/` and `scripts/`, and the
available JSON/CSV artifacts under `results/`. The paper compiles successfully
to 118 pages. The two available Hyper-Ridge implementation tests pass, but they
only check identity reduction and positive definiteness.

## Reviewer Summary

The paper contains one sound core: the four-term decomposition is a correct
accounting identity, and the static-versus-routed equality condition is correct.
The new Hyper-Ridge contribution is not soundly established in its current
form. The advertised effective-dimension rate does not follow from the explicit
confidence radius, the theorem for estimated covariance analyzes a different
algorithm from the one presented and implemented, and the implemented estimator
fails the experiment designed to test its mechanism. The headline experimental
tables evaluate the old router and have serious provenance, protocol, and
internal-consistency problems. I would not rely on the current empirical or
rate claims without a substantial theory and experiment rebuild.

## Scores

- **Soundness:** 1/4
- **Overall recommendation based only on soundness:** 2/10 (reject in current form)
- **Confidence:** 5/5
- **Presentation and contribution:** not assessed by request

## Tier 1: Critical Findings

### 1. The advertised LinUCB and Hyper-Ridge rates do not follow from the paper's own confidence radii

The base claim is
`B_T = O(sqrt(K_max d T log T))` at `iclr2026.tex:713-718`. However, the
appendix derives

`B_T = 2 beta_T sqrt(2 T K_max d log(1 + T L_x^2/(lambda d)))`

at `iclr2026.tex:2743-2787`, where `beta_T` itself contains a
`sqrt(log T)` factor. With the paper's choice `delta=1/T`, even when `d` is
treated as fixed, the displayed derivation gives
`B_T/T = O(log(T)/sqrt(T))`, not
`O(sqrt(log(T)/T))` as asserted at `iclr2026.tex:3186-3204` and
`iclr2026.tex:3235-3256`. When dimension is not fixed, an additional dimension
factor also enters through `beta_T`.

The same missing confidence-radius factor drives the central Hyper-Ridge claim.
The valid pathwise oracle bound at `iclr2026.tex:10337-10352` contains
`alpha_T`. The simplified effective-dimension rate is obtained only by assuming

`alpha_T^2 [1 + log(1 + T L_Omega^2/lambda)] <= C_Omega log(eT/delta)`

at `iclr2026.tex:10262-10303`. This condition is not derived from
Lemma `hyper-sn`. In fact, it conflicts with the paper's explicit frozen
covariance radius at `iclr2026.tex:11978-11990`. Under the theorem's own choice
`delta_route=T^{-2}` (`iclr2026.tex:12376-12390`), that radius has
`alpha_T^2 = Omega(log T)` before accounting for the determinant term. The
left-hand side of the calibrated condition is therefore
`Omega((log T)^2)`, while its right-hand side is `O(log T)` for a fixed
`C_Omega`.

Thus the self-normalized lemma supports a bound with the explicit `alpha_T`,
but not the headline `sqrt(K_max d_eff T log T)` rate. This affects the abstract
(`iclr2026.tex:119`), introduction (`iclr2026.tex:159-168`), main proposition
(`iclr2026.tex:979-1018`), theory map (`iclr2026.tex:11111-11120`), and
conclusion (`iclr2026.tex:1405-1435`).

**Required correction:** either prove a confidence result that actually
satisfies the calibrated condition, or retain the explicit confidence radius
and state the resulting extra logarithmic/dimensional factor everywhere.

### 2. The analyzed estimated-covariance algorithm is not the proposed algorithm

Algorithm 1 periodically re-estimates covariance from the same adaptively
collected archive data (`iclr2026.tex:343-385`). The only finite-sample
estimated-covariance theorem instead requires:

- fresh calibration candidates excluded from routing;
- exogenous uniform calibration;
- `m` observations for every calibration candidate;
- a covariance frozen before routing;
- a fixed post-calibration archive.

These conditions appear at `iclr2026.tex:12194-12240` and are materially
different from Algorithm 1. The calibration-split procedure is neither
implemented nor evaluated. The paper does disclose this gap in several places,
but still titles and frames the work around the periodic Hyper-Ridge method and
states Proposition `prop:deff-bound` for Algorithm 1
(`iclr2026.tex:979-1018`). A nonnegative, otherwise uncontrolled
`Err_T` makes that proposition non-informative for the implemented algorithm.

The current periodic estimator also fails its clean mechanism experiment:

| d | matched isotropic | known covariance | periodic estimate |
|---:|---:|---:|---:|
| 20 | 314.082 | 170.166 | 375.009 |
| 50 | 447.612 | 213.067 | 504.865 |
| 100 | 455.422 | 188.119 | 487.548 |

Known covariance helps, but the proposed estimator raises exploration cost by
7.1--19.4%. The same artifact reports relative covariance errors from about
10 to 382 in the archive-size sweep, far outside the perturbation theorem's
required `rho_T < 1`. On all seven adapters, full and diagonal Hyper-Ridge are
exactly identical for every seed because the one-hot representation makes the
estimated covariance diagonal.

**Required correction:** choose one contribution. Either present a
calibration-split method and implement/evaluate that exact algorithm, or provide
new theory and feature-aware evidence for the periodic estimator.

### 3. The controlled Hyper-Ridge diagnostic violates the assumptions of the theorem it is said to support

The theory assumes mean rewards in `[0,1]` and uses `0 <= L_t <= 1` to convert
high-probability bounds to expectation. The planted diagnostic instead samples
Gaussian candidate parameters and uses unbounded linear rewards
(`scripts/run_hyper_ridge_suite.py:140-179`). Recomputing the five saved seeds
gives means ranging approximately from `-6.60` to `6.35`. Its reported
cumulative exploration costs reach `447.6` and `504.9` at `T=400`, which is
impossible under `L_t <= 1`.

The diagnostic also runs with fixed `alpha=0.8`, whereas the theorem requires a
valid time-uniform confidence multiplier with `alpha_T >= 1`. Gaussian
parameters do not satisfy the assumed almost-sure ellipsoid bound either.
Therefore the experiment can show that a supplied covariance is a useful
regularizer in this simulator, but it cannot validate the theorem or its rate.

### 4. The headline numerical results are internally inconsistent and not reproducible from the current artifacts

The four-term table and baseline table report different rewards for what both
call the full PersonalEvo method. Reconstructing routed reward from Table
`tab:main` gives:

| Benchmark | Four-term reconstruction | Baseline table PersonalEvo | Difference |
|---|---:|---:|---:|
| General | 0.865 | 0.797 | +0.068 |
| Workspace | 0.699 | 0.678 | +0.021 |
| Support | 0.816 | 0.757 | +0.059 |
| GEPA | 0.700 | 0.681 | +0.019 |
| Hermes | 0.803 | 0.771 | +0.032 |
| PAHF shopping | 0.734 | 0.754 | -0.020 |

These differences are far larger than rounding error, and no protocol change is
stated between `iclr2026.tex:1223-1267` and `iclr2026.tex:1315-1343`.

The current result files also do not support the paper's provenance claims:

- `results/benchmarks_e6_final/e6_baselines_comparison.json` contains only
  4/9 rows.
- `experiment_plan_new/10_evidence_triage.md:14` says the other rows came from a
  missing sibling run and instructs the writer to treat paper values as
  canonical.
- No current source implements the reported GEPA-Pareto, Thompson, and
  RouteLLM nine-benchmark table.
- `scripts/generate_paper_figures.py:114-125` hardcodes PAHF-Embodied and
  PrefEval values from the paper.
- That script can execute only four external adapters, hides unfilled panels,
  and does not generate the five traces claimed by the 3x3 figure caption.
- The appendix simultaneously claims every Table 1 entry is generated by one
  bitwise-reproducible harness (`iclr2026.tex:8514-8516`,
  `iclr2026.tex:8641-8667`) and says that harness is synthetic-only with no
  real-data benchmark (`iclr2026.tex:8653-8661`).

Even the available synthetic artifact disagrees with the appendix's claimed
per-seed values. For example,
`results/decomposition_main_20260617/decomposition_summary.json` gives General
discovery `0.0439` and full reward `0.8572`, rather than `0.051` and about
`0.865`.

**Required correction:** regenerate every main table and figure from one
versioned command and publish the raw per-seed outputs. Remove any row that
cannot be regenerated.

### 5. The claimed framework and real-data evidence is mostly synthetic, imputed, or absent

The abstract calls GEPA and Hermes agent-framework substrates and describes four
real-data preference datasets (`iclr2026.tex:113-117`). The current
implementations do not support that interpretation:

- GEPA candidates, scores, and users are randomly generated
  (`src/personal_evo/benchmarks/gepa_archive_benchmark.py:53-81`).
- Hermes configurations, quality vectors, users, and rewards are randomly
  generated (`src/personal_evo/benchmarks/hermes_benchmark.py:65-149`).
- BESPOKE pools responses from unrelated users and queries. In the current
  30-by-300 matrix, only 300 of 9,000 user-candidate entries are observed;
  approximately 96.7% are imputed as a population mean, followed by a synthetic
  user-level bias (`bespoke_benchmark.py:77-123`).
- PAHF rewards are an author-designed attribute-matching formula, not logged
  interaction or preference rewards (`pahf_benchmark.py:42-87`).
- No current source implements PAHF-Embodied or PrefEval, although their rows
  appear in all headline tables and figures.

The current configurations also differ sharply from the paper: BESPOKE has 300
candidates in code versus 40 in the paper; PAHF Shopping has 450 in the
Hyper-Ridge suite versus 40 in the paper; the Hyper-Ridge synthetic adapters use
12 users and 10 candidates versus 8 and 12; and GEPA/Hermes diagnostics run for
75/100 rounds rather than the stated 250.

Consequently, statements that real-data heterogeneity confirms practical
significance are not supported by the available protocol.

## Tier 2: Major Findings

### 6. The empirical exploration-cost estimator is not the theoretical quantity

The theorem defines
`L_t = m_t - mu(U,a_t)`, but the implementation and appendix use
`m_t-r_t` (`iclr2026.tex:8525-8530`;
`src/personal_evo/benchmarks/run_all.py:39-62`). The simulator knows `mu`, so
there is no need to substitute noisy reward.

Moreover, the code clips `mu + Gaussian noise` to `[0,1]`
(`src/personal_evo/core.py:204-212`), so the induced residual is not generally
conditionally mean-zero near the boundaries. The appendix instead states an
unclipped Gaussian observation model (`iclr2026.tex:8391-8396`) and incorrectly
attributes the caveat to clipping the deterministic mean
(`iclr2026.tex:8402-8406`).

Finally, the reported reconstruction residual is nearly zero by construction:
the code defines learning tax as `oracle_mean-routed_mean` and then reconstructs
with the same routed mean. This is an accounting check, not independent
empirical validation of the decomposition.

### 7. The documented reward model and reflection protocol do not match current code

The appendix specifies
`mu=clip(0.4q+0.6w+0.2qw,0,1)` at `iclr2026.tex:8377-8389`. Current code uses
`clip(0.5q+0.7w,0,1)` with no interaction
(`src/personal_evo/core.py:192-201`). The saved JSON confirms weights 0.5 and
0.7.

The paper describes the full method as gated reflection, while
`run_decomposition_study` calls the `ungated` branch
(`src/personal_evo/studies.py:247-263`). These mismatches can materially change
both discovery gain and exploration cost.

### 8. The failure-prioritization lemma is not valid as written

At `iclr2026.tex:665-672`, the paper claims

`E[# mutations] <= T P(L_t + |xi_t| > tau)`

with a free, unspecified `t`. In general the valid bound is

`E[# mutations] <= sum_t P(L_t + |xi_t| > tau)`

or `T sup_t P(...)`. A stationarity assumption would be needed to use one
round's probability. This lemma should be corrected or removed.

### 9. Experimental claims of rate consistency are not rate tests

The statement that a decreasing empirical curve is "consistent with" the
claimed regret rate (`iclr2026.tex:1302-1307`) does not estimate an exponent,
vary horizon, compare confidence-scaled alternatives, or test the corrected
rate. A single `T=250/300` trace cannot distinguish `sqrt(T log T)`,
`sqrt(T) log T`, or linear behavior over the relevant range.

### 10. Fixed `alpha=0.8` experiments are disconnected from all confidence guarantees

Both base and Hyper-Ridge experiments use `alpha=0.8`
(`src/personal_evo/core.py:43-56`; `run_hyper_ridge_suite.py:69-75`).
The proved UCB statements require a radius depending on noise, dimension,
archive size, horizon, parameter norm, and failure probability. No empirical
coverage check is reported. The experimental router can be evaluated as a
heuristic, but its curves are not evidence for the stated high-probability
guarantees.

## Tier 3: Minor Soundness Issues

1. The main text defines `d_eff` using a pooled design
   (`iclr2026.tex:955-967`), while the appendix theorem uses the maximum
   candidate-wise effective dimension (`iclr2026.tex:10306-10335`). The bound
   should use one definition consistently.
2. Algorithm 1 re-estimates covariance after round `t`, but the implementation
   triggers re-estimation inside `select()` before updating with round `t`'s
   reward (`src/personal_evo/core.py:423-447`).
3. The paper alternates between a fixed-user trajectory `U` and a stream of
   users `u_t`. The probability model and which data are shared across users
   should be stated once and used consistently.
4. The only implementation tests do not check confidence coverage,
   effective-dimension calculations, covariance recovery, regret scaling,
   paper-table reproduction, or the calibration-split procedure.

## Questions for the Authors

1. What exact command, source revision, and raw per-seed artifact generated each
   of the nine rows in Tables `tab:main` and `tab:baselines`?
2. Why do the routed rewards reconstructed from the four-term table disagree
   with the PersonalEvo rewards in the baseline table under the stated common
   protocol?
3. How can the calibrated-confidence condition hold with the explicit
   `alpha_T` and `delta_route=T^{-2}` given in the same theorem?
4. Is the claimed contribution Algorithm 1's periodic estimator or the
   calibration-split frozen estimator? Which one will be implemented and
   evaluated?
5. What observed user-candidate labels, rather than imputed or heuristic
   rewards, support the four "real-data" rows?

## Ordered Revision Plan

### Layer 0: Hours, certain

1. Remove or clearly quarantine every table row and figure panel that lacks a
   regenerating script and raw artifact.
2. Replace all simplified base and Hyper-Ridge rates with the rates obtained
   from the explicit confidence multipliers.
3. Remove the contradictory claims that one synthetic-only harness generated
   all nine benchmark rows.
4. Relabel GEPA, Hermes, BESPOKE, and PAHF accurately as synthetic,
   constructed, imputed, or heuristic where applicable.

### Layer 1: Days, no new scientific idea required

1. Create one experiment manifest containing code revision, configuration,
   seeds, command, raw output, and figure/table generator for every claim.
2. Recompute `L_t` from the selected arm's known `mu` in simulation; report
   observed-reward regret separately.
3. Make the paper reward function, clipping rule, reflection gate, dimensions,
   horizons, and candidate counts match the executable code.
4. Correct the failure-prioritization lemma and all derived references.
5. Either implement all named baselines faithfully or remove the unsupported
   comparison table.

### Layer 2: Weeks, required experiments

1. Implement the calibration-split algorithm exactly as analyzed, including
   fresh calibration candidates, charged calibration pulls, frozen covariance,
   and the stated confidence radius.
2. Run base, known-covariance, calibration-split, periodic, and diagonal
   routers on matched seeds, horizons, archives, and shared feature maps.
3. Use cumulative exploration cost as the primary endpoint and report
   covariance error, confidence coverage, `d_eff`, fitted horizon exponent, and
   all per-seed values.
4. Add at least one genuine logged user-by-candidate reward dataset. Otherwise
   frame the evidence as simulation rather than real-data validation.

### Layer 3: New theory

Prove covariance and confidence control for the fully adaptive periodic
estimator, or make that estimator explicitly future work. Do not transfer the
calibration-split theorem to it by notation alone.

## Proofs and Experiments Checked

### Proof conclusions

- Exact four-term decomposition: **correct as an accounting identity**.
- Static-versus-routed inequality and equality condition: **correct**.
- First-useful-specialist geometric bound: **correct under its stated,
  very strong per-attempt assumption**.
- Failure-prioritization lemma: **incorrect as written**.
- Disjoint LinUCB finite expression before big-O simplification: **substantially
  standard; final rate simplification is incorrect**.
- Known-covariance self-normalized and elliptical-potential steps:
  **the pathwise structure is reasonable; the advertised simplified rate is
  not established**.
- Relative-Loewner perturbation lemmas: **correct conditional on
  `rho_T<1`**.
- Calibration covariance-control argument: **plausible under its strong fresh,
  independent calibration assumptions; it covers a different algorithm, and
  the oracle-rate recovery condition conflicts with the explicit radius**.

### Experiment checks

- `PYTHONPATH=src pytest -q src/personal_evo/tests`: **2 passed**.
- Hyper-Ridge suite numbers were recomputed from
  `results/hyper_ridge/suite.json`.
- Adapter dimensions, horizons, and full-versus-diagonal equality were checked
  against `scripts/run_hyper_ridge_suite.py`.
- Benchmark reward construction was traced through all current adapters.
- Main-table values were cross-checked against available JSON/CSV artifacts and
  against the algebraic reconstruction implied by the paper itself.
