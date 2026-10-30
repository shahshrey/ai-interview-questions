---
company: Meta
title: Software Engineer, Machine Learning
role_type: ml-engineering
level: null
location: Menlo Park, CA
salary: "$347,000/year to $403,000/year + bonus + equity + benefits"
date_added: 2026-10-30
linkedin_job_url: https://www.linkedin.com/jobs/view/software-engineer-machine-learning-at-meta-4479485548
status: open
question_count: 15
topics:
  - ml-systems
  - ranking-recommendation
  - system-design
  - agent-evaluation
  - ai-safety
  - behavioral
---

# Meta: Software Engineer, Machine Learning

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## About the role

- A very senior individual contributor role that sets the architecture for core ML systems at Meta: training pipelines, inference infrastructure, and feature platforms that run for billions of users.
- The work spans the hardest problems that cross product lines, such as model quality, training efficiency, serving latency, and data integrity, plus performance work from data loading to hardware utilization.
- You would set multi-year technical strategy across research, infrastructure, and product teams, and build frameworks and standards that several engineering orgs adopt.
- The listing puts real weight on AI-native engineering: using AI tooling and agents to shorten model development cycles and automate evaluation.
- Requirements include 12+ years building and deploying large-scale production ML systems, a track record of fixing cross-system ML failures, and the ability to write clearly for executives.
- Based in Menlo Park, CA. The listed pay is $347,000/year to $403,000/year + bonus + equity + benefits, and the posting says the range is base salary only.

Job link: https://www.linkedin.com/jobs/view/software-engineer-machine-learning-at-meta-4479485548

## Likely interview loop (estimate)

This is an estimate. Meta publishes only the broad stages; round counts and names below come from public prep pages and anecdotal candidate reports.

1. **Recruiter call (about 30 minutes):** background, the scale of ML systems you have owned, level calibration, and logistics.
2. **Technical screen (about 45 minutes):** usually a coding round on data structures and algorithms.
3. **Full loop:** Meta's ML prep page describes up to six 45-minute conversations. For a role at this scope, expect roughly:
   - 1 to 2 coding rounds. Meta began piloting a 60-minute AI-assisted coding round in October 2025 that replaces one coding round; whether it applies to this role is unknown.
   - 1 to 2 ML system design rounds focused on large ranking or recommendation systems, training, and serving.
   - Possibly a general or infrastructure design round framed around ML platforms.
   - A behavioral round that probes org-level scope, influence without authority, conflict, and mentoring. Guides say this round weighs heavily on leveling at senior levels.
   - Possibly a deep dive on a large ML system you led.
4. **Hiring committee and team matching.**

## Questions and answers

### ML system design at scale

#### Q1. Design the training and serving architecture for a recommendation model with trillions of sparse embedding parameters and billions of dense parameters.

Topics: ml-systems, ranking-recommendation, system-design

Start by separating the two parts of the model, because they behave differently. Sparse embedding tables are huge and memory bound, so shard them across GPUs with model parallelism and exchange lookups with all-to-all. Dense layers are smaller but compute heavy, so shard them with FSDP or replicate them with data parallelism. Map each collective to the fastest link that can carry it: heavy traffic stays inside a node on NVLink, lighter replica sync crosses the scale-out network. Meta has written publicly about this kind of topology-aware, multi-dimension parallelism for its ads foundation model.

For training data, use a streaming pipeline with jagged (variable-length) user sequences, so you do not waste compute on padding. Checkpoint sparse and dense state on separate schedules, because embedding state is far larger.

For serving, split it into an embedding lookup tier and a dense model tier. Cache hot IDs, quantize embeddings to cut memory, and set a latency budget for each stage.

A strong answer gives rough sizes (bytes per row, table count), names which bottleneck dominates at each stage, and explains how the design changes when sequence lengths grow tenfold.

#### Q2. Design an online inference stack for a high-traffic ranking model with a tight P99 latency budget.

Topics: ml-systems, ranking-recommendation, llm-inference-performance

Break the request path into stages with explicit budgets: candidate retrieval, a cheap early-stage ranker that cuts thousands of items to hundreds, then a heavy late-stage ranker. Spend the most latency where model quality matters most.

