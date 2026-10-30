# Method Map

This skill packages reusable Kaggle decisions rather than a particular competition's recipe. Use the entrypoint to select the requested mode and its owning references.

## Core Areas

| Area | Use |
| --- | --- |
| Environment setup | Create a repeatable local and Kaggle project layout. |
| Task framing | Identify target type, metric, data constraints, submission mode, and scoring boundary. |
| Validation | Match prediction folds/holdouts or episode evaluation to the target distribution and independence unit. |
| Simulation agents | Use replay action contracts, episode splits, executable policies, interactive match panels, and BC/RL decision gates. |
| Metrics | Reproduce the official metric locally before trusting experiments. |
| Project structure | Separate raw inputs, code/configs, model and validation artifacts, logs, and submissions. |
| Categorical and tabular features | Use encodings, missing-value policy, feature selection, and tuning with fold boundaries. |
| Image and text workflows | Validate labels, transforms, pretrained assets, inference shape, and modality-specific leakage. |
| Ensembling and stacking | Use leakage-safe held-out base predictions and record blend/stack evidence. |
| Reproducibility | Track seeds, code version, data version, run config, metrics, artifacts, and scoring receipts. |
| Kaggle execution | Offload heavy runs, manage producer/consumer notebooks, create private artifact datasets, and retrieve outputs. |

## Distillation Principles

- Convert general competitive ML practice into action rules for Kaggle work, not copied prose from any source.
- Prefer compact workflows, checklists, and reusable scripts over long explanations.
- Keep the scored metric, mode-appropriate validation, and artifact provenance explicit.
- Prefer repeatable execution and evaluate against the actual scoring contract.
- Modernize implementation choices when current Kaggle practice, competition rules, or platform mechanics require it.

Use current competition rules and supported tooling for platform details; the corresponding execution reference owns those procedures. Historical examples in the repository's `docs/` are context, not mandatory runtime instructions.
