# Ensembling And Reproducibility

## Project Layout

A practical competition layout:

```text
input/
notebooks/
src/
models/
oof/
submissions/
logs/
configs/
kaggle_kernels/
kaggle_datasets/
remote_outputs/
```

Keep raw downloaded data in `input/` and treat it as read-only. Put repeatable training code in `src/`; use notebooks for EDA, visual checks, and quick diagnostics.

## Script Pattern

Prefer command-driven training:

```bash
python src/train.py --fold 0 --model lgbm --config configs/lgbm.yaml
python src/predict.py --model-dir models/lgbm_fold0 --output submissions/lgbm_fold0.csv
```

Keep model dispatch separate from training loops when many models share the same dataset and metric. Save enough metadata that another run can reproduce the exact artifact.

## Required Artifacts

For each run, save:

- Config/params.
- Fold assignments or fold source file path.
- Per-fold metric and mean/std.
- OOF predictions.
- Test predictions per fold and averaged test predictions.
- Feature list and categorical column list.
- Model files or checkpoints.
- Submission file.
- Kaggle submission receipt/status and public score after scoring.
- Remote run logs and downloaded Kaggle outputs when compute was offloaded.
- Pipeline manifests and intermediate Kaggle dataset handles/versions for multi-notebook runs.

## Safe Blending Workflow

Ensembles help when the members' errors are uncorrelated; they hurt when members are near-duplicates, when a weak member dilutes a strong one, or when weights are tuned on the public leaderboard.

1. Score each candidate independently on OOF: mean, fold std, and where it fails.
2. Check pairwise OOF prediction correlation (Pearson or Spearman). Above roughly 0.90, treat the pair as one source and keep the stronger member; also check error overlap on the validation rows.
3. Start from the best single model. The blend must beat it on OOF or it is not used.
4. Add candidates greedily: add the next best, re-score the OOF blend, keep it only if the score improves; a member that hurts every combination is dropped regardless of its standalone score.
5. Search weights on OOF only, with a small search space. Never tune weights on public LB scores.
6. Submit the blend only if it beats the best single model on OOF; otherwise submit the single model.

Method choice:

- Simple average: diverse members of similar quality.
- Weighted average: members of different quality.
- Rank average: rank-based metrics such as AUC, or members with different calibration.
- Geometric mean: probabilities that need mild recalibration.
- Stacking: many members and enough validation data (see below).

Two or three diverse members usually beat ten correlated ones. Keep a model-correlation or disagreement report in the final consumer for staged architectures.

## Stacking

Stack only with OOF base predictions:

1. Train base models on K-1 folds and predict the held-out fold.
2. Concatenate OOF base predictions into a level-1 train matrix.
3. Average each base model's test predictions across folds for the level-1 test matrix.
4. Train the stacker on the level-1 train matrix using the same or a nested validation plan.

Good stackers are often simple: logistic regression, ridge, linear models, shallow boosting, or constrained weighted averages. Complex stackers overfit quickly.

Blending with a holdout set is stacking with one split instead of full OOF folds. Use it when runtime is limited, but prefer OOF stacking when data is scarce.

## Reproducibility Guardrails

- Set seeds for Python, NumPy, model libraries, and deep-learning frameworks.
- Log package versions and hardware assumptions when results are sensitive.
- Use deterministic data paths and run IDs.
- Avoid hidden notebook state for anything that affects submissions.
- Keep train/test row order explicit through ID columns.
- Version feature generation and preprocessing.
- Make every submitted file traceable to code, config, data, and OOF score.
- Make every Kaggle score traceable to the exact submission file or kernel/version that produced it.
- For Kaggle-offloaded runs, keep the pushed kernel folder, `kernel-metadata.json`, downloaded `experiment_log.json`, `metrics.jsonl`, and output artifacts together under `remote_outputs/RUN_ID/`.
- For multi-notebook runs, keep producer dataset folders under `kaggle_datasets/` and final downloaded outputs under `remote_outputs/CONSUMER_RUN_ID/`.
