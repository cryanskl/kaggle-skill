# Kaggle Pipeline And Scoring Contracts

Use for notebook reruns, inference servers, model/adapter submissions, or reusable remote artifact stages. `kaggle-offload.md` owns remote execution and logs; `submission-endgame.md` owns submission commands, result collection, and completion states. Read `information-sharing-policy.md` before sharing artifacts.

## Identify The Contract

| Mode | Scoring boundary | Required artifact |
| --- | --- | --- |
| File | Uploaded predictions | Competition-defined file/schema |
| Code/notebook | A specific kernel version rerun on hidden data | Reproducible scoring entry point and required inputs |
| Inference server | Competition gateway calls the submitted callback | Server lifecycle, callback schema, and timing from the starter |
| Model/adapter | Submitted weights or adapter evaluated by the host | Competition-defined archive/layout and metadata |
| Interactive agent | Hosted episodes call the policy | Agent package; see `simulation-game-agents.md` |

These contracts can overlap: a code notebook may host an inference server or produce an adapter archive. Follow its specific scoring boundary rather than forcing every notebook through a prediction CSV.

Read Evaluation, Data, Rules, the official starter, and current CLI help. A generic `400`, `403`, or `ERROR` does not identify the mode; inspect its actual message and context. Do not spend a submission solely to guess the mode. Include all and only the files required by the archive contract, with the required root layout.

Use kernel/version arguments when the contract requires them or supported notebook attribution is useful. Otherwise record the producing kernel/version in the run manifest; a plain prediction file does not inherently require notebook scoring.

## Hidden Rerun And Inputs

- Test from a clean run with explicit dependencies and versioned inputs. Local sibling files are not automatically uploaded with a kernel's `code_file`; inline helpers or provide them through a supported attached input or utility source.
- Resolve required input paths from actual mounts, schemas, or documented discovery. Log a bounded summary of relevant paths rather than assuming a mount prefix or dumping every file.
- A visible test sample may be tiny or empty. Follow the official starter's visible/hidden branch. Use a sample-submission placeholder only when that file-output contract explicitly requires it; servers must run their gateway. Fail clearly on missing required inputs. Never substitute training rows for test rows.
- Match the scoring network, hardware, memory, and time limits. Vendor missing dependencies only when scoring cannot download them; see Offline Dependencies below.
- Avoid fixed public-sample row counts, sizes, sequence lengths, or category sets. Keep valid empty groups distinct from failed data loading.

## Inference Servers

Follow the exact module, class, callback arguments, rerun detection, and lifecycle in the competition's demo. This is an illustrative shape, not a universal API:

```python
models = load_models()

def predict(batch, *extra):
    return predict_batch(models, batch, *extra)

server = competition_server(predict)
if is_hidden_rerun:
    server.serve()
else:
    server.run_local_gateway(public_inputs)
```

Measure initialization and cold/warm callbacks against the documented timing boundary. Preload or cache models where that contract permits it. Validate return types, shapes, IDs, ordering, and batch handling with the local gateway. When the gateway assembles predictions, writing a separate `submission.csv` is not the scoring path.

## Artifact Handoff For Staged Work

A single kernel does not need a producer/consumer graph. When stages have justified reuse:

1. Each producer writes its useful checkpoints, features, predictions, configuration, and logs. Supervised predictions used for selection/stacking must meet `cross-validation-and-metrics.md`'s held-out contract.
2. Record source kernel/version, run/config/data version, artifact paths and checksums, schema, label order, fold or holdout coverage, expected inputs, and loading/inference requirements. Reuse an existing manifest rather than creating overlapping files with the same information.
3. Prefer versioned private datasets for durable shared artifacts. Versioned notebook outputs or approved model sources can also work when their identity and loading contract are explicit.
4. Attach required sources, verify the manifest against mounted files, and load the artifacts before the stage that consumes them. Group small related artifacts when useful; one dataset per producer is not mandatory.

Keep waves small enough for available compute, quota, and review. Choose concurrency from those constraints rather than a fixed producer count. The scoring run must have every required dependency available; do not rely on files left by a previous interactive run.

## Pipeline Record

For multiple stages, maintain a record of dependencies and meaningful changes in status, versions, artifacts, and submission results. Routine unchanged status polls need not rewrite it. A possible `pipeline_manifest.json` is:

```json
{
  "pipeline_id": "exp_042",
  "competition": "competition-slug",
  "producers": [{
    "kernel": "user/train-folds",
    "kernel_version": "8",
    "artifact_source": "user/fold-models/versions/2",
    "outputs": ["fold0/model.safetensors", "oof.parquet"],
    "validation": {"split_manifest": "folds.json", "metric": "auc", "score": 0.912},
    "status": "complete"
  }],
  "consumer": {"kernel": "user/final-inference", "kernel_version": "3"},
  "submission": null
}
```

The consumer writes run/metric/artifact evidence. After hosted submission, the orchestrator records `submission_receipt.json` and updates the pipeline record with the observed status/result; a scoring notebook cannot know its future hosted receipt.

Before scoring, verify required files, schema, group/fold coverage, label order, prediction shape, and measured resource use. Apply blend/stack/calibration checks only when those operations are present. Produce the contract's actual output: file, server response, or model archive. Debug a failed consumer first; rerun producers only when evidence implicates their artifacts.

## Durable Dataset Example

Use existing project tools or the bundled helper, checking current CLI help for platform-specific flags:

```bash
python3 <skill-dir>/scripts/prepare_kaggle_dataset.py \
  --output kaggle_datasets/fold_models \
  --username user --slug fold-models --title "fold models" \
  --description "Versioned fold model artifacts" --artifact-kind model \
  --source-kernel user/train-folds --source-run-id exp_042
kaggle datasets create -p kaggle_datasets/fold_models -t -r zip
kaggle datasets status user/fold-models
```

Use formats that preserve types and directory layout. For subsequent updates, version the dataset and record the consumed version. Attach competition inputs with `competition_sources` and other artifacts through the supported dataset/kernel/model source fields.

## Offline Dependencies

When the scoring environment lacks a required package and cannot use the network:

1. Download compatible wheels and their dependencies in an environment matching the scoring Python, OS, and architecture.
2. Attach the wheel directory through an allowed private input.
3. Install with `pip install --no-index --find-links /kaggle/input/DATASET_SLUG PACKAGE` and verify the import/version in the scoring environment.

Do not assume wheels downloaded on the user's laptop match Kaggle's platform. Preserve package and source versions in the manifest.

## KaggleHub

Use KaggleHub when already working in Python; use the CLI or connector when orchestrating remotely. Pin artifact versions where supported and verify the installed API. Common download patterns are:

```python
import kagglehub
features = kagglehub.dataset_download("user/features")
prior_output = kagglehub.notebook_output_download("user/features-kernel")
```

For runtime commands and output retrieval, use `kaggle-offload.md`; for the final submit/version/result flow, use `submission-endgame.md`.
