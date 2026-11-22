# Netflix: Senior Engineering Manager, Agent Platform, AI Platform

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop combines Netflix's own description of its interview process, an engineering manager prep guide built on candidate reports, a recent first-hand Netflix EM screen report on 1point3acres, and a recent Netflix Staff loop report that shows the current partner and team-specific design rounds. Netflix hiring is team-specific and recruiters brief each round, so confirm the mix with yours.

1. Recruiter screen (about 30 minutes, video). Background, why Netflix, the culture memo, level. Recruiters have a real vote in the decision. AI: not relevant.
2. Hiring manager screen (30 to 60 minutes, video). A recent Netflix EM candidate reports the hiring manager skipped behavioral questions and went straight into a system design problem. Others report a talk about past scope and how you lead. AI: unknown, no report of AI use.
3. Technical screen (45 to 60 minutes, varies by team). A take-home design exercise, a deep technical talk with a senior engineer, or a walkthrough of a system you built and its trade-offs. The listing asks for hands-on depth in agent runtimes, tool use, memory and evaluation. AI: unknown.
4. Onsite system design, team-specific (45 to 60 minutes). A design tied to the team's real work: agent runtime, trajectory capture, evaluation signal, tool and MCP interfaces, identity and permissioning. One recent Staff candidate failed this round for not finishing the end-to-end design in time. AI: unknown, no report of AI use.
5. Onsite architecture deep dive (45 minutes). Walk through a complex system you built, the alternatives and why you chose one, including build, adopt or partner calls. AI: unknown.
6. Onsite leadership rounds (one or two, 45 minutes each). Hiring and growing senior engineers and managers, the keeper test, candid feedback, holding a platform charter. AI: unknown; a conversation round, so AI tools are not relevant.
7. Onsite culture round (45 minutes). The culture memo: selflessness, judgment, candor, context not control. AI: unknown; a conversation round, so AI tools are not relevant.
8. Partner rounds (two or three, 45 minutes each). A director, a partner engineer or manager from another org. Here that likely means AI Platform peers, infrastructure and security. AI: unknown; a conversation round, so AI tools are not relevant.
9. Final conversation with the hiring manager or director, then a decision. Netflix often gives detailed feedback.

