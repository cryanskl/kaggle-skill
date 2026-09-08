# Kaggle Pipeline: Submission Modes, Code Competitions, And Artifact Datasets

Use this reference when the competition scores a notebook/kernel, reruns code on hidden data, provides a `kaggle_evaluation` inference server, or when the solution needs several remote stages, intermediate artifacts, or `kagglehub` access.

Read `information-sharing-policy.md` before making any kernel, dataset, model, or downloaded output public.

## Submission Modes

| Mode | What Kaggle scores | How to submit |
| --- | --- | --- |
| File submission | A prediction file you upload | `kaggle competitions submit SLUG -f submission.csv -m MSG` |
| Code competition | A notebook/kernel version rerun against hidden test data | `kaggle competitions submit SLUG -f submission.csv -k USER/KERNEL -v VERSION -m MSG` |
| Model/adapter submission | Model weights, a LoRA adapter, or ONNX files, usually zipped | Per the competition page; often a kernel that writes the archive |

Discover the mode cheaply: read the Overview/Evaluation page; if still unclear, upload a valid placeholder file directly. A file that scores means file submission is accepted; an immediate `ERROR` or a rejection message means the competition is code-only. For zipped submissions, zip only the required file (`zip submission.zip submission.csv`), never the whole folder.

Always pass `-k USER/KERNEL -v VERSION` when a notebook produced the file, even if a plain upload would be accepted: it links the submission to the code for traceability and satisfies competitions that require notebook attribution.

## Code Competition Rerun Rules

- Kaggle reruns the submitted kernel version on hidden data. The visible run sees only the public test sample, which may be tiny or a placeholder. Never fall back to training data when the test folder looks empty; write a valid placeholder from `sample_submission.csv` instead so the visible run stays honest and the hidden rerun does real inference.
- Discover input paths at runtime and print them (`os.walk('/kaggle/input')`). The competition mount is usually `/kaggle/input/SLUG/` but some competitions mount under `/kaggle/input/competitions/SLUG/`; never hard-code a guess.
- Internet is normally disabled during the rerun. Vendor dependencies (see Offline Dependencies) and attach models and code as datasets.
- Respect the published limits: total run time, GPU/CPU choice, memory, and, for inference servers, the per-batch time limit.
- Keep scoring logic in a single script-style entry point; do not rely on notebook cell state, cached globals, or files produced in an earlier run.

## `kaggle_evaluation` Inference Servers

Some code competitions do not score a file you write. Instead the competition data ships a `kaggle_evaluation/` Python package, and the notebook must start an inference server that Kaggle calls batch by batch.

Detect it: `kaggle competitions files -c SLUG` or the Data tab lists `kaggle_evaluation/`, and the competition's demo notebook imports it.

