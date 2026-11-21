# TMLR Submission (2026-08-08)

**Title**: Auditable Recursive Self-Improvement: A PDP-style Formal Framework for Bi-Level Agent Systems
**Venue decision**: TMLR (converted from arXiv v0.4 style, 2026-08-08)
**Build**: `main.pdf` = 35 pages, 0 errors, 0 undefined citations (pdflatex + tmlr.sty/bst)

## Contents
- `main.tex` + 11 section files + `references.bib` + `figures/` — full TMLR source
- `tmlr.sty` / `tmlr.bst` — official TMLR style (from `[TMLR style sources]`)
- `main.pdf` — compiled preprint version
- `bi_level_pepa_tmlr_submission_2026-08-08.zip` — OpenReview submission package

## Double-blind changes vs arXiv v0.4
- Author block anonymized (Anonymous; code repo linked at acceptance)
- All personal repository references were replaced with `[anonymized repository]` throughout the TeX sources and bibliography
- `\date{}` cleared; arXiv version stamp neutralized
- Third-party URLs (e.g., WecoAI/aideml) retained

## Known open item (pre-submission)
- P1/P3 companion drill files (`p1_drill/`, `p3_drill/`) are not restored; see
  `experiments/p2_falsification/P2_REPRODUCIBILITY.md`. P2 is re-runnable
  (canonical script, rerun AUC 0.9970).
## Pre-registered experiment cards
- pre_registration/P1_EXPERIMENT_CARD.md — P1 ignition-test noise-bound protocol (falsification criteria, grid, decision rules)
- pre_registration/P3_EXPERIMENT_CARD.md — P3 DDB-Drill injection protocol (4 α-classes × 30 steps × 100 trials)