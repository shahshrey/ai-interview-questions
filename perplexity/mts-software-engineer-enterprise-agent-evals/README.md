# Perplexity: Member of Technical Staff (Software Engineer, Enterprise Agent Evals & Efficiency)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop is an estimate drawn from public guides and anecdotal candidate reports. Perplexity has not confirmed it.

1. Recruiter screen (20 to 30 minutes) covering your background, your motivation, and your real opinions about the product.
2. Technical screen (45 to 60 minutes) with applied coding, most likely in Python. It often comes in multiple parts, with requirements added as you go.
3. Onsite or virtual onsite of about 3 to 5 rounds. Expect applied coding, a system design round likely tilted toward eval or agent infrastructure, a domain deep dive on agent evaluation and efficiency, and a behavioral round on ownership and cross-team leadership.
4. Possibly a conversation with a senior leader. Some guides report this round, so treat it as uncertain.

Reports describe the process as fast. Ask your recruiter for the exact list of rounds.

## Questions and answers

### Evaluation environments and graders

#### Q1. Design an evaluation environment that simulates an enterprise workspace (chat, docs, tickets, CRM) with user roles and permissions.

I'd start from the connector interfaces, because those are what the agent actually touches. Put sandboxed or mock services behind the same interfaces, so the tool schemas match production and the agent under test sees the same thing it would see live. Seed each service with data generated from templates that mirror how enterprise data really looks, with overlapping project names, stale documents, duplicate contacts, and long threads.

Reproducibility is the hard part. Snapshot the full state of every service before a trial and restore it afterward, freeze the clock, and fix random seeds for any simulated users. Run each trial in its own isolated container so parallel runs keep separate state.

Model roles and permissions directly. Define identities with different access lists, and include tasks where the correct answer is "you do not have access." Grade on the final state of the environment as well as on what the agent says. Was the ticket created with the right fields? Did the message go to the right channel?

High-fidelity mocks are expensive to maintain and drift away from the real APIs. Recorded responses are cheap, but they break when the agent takes a new path. I'd mention contract tests that keep the mocks in sync with the real connectors, plus a small set of tasks that run against real staging accounts.

#### Q2. How would you build an automated grader for multi-step enterprise tasks, and how would you know it agrees with humans?

Pick the cheapest reliable grader for each check. Anything with a verifiable outcome gets deterministic checks against environment state: the record exists, the fields match, no extra writes happened, and permissions were respected. Save the LLM judge for open-ended output like a summary or a drafted email, and give it a task-specific rubric with clear pass and fail examples.

Break each task into several independent assertions, so a failure tells you what went wrong. Track partial credit separately from the binary success metric you report.

To measure agreement, build a calibration set of a few hundred trajectories labeled by people who know the domain, and talk through their disagreements until they're resolved. Measure the judge's agreement with Cohen's kappa. Its precision and recall on failures matter more, because a grader that misses real failures does more damage than a noisy one. Re-run calibration whenever you change the judge model or prompt, and spot-check a sample of graded runs every week.

LLM judges scale, but they lean toward long or confident answers. Human review is accurate and slow. I'd treat the grader as a product in its own right, with its own tests and versioning.

#### Q3. How do you choose tasks so the eval suite reflects real enterprise work?

Start from real usage. Sample anonymized production request patterns and failure reports, and turn them into tasks with clear success criteria. Then stratify along the dimensions that matter: which connectors are involved, how many steps the workflow takes, whether it reads or writes, whether it crosses apps, and how ambiguous the request is.

Weight the suite toward the work customers care about most, and keep coverage of the long tail, because rare workflows are where agents tend to fail badly. Include negative cases too. That means requests the agent should refuse, data it cannot access, and tasks where the right move is to ask a clarifying question.

The suite needs upkeep. Retire tasks that every model passes, since they've stopped telling you anything, and add new tasks mined from fresh production failures. Keep a held-out set that nobody tunes against, so you can detect overfitting to the main suite.

Realistic tasks are expensive to write and grade. Synthetic tasks are cheap but can miss real difficulty. In the interview, describe a pipeline that turns production signals into new tasks, with a review step for quality and privacy, and report results slice by slice.

### Statistics and experiment judgment

#### Q4. When would you report pass@k versus pass^k, and how many trials per task do you need?

pass@k is the chance that at least one of k attempts succeeds. It fits settings where a user or system can pick the best attempt. pass^k is the chance that all k attempts succeed, so it measures consistency.

An agent doing real enterprise work gets judged one run at a time, and users expect it to work every time. For that case, pass^k, or plain per-trial success with its variance, is closer to what matters. The two numbers diverge sharply as k grows, which is how an unreliable agent can look good when pass@k is the only number reported.

