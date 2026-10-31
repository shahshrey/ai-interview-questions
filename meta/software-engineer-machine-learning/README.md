# Meta: Software Engineer, Machine Learning

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Meta publishes only the broad stages. The round counts and names below come from public prep pages and anecdotal candidate reports.

1. A recruiter call of about 30 minutes covers your background, the scale of ML systems you have owned, level calibration, and logistics.
2. The technical screen runs about 45 minutes and is usually a coding round on data structures and algorithms.
3. Meta's ML prep page describes a full loop of up to six 45-minute conversations. For a role at this scope, expect something close to this:
   - One or two coding rounds. In October 2025 Meta began piloting a 60-minute AI-assisted coding round that replaces one coding round. Whether it applies to this role is unknown.
   - One or two ML system design rounds on large ranking or recommendation systems, training, and serving.
   - Possibly a general or infrastructure design round framed around ML platforms.
   - A behavioral round on org-level scope, influence without authority, conflict, and mentoring. Guides say this round weighs heavily on leveling at senior levels.
   - Possibly a deep dive on a large ML system you led.
4. The hiring committee review and team matching come last.

## Questions and answers

### ML system design at scale

#### Q1. Design the training and serving architecture for a recommendation model with trillions of sparse embedding parameters and billions of dense parameters.

Treat the two halves of the model separately, because they behave differently. The sparse embedding tables are huge and memory bound. Shard them across GPUs with model parallelism and exchange lookups with all-to-all. The dense layers are smaller and compute heavy, so shard them with FSDP or replicate them with data parallelism.

Then map each collective to the fastest link that can carry it. Heavy traffic stays inside a node on NVLink, and lighter replica sync crosses the scale-out network. Meta has written publicly about this kind of topology-aware, multi-dimension parallelism for its ads foundation model.

Feed training from a streaming pipeline with jagged, variable-length user sequences, so compute goes to real data instead of padding. Checkpoint sparse and dense state on separate schedules, since the embedding state is far larger.

Serving splits into an embedding lookup tier and a dense model tier. Cache hot IDs, quantize embeddings to cut memory, and give each stage its own latency budget.

In the interview, put rough sizes on everything, such as bytes per row and table count. Say which bottleneck dominates at each stage, and walk through how the design changes when sequence lengths grow tenfold.

#### Q2. Design an online inference stack for a high-traffic ranking model with a tight P99 latency budget.

Break the request path into stages and give each one an explicit budget. Candidate retrieval comes first, then a cheap early-stage ranker that cuts thousands of items to hundreds, then a heavy late-stage ranker. Spend the most latency where model quality matters most.

Precompute the slow-changing user and item features and keep them in a low-latency key-value store, so the only thing you compute at request time is the request context. Batch candidates into one forward pass and reuse the user-side computation across them. On the accelerators, use reduced precision and kernel fusion, and set dynamic batching windows to balance throughput against tail latency.

Plan for failure from the start. Put timeouts on the stages with a fallback ranker behind them, degrade gracefully when the feature store is slow, and pin model versions so features and weights always match.

For rollout, run the new model in shadow mode first. Then canary it, with automatic rollback when guardrail metrics regress past a set threshold.

Interviewers will want a number for the budget at each stage and your reasoning on which features can sit in cache and which must stay fresh. Also explain how you would catch a slow regression that the canary misses.

#### Q3. How would you prevent train-serve skew in a feature platform that many teams use?

Skew usually has one of two causes. Either two code paths compute the "same" feature differently, or training sees data that would not exist yet at serving time, which is time leakage. The best fix is structural. Write each feature definition once and compile it into both the batch and online paths, with event-time semantics, watermarks, and clear rules for late or missing events.

Where you can, train on features logged at serving time instead of recomputing them offline. That removes a whole class of skew. You pay for it in storage and in slower feature iteration.

Then add detection. Sample requests and compare the logged online values with the offline values on a daily parity dashboard, broken down by slices such as region and device. Version features so a running model keeps the definition it was trained with, and add contract tests that fail the build when a feature's distribution moves too far.

Be ready to discuss the tradeoff between logged features and backfills, how you would handle identity changes and late data, and how a platform team gets dozens of teams to adopt the shared definitions.

### Training efficiency and reliability

#### Q4. End-to-end training MFU is low. How do you tell whether the problem is compute or scaling, and what do you fix first?

Split the problem in two. Meta's engineering blog describes end-to-end MFU as local, single-GPU efficiency multiplied by a scaling ratio. Measure local MFU by profiling layers on one GPU without communication. If local MFU is low, the problem is kernels and precision. If local MFU looks fine and end-to-end MFU is still low, the problem is distributed overhead.

