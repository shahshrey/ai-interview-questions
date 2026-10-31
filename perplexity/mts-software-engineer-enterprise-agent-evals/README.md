# Perplexity: Member of Technical Staff (Software Engineer, Enterprise Agent Evals & Efficiency)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Perplexity publishes a technical interviewing portal that lists its round types. Each role uses a subset of them, and your recruiter tells you which ones. The stages below combine that portal with candidate reports from 2025 and 2026. Treat the exact order and count as a guess.

1. Recruiter call (30 to 45 minutes). Covers your background, why Perplexity, and the team's work. Several candidates say this call can include a real technical question, so prepare for one.
2. Practical assessment or hands-on coding screen. The practical assessment runs in CoderPad or CodeSignal. It is designed to take 45 to 90 minutes, and you get a 2 or 4 hour window. The live screen is a multi-part Python exercise where each part has its own instructions and tests, and each later part has to keep the earlier parts' tests passing. Perplexity's public example is an agent todo list with task dependencies. Candidates report an in-memory file system and an LLM provider pool with failover. The portal says these rounds skip LeetCode banks, dynamic programming recipes and advanced graph algorithms, and candidates work without AI assistance.
3. Onsite of about 3 to 5 rounds:
   - Hands-on coding: another applied, multi-part component. For this team, expect an eval runner, a statistics utility, or a cache.
   - Agentic coding (possible): a 60 to 120 minute live session, or a 3 hour take-home, where you are expected to use coding agents.
   - System design (45 to 60 minutes): graded on problem framing, requirements, core architecture, quantitative scaling, tradeoffs and domain expertise. For this role, expect eval environments, regression gates or routing.
   - Core expertise (likely for this specialized role): a deep conversation about evaluation statistics, grader calibration, routing and inference efficiency, including your own past systems in detail.
   - Infra or debugging round (reported for some SWE loops): an overloaded service, where you say which metrics you want to see.
4. Manager or cross-functional conversation focused on ownership, ambiguity and impact. Technical topics still come up.

## Questions and answers

### Coding: eval harness infrastructure

#### Q1. Implement an async eval runner that executes N agent tasks against a rate-limited model API, with bounded concurrency, retries on transient errors, per-call timeouts, and results returned in input order.

Problem. Input: a list of `(task_id, payload)` pairs and an async function `call(payload)` that may raise a retryable error (429, 503), raise a non-retryable error (a bad schema), or hang. Output: one `Result` per task, in input order, recording success, output or error, attempt count and latency. Constraints: at most `max_concurrency` requests in flight, at most `rate_per_s` request starts per second, at most `max_retries` retries per task, and each attempt bounded by `timeout_s`.

```python
from __future__ import annotations
import asyncio, random, time
from dataclasses import dataclass
from typing import Any

class RetryableError(Exception):
    pass

class TokenBucket:
    def __init__(self, rate_per_s: float, capacity: int):
        self.rate, self.capacity = rate_per_s, capacity
        self.tokens = float(capacity)
        self.last = time.monotonic()
        self.lock = asyncio.Lock()

    async def acquire(self) -> None:
        while True:
            async with self.lock:
                now = time.monotonic()
                self.tokens = min(self.capacity, self.tokens + (now - self.last) * self.rate)
                self.last = now
                if self.tokens >= 1:
                    self.tokens -= 1
                    return
                wait = (1 - self.tokens) / self.rate
            await asyncio.sleep(wait)  # sleep outside the lock

@dataclass
class Result:
    task_id: str
    ok: bool
    output: Any = None
    error: str | None = None
    attempts: int = 0
    latency_s: float = 0.0

async def run_eval(tasks, call, max_concurrency=16, rate_per_s=10.0,
                   max_retries=4, timeout_s=60.0, base_backoff_s=0.5):
    sem = asyncio.Semaphore(max_concurrency)
    bucket = TokenBucket(rate_per_s, capacity=max(1, int(rate_per_s)))

    async def one(task_id, payload):
        start = time.monotonic()
        async with sem:
            for attempt in range(1, max_retries + 2):
                await bucket.acquire()
                try:
                    out = await asyncio.wait_for(call(payload), timeout_s)
                    return Result(task_id, True, out, None, attempt, time.monotonic() - start)
                except (RetryableError, asyncio.TimeoutError) as e:
                    if attempt > max_retries:
                        return Result(task_id, False, None, repr(e), attempt, time.monotonic() - start)
                    # full jitter exponential backoff
                    await asyncio.sleep(random.uniform(0, base_backoff_s * 2 ** (attempt - 1)))
                except Exception as e:  # non-retryable: fail fast, keep the run going
                    return Result(task_id, False, None, repr(e), attempt, time.monotonic() - start)

    return await asyncio.gather(*(one(t, p) for t, p in tasks))
```

Complexity. O(N) tasks and O(N x retries) calls. Memory is O(N) for results plus O(max_concurrency) live coroutines doing work. `gather` preserves input order, so the results come back ordered without a sort step.

There are two separate limiters, and the interviewer will want you to say why. The semaphore caps in-flight requests, which protects memory and the provider's concurrency limit. The token bucket caps the start rate, which is what RPM limits measure. Full jitter on the backoff stops synchronized retry storms after a provider blip. Timeouts count as retryable, while schema errors fail fast so one bad task cannot burn the retry budget.

Edge cases: an empty task list returns `[]`. With `rate_per_s` below 1, capacity still floors at one token. `asyncio.CancelledError` must propagate, which it does because it is not an `Exception` subclass on 3.8+. A hung call is bounded by `wait_for`.

