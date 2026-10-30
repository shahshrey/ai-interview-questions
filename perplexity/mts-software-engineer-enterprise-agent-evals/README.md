# Perplexity: Member of Technical Staff (Software Engineer, Enterprise Agent Evals & Efficiency)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This is an estimate from public guides and anecdotal candidate reports, not confirmed by Perplexity.

1. Recruiter screen (20 to 30 minutes): background, motivation, and real opinions about the product.
2. Technical screen (45 to 60 minutes): applied coding, most likely in Python, often in multiple parts with requirements added as you go.
3. Onsite or virtual onsite, about 3 to 5 rounds: applied coding, a system design round likely tilted toward eval or agent infrastructure, a domain deep dive on agent evaluation and efficiency, and a behavioral round on ownership and cross-team leadership.
4. Possibly a conversation with a senior leader. Some guides report this; treat it as uncertain.

Reports describe the process as fast. Ask your recruiter for the exact round list.

## Questions and answers

### Evaluation environments and graders

#### Q1. Design an evaluation environment that simulates an enterprise workspace (chat, docs, tickets, CRM) with user roles and permissions.

Start from what the agent actually touches: the connector interfaces. Put sandboxed or mock services behind those same interfaces so the agent under test cannot tell the difference, and so tool schemas stay identical to production. Each service gets a seeded dataset generated from templates that mirror real enterprise shapes: overlapping project names, stale documents, duplicate contacts, long threads.

Reproducibility is the hard part. Snapshot the full state of every service before a trial and restore it after, freeze the clock, and fix random seeds for any simulated users. Run each trial in an isolated container so parallel runs never share state.

Roles and permissions should be first-class: define identities with different access lists, and include tasks where the right answer is "you do not have access." Grade on the final state of the environment (was the ticket created with the right fields, was the message sent to the right channel), not only on what the agent says.

Tradeoffs: high-fidelity mocks are expensive to maintain and drift from real APIs; recorded responses are cheap but break when the agent takes a new path. A strong answer mentions contract tests that keep mocks in sync with real connectors, plus a small set of tasks run against real staging accounts.

#### Q2. How would you build an automated grader for multi-step enterprise tasks, and how would you know it agrees with humans?

Use the cheapest reliable grader for each check. For anything with a verifiable outcome, write deterministic checks against environment state: the record exists, fields match, no extra writes happened, permissions were respected. Use an LLM judge only for open-ended output such as a summary or drafted email, and give it a task-specific rubric with clear pass and fail examples instead of a vague 1 to 10 score.

Split grading into several independent assertions per task so a failure tells you what went wrong. Keep partial credit separate from the binary success metric you report.

To check agreement, build a calibration set: a few hundred trajectories labeled by people who know the domain, with disagreements discussed and resolved. Measure the judge's agreement with Cohen's kappa and, more important, its precision and recall on failures, since a grader that misses real failures is worse than a noisy one. Re-run calibration when you change the judge model or prompt, and spot-check a sample of graded runs every week.

Tradeoffs: LLM judges scale but carry bias toward long or confident answers; human review is accurate but slow. A strong answer treats the grader as a product with its own tests and versioning.

#### Q3. How do you choose tasks so the eval suite actually represents enterprise work?

Start from usage, not imagination. Sample anonymized production request patterns and failure reports, then turn them into tasks with clear success criteria. Stratify across the dimensions that matter: which connectors are involved, how many steps the workflow takes, whether it reads or writes, whether it crosses apps, and how ambiguous the request is.

Weight the suite toward work that customers care about most, but keep coverage of the long tail, because rare workflows are where agents tend to fail badly. Include negative cases: requests the agent should refuse, data it cannot access, and tasks where asking a clarifying question is the right move.

Maintain the suite over time. Retire tasks every model passes, since they no longer tell you anything, and add new tasks mined from fresh production failures. Keep a held-out set that nobody tunes against, so you can detect overfitting to the main suite.

Tradeoffs: realistic tasks are expensive to write and grade; synthetic tasks are cheap but can miss real difficulty. A strong answer describes a pipeline from production signal to new task, with a review step for quality and privacy, and reports results per slice rather than one blended number.

### Statistics and experiment judgment

#### Q4. When would you report pass@k versus pass^k, and how many trials per task do you need?

pass@k is the chance that at least one of k attempts succeeds. It fits settings where a user or system can pick the best attempt. pass^k is the chance that all k attempts succeed, which captures consistency. For an agent doing real enterprise work, users experience one run at a time and expect it to work every time, so pass^k, or simply per-trial success with its variance, is closer to what matters. As k grows the two diverge sharply, which is why reporting only pass@k can hide an unreliable agent.

