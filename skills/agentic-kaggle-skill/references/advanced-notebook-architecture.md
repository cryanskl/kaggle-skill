# Choosing A Stronger Architecture

Use when adopting a public baseline, diagnosing a plateau, or considering staged work. Remote GPU use or a required hidden-test notebook can still fit a single kernel; architecture complexity needs its own justification.

## Choose The Intervention Layer

1. Break the metric deficit into components or error clusters. Use ablations or marked development-only hindsight oracles to estimate headroom within the tested candidate set. Oracle outcomes are not a deployable score or a ceiling for all possible methods; interacting component bounds need not add up.
2. Check representability before parameter sweeps: does the candidate set contain the needed outcome, can features/model distinguish it, and do solver constraints or relative costs prevent its selection? Repair a binding bottleneck before tuning a knob that cannot affect it.
3. Compare feasible interventions by expected independent-validation gain, uncertainty, implementation time, inference/training cost, and deadline. A small residual, reranker, calibrator, or feature correction may suffice; audit cached features and checkpoint exposure using `cross-validation-and-metrics.md`.
4. Give promising routes a bounded pilot with promotion/stop evidence. Reserve time for confirmation, handoff, scoring, and a feasible recovery attempt. Record rejected hypotheses and reopening conditions.

A public plateau does not establish a method's ceiling or require retraining. Keep heuristics when independent evidence supports them, and retain a reliable baseline while testing alternatives.

## When Stages Are Useful

Consider staged work when expensive features or models will be reused, independent producers can run efficiently in parallel, or validated model diversity improves a consumer. Public examples, multimodal inputs, or metric structure can motivate a pilot, but do not automatically require a pipeline.

Possible producers include embeddings/features, complementary fold models, pseudo-labels, distilled checkpoints, or calibration/postprocessing assets. Choose only the stages that resolve the current hypothesis; there is no required order of adding embedding, model, and postprocessing stages.

Use `kaggle-pipeline.md` for producer/consumer construction, artifact datasets, manifests, and the scoring boundary rather than recreating that contract here. Export evaluation predictions appropriate to the intended use: leakage-safe OOF or an independent holdout for supervised selection, and unexposed predictions when fitting a stacker. A final full-data checkpoint need not generate fresh K-fold evidence solely for packaging.

## Experiment Admission And Final Selection

An artifact can merit consumer experiments because it shows a credible metric gain, addresses a known error cluster or stability risk, contributes useful diversity, or enables a worthwhile downstream stage. That is experiment admission, not automatic inclusion in the submitted blend.

Check suspicious train/valid gaps, schema drift, unexplainable public-only gains, or irreproducible artifacts before promotion. Evaluate the complete candidate against strong simpler alternatives with `ensembling-and-reproducibility.md`, including costs and group risks. Use `submission-endgame.md` for final selection mechanics, scoring, and completion states.