For features, precompute slow-changing user and item features and store them in a low-latency key-value store. Compute only request context at request time. Batch candidates into one forward pass and reuse user-side computation across candidates. Use reduced precision and kernel fusion on the accelerators, and set dynamic batching windows to balance throughput against tail latency.

Plan for failure. Use timeouts with a fallback ranker, degrade gracefully when the feature store is slow, and pin model versions so features and weights always match.

For rollout, run the new model in shadow mode, then canary it with automatic rollback when guardrail metrics regress beyond a set threshold.

A strong answer quantifies the budget per stage, explains which features belong in cache and which must stay fresh, and covers how to detect a slow regression that the canary misses.

#### Q3. How would you prevent train-serve skew in a feature platform used by many teams?

Topics: ml-systems, system-design

Skew usually comes from two code paths computing the "same" feature differently, or from time leakage, where training sees data that would not exist at serving time. The strongest fix is structural: one feature definition, written once and compiled into both the batch and online paths, with event-time semantics, watermarks, and clear rules for late or missing events.

Where you can, train on features logged at serving time instead of recomputing them offline. That removes a whole class of skew, at the cost of storage and slower feature iteration.

Add detection on top. Sample requests and compare logged online values against offline values on a daily parity dashboard, broken down by slices such as region and device. Version features so a definition change cannot silently alter a running model, and add contract tests that fail the build when a feature's distribution moves too far.

A strong answer names the tradeoff between logged features and backfills, explains how to handle identity changes and late data, and describes how a platform team gets dozens of teams to adopt the shared definitions.

### Training efficiency and reliability

#### Q4. End-to-end training MFU is low. How do you find whether the problem is compute or scaling, and what do you fix first?

Topics: ml-systems, llm-inference-performance

Split the problem in two. Meta's engineering blog describes end-to-end MFU as local, single-GPU efficiency multiplied by a scaling ratio. Measure local MFU by profiling layers on one GPU without communication. If local MFU is low, the problem is kernels and precision. If local MFU is fine but end-to-end MFU is low, the problem is distributed overhead.

For compute problems, look for memory-bound ops such as small matmuls and normalizations, padding waste on variable-length inputs, and unfused elementwise chains. Fixes include fused or custom kernels, jagged attention, and lower precision such as FP8, with checks for numerical quality.

For scaling problems, look at exposed communication, load imbalance across embedding shards, data loader stalls, and stragglers. Fixes include overlapping communication with compute, resharding tables by cost, moving heavy collectives to faster links, and prefetching input.

Fix whatever the profile shows is largest, not whatever is easiest. A strong answer uses a timeline trace, names concrete percentages, and checks that every speedup leaves model quality unchanged.

#### Q5. A large training job slowly diverges after weeks, with no NaNs. What could cause it and how do you detect it?

Topics: ml-systems, research-depth

Group the causes. Optimization: learning rate schedule, optimizer state issues, or loss spikes that never fully recover. Data: distribution shift in the stream, a broken upstream join, or label delays. Numerics: low-precision overflow or rounding bias. Hardware: Meta has written that silent data corruption can produce wrong gradients that stay within numeric bounds, so training looks healthy while it drifts.

Detection needs layers. Track per-layer gradient norms, loss variance, and the update-to-weight ratio, with alerts on trend changes, not just spikes. Monitor data drift on key features and label rates. For hardware, run periodic fleet health tests and compare results across replicas; in data-parallel training, disagreeing replicas point to a bad device.

To diagnose, replay a short window from a checkpoint on known-good nodes. If the run diverges again, the cause is data or optimization; if it does not, suspect hardware. Bisect checkpoints to find when the drift began.

A strong answer separates detection from root cause, explains the cost of each monitor, and proposes guardrails such as automatic rollback to a healthy checkpoint.

### Debugging, metrics, and evaluation

#### Q6. Offline AUC improved but the online metric dropped after launch. Walk through the investigation.

Topics: ml-systems, ranking-recommendation

First, limit the damage. If the drop is significant, roll back or shrink the exposure, then investigate.

