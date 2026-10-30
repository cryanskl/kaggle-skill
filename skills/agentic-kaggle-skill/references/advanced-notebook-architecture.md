# Advanced Notebook Architecture

Use this reference when a Kaggle competition is likely to benefit from a stronger staged solution rather than a single baseline notebook.

## When To Escalate

Consider a staged experiment after a metric-correct baseline exists when these signals suggest a useful gain:

- Public notebook/discussion intel shows strong solutions using multiple models, external pretrained features, staged inference, stacking, pseudo-labeling, or custom postprocessing.
- The data is multimodal, high-dimensional, large, grouped, time-based, image/text-heavy, or expensive to featurize.
- A single baseline plateaus while fold variance and error analysis show recoverable errors.
- The metric rewards calibration, ranking, thresholding, segmentation postprocessing, sequence aggregation, or ensembling.
- Expensive embeddings or fold-model outputs can be reused across independently evaluated downstream experiments.

Do not overcomplicate before the split, metric, submission format, and one reliable baseline are correct.

These signals justify considering a staged experiment, not automatically creating several producers. Needing remote GPU execution or a hidden-test notebook alone can be handled by one kernel; add stages when artifact reuse or measured model diversity justifies their cost.

## Choose The Intervention Layer

At public-baseline adoption or a plateau:

1. Break the metric deficit into components or error clusters. Use ablations or clearly marked development-only hindsight oracles to estimate headroom within the tested candidate set. An oracle can use unavailable outcomes; it is not a deployable score or a ceiling for every possible method, and interacting component bounds need not add up.
2. Check representability before sweeping parameters: does the candidate set contain the needed outcome, can the features/model distinguish it, and do solver constraints or relative costs prevent its selection? Repair that bottleneck when a desired correction cannot be expressed.
3. Compare feasible interventions by expected independent-validation gain, uncertainty, implementation time, training/inference cost, and deadline. Cached embeddings, logits, or candidate features may support a small residual, reranker, calibrator, or specialist; audit upstream exposure using `cross-validation-and-metrics.md` before treating their validation as OOF.
4. Give promising routes a bounded pilot and promotion/stop evidence. Reserve time for confirmation, artifact handoff, scoring latency, and a feasible recovery attempt. Keep rejected hypotheses and reopening conditions in the run record.

A public plateau does not prove a method's ceiling or require retraining. Keep heuristics when independent evidence supports them, and add learned components or stages when their expected benefit justifies the cost.

## Architecture Pattern

Use a shallow DAG:

1. Baseline notebook/script: metric-correct OOF/test predictions and valid submission.
2. Producer notebooks/scripts: independent heavy stages that can run in parallel, especially model-training producers for code competitions.
3. Artifact datasets: each useful producer output is saved/versioned as a private Kaggle dataset; trained producer models should be exported with checkpoints, configs, OOF predictions, and manifests.
4. Consumer notebook/script: attaches producer datasets, loads models from `/kaggle/input/...`, validates artifacts, blends/stacks/postprocesses, writes final submission, submits for scoring, and records score/status.

Prefer a second wave after reviewing OOF artifacts over launching a sprawling graph (wave size limits are in `kaggle-pipeline.md`).

## Producer Ideas

Choose producers that create diversity:

- Tabular: LightGBM/XGBoost/CatBoost variants, neural tabular embeddings, target/count/stat features, time/group aggregations, adversarial-drift features.
- Text: TF-IDF linear baseline, transformer embeddings, fine-tuned transformer logits, retrieval/reranking features, language/source metadata.
- Images: different backbones/resolutions, segmentation masks, crop generators, test-time augmentation predictions, embedding extractors.
- Time series: rolling/lag feature stores, sequence models, entity-level aggregate features, horizon-specific models.
- Multimodal: separate text/image/tabular producers plus a late-fusion consumer.
- Meta-modeling: calibrated probabilities, threshold search, rank averaging, stacking, pseudo-label generation, distillation, uncertainty estimates.

Every producer must write OOF predictions when it trains on labels. Without OOF predictions, its output is not safe for stacking.

When a producer trains a model for a downstream code-competition consumer, it must also write a model artifact dataset contract: checkpoint paths, tokenizer/config paths, label order, fold IDs, expected input schema, inference batch-size guidance, metric summary, and source kernel/version.

## Promotion Gates

Promote a producer output to the final consumer when at least one is true:

- It improves mean CV beyond noise.
- It reduces fold variance or fixes a known error cluster.
- It adds ensemble diversity despite not winning alone.
- It enables a downstream stage such as pseudo-labeling, postprocessing, or calibration.

These gates admit artifacts to consumer experiments; they do not automatically qualify a final submitted blend. Evaluate the complete blend against the strongest single-model baseline with the selection procedure in `ensembling-and-reproducibility.md`.

Reject or quarantine producers with suspicious train/valid gaps, schema drift, unexplainable public LB-only gains, or artifacts that cannot be reproduced.

## Consumer Responsibilities

The final consumer is the only scoring boundary for the full architecture unless the user asks otherwise. Its requirements (attached datasets, manifest validation, OOF-only blending, required outputs, submission) are listed once in `kaggle-pipeline.md`, Final Consumer Requirements; the safe blending procedure is in `ensembling-and-reproducibility.md`.

## Practical Escalation Order

1. Strong single-model baseline.
2. Add one high-signal feature/embedding producer.
3. Add one diverse model-family producer.
4. Add one domain-specific postprocessing or calibration stage.
5. Blend or stack producers using OOF predictions.
6. If score improves and resources remain, start a second producer wave.

If Kaggle submission quota is scarce, prefer OOF-validated improvements and visible kernel tests before spending another scoring attempt.
