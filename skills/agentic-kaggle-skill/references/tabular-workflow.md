# Tabular Workflow

Use `cross-validation-and-metrics.md` for split design, target leakage, upstream exposure, metrics, and confirmation. Reuse the chosen holdout or folds across comparisons.

## Baseline And Representation

Identify target, IDs, groups/time, numeric and categorical columns, missingness, and unavailable-at-inference features. A tree booster is a useful baseline for ordinary tables; sparse linear models or neural representations may be more suitable for the actual inputs and budget. Verify the metric, row identity, and inference path before extensive feature search.

Choose encodings for the model family:

- Tree models: categorical dtype or native categorical support, label/count/frequency encodings, and rare-category handling when useful.
- Linear models: one-hot or hashing, sparse features, and appropriate scaling.
- Neural models: categorical embeddings or other representations when their expected benefit justifies training cost.

For target encoding, cross-fit statistics for training rows within the evaluation split, and compute evaluation statistics from its training partition only. Smooth rare categories when needed; no row may see its own target through an encoding. A final full-data fit can build test-time statistics from the permitted training data.

## Feature Candidates

Try features with a plausible inference-time meaning:

- Missingness and unknown-category flags; category counts/frequencies and cross-category combinations.
- Numeric transforms, ratios, differences, interactions, and group-normalized values.
- Date parts, recency, elapsed periods, lags, and trends using only information available at the prediction time.
- Entity/time/location aggregates whose source rows respect the validation and inference protocol.

Inspect train/test availability and drift rather than deleting a feature merely because the distributions differ. For learned feature construction, follow the validation boundary above.

## Selection And Tuning

Remove accidental duplicates, unusable constants, and known post-outcome leaks. Use fold/holdout error analysis, permutation importance, and model importance as diagnostics; a ranking alone is not a promotion gate. Correlation pruning or recursive elimination are optional when appropriate for the model and budget.

Tune after the baseline and validation are stable. For boosting, useful candidates include learning rate/iterations with early stopping, depth/leaves, minimum child size, sampling, regularization, and class weighting. Use a bounded search suited to cost; random/Bayesian search or small targeted grids are choices, not mandatory stages.

Compare paired validation changes and group regressions, not a lucky fold or a raw comparison against fold standard deviation. Keep simpler alternatives when gains are uncertain; a feature with weak standalone value can still be useful in a validated combination. Save selection history so repeated searches are not presented as untouched confirmation.
