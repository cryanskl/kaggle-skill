---
name: agentic-kaggle-skill
description: Kaggle competition execution and evidence-based post-mortems for predictive ML and simulation agents. Covers validation, public solution intelligence, modeling, replay-based behavior cloning, reinforcement learning, Kaggle compute and artifact pipelines, submission debugging, and score retrieval.
---

# Agentic Kaggle Skill

Treat every competition as a validation problem first and a modeling problem second. The default target platform is Kaggle: prefer Kaggle-native notebooks/scripts, datasets, model artifacts, competition submissions, and score receipts. For execution requests, continue to the applicable Definition Of Done; a local model or CV score alone is not a scored submission.

In commands below, `<skill-dir>` means the directory that contains this SKILL.md.

Choose the request mode before executing the steps below. Predictive ML uses folds and OOF predictions. Simulation/game-agent competitions use complete episodes, action contracts, interactive opponents, and match-level validation: read `references/simulation-game-agents.md` and replace predictive row/fold/OOF steps with its episode workflow. An interactive agent can use a Python/archive submission rather than a hidden-test notebook. For analysis, post-mortems, and skill maintenance, audit the requested sources and actual artifacts, deliver evidence-based conclusions, and do not start training or submissions.

## Setup Checklist (Execution Requests)

For execution requests, check the setup needed by the chosen mode before modeling. Interactive tasks may use an environment and replays rather than prediction tables; do not download or scaffold unused inputs.

