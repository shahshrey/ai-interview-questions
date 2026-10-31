# Databricks: Staff Product Manager, Agentic AI Applications

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop starts from the general Product Manager process that Databricks publishes. Candidate reports and the listing's technical requirements fill in the rest. Teams can change it.

1. A recruiter screen of about 30 minutes covers background, motivation, level, and whether you can talk about data and AI platforms with real specificity.
2. A hiring manager interview of about 45 minutes goes deep on one or two platform projects. Candidates report hiring managers who push back and ask for a deeper second answer. Expect "why that design, what did you trade off, what broke."
3. Panel 1 has four interviews of about 45 minutes each:
   - Building Products: a design case. For this team, expect something like "design the eval gate" or "design the connector SDK."
   - Bringing Products to Market: for an internal platform, this means adoption, migration, and a rollout plan for domain teams.
   - Engineering Interview: architecture tradeoffs at the level the listing names, such as sync versus async, checkpointing, context window management, and auth propagation. One candidate also reported a short, simple Python exercise.
   - Product Management Leadership: influence across 4 or more teams, and how you handle conflict.
4. Panel 2 is one Executive Product Leadership interview of about 45 minutes on your 2 or 3 biggest contributions. One candidate report describes a leadership panel that gave feedback mid-interview and tested how the candidate took objections.
5. A take-home asks for Critical User Journeys and a PRD. A plausible prompt is one internal agent or one platform capability. Some senior candidates present it.
6. References, which candidates say can include backchannel checks, then a hiring committee decision and the offer.

## Questions and answers

### Platform strategy

#### Q1. You inherit the internal agent platform with 14 stalled agent projects across GTM, Finance, HR and Legal. How do you sequence the first two quarters? Show your scoring.

Start with an audit. For each stalled project, record the blocking step, the weeks it has been blocked, and whether the blocker is shared. Shared blockers become platform work. One-off blockers go back to the domain team.

Then score each candidate investment by unblocked value per unit of engineering cost:

score = (projects unblocked x weeks saved per project) / engineering weeks to build

Illustrative numbers from an audit like this:

- Identity-propagating connectors for Salesforce and Workday unblock 9 projects and save about 6 weeks each, for 12 engineering weeks. Score = (9 x 6) / 12 = 4.5.
- A shared eval harness with a CI gate unblocks 14 projects and saves 2 weeks each, for 8 engineering weeks. Score = 28 / 8 = 3.5.
- Durable execution for long-running jobs unblocks 4 projects and saves 5 weeks each, for 10 engineering weeks. Score = 20 / 10 = 2.0.
- Long-term memory unblocks 3 projects and saves 3 weeks each, for 10 engineering weeks. Score = 9 / 10 = 0.9.

Quarter 1 ships connectors plus the eval harness as one thin vertical slice for two lighthouse agents. Pick one read-heavy agent, such as GTM account research, and one write agent, such as Finance accruals, so that both safety paths get exercised early. Quarter 2 adds durable execution and the self-serve SDK, and memory waits until a third domain asks for it.

Two adjustments keep the score honest. First, apply a risk multiplier, because a write-path capability that prevents a duplicate journal entry deserves more weight than raw weeks saved suggest. Second, add a dependency check. Durable execution scores lower, but the Finance write agent cannot go to production without idempotent retries, so it moves ahead of anything that does not gate a lighthouse.

State the exit criteria for each quarter in numbers. By the end of Q1, both lighthouses are in production, median time from idea to production is under 8 weeks, and every production agent ran through the eval gate. By the end of Q2, median time is under 4 weeks, which is the listing's target, and at least 60 percent of new agents start on the platform.

Why this is likely: the listing asks for a roadmap with measurable success criteria, and the official PM guide's Building Products round tests prioritization and execution.

### Agent runtime and model gateway

#### Q2. A Finance reconciliation agent runs 40 tool-calling steps over 30 minutes and posts journal entries. Explain checkpointing versus durable execution, and what the platform must guarantee.

Checkpointing saves state. LangGraph, for example, persists graph state at each superstep under a thread_id to Postgres or SQLite. If the process dies, the data survives, but nothing detects the failure or resumes the run. Durable execution, the Temporal or Dapr Workflows model, also owns recovery. The workflow is a deterministic function whose event history is persisted. Each side-effecting call runs as an activity with its own retry policy. After a crash, a worker replays the history, skips the activities that already completed using their recorded results, and continues from the first unfinished step. Human approval becomes a durable wait that can last days without holding a process.

To see why this matters in numbers, suppose each step has a 0.5 percent chance of a transient failure, such as a model timeout, a 429, or a deploy.

- P(a 40-step run completes with no failure) = 0.995^40 = e^(40 x ln 0.995) = e^(-0.2005), which is about 0.818.
- Without durability, about 18 percent of runs restart from step 1. With checkpoints but no recovery, they sit dead until someone notices.
- With durable execution, a failure costs one retried step instead of an average of about 20 repeated steps.

