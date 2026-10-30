# Submission Endgame

Use this reference before stopping a competition execution task. Normally the workflow expects a scored Kaggle submission, not just a local model, remote run, or notebook architecture. For bounded improvements to a scored incumbent, retain it if no candidate qualifies. Analysis, post-mortem, and skill-maintenance requests do not require a new submission.

Before publishing final code, reports, models, or artifacts outside the team, read `information-sharing-policy.md`.

## End Condition

Continue working until one of these states is reached:

- `scored`: Kaggle accepted and processed a submission, and the submission status plus public score or visible scoring result has been retrieved.
- `no_qualified_upgrade`: for a bounded improvement task with an existing scored incumbent, the defined budget or stop rule was reached and no candidate passed the validation gate. Keep the incumbent and report the evidence and reopening condition; this is not an external scoring blocker or permission to skip a requested first submission.
- `blocked`: an external condition prevents scoring, such as missing Kaggle credentials, competition rules not accepted, submission quota exhausted, competition closed, scoring disabled, required manual UI-only action, missing kernel version, or Kaggle service failure.

If a code-competition submission fails with a vague scoring/runtime error, do not mark it blocked immediately. Read `code-competition-debugging.md`, patch the final kernel defensively, rerun/resubmit, and only mark blocked after retry attempts are exhausted by an external condition.

If blocked, record the exact error text, command attempted, competition slug, next command/action, retry count, and whether any local/remote artifacts are ready.

## Interactive Agent Flow

Use `simulation-game-agents.md` for local episode and opponent evaluation. A Python/archive agent is not necessarily a prediction-file or hidden-test notebook competition.

Before replacement, verify the exact action-producing chain, runtime dependencies, archive entrypoint, cold start and per-turn budget, plus the current active-slot/eviction rule. Use representative interactive matchups and a frozen confirmation rule; fixed recorded opponents and validation self-play establish narrower evidence.

Submit the qualified agent using the competition's actual contract. Retrieve hosted validation status and available rated episode outcomes or rating/status receipts with their observation time, opponent coverage, and exact artifact identity. A processed validation game is not a settled competitive rating. Report available evidence without waiting indefinitely for rank convergence. If no candidate qualifies within a defined improvement budget, retain the scored incumbent and record `no_qualified_upgrade`.

## Submission Budget

- Each competition sets its own daily submission limit; read it from the competition page and count today's attempts with `kaggle competitions submissions SLUG`.
- Spend scoring attempts on validated improvements. Use visible kernel runs, OOF metrics, and local checks to catch format or runtime problems first.
- When only a few attempts remain, prefer the OOF-best candidate over a public-LB-best candidate whose CV does not support it.

## Pre-Submission Checks

- Rules accepted and credentials work.
- The file exists, the row count matches `sample_submission.csv`, IDs and order are correct, required columns exist, labels/probabilities are in the expected format, and values are clipped or rounded only when the metric or competition requires it.
- For zipped submissions, the archive contains only the required file: `zip submission.zip submission.csv`.
- Local CV and OOF diagnostics agree with the candidate being sent.

## Classic Submission Flow

```bash
kaggle competitions submit COMPETITION_SLUG -f SUBMISSION_FILE -m "RUN_ID_OR_MESSAGE"
kaggle competitions submissions COMPETITION_SLUG -v -q
```

If a notebook produced the file, add `-k USER/KERNEL -v VERSION` so the submission is linked to the code.

After submitting:

- Poll `kaggle competitions submissions` until the submission is processed or a failure is visible.
- Save the latest submission record to `submission_receipt.json`.
- Add the public score/status to the experiment log and the final response.

A `400 Bad Request` on submit usually means a format problem: wrong columns or header, IDs that do not match the test set, a zip that was expected (or not), or a competition that requires a kernel or model submission rather than a file. Check `sample_submission.csv` and a top public notebook's output format before retrying.

## Code Competition Flow

Follow `kaggle-pipeline.md` for the full producer/consumer procedure. The endgame steps are:

1. All producers complete; durable outputs are versioned as private datasets and attached to the consumer.
2. Push the final consumer, poll until complete, download outputs, verify `submission.csv` (or a clean local-gateway run for `kaggle_evaluation` competitions), logs, and manifests.
3. Submit the consumer kernel version: `kaggle competitions submit SLUG -f submission.csv -k USER/KERNEL -v VERSION -m MSG`.
4. Retrieve the status/score; if scoring fails, run the debugging loop on the consumer and resubmit.

Determine `VERSION` from the kernel status/output metadata or the Kaggle UI. If it cannot be determined programmatically, record that as a blocker with the final kernel ref and ready artifact.

## Score Interpretation

- For simulation, game-agent, and episode-averaged competitions, early scores are inflated and noisy: the rating converges over hours as more episodes run. Submit early to start evaluation, and do not make decisions on a score younger than a few hours; a drop from the initial peak is normal.
- For ordinary competitions, a public score that disagrees with CV is a validation question (split choice, metric implementation, train/test drift) before it is a modeling result.
- A score drop after a code change: confirm the new kernel version actually ran and completed, compare the two submission files, then look at the model.

## Final Response Requirements

A completed run summary must include:

- Competition slug.
- Run ID and final artifact/kernel reference.
- Mode-appropriate local validation: CV/OOF mean and fold std for prediction tasks, or complete-episode W/D/L, paired uncertainty, opponent panel/version/time and confirmation evidence for interactive agents.
- Kaggle submission status and public score if processed.
- For an unchanged incumbent, its existing submission identity, why no replacement qualified, and the evidence needed to reopen the route.
- Error/debug loop summary if a scoring failure occurred.
- Submitted file path or kernel/version.
- Producer dataset/kernel refs for staged architectures.
- Submission attempts used today versus the competition limit.
- Any remaining private-LB or validation-risk caveats.