For trial count, start by estimating per-task variance. Agent runs are noisy, so one trial per task gives misleading rankings. A practical approach: run 3 to 5 trials per task, compute confidence intervals with bootstrap resampling over tasks, and increase trials for the slices where intervals overlap.

When comparing two versions, use paired comparisons on the same tasks and seeds, since that removes much of the between-task variance. A strong answer also mentions the cost of trials, choosing a smaller, high-signal subset for fast iteration, and the full suite for decisions.

#### Q5. A change improves task success from 62% to 65% on your suite. Is it real?

First ask how big the suite is and how many trials ran. Three points on 100 tasks with one trial each is well within noise. Compute a confidence interval on the difference using a paired test, such as a paired bootstrap over tasks or McNemar's test on per-task outcomes, since both versions ran on the same tasks.

Then look beneath the average. Which tasks flipped from fail to pass, and which flipped the other way? If the gain comes from a handful of tasks with high variance, re-run them with more trials. If the gain clusters in one slice that matches the change's intent, that is more convincing than a diffuse gain.

Check for confounds: different model versions, changed grader prompts, flaky environment services, or time-dependent data. Look at secondary metrics too, such as latency, tokens, and tool calls, because a quality gain bought with double the cost may not be worth shipping.

A strong answer ends with a decision rule set before the experiment: minimum detectable effect, number of trials, and which slices must not regress. That prevents reading noise as progress after the fact.

### Trajectory analysis and tool design

#### Q6. Trajectories show the agent picking the wrong connector tool. How do you find the root cause and prove the fix worked?

Start by quantifying it. Tag every failed trajectory with the first step where it went wrong, then group by tool pair: which tool was called and which should have been. Usually a few pairs dominate.

For each pair, read raw transcripts, not just summaries. Common causes: two tools with overlapping descriptions (search messages versus search files), too many tools exposed at once so the right one gets lost, vague parameter names, missing context about which apps the user has connected, or a system prompt that biases toward one tool.

Fixes map to causes. Rewrite descriptions to state when to use each tool and when not to. Namespace tools by app. Expose a smaller, task-relevant set through tool retrieval or on-demand loading. Add the user's connected apps to context.

To prove it, build a targeted slice of tasks that exercise the confused pair, run multiple trials before and after, and compare tool-selection accuracy and end-to-end success. Then run the full regression suite, because a description change that helps one tool can steal calls from another. A strong answer also turns the fix into a permanent regression test.

#### Q7. Walk through how you would improve one tool's description, schema, and result format for agents.

Treat the tool as an interface for a model, not for a developer. The description should say what the tool does, when to use it, when to use something else instead, and what it returns. Short examples of good calls help more than long prose.

For the schema, use clear, unambiguous parameter names, enums instead of free text where values are fixed, sensible defaults, and validation that rejects bad input with a useful message. Avoid parameters that require IDs the agent cannot easily get; offer lookup by name or return IDs from related tools.

For results, return only high-signal fields, readable names alongside IDs, and pagination or truncation with a clear note when more data exists. Huge raw JSON payloads waste tokens and bury the answer. Consider a concise and a detailed mode.

Measure before and after on a focused slice: correct tool selection, argument validity, retries, tokens per call, and end-to-end success. Tradeoffs: trimming results saves tokens but can force extra calls when the agent needs more detail. A strong answer shows the measurement loop and mentions reading transcripts to see how the model actually used the tool.

#### Q8. The agent keeps looping on a failing API call. How do you design error handling and recovery?

Loops usually mean the agent cannot tell what went wrong or what to try next. Start with the error message the tool returns: make it actionable. Say what failed, why, and what the agent can do, for example "date must be ISO 8601" or "you lack write access to this channel; ask the user or choose another."

Separate error types. Transient errors (timeouts, rate limits) should be retried inside the tool with backoff, invisible to the model. Permanent errors (bad input, missing permission) should go back to the model immediately with guidance. Add idempotency keys for writes so retries never create duplicates.

In the harness, cap repeated identical calls, detect loops by comparing recent actions, and give the agent a way to stop and report a partial result instead of spinning.

To evaluate, add fault injection to the eval environment: random timeouts, rate limits, expired tokens, and malformed responses. Track recovery rate, wasted calls per task, and how often the agent tells the user honestly that something failed. A strong answer balances persistence against cost and notes that silent failure is worse than an honest partial result.

#### Q9. How would you evaluate whether an agent respects permissions and never leaks data across users?

Build this into the environment from the start. Seed data with explicit access rules: private channels, restricted documents, records visible only to managers. Then run tasks as users with different roles, including tasks designed to tempt leakage, such as summarizing a project when some related documents are restricted.

