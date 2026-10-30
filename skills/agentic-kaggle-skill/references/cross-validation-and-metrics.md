# Cross-Validation And Metrics

## Fold Design Checklist

Choose validation before modeling:

- Identify what one test row represents and how Kaggle likely sampled the hidden set.
- Check duplicate IDs, near-duplicate text/images, repeated users, patients, stores, products, locations, sessions, and timestamps.
- Decide whether labels need stratification, groups need isolation, or time needs ordering.
- Add a persistent `fold` column to the train table and use it in every script.
- Report mean, standard deviation, and per-fold scores.
- Keep OOF predictions for diagnostics, thresholding, blending, and stacking.

## Split Selection

Use `StratifiedKFold` for ordinary classification. For imbalanced targets, keep class ratios stable and inspect minority-class counts per fold.

Use `KFold` for ordinary regression. If the target is skewed or multimodal, bin the target and stratify on bins. A good default bin count is Sturges' rule: `1 + log2(n_rows)`.

Use `GroupKFold` when the same entity can appear multiple times. Common groups are user, patient, product, session, document, author, household, image source, prompt, location, and time block.

Use a stratified group split when both label balance and group isolation matter. If the installed scikit-learn has `StratifiedGroupKFold`, use it; otherwise decide whether group isolation or label balance is the higher risk and document the tradeoff.

Use time-aware validation when chronological leakage is possible. Sort by time and create holdouts that mimic the public/private periods.

## Leakage Checks

- Feature values computed using all rows before splitting can leak validation information. Fit transforms inside each fold.
- Target encoding must be out-of-fold.
- Scaling, imputation, SVD/PCA, vectorizers, oversampling, feature selection, and threshold search must be trained only on training-fold data when they learn from data.
- Augment images only after splitting; never let augmented variants cross folds.
- Near-duplicate rows should stay in the same fold or be removed.
- Check whether test rows (IDs, prompts, texts, image hashes) also appear in train. Exact or near matches change the approach from modeling to lookup and are common in text/QA competitions; record the overlap rate either way.
- If CV and public LB disagree, audit split choice, metric implementation, submission format, and train/test drift before chasing the leaderboard.

## Upstream Exposure And Confirmation

- Record training groups, label exposure, folds, and versions of upstream checkpoints or cached features, including public baselines; mark unknown coverage. A downstream split cannot undo upstream exposure to validation labels. Use upstream predictions that excluded evaluation groups or an independent untouched holdout; otherwise label the result a conditional diagnostic, not end-to-end OOF, and do not train a stacker on exposed predictions.
- Use small panels for debugging and cheap rejection. Before promotion, freeze the candidate and acceptance rule, then confirm on broader independent groups where feasible. After extensive adaptive tuning, use an untouched confirmation set or nested evaluation rather than reusing the selection panel as confirmation.
- Compare on the same examples with the actual aggregation rule. Recompute pooled/nonlinear metrics instead of averaging per-example or per-group score deltas. Report paired differences and group or leave-one-group-out sensitivity when useful.
- Estimate uncertainty at the independent unit, such as patient, site, or time block. Many clips from two subjects do not establish broad cross-subject validity; report coverage and untested groups under a tight budget.
- When learning corrections across sources, audit missing-label policies, units, coordinate conventions, class definitions, and annotation timing first. Treat source differences as hypotheses to validate, not offsets to apply blindly to an unseen group.

## Metric Selection

Use the competition metric exactly. Common mappings:

- Binary classification: log loss, ROC AUC, PR AUC, F1, MCC, accuracy, or balanced accuracy.
- Multiclass: multiclass log loss, macro/micro F1, accuracy, Cohen's kappa.
- Multilabel: per-class log loss, mean F1, label-ranking average precision, mAP.
- Regression: RMSE, RMSLE, MAE, MAPE, R2, correlation, quantile loss.
- Ranking/recommendation: MAP@K, NDCG@K, hit rate, recall@K.
- Segmentation: Dice, IoU/Jaccard, pixel accuracy, boundary-aware metrics.

For custom metrics, inspect the scorer and data contract for matching, sparse annotation semantics, ignored predictions, clipping/caps, count penalties, and aggregation. An unannotated example is not automatically a negative. Local scorer agreement does not establish hosted-reader compatibility; use the format checks in `submission-endgame.md` as well.

When the metric has thresholds, tune thresholds on OOF predictions only. For F1/MCC/kappa, thresholding can matter as much as model choice.

For log-loss style metrics, calibrate probabilities, avoid overconfident clipping mistakes, and validate the exact label order in the submission.

For RMSLE, train or evaluate in log space only if it matches the metric and the target is nonnegative. Clip negative predictions before scoring when required.