On the compute side, look for memory-bound ops such as small matmuls and normalizations, padding waste on variable-length inputs, and unfused elementwise chains. The fixes are fused or custom kernels, jagged attention, and lower precision such as FP8, with checks on numerical quality.

On the scaling side, look for exposed communication, load imbalance across embedding shards, data loader stalls, and stragglers. Overlap communication with compute, reshard tables by cost, move heavy collectives to faster links, and prefetch input.

Fix the largest item in the profile first. Show the interviewer a timeline trace, put concrete percentages on each source of loss, and confirm that every speedup leaves model quality unchanged.

#### Q5. A large training job slowly diverges after weeks, with no NaNs. What could cause it, and how do you detect it?

I would sort the causes into four groups. On the optimization side, look at the learning rate schedule, optimizer state issues, and loss spikes that never fully recover. On the data side, look for distribution shift in the stream, a broken upstream join, or label delays. Numerics can drift through low-precision overflow or rounding bias. Then there is hardware. Meta has written that silent data corruption can produce wrong gradients that stay within numeric bounds, so training looks healthy while it drifts.

Detection needs several layers. Track per-layer gradient norms, loss variance, and the update-to-weight ratio, and alert on changes in trend as well as on spikes. Monitor drift in the main features and in label rates. For hardware, run periodic fleet health tests and compare results across replicas. In data-parallel training, a replica that disagrees with the others points to a bad device.

To diagnose, replay a short window from a checkpoint on known-good nodes. If the run diverges again, the cause is data or optimization. If it stays stable, suspect hardware. Bisect checkpoints to find when the drift began.

Keep detection and root cause as separate steps in your answer, give the cost of each monitor, and propose guardrails such as automatic rollback to a healthy checkpoint.

### Debugging, metrics, and evaluation

#### Q6. Offline AUC improved, but the online metric dropped after launch. Walk through the investigation.

Limit the damage first. If the drop is significant, roll back or shrink the exposure, and then investigate.

Next, confirm the experiment is sound. Check that randomization is correct, the sample ratios match, the run was long enough, and novelty effects are out of the picture. Then look for train-serve skew by comparing logged online features with offline features for the same requests. Check calibration too. AUC measures ranking, so a model can rank better while being badly calibrated, and that breaks downstream auctions or blending.

After that, question the offline metric. Look for label leakage, an evaluation set that differs from live traffic, or an objective that differs from the online goal, such as clicks against long-term value. Break the results down by segment, such as new users, locales, and surfaces, because a gain for most users can hide a large loss for a few. Finally, consider feedback loops. The old model shaped the training data, so the new model may be learning the old model's biases.

Order the checks from cheapest to most expensive, and finish with a process fix, such as a standing online-offline correlation check.

#### Q7. Design an automated evaluation pipeline for model launches across several product areas.

The goal is fast, consistent launch decisions from one shared process that every team uses. Set up tiered gates that get more expensive as a model advances. Start with config and data validation, then offline metrics on a frozen benchmark, then slice analysis for fairness and important segments. After that come shadow traffic to check latency and prediction drift, a small canary, and finally an A/B test with guardrails.

Share and version the metric definitions, so "engagement" means the same thing everywhere. Store every evaluation with the exact model, data, and code versions, so anyone can reproduce any result. Automate the reporting as well. A launch review should be a generated document with deltas, confidence intervals, and flagged regressions.

LLM-based tools can help summarize results and spot anomalies. People still own the launch decision, and every generated summary should link to the raw numbers.

The tradeoffs are speed against rigor, and a central standard against team flexibility. Let teams add custom gates on top while the shared gates keep their full strength, and track how often the offline gates correctly predict online results.

#### Q8. How do you define metrics for a multi-year ML project so that model signals connect to business outcomes?

Build a hierarchy of metrics. At the top sits a slow-moving business outcome, such as long-term retention or advertiser value. Below it are online proxies you can measure in an experiment, such as sessions or conversions. Below those are offline model metrics, such as normalized entropy, calibration, and recall at K. Each level has to be shown, with evidence, to predict the level above it.

Validate the proxies with long-running holdouts and historical launches. Ask whether the experiments that moved the proxy also moved the outcome months later. If they did not, the proxy is the wrong thing to optimize. Add guardrail metrics for integrity, user wellbeing, latency, and cost, so a team can only claim a win on the main metric when those measures hold steady too.

For multi-year projects, write the metric plan down, publish it, assign owners, and review it every year. Products change and proxies decay, so the plan has to keep up.