Grade with hard checks. Scan the agent's output and every tool call argument for content or identifiers from data the acting user cannot see. Verify that the agent used the user's own credentials, not an elevated service account. Check that writes only touched resources the user may modify.

Include adversarial cases: instructions hidden in documents that tell the agent to forward data elsewhere, and requests that try to get around access rules through wording.

Treat these results as gates rather than averages. One leak in a thousand runs is a launch blocker, so report counts and examples, not a percentage blended into a quality score. Tradeoffs: strict scanning can produce false positives on public information that happens to match; tune with human review. A strong answer mentions defense in depth: permission enforcement in connectors, plus evals that confirm it holds end to end.

### Efficiency, routing, and context

#### Q10. Design a model routing system that picks the model and effort level for each task or step.

Define the objective first: maximize task success subject to a quality floor, or minimize cost per successful task subject to latency limits. Routing without a clear objective just moves cost around.

Start simple. Rules based on workflow type and step type (planning, tool argument filling, summarization) often capture most of the savings. Then collect data: run candidate models on eval tasks, record success, latency, and cost, and train a lightweight router on request and context features to predict which model will succeed cheaply. Preference-based routers in public research have shown large cost cuts at similar quality.

Route per step, not just per request. A strong model can plan while cheaper models handle extraction or formatting. Add escalation: if a cheap model fails validation or confidence is low, retry with a stronger one.

Evaluate the router itself on held-out tasks, tracking the quality-cost curve, not a single point. Watch for traps: a router trained on old models goes stale when new models ship, and escalation retries can erase savings. A strong answer includes monitoring of routing decisions in production and a fallback path when a provider is down.

#### Q11. How would you reduce cost per successful task by 30% without hurting quality?

Measure before cutting. Break token spend down by component: system prompt, tool schemas, skill instructions, conversation history, tool results, and reasoning. Also count failed runs, since cost per successful task rises sharply when tasks fail and get retried.

Then work the largest buckets. Load only the tools and skills relevant to the task instead of all of them. Trim verbose tool results. Compact long histories into summaries. Order prompts so the stable prefix is cached and reused. Route simple steps to cheaper models and reserve the strongest model for planning and hard reasoning. Reduce wasted calls by fixing tool confusion, which also improves quality.

Raising success rate is itself a cost lever: if success goes from 60% to 70% at the same spend, cost per successful task drops by about 14%.

Validate with paired evals and set guardrails before starting: success, latency percentiles, and safety checks must not regress beyond agreed thresholds. Roll out behind a flag and confirm online. A strong answer lists changes in order of expected savings and risk, and shows a few clear wins rather than many tiny tweaks.

#### Q12. What are the tradeoffs of context compaction and prompt caching in long-running agents?

Prompt caching saves cost and time when the start of the prompt stays the same across calls. That rewards a stable layout: fixed system prompt and tool definitions first, then growing history. Anything that changes early in the prompt, such as a timestamp or a reordered tool list, breaks the cache for everything after it.

Compaction shrinks history by summarizing older turns or dropping old tool results. It keeps long tasks within the context window and lowers cost, but it can drop a detail the agent needs later, such as an ID or a user constraint. Compaction also rewrites the prompt, which can invalidate the cache, so the two pull against each other.

Practical approaches: compact at defined points rather than every turn, keep structured notes of key facts (IDs, decisions, constraints) outside the summary, and let the agent re-fetch details through tools instead of holding everything in context.

Evaluate with long-horizon tasks that require recalling early facts late in the run. Track success, tokens, cache hit rate, and latency together. A strong answer explains why optimizing one metric alone misleads.

### Launch gates and production

#### Q13. Design regression gates for model, connector, tool, skill, and harness changes.

Different changes need different gates, but they should share one system. Tag every task with the components it exercises so you can run the relevant slice for a given change.

Use tiers. A fast smoke suite runs on every pull request and finishes in minutes. A broader slice runs on merge for the touched connectors or skills. The full suite with multiple trials runs nightly and before any model swap or launch.

Set thresholds per slice using the noise level you measured, not a fixed percentage. Some checks are hard gates with zero tolerance, such as permission leaks or destructive writes. Others are soft gates that require a human to sign off when they dip.

Store every run with versions of model, prompts, tools, and graders so you can explain changes and reproduce failures. Publish a launch-readiness view that shows success, latency, cost, and safety per slice against the current baseline.

Tradeoffs: big suites catch more but slow everyone down and cost real money. A strong answer handles flaky tasks explicitly, quarantining them rather than letting them erode trust in the gate.