For trial count, start by estimating per-task variance. Agent runs are noisy, and one trial per task gives misleading rankings. In practice I'd run 3 to 5 trials per task, compute confidence intervals with bootstrap resampling over tasks, and add trials for the slices where the intervals overlap.

To compare two versions, run paired comparisons on the same tasks and seeds, which removes much of the between-task variance. Trials cost money, so keep a smaller, high-signal subset for fast iteration and use the full suite when you need to make a decision.

#### Q5. A change improves task success from 62% to 65% on your suite. Is it real?

First ask how big the suite is and how many trials ran. Three points on 100 tasks with one trial each is well within noise. Both versions ran on the same tasks, so compute a confidence interval on the difference with a paired test, such as a paired bootstrap over tasks or McNemar's test on per-task outcomes.

Then look under the average. Which tasks flipped from fail to pass, and which flipped back? If the gain comes from a handful of high-variance tasks, re-run those with more trials. A gain concentrated in the slice the change was meant to help is more convincing than one spread thinly across the suite.

Check for confounds: different model versions, changed grader prompts, flaky environment services, or data that depends on time. Look at the secondary metrics as well, including latency, tokens, and tool calls. A quality gain that doubles the cost may not be worth shipping.

I'd close with a decision rule set before the experiment runs. It covers the minimum detectable effect, the number of trials, and the slices that are not allowed to regress. Committing to it up front keeps you from reading noise as progress after the fact.

### Trajectory analysis and tool design

#### Q6. Trajectories show the agent picking the wrong connector tool. How do you find the root cause and prove the fix worked?

Start by counting. Tag every failed trajectory with the first step where it went wrong, then group the failures by tool pair, meaning the tool that was called and the tool that should have been. A few pairs usually account for most of them.

For each pair, read the raw transcripts as well as the summaries. Common causes are two tools with overlapping descriptions (search messages versus search files), so many tools exposed at once that the right one gets lost, vague parameter names, missing context about which apps the user has connected, and a system prompt that biases the agent toward one tool.

Each cause has its own fix. Rewrite descriptions to say when to use each tool and when another tool fits better. Namespace tools by app. Expose a smaller, task-relevant set through tool retrieval or on-demand loading. Add the user's connected apps to the context.

To prove the fix, build a targeted slice of tasks that exercise the confused pair, run multiple trials before and after, and compare tool-selection accuracy and end-to-end success. Then run the full regression suite, because a description change that helps one tool can steal calls from another. Keep the targeted slice afterward as a permanent regression test.

#### Q7. Walk through how you would improve one tool's description, schema, and result format for agents.

Write the tool for a model as the reader. The description should say what the tool does, when to use it, when another tool is the better choice, and what it returns. A few short examples of good calls help more than long prose.

In the schema, give parameters clear, unambiguous names, use enums where the values are fixed, set sensible defaults, and validate input so a bad call gets back a useful message. When a parameter needs an ID, let the agent look it up by name or have related tools return the ID.

For results, return the high-signal fields, with readable names next to the IDs. When more data exists, paginate or truncate and say so in a clear note. Huge raw JSON payloads waste tokens and bury the answer. A concise mode and a detailed mode can cover both needs.

Measure before and after on a focused slice, tracking correct tool selection, argument validity, retries, tokens per call, and end-to-end success. Trimming results saves tokens, but it can force extra calls when the agent needs more detail. Read the transcripts to see how the model actually used the tool, and walk the interviewer through that measurement loop.

#### Q8. The agent keeps looping on a failing API call. How do you design error handling and recovery?

Loops usually mean the agent cannot tell what went wrong or what to try next, so start with the error message the tool returns. Make it actionable by saying what failed, why, and what the agent can do about it, for example "date must be ISO 8601" or "you lack write access to this channel; ask the user or choose another."

Handle the two error types differently. Retry transient errors such as timeouts and rate limits inside the tool with backoff, where the model never sees them. Send permanent errors such as bad input or a missing permission straight back to the model with guidance. Give writes idempotency keys so a retry cannot create a duplicate.

In the agent loop, cap repeated identical calls, detect loops by comparing recent actions, and give the agent a way to stop and report a partial result.

To evaluate all this, add fault injection to the eval environment: random timeouts, rate limits, expired tokens, and malformed responses. Track recovery rate, wasted calls per task, and how often the agent tells the user honestly that something failed. Balance persistence against cost, and count an honest partial result as better than a silent failure.

#### Q9. How would you evaluate whether an agent respects permissions and keeps data from leaking across users?

Build it into the environment from the start. Seed data with explicit access rules, such as private channels, restricted documents, and records only managers can see. Then run tasks as users with different roles, including tasks built to tempt leakage, like summarizing a project when some of the related documents are restricted.

