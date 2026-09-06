# Theory Revision Report for `iclr2026_v1.tex`

## Scope

This report audits mathematical and statistical soundness. It distinguishes
proved results from conditional statements, diagnostics, and open problems.
It does not certify the provenance of every experimental table.

## Lineage

### Verified

- Commit `e5831ff` introduced the Hyper-Ridge revision.
- Commit `b7b487b` copied the then-current `latex/iclr2026.tex` to
  `latex/iclr2026_v1.tex` as the snapshot before this theory revision.
- The baseline snapshot has SHA-256
  `a3d46c8fc04a80c62524faf024ac25fb331d8629dfd24ce402181ef1628d0f12`.
- The revised draft has SHA-256
  `6a4585a988d7bc5bc6ce56ea9f088bccf5b0962f51a6d24f8634efc67fcb8114`.
- The revision changes the existing draft. It does not replace the baseline
  `latex/iclr2026.tex`.
- The claim inventory has the same number of theorem-like claims as the
  snapshot. The revision changes assumptions, statements, proofs, and claim
  boundaries.
- The authoritative dual-CLI review is
  `reviews/dual_cli_soundness/runs/20260906_071658_personalevo_soundness_consensus_stable_parent`.
  Codex and Claude both accepted bundle
  `f0add870d792552d0dd70fdd76b6b275d9f2da956fd0a8fe9e2dbebaa9622879`.
  The source-history audit passed for both parent sessions.

### Inferred

- The intended new contribution is the archive-geometry regularizer, not a
  theorem for the periodically re-estimated implementation.
- The calibration-split result was added to provide a valid estimated-covariance
  theorem without claiming control of adaptive covariance reuse.

### Unknown

- The exact raw artifact and command that generated every row of the nine-table
  benchmark comparison are not available in the current repository.
- The current evidence does not establish that every benchmark described as
  real-data uses observed user-candidate rewards.

## Revision Diagnosis

The exact four-term decomposition was sound. The main defects were in the
claims built around it:

1. The stated LinUCB and Hyper-Ridge rates omitted the confidence radius.
2. The only estimated-covariance theorem analyzed a frozen calibration split,
   while the main algorithm periodically re-estimated covariance.
3. The tables used `m_t-r_t`, but the theorem uses
   `m_t-mu(U,a_t)`.
4. Clipped observations were treated as centered around the pre-clipping mean.
5. The probability model mixed a fixed user trajectory with a stream of users.
6. Several concentration statements omitted filtration, archive-cap, or
   bounded-loss assumptions used in their proofs.
7. One discovery event was undefined when the required attempt never occurred.
8. The final effective-dimension proof used the pre-round matrix \(X_{a,T}\)
   as if it contained round \(T\). The complete design is \(X_{a,T+1}\).
9. The probability sample space and the archive covariance both used
   \(\Omega\), and the calibration certificate did not say which constants
   must be supplied.

The revision keeps the decomposition and valid proof components. It restores
the full confidence radii, separates three different routers, and narrows each
claim to the model its proof covers.

## Gap Memo

| Gap | Invalid or missing step | Correct statement | Repair in v1 |
|---|---|---|---|
| Simplified exploration rate | The potential term was reported without the UCB confidence radius. | The known-geometry bound is the explicit radius times the effective-dimension potential. For fixed problem parameters it is `O(sqrt(T) log T)`. | Replaced every headline rate by the explicit budget or its valid fixed-parameter order. |
| Periodic covariance transfer | A theorem for fresh calibration candidates and frozen covariance was attached to a periodically updated estimator using adaptive routing data. | The oracle, calibration-split, and periodic routers are different procedures. | The first two have separate theorems. The periodic procedure is marked open. |
| Empirical exploration cost | `m_t-r_t` was identified with `m_t-mu(U,a_t)`. | The first is an observed-reward residual. Clipping can make it biased for the theoretical cost. | Renamed the logged quantity and removed theorem-validation claims based on it. |
| Frozen-covariance confidence | Conditioning did not explicitly include calibration data and the fixed routing parameters. | The routing filtration must contain the pre-routing sigma-field, and the conditional noise MGF must still hold. | Added `F_0^route` and stated the enlarged-filtration assumption. |
| User process | A user stream was combined directly with a decomposition for one user drawn once. | The combined theorem uses `u_t=U`. Confidence and potential lemmas may separately allow predictable user streams. | Added the fixed-user specialization at each combination point. |
| Missing attempt | `S_{g,m}` used an indicator that existed only if attempt `m` occurred. | Define the success indicator as zero when its stopping time is infinite. | Added a piecewise definition on the full sample space. |
| Design endpoint | `X_{a,t}` contains observations with `s<t`, but the proof used `X_{a,T}` for all rounds through `T`. | The full design is `X_{a,T+1}`; all prefix caps run through `T+1`. | Defined fixed-user and streaming candidate unions, changed the full design to `T+1`, and summed over every candidate that appeared. |
| Calibration tuning | The theorem used unknown problem constants without saying how the radius is obtained. | The constants must be supplied as deterministic bounds; otherwise the result is an oracle-tuned certificate. | Added the required constants and the oracle-tuning qualification. |