Likely follow-ups:
- Limit tokens per minute instead of requests: make `acquire(cost)` take the estimated prompt plus max output tokens, then refund the difference after the response.
- Honor `Retry-After` headers and use them to set the backoff.
- Release the semaphore while sleeping in backoff, so slow retries do not starve fresh tasks.
- Checkpoint results to disk with an idempotency key `(run_id, task_id, trial)`, so a crashed run resumes and skips the trials it already paid for.
- Separate agent failures from infrastructure failures in the result, so the eval score counts agent failures only and a 429 stays an infrastructure event.

Why this is likely: Perplexity's hands-on coding guide says exercises build "practical components" of its systems, and the listing makes this team own eval infrastructure, so a concurrent, rate-limited runner is the most direct component to ask for.

#### Q2. Build an LLM provider pool: route each request to the highest-priority healthy provider, fail over on errors, and bring providers back after a cooldown.

Problem. Input: providers in priority order and an async `call(provider, prompt)` that can raise `RetryableError`. Output: `(provider_used, response)`, or `UNAVAILABLE` when nothing is healthy. Constraints: after `failure_threshold` consecutive failures a provider is ejected (circuit open) for `cooldown_s`. Once the cooldown ends, it is eligible again (half-open), and a single further failure re-ejects it. A success resets its failure count. Inject the clock so the pool can be tested deterministically.

```python
import time

class ProviderPool:
    def __init__(self, providers, failure_threshold=3, cooldown_s=30.0, clock=time.monotonic):
        self.order = list(providers)
        self.fails = {p: 0 for p in providers}
        self.open_until = {p: 0.0 for p in providers}
        self.threshold, self.cooldown, self.clock = failure_threshold, cooldown_s, clock

    def _available(self, p):
        return self.clock() >= self.open_until[p]

    def pick(self):
        return next((p for p in self.order if self._available(p)), None)

    def record(self, p, ok):
        if ok:
            self.fails[p] = 0
            self.open_until[p] = 0.0
        else:
            self.fails[p] += 1
            if self.fails[p] >= self.threshold:
                self.open_until[p] = self.clock() + self.cooldown
                self.fails[p] = self.threshold - 1  # half-open after cooldown

    async def complete(self, prompt, call):
        tried = set()
        while True:
            p = next((q for q in self.order if q not in tried and self._available(q)), None)
            if p is None:
                raise RuntimeError("UNAVAILABLE")
            tried.add(p)
            try:
                out = await call(p, prompt)
                self.record(p, True)
                return p, out
            except RetryableError:
                self.record(p, False)
```

Complexity. `pick` is O(P) per request, which is fine for a handful of providers. For the large-P variant that candidates report, where up to 1e5 providers get UP and DOWN events, keep a min-heap of healthy indices with lazy deletion, or a sorted set. That gives O(log P) per event.

Edge cases: every provider down returns UNAVAILABLE without spinning, because each provider is tried at most once per request. UP and DOWN are idempotent. A recovered higher-priority provider wins the very next request. Non-retryable errors such as a 400 from a bad request should propagate and leave the provider's health count untouched, because the fault lies in the request.

Likely follow-ups:
- Weighted or latency-aware selection instead of strict priority.
- Health measured by error rate over a sliding window in place of a consecutive-failure count.
- Thread safety, or `asyncio.Lock` around `record`, under concurrent use.
- Model equivalence: failing over from one model to another changes behavior, so an eval run must log which provider actually served each call. Without that log, a silent failover can look like a real quality regression.
- Hedged requests for tail latency, at extra cost.

Why this is likely: an LLM provider pool with failover appears in several 2025 and 2026 candidate reports of Perplexity coding rounds, and the role owns reliability and multi-model routing.

### Coding: eval statistics

#### Q3. Given per-task scores for a baseline agent A and a candidate agent B on the same task suite, with several trials per task, write a function that returns the mean difference, a 95% bootstrap confidence interval, and a p-value.

Problem. Input: `scores_a` and `scores_b`, each a dict from `task_id` to a list of trial scores in [0, 1]. Output: delta (B minus A), CI, p-value and n. Constraints: tasks are the unit of independence. Trials within a task are correlated, so resampling individual trials would understate variance. Use only the tasks both systems ran.

```python
import random

def paired_bootstrap(scores_a, scores_b, n_boot=10_000, alpha=0.05, seed=0):
    task_ids = sorted(set(scores_a) & set(scores_b))
    if len(task_ids) < 2:
        raise ValueError("need at least 2 shared tasks")
    mean = lambda xs: sum(xs) / len(xs)
    # collapse trials to a per-task mean first: this is the cluster step
    diffs = [mean(scores_b[t]) - mean(scores_a[t]) for t in task_ids]
    n = len(diffs)
    observed = mean(diffs)
    rng = random.Random(seed)

    boots = sorted(mean([diffs[rng.randrange(n)] for _ in range(n)]) for _ in range(n_boot))
    lo = boots[int((alpha / 2) * n_boot)]
    hi = boots[int((1 - alpha / 2) * n_boot) - 1]

    # sign-flip randomization test: under H0 each paired diff is symmetric around 0
    extreme = 0
    for _ in range(n_boot):
        flipped = mean([d if rng.random() < 0.5 else -d for d in diffs])
        if abs(flipped) >= abs(observed) - 1e-12:
            extreme += 1
    p_value = (extreme + 1) / (n_boot + 1)
    return {"delta": observed, "ci": (lo, hi), "p_value": p_value, "n_tasks": n}
```

Complexity. O(n_boot x n) time and O(n_boot) memory. With numpy you vectorize this as `diffs[rng.integers(0, n, (n_boot, n))].mean(1)`.

Pairing matters because the variance of a difference is Var(A) + Var(B) - 2Cov(A, B). Task difficulty dominates agent eval variance and is shared by both systems, so the covariance is large and the paired interval is often a third to a half the width of an unpaired one. Averaging trials first follows Miller's "Adding Error Bars to Evals": the per-task mean shrinks within-task variance by a factor of K, and the task-level resample gives a cluster-robust interval. When tasks come in families, such as many tasks from one connector, resample families instead.

