# Image And Text Workflow

Use `cross-validation-and-metrics.md` for group/time splits, leakage-safe held-out predictions, upstream model exposure, and confirmation. Choose experiments for the data, allowed assets, and compute budget rather than requiring every model family.

## Images

Verify paths, labels, orientation, channels, corrupted files, duplicates, and source/subject identity. Match preprocessing between training and inference, record resolution, and keep augmentations plausible for the label semantics. Respect evaluation groups before augmenting.

A pretrained backbone is a useful candidate when rules, input domain, and runtime permit it. Validate against the competition metric or an understood surrogate. Preserve held-out predictions for comparison; try class sampling/loss changes, mixup/cutmix, resolution diversity, or test-time augmentation only when error analysis and budget justify them.

Inspect errors by class, source, lighting, resolution, or duplicate group where informative. More backbones and larger images are not automatic upgrades.

## Segmentation

Check mask alignment, dimensions, label values, missingness, and annotation coverage before modeling. Choose a loss that supports the official metric and verify the effect of changes on that metric. Missing masks or unannotated pixels must follow the actual label contract.

Tune thresholds, components, holes, or object-size postprocessing on leakage-safe held-out predictions. Use the same inference and output-writing path during confirmation; see `ensembling-and-reproducibility.md` for submission-boundary replay.

## Text

Start with an economical baseline appropriate to the task. Sparse word/character TF-IDF with a linear classifier or regressor can be informative; a pretrained neural model may be preferable when semantics and available assets justify it. Running both tracks is optional.

Preserve useful punctuation, casing, and domain tokens until validation supports removing them. For neural models, inspect tokenization, length/truncation, label encoding, and collation. Compare predictive benefit and inference cost with simpler alternatives.

Optional complementary features include text length, punctuation/case statistics, language/source markers, and SVD features. Model probabilities used as meta-features must satisfy the held-out contract, not come from predictions fitted on those same labels.

## Mixed Modalities

Use comparable evaluation units and splits to compare unimodal, early-fusion, or late-fusion candidates. Cheap unimodal ablations can reveal which inputs matter, but building every modality's strongest standalone model is not a prerequisite for fusion.

Add modality-specific producers only when they provide reusable artifacts or validated complementary information. Blend or stack with the procedure in `ensembling-and-reproducibility.md`; do not infer benefit from architecture complexity alone.