## Consensus Finding Map

| Finding | Outcome in `iclr2026_v1.tex` |
|---|---|
| C1: sharp Hyper-Ridge rate dropped the radius | Repaired. Every proved bound keeps the full radius; the fixed-parameter order is `O(sqrt(T) log T)`. |
| C2: theorem, algorithm, and code were different procedures | Repaired as a claim boundary. Known-\(\Omega\), calibration-split, and periodic routing are separate. Periodic routing remains open. |
| C3: centered effects plus nonnegative linear rewards were degenerate | Repaired with the candidate-independent baseline `b(u)` and centered response `r-b(u)`. |
| C4: tables and figures mixed experiment lineages | Not repaired by a theory edit. It remains an empirical blocker. |
| C5: benchmark provenance was overstated | Not repaired by a theory edit. It remains an empirical blocker. |
| C6: appendix seeds and uncertainty values failed replay | Not certified. The paper no longer uses the disputed bootstrap title, but raw per-seed provenance still needs rebuilding. |
| M1: base LinUCB rate dropped `beta_T` | Repaired. The exact radius is retained and the fixed-parameter order is corrected. |
| M2: the Hyper-Ridge diagnostic was outside the theorem regime | Repaired as scope. The paper reports it as a diagnostic and states that the periodic estimator performs worse. |
| M3: theory and code used different rewards and cost estimands | Partly repaired. Mean-reward cost and observed-reward residual are separated, and clipping bias is stated. The experiment protocol still needs rebuilding. |
| M4: statistical and scaling claims exceeded the evidence | Not certified. New paired-seed and multi-horizon experiments are still required. |

## Claim Ledger

| Claim | Target and observations | Main assumptions | Dependency | Status |
|---|---|---|---|---|
| `lem:hetero` | Static and routed values from the same finite archive | Deterministic finite archive; bounded measurable means | Internal max/expectation proof | Proved |
| `thm:decomp` / `thm:main` | Mean routed reward on one user's online trajectory | Common deterministic `A_1`; finite current archives | Accounting identity | Proved |
| `cor:dominance` | Routed mean reward versus best static initial candidate | Conditions of the decomposition | Algebraic rearrangement | Proved |
| `lem:predictable` / `lem:3` | Valid per-candidate martingale processes after archive growth | Predictable birth times and features; deterministic archive cap | Internal filtration argument | Proved under stated predictability |
| `lem:failure` | Expected number of triggered mutations | Implemented trigger | Event inclusion and sum over rounds | Proved |
| `cor:plugin` / `thm:convergence` | Mean-reward lower bound | External expected exploration budget `B_T` | Decomposition | Conditional |
| `prop:2` | Base disjoint-LinUCB exploration budget | Centered sub-Gaussian noise; bounded features and parameters; predictable slots | Self-normalized concentration | Proved for the theoretical multiplier |
| `thm:specialist` / `thm:generalization` | Success among active repair attempts | Attempts exist; conditional success probability before first success | Stopping-time induction | Conditional |
| `prop:3` | Archive and rollout complexity | Cooldown, append-at-most-one, archive cap | Deterministic counting | Proved |
| `prop:4` | Discovery reserve after useful specialists | Attempt occurrence, repair probability, persistence, exploration budget | Decomposition and specialist theorem | Conditional |
| `prop:5` | Per-user archive copies | Common initial archive; separate later paths | Pointwise accounting | Proved |
| `prop:6` / `prop:16` | Finite-sample decomposition estimates | Stated sampling and noise model | Concentration plus algebra | Proved for that model |
| `prop:7`--`prop:11` | Comparator and model counterexamples | Explicit finite constructions | Direct calculation | Proved |
| `prop:12`--`prop:15` | Trigger, cluster, repair, and archive-cap constants | Each proposition's explicit structural assumptions | Internal concentration/counting | Conditional as stated |
| `lem:hyper-sn` | Known-geometry prediction confidence | Fixed candidate parameter; centered conditional noise; predictable selected features | Gaussian-mixture self-normalization | Proved |
| `lem:hyper-elliptical` | Sum of generalized-ridge confidence widths | Positive-definite `Omega`; bounded scaled feature norm | Determinant identity and scalar spectral bound | Proved |
| `prop:deff-bound` / `prop:deff-bound-app` | Known-geometry cumulative exploration cost | Fixed-user route when combined with decomposition; bounded means; archive cap; ellipsoid and effective-dimension caps | The two preceding lemmas | Proved |
| `lem:hyper-perturb-loewner` and `lem:hyper-perturb-width` | Matrix, width, and effective-dimension stability | Relative covariance error below one | Loewner order and inversion | Proved |
| `prop:hyper-estimated-perturbation` | Estimated-geometry exploration bound | Positive-definite estimates; archive cap; valid estimated-geometry confidence event; relative spectral control | Perturbation lemmas | Conditional |
| `lem:err-control-independent-subg-cov-final` | Sample covariance operator error | Independent mean-zero sub-Gaussian vectors | Vershynin Lemma 2.7.7 and Theorem 2.8.1 plus a net argument | Imported and checked |
| `lem:calibration-gram-lower-tail-final` | Uniform invertibility of calibration designs | Independent bounded diverse calibration features | Tropp Theorem 5.1.1 | Imported and checked |
| `lem:calibration-block-covariance-control-final` | Bias and tails of per-candidate OLS estimates | Fresh independent candidate effects; enlarged calibration filtration | Internal conditional calculation | Proved |
| `lem:relative-covariance-error-final` | Relative error of frozen covariance estimate | Calibration assumptions and sufficient samples per candidate | Previous three lemmas | Proved |
| `lem:confidence-under-frozen-omegahat-final` | Routing confidence after calibration | Frozen positive-definite estimate; routing MGF after conditioning on calibration | Conditional self-normalization | Proved |
| `thm:err-control-calibration-split-final` | Frozen estimated-covariance exploration cost | Fresh calibration candidates; exogenous features; fixed post-calibration archive; fixed routing user when combined with decomposition; supplied deterministic tuning bounds | Covariance and confidence lemmas | Proved; oracle-tuned if the bounds are unavailable |
| `conj:err-control` | Sharper estimated-covariance rate | Not established | None | Conjectural |
| Periodic covariance update in Algorithm 1 | Adaptive estimated-covariance routing | Same data used for routing and repeated covariance updates | Missing adaptive confidence and covariance analysis | Open |