Edge cases: the two runs having different task sets (intersect them and report what was dropped), all diffs zero (the p-value is near 1), and tiny n, where the percentile interval undercovers and BCa or a t-interval is safer.

Likely follow-ups:
- For binary single-trial outcomes, McNemar's test on the discordant pairs is the exact paired test.
- Run a power analysis before launching. The minimum detectable effect is about (z_alpha/2 + z_beta) x sd(diff) / sqrt(n). With sd 0.4 and 300 tasks that is about 2.8 x 0.4 / 17.3, or roughly 6.5 points.
- Correct for multiple comparisons when a gate checks twenty slices (Holm or Benjamini-Hochberg).

Why this is likely: the listing asks for "strong experimental and statistical judgment", and a candidate report lists stream-probability and sampling questions in a Perplexity technical screen, so implementing a paired significance test in code is a natural applied version.

#### Q4. Implement unbiased estimators for pass@k and pass^k from n trials per task, and explain when each is the right reliability metric for an enterprise agent.

Problem. Input: `trials`, a dict from task to a list of 0/1 outcomes (n_i trials, c_i successes). Output: the suite-level pass^k (all k attempts succeed) and pass@k (at least one of k succeeds). Constraint: k ≤ n_i for every task. Run more than k trials per task where the budget allows, because an estimate from exactly k trials has high variance.

```python
import math

def pass_hat_k(trials, k):
    vals = []
    for task, outcomes in trials.items():
        n, c = len(outcomes), sum(outcomes)
        if n < k:
            raise ValueError(f"task {task} has {n} trials < k={k}")
        vals.append(math.comb(c, k) / math.comb(n, k))       # P(all k sampled are successes)
    return sum(vals) / len(vals)

def pass_at_k(trials, k):
    vals = []
    for task, outcomes in trials.items():
        n, c = len(outcomes), sum(outcomes)
        if n < k:
            raise ValueError(f"task {task} has {n} trials < k={k}")
        vals.append(1 - math.comb(n - c, k) / math.comb(n, k))  # 1 - P(all k are failures)
    return sum(vals) / len(vals)
```

With `{"t1": [1,1,1,1], "t2": [1,1,0,1], "t3": [0,0,0,1]}`: pass^1 = pass@1 = 0.667, pass^2 = 0.5 and pass@2 = 0.833.

Complexity. O(T) after counting. `math.comb` is exact for integers, so there is no float overflow. For large n you would compute the product of (c - i)/(n - i) for i in range(k) instead.

The estimators are unbiased because choosing k of the n observed trials without replacement and asking whether all succeeded gives C(c,k)/C(n,k), whose expectation over trial draws equals p^k for the task's true success rate p. These are the formulas from the tau-bench paper (Yao et al., 2024) and the Codex paper's pass@k.