The hard part is writes. Replay plus retry means an activity can execute twice, so every write to a system of record needs an idempotency key. A deterministic key such as hash(run_id, step_id, entity_id) lets the ERP connector deduplicate. Activities also need to be split so that one activity does one external write. Model calls are non-deterministic, so their outputs get recorded in history and never re-sampled on replay. Otherwise the resumed run can take a different branch.

The PM decides on these platform guarantees:

- Exactly-once effect for writes, which means at-least-once execution plus idempotent connectors.
- Durable human-in-the-loop waits for approvals above a dollar threshold.
- Per-run ceilings on cost, wall clock, and records modified.
- Encrypted checkpoint storage with the same access controls as the source data, because checkpoints hold financial data.

Track run completion rate, mean steps re-executed per failure, and duplicate-write incidents, with a target of zero.

Why this is likely: the listing names durable execution, checkpointing, and sync versus async tradeoffs, and Temporal and Diagrid both publish the checkpoint-versus-durability distinction.

#### Q3. Walk through how the model gateway should route traffic when you migrate one domain's agents to a new model. Cover traffic splitting, fallbacks, rate limits and caching.

In Databricks Unity Gateway, a model service can hold up to 5 weighted destinations. For each request, a weighted random draw picks the primary destination. When a client sends a session header, session affinity pins the whole session to one destination. That keeps prefix caching working, because the same long system prompt and tool schemas hit the same backend, and it keeps results consistent within a conversation. If the primary returns a 429 or a 5xx, the gateway tries the fallback destinations in their configured order and does not re-apply the split. Routing decisions land in system.ai_gateway.usage under routing_information, so you can verify what actually happened.

Rate limits are QPM and TPM at the service level, a default per-user level, and custom levels for users, service principals, or groups. The service limit is a global cap. When a request and a token limit both apply, the more restrictive one wins. MCP services get QPM only. Budgets add hard spend caps that block requests once a budget is exceeded.

The migration plan:

1. Run the domain's golden set offline against the new model. Tool-call formatting and instruction following regress most often.
2. Send 5 percent of traffic to the new model with affinity on, compare online judge scores per destination, and widen to 25 percent and then 100 percent only when the per-domain difference is inside the gate threshold.
3. Configure the old model as the first fallback. A fallback destination has to pass the same eval gate, because a silent failover to an unevaluated model is an untested release.

Size limits from the traffic. If the domain peaks at 600 requests per minute with 7,000 tokens per request, it needs 4.2M TPM. Set the service limit about 30 percent above that, and set per-user limits so that one runaway loop cannot take the shared budget.

The PM decides who may change routing for a domain (the domain owner, with platform review), whether cost-driven routing needs per-task-class eval parity, and the rollback SLO, for example routing reverted within 15 minutes of an alert.

Why this is likely: the listing names a model gateway across providers, and Databricks documents traffic splitting, fallbacks, session affinity, and rate limits in detail.

#### Q4. Estimate the annual model spend for internal agents, then estimate the savings from prompt caching and model routing.

These assumptions are illustrative. State them out loud in the interview.

- 8,000 employees, with 40 percent using agents daily, gives 3,200 daily users.
- 6 agent tasks per user per day gives 19,200 tasks per day.
- 8 model calls per task, each with 6,000 input tokens and 400 output tokens.
- Frontier model at $3 per million input tokens and $15 per million output tokens. 250 working days.

Baseline:

- Input tokens per day = 19,200 x 8 x 6,000 = 921.6M, costing 921.6 x $3 = $2,764.80.
- Output tokens per day = 19,200 x 8 x 400 = 61.44M, costing 61.44 x $15 = $921.60.
- Daily total is $3,686.40, so the annual total is $3,686.40 x 250, about $922K.

Step 1, prompt caching. System prompts, tool schemas, and policy text make up most of the input. Assume 70 percent of input tokens are cache hits billed at 10 percent of the input price.

- The blended input price is 0.3 x $3 + 0.7 x $0.30 = $1.11 per million.
- Input cost per day becomes 921.6 x 1.11 = $1,022.98, and output stays at $921.60.
- The new daily total is $1,944.58.

Step 2, routing. Send 60 percent of calls, such as classification, extraction, and short tool selection, to a small model at $0.30 input and $1.20 output, with the same 70 percent cache rate.

- Frontier input: 0.4 x $1,022.98 = $409.19.
- Small model input: 921.6 x 0.6 x (0.3 x 0.30 + 0.7 x 0.03) = 552.96 x 0.111 = $61.38.
- Frontier output: 61.44 x 0.4 x $15 = $368.64.
- Small model output: 61.44 x 0.6 x $1.20 = $44.24.
- The daily total is $883.45, about $221K a year, which is 76 percent below baseline.

