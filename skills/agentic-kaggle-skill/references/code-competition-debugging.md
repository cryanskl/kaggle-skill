# Code Competition Debugging

Use when a submission fails, OOMs, times out, or passes its visible run but fails during hosted rerun. This reference owns diagnosis and retries; `submission-endgame.md` owns completion states. Respect the requested stage and authorized time/compute/submission budget.

## Evidence-Driven Loop

1. Save the exact failure/status, command or connector request, kernel/submission version, elapsed time, logs, and available output. Separate visible execution, hosted rerun, and parsing/scoring failures.
2. Retrieve available evidence, for example:

```bash
kaggle kernels status USER/KERNEL_SLUG
kaggle kernels output USER/KERNEL_SLUG -p remote_outputs/RUN_ID -o
kaggle competitions submissions COMPETITION_SLUG -v -q
```

3. Form a concrete hypothesis from that evidence. A generic HTTP status or vague notebook error does not establish a particular cause or submission mode.
4. Apply a targeted fix and exercise the relevant local/visible contract before another hosted attempt. Change one meaningful cause at a time when feasible.
5. Retry only within the existing authorization and when a new fix or service-state change makes it informative. Record the attempt/result. Do not repeatedly resubmit unchanged code or exhaust quota to infer hidden data.
6. Stop at the authorized phase or agreed budget/stopping rule, and report evidence with `submission-endgame.md`. If no supported next fix remains, explain what new information or scope is needed rather than repeating an unchanged attempt. Do not label an internal failure an external blocker or repeat an approval already supplied.

## Diagnose The Contract That Failed

| Failure class | Evidence to inspect |
| --- | --- |
| Output/format | Required file, archive, server response, path, root layout, IDs/order, schema, labels, NaN/inf, expected counts |
| Visible/hidden inputs | Missing mount vs legitimately empty sample/group; never substitute training rows for test rows; follow `kaggle-pipeline.md` for documented placeholder/gateway behavior |
| Hidden shape | Variable image/text sizes, batch lengths, unseen categories, optional columns, valid empty groups |
| Paths/version drift | Actual mounts, nested folders, case/extension handling, attached artifact versions, missing weights/manifests |
| Dependencies/network | Scoring network rules, compatible packages, mounted code/models, import/version failures |
| Resources | Cold/warm timing, memory/disk, batches/workers, model count, TTA, callback initialization and per-batch limits |
| State | Dependence on earlier interactive cells or unpersisted files; uncontrolled random or mutable state across requests |
| API mismatch | Wrong scoring path: file instead of server, unsupported callback/iterator, wrong archive; use the official starter |
| Silent failure | Swallowed exceptions or skipped records; log affected items and validate expected counts rather than blanket non-empty assertions |
| Rules | Required data/model/code source disallowed or unavailable under the competition terms |

## Test Only Relevant Safeguards

Use public or synthetic inputs to reproduce the suspected failure: boundary sizes, optional fields, valid empty groups, or schema round trips as applicable. Log stage, shapes, counts, dtypes, artifact identity, and the actual exception; fail clearly when required inputs are absent.

For file-output contracts, write atomically and reload through the expected reader. For inference servers, test the documented lifecycle, callback results, and cold/warm timing; a CSV is not automatically required. For model/adapter contracts, inspect the actual archive and metadata.

Reduce resource pressure based on measurement. Smaller batches, fewer workers, lower precision, sequential model loading, or limited TTA are possible treatments, not requirements for every failure. When network access is forbidden, validate attached compatible offline dependencies (`kaggle-pipeline.md`).

## Attempt Record

Reuse the project's retry log or append `debug_attempts.jsonl` records with run/attempt ID, kernel/version, observed failure, evidence-backed hypothesis, patch, test, elapsed cost, and result. Preserve each submitted version and fallback artifact.

A hosted attempt should test a genuine schema, path, dependency, resource, or robustness fix, not hidden labels or test properties. After a result arrives, record its observed status/score in the orchestrator's submission receipt and staged pipeline record when present. A runtime cannot produce its own future hosted receipt.