The pass@k metric suits tools where a human picks the best of several attempts, or a verifier can. pass^k suits agents that act on enterprise systems, because a user who sends the same request on Monday and Tuesday expects both to work. The metrics diverge sharply. A 75% per-trial agent has pass^3 = 0.42 (Anthropic's example), and at k = 10 pass@k approaches 1 while pass^k approaches 0. Suite-level pass^k is the mean of p_i^k, not (mean p)^k, so a suite split between always-pass and coin-flip tasks scores far higher than one where every task is 75%.

Edge cases: tasks with different n_i are fine. Exclude tasks with n_i < k and list them in the report. Trials must vary only through sampling. If environment state leaks between trials, they are not independent.

Likely follow-ups:
- Put a CI on pass^k by bootstrapping over tasks.
- Choose n per task when trials are expensive (n = 2k is a common minimum).
- Report pass^k alongside cost, because retry-until-success shifts cost into a reliability number.

Why this is likely: the listing measures "task success and reliability", and pass^k is the standard reliability metric from tau-bench, which Perplexity's own agent evaluation guide and Anthropic's agent eval guide both discuss.

### Coding: caching

#### Q5. Implement a block-level prefix cache for prompt KV state, keyed so that two requests sharing a token prefix reuse the same blocks, with LRU eviction and tenant isolation.

Problem. Input: token id sequences. `insert(tokens, salt)` stores blocks for a processed prompt. `lookup(tokens, salt)` returns how many leading tokens are already cached. Constraints: fixed `block_size` (vLLM defaults to 16 tokens), only full blocks are cached, there is a capacity of `max_blocks` with least-recently-used eviction, and a block is identified by its own tokens plus everything before it.

```python
import hashlib
from collections import OrderedDict

class PrefixCache:
    def __init__(self, block_size=16, max_blocks=1024):
        self.bs, self.max_blocks = block_size, max_blocks
        self.blocks = OrderedDict()   # block_hash -> KV handle (placeholder here)

    def _hashes(self, tokens, salt=""):
        h = hashlib.sha256(salt.encode()).hexdigest()   # salt isolates tenants
        out = []
        full = len(tokens) - len(tokens) % self.bs       # only full blocks
        for i in range(0, full, self.bs):
            block = tokens[i:i + self.bs]
            # chain: hash(parent_hash, block tokens) so a block encodes its whole prefix
            h = hashlib.sha256((h + "|" + ",".join(map(str, block))).encode()).hexdigest()
            out.append(h)
        return out

    def lookup(self, tokens, salt=""):
        hit = 0
        for h in self._hashes(tokens, salt):
            if h not in self.blocks:
                break                                   # prefix match stops at first miss
            self.blocks.move_to_end(h)
            hit += 1
        return hit * self.bs

    def insert(self, tokens, salt=""):
        for h in self._hashes(tokens, salt):
            if h in self.blocks:
                self.blocks.move_to_end(h)
            else:
                self.blocks[h] = object()
                if len(self.blocks) > self.max_blocks:
                    self.blocks.popitem(last=False)     # evict LRU
```

Complexity. Hashing is O(L) per request for L tokens. Lookup and insert are O(L / block_size) dict operations, each O(1). Memory is O(max_blocks).

The hashes are chained because hashing a block alone would let "B" after prefix "A" collide with "B" after prefix "X". Those blocks have different KV values, because attention at every position depends on everything before it. Chaining the parent hash makes the key encode the full prefix, so a dict replaces an explicit trie. This is the vLLM automatic prefix caching design, which also folds LoRA ids, multimodal input hashes and a cache salt into the key. SGLang's RadixAttention reaches the same reuse with a radix tree instead.

Edge cases:
- A one-token change near the top of the prompt misses every later block. This is why timestamps and per-user fields belong at the end of the prompt.
- Without a salt, cache hit latency becomes a cross-tenant timing side channel.
- Evicting a parent while its child stays cached leaves an unreachable child. Production engines evict leaves first, or refcount blocks in use by running requests and never evict those.

Likely follow-ups:
- Evict only blocks with refcount zero.
- Compute the hit rate for an agent loop where each turn appends to the previous prompt (near 100% when the prefix is stable).
- Make a router cache-aware, by sending a session to the replica that holds its prefix.

Why this is likely: the listing names caching and context management as efficiency levers, and the coding brief for this team calls for a data structure or caching problem. A prefix cache is the one that maps directly onto inference cost.

### System design

#### Q6. Design a reproducible evaluation environment for enterprise agents: connected apps (mail, docs, tickets, CRM), realistic data, user roles and permissions, multi-step workflows.

Requirements. Thousands of tasks, each run 3 to 8 times for pass^k, so about 20k to 50k trials per full suite. Each trial starts from an identical state in isolation, sees production tool schemas, and is graded on end state. A full suite finishes in under 2 hours.

Components:
1. Connector fakes behind the production interfaces. Each app is a stateful service that implements the same MCP or tool schema as the real connector. Contract tests replay recorded real API calls against each fake on every connector release to catch drift.
2. A world generator. It produces an org (people, teams, managers), documents, threads and tickets from templates and seeds. It injects near-duplicate names, stale docs with newer replacements, long threads and partial access.
3. Identity and permissions. Each task runs as a specific principal with ACLs enforced inside the fakes, and the suite includes tasks whose correct outcome is a refusal or "you lack access".
4. Snapshot and restore. Each world is a versioned snapshot: a database dump plus a blob store. A trial gets a copy-on-write clone, with a frozen clock and seeded simulated users (an LLM user simulator, as in tau-bench).
5. A task spec: instruction, principal, world version, success predicates over end state (for example "a ticket exists with priority P1 assigned to the on-call owner, and no email was sent outside the domain"), forbidden actions, and an optional rubric for an LLM grader.
6. A runner and trace store. Each trial logs every tool call, result, token count and latency, plus the before and after world diff.

The hard parts:
- Nondeterminism from the model, the user simulator and wall-clock tools. Freeze what you can, and measure the rest with repeated trials.
- Graders that check state diffs. They catch an agent that claims success without acting, or does extra damage.
- Task validity. Have a human solve a sample of tasks. Anthropic's guidance is that when no one succeeds at a task, the task itself is usually broken.

Tradeoffs. High-fidelity fakes are expensive to maintain, while record and replay is cheap but breaks on any new trajectory. Use fakes for the core apps, replay for the long tail, and a small canary suite against real staging tenants to catch fake drift.

Why this is likely: building "realistic enterprise evaluation environments spanning connected applications, representative datasets, user roles and permissions" with "reproducible system state" is the listing's first responsibility, and Perplexity's system design guide says questions cover domains relevant to the team.

#### Q7. Design the regression gate that decides whether a model, connector, tool, skill, or harness change can ship.

Requirements. Changes land daily from four teams. The gate must answer three questions: did anything get worse beyond noise, on which slices, and at what cost. A verdict should arrive in under 30 to 60 minutes for normal PRs, with a deeper nightly run. False blocks must stay rare, or engineers stop trusting the gate.

Design:
1. Tiered suites. A smoke tier (about 100 tasks, 1 trial) runs on every PR in about 10 minutes. A gate tier (1k to 2k tasks, 3 trials) runs before merge to main or a model swap. A full tier (all tasks, 5+ trials plus long-horizon tasks) runs nightly and before launches. Each change type maps to the slices it can affect, such as a connector change triggering that connector's tasks.
2. Fixed baselines. Compare against a pinned baseline run on the same task versions, and re-run the baseline when the world or graders change.
3. Statistical decision. For each metric and slice, compute the paired delta with a cluster bootstrap CI (Q3). Block if the upper bound of the regression CI exceeds a pre-agreed non-inferiority margin, for example -2 points on task success, or a 10% p95 latency increase, or a 10% increase in cost per successful task. Use Holm or Benjamini-Hochberg across slices so twenty slices do not produce a false alarm on every run.
4. Hard invariants with no statistics. Forbidden actions (data sent outside an ACL, destructive calls without confirmation), schema validity, and safety tasks have zero tolerance. A single failure blocks.
5. Grader health. Run the grader on a fixed calibration set on each gate run. If judge agreement with human labels drops, the gate marks the result untrusted, and an untrusted result cannot count as a pass.
6. Output. A report with deltas, CIs, newly failing tasks with links to trajectories, and a cost and latency table. Overrides need a named owner and are logged.

The hard parts: flakiness (track a per-task flip rate and quarantine tasks that flip with no code change), suite saturation (rotate in tasks drawn from fresh production failures), and budget. Example budget: 2k tasks x 3 trials x about 40k tokens is about 240M tokens per gate run, so prompt caching of shared prefixes and cheaper smoke tiers matter.

Tradeoffs. A strict margin catches more regressions but blocks more good changes. Non-inferiority testing is the right framing for efficiency work, where the goal is "no worse and cheaper".

Why this is likely: the listing asks you to build "regression gates and launch-readiness signals for model, connector, tool, skill, and harness changes", and Perplexity's agent evaluation blog lays out the same regression-check workflow.

#### Q8. Design a model routing and effort-selection system for enterprise agent tasks that minimizes cost per successful task without dropping quality.

Requirements. Several model families (Perplexity's Computer runs GPT, Claude Opus and Claude Sonnet orchestrators), each with effort or reasoning levels. The router decides per task, and sometimes per step, which model and effort to use. Targets: quality within a non-inferiority margin of an all-frontier baseline, lower cost per successful task, and p95 latency budgets per product surface. Routing overhead has to stay under about 50 ms.

Components:
1. Features. Request text embedding, task type from a classifier, connectors in scope, expected horizon, tenant tier, and conversation history so far.
2. A router model that predicts P(success | model, effort, features) and expected cost and latency for each option. Options include matrix factorization, a small BERT classifier, or similarity-weighted ranking, as in RouteLLM, which reports cutting cost by more than 2x on benchmarks at near-equal quality. Pick the cheapest option whose predicted success clears a threshold, or maximize success minus lambda x cost, where lambda is tuned on the eval set to land on the cost-quality frontier.
3. Escalation, the cascade pattern from FrugalGPT. Start cheap and escalate when a verifier, a self-check or a tool-error signal says the step failed. Escalation suits agents because failures are often detectable mid-trajectory: a schema error, an empty search result, a loop.
4. Training data. Run the eval suite with every candidate model and record outcomes per task, giving labels for "would the cheap model have succeeded". Add production traces graded by calibrated judges.
5. Online control. Shadow the router before it takes traffic, then run an A/B test on task success, retries and user-visible failure. Keep a per-tenant kill switch.

The hard parts:
- Mid-trajectory switching breaks the prompt cache, because a new model starts with a cold prefix. Account for that in cost, or route only at task start and at subagent boundaries.
- Router calibration drifts when providers update their models.
- Different models need different tool descriptions and skills. Perplexity notes that Sonnet and GPT behave quite differently with skills, so each route needs its own eval coverage.
- Evaluate the router itself: plot the cost-quality curve over thresholds, and compare against an oracle router (cheapest model that succeeded) to see the headroom.

Tradeoffs. Per-step routing saves more but risks cache misses and inconsistent behavior. Per-task routing is simpler and keeps cache locality.

Why this is likely: the listing asks you to "build and evaluate model selection and routing strategies that match each task to the right model and effort level", and Perplexity's Agent API docs describe multi-provider orchestration across model families.

#### Q9. Design a pipeline that turns production agent trajectories into root-caused failure categories and new eval tasks.

Requirements. Millions of agent sessions a week across enterprise tenants. Privacy constraints: tenant data must stay scoped, and humans review only consented or redacted traces. The output is a ranked list of failure modes with frequency and impact, and a flow of reproducible tasks into the eval suites.

Pipeline:
1. Ingest. Structured spans per step (model call, tool call, arguments, result, error, tokens, latency) using OpenTelemetry-style trace and span ids, stored in a columnar store partitioned by tenant and day.
2. Detect candidate failures cheaply first. Explicit signals: tool errors, schema validation failures, retries above N, loops (the same tool and arguments repeated), timeouts, user thumbs-down, user re-asks, abandoned sessions. Then sample by stratum and run calibrated LLM judges for outcome success and policy compliance.
3. Attribute. For each failed trace, assign a fault owner (agent, tool or connector, environment, user) and a primary type (wrong tool, wrong arguments, missing permission handling, goal partially done, unintended action, context lost). tau-bench ships exactly this two-level labeling. AgentRx-style labeling also marks the critical failure step, which is the first step after which the task could no longer succeed.
4. Build the taxonomy with humans. Open-code 50 to 100 traces per new product area into categories, then let the LLM labeler apply the taxonomy at scale. Re-check labeler agreement with humans monthly.
5. Prioritize by frequency x severity x fixability, and by cost, since a loop that burns 200k tokens matters even if it is rare.
6. Close the loop. For each top category, minimize a trace into a task spec: world snapshot, principal, instruction, success predicate. Add it to the gate suite, ship the fix (a tool description, a skill gotcha, harness retry logic), and confirm with a paired comparison.

The hard parts:
- Reproducing production state in a fake world, which needs synthetic stand-ins for tenant data.
- Attribution ambiguity: a vague tool description makes the model pick the wrong tool, so is that the model or the tool?
- Selection bias, because user complaints over-represent some surfaces.

As a sizing example, if 3% of 1M weekly sessions fail and you judge-sample 2%, that is 600 judged failures a week, enough to rank the top 10 categories with tight intervals.

Why this is likely: the listing asks you to "analyze agent trajectories and production failures, identify root causes, and ship improvements", and Perplexity's evaluation guide recommends building test sets from real failures and running calibrated judges on production traces.

### Domain: graders and calibration

#### Q10. You have an LLM judge that grades whether an enterprise task was completed. How do you calibrate it against humans, and what numbers would make you trust it?

Start with labels you can trust. Pick 200 to 500 trajectories stratified by task type and by the judge's own verdict, so both rare failures and passes are covered. Have at least two domain experts label each one blind to the judge, because annotators shown the judge's answer anchor on it. Write the rubric first and adjudicate disagreements, which shows where it is ambiguous.

Metrics:
- Human-human Cohen's kappa is the ceiling: κ = (p_o - p_e) / (1 - p_e), where p_o is observed agreement and p_e is chance agreement from the marginals. If two experts agree at κ = 0.62, a judge at 0.60 with each of them behaves like a third annotator, and asking for 0.9 asks more than the task allows. Below about 0.6 human-human, fix the rubric before tuning the judge.
- Judge-human kappa, plus the full confusion matrix. Kappa matters because accuracy is misleading at skewed prevalence: if 90% of runs pass, a judge that always says pass scores 90% accuracy and κ = 0.
- Track TPR and TNR separately, because the pass rate you report is biased by both. If the judge's true positive rate is s and its true negative rate is t, the corrected pass rate is θ = (p_obs + t - 1) / (s + t - 1), the Rogan-Gladen correction. With s = 0.95, t = 0.80 and an observed rate of 0.70, the true rate is 0.667.
- Consistency. Grade the same saved trajectory 5 times without rerunning the agent. Perplexity's own guide recommends this to separate agent variance from grader variance.
- Calibration, if the judge gives confidence: expected calibration error and Brier score.

```python
def cohen_kappa(a, b):
    n = len(a)
    po = sum(x == y for x, y in zip(a, b)) / n
    pe = sum((a.count(l) / n) * (b.count(l) / n) for l in set(a) | set(b))
    return 1.0 if pe == 1 else (po - pe) / (1 - pe)
```

Iterate on the individual disagreements and read every false pass. In an enterprise setting a false pass hides a real failure, so I would weight false negatives on safety and permission criteria far more heavily than false fails. Prefer narrow binary criteria ("ticket priority equals P1", "cites the right doc") over a 1 to 10 holistic score, and use code checks on end state wherever the criterion is verifiable. Hold out a test split to catch rubric overfitting. Re-run the calibration set whenever the judge model or prompt changes, and on a schedule, because provider updates shift judges silently.

Why this is likely: the listing says graders must be "calibrated against human judgment", and Perplexity's October 2026 agent evaluation post describes a five-step human calibration process and a judge consistency check.

#### Q11. What are the main failure modes of LLM judges and outcome graders for agents, and how do you defend against each?

Biases in model judges:
- Position bias in pairwise grading: the judge prefers whichever answer appears first, or second. Measure swap consistency by running both orders, count disagreements as ties, and report the rate.
- Verbosity bias: longer, more confident answers win. Control by grading against explicit criteria, and add length-matched pairs to the calibration set.
- Self-preference: a judge from the same family as the agent rates its own style higher. Use a judge from a different family, or an ensemble, and check agreement separately for each agent family under test.
- Rubric drift and prompt sensitivity: small judge prompt edits change pass rates by several points. Version judge prompts like code and re-run calibration on every change.

Failure modes specific to agents:
- Transcript versus outcome. An agent says "I booked it" and nothing exists in the database. Grade end state with code where possible, and use the judge for what code cannot check, such as tone, completeness and correct refusal.
- Over-specified graders. Checking that the agent called tools in a fixed order punishes valid alternate paths. Anthropic's eval guide recommends grading outcomes over the exact path. Check outcomes and constraints (no forbidden tool, at most N calls).
- Reward hacking and loopholes. Agents find paths the task author did not intend, sometimes valid ones (Anthropic describes a model "failing" a flight eval by finding a better policy loophole). Review a sample of passes as well as failures.
- Broken tasks. A 0% pass rate across strong models usually means an impossible task or a buggy grader. Have a human solve the task, and require a reference solution that passes the grader.
- Saturation. When a suite is above 95%, it only detects regressions, so add harder tasks from production.
- Judge cost and latency at scale. Use a cheap judge for screening and escalate disagreements or low-confidence verdicts to a stronger judge or a human.
- Contamination. Tasks published in papers or repos end up in training data, so keep a private held-out set.

What I would monitor continuously: judge-human kappa on a rolling labeled sample, swap consistency, the pass rate of reference solutions (should be 100%), and the flip rate of verdicts on unchanged trajectories.

Why this is likely: the listing requires graders "grounded in end-to-end outcomes", and Perplexity's skills post describes LLM judges grading full agent loops against rubrics, so knowing where those judges break is core to the role.

### Domain: efficiency, caching and context

#### Q12. Explain how prompt caching works at the provider and engine level, its economics, and how you would restructure an agent harness to raise the hit rate.

Mechanism. Prefill computes key and value tensors for every prompt token at every layer. A prompt cache stores those KV blocks for a prefix and reuses them when a later request starts with identical tokens, skipping that prefill compute and cutting time to first token. Matching is exact and prefix-only, at block granularity. In vLLM, a block is identified by a hash chained over the previous block hashes plus its own tokens (Q5). A single changed token invalidates that block and everything after it. The model still generates fresh output, whereas a semantic response cache returns a stored response.

Provider economics (as published in 2026; verify current pricing):
- Anthropic: explicit `cache_control` breakpoints. Reads cost about 0.1x base input, 5-minute writes cost 1.25x and 1-hour writes cost 2x.
- OpenAI: automatic caching above a minimum of about 1,024 tokens, with a `prompt_cache_key` to steer routing.
- Gemini: implicit caching plus explicit caches billed for storage.

Break-even for a 5-minute write: n uses cost 1.25 + 0.1(n - 1) against n uncached, so caching wins from the second use. A 1-hour write needs three uses.

Harness changes that raise the hit rate:
1. Order the prompt from most stable to least stable: system prompt, tool definitions, skill index, long-lived documents, conversation history, then the newest turn. Put timestamps, user ids and request ids at the end, after the stable prefix.
2. Keep tool definitions deterministic, with a fixed order and no per-request reordering. Loading a skill or tool mid-session inserts tokens in the middle and invalidates everything after. Append new context instead, for example skill files read as tool results.
3. Make context compaction cache-aware. Summarizing old turns rewrites the prefix and forces a full re-prefill. Compact rarely and in large steps, and at a breakpoint.
4. Keep sessions sticky to a replica or cache key. A router that switches models or replicas mid-session throws the cache away.
5. Measure: the cached-token fraction per call, TTFT with and without hits, and cost per successful task. Production coding agents report near 100% cached-input fraction with a stable prefix.

One failure mode worth naming is a "harmless" harness change, such as adding the date to the system prompt, can double input cost overnight. The regression gate should track cache hit rate as a first-class metric.

Why this is likely: the listing names "caching" and "context management" as efficiency levers and makes cost per successful task a target metric. Perplexity runs its own inference engine (ROSE), so KV-level reasoning is in scope.

#### Q13. Define "cost per successful task" precisely, explain how to estimate it with uncertainty, and describe how you would use it to decide between two agent configurations.

Definition. Cost per successful task (CPST) = total cost of all attempts / number of successful tasks. The numerator includes everything spent on the run, failed attempts included:
- Input tokens, split into cached and uncached because they are priced differently.
- Output and reasoning tokens.
- Tool and connector calls with their own costs (search, sandbox minutes).
- Grader cost, if you are measuring the eval pipeline itself.
- Retries.

Failed attempts belong in the numerator because a cheap model that fails half the time and retries can cost more per success than an expensive model that succeeds first time. With retries allowed up to R, the expected cost per success is roughly E[cost per attempt] x E[attempts] / P(eventual success).

Estimation. CPST is a ratio of two random sums, so its variance is not just the variance of cost. Use the delta method: for C = mean cost per task and S = success rate, Var(C/S) ≈ (1/S²)Var(C) + (C²/S⁴)Var(S) - 2(C/S³)Cov(C, S), all divided by n. Or bootstrap over tasks, which handles skewed costs. Agent costs are heavy-tailed, because a few loops cost 50x the median, so also report p50, p95 and the max per task, and cap or flag runaway runs. Compare configurations with a paired bootstrap on the same tasks.

Deciding between A and B:
1. Treat quality as a constraint. B must be non-inferior on task success (for example, the lower CI bound of B - A > -1.5 points) and must hold steady on every hard invariant and high-value slice.
2. Given that, prefer lower CPST, and check p95 latency against the product SLO.
3. Plot the frontier. For routing thresholds or effort levels, plot success against CPST and choose a point on the Pareto frontier. Show the whole curve, because stakeholders weigh quality and cost differently by tenant.
4. Segment. A blended CPST can hide that B is cheaper on easy tasks but far worse on long-horizon ones. Report by task family and horizon.

Failure modes: comparing CPST across different task mixes (normalize per family), ignoring cache effects (A's cost measured warm and B's cold), and counting "success" with an uncalibrated judge, which inflates the denominator.

Why this is likely: the listing names "cost per successful task" as a metric the team moves and asks for sound "quality, latency and cost tradeoff judgment", so expect to be asked to define it precisely.

### Domain: tools, skills and harness

#### Q14. How do you evaluate and improve tool descriptions, schemas, and skill loading so the agent picks the right tool, calls it correctly, and recovers from errors with fewer calls?

Treat tool and skill selection as a retrieval and classification problem with its own eval, separate from end-to-end success.

1. Selection evals. For each tool or skill, write positive queries (it should be chosen), negative queries for adjacent tools (it must not be chosen), and forbidden cases. Perplexity describes exactly this for skills: suites that check precision, recall and forbidden loads, plus whether the agent actually reads the accessory files after loading (progressive loading). Track a confusion matrix across tools. Most selection bugs are two tools with overlapping descriptions.
2. Call-correctness evals. Check argument validity against the schema, semantic correctness of the arguments (the right user id, the right date range), and the number of calls to reach the goal.
3. End-to-end evals with the full agent loop and a rubric judge, to confirm that a description fix that raises selection accuracy also holds up on the overall task.
4. Run everything for each orchestrator model. Different model families react differently to the same description.

Improvements that usually move the numbers:
- Descriptions written as routing triggers ("Use when the user asks about X; not for Y") and aimed at the model that routes on them. Perplexity's skill guide says the description is the hard part, because the model routes on it.
- Namespacing and consolidation. One `search_tickets(query, status, assignee)` beats three overlapping list tools.
- Unambiguous parameter names (`user_id` rather than `user`), enums instead of free strings, and examples in the schema.
- Token-efficient responses: pagination, filtering, and a `response_format` of concise or detailed. Return semantic fields (names), which saves the model a second call to resolve an opaque UUID.
- Actionable errors. "Invalid date; expected YYYY-MM-DD; did you mean 2026-10-30?" lets the agent recover in one step instead of three. Anthropic's tool-writing guide reports large gains from exactly these edits.
- Gotchas in skills, added as the agent fails. Start thin and grow from observed failures.

Adding a new tool or skill can break selection of existing ones even though you never touched them (action at a distance), so every addition re-runs the full selection suite. Measure success and calls per successful task together, so a fix that adds an extra lookup call is visible as a cost regression.

Why this is likely: the listing asks you to "improve tool descriptions, schemas and result handling so agents select the right tools ... and recover from errors with fewer unnecessary calls", and Perplexity's published skills guide describes the precision, recall and forbidden-load suites it runs.

### Behavioral

#### Q15. Tell me about a time you led an ambiguous, cross-team effort to define what "good" meant for a system and then moved the metric.

The interviewer is testing whether you can take a vague goal ("make agents better for enterprise customers"), turn it into a measurable definition, earn agreement from teams you do not manage, and ship changes that move the number. Perplexity's portal lists the traits it screens for: rigorous thinking, controlling complexity, principled tradeoffs, fast learning, and clear communication. The listing adds leading "ambiguous, cross-team initiatives" and mentoring.

A strong structure:
1. Situation in two sentences, with scale: which system, who used it, and what was broken or unmeasured.
2. The ambiguity. Name the specific disagreement, for example product wanted user satisfaction, research wanted benchmark scores, and platform cared about latency.
3. Your definition of success, and how you got buy-in. For example, you wrote a one-page metric spec, labeled 100 real cases with each team, showed where their intuitions disagreed, and agreed a primary metric with guardrails.
4. The mechanism you built: an eval set, a grader, a dashboard, a gate. Give a concrete number (300 tasks, κ = 0.7 with experts, 25 minutes per run).
5. What you changed, and the measured effect, with honest uncertainty: "task success went from 61% to 72%, a CI of +7 to +15 on a paired comparison, and cost per success fell 30%."
6. A tradeoff you made deliberately and one thing that went wrong. For example, the first grader over-passed, you caught it by reading false passes, and you changed the rubric.
7. What persisted after you: other teams adopted the gate, or you mentored someone who now owns it.

Answers lose points for missing numbers, a metric you defined but nobody used, "we" throughout with no clear personal contribution, or a story where the eval number moved but you cannot say whether users noticed. Prepare one backup story about a time your eval was wrong and how you found out, because interviewers in eval roles often probe whether you distrust your own measurements.

Why this is likely: candidate reports describe a behavioral and project deep-dive in the recruiter and manager rounds, and Perplexity's portal says most technical loops include at least one non-technical round.

## Prep checklist

- [ ] Solve Perplexity's public Todo List practice exercise end to end in Python under 60 minutes, keeping earlier parts' tests green as you add parts.
- [ ] Write the async eval runner from Q1 from memory, including token bucket, semaphore, jittered retries, timeouts and ordered results, then add a tokens-per-minute limit.
- [ ] Implement the provider pool with circuit breaking (Q2), plus the heap-based variant for 1e5 providers and events.
- [ ] Code a paired cluster bootstrap and a McNemar test, and compute a minimum detectable effect for 300 tasks by hand.
- [ ] Derive the unbiased pass@k and pass^k estimators and explain why suite pass^k is the mean of p_i^k.
- [ ] Implement a chained-hash prefix cache with LRU and refcounts, and explain why one changed token invalidates every later block.
- [ ] Compute Cohen's kappa, TPR, TNR and a prevalence-corrected pass rate on a small labeled set, and be able to explain why accuracy misleads at 90% prevalence.
- [ ] Read Perplexity's posts on agent evaluation, agent skills, and the WANDR benchmark, and form an opinion on each grading choice.
- [ ] Read the tau-bench paper and Anthropic's "Demystifying evals for AI agents" and "Writing effective tools for agents".
- [ ] Practice one 45-minute whiteboard design each for an eval environment with snapshot and restore, a regression gate with non-inferiority margins, and a cost-aware router.
- [ ] Learn current prompt caching rules and prices for at least two providers, and compute break-even uses for a 5-minute and a 1-hour write.
- [ ] Prepare two behavioral stories with numbers: one where you defined a metric across teams, one where your own eval was wrong.

## Sources

- https://interviewresources.perplexity.ai
- https://interviewresources.perplexity.ai/hands-on-coding
- https://interviewresources.perplexity.ai/hands-on-coding/examples/todo-list/
- https://interviewresources.perplexity.ai/system-design/
- https://interviewresources.perplexity.ai/core-expertise/
- https://interviewresources.perplexity.ai/agentic-coding/
- https://interviewresources.perplexity.ai/practical-assessment/
- https://www.perplexity.ai/hub/blog/ai-agent-evaluation
- https://www.perplexity.ai/hub/blog/designing-refining-and-maintaining-agent-skills-at-perplexity
- https://www.perplexity.ai/hub/blog/wandr-benchmark-evaluating-research-agents-that-must-search-wide-and-deep
- https://github.com/perplexityai/search_evals
- https://docs.perplexity.ai/docs/cookbook/articles/multi-provider-orchestration/README
- https://jobs.accel.com/companies/perplexity-2/jobs/97418124-member-of-technical-staff-software-engineer-enterprise-agent-evals-efficiency
- https://aceoffer.app/interviews/perplexity
- https://www.reddit.com/r/Hack2Hire/comments/1us3j19/perplexity_interview_process_experience
- https://ratelys.com/company/perplexity-ai/interviews
- https://www.linkjob.ai/interview-questions/perplexity-ai-interview
- https://www.fastprep.io/problems/perplexity-llm-provider-pool-failover
- https://www.calibrd.com/interview-prep/perplexity-interview
- https://www.techprep.app/blog/perplexity-interview-process
- https://www.1point3acres.com/interview/problems/company/perplexity
- https://arxiv.org/pdf/2406.12045
- https://github.com/sierra-research/tau-bench
- https://www.anthropic.com/research/demystifying-evals-for-ai-agents
- https://www.anthropic.com/research/writing-tools-for-agents
- https://arxiv.org/abs/2411.00640
- https://cameronrwolfe.substack.com/p/stats-llm-evals
- https://www.aievals.co/learn/statistics/bootstrap-and-paired-tests
- https://www.potatoannotator.com/docs/guides/llm-judge-human-agreement
- https://arxiv.org/html/2406.07791v9
- https://arxiv.org/html/2406.18665v4
- https://github.com/vllm-project/vllm/blob/main/docs/design/prefix_caching.md
- https://www.morphllm.com/prompt-caching
- https://arxiv.org/html/2602.02475
