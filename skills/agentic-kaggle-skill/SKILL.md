---
name: agentic-kaggle-skill
description: Kaggle competition execution and evidence-based post-mortems for predictive ML and simulation agents. Covers validation, public solution intelligence, modeling, replay-based behavior cloning, reinforcement learning, Kaggle compute and artifact pipelines, submission debugging, and score retrieval.
---

# Agentic Kaggle Skill

Start with the requested outcome, scoring contract, and credible validation. Prefer the simplest workflow that can answer the next useful question within the user's budget. `<skill-dir>` means the directory containing this file.

## Select The Scope And Mode

Respect the requested phase. A training-only, validation-only, debugging, or artifact-preparation request ends at that phase; using this skill does not add a submission or publication request. For end-to-end scoring requests, use `references/submission-endgame.md` for completion states and receipts. For analysis or skill maintenance, deliver the requested audit without starting training or submissions.

| Evaluation mode | Validation and workflow |
| --- | --- |
| Predictive ML | Match the split and metric to the hidden-test risk. Use leakage-safe held-out evidence: cross-fitted OOF or an independent holdout. Read `references/cross-validation-and-metrics.md`. |
| Interactive simulation/game agent | Use complete episodes, an executable action contract, and representative interactive opponents. Read `references/simulation-game-agents.md`; do not substitute row folds or imitation loss for game strength. |
| Retrospective | Read primary sources and actual team artifacts. Separate author claims, reproduced results, local measurements, and inference. Use `references/competition-intel.md` for source and attribution rules. |

Evaluation mode is separate from submission format: a prediction file, notebook/kernel, inference server, model/archive, or agent package can each have its own contract. Inspect the competition's rules and demo rather than infer the mode from a filename or generic error.

## Execution Loop

Apply only the steps needed by the requested phase:

1. **Establish the contract.** Read rules, data-use terms, metric, allowed information, artifact format, resource limits, and final-selection or active-slot rules. Check needed tools, supported authentication, and rules acceptance with read-only operations; never ask for token contents in chat. Preserve existing raw inputs and project layout.
2. **Freeze a useful comparison.** Define the metric, validation units or opponent panel, baseline, budget, and promotion/stop evidence. Keep derived data separate from raw files. Audit upstream checkpoint exposure before calling downstream predictions held-out.
3. **Build the smallest viable baseline.** Choose a method appropriate to the modality and budget, and verify its full output/action path. A constant, classical, or pretrained baseline is an option, not a mandatory sequence. Keep a known valid artifact while experimenting.
4. **Choose the intervention.** Scout relevant public solutions and discussions before expensive architecture decisions. At baseline adoption or a plateau, inspect recoverable loss and whether the current representation/candidate set can express the correction. Use bounded pilots to compare plausible improvements; retain failed evidence and reopening conditions.
5. **Confirm the candidate.** Compare on the same validation surface with the real aggregation rule and appropriate uncertainty. Separate selection data from independent confirmation where feasible. Evaluate the complete deployed path, including postprocessing, writer/decoder, dependencies, and runtime limits.
6. **Deliver the authorized result.** For scoring requests, submit the qualified artifact through its actual contract and collect hosted evidence using `references/submission-endgame.md`. For a limited phase, report its artifacts and remaining work. For failures, preserve diagnostics and use `references/code-competition-debugging.md` within the available retry budget.

## Local, Remote, Or Staged

- Work locally or in one Kaggle kernel when sufficient. GPU use or a hidden-test notebook alone does not require several producers; use `references/kaggle-offload.md` for remote execution.
- Add reusable stages or diverse producers when expected validation benefit justifies handoff, compute, and inspection cost. Use `references/advanced-notebook-architecture.md` to choose the layer, and `references/kaggle-pipeline.md` for artifact and scoring contracts.
- Select blends/stacks using leakage-safe held-out predictions, never in-sample base predictions or public-LB weight sweeps. Use `references/ensembling-and-reproducibility.md` for construction and final comparison.

## Resource Map

Read relevant references as needed, rather than loading the whole directory:

| Reference | When to read |
| --- | --- |
| `references/cross-validation-and-metrics.md` | Split choice, held-out/OOF definitions, upstream exposure, custom metrics, confirmation and uncertainty. |
| `references/simulation-game-agents.md` | Replay data, action semantics, executable policies, BC/RL, interactive evaluation. |
| `references/competition-intel.md` | Public solutions/discussions, evidence grades, source gaps and post-mortems. |
| `references/tabular-workflow.md` | Tabular encodings, features, selection and tuning. |
| `references/image-text-workflow.md` | Image, segmentation, text and multimodal choices. |
| `references/advanced-notebook-architecture.md` | Plateau, intervention layer, headroom, producer experiments and cost. |
| `references/kaggle-offload.md` | Remote run setup, logs, status and output retrieval. |
| `references/kaggle-pipeline.md` | Submission modes, inference servers, artifact datasets, dependency graph and manifest. |
| `references/ensembling-and-reproducibility.md` | Blending/stacking, provenance and submission-boundary replay. |
| `references/submission-endgame.md` | Authorized submission, completion/stop states, receipts and final selection. |
| `references/code-competition-debugging.md` | Failed or vague hosted runs, diagnosis and bounded retries. |
| `references/information-sharing-policy.md` | Before sharing code, reports, notebooks, datasets or models outside the team. |
| `references/method-map.md` | Short orientation to the skill's areas. |

## Artifacts And Helpers

Keep run IDs, configuration, code/data versions, validation identity, applicable metrics, and artifact paths traceable. Save held-out predictions for experiments that use them; final full-data fitting does not itself produce OOF evidence. Remote logs, pipeline manifests, and submission receipts belong to their respective stages, not every run.

Keep raw data unchanged, secrets out of logs/Git, and competition-derived artifacts within allowed sharing boundaries. Use existing authorizations and record unresolved constraints without inventing approvals or external blockers.

The bundled helpers are optional when an existing project already provides equivalent tooling. Inspect their `--help` for supported arguments:

- `scripts/scaffold_competition.py`: a new competition workspace.
- `scripts/make_folds.py`: predictive fold assignments; time splits may leave warm-up rows with `fold = -1`.
- `scripts/prepare_kaggle_kernel.py`: a self-contained script/notebook starter with retrievable run logs.
- `scripts/prepare_kaggle_dataset.py`: versioned artifact dataset metadata.

Historical cases and maintenance records live in the repository's top-level `docs/`; installed skills do not need those records to run.
