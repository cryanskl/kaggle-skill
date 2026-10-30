# Submission Endgame

This is the owner of completion states and hosted submission/result collection. Respect the user's requested stage: training, local validation, a code fix, analysis, or skill maintenance can finish at that stage without a Kaggle submission. Use the states below when hosted scoring/submission is part of the request.

Read `information-sharing-policy.md` before sharing code, models, data, or reports outside the team. Use `kaggle-pipeline.md` for the actual scoring contract, `kaggle-offload.md` for remote execution, and `code-competition-debugging.md` for failures.

## Completion States

- `scored`: a valid submission was processed and its available hosted status/result was retrieved. Record observation time and artifact/version identity. For agents, hosted validation and available rated-episode evidence do not imply a settled rank.
- `no_qualified_upgrade`: an existing scored incumbent remains because no candidate passed the defined improvement gate within the budget. Record rejected candidates and reopening evidence; this does not replace a requested first submission.
- `blocked`: a concrete external condition prevents further progress, such as authentication/access, unaccepted rules, unavailable required platform input, closed competition, or confirmed quota/service failure. Record the exact evidence and next feasible action; an unknown quota or a vague runtime error is not proof of this state.
- `stopped`: an agreed budget or stopping rule was reached before scoring succeeded. Say explicitly that the scoring objective remains unmet; preserve artifacts, attempted fixes, remaining work, and conditions for resuming. Do not relabel this as `scored` or an external blocker.

If work is limited to an earlier phase, report that phase's result and remaining hosted work instead. Do not expand scope or repeatedly ask for approval already supplied. Additional scope/cost outside the existing authorization needs the user's decision.

## Before Spending A Submission

- Check the competition's actual submission limit and remaining attempts; record the observation time. Preserve a validated fallback and allow processing/recovery time before the deadline.
- Select using the relevant validation: leakage-safe held-out predictive evidence or representative interactive matches. When a scored incumbent exists, account for active-slot eviction and require a qualified replacement.
- Verify the exact submission contract, artifact hash/version, dependencies, and resource limits. Archives include the required files/layout; files match the required IDs, schema and values; servers/adapter submissions follow their own contracts.
- Test parsing/serialization separately from the metric. A change to precision, rounding, row types, or optional fields needs the real writer/reader round trip (`ensembling-and-reproducibility.md`). Local metric agreement does not prove hosted compatibility.
- For a contract-compliant format or CV/LB comparison, define the hypothesis and interpretation before a small informative attempt. Use the debugging loop for failures; do not use failures to probe hidden data.

A `400`, `403`, or generic `ERROR` can have several causes. Inspect the response, access, format, and official starter rather than inferring rules acceptance or submission mode from the status alone.

## Submit And Collect The Result

Use current command help and the competition's documented path. Typical prediction-file and notebook-linked patterns are:

```bash
kaggle competitions submit SLUG -f SUBMISSION_FILE -m RUN_ID
# When kernel/version scoring or attribution is required/supported:
kaggle competitions submit SLUG -f SUBMISSION_FILE -k USER/KERNEL -v VERSION -m RUN_ID
kaggle competitions submissions SLUG -v -q
```

For code/model/server submissions, verify the scoring entry point and every required artifact first (`kaggle-pipeline.md`). A single kernel need not have producers. If staged, all required producer outputs must be complete and mounted. Determine the actual kernel version from available platform metadata; record uncertainty or missing access rather than inventing one.

Retrieve processed status/result within the authorized budget. Save `submission_receipt.json` (or the existing equivalent) in the orchestration workspace, with command/request, timestamp, file hash or kernel/version, submission ID when available, observed status, score/result, and error evidence. Update a pipeline record only when present. A push receipt or completed visible run alone does not establish hosted scoring.

For interactive agents, use the exact policy package and loader, then collect hosted validation and available rated episodes/status with opponent context. Fixed replay counterfactuals and self-matches provide narrower evidence than interactive strength. Report what is available without waiting indefinitely for rank convergence.

## Interpret And Select

- Public/CV disagreement calls for a split, metric, format, or distribution audit. For code changes, first confirm the intended version/artifact actually ran.
- Early game ratings can vary in either direction. Use episode count, opponent coverage, uncertainty where available, and observation time; a fixed age in hours does not establish convergence.
- Read the final-selection rule: eligibility, number of picks, defaults, and result assignment. Another pick needs credible complementary performance or a specific risk hedge; a different name alone is not diversity.
- After final held-out/private or tournament results, separate candidate quality from selection regret. Compare chosen and eligible alternatives under that rule, without treating hindsight as information available during selection.

## Final Report

Include the requested phase and completion state, competition/run ID, exact artifact/kernel version, mode-appropriate validation and its limits, hosted status/result if attempted, consumed budget, relevant failure/retry evidence, and any unresolved objective. For retained incumbents, identify the existing submission and why no upgrade qualified. For staged work, include artifact source/version lineage. State the next action or reopening condition where work remains.
