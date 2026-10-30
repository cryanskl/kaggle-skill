# Kaggle Compute Offload

Use Kaggle when its data mounts, hardware, runtime, or reproducibility needs make it a suitable execution target. Local execution remains appropriate when it fits the task, budget, and user preferences. Remote execution does not imply a multi-stage architecture or permission to submit.

## Choose And Prepare The Runtime

- Compare local RAM/VRAM, expected runtime, available Kaggle hardware/quota, and scoring-environment requirements. GPU work or full-fold training does not automatically require offload.
- When feasible, smoke-test imports, required paths, one representative batch, metric computation, and output schema before a large run.
- If a game environment is missing locally, check its supported package/version first. Choose an available compatible environment or a pinned source installation when justified; do not silently change the rules version.
- Confirm CLI/connector authentication and the actual competition inputs and resource limits. Inspect current command help when flags or accelerator names are uncertain.

## Kernel Execution

1. Prepare `kernel-metadata.json` and its script/notebook entry point. `scripts/prepare_kaggle_kernel.py` provides a self-contained scaffold; replace its smoke placeholders before treating it as a competition run.
2. Provide helpers and required inputs through mechanisms the platform actually uploads or mounts. Set supported competition/dataset/kernel/model sources and the required network/accelerator settings.
3. Push the kernel, monitor within the authorized budget, then retrieve outputs or failure evidence.
4. Check the run's identity, config, metrics, and artifact manifest. Preserve observed failures and their logs.
5. If the user requested only training, validation, or a code fix, deliver that stage's artifacts. If hosted submission was requested, hand off to `submission-endgame.md`; it owns completion states. Use `code-competition-debugging.md` for failures and `kaggle-pipeline.md` for staged artifact handoffs.

Typical CLI commands, subject to current help:

```bash
kaggle kernels push -p kaggle_kernels/EXPERIMENT
kaggle kernels status USER/KERNEL_SLUG
kaggle kernels output USER/KERNEL_SLUG -p remote_outputs/RUN_ID -o
```

## Retrievable Evidence

Use existing structured project logs when they carry the needed information. The scaffold defaults are:

- `experiment_log.json`: run ID, timestamp, config, code/data versions, task/split, hardware, elapsed time, error state, metric summary, and artifact paths.
- `metrics.jsonl`: relevant fold/epoch/stage measurements; do not invent metrics for a run that has none.
- `artifacts_manifest.json`: produced files, schemas, versions, and checksums when useful.
- Wrapper stdout/stderr and failure logs when available.

Emit a failure summary if the process can recover; a hard OOM may leave only platform logs. Log enough to distinguish a training run, inference run, and smoke scaffold.

## Retrieval Checks

- Retrieve machine-readable artifacts via CLI, connector, or supported download; browser text alone does not establish artifact contents.
- Keep outputs under an identifiable run directory such as `remote_outputs/RUN_ID/`.
- Compare the remote validation to the relevant local baseline when available. For predictive inference, verify row count, IDs, ordering, labels, values, and the actual output file; for agents, use episode/action evidence.
- Retrieve held-out predictions and split metadata before blending supervised models. Reused stage outputs need explicit source/version identity; see `kaggle-pipeline.md`.

## Metadata

Inspect the current platform schema and the competition contract. Typical fields identify the kernel (`id`, `title`, `code_file`, `language`, `kernel_type`), privacy, network and accelerator settings, and attached input sources. Keep experiments private unless publishing is intentional and rules/license-safe (`information-sharing-policy.md`).

A successful push is only a dispatch receipt. Report the requested stage's actual run result, and use `submission-endgame.md` when hosted scoring is in scope.