## Validation

- `latexmk -pdf -interaction=nonstopmode -halt-on-error -file-line-error iclr2026_v1.tex`
  succeeds and produces a 118-page PDF. A stale auxiliary file first required
  one direct `pdflatex` pass; the final `latexmk` run is clean.
- The final log has no undefined references, undefined citations, duplicate
  labels, LaTeX errors, or overfull boxes.
- All 1,101 labels are unique, and every `ref`/`eqref` target exists.
- `git diff --check` reports no whitespace errors.
- `PYTHONPATH=src pytest -q src/personal_evo/tests` reports `2 passed`.
- No undefined `\mathcal A_t(U)`, probability-space use of covariance
  `\Omega`, or full-design use of `X_{a,T}` remains.
- Every formal substitution `\delta=1/T` now states `T>=2`.
- The final baseline-to-revision inventory has no heading or theorem-like
  claim-count change and no duplicate labels.
- Randomized matrix checks passed in 4,000 cases for the log-determinant
  effective-dimension inequality and the Loewner, width, and effective-dimension
  perturbation inequalities.
- The dual-CLI consensus audited the baseline. The final revision maps every
  Critical and Major finding to a repair, a narrower claim, or an explicit
  unresolved item.
- Vershynin Lemma 2.7.7 and Theorem 2.8.1 were checked against the 2018 first
  edition. Tropp Theorem 5.1.1 was checked against the cited monograph.
- Remaining log messages are underfull boxes, float-placement changes, and
  PDF-bookmark warnings for math in headings.

## Unresolved Work

1. No theorem covers the periodic covariance estimator in Algorithm 1.
2. The calibration-split procedure is not implemented or evaluated.
3. The planted Gaussian diagnostic violates the bounded-reward theorem and
   uses a fixed heuristic exploration multiplier.
4. The saved headline tables do not log the theoretical mean-reward
   exploration cost.
5. Several nine-benchmark rows lack a complete raw-artifact and command trail.
6. The main text's real-data descriptions conflict with the synthetic-only
   reproduction appendix and require a separate empirical revision.
7. A theorem combining shared multi-user routing with the four-term
   fixed-user decomposition remains open.
8. The decomposition accounts for evolution gain, but no non-oracle rate
   controls the proposer or archive growth. The calibration-split theorem uses
   a fixed routing archive.

These items are not hidden inside a remainder term. The revised theorem claims
are restricted so that none of them depends on these unresolved steps.