1. Kaggle CLI and credentials: `kaggle --version`. Credentials come from `~/.kaggle/kaggle.json` (`chmod 600`) or the `KAGGLE_USERNAME`/`KAGGLE_KEY` environment variables. If missing, ask the user to create an API token at kaggle.com → Settings → API and place the file; do not ask for the token contents in chat.
2. Rules acceptance: the user must accept the competition rules once on the website. A `403 Forbidden` from `kaggle competitions download` or `submit` almost always means the rules are not accepted yet.
3. Data: `kaggle competitions files -c SLUG`, then `kaggle competitions download -c SLUG -p input/` and unzip into `input/`. Preserve downloaded raw files; write derived tables and fold assignments under `processed/` or the project's established derived-data directory.
4. Submission mode and budget: read the competition Overview, Evaluation, Data, and Rules pages (web fetch or the user's browser). Record the metric, the submission format, whether it is a file submission or a code competition (notebook rerun on hidden data), run-time/GPU/internet limits, and this competition's daily submission limit. Quotas differ per competition; plan scoring attempts against the actual limit and count today's attempts with `kaggle competitions submissions SLUG`.
5. Workspace: `python3 <skill-dir>/scripts/scaffold_competition.py --root .` when no project layout exists.

## Predictive ML Execution Loop

This loop and Choose The Path apply to predictive ML. Interactive agents follow `references/simulation-game-agents.md`; the shared setup, hygiene, and completion conditions still apply. Analysis requests use the source/artifact audit described above.

1. Classify the predictive task: binary, multiclass, multilabel, regression, ranking, image, segmentation, text, time series, grouped entities, or a hybrid. Classify the submission path: classic file submission, code competition, or model/adapter submission.
2. Scout public intel (see Competition Intelligence) before major architecture choices; treat it as clues, not authority.
3. Reproduce the competition metric locally, including clipping, transforms, averaging mode, thresholds, and sample weights.
4. Design folds before feature engineering (see Validation Choices). Save a `fold` column into the training data so every experiment shares one comparison surface.
5. Build baselines in this order: (a) constant or metric-only baseline to verify scoring and submission format, (b) fast classical baseline (LightGBM/CatBoost/XGBoost for tabular, TF-IDF plus linear model for text, pretrained backbone for images), (c) strong baseline with stable folds, clean artifacts, and OOF predictions. Submit the first valid file early to confirm the scoring path.
6. Choose the path (next section) and iterate with validation gates: one meaningful change at a time locally, or several independent producers in parallel on Kaggle when they yield comparable OOF artifacts.
7. Submit the final artifact and retrieve the score/status. If scoring fails, run the debugging loop in `references/code-competition-debugging.md` until scored or concretely blocked.
8. Track for every run: local CV (mean and fold std), Kaggle run status, submission score, public LB, seed, code version, data version, artifact paths, and private-LB risk notes.
9. Ensemble only with OOF predictions produced without in-fold leakage.

## Choose The Path (Predictive ML)

Decide after the baseline exists, and revisit whenever the evidence changes.

At public-baseline adoption or a plateau, use `references/advanced-notebook-architecture.md` to check recoverable loss and whether the candidate/model can express the correction before tuning or adding stages. Audit upstream checkpoint exposure with `references/cross-validation-and-metrics.md` before trusting downstream validation.

**Simple path** (default): local compute or one Kaggle training/inference notebook is enough. A required hidden-test notebook or a GPU can still fit this path; follow its scoring contract in `references/kaggle-pipeline.md`.

- Work locally, or in one Kaggle notebook when the data or GPU lives there.
- Folds → boosting or TF-IDF/pretrained baseline → feature and model iteration → blend of a few diverse, individually validated models → `kaggle competitions submit`.
- Stay on this path while a single run remains practical. Escalate when reusable stages or independently useful models justify the added handoff and evaluation cost.

**Staged path**: use when evidence and budget justify reusable or independently evaluated stages, for example:

- Expensive training produces checkpoints or features that several downstream runs can reuse.
- Several model families or feature producers provide validated diversity and can be compared or blended with OOF evidence.
- The data is multimodal, large, or expensive to featurize, or the metric rewards calibration, thresholding, or postprocessing that benefits from several model families.
- CV plateaus while fold variance and error analysis show recoverable errors, or public solutions and discussions show that strong entries use multi-model pipelines.

Staged-path shape (details in `references/kaggle-pipeline.md`):

1. Fewer than five independent producer notebooks per wave, each owning one artifact family: fold checkpoints, embeddings, feature tables, pseudo labels, or OOF/test predictions. Every remote run writes `experiment_log.json`, `metrics.jsonl`, and `artifacts_manifest.json` so results can be retrieved with `kaggle kernels output`.
2. Each durable producer output is promoted to a versioned private Kaggle dataset.
3. One final consumer notebook attaches those datasets, loads from `/kaggle/input/...`, validates manifests, blends/stacks on OOF predictions, writes `submission.csv`, and is the only scoring boundary.
4. Submit the consumer kernel version (`-k USER/KERNEL -v VERSION`) for code competitions, or its `submission.csv` for file competitions.

Once the baseline is trustworthy, test justified architecture improvements within the available budget. Remote execution and multi-stage architecture are separate choices: needing a GPU or a scoring notebook alone does not require several producers. Keep staged work gated by OOF evidence and artifact manifests, and report compute and submission cost.

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
- `references/cross-validation-and-metrics.md`: folds, custom metrics, leakage, upstream checkpoint exposure, and independent confirmation.
- `references/simulation-game-agents.md`: game/simulation agents, replay imitation, executable action policies, BC/RL, interactive evaluation, and agent-competition post-mortems; replaces predictive OOF validation in this mode.
- `references/tabular-workflow.md`: categorical encodings, feature engineering, selection, tuning.
- `references/image-text-workflow.md`: image, segmentation, and NLP approaches.
- `references/kaggle-offload.md`: running heavy jobs on Kaggle and retrieving logs/outputs.
- `references/kaggle-pipeline.md`: submission modes, code competitions, `kaggle_evaluation` inference servers, producer/consumer notebooks, private artifact datasets, the pipeline manifest, and `kagglehub`.
- `references/advanced-notebook-architecture.md`: choose the intervention layer, estimate headroom and cost, and decide when multi-producer architectures are justified.
- `references/ensembling-and-reproducibility.md`: project layout, OOF artifacts, safe blending/stacking, repeatability.
- `references/submission-endgame.md`: execution completion states, submission budget, hosted results, and final reports; includes retaining an existing scored incumbent when no upgrade qualifies within the defined budget. Analysis requests do not require scoring.
- `references/code-competition-debugging.md`: failed, timed-out, OOM, or vaguely failing submissions.
- `scripts/scaffold_competition.py`: create a competition workspace.
- `scripts/make_folds.py`: add a fold column to a training CSV.
- `scripts/prepare_kaggle_kernel.py`: create a self-contained Kaggle kernel (script or notebook) with retrievable experiment logs.
- `scripts/prepare_kaggle_dataset.py`: metadata and commands for versioned private artifact datasets.

Historical case studies and research notes live in the repository's top-level `docs/` folder; they are not needed for normal runs.

## Definition Of Done

For execution requests, do not stop at a plan, scaffold, trained model, notebook run, downloaded output, or local validation score. Continue until one of these is true:

- A competition submission has been sent to Kaggle, Kaggle has processed it, and the public score or submission status has been retrieved and recorded.
- A code-competition final notebook/kernel has consumed all required producer artifacts, has been submitted with its kernel/version reference, and the resulting submission status/score has been retrieved and recorded.
- An interactive agent's exact archive passed hosted validation, and available rated episodes or rating/status receipts were retrieved with observation time and opponent context. A validation self-match establishes execution, not competitive strength or settled rank.
- For a bounded improvement request with an existing scored incumbent, the defined budget/stop rule was reached and no candidate qualified. Preserve the incumbent, report rejection evidence and reopening conditions, and do not submit a weaker replacement merely to obtain a new receipt.
- A concrete external blocker prevents scoring: missing Kaggle credentials, rules not accepted, quota exhausted after retry attempts, competition closed, scoring disabled, required manual UI-only action, unavailable kernel version, or Kaggle service failure. Record the exact blocker and the next command/action needed.

## Validation Choices (Predictive ML)

- Balanced classification: `StratifiedKFold`.
- Imbalanced classification: stratified folds plus metric-sensitive thresholding or class weights.
- Regression: `KFold` for ordinary targets, binned stratified folds for skewed or multimodal targets.
- Same user, patient, product, session, document, or image source appears multiple times: `GroupKFold` or stratified group splitting.
- Ordered events or forecasting: time split, expanding-window validation, or competition-specific period split.
- Images from the same subject or scene: group by subject/scene before augmentation.
- Text with duplicated sources, authors, questions, products, or prompts: group by source key.
- Tiny data: repeated folds or more folds, but keep the final model selection discipline strict.
Compare mean fold score and fold variance, not only the best fold. Treat a public LB jump that disagrees with CV as a validation question before treating it as a modeling victory.

## Modeling Priorities (Predictive ML)

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
  --output processed/train_folds.csv \
  --target target \
  --strategy stratified \
  --n-splits 5
```

Create grouped folds:

```bash
python3 <skill-dir>/scripts/make_folds.py \
  --input input/train.csv \
  --output processed/train_folds.csv \
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
