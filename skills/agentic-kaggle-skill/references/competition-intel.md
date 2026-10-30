# Competition Intelligence

Use this reference when a Kaggle competition slug, URL, title, or search phrase is available and public notebook solutions or discussion activity may inform the plan.

Public notebooks and discussions are scouting signals, not permission to copy code, text, data, or private strategy. See `information-sharing-policy.md`.

## Source 1: `kaggle-competition-intel` MCP Tools (when installed)

Use `top_open_solutions` for public notebook intelligence:

```json
{
  "competition": "Kaggle competition URL, slug, title, or search phrase",
  "top_score_x": 5,
  "latest_y": 5,
  "min_latest_votes": 30
}
```

- `top_score_x`: number of public solutions by public score, 1 to 20.
- `latest_y`: number of latest high-vote public notebooks, 1 to 50.
- `min_latest_votes`: latest notebooks are included only when `totalVotes` is strictly greater than this threshold.

Use `competition_discussions` for discussion intelligence:

```json
{
  "competition": "Kaggle competition URL, slug, title, or search phrase",
  "top_votes_x": 5,
  "latest_replies_y": 5
}
```

Use the defaults above when the user has not specified values, and state the scouting depth and vote threshold in the report.

## Source 2: Kaggle CLI (always available with credentials)

```bash
# top public-scored and most-voted notebooks
kaggle kernels list --competition SLUG --sort-by scoreDescending --page-size 20
kaggle kernels list --competition SLUG --sort-by voteCount --page-size 20
kaggle kernels list --competition SLUG --sort-by dateRun --page-size 20   # recently active

# read one notebook's code and its attached inputs
kaggle kernels pull USER/KERNEL_SLUG -p scouting/USER_KERNEL_SLUG -m
# optional: jupytext --to py:percent scouting/USER_KERNEL_SLUG/*.ipynb

# leaderboard context
kaggle competitions leaderboard SLUG -s
```

`-m` also writes the notebook's `kernel-metadata.json`, which reveals attached datasets and models, GPU use, and internet settings. Use `--sort-by scoreAscending` when lower is better for the metric.

## Source 3: Web Fetch Or User-Provided Links

Fetch the competition's Discussion and Code tabs when browsing is available, or read links the user provides. Prioritize pinned/sticky topics, topics with many votes, and topics mentioning data bugs, leakage, metric clarifications, or rule changes.

If none of the sources are available, proceed with the baseline workflow and note that live intel was unavailable.

## How To Use Results

Extract actionable patterns:

- Validation scheme hints: folds, time split, group split, leakage controls.
- Data processing: cleaning, resizing, text preprocessing, feature stores, artifact datasets.
- Model families: tree boosting, CatBoost, transformer, CNN, segmentation model, retrieval/reranker, ensembling.
- Training details: loss, metric surrogate, augmentation, class imbalance handling, thresholding, inference tricks.
- Submission mechanics: file vs kernel submission, `kaggle_evaluation` server, zip format, attached datasets, internet and GPU settings.
- Pipeline shape: single notebook, producer/consumer notebooks, dataset artifacts, Kaggle GPU use.
- Leaderboard risk: suspicious public-only tricks, overly tuned thresholds, or public LB chasing.
- Discussion signals: rule clarifications, data bugs, leakage warnings, metric traps, resource constraints, high-signal Q&A, newly shared techniques.

Adapt ideas into the local validation and artifact discipline; do not paste notebooks in. Do not publish participant names, handles, or private links in generic reports unless the user explicitly needs source attribution for a public resource.

For claims that change the strategy:

- Distinguish a title or author claim, a processed leaderboard score, a reproduced result, and an ablation. Record source, date, artifact version, and evaluation split when known; do not attribute a whole-pipeline score to one component.
- Check shared weights and code lineage. Similar notebooks from one lineage are not independent evidence, and a public plateau does not establish a private-score ceiling.
- Read relevant discussions for scorer, data, and solver issues. If a page is only a JavaScript shell, try permitted UI/API access or saved local material and record any missing evidence. Do not make a temporary undocumented endpoint a permanent dependency or bypass access controls.
- In post-mortems, compare final held-out/private or tournament results where applicable, validation design, ablations, resource accounting, and what actually shipped. Cross-team score differences suggest hypotheses, not causal component gains; preserve contradictory successful methods as conditional branches.

## Reporting

When summarizing intel, include:

- Competition slug/title and metric.
- Score-ranked notebook title, public score, votes, last run time, and URL.
- Latest high-vote notebook title, votes, last run time, public score if available, and URL.
- Top-voted and latest-replied discussion title, votes, comment count, latest reply time, and URL.
- Common technique themes.
- Which ideas are worth testing locally.
- Which ideas look risky because they may depend on public leaderboard overfitting or leakage.