Bring up Goodhart's law, explain how you would handle metrics that conflict between teams, and have an example ready of a proxy that turned out to be misleading and how it was replaced.

### Platforms and AI-native engineering

#### Q9. Design an ML framework or modeling standard for several engineering orgs. How do you get teams to adopt it without mandating it?

Start from evidence. Interview teams, list the duplicated work, and measure what it costs, such as weeks to launch a model or incidents caused by inconsistent pipelines. Then design a small, stable core that covers the common path, which means data loading, training loops, evaluation hooks, and export to serving. Give it clear extension points and escape hatches for teams with unusual needs, because power users who feel blocked will fork the framework.

Make the right path the easy path with good defaults, templates, fast onboarding, and tooling that enforces the standards automatically. Pilot with one anchor team whose success others will notice, then measure the gains and publish them.

To drive adoption, write migration guides, hold office hours, and fund the migration work for each team. Retire the old paths gradually, with dates announced well ahead.

The tradeoffs are flexibility against consistency, and short-term migration cost against long-term speed. Come with adoption metrics, a plan for a strong team that refuses to migrate, and an answer for how the framework's ownership is staffed over the long term.

#### Q10. How would you use LLM-based tooling or agents to speed up the model development cycle?

Map the cycle and find where engineers lose the most time. The usual places are writing experiment configs, waiting on and triaging failed jobs, reading dashboards, comparing runs, and writing launch reviews. These tasks are repetitive and their outputs can be checked, which makes them good targets for automation.

There are several concrete uses. An agent can turn a written hypothesis into an experiment config and a sweep plan. A triage assistant can read job logs and stack traces and suggest the likely cause. Automated comparisons can summarize metric deltas between runs and flag unusual slices, and regression bisection can search across commits and data versions.

Put guardrails around all of it. Agents propose and run work inside a sandbox with limited permissions, every claim links to evidence, and a person approves anything that touches production or a launch decision. Measure the impact through cycle time, the number of experiments per engineer, and the error rate of agent suggestions.

Name the failure modes, such as confident wrong diagnoses and runaway cost, and explain how you would evaluate the tool itself. Then describe how you would widen its scope step by step as it earns trust.

### Responsible AI and integrity

#### Q11. How would you build privacy and integrity safeguards into an ML pipeline at scale?

Build the safeguards into the platform so they apply to every team by default. On the data side, track lineage and purpose for every dataset, enforce access controls and retention limits automatically, and detect personal data at ingestion. A training job should only be able to read data whose approved purpose matches the model's purpose.

On the model side, run standard evaluations by slice to find quality and bias gaps across groups, regions, and languages. Where models generate text, test for memorization, and red-team high-risk surfaces before launch. For integrity problems where attackers adapt, plan for frequent retraining, delayed labels, and drift monitoring.

On the process side, tie launch gates to required reviews based on risk tier. Low-risk changes move fast, and high-risk ones get policy and legal review. Bring in the compliance and policy teams early and turn their requirements into automated checks.

The tradeoff is speed against safety, and tiering by risk keeps most changes moving quickly. Be ready to explain how you would handle a deletion request for data already used in training, and how you would measure whether the safeguards catch real problems.

### Coding

#### Q12. Find the top K most frequent items in a large stream, then explain how you would scale it.

Clarify first. Ask whether the data is bounded or streaming, whether K changes, and whether you need an exact answer. For a bounded dataset, count with a hash map, then keep a min-heap of size K. That runs in O(n log K) time and O(n) space. Bucket sort by frequency gets you to O(n) when counts are bounded by n. Handle the edge cases, which are K larger than the number of distinct items, ties, and empty input, and write a test for each.

For a stream at scale, exact counts may not fit in memory. A count-min sketch estimates frequencies in fixed memory and pairs with a small heap of candidates. The Space-Saving algorithm is another option, and it gives bounded error. Explain the error guarantees and how memory trades against accuracy.

For distributed data, shard by item key so each worker counts its share, then merge the local top-K lists. Merging approximate top-K results needs some extra candidates from each shard to stay accurate.

Get correct code working before you optimize, state the complexity clearly, and connect the scaled version to real uses such as trending content or hot embedding IDs.

### Leadership and behavioral

#### Q13. Tell me about the most complex cross-system ML failure you diagnosed.

Pick a story that crossed system boundaries, such as data, training, serving, and hardware, so the interviewer sees your range. Tell it in order. Start with the symptom and its business impact, then explain why it was hard, for example because it was intermittent, had no clear owner, or gave misleading signals. After that come the hypotheses you formed and how you ruled each one out, the root cause, and the fix.