Required notebook shape (copy the exact module, class, and argument names from the competition's demo notebook; they differ per competition):

```python
import os
import kaggle_evaluation.SLUG_inference_server as kes  # exact name from the demo notebook

MODELS = load_models_once()  # load everything before serving

def predict(test_batch, *extra):
    # return predictions for this batch in the format the competition specifies
    ...

inference_server = kes.SLUGInferenceServer(predict)
if os.getenv("KAGGLE_IS_COMPETITION_RERUN"):
    inference_server.serve()
else:
    inference_server.run_local_gateway(("/kaggle/input/SLUG/test.csv",))
```

Rules for this mode:

- Load models, tokenizers, and feature stores once, before `serve()`; the per-batch time limit is strict and applies to every `predict` call.
- `predict` must return the exact type and shape the gateway expects; validate against the demo notebook's return value.
- The local gateway run in the visible notebook is the smoke test; it must complete without errors before submitting.
- Writing `submission.csv` yourself does nothing in this mode; the gateway assembles the submission.

## Producer Model Dataset Pattern

For staged architectures, producer notebooks train models and publish their outputs as private Kaggle datasets:

1. The producer runs on Kaggle GPU/TPU when useful and writes to `/kaggle/working`: checkpoints (`.pt`, `.pth`, `.ckpt`, `.safetensors`, `.pkl`, `.joblib`), tokenizer/config files, label maps, fold metadata, OOF predictions, test predictions, and logs.
2. It writes `model_manifest.json` and `artifact_dataset_manifest.json`: source kernel and version, run ID, fold coverage, model family, training config, metric summary, expected input schema, prediction columns and class order, device and batch-size guidance, and checksums when available.
3. After the producer completes, download its output and create or version a private Kaggle dataset from those files.
4. Downstream notebooks attach that dataset through `dataset_sources` in `kernel-metadata.json`.
5. Consumer code loads from `/kaggle/input/DATASET_SLUG/...`, validates the manifest, then performs inference, blending, stacking, calibration, or postprocessing.

Use `kernel_sources` only for short-lived notebook-output chains. Prefer datasets for trained models: they are versioned, durable, reusable, and explicit in the final consumer's input list.

## Dependency Graph

Keep the graph shallow and auditable:

- Producer nodes create one artifact family each: fold checkpoints, embeddings, feature tables, pseudo labels, calibration data, distilled models, OOF/test predictions, or preprocessing assets.
- Consumer nodes combine or use those artifacts: inference, blend, stack, calibrate, threshold, postprocess, and write the final submission.
- Use fewer than five independent producers per wave. Start a second wave only after reviewing OOF metrics, artifact manifests, and resource cost. If more producers seem necessary, batch them into waves and document why.
- The final consumer is the scoring boundary. It must consume all required producer datasets before scoring and must not depend on rerunning producers during scoring.

## Pipeline Manifest

Maintain one `pipeline_manifest.json` locally and in final outputs. It is the agent's memory between remote runs; update it after every push, status poll, dataset create/version, output retrieval, submission, retry, and score retrieval.

```json
{
  "pipeline_id": "exp_042",
  "competition": "competition-slug",
  "kind": "code_competition",
  "producers": [
    {
      "stage": "train_transformer_fold_models",
      "kernel": "user/exp-042-train-transformer",
      "kernel_version": "8",
      "dataset": "user/exp-042-transformer-models",
      "dataset_version": "2",
      "created_from": "producer_notebook_output",
      "outputs": ["fold0/model.safetensors", "model_manifest.json", "oof.parquet"],
      "metric": {"name": "auc", "mean": 0.912, "std": 0.004},
      "status": "complete"
    }
  ],
  "consumer": {
    "kernel": "user/exp-042-final-inference",
    "kernel_version": "3",
    "consumes_datasets": ["user/exp-042-transformer-models/versions/2"],
    "outputs": ["submission.csv", "oof_final.parquet"],
    "submission": {
      "status": "complete",
      "public_score": "0.12345",
      "submitted_at": "2026-06-11T00:00:00Z"
    }
  }
}
```

## Artifact Dataset Rules

Use Kaggle datasets for intermediate artifacts when outputs are large, expensive to recompute, needed by multiple notebooks, useful across experiments, or should be versioned. Intermediate artifacts derive from competition data and stay private unless the rules and licenses clearly allow redistribution.

- One dataset per logical stage or reusable artifact family.
- Prefer Parquet/NumPy/NPZ/safetensors/checkpoint formats that preserve types; avoid accidental CSV conversion.
- Include schema files for feature tables and prediction files.
- Include `artifact_dataset_manifest.json` with run ID, source kernel and version, source config, fold coverage, row counts, checksums when available, and file descriptions.
- Use `kaggle datasets create` for the first version and `kaggle datasets version` for later versions. Use `-t` (`--keep-tabular`) for Parquet and other files that must not be converted, and `-r zip` when nested checkpoint folders are present.
- Record the dataset handle and version in `pipeline_manifest.json` before running consumers.

## Final Consumer Requirements

Before submitting a code-competition kernel/version, the final consumer must:

- Attach every required producer dataset through `dataset_sources`; keep competition data mounted through `competition_sources`.
- Load model files only from attached datasets, competition inputs, or approved `model_sources`.
- Validate manifests, file existence, fold coverage, expected columns, label order, and prediction shape before inference; fail early with an actionable message if anything is missing.
- Avoid hidden-data assumptions: fixed row counts, fixed image sizes, fixed text lengths, unseen categories, empty groups.
- Use conservative memory settings: small batches, limited workers, cleanup between models, compressed artifacts, limited TTA.
- Compare model correlation/diversity and use OOF predictions for blend weights, stackers, thresholds, and calibration.
- Write `submission.csv` (unless an inference server owns the output), `experiment_log.json`, `metrics.jsonl`, `pipeline_manifest.json`, and `submission_receipt.json`.
- Log which artifact versions were consumed.

If scoring fails, debug the final consumer first (`code-competition-debugging.md`); rerun producers only when the failure implicates a missing, corrupt, incompatible, or rules-invalid producer artifact.

## Offline Dependencies

When internet is disabled during scoring and a package is missing from the Kaggle image:

1. In an internet-enabled notebook or locally: `pip download PACKAGE -d packages/` (matching the Kaggle image's Python version and platform).
2. Upload `packages/` as a private dataset and attach it to the scoring kernel.
3. Install at the top of the kernel: `pip install --no-index --find-links /kaggle/input/DATASET_SLUG PACKAGE`.
4. Verify the import and version before loading models.

## Commands

Producer run:

```bash
kaggle kernels push -p kaggle_kernels/exp_042_train_transformer --accelerator NvidiaTeslaT4
kaggle kernels status user/exp-042-train-transformer
kaggle kernels output user/exp-042-train-transformer -p kaggle_datasets/exp_042_transformer_models -o
```

Promote producer output to a private model artifact dataset:

```bash
python3 <skill-dir>/scripts/prepare_kaggle_dataset.py \
  --output kaggle_datasets/exp_042_transformer_models \
  --username user \
  --slug exp-042-transformer-models \
  --title "exp 042 transformer models" \
  --description "Trained fold model artifacts from exp_042_train_transformer" \
  --artifact-kind model \
  --source-kernel user/exp-042-train-transformer \
  --source-run-id exp_042_train_transformer
kaggle datasets create -p kaggle_datasets/exp_042_transformer_models -t -r zip
kaggle datasets status user/exp-042-transformer-models
```

Final consumer metadata attaches the produced dataset:

```json
{
  "dataset_sources": ["user/exp-042-transformer-models"],
  "competition_sources": ["competition-slug"],
  "kernel_sources": []
}
```

Run the consumer and submit:

```bash
kaggle kernels push -p kaggle_kernels/exp_042_final_inference
kaggle kernels status user/exp-042-final-inference
kaggle kernels output user/exp-042-final-inference -p remote_outputs/exp_042_final -o
# code competition: submit the kernel version
kaggle competitions submit competition-slug -f submission.csv -k user/exp-042-final-inference -v KERNEL_VERSION -m "exp_042 final consumer"
# file competition: submit the downloaded file
kaggle competitions submit competition-slug -f remote_outputs/exp_042_final/submission.csv -m "exp_042 final consumer"
kaggle competitions submissions competition-slug -v -q
```

Determine `KERNEL_VERSION` from the kernel status/output metadata or the Kaggle UI. If it cannot be determined, record that as a blocker together with the final kernel ref.

## KaggleHub In Python

Use `kagglehub` when the code is already running in Python or inside Kaggle; use the CLI when orchestrating from the local workspace.

```python
import kagglehub

competition_dir = kagglehub.competition_download("competition-slug")
features_dir = kagglehub.dataset_download("user/exp-features")
prior_output_dir = kagglehub.notebook_output_download("user/exp-features-kernel")
kagglehub.dataset_upload("user/exp-features", "kaggle_datasets/exp_features", version_notes="new features")
```

Prefer explicit output directories; use force flags only when intentionally overwriting cached artifacts.