Capacity check: if the peak hour carries 20 percent of daily traffic, input runs at 921.6M x 0.2 / 60, about 3.1M TPM. That number sets the gateway rate limits and provisioned throughput.

As PM, I would watch three things. Output tokens dominate after caching (about $413 of the $883 per day), so cap output length and verbosity per task class. Routing is only allowed for task classes where the small model is within the eval gate threshold. Online judge sampling adds cost too, and Q12 sizes it.

Why this is likely: Staff PM loops usually include a quantitative case, and the listing makes cost ceilings and a model gateway the PM's responsibility.

### Connectors, identity and governance

#### Q5. An HR agent calls a Workday MCP connector. Walk through the OAuth flow end to end, and explain why the agent should not reuse the user's token downstream.

In the MCP authorization spec, a remote MCP server is an OAuth 2.1 resource server and the agent runtime is the OAuth client.

1. The client calls the server without a token. The server returns 401 with a WWW-Authenticate header that points to its Protected Resource Metadata (RFC 9728), which lists the authorization servers.
2. The client runs the authorization code flow with PKCE. It sends the resource parameter (RFC 8707) with the MCP server's canonical URI, so the token's audience is bound to that one server.
3. The MCP server validates the token's audience and scopes on every call.

Token passthrough is forbidden. The MCP server must not forward the client's token to Workday. A token minted for the MCP server has the wrong audience for Workday. Forwarding it also erases the audit boundary and creates a confused deputy, where a server holding broad credentials acts for whichever caller tricks it. Two patterns work instead. The server can perform a token exchange (RFC 8693) to get a Workday token for the same user, with an act claim that names the agent. Or a managed connection can store per-user OAuth grants and mint downstream tokens server-side. Unity Gateway's external MCP "Per-User OAuth" mode works the second way.

On Databricks there are two models:

- On-behalf-of-user. The agent acts as the requesting user, Unity Catalog enforces that user's grants including row filters and column masks, and the token is downscoped to the API scopes the agent declares.
- Automatic auth passthrough. A service principal is created per agent version with least-privilege grants and short-lived M2M tokens. Use this when every user should see the same result.

The decisions the PM makes:

- Default to per-user auth for any system with person-level data, such as Workday, Salesforce, or email. Allow a shared principal only for non-sensitive reference data, with security sign-off.
- Batch jobs have no user present. They run as a dedicated service principal with narrow grants and explicit approval, never with a stored user refresh token.
- Fail closed when the user token is missing, and never persist user tokens.
- Log both identities, the human and the agent, on every call.
- Treat consent UX as a product surface. Ten consent prompts on day one will kill adoption, so pre-approve scopes at the connection level.

Why this is likely: the listing names identity propagation and MCP, and the MCP spec, Databricks OBO docs, and the Unity Gateway external MCP post all describe this flow.

#### Q6. Design the connector SDK that lets a Finance engineering team ship a governed MCP connector to the ERP in under a week.

Users and jobs to be done:

- The domain engineer needs to expose 10 ERP operations to agents without becoming an auth expert.
- The security reviewer needs to approve a connector from a checklist instead of reading code.
- The agent builder needs to discover tools and trust their behavior.

Design:

- Scaffold. A CLI command generates an MCP server template that deploys on Databricks Apps with OAuth built in. It registers in Unity Catalog as an MCP service, so the same GRANT model and audit tables apply.
- Tool contract. Each tool declares input and output schemas, a read or write annotation, a risk tier, and for writes, a required idempotency_key parameter and a dry_run flag. The dry-run response returns a structured diff of what would change.
- Enforcement. The gateway applies QPM rate limits per MCP service and per user, and tool filtering exposes only the approved tools to each group. Tier 3 writes, which change money or person records, require a human approval step.
- Contract tests in CI. Calling a write twice with the same key leaves one record. A dry run changes nothing. A call with no token gets a 401. An out-of-scope user gets a 403. Every call emits an MLflow trace span with both identities.
- Docs and examples. A reference connector for one system of record, plus a test sandbox with synthetic data.

Success metrics:

- Time to first governed connector = p50 days from scaffold to UC registration, with a target of 5 days or fewer.
- Governed call share = tool calls through registered MCP services / all agent tool calls seen at the gateway, with a target of 95 percent or more.
- Contract test pass rate on first submission, which shows whether the docs work.
- Duplicate-write incidents, with a target of zero, and security review cycle time.

Risks:

- Teams wrap a broad "run any SQL" tool to skip the contract. Block generic execute tools in tier 2 and above.
- Some ERP APIs lack native idempotency. For those, the connector keeps a dedupe table keyed by idempotency_key with a TTL.
- Contract strictness slows the first connectors. The scaffold should pass all tests out of the box.

Launch: alpha with Finance and HR (ERP and Workday), then beta with GTM (CRM), with platform engineers pairing on each first connector. Publish the connector catalog in Unity Catalog so teams reuse existing connectors instead of rebuilding them.