Spend time on your method. Explain how you narrowed the search, whether by bisecting changes, building a tool to compare logged features, or replaying traffic. Name the people you brought in and how you coordinated across teams that each believed the problem belonged to someone else.

Then cover the systemic fix. At this level the interviewer wants to see that you prevented the whole class of failure with a new monitor, a contract test, a platform change, or a better on-call playbook. Give numbers for time to detect before and after, the revenue or engagement recovered, and the incidents avoided since.

Expect follow-ups on what you would do differently and on your wrong turns. Own the mistakes plainly and say what you learned from each.

#### Q14. Describe a time you set multi-year technical direction across teams that disagreed.

Pick a decision with real stakes, such as a platform migration, an architecture bet, or a change to modeling standards, where reasonable teams had competing interests. Lay out each side's position fairly, because interviewers pay attention to how you describe the people who disagreed with you.

Then explain how you built the case. Talk about the data you gathered, the prototypes or benchmarks you ran, and the written proposal that set out alternatives, costs, and risks. Show where you listened to dissent and changed the plan because the critics were right. Explain how the decision was made and who approved it, and how you kept teams aligned over time with milestones, check-ins, and clear owners.

Give the outcome in numbers and say plainly which parts went off plan. For multi-year work, explain how you balanced near-term delivery against long-term architecture, for example by shipping intermediate wins that funded the larger effort.

The story should show influence without authority, strong written communication that works for executives too, and the judgment to know when to commit and when to change direction.

#### Q15. How do you mentor senior engineers on debugging complex model behavior?

Senior engineers already know the basics. What they need is judgment and a repeatable method. Teach a debugging playbook. Confirm the symptom is real, find the smallest reproducible case, separate data, model, and serving causes, and change one variable at a time. Then practice it together on real incidents.

Pair on live investigations and let the other engineer drive. Your part is to ask questions that expose gaps, such as "how would you know if this feature is skewed?" Review postmortems together and focus on how the problem could have been caught earlier.

Scale yourself through written guides, shared debugging tools, and recorded walkthroughs of hard cases. Give people stretch problems slightly beyond their current scope, with support in place.

Measure success by the growth in scope people take on, by escalations to you dropping over time, and by mentees who go on to mentor others.

Have a concrete example ready of someone you helped grow, explain how you adjusted your approach for that person, and show that you share credit openly.

## Prep checklist

- [ ] Read Meta's public ML full loop prep page and its downloadable guide.
- [ ] Prepare two or three org-scale ML system stories with real numbers for latency, efficiency, business impact, and the number of teams involved.
- [ ] Practice end-to-end ML system design for large ranking and recommendation systems, covering retrieval, ranking, features, training, serving, and monitoring.
- [ ] Review sparse plus dense distributed training, including embedding sharding, FSDP, all-to-all, and how to reason about MFU.
- [ ] Read Meta's engineering posts on training its ads foundation model and on AI hardware reliability.
- [ ] Rehearse a structured investigation of gaps between offline and online metrics and of train-serve skew.
- [ ] Do timed coding practice, with at least one session using an AI coding assistant in the editor.
- [ ] Write a one-page design memo for a past system to practice written communication for executives.
- [ ] Prepare an example of using AI tooling to measurably speed up an engineering workflow.

## Sources

- https://www.metacareers.com/ML-prep-onsite : Meta's ML full loop prep page, which describes up to six 45-minute conversations.
- https://www.metacareers.com/swe-prep-onsite : Meta's software engineering full loop prep page.
- https://interviewing.io/blog/how-to-use-ai-in-meta-s-ai-assisted-coding-interview-with-real-prompts-and-examples : A third-party write-up of Meta's AI-assisted coding round pilot.
- https://www.datainterview.com/blog/meta-machine-learning-engineer-interview : A third-party guide to the stages and themes of Meta's MLE loop.
- https://www.teamblind.com/post/meta-e7-interview-experience-iw3gnkuz : An anecdotal candidate discussion of a senior IC loop.
- https://crackingwalnuts.com/companies/meta/e6 : A third-party guide to the staff-level loop, based on candidate reports.
- https://engineering.fb.com/2026/08/03/ml-applications/training-gem-at-llm-scale-meta-ads-recommendation-foundation-model/ : Meta's engineering post on scaling training for its ads foundation model.
- https://engineering.fb.com/2025/07/22/data-infrastructure/how-meta-keeps-its-ai-hardware-reliable : Meta's engineering post on silent data corruption and AI hardware reliability.
- https://github.com/meta-pytorch/torchrec : TorchRec, Meta's PyTorch library for large-scale recommender systems.
- https://github.com/meta-recsys/generative-recommenders : Code for Meta's generative recommenders (HSTU).
