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