Why this is likely: the listing asks for a connector SDK with idempotency and dry-run mode, and Databricks documents MCP hosting on Databricks Apps plus UC-registered MCP services.

#### Q7. Security asks, "Which agents touched employee PII last week, and on whose behalf?" Design the governance model so that question takes minutes, not weeks.

Users and jobs to be done: the CISO and Legal need audit answers, domain teams need to ship without a bespoke review each time, and the platform team needs to enforce policy in one place.

Design. Every AI asset becomes a Unity Catalog securable:

- Agents are registered as UC models.
- Tools are UC functions, MCP services, and HTTP connections.
- Models are model services behind Unity Gateway.

Data carries governed tags such as pii=employee. ABAC GRANT policies keep tagged columns masked unless the caller's group and the agent's tier both allow access. Because agents run with on-behalf-of tokens, every access logs both the human and the agent identity. The question becomes a join of the audit and lineage system tables, filtered on tag = pii and grouped by agent and user. It runs in seconds.

Federation tiers set what each class of agent may touch:

- Tier 1 sandbox: read-only, no tagged data, a hard budget, and a 30-day expiry.
- Tier 2 domain production: named owner, passing eval gate, security checklist, and on-call.
- Tier 3 write or sensitive access: tier 2 requirements plus human approval for irreversible actions and a quarterly access review.

Promotion runs automatically through a pipeline that checks registration, gate results, and grants.

Success metrics:

- Governed coverage = agents with UC registration / distinct agent identities seen in gateway traffic. Target 100 percent for anything touching tagged data.
- Audit answer time: p50 minutes from question to evidence, compared with the weeks it takes today.
- Tier 1 to tier 2 promotion rate and median promotion time. Measure both, because a policy no one can pass is a failure too.
- Policy violations blocked versus violations found after the fact.

Risks:

- Shadow agents call model APIs directly. Make the gateway the only network egress to model providers and MCP servers.
- Tagging debt builds up. Auto-classification suggests tags, and owners confirm them.
- Over-masking breaks legitimate HR agents. Pilot the policies with the HR team before enforcing them.

Launch: run an inventory, publish read-only audit dashboards first so the value is visible, enforce on new agents next, then migrate existing agents domain by domain within a fixed deadline.

Why this is likely: the listing asks for a three-tier federation model with governance gates, and the Databricks "Governing AI agents at scale" post and AI governance docs open with this exact PII question.

### Retrieval and memory

#### Q8. An HR policy agent gives outdated answers 8 percent of the time. How do you split retrieval failure from generation failure, and which metrics do you gate on?

Label 200 failing and passing questions with the gold policy chunk. Then score the two halves separately.

Retrieval metrics:

- Recall@k = relevant chunks in the top k / all relevant chunks.
- Precision@k = relevant chunks in the top k / k.
- MRR = mean over queries of 1 / rank of the first relevant chunk. For three queries with the first hit at rank 1, rank 3, and not found, MRR = (1 + 1/3 + 0) / 3 = 0.44.
- nDCG@k = DCG / ideal DCG, where DCG = sum of (2^rel - 1) / log2(i + 1). It rewards putting the current policy above the superseded one, which binary recall misses.

Databricks' built-in judges map onto the same split:

- RetrievalSufficiency asks whether the context contains the expected facts.
- RetrievalGroundedness asks whether the answer is supported by the context.
- RetrievalRelevance asks whether the retrieved chunks are relevant to the request.

Diagnosis:

- Low sufficiency means retrieval is at fault. The new policy is missing, ranked below the old one, or chunked so that the effective date sits in a different chunk.
- High sufficiency with low groundedness means generation is at fault. The model blends both versions or answers from its own prior knowledge.
- With outdated answers, the usual cause is both versions in the index with near-identical embeddings.

Fixes:

- Add effective_date, status, and supersedes metadata, and pre-filter on status = published with no newer version.
- Use hybrid search for policy IDs and acronyms. Databricks reported that hybrid search reached 0.9 recall with 40 retrieved documents, against 50 for dense retrieval alone.
- Rerank the top 50 candidates down to 5.
- Keep the index fresh with a Delta Sync index and a freshness SLA, for example ingest within 1 hour of publication.

The PM decides:

- The gate: recall@5 of 0.95 or higher on the versioned policy set, and RetrievalGroundedness of 0.9 or higher.
- k versus cost and latency: each extra 1,000-token chunk is paid on every call.
- Display rules: the answer shows the source document and its effective date.
- Ownership: HR content owners own retirement of old documents, because no ranking trick fixes a corpus that still serves both versions as current.

Add every failure to the HR golden set so the eval gate catches regressions.

Why this is likely: the listing requires RAG and evaluation experience plus source traceability, and Databricks docs define these retrieval judges and report hybrid search recall.