Next, check that the experiment itself is sound: correct randomization, no sample ratio mismatch, a long enough run, and no novelty effects. Then look for train-serve skew by comparing logged online features with the offline features for the same requests. Check calibration as well: AUC measures ranking, so a model can rank better but be badly calibrated, which breaks downstream auctions or blending.

Then question the offline metric. Look for label leakage, an evaluation set that does not match live traffic, or an objective that does not match the online goal, such as clicks against long-term value. Break results down by segment, such as new users, locales, and surfaces, because a gain for most users can hide a large loss for a few. Finally, consider feedback loops: the old model shaped the training data, so the new model may be learning the old model's biases.

A strong answer orders the checks from cheapest to most expensive and ends with a process fix, such as a standing online-offline correlation check.

#### Q7. Design an automated evaluation pipeline for model launches across several product areas.

Topics: agent-evaluation, ml-systems, system-design

The goal is fast, consistent launch decisions without every team building its own process. Use tiered gates that get more expensive as a model advances: config and data validation, offline metrics on a frozen benchmark, slice analysis for fairness and key segments, shadow traffic for latency and prediction drift, a small canary, and then an A/B test with guardrails.

Make metric definitions shared and versioned, so "engagement" means the same thing everywhere. Store every evaluation with the exact model, data, and code versions, so any result can be reproduced. Automate the reporting: a launch review should be a generated document with deltas, confidence intervals, and flagged regressions, not a slide deck built by hand.

LLM-based tools can help summarize results and spot anomalies, but people still own launch decisions, and every generated summary should link to the raw numbers.

The tradeoffs are speed against rigor and a central standard against team flexibility. A strong answer lets teams add custom gates without weakening the shared ones, and tracks how often offline gates correctly predict online results.

#### Q8. How do you define metrics for a multi-year ML project so model signals connect to business outcomes?

Topics: product-strategy, ml-systems

Build a hierarchy. At the top is a business outcome that moves slowly, such as long-term retention or advertiser value. Below that are online proxies you can measure in an experiment, such as sessions or conversions. Below those are offline model metrics, such as normalized entropy, calibration, and recall at K. Each level should be shown to predict the level above it, not assumed to.

Validate the proxies with long-running holdouts and historical launches: did the experiments that moved the proxy also move the outcome months later? If not, the proxy is the wrong thing to optimize. Add guardrail metrics for integrity, user wellbeing, latency, and cost, so a team cannot win the main metric by quietly hurting something else.

For multi-year projects, publish the metric plan in writing, assign owners, and review it every year, because products change and proxies decay.

A strong answer shows awareness of Goodhart's law, explains how to handle metrics that conflict between teams, and gives an example of a proxy that turned out to be misleading and how it was replaced.

### Platforms and AI-native engineering

#### Q9. Design an ML framework or modeling standard for several engineering orgs. How do you drive adoption without mandates?

Topics: ml-systems, system-design, behavioral

Start from evidence: interview teams, list duplicated work, and measure what it costs, such as weeks to launch a model or incidents caused by inconsistent pipelines. Design a small, stable core that covers the common path, such as data loading, training loops, evaluation hooks, and export to serving, with clear extension points and escape hatches for teams with unusual needs. A framework that blocks power users will be forked.

Make the right path the easy path: good defaults, templates, fast onboarding, and tooling that quietly enforces standards. Pilot with one anchor team whose success others will notice, measure the gains, and publish them.

For adoption, write migration guides, provide office hours, and fund the migration work instead of asking teams to do it on their own time. Retire the old paths gradually, with dates announced well ahead.

The tradeoffs are flexibility against consistency and short-term migration cost against long-term speed. A strong answer includes adoption metrics, a plan for handling a strong team that refuses to migrate, and how ownership of the framework is staffed long term.

#### Q10. How would you use LLM-based tooling or agents to speed up the model development cycle?

Topics: agents, agent-evaluation, ml-systems

Map the cycle and find where engineers lose the most time: writing experiment configs, waiting on and triaging failed jobs, reading dashboards, comparing runs, and writing launch reviews. Those are good targets for automation because they are repetitive, and their outputs can be checked.

