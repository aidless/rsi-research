# P1 Pre-Registered Experiment Card

**Paper**: Auditable Recursive Self-Improvement (TMLR submission)
**Card version**: v1.0 (2026-08-08)
**Status**: pre-registered protocol; **EXECUTED 2026-08-09 (synthetic AR(1) simulator)**; verdict: **P1 supported** (ESS-corrected power at N_inner=5 < 0.8 for all rho); real AIDE85 lineage replication still pending

## 1. Prediction (P1)
Ignition-test outcomes are subject to a measurement-noise lower bound: with
inner-loop evaluation noise σ_step and outer-loop autocorrelation ρ, the
effective sample size per candidate collapses, so per-candidate ignition
tests with N_inner ≈ 5 are underpowered regardless of ρ ≥ 0.

## 2. Falsification criterion
P1 is falsified if a per-candidate ignition test with N_inner = 5 detects a
true effect d = 0.5 with power ≥ 0.8 under the stated AR(1) noise model.

## 3. Data / procedure
- Simulator: AR(1) effective-sample-size computation (σ_cross amplification);
  ρ grid {0.0, 0.3, 0.5, 0.7, 0.9}; N_inner grid {1, 2, 5, 10, 50, 100}.
- Seed-controlled (seed = 42), single consumer GPU/CPU, runtime < 1 s per cell.
- When AIDE85 lineage is released: replicate on real lineage traces and report
  the same metrics; the synthetic simulator is the development proxy.

## 4. Sample size / analysis
- 5 ρ values × 6 N_inner values = 30 cells, each with 1000 Monte-Carlo
  replications; report mean and 95% CI of detected effect and power.
- Primary statistic: empirical power at d = 0.5, two-sided α = 0.05.

## 5. Decision rules (pre-registered)
- Power(N_inner=5, d=0.5) < 0.8 for all ρ ≥ 0  → P1 supported.
- Power(N_inner=5, d=0.5) ≥ 0.8 for any ρ ≥ 0 → P1 falsified.

## 6. Deviations policy
Any change to the simulator, parameter grid, or decision rules is recorded as
a pre-registered deviation with justification, in this file's changelog.

## Changelog
- 2026-08-08: v1.0 card created.
- 2026-08-09: EXECUTED per frozen protocol -- experiments/p1_drill/outer_loop_correlation.py, 30 cells x 1000 MC reps, seed 42, d=0.5, alpha=0.05 two-sided (Welch naive + ESS-corrected z). Verdict: P1 supported; ESS-corrected power at N_inner=5: 0.130/0.075/0.047/0.015/0.000 for rho 0.0/0.3/0.5/0.7/0.9 (all < 0.8). Naive Welch power 0.106->0.662 (anti-conservative under rho>0). Results: p1_power_results.json; provenance: P1_REPRODUCIBILITY.md. No protocol deviations.

## 阈值冻结声明
第 5 节的判定阈值在数据收集前冻结；执行后不做任何事后调整。任何变更须记录为 pre-registered deviation。