#### Q9. The listing calls for temporal memory and a context graph. How would you design agent memory, and how does it interact with context window management?

There are three layers, each solving a different problem.

- Working context is what goes into the next model call. Its limits are the context window and cost. Manage it by trimming tool outputs (return 20 rows plus a count instead of 20,000 rows), summarizing older turns past a token threshold, and loading tool schemas only for the tools relevant to the task.
- Session memory is the ordered record of messages, tool calls, and results for one conversation or run. It supports resume and audit, while retrieval happens in the long-term layer.
- Long-term memory holds distilled facts, preferences, and procedures that last across sessions. Examples: "the user's team defines ARR excluding services," or "the join between these two tables needs a fiscal calendar."

Databricks' agent memory, built on its MemCurrent research, uses this split. Managed sessions record history. Managed memory loads an index at session start and runs hybrid keyword and semantic search during the session. A background pipeline extracts durable facts, removes duplicates, and replaces facts that newer conversations contradict. Everything is stored on Lakebase, Databricks' serverless Postgres. Databricks reports that the design tripled pass rate on internal enterprise tasks with 31 percent fewer tokens. It also reports that replaying full history used about 12 times more input tokens than a well-designed memory at the same accuracy, and scored worse on the longest histories.

Temporal matters because facts change. Store valid_from, valid_to, and source on each memory, so that "who owns account X" returns the current owner and the history stays queryable. The context graph links memories to Unity Catalog entities (tables, accounts, employees) so that retrieval can follow relationships as well as text similarity.

PM decisions:

- Scope: user, team, and org namespaces. An HR agent's user memories are never visible to a GTM agent.
- Retention and deletion: TTL per namespace and a delete-my-memory path. Legal and HR memory are opt-in.
- Writes: background distillation only, because ad hoc writes mid-task accumulate contradictions.
- Traceability: every memory read appears as a trace span, so a bad answer can be traced to the memory that caused it.

Metrics: task pass rate with memory on versus off, input tokens per task, memory precision (useful recalls / recalls), and stale-memory incidents.

Why this is likely: the listing names temporal memory, a context graph, and context window management, and Databricks shipped agent memory on Lakebase with published results.

### Evaluation and quality

#### Q10. Legal says the LLM judge is too lenient. You have 200 answers labeled by both lawyers and the judge. How do you measure agreement and fix the judge?

Build the 2x2 table first. Illustrative counts:

- Both pass: 120. Both fail: 30.
- Judge pass but lawyer fail: 40. Judge fail but lawyer pass: 10.

Work through the agreement math:

- Percent agreement p_o = (120 + 30) / 200 = 0.75. That looks fine and hides the problem.
- The judge passes 160 / 200 = 0.80 of answers, and lawyers pass 130 / 200 = 0.65.
- Chance agreement p_e = 0.80 x 0.65 + 0.20 x 0.35 = 0.52 + 0.07 = 0.59.
- Cohen's kappa = (p_o - p_e) / (1 - p_e) = (0.75 - 0.59) / 0.41 = 0.39, which is only fair agreement.
- The number that matters for Legal is the false pass rate on bad answers: 40 / 70 = 57 percent. The judge waves through more than half of what lawyers reject.

Research on judging the judges finds that percent agreement can stay high while the scores diverge, so report kappa together with the error rate on the class you care about.

The fix:

1. Decompose the judge. Replace "is this a good legal answer" with binary checks: cites the governing clause, states the jurisdiction, includes the required caveat, does not give advice outside policy. A judge applies narrow binary checks far more reliably than holistic scores.
2. Write the rubric with lawyers and add few-shot examples of borderline failures.
3. Align the judge with labels. In MLflow, collect lawyer labels through a labeling session in the Review App, then call judge.align(traces), which uses the MemAlign optimizer by default. Databricks docs say 10 traces is a minimum and 50 to 100 work better, and report 30 to 50 percent better agreement with humans.
4. Keep a held-out set the alignment never saw, and pin the judge model version, because a judge model upgrade is a regression risk too.

Set the bar at kappa of 0.7 or higher and a false pass rate of 10 percent or lower on the held-out set before the judge can block or approve releases. Until then, lawyers review every high-risk answer and the judge only orders the review queue.

Why this is likely: the listing names domain-specific judges for Legal and Finance and LLM-as-judge experience, and MLflow documents judge alignment from expert feedback.

#### Q11. Design the mandatory eval gate in CI/CD for every production agent.

Users and jobs to be done:

- The agent developer needs a fast pass or fail result with a reason.
- The domain owner needs proof that quality held.
- Security needs non-negotiable safety checks.
- The platform team needs one gate that works for every framework.

Design. A pull request triggers mlflow.genai.evaluate against the agent's golden set, which is stored as a versioned Unity Catalog table, and against the current production version on the same inputs. The gate runs three classes of scorers:

- Hard checks with zero tolerance: Safety, PII leakage, and tool-call schema validity. Any failure blocks the release.
- Quality checks: Correctness against expectations, Guidelines, RetrievalGroundedness, and ToolCallCorrectness.
- Domain judges such as Finance or Legal, aligned per Q10.

Set thresholds on the paired difference against production. The math shows why. With n = 300 and a pass rate p = 0.85:

- The standard error for one run is sqrt(0.85 x 0.15 / 300), about 0.021.
- Comparing two independent runs gives sqrt(2) x 0.021, about 0.029, so the 95 percent interval is about plus or minus 5.7 points. That is too noisy to catch a 3-point regression.
- Paired scoring on identical inputs only counts the discordant items. If 6 percent of items flip in each direction, the standard error of the difference is about sqrt(0.12 / 300) = 0.020, a narrower interval for the same n.

Gate rule: block when the lower bound of the 95 percent interval on the paired quality difference is below -2 points, or when any hard check fails. Tier 3 agents get a stricter margin and a bigger golden set.

Success metrics:

- Gate runtime p50 of 15 minutes or less, using parallel scoring and cached production outputs.
- False block rate = blocks overridden and later shown to be safe / all blocks. Target under 5 percent.
- Escape rate = production quality incidents the gate passed / all production quality incidents.
- Golden set freshness = share of items added from production traces in the last 90 days.

Risks: teams overfit to a stale golden set, so refresh it monthly from sampled failures. Judge noise causes flaky blocks, so require pinned judge versions and allow one automatic rerun. Overrides become routine, so log them and send them to the domain owner and the platform team.

Launch: report-only shadow mode for 4 weeks so the thresholds can be calibrated on real pull requests, then blocking for tier 2 and tier 3, with tier 1 sandboxes kept report-only.

Why this is likely: the listing requires mandatory eval gates in CI/CD on golden datasets, and MLflow 3 evaluation is Databricks' own product for this.

#### Q12. How many production traces should online judges score per day to detect a groundedness drop from 90 to 85 percent, and what does it cost?

Use the sample size formula for comparing two proportions at alpha = 0.05 (two-sided) and 80 percent power:

n = (z_alpha/2 + z_beta)^2 x (p1(1 - p1) + p2(1 - p2)) / (p1 - p2)^2

- (1.96 + 0.84)^2 = 7.84.
- p1(1 - p1) + p2(1 - p2) = 0.90 x 0.10 + 0.85 x 0.15 = 0.09 + 0.1275 = 0.2175.
- (p1 - p2)^2 = 0.05^2 = 0.0025.
- n = 7.84 x 0.2175 / 0.0025 = 7.84 x 87 = 682 judged traces per comparison window.

If the agent serves 5,000 traces a day, a daily window needs a sample rate of 682 / 5,000, about 14 percent. An agent with 800 traces a day cannot detect a 5-point drop daily. It needs a weekly window or acceptance of only larger shifts. To detect a 2-point drop (90 to 88), n rises to about 3,800, which is why tight thresholds belong in the offline gate, where you control n, while online monitoring watches for larger shifts.

Cost per judgment, with 3,000 input tokens (trace plus rubric) and 200 output tokens at $3 and $15 per million:

- Input: 3,000 x $3 / 1M = $0.009.
- Output: 200 x $15 / 1M = $0.003.
- Total: $0.012 per judgment, so 682 x $0.012 is about $8.20 per day per scorer per agent, or about $3,000 a year.
- Across 50 agents with 3 sampled LLM scorers each: 50 x 3 x $8.20, about $1,230 per day, or about $450K a year.

That is about twice the optimized inference bill from Q4, so sampling policy is a real budget decision.

MLflow production monitoring works this way. You register a scorer and start it with ScorerSamplingConfig(sample_rate, filter_string). Databricks guidance is 1.0 for critical safety scorers and 0.05 to 0.2 for expensive judges.

PM decisions:

- Run cheap safety classifiers at 100 percent.
- Stratify the LLM judges with filter_string, oversampling write actions and new versions in their first 72 hours.
- Use a cheaper aligned judge model where kappa holds.
- Alert only on a sustained drop across two consecutive windows, which keeps false pages down.

Why this is likely: the listing names online LLM-as-judge scoring, Staff PM loops test quantitative reasoning, and Databricks documents sampling-based production monitoring.

### Developer experience and adoption

#### Q13. Design the self-serve path that gets a standard internal agent from idea to production in under 4 weeks. When should a team use Agent Bricks, and when should it write custom code?

Users and jobs to be done:

- A Legal ops analyst wants a policy Q&A agent with no engineering team.
- A GTM engineer wants an account research agent that calls the CRM and the warehouse.
- A Finance engineer wants a write-path agent with approvals.

A decision tree routes each team to a golden path:

1. Questions over documents go to Agent Bricks Knowledge Assistant, which ingests documents and handles retrieval. It also feeds the eval loop from expert feedback.
2. Questions over governed tables go to a Genie space, which uses the semantic layer instead of raw column names.
3. Workflows that combine documents, tables, and tools go to Supervisor Agent, which orchestrates Genie spaces, Knowledge Assistants, and MCP servers under on-behalf-of auth.
4. Custom control flow, durable writes, or unusual latency budgets get a code template (LangGraph or OpenAI Agents SDK) on the platform runtime, with tracing, the gateway, connectors, and the eval gate already wired in.

Week-by-week target for the code path:

- Week 1: scaffold from a template, sandbox with synthetic data, first trace.
- Week 2: connect governed connectors and seed a golden set from 50 expert-labeled questions.
- Week 3: the eval gate passes in shadow mode and a pilot group of 20 users starts.
- Week 4: security checklist auto-verified, promotion to tier 2, production.

Managed paths should hit week 2.

Success metrics:

- Time to production at p50 and p90 from project creation to first production release, with targets of 4 and 8 weeks.
- Managed path share = production agents on Agent Bricks paths / all production agents. Higher means less code to maintain.
- Graduation rate = managed agents later rewritten as custom / managed agents. A high rate means the managed paths hit a customization ceiling too early.
- Template first-run success rate and developer NPS (percent promoters minus percent detractors).

Risks: managed agents behave like black boxes for debugging, which traces and judges mitigate. Some teams need behavior the managed product cannot express, so the custom path must reuse the same golden set and connectors and switching costs nothing. And with many niche bots, users stop knowing which agent to ask, so route through one supervisor entry point.

Launch: pick 3 teams, one per path, publish their time-to-production numbers, and turn their agents into templates.

Why this is likely: the listing sets the under-4-weeks target and asks for SDK, CLI, sandboxes and templates, and Agent Bricks Knowledge Assistant and Supervisor Agent are Databricks' own managed paths.

#### Q14. Define the metric tree for the platform. What is the north star, and how do you compute it?

North star: weekly successful agent tasks in production on the platform.

successful tasks = sum over production agents of (tasks x success rate)

The success rate per agent comes from explicit acceptance, such as the user applying the output or the approver confirming it, or, where that signal is missing, from an aligned online judge. Counting tasks alone rewards noisy agents. Counting successes rewards useful ones.

Value translation, which is what the CIO funds:

hours saved per week = successful tasks x minutes saved per task x realization rate / 60

For example, 50,000 successful tasks x 12 minutes x 0.7 / 60 = 7,000 hours per week. Measure minutes saved per task class with a timed baseline study. The realization rate discounts for time that does not turn into other work.

Input metrics the platform team can move:

- Adoption: paved-road share = new production agents on the platform / all new production agents, with a target of 80 percent or more. Also track domains with 2 or more production agents.
- Speed: time to production at p50 and p90.
- Quality: share of production deploys that passed the eval gate (target 100 percent), escape rate, and online groundedness per agent.
- Cost: cost per successful task = (model spend + judge spend + compute) / successful tasks. With the Q4 numbers, $883 a day of model spend over about 17,000 successful tasks a day is about $0.05 per task before judge and compute costs.
- Developer experience: developer NPS and tickets per project.

Counter-metrics:

- Incidents per 1,000 write actions, so safety holds as speed rises.
- Overridden gate blocks, so you can see whether teams route around the gate.
- Abandoned tier 1 apps past their expiry, so adoption counts stay honest.

Reporting cadence: weekly operating review on the input metrics, and a monthly executive brief that pairs the north star with hours saved and cost per task, because those two numbers decide future funding.

Why this is likely: the listing asks for measurable success criteria, adoption and developer NPS, and the official PM guide's Bringing Products to Market round tests how success is measured.

### Leadership

#### Q15. Tell me about getting several teams that did not report to you to accept a standard they saw as slowing them down.

Choose a story with a real cost to the other teams. For this role, the strongest analogue is a mandatory gate (security review, eval, API standard) imposed across 4 or more teams. Structure it as situation, the specific objection, what you changed, the result in numbers, and what you would do differently.

What the interviewer is probing, and what a strong answer shows:

- You found the real constraint behind the objection. "Gates slow us down" often means "your gate takes 3 days and fails for reasons we cannot see." Show that you measured it, for example by timing the path or counting blocked releases and the reasons behind them.
- You changed the standard itself as well as the messaging. Examples: report-only mode first, a fast path for low-risk changes, clear failure reasons, an exception process with an owner and an expiry date.
- You built a coalition before escalating. Pick one respected team as the first adopter, publish their before and after numbers, and let peers see the result. Escalate only with a written tradeoff, and frame the escalation as a decision for the executive to make.
- You gave something up. A Staff-level answer names the compromise, such as a delayed enforcement date or a reduced scope, and explains why the compromise was right.