Concrete uses: an agent that turns a written hypothesis into an experiment config and a sweep plan; a triage assistant that reads job logs and stack traces and suggests the likely cause; automated comparisons that summarize metric deltas between runs and flag unusual slices; and regression bisection that searches commits and data versions.

Guardrails matter. Agents should propose and run work inside a sandbox with limited permissions, every claim should link to evidence, and people approve anything that touches production or launch decisions. Measure impact with cycle time, the number of experiments per engineer, and the error rate of agent suggestions.

A strong answer names the failure modes, such as confident wrong diagnoses and cost runaway, explains how the tool itself is evaluated, and shows how to earn trust gradually instead of automating everything at once.

### Responsible AI and integrity

#### Q11. How would you build privacy and integrity safeguards into an ML pipeline at scale?

Topics: ai-safety, ml-systems

Build the safeguards into the platform so individual teams cannot skip them. For data: track lineage and purpose for every dataset, enforce access controls and retention limits automatically, and detect personal data at ingestion. Training jobs should only be able to read data whose approved purpose matches the model's purpose.

For models: run standard evaluations by slice to find quality and bias gaps across groups, regions, and languages. Test for memorization where models generate text, and red-team high-risk surfaces before launch. For integrity problems where attackers adapt, plan for frequent retraining, delayed labels, and drift monitoring.

For process: tie launch gates to required reviews based on risk tier, so low-risk changes move fast and high-risk ones get policy and legal review. Work with compliance and policy teams early, and turn their requirements into automated checks.

The tradeoff is speed against safety. Tiering by risk is how you avoid slowing every change. A strong answer explains how to handle a deletion request for data already used in training, and how to measure whether the safeguards actually catch problems.

### Coding

#### Q12. Find the top K most frequent items in a large stream, then explain how you would scale it.

Topics: coding, ml-systems

Clarify first: is the data bounded or streaming, does K change, and is an exact answer required? For a bounded dataset, count with a hash map, then keep a min-heap of size K: O(n log K) time and O(n) space. Bucket sort by frequency gives O(n) when counts are bounded by n. Cover edge cases: K larger than the number of distinct items, ties, and empty input. Write tests for each.

For streaming at scale, exact counting may not fit in memory. Use a count-min sketch to estimate frequencies in fixed memory, paired with a small heap of candidates, or use the Space-Saving algorithm, which gives bounded error. Explain the error guarantees and how memory trades against accuracy.

For distributed data, shard by item key so each worker counts its share, then merge the local top-K lists. Note that merging approximate top-K results needs some extra candidates per shard to stay accurate.

A strong answer gets correct code working before optimizing, states complexity clearly, and links the scaled version to real uses such as trending content or hot embedding IDs.

### Leadership and behavioral

#### Q13. Tell me about the most complex cross-system ML failure you diagnosed.

Topics: behavioral, ml-systems

Choose a story that crossed system boundaries, such as data, training, serving, and hardware, so the interviewer sees range. Use a clear structure: the symptom and its business impact, why it was hard (intermittent, no clear owner, misleading signals), the hypotheses you formed and how you ruled each out, the root cause, and the fix.

Show your method, not just the result. Explain how you narrowed the search: bisecting changes, building a tool to compare logged features, or replaying traffic. Name the people you brought in and how you coordinated across teams that each believed the problem belonged to someone else.

Then cover the systemic fix. At this level the interviewer wants to see that you prevented the whole class of failure: a new monitor, a contract test, a platform change, or a better on-call playbook. Give numbers: time to detect before and after, the revenue or engagement recovered, and incidents avoided since.

Expect follow-ups on what you would do differently and on any wrong turns. A strong answer admits the mistakes honestly and shows what you learned from them.

#### Q14. Describe a time you set multi-year technical direction across teams that disagreed.

Topics: behavioral, people-management, product-strategy

Pick a decision with real stakes, such as a platform migration, an architecture bet, or a change to modeling standards, where reasonable teams had competing interests. Explain each side's position fairly. Interviewers notice when a candidate treats everyone who disagreed as wrong.

