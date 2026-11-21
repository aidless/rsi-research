# arXiv Submission Package

**Title**: Auditable Recursive Self-Improvement: A PDP-style Formal Framework for Bi-Level Agent Systems

**Authors**: Liu Zewen, Yimu (OPC one-person AI research practice)

**Date**: 2026-07-21 (Draft v0.4)

**Status**: Pre-release for arXiv submission. Preprint forthcoming; AIDE85 lineage not yet released.

**arXiv class**: cs.AI (primary), cs.MA / cs.CR / cs.SE (cross-list)

## File listing

- `main.tex` — entry point, all sections via \input{}
- `references.bib` — 24 references with full author lists
- `section_1_formatted.tex` ... `section_7_formatted.tex` — 7 main body sections
- `section_8_appendix_A_companion.tex` — Appendix A: Companion Material
- `section_9_appendix_D_p2_proof.tex` — Appendix D: Formal Proof of P2
- `section_10_appendix_C_p1_proof.tex` — Appendix C: Formal Proof of P1
- `section_11_appendix_E_p3_proof.tex` — Appendix E: Formal Statement of P3
- `figures/fig1_architecture.pdf` ... `figures/fig5_cost_analysis.pdf` — 5 real figures

## Build instructions

```
pdflatex main && bibtex main && pdflatex main && pdflatex main
```

Requires: TeX Live 2025+ with `amsmath`, `amssymb`, `amsthm`, `graphicx`, `booktabs`, `enumitem`, `hyperref`, `xcolor`, `geometry`, `lmodern`, `textcomp`.

## Submission metadata

- **Primary category**: cs.AI (Artificial Intelligence)
- **Cross-list categories**: cs.MA (Multiagent Systems), cs.CR (Cryptography and Security), cs.SE (Software Engineering)
- **License**: arXiv non-exclusive license to distribute (default)
- **Comments**: Companion code at [anonymized repository] (5-model robustness in p2_falsification/, DDC simulator in p1_drill/, DDB-Drill design in p3_drill/)

## Key results (single sentence each)

- **P1**: noise-bounded effect size for ignition tests (formal proof in Appendix C, Hoeffding + AR(1) effective sample size)
- **P2**: ungameable µpriv is necessary for Level-2 RSI (formal proof in Appendix D, 3-step gap-monotonicity argument)
- **P3 (conjecture)**: anti-reward-hacking capacity ≡ audit-channel capacity (formal statement in Appendix E, awaiting DDB-Drill execution)
- **Empirical**: P2 meta-predictor achieves AUC = 0.9925 (linear) and 0.9949 (gradient boosting) on synthetic data calibrated to AIDE²'s reported statistics, evaluated across 5 model classes with 10-fold cross-validation (n=1000 agents)

## Honest scope statement

The empirical P2 result is on **synthetic data** calibrated to AIDE²'s reported statistics; real AIDE85 lineage has not been released by Weco AI as of 2026-07-21. The P3 statement is a **falsifiable conjecture** rather than a theorem --- the DDB-Drill injection-attack protocol (Appendix E.5) is the immediate first-step experimental design; full 4-condition × 30-step ablation awaits real-lineage verification.


## Companion P2 reproducibility (2026-08-08)

- experiments/p2_falsification/mu_priv_meta_predictor.py — canonical P2 meta-predictor (numpy-only linear backend; torch optional).
- experiments/p2_falsification/P2_REPRODUCIBILITY.md — provenance of the v0.4 headline numbers and the 2026-08-08 rerun (linear AUC = 0.9970).
- experiments/p2_falsification/artifacts/result_linear_seed0.json — rerun artifact (seed 0, n=1000).
- The v0.4 companion drill files (p1_drill/, p2_drill/, p3_drill/, dde_drill/) were not restored into this repository; see P2_REPRODUCIBILITY.md for the open-item list.