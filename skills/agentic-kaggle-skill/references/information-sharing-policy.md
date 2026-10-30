# Information Sharing Policy

Use before sharing competition-specific code, data, models, artifacts, or reports outside the active team. Generic workflow documentation does not by itself authorize or require publishing competition materials.

## Policy Checkpoint

Check the target competition's rules, data license, and sharing terms for the material and audience involved. Public availability alone is not permission to redistribute it.

Default to these guardrails:

- Share competition code or derived artifacts outside the team only as the rules permit. For active competitions, respect equal-access requirements; private sharing and public sharing can have different restrictions.
- Redistribute raw data, features, embeddings, pseudo labels, checkpoints, predictions, or submissions only when the applicable terms allow it and third-party license/attribution requirements are met.
- Keep API credentials, tokens, cookies, and other secrets out of Git. Necessary non-secret private dataset/kernel refs may be recorded in access-controlled manifests; exclude them from public Git or documentation when disclosure is not authorized.
- Attribute public sources when used; check licenses before copying code, prose, weights, or other assets.
- For active competitions, prefer Kaggle-native public sharing surfaces such as public notebooks or discussion posts when the goal is to make code visible to all participants.
- Keep experiment kernels, artifact datasets, and model outputs private by default; public visibility should be intentional and permitted.
- If rules are ambiguous, keep materials private and record the uncertainty in the run log.

## GitHub Sharing Checklist

Before pushing a competition repository to public GitHub:

Review the exact export or commit set against the terms above: exclude secrets and unauthorized data/artifacts, retain required notices, and avoid disclosing private materials through logs, manifests, or report links. Generic workflow code, permitted training/inference code, config templates, and cleared reports can remain. Artifact versioning and producer/consumer handoff mechanics belong in `kaggle-pipeline.md`.