Describe how you built the case: data you gathered, prototypes or benchmarks you ran, and a written proposal that laid out alternatives, costs, and risks. Show how you listened to dissent and changed the plan where the critics were right. Explain how the decision was made, who approved it, and how you kept alignment over time with milestones, check-ins, and clear owners.

Cover the outcome with numbers, and be honest about what did not go to plan. For multi-year work, explain how you balanced near-term delivery against long-term architecture, for example by shipping intermediate wins that funded the larger effort.

A strong answer shows influence without authority, strong written communication, including for executives, and the judgment to know when to commit and when to change direction.

#### Q15. How do you mentor senior engineers on debugging complex model behavior?

Topics: behavioral, people-management

Senior engineers rarely need basic lessons. They need judgment and a repeatable method. Teach a debugging playbook: confirm the symptom is real, find the smallest reproducible case, separate data, model, and serving causes, and change one variable at a time. Then practice it together on real incidents instead of in theory.

Pair on live investigations, but let the other engineer drive while you ask questions that expose gaps, such as "how would you know if this feature is skewed?" Review postmortems together, focusing on how the problem could have been caught earlier.

Scale yourself through artifacts: written guides, shared debugging tools, and recorded walkthroughs of hard cases. Give people stretch problems slightly beyond their current scope, with support in place.

Measure success by growth in the scope people take on, by fewer escalations to you over time, and by mentees who become mentors themselves.

A strong answer gives a concrete example of someone you helped grow, explains how you adjusted your approach to that person, and shows that you share credit openly.

## Prep checklist

- [ ] Read Meta's public ML full loop prep page and its downloadable guide.
- [ ] Prepare two or three org-scale ML system stories with real numbers: latency, efficiency, business impact, and the number of teams involved.
- [ ] Practice ML system design for large ranking and recommendation systems end to end, covering retrieval, ranking, features, training, serving, and monitoring.
- [ ] Review sparse plus dense distributed training: embedding sharding, FSDP, all-to-all, and how to reason about MFU.
- [ ] Read Meta's engineering posts on training its ads foundation model and on AI hardware reliability.
- [ ] Rehearse a structured investigation for offline and online metric gaps and for train-serve skew.
- [ ] Do timed coding practice, including at least one session with an AI coding assistant in the editor.
- [ ] Write a one-page design memo for a past system to sharpen executive-level written communication.
- [ ] Prepare an example of using AI tooling to measurably speed up an engineering workflow.

## Sources

- https://www.linkedin.com/jobs/view/software-engineer-machine-learning-at-meta-4479485548 : The job listing (pay, location, responsibilities, qualifications).
- https://www.metacareers.com/ML-prep-onsite : Meta's ML full loop prep page, which describes up to six 45-minute conversations.
- https://www.metacareers.com/swe-prep-onsite : Meta's software engineering full loop prep page.
- https://interviewing.io/blog/how-to-use-ai-in-meta-s-ai-assisted-coding-interview-with-real-prompts-and-examples : Third-party write-up of Meta's AI-assisted coding round pilot.
- https://www.datainterview.com/blog/meta-machine-learning-engineer-interview : Third-party guide to Meta MLE loop stages and themes.
- https://www.teamblind.com/post/meta-e7-interview-experience-iw3gnkuz : Anecdotal candidate discussion of a senior IC loop.
- https://crackingwalnuts.com/companies/meta/e6 : Third-party staff-level loop guide based on candidate reports.
- https://engineering.fb.com/2026/08/03/ml-applications/training-gem-at-llm-scale-meta-ads-recommendation-foundation-model/ : Meta engineering post on scaling training for its ads foundation model.
- https://engineering.fb.com/2025/07/22/data-infrastructure/how-meta-keeps-its-ai-hardware-reliable : Meta engineering post on silent data corruption and AI hardware reliability.
- https://github.com/meta-pytorch/torchrec : TorchRec, Meta's PyTorch library for large-scale recommender systems.
- https://github.com/meta-recsys/generative-recommenders : Meta's generative recommenders (HSTU) code.