Numbers make the story credible. Examples: gate runtime cut from 2 days to 20 minutes, adoption from 1 team to 6 in a quarter, a measurable incident drop, a false-block rate you tracked.

Close with what changed in the relationship. The test is whether those teams brought you their next problem.

Prepare for follow-ups: what you would do if the most important team still refused, how you handled a peer leader who escalated around you, and what data would have changed your own mind.

Why this is likely: the listing asks for leading initiatives across 4 or more teams without authority, the official guide includes a Product Management Leadership round, and candidate reports describe deep follow-ups on behavioral answers.

## Prep checklist

- [ ] Draw the full request path on one page: user, agent runtime, Unity Gateway (routing, rate limits, budgets), MCP services, Unity Catalog grants, MLflow traces. Explain each hop in two sentences.
- [ ] Rehearse the MCP OAuth flow out loud: 401 and Protected Resource Metadata, PKCE, RFC 8707 resource binding, why token passthrough is forbidden, RFC 8693 token exchange, OBO versus a service principal.
- [ ] Explain checkpointing versus durable execution, and do the 0.995^40 completion math from memory.
- [ ] Compute Cohen's kappa by hand from a 2x2 table, and explain why percent agreement misleads.
- [ ] Compute recall@k, MRR and nDCG for a three-query example, and map them to RetrievalSufficiency and RetrievalGroundedness.
- [ ] Redo the Q4 cost model with your own assumptions, including caching and routing, in under 5 minutes.
- [ ] Practice the two-proportion sample size formula, and translate the result into a judge sampling rate and a dollar cost.
- [ ] Build a small agent with Agent Bricks Knowledge Assistant or an MLflow-traced custom agent, run mlflow.genai.evaluate with two built-in judges, and align one judge with 20 labels.
- [ ] Write a 2-page take-home practice PRD with Critical User Journeys for the eval gate or the connector SDK, with success metric formulas and a launch plan.
- [ ] Prepare 3 platform stories with numbers for the Executive Product Leadership round, plus one influence story with a real compromise.
- [ ] Read the Databricks posts on governing agents with Unity Catalog, agent memory, and Unity Gateway spend controls, and form one opinion on what the internal platform should do differently from the external product.

## Sources

- https://www.databricks.com/sites/default/files/2025-07/product-careers-site-interview-prep-april-2025.pdf
- https://www.databricks.com/company/careers/interview-prep
- https://www.interviewquery.com/guides/databricks-product-manager
- https://igotanoffer.com/en/advice/databricks-product-manager-interview
- https://www.datainterview.com/blog/databricks-product-manager-interview
- https://productinterview.com/companies/databricks/process
- https://www.aced.io/questions?company=databricks&role=pm
- https://www.teamblind.com/post/databricks-product-manager-interview-3zkay3sc
- https://www.databricks.com/blog/introducing-agent-bricks
- https://www.databricks.com/blog/agent-bricks-governed-enterprise-agent-platform
- https://www.databricks.com/blog/agent-bricks-supervisor-agent-now-ga-orchestrate-enterprise-agents
- https://www.databricks.com/blog/announcing-managed-mcp-servers-unity-catalog-and-mosaic-ai-integration
- https://www.databricks.com/blog/ai-gateway-how-connect-agents-external-mcps-securely
- https://www.databricks.com/blog/governing-ai-agents-scale-unity-catalog
- https://www.databricks.com/blog/now-ga-building-permission-aware-databricks-apps-behalf-user-authorization
- https://www.databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway
- https://www.databricks.com/blog/introducing-agent-memory-databricks
- https://www.databricks.com/blog/announcing-hybrid-search-general-availability-mosaic-ai-vector-search
- https://www.databricks.com/blog/what-is-agent-evaluation
- https://docs.databricks.com/aws/en/ai-gateway
- https://docs.databricks.com/aws/en/ai-gateway/configure-traffic-splitting
- https://docs.databricks.com/aws/en/ai-gateway/rate-limits
- https://docs.databricks.com/aws/en/generative-ai/agent-framework/agent-authentication-model-serving
- https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/concepts/judges
- https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/align-judges
- https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/production-monitoring
- https://docs.databricks.com/aws/en/mlflow3/genai/human-feedback/concepts/labeling-sessions
- https://mlflow.org/blog/custom-llm-judges-make-judge
- https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization
- https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices
- https://workos.com/blog/oauth-on-behalf-of-ai-agents
- https://temporal.io/blog/temporal-langgraph-plugin-durable-execution
- https://www.diagrid.io/blog/checkpoints-are-not-durable-execution-why-langgraph-crewai-google-adk-and-others-fall-short-for-production-agent-workflows
- https://arxiv.org/html/2406.12624v1
- https://mbrenndoerfer.com/writing/rag-evaluation-metrics-retrieval-generation
