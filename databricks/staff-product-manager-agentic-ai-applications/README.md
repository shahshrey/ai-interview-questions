# Databricks: Staff Product Manager, Agentic AI Applications

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Databricks publishes a PM-specific interview guide, and it is still linked from the careers interview-prep page. Candidate reports say teams adjust the loop, so treat this as the official frame plus what candidates add. All rounds are virtual on Google Meet unless the recruiter says otherwise.

1. Recruiter screen, about 30 minutes: background, motivation, why Databricks, level. AI: not relevant.
2. Hiring manager interview, about 45 minutes: one or two platform projects in depth. Candidates report hard pushback and repeated "why" follow-ups. AI: no published rule; plan for none.
3. Technical screen or skill evaluation, some loops only, about 45 minutes. The careers FAQ lists it as a possible pre-onsite step. One candidate reported a straightforward Python exercise, another a question on how relational databases work. For this role expect questions on agent runtime, connectors, evals and cost reasoning. AI: no published rule; plan for none.
4. Panel 1, four interviews of about 45 minutes: Building Products, Bringing Products to Market, an Engineering Interview on technical constraints and turning requirements into engineering work, and Product Management Leadership. AI: no published rule; plan for none.
5. Panel 2, Executive Product Leadership, about 45 minutes on your two or three highest-impact contributions. Some candidates describe mid-interview feedback and pushback. AI: no published rule; plan for none.
6. Take-home: Critical User Journeys plus a PRD, written and async. Some loops add a panel review. AI: unknown; ask the recruiter before using AI on the draft.
7. References, then the offer. AI: not relevant.

