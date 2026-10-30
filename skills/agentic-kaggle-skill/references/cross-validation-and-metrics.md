# Cross-Validation And Metrics

Use for predictive ML. Interactive policies use `simulation-game-agents.md` instead of row-level folds.

## Validation Contract

Choose the evaluation unit and hidden-test generalization target before modeling. Save reusable split IDs or a split manifest; a `fold` column is convenient when using cross-validation, not mandatory for a holdout.

**Leakage-safe held-out predictions** are produced without fitting on the evaluation labels or using information unavailable under the intended inference protocol. They may be cross-fitted **OOF predictions** or predictions on an **independent holdout**. Both can support diagnostics, calibration, blending, and stacking when their coverage and provenance are appropriate. Do not call in-sample predictions OOF. A final refit on all permitted training data need not repeat K folds, but its training predictions are not new held-out evidence.

Separate data used to fit or select a correction from data used to confirm it. Repeated tuning on one panel makes that panel selection evidence, not untouched confirmation. A meta-model fitted on held-out base predictions still needs its own assessment; see `ensembling-and-reproducibility.md`.

## Split Selection

- For independent classification examples, use stratification when class coverage matters. For ordinary regression, random folds or a holdout may suffice; target-bin stratification is optional if it improves coverage without violating group/time constraints.
- Group related records and their augmented or near-duplicate variants when the target is transfer to unseen entities, sources, patients, or scenes. Repeated entity IDs alone do not require GroupKFold: forecasting future observations of known stores or users may instead require a time split.
- For ordered events, use time holdouts or expanding windows that match forecast horizon and information availability. Combine group and time constraints when the test requires both; document any coverage tradeoff.
- Choose the number of folds or holdout size for available independent units, class coverage, runtime, and precision. Do not treat a fixed fold count as a universal requirement.

## Leakage And Upstream Exposure

- Fit data-dependent transforms and encoders within training partitions. Cross-fit target-derived training features where needed; never use evaluation labels in their construction. If the rules allow transductive inputs, record that access and evaluate the same protocol.
- Audit duplicate IDs, hashes, text, and source overlap across train/test. Record provenance and label availability before deciding whether overlap is valid signal, contamination, or a join error; do not automatically replace modeling with lookup.
- Record training groups, label exposure, and versions of upstream checkpoints or cached features, including public baselines. A downstream split cannot undo upstream exposure to validation labels. Upstream predictions must respect the chosen group/time protocol and exclude the relevant evaluation targets, or use an independent untouched holdout. Mark unknown coverage and conditional diagnostics explicitly; do not fit a stacker on target-exposed predictions or present unknown coverage as leakage-safe validation.
- When learning corrections across sources, inspect missing-label policies, units, coordinate conventions, class definitions, and annotation timing. Treat differences as hypotheses to validate, not offsets to apply blindly to an unseen group.

## Comparison And Confirmation

- Compare candidates on the same evaluation examples with the official aggregation. Recompute pooled/nonlinear metrics rather than averaging per-example or per-group score deltas; fold mean/std remain useful diagnostics.
- Estimate uncertainty at the independent unit, such as patient, site, or time block. Many clips from two subjects do not establish broad cross-subject validity. Report group coverage, paired differences, and sensitivity where useful.
- Use small panels for debugging and cheap rejection. Before promotion, freeze the candidate and acceptance rule, then confirm on independent groups where feasible. After extensive adaptive tuning, use untouched confirmation or nested evaluation; if infeasible, report the resulting selection uncertainty.
- When public scores disagree with local validation, inspect split semantics, metric implementation, submission format, upstream exposure, and drift before tuning to the leaderboard.

## Metric Contract

Reproduce the competition metric, including transforms, weights, class/ID order, aggregation, clipping, and required output range. Optimize a surrogate only after checking how its changes affect that metric. Tune thresholds and calibration on leakage-safe held-out predictions, keeping final confirmation separate from selection.

For custom scorers, inspect matching, sparse annotation semantics, ignored predictions, count penalties, and caps. An unannotated example is not automatically a negative. Local scorer agreement does not establish hosted-reader compatibility; use `submission-endgame.md` for the submission contract and format checks.