No classic algorithm round is expected: the evidence for Netflix EM loops points to system design, depth and leadership. Netflix runs AI-assisted coding rounds on some engineering teams, but no EM report mentions one, so the coding-flavored questions below are labelled AI policy Unknown. Most questions are design, worked math and technical depth, as the role calls for.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Design the agent runtime and execution environment that every agent at Netflix runs on... | Design | Hiring manager screen or onsite system design | Unknown | Hard |
| [Q2](#q2) | Design trajectory capture: store each agent run's full reasoning and tool calls as... | Design | Onsite system design | Unknown | Hard |
| [Q3](#q3) | Design the tool and MCP layer: registry, discovery, agent identity and permissions... | Design | Onsite system design | Unknown | Hard |
| [Q4](#q4) | Turn real-world agent outcomes into evaluation signal and gate platform changes on it... | Design | Onsite system design | Unknown | Hard |
| [Q5](#q5) | How many eval tasks do you need to detect a 3 point drop in agent task success, and... | Concept plus code | Technical screen | Unknown | Medium |
| [Q6](#q6) | This agent trajectory burned 40 times its normal cost. Find the root cause from the trace... | Trace debug | Technical screen | Unknown | Medium |
| [Q7](#q7) | An AI assistant wrote this tool executor for the agent runtime. Review it before it ships... | Review | Technical screen | Unknown | Medium |
| [Q8](#q8) | Teams say the cost dashboard shows them under budget while the invoice is much higher... | Debug | Technical screen | Unknown | Medium |
| [Q9](#q9) | Estimate the monthly LLM spend of agent workloads and design cost governance for it... | Concept plus code | Onsite system design | Unknown | Medium |
| [Q10](#q10) | How many sandboxes does the runtime need at peak, and how big is the warm pool? | Concept plus code | Onsite system design | Unknown | Medium |
| [Q11](#q11) | An internal agent reads a document that tells it to send data to an outside address... | Design | Onsite system design | Unknown | Hard |
| [Q12](#q12) | Build, adopt or partner for agent memory and eval tooling on a multi-year cost horizon... | Design | Onsite architecture deep dive | Unknown | Hard |
| [Q13](#q13) | Twelve teams run their own agent harnesses. Get them onto the paved path and deprecate... | Design | Onsite architecture deep dive | Unknown | Hard |
| [Q14](#q14) | Six teams ask the Agent Platform for six things this quarter. Which do you take and... | Design | Hiring manager screen | Unknown | Medium |
| [Q15](#q15) | A strong senior engineer pushes back on the platform direction and the team is splitting... | Behavioral | Onsite leadership | Unknown | Medium |
| [Q16](#q16) | Security wants to block agents from writing to production systems. Build shared... | Behavioral | Partner round | Unknown | Medium |
| [Q17](#q17) | Your team ships much of its code with coding agents. How do you use AI yourself and set... | Behavioral | Onsite leadership | Unknown | Medium |
| [Q18](#q18) | Which part of the Netflix culture memo resonates least with you, and how would you run... | Behavioral | Onsite culture round | Unknown | Medium |
| [Q19](#q19) | You inherit an 8 person team: one engineer is below the bar, and you need to hire two... | Behavioral | Onsite leadership | Unknown | Hard |
| [Q20](#q20) | Walk us through the most complex platform you built or led: the architecture, two... | Design | Onsite architecture deep dive | Unknown | Hard |

### Agent runtime and execution

<a id="q1"></a>

#### Q1. Design the agent runtime and execution environment that every agent at Netflix runs on: how a run starts, executes tool calls, survives failures and stops.

Round: Hiring manager screen or onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: agent runtime, durable execution, sandboxing.

<details>
<summary>Show answer</summary>

Task: Define the run lifecycle, isolation model, failure handling and limits for a shared runtime used by many teams.

What they grade: scoping fast, isolation choices with numbers, failure semantics, finishing an end-to-end design in time.

Approach: Spend the first five minutes on scope. Assume about 50,000 agent runs a day across internal tools, operations and knowledge work, a mean run of 6 minutes and about 12 model calls per run, with long tails. Split the system into a control plane (run API, registry of agent definitions, policy, quotas) and a data plane (workers that execute steps inside sandboxes).

A run is a durable workflow. Each step (model call, tool call, human approval) is written to a log before and after it executes, so a crashed worker resumes from the last completed step instead of replaying side effects. Tool calls carry an idempotency key built from run id and step number so a retry never charges a card or files a ticket twice.

Alternatives: run steps in a plain queue with checkpoints (simple, but you rebuild timers, retries and signals yourself), use a durable workflow engine (Temporal or similar, more operational weight but retries, timers and human waits come free), or run the agent loop inside a long-lived process (fastest, loses everything on deploy). I pick the workflow engine because runs last minutes to hours and include human approval waits.

Isolation: tool code that executes model-written commands runs in a per-run sandbox. A shared process is cheapest but one bad run can read another's data. Containers on the existing container platform are fine for trusted tools. Model-written code gets a microVM or gVisor-style sandbox with no default network egress.

Risks: runaway loops (cap steps, wall clock and tokens per run, enforced by the runtime and not by the agent), noisy neighbors (per-team concurrency quotas), and silent partial failure (every run ends in an explicit terminal state with a reason).

Numbers: 50,000 runs a day at 6 minutes mean is about 210 runs in flight on average and roughly 625 at a 3x peak (see Q10).

Follow-ups: How do you cancel a run mid-tool-call? What changes for streaming user-facing agents with a 2 second first-token budget?

Why this is likely: the listing says the team owns "the runtime and execution environment" every agent depends on, and a recent Netflix EM screen report says the hiring manager went straight to system design (job listing, high; 1point3acres, medium).

</details>

<a id="q2"></a>

#### Q2. Design trajectory capture: store each agent run's full reasoning and tool calls as structured, replayable data that other teams can debug, evaluate and learn from.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: trajectories, observability, replay, privacy.

<details>
<summary>Show answer</summary>

Task: Define the schema, write path, storage, access controls and replay semantics for agent trajectories.

What they grade: a schema that supports replay and eval, storage math, privacy controls, and a clear line between tracing and data products.

Approach: Model a trajectory as an ordered list of spans: run metadata (agent id and version, model, prompt template hash, runtime version), then one span per step with type (model call, tool call, approval), inputs, outputs, token usage, latency, errors and the parent span. Store large payloads (retrieved documents, tool outputs) once by content hash and reference them, because the same context is re-sent on every model call.

Write path: the runtime emits spans asynchronously to a log stream, so capture never blocks a run. A consumer writes hot data to a queryable store for 14 to 30 days and compacted columnar files to the data lake for training and eval.

Replay has two modes. Deterministic replay feeds the recorded model and tool outputs back to reproduce a run exactly, which is what debugging needs. Counterfactual replay re-runs the recorded inputs against a new model or prompt with tool calls stubbed from the record, which is what eval needs. Side-effecting tools are never re-executed in replay.

Numbers: 50,000 runs a day times 12 steps times about 30 KB per step is about 18 GB a day raw. Content hashing and compression at roughly 5x bring that near 3.6 GB a day, about 1.3 TB a year, which is small. Query load drives the cost.

Alternatives: generic tracing (OpenTelemetry spans only) is cheap to adopt but drops payloads and cannot replay. Logging full prompts as text is easy but costly to search and hard to redact. A typed span schema with payload references wins because eval and debugging both need structure.

Risks: personal and confidential data in prompts. Classify payloads at write time, redact or tokenize known sensitive fields, restrict raw payload access by agent owner, and set retention per data class.

Follow-ups: How do you version the schema when the runtime adds a new step type? Who may train on trajectories?

Why this is likely: the listing names "capturing each run's full reasoning-and-tool-call trajectory as structured, replayable data" as a core problem for this team (job listing, high).

</details>

<a id="q3"></a>

#### Q3. Design the tool and MCP layer: a registry where teams publish tools, discovery for agents, and identity and permissions so an agent can only do what its user and owner allow.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: MCP, tool registry, agent identity, authorization.

<details>
<summary>Show answer</summary>

Task: Let hundreds of internal tools be published once and called safely by many agents, with clear identity and audit.

What they grade: the identity model, least privilege, how discovery scales past a context window, and audit.

Approach: Tools are published as MCP servers behind a gateway. The registry stores each tool's schema, owner, risk tier (read, write, destructive), data classes it touches and its scopes. Agents never connect to tool servers directly; the gateway authenticates, authorizes, rate limits and logs every call.

Identity is the core decision. Alternatives: (1) each agent has its own service identity with broad rights, simple but an agent acting for an intern can read what the agent can; (2) the agent acts purely as the user with the user's token, safe on scope but any user permission becomes an agent permission; (3) delegated identity: a token that carries both the user and the agent, and the gateway allows a call only when both the user's rights and the agent's declared scopes allow it. I pick option 3, with short-lived tokens issued per run.

Risk tiers drive control: read tools run freely within scope; write tools need the agent to declare them up front; destructive tools need a human approval step that the runtime enforces.

Discovery: an agent cannot load 500 tool schemas into context. Each agent declares a tool set; for open-ended agents, the registry serves a search tool that returns the top 5 to 10 relevant tools for the current step.

Numbers: if 500 tools average 400 tokens of schema, loading all of them costs 200,000 tokens per call, which is why retrieval of tools beats static loading at that size.

Verify: every call in the audit log shows user, agent, tool, scope decision and trajectory span id, so security can answer "who did this" in one query.

Pitfalls: tool descriptions written by publishers can carry prompt injection; review and lint them at publish time. Token passthrough to downstream services leaks rights; mint downstream tokens per call instead.

Why this is likely: the listing lists "identity and permissioning, discovery and registry" and "the tool/MCP interfaces agents use to act" as team scope, and Netflix's tech blog describes internal agent tools exposed through MCP (job listing, high; Netflix tech blog, high).

</details>

### Evaluation

<a id="q4"></a>

#### Q4. Turn real-world agent outcomes into evaluation signal, then use it to gate platform changes such as a model upgrade or a new runtime version.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: agent evaluation, online signals, regression gates.

<details>
<summary>Show answer</summary>

Task: Define what outcome signals the platform collects, how they become eval sets, and how a change is blocked when quality drops.

What they grade: signal quality, judge calibration, gating logic with statistics, and who owns which eval.

Approach: Separate three layers. Platform evals check the runtime itself: tool call success, schema validity, latency, cost, loop rate. Agent evals belong to each agent team: task success on their own golden set. Outcome signals come from production: user accepted or edited the result, the ticket the agent filed was reopened, the pull request was merged or reverted, a human approver rejected the action.

Outcome signals are noisy and delayed, so they feed the eval set instead of being the gate. Each week, sample trajectories with clear outcomes, have humans confirm a slice, and add them to a versioned eval set with the recorded tool outputs. That gives counterfactual replay (Q2) on real tasks without side effects.

Gate: for a model upgrade, replay the eval set on old and new versions with the same tasks. Use a paired test, because the same tasks run on both arms; this needs far fewer tasks than independent samples (Q5). Block the rollout when task success drops more than a set margin, cost per successful task rises more than 15 percent, or any safety eval regresses at all.

LLM judges: use them for open-ended outputs, but calibrate each judge prompt against a few hundred human labels and track agreement (Cohen's kappa). A common rule of thumb: below about 0.6, the judge is not reliable enough to gate on.

Alternatives: gating only on offline golden sets is fast but drifts from real use. Gating only on online A/B tests is realistic but slow and risky for write actions. Offline replay first, then a small canary watched on outcome signals, wins.

Risks: teams overfit to the eval set; keep a held-out slice the agent teams cannot see. Judge drift when the judge model changes; pin judge versions.

Follow-ups: Who owns an agent's eval when the platform changes the model under it?

Why this is likely: the listing names "evaluation" and "turning real-world outcomes into evaluation signal" as team scope, and Netflix speakers describe evaluation frameworks as central to their internal AI platform (job listing, high; Anthropic webinar with Netflix AI Platform speakers, medium).

</details>

<a id="q5"></a>

#### Q5. How many eval tasks do you need to detect a 3 point drop in agent task success from an 80 percent baseline, and how do you know your LLM judge can be trusted?

Round: Technical screen. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: eval statistics, judge agreement.

<details>
<summary>Show answer</summary>

Task: Size the eval set for a platform gate and check judge reliability with numbers.

What they grade: correct power math, knowing why paired designs are cheaper, and a concrete judge calibration method.

Approach: With independent samples, a two-proportion test at 5 percent significance and 80 percent power needs about 2,943 tasks per arm to detect 80 to 77 percent. That is expensive when each task is a multi-step agent run.

Run the same tasks on both versions instead. Then only the tasks where the versions disagree carry information (McNemar's test). If about 10 percent of tasks flip between versions and the net drop is 3 points, about 870 tasks are enough, roughly a third of the unpaired count.

For the judge, label a sample by hand and compute Cohen's kappa against the judge. In the toy sample below the judge agrees on 8 of 10 but kappa is only 0.52 because both raters say "pass" most of the time, so raw agreement flatters the judge.

Code:

```python
from statistics import NormalDist


def n_per_arm(p1, p2, alpha=0.05, power=0.8):
    """Unpaired two-proportion sample size per arm (normal approximation)."""
    z_a = NormalDist().inv_cdf(1 - alpha / 2)
    z_b = NormalDist().inv_cdf(power)
    pbar = (p1 + p2) / 2
    num = z_a * (2 * pbar * (1 - pbar)) ** 0.5 + z_b * (p1 * (1 - p1) + p2 * (1 - p2)) ** 0.5
    return int(num ** 2 / (p1 - p2) ** 2) + 1


def n_paired(p_discordant, diff, alpha=0.05, power=0.8):
    """McNemar sample size: same tasks run on old and new platform version."""
    z_a = NormalDist().inv_cdf(1 - alpha / 2)
    z_b = NormalDist().inv_cdf(power)
    num = z_a * p_discordant ** 0.5 + z_b * (p_discordant - diff ** 2) ** 0.5
    return int(num ** 2 / diff ** 2) + 1


def cohen_kappa(a, b):
    """Agreement between LLM judge (a) and human labels (b), binary 0/1."""
    n = len(a)
    po = sum(x == y for x, y in zip(a, b)) / n
    pa, pb = sum(a) / n, sum(b) / n
    pe = pa * pb + (1 - pa) * (1 - pb)
    return (po - pe) / (1 - pe)


if __name__ == "__main__":
    unpaired = n_per_arm(0.80, 0.77)
    paired = n_paired(0.10, 0.03)
    print("unpaired per arm:", unpaired, "paired tasks:", paired)
    assert 2900 < unpaired < 3000
    assert paired < unpaired / 3
    judge = [1, 1, 0, 1, 0, 1, 1, 0, 1, 1]
    human = [1, 1, 0, 1, 1, 1, 0, 0, 1, 1]
    k = cohen_kappa(judge, human)
    print("kappa:", round(k, 3))
    assert 0.5 < k < 0.6
    assert cohen_kappa(human, human) == 1.0
    print("ok")
```

Verify: the script prints 2,943 unpaired and 870 paired tasks and a kappa of 0.524. Check the discordant rate on a pilot run before trusting 870; if 20 percent of tasks flip, the paired count roughly doubles.

Pitfalls: agent runs are stochastic, so run each task more than once or fix sampling settings; treating raw agreement as judge quality; testing many metrics at once without correcting for it.

Why this is likely: evaluation depth is an explicit requirement in the listing, and Netflix's EM technical screens go deep on the candidate's own technical decisions (job listing, high; EM prep guide built on candidate reports, medium).

</details>

<a id="q6"></a>

#### Q6. This agent trajectory burned 40 times its normal cost before the runtime killed it. Find the root cause from the trace and say what the platform should change.

Round: Technical screen. AI policy: Unknown. Type: Trace debug. Difficulty: Medium. Topic: agent loops, tool contracts, runtime limits.

```text
run=r-7731 agent=ops-triage v14 model=model-a budget_steps=60
step 01 model   in=3,100   out=180 -> call search_tickets(q="playback errors", cursor=null)
step 02 tool    search_tickets ok 2.1s results=50 next_cursor="p2" truncated=true
step 03 model   in=9,400   out=210 -> call search_tickets(q="playback errors", cursor=null)
step 04 tool    search_tickets ok 2.0s results=50 next_cursor="p2" truncated=true
step 05 model   in=15,700  out=205 -> call search_tickets(q="playback errors", cursor=null)
step 06 tool    search_tickets ok 2.2s results=50 next_cursor="p2" truncated=true
step 07 model   in=22,000  out=220 -> call search_tickets(q="playback errors", cursor=null)
...
step 57 model   in=179,500 out=230 -> call search_tickets(q="playback errors", cursor=null)
step 58 tool    search_tickets ok 2.1s results=50 next_cursor="p2" truncated=true
step 59 model   in=185,800 out=190 -> call summarize(...)
step 60 runtime KILLED reason=step_budget tokens_in=2.8M cost_ratio=40x
tool schema (registry): search_tickets(q: str, cursor: str | null) -> {results, next_cursor}
tool description: "Search tickets. Returns results."
```

<details>
<summary>Show answer</summary>

Task: Explain why the agent made the same call 29 times and what to fix in the agent, the tool and the platform.

What they grade: reading the trace for evidence, separating the agent bug from the platform gap, and fixes that prevent the whole class of failure.

Approach: The agent calls search_tickets with cursor=null every time, so it gets page one again and again. The tool reports truncated=true and next_cursor="p2", but the tool description never says how to paginate, so the model keeps asking the same question hoping for more. Each retry appends 50 results to the context, so input grows by about 6,300 tokens per round, from 3,100 to about 186,000 per call, and 2.8 million in total. That growth, not the step count, is why cost hit 40x: the cost of a loop grows with the square of its length when context accumulates.

Root cause: a weak tool contract (no pagination guidance), plus a runtime that only stops on a step budget of 60 rather than on repeated identical calls or a token budget.

Fixes, by owner. Tool owner: describe pagination in the schema and the description, and cap results per page at a size that fits. Platform: detect identical consecutive calls and stop or nudge the agent after 3; add a per-run token budget next to the step budget; summarize or drop old tool outputs from context. Registry: lint tool descriptions at publish time for missing pagination and error semantics.

Code:

```python
import hashlib
import json


def step_key(step):
    args = json.dumps(step["args"], sort_keys=True)
    return step["tool"] + ":" + hashlib.sha256(args.encode()).hexdigest()[:12]


def find_loop(steps, window=3):
    """Return the index where an identical tool call repeats `window` times in a row."""
    run, prev = 0, None
    for i, s in enumerate(steps):
        k = step_key(s)
        run = run + 1 if k == prev else 1
        prev = k
        if run >= window:
            return i
    return None


if __name__ == "__main__":
    page = {"tool": "search_tickets", "args": {"q": "playback errors", "cursor": None}}
    steps = [page, {"tool": "search_tickets", "args": {"q": "playback errors", "cursor": None}},
             {"tool": "search_tickets", "args": {"cursor": None, "q": "playback errors"}}]
    assert find_loop(steps) == 2
    ok = [{"tool": "search_tickets", "args": {"q": "x", "cursor": c}} for c in (None, "p2", "p3")]
    assert find_loop(ok) is None
    print("ok")
```

Verify: replay this trajectory (Q2) with the detector on; it should stop at step 5. Paginated calls with new cursors must not trigger it.

Pitfalls: blaming the model and switching models; raising the step budget; fixing only this tool instead of the registry lint.

Why this is likely: runaway loops and cost are the daily failure mode of a shared agent runtime, and the listing names cost governance and trajectory data as team scope (job listing, high).

</details>

### Code you would review

<a id="q7"></a>

#### Q7. An AI assistant wrote this tool executor for the agent runtime. Review it before it ships: find the defects, rank them and show the fixed version with tests.

Round: Technical screen. AI policy: Unknown. Type: Review. Difficulty: Medium. Topic: AI-written code review, sandbox security, retries.

```text
import asyncio, os, subprocess

MAX_RETRIES = 3

async def run_tool(call, budget):
    # call = {"tool": "read_file", "args": {...}, "agent_id": "..."}
    for attempt in range(1, MAX_RETRIES):
        try:
            async with asyncio.timeout_after(30):
                return await _dispatch(call)
        except Exception:
            await asyncio.sleep(2 ** attempt)
    raise RuntimeError("tool failed")

async def _dispatch(call):
    if call["tool"] == "read_file":
        path = os.path.join("/sandbox", call["args"]["path"])
        return open(path).read()
    if call["tool"] == "shell":
        return subprocess.check_output(call["args"]["cmd"], shell=True)
```

<details>
<summary>Show answer</summary>

Task: Find every defect, rank by severity, fix, and prove the fix with tests.

What they grade: catching the security holes first, spotting the invented API, and tests that pin each fix.

Approach: Ranked findings. (1) Shell injection: shell=True on a model-written command lets any prompt injection run arbitrary commands, and it blocks the event loop. Remove the shell tool or replace it with an allowlisted argv call in a sandbox. (2) Path traversal: os.path.join("/sandbox", "../../etc/passwd") escapes, and an absolute path like "/etc/passwd" replaces the root entirely. Resolve with realpath and check the common path. (3) Hallucinated API: asyncio has no timeout_after (Python 3.11 added asyncio.timeout); use asyncio.wait_for, which also works on older versions. (4) Off-by-one: range(1, MAX_RETRIES) makes 2 attempts, not 3. (5) Retries everything: a permission error is retried with backoff instead of failing fast; retry only transient errors and timeouts. (6) budget and agent_id are unused, so no quota or authorization check happens, and an unknown tool returns None silently.

Code:

```python
import asyncio
import os
import tempfile

MAX_ATTEMPTS = 3


class Transient(Exception):
    pass


class BudgetExceeded(Exception):
    pass


def safe_path(root, user_path):
    root = os.path.realpath(root)
    full = os.path.realpath(os.path.join(root, user_path))
    if os.path.commonpath([root, full]) != root:
        raise PermissionError(f"path escapes sandbox: {user_path}")
    return full


async def dispatch(call, root):
    if call["tool"] == "read_file":
        path = safe_path(root, call["args"]["path"])
        return await asyncio.to_thread(lambda: open(path).read())
    raise ValueError(f"unknown tool: {call['tool']}")


async def run_tool(call, root, budget, cost=1, timeout_s=30, sleep=asyncio.sleep, dispatch=dispatch):
    if call["tool"] not in budget["allowed_tools"]:
        raise PermissionError(f"{call['agent_id']} may not call {call['tool']}")
    if budget["remaining"] < cost:
        raise BudgetExceeded(call["agent_id"])
    budget["remaining"] -= cost
    for attempt in range(1, MAX_ATTEMPTS + 1):
        try:
            return await asyncio.wait_for(dispatch(call, root), timeout_s)
        except (Transient, asyncio.TimeoutError):
            if attempt == MAX_ATTEMPTS:
                raise
            await sleep(min(2 ** attempt, 10))


async def _tests():
    root = tempfile.mkdtemp()
    with open(os.path.join(root, "a.txt"), "w") as f:
        f.write("hello")
    b = {"remaining": 5, "allowed_tools": {"read_file"}}
    call = {"tool": "read_file", "args": {"path": "a.txt"}, "agent_id": "ag1"}
    assert await run_tool(call, root, b) == "hello"
    for bad in ("../../etc/passwd", "/etc/passwd"):
        try:
            await run_tool({**call, "args": {"path": bad}}, root, b)
            raise AssertionError("traversal allowed")
        except PermissionError:
            pass
    tries, sleeps = [], []

    async def flaky(c, r):
        tries.append(1)
        raise Transient()

    async def nosleep(s):
        sleeps.append(s)

    try:
        await run_tool(call, root, b, sleep=nosleep, dispatch=flaky)
    except Transient:
        pass
    assert len(tries) == 3 and sleeps == [2, 4], (tries, sleeps)
    denied = []

    async def deny(c, r):
        denied.append(1)
        raise PermissionError("no")

    try:
        await run_tool(call, root, b, dispatch=deny)
    except PermissionError:
        pass
    assert len(denied) == 1
    try:
        await run_tool({**call, "tool": "shell"}, root, b)
        raise AssertionError("tool not in allowlist ran")
    except PermissionError:
        pass
    b["remaining"] = 0
    try:
        await run_tool(call, root, b)
        raise AssertionError("budget ignored")
    except BudgetExceeded:
        pass
    print("ok")


if __name__ == "__main__":
    asyncio.run(_tests())
```

Verify: traversal with relative and absolute paths is blocked, transient errors get exactly 3 attempts with backoff of 2 then 4 seconds, permission errors are not retried, unlisted tools and empty budgets are refused.

Pitfalls: fixing the timeout and missing the shell injection; trusting the AI's comment that the sandbox path is safe. If you used an assistant for the fix, say what you asked it, which tests you wrote first, and what it got wrong.

Why this is likely: Netflix runs AI-assisted coding rounds on some engineering teams, graded on understanding and extending AI output (Blind candidate and employee reports, medium); no EM report mentions one, and the listing asks for hands-on depth in agent runtimes and tool use (job listing, high).

</details>

<a id="q8"></a>

#### Q8. Teams say the cost dashboard shows them under budget while the invoice is much higher, and some runs overshoot their budget by a wide margin. Here is the meter. Find the bugs and fix them.

Round: Technical screen. AI policy: Unknown. Type: Debug. Difficulty: Medium. Topic: LLM cost metering, prompt caching, concurrency.

```text
# Prices in USD per 1M tokens (illustrative)
PRICES = {"model-a": {"in": 3.0, "out": 15.0}}

class CostMeter:
    def __init__(self, budget_usd):
        self.budget = budget_usd
        self.spent = 0.0

    def allow(self):
        return self.spent < self.budget          # checked before each model call

    def record(self, model, usage):               # called when a response completes
        p = PRICES[model]
        cost = (usage["input_tokens"] * p["in"] + usage["output_tokens"] * p["out"]) / 1_000_000
        self.spent += cost                        # called from 8 worker threads

# sample usage from the provider for one call:
# {"input_tokens": 2000, "cache_read_input_tokens": 40000,
#  "cache_creation_input_tokens": 0, "output_tokens": 500}
```

<details>
<summary>Show answer</summary>

Task: Explain each gap between the meter and the invoice, then fix the meter so budgets hold.

What they grade: reading the provider's usage fields, seeing the budget check race, and a reservation design that keeps budgets hard.

Approach: Four bugs. (1) Cached tokens are ignored. With prompt caching, providers such as Anthropic report cache reads and cache writes in separate fields, and input_tokens excludes them. Agents re-send long context on every call, so most input is cache reads. In the sample, the meter records 0.0135 dollars while the real cost is 0.0255 dollars, so it misses almost half. (2) The budget check only asks whether spend so far is below budget. One large call that starts just under budget can overshoot by its full cost, and 8 threads can all pass the check at once. (3) self.spent += cost from 8 threads is a read-modify-write without a lock, so updates can be lost. (4) record runs only when a response completes, so cancelled or failed streams that were still billed are never counted.

Fix: reserve the worst-case cost of a call before it starts, under a lock, and settle the real cost afterwards in a finally block.

Code:

```python
import threading

# Example prices in USD per 1M tokens (illustrative, not a real price list).
PRICES = {"model-a": {"in": 3.0, "out": 15.0, "cache_write": 3.75, "cache_read": 0.30}}


def usage_cost(model, u):
    p = PRICES[model]
    return (u.get("input_tokens", 0) * p["in"]
            + u.get("cache_creation_input_tokens", 0) * p["cache_write"]
            + u.get("cache_read_input_tokens", 0) * p["cache_read"]
            + u.get("output_tokens", 0) * p["out"]) / 1_000_000


class CostMeter:
    def __init__(self, budget_usd):
        self.budget = budget_usd
        self.spent = 0.0
        self.reserved = 0.0
        self._lock = threading.Lock()

    def reserve(self, model, max_in, max_out):
        p = PRICES[model]
        worst = (max_in * max(p["in"], p["cache_write"]) + max_out * p["out"]) / 1_000_000
        with self._lock:
            if self.spent + self.reserved + worst > self.budget:
                return None
            self.reserved += worst
        return worst

    def settle(self, hold, model, usage):
        """Call in a finally block, also for cancelled or failed streams."""
        cost = usage_cost(model, usage)
        with self._lock:
            self.reserved -= hold
            self.spent += cost
        return cost


if __name__ == "__main__":
    u = {"input_tokens": 2_000, "cache_read_input_tokens": 40_000,
         "cache_creation_input_tokens": 0, "output_tokens": 500}
    full = usage_cost("model-a", u)
    old = (u["input_tokens"] * 3.0 + u["output_tokens"] * 15.0) / 1_000_000
    print(f"full {full:.4f} vs old meter {old:.4f}")
    assert round(full, 4) == 0.0255 and round(old, 4) == 0.0135

    m = CostMeter(budget_usd=1.0)
    h = m.reserve("model-a", 200_000, 8_000)
    assert h is not None and m.reserve("model-a", 200_000, 8_000) is None
    m.settle(h, "model-a", u)
    assert m.reserved == 0 and abs(m.spent - full) < 1e-12

    m2 = CostMeter(budget_usd=10_000)

    def worker():
        for _ in range(2_000):
            hold = m2.reserve("model-a", 1_000, 100)
            m2.settle(hold, "model-a", {"input_tokens": 1_000, "output_tokens": 100})

    ts = [threading.Thread(target=worker) for _ in range(8)]
    [t.start() for t in ts]
    [t.join() for t in ts]
    expected = 16_000 * usage_cost("model-a", {"input_tokens": 1_000, "output_tokens": 100})
    assert abs(m2.spent - expected) < 1e-6 and abs(m2.reserved) < 1e-6
    print("ok")
```

Verify: the sample call now costs 0.0255; a second worst-case reservation is refused at a 1 dollar budget; 16,000 concurrent settles sum exactly. Reconcile the meter against the monthly invoice per team and alert on more than 2 percent drift.

Pitfalls: pricing cache writes at the base input rate; reserving the average cost instead of the worst case.

Why this is likely: cost governance for LLM and agent workloads is a preferred skill in the listing, and a platform team owns the meter every team is charged by (job listing, high).

</details>

### Cost, capacity and safety

<a id="q9"></a>

#### Q9. Estimate the monthly LLM spend of agent workloads on the platform, then design how cost is attributed, budgeted and kept under control.

Round: Onsite system design. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: cost estimation, attribution, quotas, caching.

<details>
<summary>Show answer</summary>

Task: Produce a defensible estimate in a few minutes, find the biggest cost lever, and design governance that teams accept.

What they grade: stated assumptions, finding that input context dominates, and controls sized to the risk.

Approach: Assume 50,000 runs a day, 12 model calls per run, 6,000 input tokens and 400 output tokens per call, and example prices of 3 dollars per million input tokens and 15 per million output. That is 3.6 billion input tokens and 240 million output tokens a day: about 14,400 dollars a day, or about 0.43 million dollars a month. Input is 75 percent of the bill, because agents re-send growing context on every call.

The biggest lever is prompt caching. If 70 percent of input tokens are cache reads at a tenth of the base price, spend drops to about 7,600 dollars a day, about 0.23 million a month. The next levers are routing easy steps to a smaller model and trimming old tool outputs from context (Q6).

Code:

```python
def daily_cost(runs, calls_per_run, in_tok, out_tok, p_in, p_out, cached_share=0.0, cache_mult=0.1):
    tin = runs * calls_per_run * in_tok
    tout = runs * calls_per_run * out_tok
    in_cost = tin * ((1 - cached_share) * p_in + cached_share * p_in * cache_mult) / 1e6
    return in_cost + tout * p_out / 1e6


if __name__ == "__main__":
    base = daily_cost(50_000, 12, 6_000, 400, 3.0, 15.0)
    cached = daily_cost(50_000, 12, 6_000, 400, 3.0, 15.0, cached_share=0.7)
    print(f"no cache {base:,.0f}/day, {base*30/1e6:.2f}M/month")
    print(f"70% cached {cached:,.0f}/day, {cached*30/1e6:.2f}M/month")
    assert round(base) == 14_400 and round(cached) == 7_596
    print("ok")
```

Governance design: every model call carries agent id, owning team and run id, so cost lands on the team that owns the agent. Teams get monthly budgets with soft alerts at 80 percent and per-run hard caps enforced by the runtime (Q8). The platform publishes cost per successful task next to quality, because cost per call rewards agents that fail cheaply.

Alternatives: central chargeback with no caps (teams move fast, surprise bills), hard team quotas only (predictable, but blocks launches), or showback with per-run caps plus a fast exception path. I pick showback with per-run caps.

Pitfalls: treating cache hit rate as fixed; it falls when prompts put changing content before stable content.

Why this is likely: the listing names cost governance for LLM and agent workloads, and Netflix's AI Discover write-up calls LLM calls the slowest and most expensive part of its system (job listing, high; Netflix tech blog, high).

</details>

<a id="q10"></a>

#### Q10. How many sandboxes does the runtime need at peak, and how big should the warm pool be so runs do not wait on cold starts?

Round: Onsite system design. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: capacity planning, Little's law, cold starts.

<details>
<summary>Show answer</summary>

Task: Size sandbox capacity from traffic and run duration, then size the warm pool against cold start time.

What they grade: using Little's law correctly, peak versus average, and the cost trade-off of idle warm capacity.

Approach: Little's law says the number in flight equals arrival rate times time in the system. 50,000 runs a day is about 35 arrivals a minute on average. With a 3x peak that is about 104 a minute. At a mean run of 6 minutes, about 625 runs are in flight at peak. Add 30 percent headroom for variance and failover: provision about 813 sandboxes.

Warm pool: if provisioning a sandbox from cold (scheduling, image pull, microVM boot and tool setup) takes about 30 seconds, while a pre-provisioned warm one is ready in under a second (the microVM boot itself is well under a second), the pool must cover arrivals during one cold start window: 104 a minute times half a minute is about 53 warm sandboxes. The autoscaler refills the pool as runs take sandboxes.

Code:

```python
import math


def sandboxes_needed(runs_per_day, mean_minutes, peak_factor, headroom=0.3):
    """Little's law: L = lambda * W, with lambda at peak."""
    lam_per_min = runs_per_day / (24 * 60) * peak_factor
    in_flight = lam_per_min * mean_minutes
    return math.ceil(in_flight * (1 + headroom)), in_flight


if __name__ == "__main__":
    total, in_flight = sandboxes_needed(50_000, 6, 3)
    print(f"peak in flight {in_flight:.0f}, provision {total}")
    assert round(in_flight) == 625 and total == 813
    warm, _ = sandboxes_needed(50_000, 0.5, 3, headroom=0)
    print("warm pool covering 30s of arrivals:", warm)
    assert warm == 53
    print("ok")
```

Alternatives: one sandbox per run for its full life (simple, but idle while the run waits on a model call, which is most of the time), or a sandbox only for steps that execute code, returned to the pool between steps. The second can cut sandbox count several times over when most of a run is model latency, at the cost of state handling between steps.

Verify: compare the model against real traces: measured in-flight runs at peak should be close to arrival rate times measured duration. If not, the duration distribution has a long tail that the mean hides; size on the 95th percentile for runs that hold human approval waits.

Pitfalls: sizing on the daily average; ignoring that runs waiting for human approval hold a sandbox for hours unless released.

Why this is likely: the listing asks for a strong foundation in distributed systems and cloud infrastructure for the agent runtime, and Netflix design rounds push on scale numbers (job listing, high; EM prep guide, medium).

</details>

<a id="q11"></a>

#### Q11. An internal agent reads a document that tells it to send data to an outside address, and it tries to. Design the guardrails so the platform stops this for every agent, not just this one.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: prompt injection, guardrails, egress control.

<details>
<summary>Show answer</summary>

Task: Layer defenses so a prompt injection in untrusted content cannot turn into data loss or a harmful action.

What they grade: assuming the model will be fooled, controls outside the model, and a sensible split between platform and agent team.

Approach: Treat the model as unable to tell instructions from data. Any agent that has private data, reads untrusted content and can send data out is exposed. The platform's job is to break one of those three in every risky run.

Layers, outermost first. (1) Egress control: sandboxes have no default network; outbound calls go through the tool gateway, which allows only registered destinations. That alone stops the outside address in this case. (2) Tool scopes and risk tiers (Q3): a summarization agent has no send-email or webhook tool at all. (3) Taint tracking: the runtime marks content that came from untrusted sources; when a run has read tainted content, write and send tools need human approval or are blocked for the rest of the run. (4) Input and output classifiers: scan tool outputs for injection patterns and outgoing payloads for secrets and personal data. These catch some attacks and are the weakest layer, so they are never the only control. (5) Audit and replay: every blocked action is logged with its trajectory so security and the agent owner can see the attempt.

Alternatives: rely on system prompt rules telling the model to ignore embedded instructions (cheap, and attacks get past it routinely); classifiers only (better, still probabilistic); deterministic controls in the runtime and gateway plus classifiers. The last wins because the controls that matter still hold when the model follows an injected instruction.

Numbers: if a classifier catches 95 percent of injections and agents read 1 million untrusted documents a month with 1 in 10,000 malicious, about 5 attacks a month get past the classifier, which is why egress and scopes must hold on their own.

Risks: approval fatigue when every action asks a human; scope approvals to tainted runs and write tools only.

Follow-ups: How do you test this? Keep a red-team suite of injection documents in the platform eval set (Q4) and run it on every runtime release.

Why this is likely: the listing names guardrails and safety, agent identity and permissioning as preferred experience, and a platform that runs every agent at the company has to stop this centrally (job listing, high).

</details>

### Platform strategy

<a id="q12"></a>

#### Q12. Decide whether to build, adopt open source, or partner with a vendor for agent memory and for eval tooling, on a multi-year cost horizon.

Round: Onsite architecture deep dive. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: build versus buy, total cost, strategic fit.

<details>
<summary>Show answer</summary>

Task: Make two separate calls with a clear method, and show that you would reach different answers when the facts differ.

What they grade: a repeatable decision method, counting all the costs, exit cost, and tying the decision to what makes Netflix different.

Approach: Use the same four questions for each: Is it core to what makes our agents better than what anyone can buy? How fast is the market moving? What does it cost over three years in engineer-years and spend? How hard is it to leave?

Agent memory: memory sits on top of private context such as user preferences, team knowledge and past runs, and must follow our identity and data rules (Q3). The vendor market is young and moving fast. I would build a thin memory interface owned by the platform and adopt an open-source store under it, so the storage engine can be swapped. Rough cost: 2 engineer-years to build the interface and policy layer, about 1 a year to run it. A full in-house store would add perhaps 4 to 6 engineer-years with little advantage.

Eval tooling: dashboards, judge runners and dataset versioning are not what makes Netflix different, and good vendor and open-source options exist. Partner or adopt, but keep eval sets and trajectories in our own storage so switching tools does not lose data. Rough cost: license spend plus about half an engineer-year to integrate, against 3 or more engineer-years to build a comparable tool.

Alternatives considered for each: full build, adopt and extend, vendor. The split answer, build where it touches identity and private data and buy where it is commodity, holds up best over three years.

Numbers: multiply engineer-years by your loaded cost and compare against three years of license spend, plus the cost to migrate off the vendor if it fails, which teams often forget.

Risks: open-source projects stall; vendors change pricing. Write the exit plan before signing.

Follow-ups: What would make you reverse the memory decision in a year?

Why this is likely: the listing asks the manager to "make sharp build, adopt, or partner calls" on "a multi-year cost horizon", and Netflix EM guides report architecture rounds that probe trade-offs in past decisions (job listing, high; EM prep guide, medium).

</details>

<a id="q13"></a>

#### Q13. Twelve teams run their own agent harnesses. Get them onto the paved path and deprecate the old ones without stalling their roadmaps.

Round: Onsite architecture deep dive. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: platform adoption, migrations, deprecation.

<details>
<summary>Show answer</summary>

Task: Plan a migration where adoption is earned, not mandated, at a company that prizes context over control.

What they grade: making the paved path better than the alternatives, sequencing, measurable adoption, and a real end date for old systems.

Approach: First, learn why each team built its own: missing features, speed, or distrust. Interview the twelve teams and map each harness's features against the platform. Expect a few features the platform lacks; build the two or three most shared ones first.

Make the paved path the obvious choice: one-line onboarding, trajectory capture (Q2), evals (Q4), identity and tool access (Q3) and cost controls for free. Teams on their own harness would have to rebuild all of that, and security reviews get faster on the platform.

Sequence: start with two teams who want to move and have visible agents. Pair platform engineers with them and publish their before-and-after numbers: time to ship a new agent, incidents, cost per task. Then migrate in waves by similarity of harness, with a compatibility adapter so old agent code runs on the new runtime with small changes.

Deprecation: announce a date only after the platform covers the shared features, freeze new features on old harnesses, and give a 2 quarter window. Keep a short list of approved exceptions with an owner and review date.

Numbers: track teams migrated, share of runs on the platform, and median days to onboard a new agent. A target could be 80 percent of runs on the platform in 3 quarters, with time to onboard under 1 day.

Alternatives: mandate from leadership (fast on paper, resentful teams, forks), let the market decide forever (the platform never gets the scale to justify itself), or earned adoption with a firm end date. Earned adoption fits the culture memo's context over control.

Risks: the platform becomes the bottleneck; publish a contribution model so teams can add tools and features themselves.

Follow-ups: A large team refuses to move. What do you do?

Why this is likely: the listing asks for experience running a platform other engineers consume as a product, with paved paths, migrations and deprecations, and the culture memo stresses context over control (job listing, high; Netflix culture memo, high).

</details>

<a id="q14"></a>

#### Q14. Six teams ask the Agent Platform for six things this quarter: a vector database, a no-code agent builder, model serving, a Slack bot, memory, and better evals. Which do you take, and how do you say no?

Round: Hiring manager screen. AI policy: Unknown. Type: Design. Difficulty: Medium. Topic: charter, scope, prioritization.

<details>
<summary>Show answer</summary>

Task: Apply a clear charter to real requests, decide, and keep partners on side.

What they grade: a charter you can state in one sentence, decisions that follow from it, and how you handle the teams you turn down.

Approach: State the charter first: the Agent Platform owns the agent-specific layer every agent depends on (runtime, memory, identity, registry, evaluation, tool interfaces). Anything that is not agent-specific or is a single product belongs elsewhere.

Decisions. Better evals: take it, it is core and every team benefits. Memory: take it, it is in the charter and touches identity and private data (Q12). Vector database: do not build one. Retrieval storage is general infrastructure; partner with the data platform and expose it through the memory interface. Model serving: no. It is general AI infrastructure, not agent-specific, so it belongs with whichever partner team owns inference (ask the interviewer who that is); agree on the contract between them. No-code agent builder: not now; it is a product on top of the platform, and building it before the runtime and evals are solid spreads the team thin. Offer the platform APIs so a product team can build it. Slack bot: no, that is a single agent; help the team build it on the paved path as a reference customer.

Numbers: assuming a team of about 8 engineers, two large items in a quarter is realistic; six would mean none finished.

How to say no: give each team the reason tied to the charter, the owner who should do it, and what the platform gives them in the meantime. Write the decisions down where the leaders can see and challenge them.

Alternatives: say yes to everyone (popular for a quarter, then nothing ships), or decide by who escalates loudest. A written charter plus open trade-offs wins and matches Netflix's candor value.

Risks: the boundary with the inference and data owners stays fuzzy; agree on ownership in writing with those managers.

Follow-ups: Your director asks you to take the agent builder anyway. What now?

Why this is likely: the listing asks for "setting its direction and holding its scope against pressure to do everything", and Netflix EM hiring manager rounds focus on scope and judgment (job listing, high; Blind EM discussion, medium).

</details>

### Leading the team

<a id="q15"></a>

#### Q15. A strong senior engineer pushes back on the platform direction you set and the team is splitting into two camps. What do you do, and when does the keeper test come in?

Round: Onsite leadership. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: technical disagreement, candor, keeper test.

<details>
<summary>Show answer</summary>

Task: Tell a real story, or walk through how you would handle it, showing you can be wrong, decide, and keep the team.

What they grade: whether you listen and change your mind on evidence, whether you still decide, and candor about performance.

Approach: Use a real story with numbers. A good structure: the situation, what the engineer argued, how you tested it, what you decided, and what happened.

Start by assuming they might be right. Senior engineers often see technical risks first. Meet one-on-one and ask for the strongest version of the argument, then make it a written comparison: both options, with cost, risk and reversibility. Ask the engineer to own one side of the document. Bring in a neutral senior engineer from outside the team to review it.

Decide on a date, and say who decided and why. If the evidence favors the engineer, change direction and say so in public; that earns more trust than being right. If not, explain the reasoning, record what result would make you revisit it, and ask the engineer to commit. Disagreeing and then committing is normal; undermining the decision afterwards is not.

The keeper test is a separate question: would you fight to keep this person if they said they were leaving? Disagreement is not a performance problem. Netflix values candor, so an engineer who argues well is often a keeper. It becomes a keeper test question only if behavior hurts the team after the decision, for example blocking reviews or running a shadow project. Then give direct feedback with specific examples, a clear expectation and a short timeline.

Pitfalls: deciding by seniority, letting the split drag on for months, or treating a strong dissenter as a problem to manage out.

Verify: show how you checked the outcome later: whether the chosen path hit its numbers, and whether the engineer stayed engaged.

Why this is likely: Netflix's culture memo centers on candor, judgment and the keeper test, and EM guides report "tell me about a time" and "what would you do if" leadership questions in every round (Netflix culture memo, high; EM prep guide and question bank, medium).

</details>

<a id="q16"></a>

#### Q16. Security wants to block agents from writing to production systems after an incident at another company. Build shared accountability with them instead of a standoff.

Round: Partner round. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: cross-org partnership, risk, shared ownership.

<details>
<summary>Show answer</summary>

Task: Show how you work across org boundaries when ownership is unclear and the other side has a real point.

What they grade: taking the other team's risk seriously, turning a yes or no fight into shared controls, and follow-through.

Approach: Start by agreeing with what is right in their position: agents that write to production can cause real damage, and security will be blamed if they do. Ask for the specific failure they fear: wrong change, data deletion, credential misuse, or prompt injection (Q11).

Turn the blanket block into tiers you design together. Read-only access stays open. Low-risk writes such as filing a ticket or opening a pull request go ahead with audit. Production changes need human approval enforced by the runtime, short-lived scoped credentials (Q3) and a rollback path. Destructive actions stay blocked until both teams sign off on controls.

Make accountability shared and written: security owns the risk tiers and approves new write tools; the Agent Platform owns enforcement in the runtime and gateway; agent teams own their agents' behavior. Put a joint review of every blocked or approved high-risk action on a monthly calendar for the first quarter.

Show results with data: number of write actions, approvals, rejections and incidents. If security sees the controls working, they will widen access; if a gap shows up, you fix it together.

Alternatives: escalate to leadership for an exception (fast, but you lose the partner for years), accept the block (safe, but agents cannot do useful work), or co-design tiered controls. The last is slower to start and lasts.

Pitfalls: arguing that the incident elsewhere does not apply here; promising controls you have not built.

Verify: tell a real story where you built trust with a partner team that could have blocked you, and say what you gave up.

Why this is likely: Netflix's second onsite round includes partner engineers and managers from other orgs, and the listing asks for shared accountability with platform, infrastructure and security teams (Netflix tech blog, high; job listing, high).

</details>

<a id="q17"></a>

#### Q17. Your team ships much of its code with coding agents. How do you use AI yourself, and what review standards do you set so quality and ownership hold?

Round: Onsite leadership. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: AI-assisted engineering, code review, team standards.

<details>
<summary>Show answer</summary>

Task: Show real personal use of AI tools and team standards that keep humans accountable for what ships.

What they grade: hands-on credibility, concrete checks on AI output, and measuring the effect instead of assuming it.

Approach: Start with your own use, with specifics: you use a coding agent to read unfamiliar code, draft design document sections, write tests and prototype; you check its output by running the tests, reading every diff line that touches security or data, and asking it to explain anything you would not have written. Give one real example of a wrong AI output you caught, such as an invented library function or a missing edge case, and what you changed after it.

Team standards: the engineer who opens the pull request owns it, whoever or whatever wrote it. Agent-written changes need tests written or reviewed by a human before the code. High-risk paths such as auth, permissions, billing and data deletion get a second human reviewer. Give agents good context: architecture notes and conventions in the repository, because agents amplify what is already in the codebase.

Measure: change failure rate, revert rate and review time for agent-heavy pull requests compared with others, and cycle time. If reverts rise, tighten review on the paths that cause them instead of banning the tools.

Alternatives: ban AI on core paths (safe but slow, and engineers use it anyway), allow anything with normal review (fast, but reviewers skim large AI diffs), or clear ownership plus targeted checks plus measurement. I pick ownership, targeted checks and measurement.

Pitfalls: claiming large productivity gains without numbers; giving AI work to junior engineers without teaching them to check it.

Verify: be ready for "show me how you reviewed an agent's pull request last week", with a concrete answer.

Why this is likely: Netflix engineers describe retrofitting their software lifecycle for AI coding agents, and the listing says agents are becoming a primary way software gets built at Netflix (QCon talk by a Netflix engineer, high; job listing, high).

</details>

<a id="q18"></a>

#### Q18. Which part of the Netflix culture memo resonates least with you, and how would you run the Agent Platform team on context, not control?

Round: Onsite culture round. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: culture memo, context not control, team operating model.

<details>
<summary>Show answer</summary>

Task: Name one real tension in the memo and explain how it would shape the way you lead this team.

What they grade: an honest pick with a reason, judgment about where the memo is hard to live by, and a concrete operating model.

Approach: Pick one tension you actually feel and explain it with a real example. Two strong choices for this role:

Keeper test versus psychological safety. The keeper test asks whether you would fight to keep each person. Done badly, people stop admitting mistakes because every mistake feels like a test. Your answer: keep the two apart. Blameless incident reviews for system failures; keeper test conversations about sustained patterns, never about one incident. Say what you would tell a new hire about the difference.

Freedom versus a paved path. The memo favors freedom and responsibility, but a platform team wants every agent on one runtime (Q13). Your answer: the paved path has to earn adoption on its merits. Hard rules apply only where risk is shared, such as identity, audit and egress controls (Q3, Q11). Everything else is a default, and a team can leave it if it owns the cost.

For this team, context means a written charter with what the platform owns and does not own (Q14), a short list of guardrails, and the metrics that matter (adoption, run success rate, eval coverage, cost per run). Engineers make the calls inside that context. You review decisions after the fact, and when one goes wrong you add the missing context to the charter or the guardrails.

Pitfalls: saying everything resonates; picking a tension and then dismissing it; criticizing the memo without saying how you would still work inside it.

Verify: give one past example where you replaced an approval step with better context and what happened to speed and quality.

Why this is likely: research shows a dedicated 45 minute culture round on the memo, with "which part resonates least" reported as a common prompt (Netflix culture memo, high; IGotAnOffer EM guide, medium).

</details>

<a id="q19"></a>

#### Q19. You inherit an 8 person team: one engineer is consistently below the bar, and you need to hire two senior engineers and a manager in a hot market. Walk through the next two quarters.

Round: Onsite leadership. AI policy: Unknown. Type: Behavioral. Difficulty: Hard. Topic: low performance, keeper test, hiring senior engineers and managers.

<details>
<summary>Show answer</summary>

Task: Show you can handle a performance problem quickly and fairly while building the team, without one delaying the other.

What they grade: candor and speed on performance, fairness and clear expectations, a high and explicit hiring bar, and how you grow the team you have.

Approach: Weeks 1 to 4: learn before judging. One-on-ones with everyone, read recent design documents and incidents, ask peers how the engineer works. Check whether the gap is skill, scope, motivation or a bad fit with the work; sometimes the previous manager never gave clear expectations.

Low performer: give direct, specific feedback in week 3 or 4 with examples, a written expectation of what good looks like in 6 to 8 weeks, and weekly check-ins. Help with real support, such as a better-fit project or pairing. If the gap remains, apply the keeper test honestly: if you would not fight to keep them, part ways quickly and respectfully; the culture memo says it is 'fairer to everyone to part ways quickly'. Tell the team only what is needed, and protect the person's dignity.

Hiring: write the bar first. For senior engineers: hands-on depth in agent runtimes or distributed systems, a design they led end to end, and judgment about AI output. For the manager: they have hired and coached senior people and led a platform others depend on. Source through the team's own networks, open-source contributors in agent tooling and conference speakers; a hot market rewards personal outreach over job posts. Run a short, consistent loop with a written debrief. Expect one hire per 6 to 8 weeks, so start in month one.

Grow the existing team: give one strong engineer a stretch tech lead role to cover the gap until hires land.

Pitfalls: waiting a quarter to give the first feedback; lowering the bar because hiring is slow; hiring the manager last, so senior engineers report to you for too long.

Verify: name the metrics you would show at the end: hires landed against plan, offer acceptance rate, and whether the team's delivery held.

Why this is likely: leadership rounds cover hiring and growing senior engineers and managers, the keeper test and low performers (Netflix culture memo, high; Prepfully EM bank, low; IGotAnOffer EM guide, medium).

</details>

### Your own systems

<a id="q20"></a>

#### Q20. Walk us through the most complex platform you built or led: the architecture, two alternatives you rejected and why, and what you would change now.

Round: Onsite architecture deep dive. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: past system walkthrough, trade-offs, technical judgment.

<details>
<summary>Show answer</summary>

Task: Prepare one 25 minute walkthrough of a real system that leaves 20 minutes for senior engineers to probe it.

What they grade: whether you understand your own system deeply, whether your trade-offs were reasoned with numbers, and whether you can say what went wrong.

Approach: Use a fixed structure so you finish on time.

Context and scale, 3 minutes: who used it, the load (requests per second, data volume, number of teams), the team size and the constraint that shaped everything, such as latency, cost or a deadline.

Architecture, 7 minutes: draw the main components and one request path end to end. Name the parts you designed yourself versus the parts the team owned.

Alternatives, 5 minutes: at least two you rejected, each with its cost. For example, a managed queue versus running your own, or a shared multi-tenant service versus one per team. Give the numbers that decided it: cost per month, migration effort in engineer weeks, or latency.

A wrong call, 5 minutes: one decision that did not hold up, how you found out (an incident, a metric, an engineer who pushed back), and what you changed. Senior engineers often probe this part hardest.

Results and now, 5 minutes: metrics before and after, and what you would do differently today, ideally connected to agent workloads, such as durable execution or per-tenant cost metering (Q1, Q8).

Numbers: prepare 5 to 8 real figures you can defend under follow-up questions, such as peak load, p99 latency, cost, team size and migration length.

Alternatives: pick a newer, smaller system you know in detail, or a larger older one you led at a higher level. The detailed one wins, because senior engineers probe until they hit the limit of your knowledge.

Pitfalls: describing only the happy path; claiming every decision as yours; going so deep on one component that you never finish.

Why this is likely: research shows a dedicated onsite round where senior engineers ask candidates to walk through a complex system they built and its trade-offs (IGotAnOffer EM guide, medium).

</details>

## Prep checklist

- [ ] Practice one end-to-end agent runtime design (Q1) in 45 minutes and finish every part, including failure handling.
- [ ] Draw a trajectory schema and explain deterministic versus counterfactual replay from memory.
- [ ] Explain delegated identity for agents (user plus agent scopes) and when a human approval step is required.
- [ ] Run the eval sample size and kappa math (Q5) and be ready to explain why paired evals need fewer tasks.
- [ ] Do a 5 minute LLM cost estimate out loud with stated assumptions, then name the top two cost levers.
- [ ] Size a sandbox fleet with Little's law at peak, including a warm pool.
- [ ] Prepare one real build, adopt or partner decision you made, with the numbers and what you would change now.
- [ ] Prepare a migration or deprecation story with adoption metrics and an end date.
- [ ] Prepare two leadership stories: a technical disagreement you lost or changed your mind on, and a hard performance conversation.
- [ ] Prepare a cross-org story where a partner team could block you and you built shared ownership.
- [ ] Read the Netflix culture memo and pick the part that resonates least with you, with a reason.
- [ ] Review one AI-written pull request this week and note what you caught.

## Sources

- Netflix culture memo: https://jobs.netflix.com/culture
- Netflix tech blog, Demystifying Interviewing for Backend Engineers at Netflix: https://netflixtechblog.com/demystifying-interviewing-for-backend-engineers-netflix-aceb26a83495
- Netflix tech blog, Engineering AI Discover (agent harness): https://netflixtechblog.com/engineering-ai-discover-e244944cdc38
- Netflix tech blog, Making Failure Diagnosis Reliable with Pensive AI (MCP access): https://netflixtechblog.com/making-failure-diagnosis-reliable-with-pensive-ai-6b0811a1c015
- QCon San Francisco, How Netflix Retrofitted Its SDLC for AI Coding Agents: https://qconsf.com/presentation/nov2026/how-netflix-retrofitted-its-sdlc-ai-coding-agents
- Anthropic webinar with Netflix AI Platform speakers, Scaling AI agent development at Netflix: https://claude.com/resources/webinars/scaling-ai-agent-development-at-netflix
- 1point3acres, Netflix EM hiring manager screen report (first-hand, partly gated): https://www.1point3acres.com/bbs/thread-1193041-1-1.html
- Reddit r/OfferEngineering, Netflix Staff Software Engineer loop (first-hand, relayed): https://www.reddit.com/r/OfferEngineering/comments/1wuabnb/netflix_staff_software_engineer_interview/
- IGotAnOffer, Netflix Engineering Manager Interview (prep guide built on candidate reports): https://igotanoffer.com/en/advice/netflix-engineering-manager-interview
- Blind, Netflix Engineering Manager interview discussion: https://www.teamblind.com/post/g8cj40nl
- Blind, Netflix engineering manager interview experience (older): https://www.teamblind.com/post/netflix-engineering-manager-interview-experience-ndwh7syv
- Blind, Megathread on LeetCode versus AI-assisted coding by company (Netflix employee comment): https://www.teamblind.com/post/megathread-leetcode-vs-ai-assisted-coding-by-company-7ovjpqyf
- Blind, Netflix AI Assisted Interview: https://www.teamblind.com/post/netflix-ai-assisted-interview-7o1ukyfe
- Prepfully, Netflix EM question bank (crowd-sourced, low confidence): https://prepfully.com/interview-questions/netflix/engineering-manager