AI use: Databricks publishes no rule on AI tools in interviews. One forum report describes a rejection over suspected AI use in a live round, so plan to work without AI in every live round, and ask the recruiter before using AI on the take-home. No classic LeetCode round appears in any PM report, so this set has no algorithm questions. The three hands-on questions (Q8 to Q10) mirror the light Python screen and the Engineering Interview.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | You inherit the internal agent platform with 14 stalled agent projects across GTM,... | Design | Panel 1, Building Products | Unknown | Hard |
| [Q2](#q2) | Take-home practice: write three Critical User Journeys and the PRD success metrics... | Build | Take-home | Unknown | Medium |
| [Q3](#q3) | A Finance reconciliation agent runs 40 tool-calling steps over 30 minutes and posts... | Design | Panel 1, Engineering Interview | Unknown | Hard |
| [Q4](#q4) | Estimate the annual model spend for internal agents, then estimate the savings from... | Design | Panel 1, Engineering Interview | Unknown | Medium |
| [Q5](#q5) | You migrate one domain's agents to a new model behind the AI gateway. Design the... | Design | Panel 1, Engineering Interview | Unknown | Hard |
| [Q6](#q6) | An HR agent calls an MCP connector to the HR system of record. Walk through the auth... | Design | Panel 1, Engineering Interview | Unknown | Hard |
| [Q7](#q7) | Security asks, "Which agents touched employee PII last week, and on whose behalf?"... | Design | Panel 1, Building Products | Unknown | Hard |
| [Q8](#q8) | (Technical screen, Python, AI policy unknown, so practice with no AI) This per-run... | Debug | Technical screen | Unknown | Medium |
| [Q9](#q9) | (Engineering Interview, reviewing AI output, AI policy unknown, so practice with no... | Review | Panel 1, Engineering Interview | Unknown | Medium |
| [Q10](#q10) | (Engineering Interview, AI policy unknown, so practice with no AI) The HR policy... | Trace debug | Panel 1, Engineering Interview | Unknown | Medium |
| [Q11](#q11) | The same HR agent, once fixed, still gives outdated answers 8 percent of the time.... | Design | Panel 1, Engineering Interview | Unknown | Hard |
| [Q12](#q12) | Legal says the LLM judge is too lenient. You have 200 answers labeled by both... | Concept plus code | Panel 1, Engineering Interview | Unknown | Medium |
| [Q13](#q13) | Design the mandatory eval gate in CI/CD for every production agent, and size the... | Design | Panel 1, Engineering Interview | Unknown | Hard |
| [Q14](#q14) | The self-serve agent platform is ready. Plan the internal launch to GTM, Finance, HR... | Design | Panel 1, Bringing Products to Market | Unknown | Hard |
| [Q15](#q15) | Tell me about a platform launch that missed its adoption target. The interviewer... | Behavioral | Hiring manager or Product Management Leadership | Unknown | Medium |

### Strategy and the take-home

<a id="q1"></a>

#### Q1. You inherit the internal agent platform with 14 stalled agent projects across GTM, Finance, HR and Legal. How do you sequence the first two quarters? Show your scoring.

Round: Panel 1, Building Products. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Roadmap prioritization.

<details>
<summary>Show answer</summary>

Task: Turn 14 stalled projects into a two-quarter plan with a visible scoring rule.

Approach: Start with an audit. For each stalled project, record the blocking step, how many weeks it has been blocked, and whether the blocker is shared. Shared blockers become platform work. One-off blockers go back to the domain team.

Then score each candidate investment by unblocked value per unit of engineering cost:

score = (projects unblocked x weeks saved per project) / engineering weeks to build

Numbers: An audit like this might produce the following illustrative results.

- Identity-propagating connectors for the CRM and the HR system unblock 9 projects, save about 6 weeks each, and cost 12 engineering weeks. Score = 54 / 12 = 4.5.
- A shared eval harness with a CI gate unblocks 14 projects, saves 2 weeks each, and costs 8 weeks. Score = 28 / 8 = 3.5.
- Durable execution for long-running jobs unblocks 4 projects, saves 5 weeks each, and costs 10 weeks. Score = 20 / 10 = 2.0.
- Long-term memory unblocks 3 projects, saves 3 weeks each, and costs 10 weeks. Score = 0.9.

Alternatives: Sequencing by loudest stakeholder is fast but funds one-off work. Building the full platform before any agent ships is clean but shows no value for two quarters and loses sponsors. The scored, slice-based plan wins because each quarter ends with agents in production.

Quarter 1 ships connectors plus the eval harness as one thin slice for two lighthouse agents: one read-heavy (GTM account research) and one write agent (Finance accruals), so both safety paths get exercised. Quarter 2 adds durable execution and the self-serve SDK. Memory waits until a third domain asks for it.

Two adjustments keep the score honest. Apply a risk multiplier, because preventing a duplicate journal entry is worth more than raw weeks saved. Add a dependency check: durable execution scores lower, but the Finance write agent cannot reach production without idempotent retries, so it moves ahead of anything that does not gate a lighthouse.

Verify: Set exit criteria in numbers. By end of Q1, both lighthouses in production, median idea-to-production under 8 weeks, and every production agent through the eval gate. By end of Q2, median under 4 weeks, which is the listing's target, and at least 60 percent of new agents starting on the platform.

Why this is likely: official PM interview guide (Building Products round tests prioritization and execution) and the listing's call for a roadmap with measurable success criteria. Confidence: high for the round (official guide), medium for this exact prompt.

</details>

<a id="q2"></a>

#### Q2. Take-home practice: write three Critical User Journeys and the PRD success metrics for the self-serve path that gets a standard internal agent to production in under 4 weeks. If AI is allowed, show how you used it.

Round: Take-home. AI policy: Unknown. Type: Build. Difficulty: Medium. Topic: CUJs and PRD metrics.

<details>
<summary>Show answer</summary>

Task: Write the journeys and the metrics a reviewer would hold you to.

Approach: I would write three journeys, each as actor, trigger, steps, and the moment of success.

1. A Legal ops analyst with no engineers wants a policy Q&A agent. She picks the document template, points it at a governed volume, labels 50 questions with the expected answer, and gets a shareable agent that passed the eval gate. Success: a pilot group of 20 lawyers using it inside 2 weeks.
2. A GTM engineer wants an account research agent over the CRM and the warehouse. He scaffolds from the CLI, adds two registered connectors, sees his first trace in the sandbox on day 1, and promotes to production in week 4.
3. A Finance engineer wants a write-path agent. She uses the code template with durable execution, every write tool has dry-run and an idempotency key, and an approver confirms entries above a dollar threshold.

Alternatives: for the path itself, one managed path for everyone (fast but hits a customization ceiling), or code templates only (flexible but too slow for teams without engineers). Offer both, with a decision tree, and make them share the same golden set, connectors and eval gate so switching costs nothing.

Verify: The PRD holds these success metrics.

- Time to production, p50 and p90, from project creation to first production release. Targets 4 and 8 weeks.
- Paved-road share = new production agents on the platform / all new production agents. Target 80 percent.
- Graduation rate = managed agents rewritten as custom code / managed agents. High means the managed path is too narrow.
- Developer NPS = percent promoters minus percent detractors, plus template first-run success rate.
- Counter-metric: incidents per 1,000 write actions, so safety is tracked alongside speed.

If the recruiter allows AI, state it in the doc. My delegation plan: I write the journeys and metrics myself, then prompt the assistant: "Act as a skeptical Finance engineering lead. List the five places this PRD would fail for a write-path agent." I check every claim it makes against the listing and Databricks docs. Here is one wrong output I caught. It asserted that the gateway enforces idempotency for MCP writes, but idempotency lives in the connector contract, so I rejected that line and kept the connector requirement.

Why this is likely: official PM interview guide (take-home of Critical User Journeys plus a PRD) and the listing's under-4-weeks target with SDK, CLI, sandboxes and templates. Confidence: high (official guide names this take-home).

</details>

### Runtime, gateway and cost

<a id="q3"></a>

#### Q3. A Finance reconciliation agent runs 40 tool-calling steps over 30 minutes and posts journal entries. Should the platform offer checkpointing or durable execution, and what must it guarantee?

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Durable execution.

<details>
<summary>Show answer</summary>

Task: Choose the recovery model for a long write-path agent and list what it must guarantee.

Approach: Checkpointing saves state. LangGraph, for example, persists graph state at each superstep under a thread_id. If the process dies, the data survives, but nothing detects the failure or resumes the run. Durable execution, the Temporal model, also owns recovery. The workflow's event history is persisted, each side-effecting call is an activity with its own retry policy, and after a crash a worker replays history, skips completed activities using their recorded results, and continues. Human approval becomes a durable wait that can last days without holding a process.

Numbers: Suppose each step has a 0.5 percent chance of a transient failure such as a model timeout, a 429 or a deploy.

- P(40 steps with no failure) = 0.995^40 = e^(40 x ln 0.995), about 0.818.
- Without durability, about 18 percent of runs restart from step 1, repeating about 20 steps on average.
- With durable execution, a failure costs one retried step.

Alternatives: A third option is short synchronous agents with no long runs. That works for Q&A but not for a 30-minute reconciliation with approvals, so durable execution wins for the write tier while read-only agents keep cheap checkpointing.

Risks: writes are the hard part. Replay plus retry means an activity can execute twice, so every write to a system of record needs an idempotency key, for example hash(run_id, step_id, entity_id), and one activity does exactly one external write. Model outputs are recorded in history and read back on replay, so the resumed run follows the same branch.

The guarantees I would put in the PRD:

- Exactly-once effect for writes: at-least-once execution plus idempotent connectors.
- Durable human-in-the-loop waits for approvals above a dollar threshold.
- Per-run ceilings on cost, wall clock and records modified.
- Checkpoint storage encrypted with the same access controls as the source data.

Verify: track run completion rate, mean steps re-executed per failure, and duplicate-write incidents with a target of zero.

Why this is likely: official PM interview guide (Engineering Interview on technical constraints) and the listing's explicit naming of durable execution, checkpointing and sync versus async tradeoffs. Confidence: medium (official round; the topic comes from the job requirements).

</details>

<a id="q4"></a>

#### Q4. Estimate the annual model spend for internal agents, then estimate the savings from prompt caching and model routing.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Design. Difficulty: Medium. Topic: Cost model.

<details>
<summary>Show answer</summary>

Task: Estimate a year of model spend for internal agents and the savings from caching, then routing.

Approach: Build the bill bottom up, then apply one lever at a time.

Numbers: State these illustrative assumptions out loud.

- 8,000 employees, 40 percent using agents daily: 3,200 users.
- 6 tasks per user per day: 19,200 tasks.
- 8 model calls per task, 6,000 input and 400 output tokens each.
- Frontier model at $3 per million input and $15 per million output tokens. 250 working days.

Baseline:

- Input per day = 19,200 x 8 x 6,000 = 921.6M tokens, costing $2,764.80.
- Output per day = 19,200 x 8 x 400 = 61.44M tokens, costing $921.60.
- Daily total $3,686.40, so about 0.92 million dollars a year.

Step 1, prompt caching. System prompts, tool schemas and policy text dominate input. Assume 70 percent of input tokens are cache hits billed at 10 percent.

- Blended input price = 0.3 x $3 + 0.7 x $0.30 = $1.11 per million.
- Input per day = 921.6 x 1.11 = $1,022.98. Daily total $1,944.58.

Step 2, routing. Send 60 percent of calls (classification, extraction, short tool selection) to a small model at $0.30 input and $1.20 output, same cache rate.

- Frontier input 0.4 x $1,022.98 = $409.19. Small input 552.96 x 0.111 = $61.38.
- Frontier output 61.44 x 0.4 x $15 = $368.64. Small output 61.44 x 0.6 x $1.20 = $44.24.
- Daily total $883.45, about 0.22 million dollars a year, 76 percent below baseline.

Alternatives: Routing first would save less here, because the frontier model still pays full input price. Fine-tuning a small model for everything is cheaper per call but costs eval and maintenance effort for each task class. Caching is nearly free, so it goes first, and routing is allowed only for task classes where the small model passes the eval gate.

Verify: Check capacity. If the peak hour carries 20 percent of daily traffic, input runs at 921.6M x 0.2 / 60, about 3.1M tokens per minute. That sets gateway token limits.

After caching, output is nearly half the bill ($413 of $883), so the next lever is capping output length per task class. Judge sampling adds cost too, which Q13 sizes.

Why this is likely: official PM interview guide (Staff PM rounds expect quantitative reasoning) and the listing, which puts cost ceilings and a model gateway in the PM's scope. Confidence: medium (official round; estimation format inferred from Staff PM expectations).

</details>

<a id="q5"></a>

#### Q5. You migrate one domain's agents to a new model behind the AI gateway. Design the rollout: traffic splitting, fallbacks, rate limits and rollback.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Model gateway rollout.

<details>
<summary>Show answer</summary>

Task: Move one domain to a new model with a capped blast radius and a fast rollback.

Approach: Databricks AI Gateway lets a model service hold up to 5 weighted destinations. A weighted random draw picks the primary destination per request, and session affinity pins a whole session to one destination when the client sends a session header. Affinity keeps prefix caching working and keeps answers consistent within a conversation. If the primary returns a 429 or a 5xx, the gateway tries fallbacks in configured order. Routing decisions land in the usage system table, so you can verify what actually happened.

Alternatives: There are three rollout options.

- Big-bang switch. Cheapest to run, but any regression hits every user at once.
- Shadow traffic. Safest, but doubles model cost for the period and cannot measure user acceptance.
- Weighted canary with affinity. Chosen, because it measures real outcomes at a capped blast radius.

The plan:

1. Run the domain's golden set offline against the new model. Tool-call formatting and instruction following regress most often.
2. Send 5 percent of traffic with affinity on, compare online judge scores per destination, then widen to 25 and 100 percent only when the per-domain difference is inside the gate threshold.
3. Configure the old model as the first fallback. A fallback must pass the same eval gate, because silent failover to an unevaluated model is an untested release.

Numbers: size limits from traffic. If the domain peaks at 600 requests per minute at 7,000 tokens each, it needs 4.2M tokens per minute. Set the service limit about 30 percent above that, and set per-user limits so one looping agent stays inside its own share of the budget. When a request limit and a token limit both apply, the more restrictive wins.

The decisions that are mine as PM: who may change routing for a domain (the domain owner with platform review), whether cost-driven routing requires eval parity per task class, and the rollback objective, for example routing reverted within 15 minutes of an alert. Risks: at 5 percent, a domain that serves only 2,000 sessions a day sends 100 canary sessions a day, too few to detect a 5-point drop, so the canary stage lasts days.

Verify: from the usage system table, compare judge score per destination, fallback rate (alert above 1 percent of requests) and time from alert to reverted routing (target under 15 minutes, rehearsed once before the canary starts).

Why this is likely: official PM interview guide (Engineering Interview) and the listing's model gateway across providers, which Databricks documents with traffic splitting, fallbacks and rate limits. Confidence: medium (official round; topic from the job requirements and Databricks docs).

</details>

### Connectors, identity and governance

<a id="q6"></a>

#### Q6. An HR agent calls an MCP connector to the HR system of record. Walk through the auth flow end to end, and decide between per-user and shared-principal identity.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: MCP auth and identity.

<details>
<summary>Show answer</summary>

Task: Trace one HR tool call from 401 to audited response, and pick the identity model for the connector.

Approach: In the MCP authorization spec, a remote MCP server is an OAuth 2.1 resource server and the agent runtime is the OAuth client.

1. The client calls without a token. The server returns 401 with a WWW-Authenticate header pointing to its Protected Resource Metadata (RFC 9728), which lists the authorization servers.
2. The client runs the authorization code flow with PKCE and sends the resource parameter (RFC 8707) set to the MCP server's URI, so the token's audience is bound to that server.
3. The server validates audience and scopes on every call.

Risks: token passthrough is forbidden, so the MCP server keeps the client's token to itself. Forwarding it to the HR system gives the wrong audience, removes the audit boundary, and creates a confused deputy. Two patterns work instead. The server performs a token exchange (RFC 8693) for a downstream token for the same user, with an act claim naming the agent. Or a managed connection stores per-user OAuth grants and mints downstream tokens server-side, which is how the gateway's external MCP Per-User OAuth mode works.

Alternatives: Databricks offers these identity options.

- On-behalf-of-user. The agent acts as the requester, Unity Catalog enforces that user's grants including row filters and column masks, and the token is downscoped to the scopes the agent declares. Chosen for HR, because two employees asking the same question must see different data.
- Automatic passthrough with a service principal per agent version, short-lived machine tokens and least-privilege grants. Right when every user should see the same result, such as a public policy handbook.
- A long-lived shared API key. Rejected, because it has no per-user audit and the largest blast radius.

The PM decisions:

- Per-user auth is the default for any system with person-level data. A shared principal needs security sign-off and non-sensitive data.
- Batch jobs with no user present run as a dedicated service principal with narrow grants, never a stored user refresh token.
- Fail closed when the user token is missing, and hold user tokens in memory only.
- Log both identities, human and agent, on every call.
- Treat consent as product surface. Ten consent prompts on day one kill adoption, so pre-approve scopes at the connection level.

Verify: measure consent completion rate, share of tool calls with dual identity logged (target 100 percent), and auth-related support tickets per new connector.

Why this is likely: official PM interview guide (Engineering Interview) and the listing's identity propagation and MCP requirements, with the MCP spec and Databricks on-behalf-of docs describing this flow. Confidence: medium (official round; topic from the job requirements and the MCP spec).

</details>

<a id="q7"></a>

#### Q7. Security asks, "Which agents touched employee PII last week, and on whose behalf?" Design the governance model so that question takes minutes, not weeks.

Round: Panel 1, Building Products. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Agent governance.

<details>
<summary>Show answer</summary>

Task: the CISO and Legal need audit answers, domain teams need to ship without bespoke reviews, and the platform team needs one enforcement point.

Approach: every AI asset becomes a Unity Catalog securable. Agents are registered models, tools are UC functions, MCP services and connections, and models are services behind the AI gateway. Data carries governed tags such as pii=employee. Attribute-based policies keep tagged columns masked unless both the caller's group and the agent's tier allow access. Because agents run with on-behalf-of tokens, every access logs the human and the agent. The audit question becomes a join of the audit and lineage system tables filtered on the pii tag and grouped by agent and user. It runs in seconds.

Alternatives:

- Per-agent manual security review. Thorough, but at 2 weeks per review and 50 agents a year it costs about 100 reviewer-weeks and still cannot answer a cross-agent question.
- Network-level logging only. Cheap, but it sees only the calls, without the columns or whose identity, so it cannot answer the question.
- Catalog-native governance. Chosen, because the answer is a query and enforcement happens at the same layer as the data.

Federation tiers:

- Tier 1 sandbox: read-only, no tagged data, hard budget, 30-day expiry.
- Tier 2 domain production: named owner, eval gate passed, security checklist, on-call.
- Tier 3 write or sensitive access: tier 2 plus human approval for irreversible actions and a quarterly access review.

Promotion runs through a pipeline that checks registration, gate results and grants automatically.

Metrics:

- Governed coverage = agents registered in Unity Catalog / distinct agent identities seen in gateway traffic. Target 100 percent for anything touching tagged data.
- Audit answer time, p50 minutes from question to evidence.
- Tier promotion rate and median promotion time, because a policy nobody can pass is also a failure.

Risks: shadow agents calling model APIs directly, so make the gateway the only egress to model providers and MCP servers; tagging debt, so auto-classification suggests tags and owners confirm; over-masking that breaks legitimate HR agents, so pilot with HR before enforcing. Launch read-only audit dashboards first so the value is visible, enforce on new agents next, then migrate existing agents by a fixed deadline.

Why this is likely: official PM interview guide (Building Products round) and the listing's three-tier federation with governance gates; Databricks' post on governing AI agents with Unity Catalog covers PII audit for agents. Confidence: medium.

</details>

### Hands-on technical screen

<a id="q8"></a>

#### Q8. (Technical screen, Python, AI policy unknown, so practice with no AI) This per-run cost ceiling helper fails both tests. Find the bugs, fix them, and say whether a post-call check is the right design.

Round: Technical screen. AI policy: Unknown. Type: Debug. Difficulty: Medium. Topic: Cost ceilings.

<details>
<summary>Show answer</summary>

```python
PRICES = {"frontier": (3.00, 15.00), "small": (0.30, 1.20)}  # USD per 1M tokens (in, out)
CACHE_DISCOUNT = 0.1  # cached input billed at 10% of the input price

def call_cost(model, tokens_in, tokens_out, cached_in=0):
    p_in, p_out = PRICES[model]
    billable_in = tokens_in - cached_in + cached_in * CACHE_DISCOUNT
    return (billable_in * p_in + tokens_out * p_out) / 1000

def enforce_ceiling(calls, ceiling_usd):
    """Return (allowed_calls, spent). Never exceed the per-run ceiling."""
    spent, allowed = 0.0, []
    for c in calls:
        cost = call_cost(**c)
        if spent > ceiling_usd:
            break
        spent += cost
        allowed.append(c)
    return allowed, spent

# test_ceiling.py
def test_call_cost():
    assert abs(call_cost("frontier", 1_000_000, 0) - 3.00) < 1e-9

def test_ceiling_stops_before_overrun():
    calls = [{"model": "frontier", "tokens_in": 1_000_000, "tokens_out": 0}] * 3
    allowed, spent = enforce_ceiling(calls, ceiling_usd=5.00)
    assert len(allowed) == 1 and spent <= 5.00
```

Task: Make both tests pass, find any untested defect, and judge the design.

Approach: Read the failures before touching code. test_call_cost reports 3000 instead of 3: prices are per million tokens, but the function divides by 1,000. That is a units bug, and it overstates cost a thousandfold, which in production would block every run.

The second failure is ordering. The check compares spent before adding the current call, so a run at $4.99 with a ceiling of $5 is allowed one more call of any size. The ceiling must test spent + cost. A third defect has no test yet: cached_in larger than tokens_in produces a negative cost, which silently raises the effective ceiling. Reject it.

```python
PRICES = {"frontier": (3.00, 15.00), "small": (0.30, 1.20)}
CACHE_DISCOUNT = 0.1

def call_cost(model, tokens_in, tokens_out, cached_in=0):
    if not 0 <= cached_in <= tokens_in:
        raise ValueError("cached_in must be between 0 and tokens_in")
    p_in, p_out = PRICES[model]
    billable_in = tokens_in - cached_in + cached_in * CACHE_DISCOUNT
    return (billable_in * p_in + tokens_out * p_out) / 1_000_000

def enforce_ceiling(calls, ceiling_usd):
    spent, allowed = 0.0, []
    for c in calls:
        cost = call_cost(**c)
        if spent + cost > ceiling_usd:
            break
        spent += cost
        allowed.append(c)
    return allowed, spent

def test_exact_ceiling_is_allowed():
    calls = [{"model": "small", "tokens_in": 1_000_000, "tokens_out": 0}] * 2
    allowed, spent = enforce_ceiling(calls, ceiling_usd=0.60)
    assert len(allowed) == 2

def test_bad_cache_count_rejected():
    try:
        call_cost("small", 100, 0, cached_in=200)
        assert False
    except ValueError:
        pass

def test_call_cost():
    assert abs(call_cost("frontier", 1_000_000, 0) - 3.00) < 1e-9

def test_ceiling_stops_before_overrun():
    calls = [{"model": "frontier", "tokens_in": 1_000_000, "tokens_out": 0}] * 3
    allowed, spent = enforce_ceiling(calls, ceiling_usd=5.00)
    assert len(allowed) == 1 and spent <= 5.00

for test in (test_call_cost, test_ceiling_stops_before_overrun,
             test_exact_ceiling_is_allowed, test_bad_cache_count_rejected):
    test()
print("Q8 ok")
```

Verify: all four tests pass (the two originals plus the boundary and bad-input cases).

Pitfalls: fixing only the units bug, testing just under the ceiling instead of exactly at it, and summing money as floats.

Alternatives: the design question matters more for a PM. This function computes cost after the fact from actual tokens, but output tokens are unknown before the call. Two alternatives: post-call accounting, which is exact but can overshoot by one call, or pre-call reservation of tokens_in plus max_tokens at output price, released when the call returns. Reservation wins for a hard ceiling. A 4,000-token max output on the frontier model reserves $0.06, so overshoot is bounded. Post-call accounting is fine for soft budgets and dashboards. Production code should also store money as integer micro-dollars or Decimal, because float sums drift at the boundary.

Why this is likely: candidate report of a straightforward Python exercise in the Databricks PM loop (Interview Query) and the listing's per-agent cost ceilings. Confidence: low to medium (one candidate report of a light Python exercise).

</details>

<a id="q9"></a>

#### Q9. (Engineering Interview, reviewing AI output, AI policy unknown, so practice with no AI) An assistant generated this MCP tool for the Finance connector. Review it like a pull request: list the defects, decide whether to merge, and write the tests that would have caught them.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Review. Difficulty: Medium. Topic: AI-generated connector code.

<details>
<summary>Show answer</summary>

```python
import logging
import requests

ERP_URL = "https://erp.internal/api/v2"
log = logging.getLogger("erp_connector")

def post_journal_entries(entries, user_token, dry_run=False):
    """MCP tool: post a batch of journal entries to the ERP for the calling user."""
    log.info("posting %d entries with token %s", len(entries), user_token)
    results = []
    for i in range(1, len(entries)):
        entry = entries[i]
        if dry_run:
            results.append({"would_post": entry})
        resp = requests.post(
            f"{ERP_URL}/journal",
            json=entry,
            headers={"Authorization": f"Bearer {user_token}"},
            retries=3,
        )
        results.append(resp.json())
    return results
```

Task: Review the generated tool as a pull request, decide on the merge, and write the tests.

Approach: Block the merge. The defects, most severe first:

1. Security: the user's token is written to logs, and the same token is forwarded to the ERP. That is token passthrough, which the MCP spec forbids. The tool should receive a client that already holds a downstream token from token exchange.
2. Dry run posts anyway. There is no continue, so a preview creates real entries.
3. Hallucinated API: requests.post has no retries argument, so every call raises TypeError. Retries belong in a session adapter or the platform client.
4. Off-by-one: range(1, len(entries)) skips the first entry.
5. Missing edge cases: no idempotency key, so any retry duplicates a posting, and no check that debits equal credits.

Alternatives: patch each line, or move transport, auth and retries into a shared platform client and keep the tool to validation and business logic. The second wins, because 30 connectors then share one implementation of auth and retries.

```python
import hashlib
import json
import logging
from decimal import Decimal

log = logging.getLogger("erp_connector")

def idempotency_key(run_id, entry):
    body = json.dumps(entry, sort_keys=True)
    return hashlib.sha256(f"{run_id}:{body}".encode()).hexdigest()

def validate(entry):
    debits = sum(Decimal(l["debit"]) for l in entry["lines"])
    credits = sum(Decimal(l["credit"]) for l in entry["lines"])
    if not entry["lines"] or debits != credits:
        raise ValueError(f"entry {entry.get('ref')} is unbalanced")

def post_journal_entries(entries, run_id, send, dry_run=True):
    for e in entries:
        validate(e)
    log.info("run %s: %d entries, dry_run=%s", run_id, len(entries), dry_run)
    results = []
    for entry in entries:
        key = idempotency_key(run_id, entry)
        if dry_run:
            results.append({"would_post": entry["ref"], "idempotency_key": key})
            continue
        results.append(send("/journal", entry, key))
    return results

class FakeERP:
    def __init__(self):
        self.posted = {}
    def send(self, path, body, key):
        self.posted.setdefault(key, body)
        return {"status": "ok", "key": key}

def entry(ref, amt="100.00"):
    return {"ref": ref, "lines": [{"debit": amt, "credit": "0"}, {"debit": "0", "credit": amt}]}

def test_posts_every_entry_including_first():
    erp = FakeERP()
    post_journal_entries([entry("A"), entry("B")], "run-1", erp.send, dry_run=False)
    assert len(erp.posted) == 2

def test_dry_run_posts_nothing():
    erp = FakeERP()
    out = post_journal_entries([entry("A")], "run-1", erp.send)
    assert erp.posted == {} and out[0]["would_post"] == "A"

def test_retry_is_idempotent():
    erp = FakeERP()
    for _ in range(2):
        post_journal_entries([entry("A")], "run-1", erp.send, dry_run=False)
    assert len(erp.posted) == 1

def test_unbalanced_entry_rejected():
    bad = {"ref": "C", "lines": [{"debit": "100.00", "credit": "0"}]}
    try:
        post_journal_entries([bad], "run-1", FakeERP().send, dry_run=False)
        assert False
    except ValueError:
        pass

for test in (test_posts_every_entry_including_first, test_dry_run_posts_nothing,
             test_retry_is_idempotent, test_unbalanced_entry_rejected):
    test()
print("Q9 ok")
```

Verify: all four tests pass on the fix. Ported to the original's transport, the first three would fail: it skips entry A, posts during dry run, and raises TypeError on the retries argument before any idempotency could apply. Dry run now defaults to True, so a write needs an explicit opt-in. As PM, I would turn these tests into the connector SDK's contract suite so every new connector runs them in CI.

Why this is likely: official PM interview guide (Engineering Interview on turning requirements into engineering work) and the listing's connector requirements of identity propagation, idempotency and dry-run mode. Confidence: medium (official round; review-of-AI-code format is our design for the Engineering Interview).

</details>

### Agents, retrieval and evaluation

<a id="q10"></a>

#### Q10. (Engineering Interview, AI policy unknown, so practice with no AI) The HR policy agent answers some questions with a confident but unsupported number. Here is one trace. Find the root cause, propose the fix, and design the eval that proves it.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Trace debug. Difficulty: Medium. Topic: Agent trace.

<details>
<summary>Show answer</summary>

```json
{"trace_id": "tr-7f3a", "agent": "hr-policy-assistant", "user": "alice@corp", "spans": [
  {"step": 1, "type": "llm", "output": "call search_policies(query='parental leave Ontario')"},
  {"step": 2, "type": "tool", "name": "search_policies", "status": "OK", "result": {"chunks": []}},
  {"step": 3, "type": "llm", "output": "no results; call get_employee(user='alice@corp') to refine region"},
  {"step": 4, "type": "tool", "name": "get_employee", "status": "OK", "result": {"region": "CA-ON"}},
  {"step": 5, "type": "llm", "output": "call search_policies(query='parental leave CA-ON')"},
  {"step": 6, "type": "tool", "name": "search_policies", "status": "OK", "result": {"chunks": []},
   "connector_log": "403 FORBIDDEN index hr_policies_v2: principal agent-sp-hr lacks SELECT"},
  {"step": 7, "type": "llm", "output": "no results; call get_employee(user='alice@corp')"},
  {"step": 8, "type": "tool", "name": "get_employee", "status": "OK", "result": {"region": "CA-ON"}},
  {"step": 9, "type": "llm", "output": "call search_policies(query='leave policy CA-ON latest')"},
  {"step": 10, "type": "tool", "name": "search_policies", "status": "OK", "result": {"chunks": []}},
  {"step": 11, "type": "stop", "reason": "max_steps=10 exceeded", "answer": "Parental leave is 12 weeks."}
]}
```

Task: Find why the agent invented a number, fix it, and prove the fix with an eval.

Approach: The root cause is that the search tool hits a 403 because the agent's service principal lacks SELECT on the new index, but the connector swallows the error and returns status OK with zero chunks. The model reads "no results", rewrites the query, re-fetches the employee record, loops until the step limit, and then answers from its own prior knowledge. Three failures stack: a permissions gap after an index migration, an error disguised as an empty result, and a stop path that still emits an answer.

Alternatives: Here are the fixes side by side.

- Return a structured error (status PERMISSION_DENIED, retryable false) from the connector. This is the primary fix. Prompting the model to "be careful with empty results" is the alternative, and it fails because the model cannot tell an empty index from a denied one.
- On step-limit exit, return a refusal with a handoff to a person.
- A loop guard that stops after the same tool returns the same result twice.

A quick detector to size the problem across stored traces:

```python
import json

def find_loops(trace, min_repeats=2):
    """Flag tool calls that repeat with the same result, the signature of a swallowed error."""
    seen, flags = {}, []
    for s in trace["spans"]:
        if s["type"] != "tool":
            continue
        sig = (s["name"], json.dumps(s["result"], sort_keys=True))
        seen[sig] = seen.get(sig, 0) + 1
        if seen[sig] == min_repeats:
            flags.append(sig[0])
    return flags

tool_spans = [("search_policies", {"chunks": []}), ("get_employee", {"region": "CA-ON"}),
              ("search_policies", {"chunks": []}), ("get_employee", {"region": "CA-ON"}),
              ("search_policies", {"chunks": []})]
trace = {"spans": [{"type": "tool", "name": n, "result": r} for n, r in tool_spans]}
assert find_loops(trace) == ["search_policies", "get_employee"]
assert find_loops({"spans": [{"type": "tool", "name": "a", "result": {"x": 1}}]}) == []
print("Q10 ok")
```

Verify: a failure-class set of 60 cases (permission denied, true empty index, tool timeout, stale index), each labeled with the expected behavior, refuse or answer with a citation. Pass criteria: zero uncited numeric answers on failure cases, and resolution rate on the normal golden set within 2 points of baseline. Score groundedness with an LLM judge, then check the judge against 50 human labels before trusting it.

Why this is likely: two research rows match it: "Explain RAG and where retrieval fails" (a candidate-reported Databricks PM question on aced.io) and "Working with engineering on technical constraints" (the Engineering Interview in the official PM guide). Confidence: medium (the trace format follows Databricks MLflow tracing).

</details>

<a id="q11"></a>

#### Q11. The same HR agent, once fixed, still gives outdated answers 8 percent of the time. How do you split retrieval failure from generation failure, and which metrics do you gate on?

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: RAG eval.

<details>
<summary>Show answer</summary>

Task: Separate retrieval failures from generation failures and pick the gate metrics.

Approach: Label 200 failing and passing questions with the gold policy chunk, then score the two halves separately.

Retrieval metrics:

- Recall@k = relevant chunks in the top k / all relevant chunks.
- MRR = mean of 1 / rank of the first relevant chunk. For three queries with first hits at rank 1, rank 3 and not found, MRR = (1 + 1/3 + 0) / 3 = 0.44.
- nDCG@k rewards ranking the current policy above the superseded one, which binary recall misses.

Databricks' built-in judges map onto the split. RetrievalSufficiency asks whether the context contains the expected facts. RetrievalGroundedness asks whether the answer is supported by the context. RetrievalRelevance asks whether the chunks are relevant.

Diagnosis:

- Low sufficiency means retrieval is at fault: the new policy is missing, ranked below the old one, or chunked so the effective date sits in another chunk.
- High sufficiency with low groundedness means generation is at fault: the model blends versions or answers from prior knowledge.
- With outdated answers, the usual cause is both versions in the index with near-identical embeddings.

Alternatives: Here are the fixes side by side.

- Bigger k. Cheap to try, but every extra 1,000-token chunk is paid on every call, and more stale chunks can make blending worse.
- Metadata filtering on effective_date, status and supersedes, pre-filtered to the current published version. Chosen as the primary fix, because it removes the stale version rather than outranking it.
- Hybrid search plus a reranker for policy IDs and acronyms. Databricks reported hybrid search reaching 0.9 recall with 40 retrieved documents against 50 for dense retrieval alone. Add it second.

Keep the index fresh with a Delta Sync index and an SLA, for example ingest within 1 hour of publication.

The PM decisions:

- The gate: recall@5 of 0.95 or higher on the versioned policy set, and RetrievalGroundedness of 0.9 or higher.
- Display rule: every answer shows the source document and its effective date.
- Ownership: HR content owners retire old documents, because ranking helps only once the corpus marks a single version as current.

Verify: every failure joins the HR golden set so the eval gate catches regressions.

Why this is likely: a candidate-reported Databricks PM question on explaining RAG (aced.io) and the listing's RAG, evaluation and source traceability requirements; Databricks docs define these judges. Confidence: medium (one aggregated candidate-reported question plus job requirements).

</details>

<a id="q12"></a>

#### Q12. Legal says the LLM judge is too lenient. You have 200 answers labeled by both lawyers and the judge. Measure agreement and fix the judge.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: LLM judge alignment.

<details>
<summary>Show answer</summary>

Task: Quantify how far the judge is from the lawyers, then fix it.

Approach: Build the 2x2 table first. Illustrative counts: both pass 120, both fail 30, judge pass but lawyer fail 40, judge fail but lawyer pass 10.

```python
def kappa(both_pass, both_fail, judge_pass_human_fail, judge_fail_human_pass):
    n = both_pass + both_fail + judge_pass_human_fail + judge_fail_human_pass
    p_o = (both_pass + both_fail) / n
    judge_pass = (both_pass + judge_pass_human_fail) / n
    human_pass = (both_pass + judge_fail_human_pass) / n
    p_e = judge_pass * human_pass + (1 - judge_pass) * (1 - human_pass)
    false_pass = judge_pass_human_fail / (both_fail + judge_pass_human_fail)
    return round(p_o, 3), round(p_e, 3), round((p_o - p_e) / (1 - p_e), 3), round(false_pass, 3)

assert kappa(120, 30, 40, 10) == (0.75, 0.59, 0.39, 0.571)
assert kappa(50, 50, 0, 0)[2] == 1.0
```

Numbers: Work the math by hand.

- Percent agreement = 150 / 200 = 0.75. That looks fine and hides the problem.
- The judge passes 0.80 of answers, lawyers 0.65. Chance agreement = 0.80 x 0.65 + 0.20 x 0.35 = 0.59.
- Cohen's kappa = (0.75 - 0.59) / 0.41 = 0.39, only fair agreement.
- The number Legal cares about is the false pass rate on bad answers: 40 / 70 = 57 percent. The judge waves through more than half of what lawyers reject.

Research on judging the judges finds percent agreement can stay high while scores diverge, so report kappa with the error rate on the class you care about.

Alternatives:

- Swap in a bigger judge model. Easy, but it adds cost to every judgment and does not encode the lawyers' standard.
- Have lawyers review everything. Accurate, but it does not scale past a few hundred answers a week.
- Decompose and align. Chosen. Replace "is this a good legal answer" with binary checks: cites the governing clause, states the jurisdiction, includes the required caveat, stays inside policy. Narrow binary checks are far more reliable than holistic scores. Then collect lawyer labels in an MLflow labeling session and call judge.align(traces). Databricks docs say 10 traces is the minimum, 50 to 100 work better, and report 30 to 50 percent better agreement with humans.

Verify: keep a held-out set the alignment never saw, and pin the judge model version, because a judge upgrade is a regression risk too. The bar before the judge can block releases: kappa of 0.7 or higher and a false pass rate of 10 percent or lower on the held-out set. Until then, lawyers review every high-risk answer and the judge only orders the queue.

Why this is likely: the listing's domain-specific judges for Legal and LLM-as-judge experience; MLflow documents judge alignment from expert feedback. Confidence: medium (job requirements and MLflow docs).

</details>

<a id="q13"></a>

#### Q13. Design the mandatory eval gate in CI/CD for every production agent, and size the online judge sampling that backs it up.

Round: Panel 1, Engineering Interview. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Eval gates.

<details>
<summary>Show answer</summary>

Task: the developer needs a fast pass or fail with a reason, the domain owner needs proof quality held, security needs non-negotiable checks, and the platform team needs one gate for every framework.

Approach: a pull request runs mlflow.genai.evaluate against the agent's golden set, stored as a versioned Unity Catalog table, and against the current production version on the same inputs. Three scorer classes: hard checks with zero tolerance (safety, PII leakage, tool-call schema validity), quality checks (correctness, guidelines, groundedness, tool-call correctness), and aligned domain judges from Q12.

Alternatives: Compare these gate designs.

- Absolute threshold, for example pass rate of 85 percent or higher. Simple, but noisy and blind to regressions above the bar.
- Human sign-off per release. Accurate, but slow and inconsistent across teams.
- Paired comparison against production. Chosen, because it isolates the change.

```python
from math import ceil, sqrt

def n_per_window(p1, p2, z_a=1.96, z_b=0.84):
    return ceil((z_a + z_b) ** 2 * (p1 * (1 - p1) + p2 * (1 - p2)) / (p1 - p2) ** 2)

def paired_se(discordant_rate, n):
    return sqrt(discordant_rate / n)

assert n_per_window(0.90, 0.85) == 683
assert round(sqrt(2 * 0.85 * 0.15 / 300), 4) == 0.0292
assert round(paired_se(0.12, 300), 4) == 0.02
```

Numbers: with 300 golden items at an 85 percent pass rate, two independent runs give a standard error of the difference of 0.029, an interval of about plus or minus 5.7 points, too noisy to catch a 3-point regression. Paired scoring only counts items that flip; with 12 percent discordant, the standard error drops to 0.020. Gate rule: block when the lower bound of the 95 percent interval on the paired difference is below minus 2 points, or any hard check fails.

Online monitoring catches what the golden set misses. Detecting a groundedness drop from 90 to 85 percent at 80 percent power needs 683 judged traces per window. At 5,000 traces a day that is a 14 percent sample. At $0.012 per judgment (3,000 input and 200 output tokens), that is about $8.20 per day per scorer per agent; 50 agents with 3 scorers cost about $1,230 a day, more than the optimized inference bill in Q4. So run cheap safety classifiers at 100 percent and expensive judges at 5 to 20 percent, stratified toward write actions and new versions.

Verify: gate runtime p50 of 15 minutes or less, false block rate under 5 percent, and escape rate. Launch in report-only mode for 4 weeks to calibrate thresholds, then block for tiers 2 and 3.

Why this is likely: the listing's mandatory eval gates in CI/CD and online LLM-as-judge scoring; Databricks documents MLflow evaluation and sampling-based production monitoring. Confidence: medium (job requirements and Databricks docs).

</details>

### Launch and adoption

<a id="q14"></a>

#### Q14. The self-serve agent platform is ready. Plan the internal launch to GTM, Finance, HR and Legal: sequencing, enablement, adoption funnel and the metrics you review weekly.

Round: Panel 1, Bringing Products to Market. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Internal platform go-to-market.

<details>
<summary>Show answer</summary>

Task: Turn a working platform into one that domain teams choose, with a launch sequence and a funnel you can manage week by week.

Approach: Treat the internal launch like a product launch with a segmented market. Segment teams by readiness: teams with a stalled agent and an engineer (ready now), teams with an idea and no builder, and teams already running a custom stack. Launch to the first group, use their production agents as proof for the second, and offer the third a migration path with a date they agree to.

Alternatives: three launch models.

- Big-bang to all 40 teams. Fast awareness, but onboarding at about 6 hours a week per team is 240 platform hours a week, roughly 6 engineers the platform does not have, so early teams would wait days in the support queue.
- Mandate that every new agent uses the platform. The adoption number looks good at once, but developer NPS falls and custom-stack teams quietly keep building on their own stacks.
- Lighthouse-first, then waves. Chosen. Two lighthouse agents (one Finance, one HR) go to production with the platform team embedded, then waves of 8 teams get templates, office hours and a sandbox on day one.

Numbers: a worked funnel for the first two quarters.

- Aware (attended a demo or read the launch brief): 40 teams.
- First agent running in a sandbox: 24 teams (60 percent).
- First agent through the eval gate into production: 12 teams (50 percent of sandbox).
- Median idea to production: under 8 weeks in Q1, under 4 weeks by end of Q2, the listing's target.

The biggest drop is sandbox to production, so that is where enablement goes: a golden-set template per domain and a pre-reviewed connector for each system of record.

Verify: the weekly review holds the funnel conversion per stage and per domain, developer NPS (target +30 at end of Q2, surveyed after each team's first production deploy), paved-road share (at least 60 percent of new agents built on the platform), and support tickets per active team. A stage that converts under half for two weeks gets a named fix and an owner.

Pitfalls: count adoption from production agents through the eval gate, open Legal's wave once its domain judge exists, and announce wave dates after both lighthouses are in production.

Why this is likely: official PM interview guide (Bringing Products to Market round) and the listing's adoption and developer NPS goals for the self-serve developer experience. Confidence: medium (official round; this exact prompt is inferred from the listing).

</details>

### Leadership

<a id="q15"></a>

#### Q15. Tell me about a platform launch that missed its adoption target. The interviewer will ask "why" at least five times.

Round: Hiring manager or Product Management Leadership. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: Ownership.

<details>
<summary>Show answer</summary>

Task: Tell one real adoption miss and survive five rounds of "why".

Approach: Choose a real miss where you owned part of the cause. For this role, the best analogue is an internal platform, SDK or gate that teams did not adopt. Give the target and the actual number, the first explanation, then each deeper "why", what you changed, the result, and what you would do differently.

A worked shape, with numbers you replace with your own:

- Target: 8 teams on the new SDK in a quarter. Actual: 2.
- Why? Teams said the SDK was missing features.
- Why did that block them? Their agents needed the CRM connector, which was not in the SDK.
- Why was it missing? We sequenced by our own architecture and skipped the audit of what blocked teams.
- Why no audit? I assumed the request backlog reflected demand. It reflected only who complained.
- Why did I assume that? I had no direct channel to the domain engineers, only their managers.

The fix that follows: interview engineers directly, re-sequence by shared blockers (Q1's scoring), and ship one team's connector with them as co-owners. Result to quote: adoption from 2 to 7 teams the next quarter, and time to first agent from 9 weeks to 4.

Verify: The interviewer is testing these things.

- Ownership. The chain should end at your own decision.
- Influence without authority. Show how you got 4 or more teams to change plans: a respected first adopter, published before and after numbers, a written tradeoff before any escalation.
- A real compromise. Name what you gave up, such as a delayed deprecation or a reduced scope, and why.
- Coachability. Executive panels give feedback mid-answer. Take it, adjust, and say what you would change.

Pitfalls: stopping at a cause outside your control, and a story with no numbers.

Follow-ups: what you would do if the most important team still refused, and what data would have changed your mind earlier.

Why this is likely: a Databricks PM leader's public post on using "5 Whys" to test ownership, the official guide's Product Management Leadership and Executive rounds, and candidate reports of repeated "why" follow-ups. Confidence: high (Databricks PM leader's public post and the official guide).

</details>

## Prep checklist

- [ ] Draw the full request path on one page: user, agent runtime, AI gateway (routing, rate limits, budgets), MCP services, Unity Catalog grants, MLflow traces. Explain each hop in two sentences.
- [ ] Rehearse the MCP auth flow aloud: 401 and Protected Resource Metadata, PKCE, resource binding, why token passthrough is forbidden, token exchange, on-behalf-of versus service principal.
- [ ] Do the 0.995^40 durable execution math and the Q4 cost model with your own assumptions in under 5 minutes.
- [ ] Practice light Python live with no AI: read a failing test, fix a units or boundary bug, and add two edge-case tests.
- [ ] Review three pieces of AI-generated connector code for hallucinated APIs, off-by-one errors, missing idempotency and token leaks.
- [ ] Read five agent traces and name the root cause of each in one sentence.
- [ ] Compute Cohen's kappa, recall@k, MRR, and the two-proportion sample size by hand.
- [ ] Build a small MLflow-traced agent, run mlflow.genai.evaluate with two built-in judges, and align one judge with 20 labels.
- [ ] Write a 2-page practice PRD with Critical User Journeys for the eval gate or connector SDK, and ask the recruiter whether AI may be used on the real take-home.
- [ ] Prepare 3 platform stories with numbers for the executive round, plus one adoption miss you can take through five "whys".

## Sources

- https://www.databricks.com/sites/default/files/2025-07/product-careers-site-interview-prep-april-2025.pdf
- https://www.databricks.com/company/careers/interview-prep
- https://www.glassdoor.com/Interview/Databricks-Product-Manager-Interview-Questions-EI_IE954734.0,10_KO11,26.htm
- https://www.interviewquery.com/guides/databricks-product-manager
- https://www.datainterview.com/blog/databricks-product-manager-interview
- https://igotanoffer.com/en/advice/databricks-product-manager-interview
- https://www.teamblind.com/post/databricks-pm-interview-moyr7mwy
- https://productinterview.com/companies/databricks/process
- https://www.aced.io/questions?company=databricks&role=pm
- https://www.databricks.com/blog/agent-bricks-governed-enterprise-agent-platform
- https://www.databricks.com/blog/announcing-managed-mcp-servers-unity-catalog-and-mosaic-ai-integration
- https://www.databricks.com/blog/ai-gateway-how-connect-agents-external-mcps-securely
- https://www.databricks.com/blog/governing-ai-agents-scale-unity-catalog
- https://www.databricks.com/blog/now-ga-building-permission-aware-databricks-apps-behalf-user-authorization
- https://www.databricks.com/blog/introducing-ai-spend-controls-unity-ai-gateway
- https://www.databricks.com/blog/introducing-agent-memory-databricks
- https://www.databricks.com/blog/announcing-hybrid-search-general-availability-mosaic-ai-vector-search
- https://www.databricks.com/blog/what-is-agent-evaluation
- https://docs.databricks.com/aws/en/ai-gateway/configure-traffic-splitting
- https://docs.databricks.com/aws/en/ai-gateway/rate-limits
- https://docs.databricks.com/aws/en/generative-ai/agent-framework/agent-authentication-model-serving
- https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/concepts/judges
- https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/align-judges
- https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/production-monitoring
- https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization
- https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices
- https://temporal.io/blog/temporal-langgraph-plugin-durable-execution
- https://www.diagrid.io/blog/checkpoints-are-not-durable-execution-why-langgraph-crewai-google-adk-and-others-fall-short-for-production-agent-workflows
- https://arxiv.org/html/2406.12624v1
