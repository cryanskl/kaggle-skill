# Ensembling And Reproducibility

Use the validation definitions in `cross-validation-and-metrics.md`: either leakage-safe OOF predictions or an independent holdout can support model comparison. Interactive agents use their episode workflow instead.

## Artifacts By Run Role

Keep the project's existing layout. Common locations include `input/` for immutable downloads, `processed/` for derived data, `models/`, `submissions/`, and `logs/`; directory names are conventions, not requirements.

Every run needs enough provenance to identify code/config, input versions, validation units/splits when relevant, outcome, and artifact paths. Save additional artifacts according to its role:

- Training: checkpoint/config, preprocessing or label mappings, and features needed to reproduce inference.
- Validation or model selection: held-out predictions with row/group IDs and coverage, metric/aggregation, and comparison evidence.
- Inference or submission: exact output and writer/version; record hosted receipts only when a submission occurred.
- Remote or staged work: retrievable logs and artifact contracts; use `kaggle-pipeline.md` for producer/consumer manifests and handoffs.

Use run IDs and deterministic paths, record relevant seeds/library versions, preserve row identity, and avoid hidden notebook state. Failed experiments need the same provenance as successes, with rejection reasons and evidence that would justify reopening them. A feature or diagnostic run need not fabricate checkpoints, test predictions, or a submission file.

## Safe Blending

1. Evaluate members on comparable leakage-safe held-out predictions. Audit upstream exposure and missing coverage before combining them.
2. Use prediction correlation and error overlap as redundancy diagnostics, not fixed rejection thresholds or proof of shared lineage. A correlated or weaker individual model can still improve a blend.
3. Start with a strong single-model baseline and simple combinations. Greedy addition is one inexpensive search option; use a limited alternative weight search when justified. Tune weights on held-out selection data, not public leaderboard scores.
4. Compare the complete blend with the strongest applicable single model using the declared metric, group risks, uncertainty, and runtime budget. Do not include a member merely because it differs or helps one cluster. Prefer the simpler candidate when the expected benefit is not credible; independently confirm selected gains where feasible.

Simple or weighted averaging is a useful starting point for compatible outputs. Rank transforms can suit rank-based metrics; stacking is useful when validation data and runtime support fitting a meta-model. Keep an error/disagreement report when it helps explain the choice rather than requiring multiple models by default.

## Stacking And Holdout Blending

Fit a stacker on base predictions generated without fitting those base models on the corresponding evaluation labels. This can use cross-fitted OOF predictions or a separate meta-training holdout. Preserve the same group/time boundaries as the underlying task.

Evaluate the fitted stacker on a further untouched set or a suitable nested/cross-fitted procedure. Fitting and scoring the meta-model on the same level-1 rows is not independent evidence, and OOF base predictions alone do not make the meta-model's fitted score OOF. When budget limits a complete assessment, label that limitation rather than claiming end-to-end generalization.

Simple regularized or constrained meta-models are often adequate. Full-data base-model refits for final inference are compatible with earlier validation, but do not relabel their in-sample predictions as held-out. For final artifact submission and completion states, use `submission-endgame.md`.

## Replay At The Submission Boundary

When adapting a public pipeline or changing postprocessing, keep an identifiable baseline and compare local outputs with the actual submission-producing runtime on the same inputs, including the final writer. Use byte identity when determinism and serialization make it meaningful; otherwise declare numerical tolerances and compare IDs, schemas, and task-specific invariants.

Agreement proves implementation consistency on those inputs, not model quality or hidden-test transfer. Make the eventual submitted artifact traceable to the relevant validation, code/config/data versions, and hosted receipt without repeating pipeline handoff requirements here.