Grade with hard checks. Scan the agent's output and every tool call argument for content or identifiers from data the acting user cannot see. Verify that the agent used the user's own credentials and stayed off any elevated service account. Check that writes touched only resources the user may modify.

Add adversarial cases as well, such as instructions hidden in documents that tell the agent to forward data elsewhere, and requests worded to get around access rules.

Treat these results as gates. One leak in a thousand runs blocks a launch, so report the counts and examples directly and keep them separate from any blended quality score. Strict scanning can flag public information that happens to match, so tune it with human review. I'd also describe defense in depth: the connectors enforce permissions, and the evals confirm that enforcement holds end to end.

### Efficiency, routing, and context

#### Q10. Design a model routing system that picks the model and effort level for each task or step.

Define the objective first. You might maximize task success subject to a quality floor, or minimize cost per successful task subject to latency limits. Without a clear objective, routing just moves cost around.

Start with rules. Routing on workflow type and step type (planning, tool argument filling, summarization) often captures most of the savings. Then collect data by running candidate models on eval tasks and recording success, latency, and cost, and train a lightweight router on request and context features to predict which model will succeed cheaply. Preference-based routers in public research have shown large cost cuts at similar quality.

Route each step as well as each request. A strong model can plan while cheaper models handle extraction or formatting. Add escalation, so when a cheap model fails validation or its confidence is low, a stronger model retries.

Evaluate the router itself on held-out tasks and track the whole quality-cost curve. Two problems come up often. A router trained on old models goes stale when new models ship, and escalation retries can erase the savings. Monitor routing decisions in production, and keep a fallback path for when a provider is down.

#### Q11. How would you reduce cost per successful task by 30% while holding quality steady?

Measure before you cut. Break token spend down by component: system prompt, tool schemas, skill instructions, conversation history, tool results, and reasoning. Count failed runs too, since cost per successful task rises sharply when tasks fail and get retried.

Then work through the biggest buckets. Load only the tools and skills the task needs. Trim verbose tool results. Compact long histories into summaries. Order prompts so the stable prefix gets cached and reused. Route simple steps to cheaper models and save the strongest model for planning and hard reasoning. Fix tool confusion to cut wasted calls, which improves quality at the same time.

A higher success rate lowers cost by itself. If success goes from 60% to 70% at the same spend, cost per successful task drops by about 14%.

Validate with paired evals, and set guardrails before you start, with agreed thresholds that success, latency percentiles, and safety checks have to stay within. Roll out behind a flag and confirm the result online. In the interview, list the changes in order of expected savings and risk, and show a few clear wins.

#### Q12. What are the tradeoffs of context compaction and prompt caching in long-running agents?

Prompt caching saves cost and time when the start of the prompt stays the same across calls. That rewards a stable layout, with the fixed system prompt and tool definitions first and the growing history after them. Any change early in the prompt, such as a timestamp or a reordered tool list, breaks the cache for everything after it.

Compaction shrinks history by summarizing older turns or dropping old tool results. It keeps long tasks inside the context window and lowers cost, but it can drop a detail the agent needs later, such as an ID or a user constraint. Compaction also rewrites the prompt, which can invalidate the cache, so the two techniques pull against each other.

In practice, compact at defined points in the run. Keep structured notes of the important facts (IDs, decisions, constraints) outside the summary, and let the agent re-fetch details through tools when it needs them.

Evaluate with long-horizon tasks that require recalling early facts late in the run. Track success, tokens, cache hit rate, and latency together, and be ready to explain why optimizing any one of them alone gives a misleading picture.

### Launch gates and production

#### Q13. Design regression gates for changes to the model, connectors, tools, skills, and agent harness.

Different changes need different gates, and they should all run on one shared system. Tag every task with the components it exercises, so each change runs the slice that covers it.

Use tiers. A fast smoke suite runs on every pull request and finishes in minutes. A broader slice for the touched connectors or skills runs on merge. The full suite, with multiple trials, runs nightly and before any model swap or launch.

Set each slice's threshold from the noise level you measured for it. Some checks are hard gates with zero tolerance, such as permission leaks or destructive writes. Others are soft gates, where a dip needs a human to sign off.

Store every run with the versions of the model, prompts, tools, and graders, so you can explain changes and reproduce failures. Publish a launch-readiness view that shows success, latency, cost, and safety for each slice against the current baseline.

Big suites catch more, but they slow everyone down and cost real money. Handle flaky tasks explicitly by quarantining them, which keeps the team's trust in the gate.

#### Q14. Offline eval gains aren't showing up in production. What do you do?

