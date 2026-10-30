# Simulation And Game Agents

Use for sequential policies competing through complete games. Replace predictive-ML row folds and OOF artifacts with episode splits, policy provenance, interactive match matrices, and hosted submission evidence. Analysis and retrospective requests do not start training or submissions.

## Frame The Decision And The Score

Read the executable rules and hosted agent contract: observation visibility, action order, simultaneous moves, invalid-action behavior, inventory/resources, stochastic events, episode length, scoring, runtime, and active submission slots. Separate simulator version from submission runtime version.

Choose the scored outcome as the promotion metric. In a win/loss tournament, money, resource production, imitation accuracy, shaped reward, and transient rating are diagnostics. More money against one plan does not establish more wins against another plan. Report the opponent distribution and evaluation date with every strength claim.

Before a long experiment, identify the missing capability. Can the current policy generate the required production, movement, timing, and resource decisions? A residual or router over fixed scripts cannot discover a sequence absent from its candidates. A daily state predictor without an executable controller is a modeling probe, not a deployed game policy. Preserve the distinction between macro BC, primitive-action BC, offline return weighting, and on-policy RL.

## Treat Replays As Training Data

Build a ledger with episode ID, timestamp, both submission IDs and teacher lineage, seat, seed if public, simulator version, observed actions/outcomes, and collection cutoff. Deduplicate episodes and near-duplicate policy lineages. When a replay contributes both seats, keep them in the same split. Keep derived snippets and augmented examples with their source episode. Reserve games or time blocks for confirmation; use teacher-family holdouts when testing transfer to unseen strategies. Record intentional teacher overlap rather than claiming unseen-teacher generalization.

Select demonstrations by current strength, recency, coverage, and available volume. Do not infer strength from dataset frequency. Compare broad multi-teacher pretraining, teacher/style conditioning, recency weighting, and specialist fine-tuning when affordable. Keep lineage diversity separate from number of handles. Strong-teacher-only data can be efficient; broader data may preserve behaviors for later exploration. Neither is universally superior.

Reconstruct the actual decision semantics before training:

- Replays may record requested actions that were clipped, partially fulfilled, superseded, or became no-ops. Retain requested action, executed effect, and any relabeled target with provenance. Evaluate whether raw-action imitation or execution-consistent targets better support the chosen decoder; do not silently replace every invalid action with PASS.
- Audit same-turn resources in engine order. Earlier purchases, deposits, unit actions, or market orders can change what is legal later in that turn. A start-of-turn mask can remove teacher moves that the engine would execute.
- Round-trip observation → encoded label → decoded action → engine. Compare transitions or relevant state traces and terminal results on representative replays, including conflicts, capacity limits, and partial orders. Mark unsupported traces and exclusion reasons.
- Predict from information available at action time. Opponent hidden inventory may be estimated from public history with uncertainty; training-only privileged critic information must not enter the submitted policy.

Split by games before sampling individual turns. Held-out cross-entropy, action accuracy, and token loss help diagnose training; choose strength with complete closed-loop games. A lower loss can still yield worse decisions through compounding errors, teacher mixture, or quantity and timing mistakes.

## Choose The Smallest Viable Learning Route

Once an executable baseline exists, run a bounded pilot that can change the architecture decision. Specify the missing behavior, dataset cutoff, executable output, evaluation panel, compute budget, and success/rejection evidence before expensive expansion. Keep a runnable baseline while testing a structurally different candidate.

Reasonable branches include:

| Branch | Useful when | Main uncertainty to resolve |
| --- | --- | --- |
| Script/search | Good rules and tractable planning expose a strong baseline | Future value, coordinated execution, opponent response, runtime |
| Full-action BC | Strong public demonstrations exist | Can a state-conditioned policy retain teacher behavior in its own visited states? |
| Macro/pointer policy + executor | The executor gives useful structure and reduces action space | Can the representation express the missing behavior, and does the executor cap it? |
| BC + narrow rules | A competent policy wastes inventory or takes provably ineffective actions | Does the rule improve whole-agent games without changing valuable plans? |
| BC → PPO / league | BC is competent, simulator is fast, and improvement budget is credible | Gradient correctness, exploration, critic quality, opponent coverage |
| Targeted demonstrations / interventions | A competent policy consistently lacks a valuable sequence | Can the behavior be introduced without forgetting existing strength? |

Do not require PPO to reach a high rank; do not dismiss it because one macro-action or poorly initialized run failed. Compare branches under feasible budgets. Separate final recipe training cost, inherited checkpoint compute, evaluation cost, wall time, hardware, and total research cost. A post-hoc cheap recipe is not the historical cost of the submitted weights.

Action representation is an experimental choice. Compare primitive actions with pointers/macros, parallel heads with autoregressive decisions, quantities and STOP encoding, history features, and constraint placement when they address a measured bottleneck. Account for all units and ordered market actions, resource competition, and inference latency. A mask, executor, or larger model is neither universally required nor universally harmful.

## Make Evaluation Interactive

Use three distinct evidence levels:

1. **Faithful replay**: identical recorded actions reproduce observed state/results. This checks implementation.
2. **Recorded-opponent counterfactual**: change our policy while replaying the opponent's fixed actions. This is a cheap diagnostic. The opponent cannot react; changed resource availability, partial orders, or RNG consumption can make it collapse. Even a faithful original replay does not make this a realistic interactive opponent.
3. **Closed-loop match**: both policies act on their evolving observations. Use this for promotion, with current representative opponents and separate confirmation where feasible.

An imitation of a rival remains a proxy. Report its fidelity and gap to available real episodes; do not claim a win over the author's original agent. A panel saturated near 100% cannot discriminate frontier improvements. Refresh with different strategy families and current strong interactive policies; if unavailable, report limited coverage and use hosted games to assess the gap.

Pair candidate and control by seed and seat, and cluster uncertainty by the independent world/seed block. Check RNG consumption: the same seed need not yield the same later shops/events after policies diverge. Describe ordinary paired-seed games as closed-loop outcomes, not identical-world causal interventions. Explicitly forced event sequences can test stress cases, but may change the official distribution and RNG coupling.

Freeze the candidate, primary outcome, opponent weights, stopping rule, and confirmation set before final selection. Choose sample size for the effect and budget rather than copying another team's numeric gate. Report wins/draws/losses, paired delta and uncertainty, L→W / W→L flips, and per-family regressions. Avoid promotion from partial jobs completed in opponent order, repeated peeking, or money-only gains.

Evaluate the entire deployed action path, including decoder, masks, executor, rules, persistence/history, and final wrapper. Run a representative match through the actual loader. Require a sample episode or board/action trace alongside aggregate claims of new behavior; verify that the claimed extra land, crop mix, transport, or adaptation actually occurs.

## If Using RL

Start from a competent policy when sparse, long-horizon rewards make random exploration impractical. Validate on small settings with known good strategies before scaling. Measure simulator-plus-feature-plus-inference-plus-update throughput, not engine speed alone. Differential-check fast engines against the official implementation, including RNG, insertion/order semantics, hidden state, and terminal accounting.

Inspect critic calibration/explained variance against the intended return, early-step credit assignment, advantage scale, entropy, KL, and clipping fraction. Critic pretraining, separate or shared critics, symmetric/antisymmetric value structure, and long-horizon settings are hypotheses to compare. Do not copy a team's critic or discount choice as a universal prescription.

For multi-factor actions, verify whether probability ratios and clipping operate per decision or on the joint action; their optimization behavior differs sharply. Align stored actions, masks, quantities, STOP handling, old/new log probabilities, and actual behavior. After masking, use normalized probabilities. If rules overwrite sampled actions, derive the induced behavior distribution or explicitly exclude forced factors from the policy objective and test the resulting estimator. Recomputing the unmodified policy's probability on a forced action alone does not prove on-policy correctness.

Keep the actor, evaluator, and submitted action contract aligned. Evaluate sampling versus greedy decoding. If a rule fixes a meaningful weakness, compare inference-only correction with training-aware handling; interventions can also destroy previously learned behavior. Use controls from the same parent and budget, plus regression opponents. A temporary BC loss after introducing missing behaviors may be acceptable for an RL branch if the intended behavior appears and a bounded continuation recovers strength; this is a separate gate from shipping that BC checkpoint directly.

Use a league appropriate to the training stage: incumbent/self-play, historical checkpoints, distinct styles and exploitable opponents. Monitor family coverage, cycles, forgetting, and whether excessively strong opponents remove useful signal. Beating the incumbent alone does not show league improvement. Changing the league changes the yardstick; retain fixed reference panels for diagnosis.

## Rules And Endgame

Profile rule opportunities before implementing them: frequency, resource value, reachability, displaced work, and effect on the scored outcome. Prefer narrow measured fixes over replacing every PASS or seemingly idle move. Idle actions may protect timing or future coordination. Re-measure rules when model weights change; a terminal planner can help one checkpoint and harm a stronger one.

Separate isolated component gains from the final combined chain. Test interactions and preserve failed variants. A final pair of active submissions should have a documented purpose: best validated policy, robust fallback, or complementary matchup coverage. Different names or one changed constant do not establish diversity. Check eviction semantics, quotas, cold-start time, per-step budget, overage, memory, dependencies, and archive size before release.

## Retrospective And Completion

For a post-mortem, read primary writeups and inspect local code, configs, receipts, failures, and actual submitted artifacts. Distinguish author claims, reproduced evidence, local measurements, and analyst inference. Verify identity before attributing another team's writeup or final rank to the user. Record unavailable evidence and source revisions; do not infer a method from the title alone.

Organize lessons by decision point: initial mode/metric, replay contract, representation, training pilot, opponent panel, plateau, and release. For each proposed change, state trigger, concrete artifact, pass/stop evidence, and what would justify reopening a failed route. Preserve contradictory successful methods as conditional branches. Do not retrofit post-deadline knowledge into claims that an earlier decision was certain to fail.

An execution request ends with the exact submission's hosted validation and available rated outcome/status receipts, or a concrete external blocker. A bounded improvement task may also end with the existing scored incumbent retained when the defined budget/stop rule is reached and no candidate qualifies; report rejection evidence and reopening conditions. Report observation time and sampling limits; do not wait indefinitely for a settled rank. An analysis request ends with the requested evidence-based analysis and skill/document updates, without launching competition work.