#### Q14. Offline eval gains are not showing up in production. What do you do?

Treat it as a measurement bug until proven otherwise. Start with distribution: compare production requests to eval tasks by connector, workflow length, ambiguity, and data size. Evals often over-represent clean, well-specified tasks.

Check environment fidelity. Mocks may respond faster, return tidier data, or skip permission errors that real systems produce. Check the grader: it may reward outputs users do not actually value, such as a correct but unhelpful answer.

Then check the production metric. Thumbs-up rates, retries, and follow-up questions are noisy and lag; make sure you are measuring task success in a comparable way, ideally with sampled production trajectories graded by the same rubric.

Fixes: mine production failures into new eval tasks, add the messy data shapes evals were missing, and run an online experiment comparing old and new versions with guardrail metrics. Keep a small set of production-sampled tasks as a standing check that offline and online stay aligned.

A strong answer frames this as closing the loop, and admits that some offline gains will not transfer, so the goal is to learn which eval signals predict production impact.

### Coding and ownership

#### Q15. Write an async Python eval runner with a concurrency limit, per-task timeouts, retries, and aggregated metrics. Then describe how you would extend it.

Structure the code around a task dataclass (id, input, slice tags, grader) and a result dataclass (task id, trial, success, latency, tokens, cost, error). Use asyncio with a semaphore to cap concurrency so you do not overload model providers or environment services.

Wrap each trial in asyncio.wait_for for the timeout. Retry only transient errors such as rate limits and connection resets, with exponential backoff and jitter, and record the attempt count. Never retry grader failures silently; log them as their own error type so they do not look like agent failures.

Aggregate after all trials finish: success rate per task and per slice, latency percentiles, mean tokens and cost, and cost per successful task. Write raw results to a file in a stable format so you can compare runs.

Likely follow-ups: resume an interrupted run without repeating finished trials, add environment reset per trial, stream progress, and compare two runs with paired statistics. A strong answer writes clean, typed, readable code, handles cancellation correctly, and keeps the runner deterministic given a seed. Interviewers usually care as much about clear error handling as about the happy path.

## Prep checklist

- [ ] Build a small agent eval end to end: a few tools, a seeded fake workspace, about 20 tasks, an outcome grader, and multiple trials per task.
- [ ] Practice explaining pass@k versus pass^k and running a paired bootstrap on two versions.
- [ ] Rewrite one tool's description and schema, and measure the change in tool-selection accuracy and tokens.
- [ ] Sketch a model routing design with escalation and a quality-cost curve.
- [ ] Review prompt caching behavior and context compaction tradeoffs for long-running agents.
- [ ] Practice timed, multi-part Python coding with asyncio, dataclasses, and clean error handling.
- [ ] Read Perplexity's public posts on Computer, DRACO, and WANDR, and form your own opinion on how they evaluate agents.
- [ ] Use Perplexity's agent features with connectors and note two concrete failure modes you could measure.
- [ ] Prepare one story about leading an ambiguous cross-team project from evaluation findings to a shipped, measured improvement.

## Sources

- https://jobs.accel.com/companies/perplexity-2/jobs/97418124-member-of-technical-staff-software-engineer-enterprise-agent-evals-efficiency : the same role on a public job board.
- https://www.finalroundai.com/blog/perplexity-interview-process : third-party guide to the Perplexity loop; treat as an estimate.
- https://www.linkjob.ai/interview-questions/perplexity-ai-interview : anecdotal candidate report noting a fast, Python-focused process.
- https://www.teamblind.com/post/perplexity-interview-zns2oibc : anecdotal comments on the round structure.
- https://dataford.io/interview-guides/perplexity/software-engineer : third-party guide describing incremental coding and design rounds.
- https://www.perplexity.ai/hub/blog/introducing-perplexity-computer : Perplexity's post on sub-agents, connectors, and multi-model orchestration.
- https://arxiv.org/html/2602.11685v1 : DRACO, Perplexity's deep research benchmark built from real usage with expert rubrics.
- https://www.perplexity.ai/hub/blog/wandr-benchmark-evaluating-research-agents-that-must-search-wide-and-deep : WANDR benchmark, reporting quality alongside cost and latency.
- https://www.anthropic.com/research/demystifying-evals-for-ai-agents : industry reference on agent eval terms, graders, and pass@k versus pass^k.
- https://www.anthropic.com/research/writing-tools-for-agents : industry reference on designing tools for agents.
- https://openreview.net/forum?id=roNSXZpUDN : tau-bench paper on tool-using agents in stateful, policy-bound environments.
- https://www.lmsys.org/blog/2024-07-01-routellm : RouteLLM, public research on learned model routing.
