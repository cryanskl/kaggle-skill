---
name: agentic-kaggle-skill
description: End-to-end Kaggle competition workflow that runs through a scored submission. Use whenever the user mentions a Kaggle competition, leaderboard, submission, kernel/notebook, code competition, cross-validation folds, OOF predictions, ensembling/stacking, or competitive ML on tabular, text, image, time-series, or simulation data. Covers rules/metric checks, fold design, baselines, Kaggle GPU offload, producer/consumer notebook pipelines with private artifact datasets, kagglehub, hidden-test debugging, submission, and score retrieval. Works with Claude Code, Codex, and Hermes.
---

# Agentic Kaggle Skill

Treat every competition as a validation problem first and a modeling problem second. The default target platform is Kaggle: prefer Kaggle-native notebooks/scripts, datasets, model artifacts, competition submissions, and score receipts. Do not stop at a plan, a trained model, or a local CV score; the work is done when Kaggle has scored a submission or a concrete external blocker is recorded (see Definition Of Done).

In commands below, `<skill-dir>` means the directory that contains this SKILL.md.

## Setup Checklist

Run these before any modeling:

1. Kaggle CLI and credentials: `kaggle --version`. Credentials come from `~/.kaggle/kaggle.json` (`chmod 600`) or the `KAGGLE_USERNAME`/`KAGGLE_KEY` environment variables. If missing, ask the user to create an API token at kaggle.com → Settings → API and place the file; do not ask for the token contents in chat.
2. Rules acceptance: the user must accept the competition rules once on the website. A `403 Forbidden` from `kaggle competitions download` or `submit` almost always means the rules are not accepted yet.
3. Data: `kaggle competitions files -c SLUG`, then `kaggle competitions download -c SLUG -p input/` and unzip into `input/`. Keep `input/` read-only.
4. Submission mode and budget: read the competition Overview, Evaluation, Data, and Rules pages (web fetch or the user's browser). Record the metric, the submission format, whether it is a file submission or a code competition (notebook rerun on hidden data), run-time/GPU/internet limits, and this competition's daily submission limit. Quotas differ per competition; plan scoring attempts against the actual limit and count today's attempts with `kaggle competitions submissions SLUG`.
5. Workspace: `python3 <skill-dir>/scripts/scaffold_competition.py --root .` when no project layout exists.

## Operating Loop

1. Classify the task: binary, multiclass, multilabel, regression, ranking, image, segmentation, text, time series, grouped entities, simulation/game agent, or a hybrid. Classify the submission path: classic file submission, code competition, or model/adapter submission.
2. Scout public intel (see Competition Intelligence) before major architecture choices; treat it as clues, not authority.
3. Reproduce the competition metric locally, including clipping, transforms, averaging mode, thresholds, and sample weights.
4. Design folds before feature engineering (see Validation Choices). Save a `fold` column into the training data so every experiment shares one comparison surface.
5. Build baselines in this order: (a) constant or metric-only baseline to verify scoring and submission format, (b) fast classical baseline (LightGBM/CatBoost/XGBoost for tabular, TF-IDF plus linear model for text, pretrained backbone for images), (c) strong baseline with stable folds, clean artifacts, and OOF predictions. Submit the first valid file early to confirm the scoring path.
6. Choose the path (next section) and iterate with validation gates: one meaningful change at a time locally, or several independent producers in parallel on Kaggle when they yield comparable OOF artifacts.
7. Submit the final artifact and retrieve the score/status. If scoring fails, run the debugging loop in `references/code-competition-debugging.md` until scored or concretely blocked.
8. Track for every run: local CV (mean and fold std), Kaggle run status, submission score, public LB, seed, code version, data version, artifact paths, and private-LB risk notes.
9. Ensemble only with OOF predictions produced without in-fold leakage.

## Choose The Path

Decide after the baseline exists, and revisit whenever the evidence changes.

**Simple path** (default): tabular or modest text/image data, file submission, and local compute or a single Kaggle notebook is enough.

- Work locally, or in one Kaggle notebook when the data or GPU lives there.
- Folds → boosting or TF-IDF/pretrained baseline → feature and model iteration → blend of a few diverse, individually validated models → `kaggle competitions submit`.
- Stay on this path until an escalation trigger fires. A well-tuned single model family plus a small OOF-validated blend wins many competitions; do not build a pipeline for its own sake.

**Staged path**: switch when any of these hold.

- The competition is a code competition (notebook rerun against hidden data) or requires a model/adapter submission.
- Training or inference needs a GPU/TPU, large memory, or Kaggle-only data mounts.
- The data is multimodal, large, or expensive to featurize, or the metric rewards calibration, thresholding, or postprocessing that benefits from several model families.
- CV plateaus while fold variance and error analysis show recoverable errors, or public solutions and discussions show that strong entries use multi-model pipelines.

Staged-path shape (details in `references/kaggle-pipeline.md`):

1. Fewer than five independent producer notebooks per wave, each owning one artifact family: fold checkpoints, embeddings, feature tables, pseudo labels, or OOF/test predictions. Every remote run writes `experiment_log.json`, `metrics.jsonl`, and `artifacts_manifest.json` so results can be retrieved with `kaggle kernels output`.
2. Each durable producer output is promoted to a versioned private Kaggle dataset.
3. One final consumer notebook attaches those datasets, loads from `/kaggle/input/...`, validates manifests, blends/stacks on OOF predictions, writes `submission.csv`, and is the only scoring boundary.
4. Submit the consumer kernel version (`-k USER/KERNEL -v VERSION`) for code competitions, or its `submission.csv` for file competitions.

Do not wait for the user to ask for sophistication. Once the baseline is trustworthy and a trigger fires, plan and execute the staged path, keep it gated by OOF evidence and artifact manifests, and report the cost (Kaggle GPU hours, submission attempts used) in the run log.

## Competition Intelligence

Read `references/competition-intel.md` for the full procedure. Order of preference:

1. `kaggle-competition-intel` MCP tools when installed: `top_open_solutions` and `competition_discussions`.
2. Kaggle CLI, available whenever credentials work:

```bash
kaggle kernels list --competition SLUG --sort-by scoreDescending --page-size 20
kaggle kernels list --competition SLUG --sort-by voteCount --page-size 20
kaggle kernels pull USER/KERNEL_SLUG -p scouting/USER_KERNEL_SLUG -m
kaggle competitions leaderboard SLUG -s
```

3. Web fetch of the competition Discussion and Code tabs, or links the user provides.

Pulled notebooks are scouting material: extract the validation scheme, model families, submission format, attached datasets/models, and internet/GPU settings. Do not copy code wholesale; see `references/information-sharing-policy.md`.

## Resource Map

- `references/method-map.md`: neutral workflow map behind the skill.
- `references/information-sharing-policy.md`: read before publishing code, notebooks, datasets, models, or reports outside the team.
- `references/competition-intel.md`: public notebook and discussion scouting.
- `references/cross-validation-and-metrics.md`: folds, metrics, thresholds, leakage checks.
- `references/tabular-workflow.md`: categorical encodings, feature engineering, selection, tuning.
- `references/image-text-workflow.md`: image, segmentation, and NLP approaches.
- `references/kaggle-offload.md`: running heavy jobs on Kaggle and retrieving logs/outputs.
- `references/kaggle-pipeline.md`: submission modes, code competitions, `kaggle_evaluation` inference servers, producer/consumer notebooks, private artifact datasets, the pipeline manifest, and `kagglehub`.
- `references/advanced-notebook-architecture.md`: when and how to escalate to multi-producer architectures.
- `references/ensembling-and-reproducibility.md`: project layout, OOF artifacts, safe blending/stacking, repeatability.
- `references/submission-endgame.md`: submitting, quota, score retrieval, and the final report; read before stopping.
- `references/code-competition-debugging.md`: failed, timed-out, OOM, or vaguely failing submissions.
- `scripts/scaffold_competition.py`: create a competition workspace.
- `scripts/make_folds.py`: add a fold column to a training CSV.
- `scripts/prepare_kaggle_kernel.py`: create a self-contained Kaggle kernel (script or notebook) with retrievable experiment logs.
- `scripts/prepare_kaggle_dataset.py`: metadata and commands for versioned private artifact datasets.

Historical case studies and research notes live in the repository's top-level `docs/` folder; they are not needed for normal runs.

## Definition Of Done

Do not stop at a plan, scaffold, trained model, notebook run, downloaded output, or local validation score. Continue until one of these is true:

- A competition submission has been sent to Kaggle, Kaggle has processed it, and the public score or submission status has been retrieved and recorded.
- A code-competition final notebook/kernel has consumed all required producer artifacts, has been submitted with its kernel/version reference, and the resulting submission status/score has been retrieved and recorded.
- A concrete external blocker prevents scoring: missing Kaggle credentials, rules not accepted, quota exhausted after retry attempts, competition closed, scoring disabled, required manual UI-only action, unavailable kernel version, or Kaggle service failure. Record the exact blocker and the next command/action needed.

## Validation Choices

- Balanced classification: `StratifiedKFold`.
- Imbalanced classification: stratified folds plus metric-sensitive thresholding or class weights.
- Regression: `KFold` for ordinary targets, binned stratified folds for skewed or multimodal targets.
- Same user, patient, product, session, document, or image source appears multiple times: `GroupKFold` or stratified group splitting.
- Ordered events or forecasting: time split, expanding-window validation, or competition-specific period split.
- Images from the same subject or scene: group by subject/scene before augmentation.
- Text with duplicated sources, authors, questions, products, or prompts: group by source key.
- Tiny data: repeated folds or more folds, but keep the final model selection discipline strict.
- Simulation/game competitions: no folds; validate by local matches against baseline agents when the environment is available, and by score stabilization on Kaggle otherwise.

Compare mean fold score and fold variance, not only the best fold. Treat a public LB jump that disagrees with CV as a validation question before treating it as a modeling victory.

## Modeling Priorities

- For tabular competitions, prefer tree boosting first unless the data is mostly sparse text or high-cardinality categorical interactions. Add CatBoost when categorical columns are central.
- For linear models and neural networks, scale numeric features and encode categoricals carefully.
- For tree models, start with label/count/frequency encodings and sensible missing values; scaling is usually unnecessary.
- For target encoding, always compute encodings out-of-fold, with smoothing and no validation-row target visibility.
- For image tasks, get dataset, transforms, labels, masks, and metric correct before changing architecture.
- For text tasks, keep a TF-IDF baseline even when using transformers; it is fast, interpretable, and often blends well.
- For hyperparameter search, tune only after folds and baselines are trustworthy. Search learning rate, depth/leaves, regularization, sampling, and number of estimators with early stopping.

## Competition Hygiene

- Keep raw data read-only.
- Keep competition data, private datasets, model checkpoints, downloaded outputs, submissions, credentials, and data-derived artifacts out of public Git repositories unless the competition rules and artifact licenses explicitly allow public redistribution.
- Put all experiment-changing values in config or command-line args.
- Seed Python, NumPy, framework, and model libraries when possible.
- Save model files, OOF predictions, test predictions, fold scores, feature lists, and submission files with run IDs.
- For Kaggle-offloaded jobs, write retrievable logs and artifacts into the notebook working directory: run ID, git commit, config, fold, metric, hardware, elapsed time, OOM notes, and output file paths.
- For multi-notebook pipelines, maintain `pipeline_manifest.json` with producer kernel refs, produced dataset refs and versions, artifact schemas, metrics, and the final consumer run ID.
- Keep producer datasets, kernels, and model artifacts private by default. Make them public only after verifying the competition rules, data license, third-party IP, and user intent.
- Record the submission message, command or kernel/version, timestamp, status, public score, and any error returned by Kaggle.
- Never train a stacker on in-sample base predictions.
- Never select features, target encoders, scalers, thresholds, or augmentations using validation targets outside the fold boundary.
- Prefer scripts for repeatable training and notebooks for exploration and plots.

## Useful Commands

Create a competition skeleton:

```bash
python3 <skill-dir>/scripts/scaffold_competition.py --root .
```

Create stratified folds:

```bash
python3 <skill-dir>/scripts/make_folds.py \
  --input input/train.csv \
  --output input/train_folds.csv \
  --target target \
  --strategy stratified \
  --n-splits 5
```

Create grouped folds:

```bash
python3 <skill-dir>/scripts/make_folds.py \
  --input input/train.csv \
  --output input/train_folds.csv \
  --target target \
  --group-col patient_id \
  --strategy group
```

Time-ordered folds (`--time-col`) use expanding windows; the earliest rows are never a validation fold and are written with `fold = -1`.

Prepare a Kaggle GPU kernel experiment folder (script by default, `--kernel-type notebook` for an `.ipynb`):

```bash
python3 <skill-dir>/scripts/prepare_kaggle_kernel.py \
  --output kaggle_kernels/exp_lgbm_gpu \
  --username kaggle-user \
  --slug exp-lgbm-gpu \
  --title "exp lgbm gpu" \
  --competition playground-series-sample \
  --accelerator NvidiaTeslaT4
```

Prepare a private Kaggle dataset folder for trained model artifacts:

```bash
python3 <skill-dir>/scripts/prepare_kaggle_dataset.py \
  --output kaggle_datasets/exp_model_v1 \
  --username kaggle-user \
  --slug exp-model-v1 \
  --title "exp model v1" \
  --description "Trained model artifacts for experiment exp_model_v1" \
  --artifact-kind model
```