Treat it as a measurement bug until the evidence says otherwise. Start with the distribution, and compare production requests to eval tasks by connector, workflow length, ambiguity, and data size. Evals often over-represent clean, well-specified tasks.

Next, check environment fidelity. Mocks may respond faster, return tidier data, or skip permission errors that real systems produce. Then check the grader, which may reward outputs users do not actually value, such as an answer that is correct but unhelpful.

Look hard at the production metric too. Thumbs-up rates, retries, and follow-up questions are noisy and lag behind changes. Measure task success in a comparable way, ideally by grading sampled production trajectories with the same rubric.

To fix the gap, mine production failures into new eval tasks, add the messy data shapes the evals were missing, and run an online experiment comparing the old and new versions with guardrail metrics. Keep a small set of production-sampled tasks as a standing check that offline and online results stay aligned.

Some offline gains will not transfer, and it's fine to say so. The goal is to close the loop and learn which eval signals predict production impact.

### Coding and ownership

#### Q15. Write an async Python eval runner with a concurrency limit, per-task timeouts, retries, and aggregated metrics. Then describe how you would extend it.

Structure the code around two dataclasses. A task has an id, an input, slice tags, and a grader. A result has the task id, the trial, success, latency, tokens, cost, and error. Use asyncio with a semaphore to cap concurrency, so the run stays within what model providers and environment services can handle.

Wrap each trial in asyncio.wait_for for the timeout. Retry only transient errors such as rate limits and connection resets, with exponential backoff and jitter, and record the attempt count. Surface grader failures instead of retrying them, and log them as their own error type so they stay distinct from agent failures.

Aggregate once all trials finish, computing success rate per task and per slice, latency percentiles, mean tokens and cost, and cost per successful task. Write raw results to a file in a stable format so you can compare runs.

Expect follow-ups such as resuming an interrupted run while skipping finished trials, resetting the environment for each trial, streaming progress, and comparing two runs with paired statistics. Write clean, typed, readable code, handle cancellation correctly, and keep the runner deterministic given a seed. Interviewers usually weigh clear error handling as heavily as the happy path.

## Prep checklist

- [ ] Build a small agent eval end to end, with a few tools, a seeded fake workspace, about 20 tasks, an outcome grader, and multiple trials per task.
- [ ] Practice explaining pass@k versus pass^k, and run a paired bootstrap on two versions.
- [ ] Rewrite one tool's description and schema, then measure how tool-selection accuracy and tokens change.
- [ ] Sketch a model routing design with escalation and a quality-cost curve.
- [ ] Review how prompt caching behaves and what context compaction trades off in long-running agents.
- [ ] Practice timed, multi-part Python coding with asyncio, dataclasses, and clean error handling.
- [ ] Read Perplexity's public posts on Computer, DRACO, and WANDR, and form your own opinion on how they evaluate agents.
- [ ] Use Perplexity's agent features with connectors and write down two concrete failure modes you could measure.
- [ ] Prepare one story about leading an ambiguous cross-team project from evaluation findings to a shipped, measured improvement.

## Sources

- https://jobs.accel.com/companies/perplexity-2/jobs/97418124-member-of-technical-staff-software-engineer-enterprise-agent-evals-efficiency : the same role listed on a public job board.
- https://www.finalroundai.com/blog/perplexity-interview-process : a third-party guide to the Perplexity loop, best read as an estimate.
- https://www.linkjob.ai/interview-questions/perplexity-ai-interview : an anecdotal candidate report describing a fast, Python-focused process.
- https://www.teamblind.com/post/perplexity-interview-zns2oibc : anecdotal comments on how the rounds are structured.
- https://dataford.io/interview-guides/perplexity/software-engineer : a third-party guide describing incremental coding and design rounds.
- https://www.perplexity.ai/hub/blog/introducing-perplexity-computer : Perplexity's post on sub-agents, connectors, and multi-model orchestration.
- https://arxiv.org/html/2602.11685v1 : DRACO, Perplexity's deep research benchmark built from real usage with expert rubrics.
- https://www.perplexity.ai/hub/blog/wandr-benchmark-evaluating-research-agents-that-must-search-wide-and-deep : the WANDR benchmark, which reports quality alongside cost and latency.
- https://www.anthropic.com/research/demystifying-evals-for-ai-agents : an industry reference on agent eval terms, graders, and pass@k versus pass^k.
- https://www.anthropic.com/research/writing-tools-for-agents : an industry reference on designing tools for agents.
- https://openreview.net/forum?id=roNSXZpUDN : the tau-bench paper on tool-using agents in stateful, policy-bound environments.
- https://www.lmsys.org/blog/2024-07-01-routellm : RouteLLM, public research on learned model routing.
