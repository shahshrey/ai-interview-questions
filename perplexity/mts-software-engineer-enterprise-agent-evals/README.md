# Perplexity: Member of Technical Staff (Software Engineer, Enterprise Agent Evals & Efficiency)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Perplexity publishes a technical interviewing portal that lists its round types and says each role gets a subset of them plus at least one non-technical round. Your recruiter tells you which. The stages below merge that portal with recent candidate reports, so treat the order and count as a guess. Reports for software roles describe AI-free, multi-part Python coding, which makes that the main thing to prepare, with an agentic round possible.

1. Recruiter call (20 to 30 minutes, virtual). Background, why Perplexity, and the team's work. Some candidates report a real technical question here. AI: not relevant.
2. Hiring manager intro call (short, virtual, reported in some MTS loops). Why this team and this role. AI: not relevant.
3. Practical assessment or live coding screen. The at-home practical assessment runs in CoderPad or CodeSignal and is designed for 45 to 90 minutes inside a 2 or 4 hour window. The live screen is 30 to 60 minutes in CoderPad with full-screen sharing. It is a multi-part Python exercise where each part adds tests and earlier tests must stay green. Reported prompts include an in-memory file system, an agent todo list with dependencies (Perplexity's public practice exercise), a byte tokenizer and an LLM provider pool with failover. The portal says it uses no LeetCode banks, dynamic programming recipes or advanced graph algorithms. AI: not allowed, apart from documentation and asking about library usage.
4. Onsite hands-on coding (one or two rounds, 45 to 60 minutes, in person or virtual depending on role and geography). It is another applied component, and for this team you can expect an eval runner, statistics code, a cache or trace analysis. AI: not allowed.
5. Agentic coding (possible). A live 60 to 120 minute session outside CoderPad, or a 3 hour take-home, on a multi-file task where you must drive frontier coding agents and the bar for scope and quality is higher than in hands-on coding. Candidates have reported it for ML roles; no first-hand report for this role yet. AI: allowed and expected.
6. System design (45 to 60 minutes, physical or virtual whiteboard). Framing, requirements, architecture, quantitative scaling and tradeoffs; some prompts hand you docs for an existing system to refine. Expect eval environments, regression gates, routing or long-running agent execution. AI: not used.
7. Core expertise (30 to 60 minutes, domain named in advance). Eval statistics, grader calibration, routing and inference cost, plus your own past systems in depth. AI: not used.
8. Hiring manager deep dive (candidates report about 90 minutes). One past project end to end, ownership and ambiguity. AI: not relevant.
9. Final conversation with a founder or senior leader (about 30 minutes). AI: not relevant.

A few loops use a Spike Round instead of some technical rounds. It is a self-directed project of up to 9 hours where any AI dev tool is allowed. There are no classic algorithm questions below, because Perplexity states it does not use LeetCode-style question banks.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build an async eval runner in three parts: bounded concurrency, retries with timeouts,... | Build | Onsite hands-on coding | Not allowed | Medium |
| [Q2](#q2) | Build an LLM provider pool that fails over, ejects unhealthy providers and recovers... | Build | Technical screen or onsite coding | Not allowed | Medium |
| [Q3](#q3) | Build an in-memory file system for eval worlds, then add O(1) snapshots and a state diff... | Build | Technical screen | Not allowed | Medium |
| [Q4](#q4) | Debug this eval statistics module: the regression gate never fires and pass^k looks... | Debug | Onsite hands-on coding | Not allowed | Medium |
| [Q5](#q5) | Review this AI-generated LLM judge harness before it merges: find every defect and fix... | Review | Onsite hands-on coding | Not allowed | Medium |
| [Q6](#q6) | Find the root cause in this agent trace, then write a detector for a million traces | Trace debug | Onsite hands-on coding | Not allowed | Medium |
| [Q7](#q7) | Implement a block-level prefix cache for prompt KV state with LRU eviction and tenant... | Build | Onsite hands-on coding | Not allowed | Hard |
| [Q8](#q8) | Agentic coding: add a per-slice regression `gate` command to this eval harness repo | Build | Agentic coding | Allowed | Hard |
| [Q9](#q9) | Design a reproducible evaluation environment for enterprise agents with connected apps... | Design | System design | Not allowed | Hard |
| [Q10](#q10) | Design the regression gate for model, connector, tool, skill and harness changes | Design | System design | Not allowed | Hard |
| [Q11](#q11) | Design model routing and effort selection that minimizes cost per successful task | Design | System design | Not allowed | Hard |
| [Q12](#q12) | Design execution for long-running agent tasks that survive crashes, deploys and budget... | Design | System design | Not allowed | Hard |
| [Q13](#q13) | Design the calibration study that decides whether an LLM judge may gate launches | Concept plus code | Core expertise | Not allowed | Hard |
| [Q14](#q14) | A timestamp sits at the top of a 30k token system prompt. Estimate the cost and fix it | Optimize | Core expertise | Not allowed | Medium |
| [Q15](#q15) | Frontier model at high effort or mid model with escalation: decide which ships on cost... | Concept plus code | Core expertise | Not allowed | Medium |
| [Q16](#q16) | Design the eval suite that proves a tool description and skill loading rewrite helped | Design | Core expertise | Not allowed | Medium |
| [Q17](#q17) | How do you use coding agents day to day, and when did an agent's output look right but... | Behavioral | Hiring manager deep dive | Not allowed | Medium |

### Coding: hands-on rounds (no AI, as at Perplexity)

<a id="q1"></a>

#### Q1. Build an async eval runner in three parts (no AI, as at Perplexity): part 1 runs N agent tasks with bounded concurrency and returns results in input order; part 2 adds retries with backoff for 429 and 503 plus a per-call timeout; part 3 adds a requests-per-second limit.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Async concurrency, retries, rate limiting.

<details>
<summary>Show answer</summary>

Task: Input is a list of `(task_id, payload)` and an async `call(payload)` that may raise `RetryableError`, raise a non-retryable error (bad schema), or hang. Return one `Result` per task in input order with ok, output or error, attempts and latency. At most `max_concurrency` calls in flight, at most `rate_per_s` starts per second, at most `max_retries` retries, each attempt bounded by `timeout_s`. Each part keeps earlier tests green.

What they grade: working code under the multi-part tests, sound abstractions, clear handling of retryable versus fatal errors, and pacing across parts.

Approach: A semaphore bounds in-flight calls and a token bucket bounds starts. I compared two other limiters. A semaphore alone leaves starts unbounded. With 16 slots and 200 ms calls it allows up to 80 starts per second, which breaks a 10 RPS quota. A fixed sleep of 1 / rate between starts honors the quota but cannot burst after an idle gap. The token bucket allows a burst of `capacity` and then holds the long-run rate, so I use it and keep the semaphore to bound memory. Retries use full jitter so 16 workers do not retry in the same millisecond after a provider blip. Timeouts count as retryable; schema errors fail on the first attempt so the retry budget goes to transient faults. `gather` keeps input order, so no sort is needed. Cost is O(N x retries) calls and O(N) memory.

Code:

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
            await asyncio.sleep(wait)

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
                    await asyncio.sleep(random.uniform(0, base_backoff_s * 2 ** (attempt - 1)))
                except Exception as e:
                    return Result(task_id, False, None, repr(e), attempt, time.monotonic() - start)

    return await asyncio.gather(*(one(t, p) for t, p in tasks))

async def _self_test():
    seen = {}
    async def call(p):
        seen[p] = seen.get(p, 0) + 1
        if p == "flaky" and seen[p] < 3:
            raise RetryableError("429")
        if p == "bad":
            raise ValueError("schema")
        if p == "hang":
            await asyncio.sleep(10)
        return p.upper()
    tasks = [("t1", "ok"), ("t2", "flaky"), ("t3", "bad"), ("t4", "hang")]
    res = await run_eval(tasks, call, max_concurrency=2, rate_per_s=100,
                         max_retries=2, timeout_s=0.05, base_backoff_s=0.01)
    assert [r.task_id for r in res] == ["t1", "t2", "t3", "t4"]
    assert res[0].output == "OK" and res[1].ok and res[1].attempts == 3
    assert not res[2].ok and res[2].attempts == 1
    assert not res[3].ok and res[3].attempts == 3 and "Timeout" in res[3].error
    assert await run_eval([], call) == []

asyncio.run(_self_test())
print("q1 ok")
```

Verify: order preserved, a flaky task succeeding on attempt 3, a schema error failing once, a hanging task timing out after all retries, and an empty input. In the interview I would also count concurrent calls with a counter to prove the cap holds.

Pitfalls:
- Holding the semaphore during backoff sleeps, which idles slots while fresh tasks wait.
- Catching `CancelledError`; it is not an `Exception` subclass on Python 3.8 and later, so it propagates here.
- Counting infrastructure failures such as a final 429 as agent failures in the score.

Follow-ups: token-per-minute limits with `acquire(cost)`, honoring `Retry-After`, and checkpointing results by `(run_id, task_id, trial)` so a crashed run resumes.

Why this is likely: Perplexity's hands-on coding guide says exercises build practical multi-part components of its systems, and this team owns eval infrastructure, so a rate-limited runner is the most direct component to ask for (official format, high confidence; topic inferred, medium).

</details>

<a id="q2"></a>

#### Q2. Build an LLM provider pool (no AI, as at Perplexity): route each request to the highest-priority healthy provider, fail over on retryable errors, eject a provider after repeated failures, and bring it back after a cooldown.

Round: Technical screen or onsite hands-on coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Failover, circuit breaking.

<details>
<summary>Show answer</summary>

Task: Given providers in priority order and an async `call(provider, prompt)` that can raise `RetryableError`, return `(provider_used, response)` or raise `UNAVAILABLE` when nothing healthy remains. After `failure_threshold` consecutive failures a provider is ejected for `cooldown_s`; after the cooldown it is half-open and one more failure ejects it again; a success resets it. The clock is injected so tests are deterministic.

What they grade: correct state transitions, fast failure when everything is down, testability, and whether you think about what failover does to eval results.

Approach: Track consecutive failures and an `open_until` time per provider. Setting `fails` to `threshold - 1` on ejection makes the half-open state cheap, because the first failure after cooldown ejects again without a separate state enum. Each request tries each provider at most once, so an all-down pool fails after one pass. The linear scan is O(P) per request, fine for 3 to 10 providers. Candidates report a large variant with up to about 1e5 providers and UP and DOWN events; that needs a min-heap of healthy priority indices with lazy deletion, O(log P) per event. Consecutive-failure counting reacts in 3 calls but flaps on a provider failing 30% of calls; an error rate over a sliding window of 50 calls is steadier but ejects later. I start with consecutive counts and name the window as the upgrade.

Code:

```python
import asyncio, time

class RetryableError(Exception):
    pass

class ProviderPool:
    def __init__(self, providers, failure_threshold=3, cooldown_s=30.0, clock=time.monotonic):
        self.order = list(providers)
        self.fails = {p: 0 for p in providers}
        self.open_until = {p: 0.0 for p in providers}
        self.threshold, self.cooldown, self.clock = failure_threshold, cooldown_s, clock

    def _available(self, p):
        return self.clock() >= self.open_until[p]

    def record(self, p, ok):
        if ok:
            self.fails[p], self.open_until[p] = 0, 0.0
            return
        self.fails[p] += 1
        if self.fails[p] >= self.threshold:
            self.open_until[p] = self.clock() + self.cooldown
            self.fails[p] = self.threshold - 1

    async def complete(self, prompt, call):
        tried = set()
        while True:
            p = next((q for q in self.order if q not in tried and self._available(q)), None)
            if p is None:
                raise RuntimeError("UNAVAILABLE")
            tried.add(p)
            try:
                out = await call(p, prompt)
            except RetryableError:
                self.record(p, False)
                continue
            self.record(p, True)
            return p, out

class FakeClock:
    t = 0.0
    def __call__(self):
        return self.t

async def _self_test():
    clk, down, calls = FakeClock(), {"a"}, []
    pool = ProviderPool(["a", "b"], failure_threshold=2, cooldown_s=10, clock=clk)
    async def call(p, prompt):
        calls.append(p)
        if p in down:
            raise RetryableError(p)
        return f"{p}:{prompt}"
    assert await pool.complete("x", call) == ("b", "b:x")
    assert await pool.complete("y", call) == ("b", "b:y")
    assert not pool._available("a")
    calls.clear()
    assert await pool.complete("z", call) == ("b", "b:z") and calls == ["b"]
    clk.t, down = 11, set()
    assert await pool.complete("w", call) == ("a", "a:w")
    down.update({"a", "b"})
    try:
        await pool.complete("v", call)
        raise AssertionError("expected UNAVAILABLE")
    except RuntimeError as e:
        assert str(e) == "UNAVAILABLE"

asyncio.run(_self_test())
print("q2 ok")
```

Verify: failover to the second provider, ejection after the threshold (the ejected provider is not even called), recovery after the cooldown, and a clean `UNAVAILABLE` when all are down.

Pitfalls:
- Treating a 400 as a provider fault; non-retryable errors propagate and leave health untouched.
- Races under concurrency; wrap `record` in an `asyncio.Lock` or accept benign races and say so.
- Silent failover inside evals: every call must log which provider served it, or a failover looks like a quality regression in the gate.

Follow-ups: the heap version for 1e5 providers, weighted load balancing among equal priorities, and per-tenant quotas.

Why this is likely: an LLM provider pool with failover appears in several candidate reports of Perplexity coding rounds, and the listing makes this team own multi-model reliability and routing (candidate reports, medium-high confidence).

</details>

<a id="q3"></a>

#### Q3. Build an in-memory file system for eval worlds in three parts (no AI, as at Perplexity): part 1 supports mkdir, write, read and ls on absolute paths; part 2 adds snapshot() and restore(id) that are O(1); part 3 adds diff(a, b) listing added, removed and changed files so a grader can check end state.

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: In-memory file system, snapshots, state diff.

<details>
<summary>Show answer</summary>

Task: Part 1 is the classic spec: `mkdir` and `write` create missing parents, `read` on a directory or a missing path raises, `ls` on a file returns its name and on a directory returns sorted children. Part 2: every eval trial must start from the same world, so `snapshot()` returns an id and `restore(id)` brings the tree back, both in O(1). Part 3: `diff(a, b)` between two snapshots, used to grade what the agent actually changed.

What they grade: a clean tree abstraction, correct error cases, a snapshot design that scales, and keeping part 1 tests green while adding parts 2 and 3.

Approach: Directories are dicts and files are strings. Writes never mutate a node in place; they copy only the dicts along the path (path copying), so every old root stays a valid, immutable version. A snapshot is then a reference to the root, and unchanged subtrees are shared by identity. That also makes the diff cheap, because `x is y` skips any subtree the agent never touched. I considered two alternatives. Deep-copying the tree per snapshot is O(N) per trial, so 10k files across 30k trials is 300M node copies. An undo log gives O(1) writes but restore costs O(changes) and diff needs a replay. Path copying costs O(depth x fanout) per write, which suits read-heavy eval worlds.

Code:

```python
class FS:
    def __init__(self):
        self.root, self.snaps = {}, []

    @staticmethod
    def _parts(path):
        if not path.startswith("/"):
            raise ValueError("absolute paths only")
        return [p for p in path.split("/") if p]

    def _node(self, path):
        node = self.root
        for name in self._parts(path):
            if not isinstance(node, dict) or name not in node:
                raise FileNotFoundError(path)
            node = node[name]
        return node

    def _put(self, path, value):
        parts = self._parts(path)
        if not parts:
            raise IsADirectoryError(path)
        def rec(node, i):
            new, name = dict(node), parts[i]
            cur = new.get(name)
            if i == len(parts) - 1:
                if isinstance(value, dict):
                    if isinstance(cur, str):
                        raise FileExistsError(path)
                    if cur is None:
                        new[name] = value
                elif isinstance(cur, dict):
                    raise IsADirectoryError(path)
                else:
                    new[name] = value
                return new
            if isinstance(cur, str):
                raise NotADirectoryError(path)
            new[name] = rec(cur or {}, i + 1)
            return new
        self.root = rec(self.root, 0)

    def mkdir(self, path):
        self._put(path, {})

    def write(self, path, data):
        self._put(path, str(data))

    def read(self, path):
        node = self._node(path)
        if isinstance(node, dict):
            raise IsADirectoryError(path)
        return node

    def ls(self, path):
        node = self._node(path)
        return [self._parts(path)[-1]] if isinstance(node, str) else sorted(node)

    def snapshot(self):
        self.snaps.append(self.root)
        return len(self.snaps) - 1

    def restore(self, snap_id):
        self.root = self.snaps[snap_id]

    def diff(self, a, b):
        out = {"added": [], "removed": [], "changed": []}
        def walk(x, y, prefix):
            for name in sorted(set(x) | set(y)):
                u, v, p = x.get(name), y.get(name), f"{prefix}/{name}"
                if u is v:
                    continue
                if u is None:
                    out["added"].append(p)
                elif v is None:
                    out["removed"].append(p)
                elif isinstance(u, dict) and isinstance(v, dict):
                    walk(u, v, p)
                elif u != v:
                    out["changed"].append(p)
        walk(self.snaps[a], self.snaps[b], "")
        return out

fs = FS()
fs.write("/crm/acct/42.json", '{"stage": "lead"}')
fs.mkdir("/docs")
s0 = fs.snapshot()
fs.write("/crm/acct/42.json", '{"stage": "won"}')
fs.write("/docs/q3.md", "draft")
s1 = fs.snapshot()
assert fs.diff(s0, s1) == {"added": ["/docs/q3.md"], "removed": [], "changed": ["/crm/acct/42.json"]}
fs.write("/docs/notes.md", "x")
assert fs.root["crm"] is fs.snaps[s1]["crm"]
fs.restore(s0)
assert fs.read("/crm/acct/42.json") == '{"stage": "lead"}' and fs.ls("/docs") == []
assert fs.ls("/crm/acct/42.json") == ["42.json"]
for bad in (lambda: fs.write("/crm/acct", "x"), lambda: fs.read("/nope"),
            lambda: fs.read("/crm"), lambda: fs.mkdir("/crm/acct/42.json/x")):
    try:
        bad()
        raise AssertionError("expected an error")
    except (IsADirectoryError, FileNotFoundError, NotADirectoryError):
        pass
print("q3 ok")
```

Verify: the diff lists exactly one added and one changed file, an untouched subtree is shared by identity after later writes, restore brings back old content, and the four error paths raise.

Pitfalls:
- Mutating a dict in place anywhere, which silently corrupts every snapshot that shares it.
- Treating mkdir on an existing directory as an error; most specs make it a no-op.
- Equal file contents written twice show as identical objects only by luck; compare values after the identity check.

Follow-ups: file moves, permission bits per principal for ACL tasks, and storing snapshots for 1,000 concurrent trials with bounded memory.

Why this is likely: an in-memory file system is the most frequently reported recent Perplexity phone-screen prompt in a candidate-report aggregator, and the listing asks for reproducible system state in eval environments (candidate reports, medium confidence).

</details>

<a id="q4"></a>

#### Q4. Debug this eval statistics module (no AI, as at Perplexity). A teammate says the regression gate never flags anything and pass^k looks too high. Find the bugs from the failing tests and fix them.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Eval statistics, pass^k, paired bootstrap.

<details>
<summary>Show answer</summary>

Task: Here is the module and its test output. Find every bug and fix it without breaking callers.

```python
import random

def pass_hat_k(trials, k):
    vals = []
    for outcomes in trials.values():
        p = sum(outcomes) / len(outcomes)
        vals.append(p ** k)
    return sum(vals) / len(vals)

def paired_bootstrap(scores_a, scores_b, n_boot=2000, seed=0):
    rng = random.Random(seed)
    a = [s for t in scores_a for s in scores_a[t]]
    b = [s for t in scores_b for s in scores_b[t]]
    delta = sum(b) / len(b) - sum(a) / len(a)
    boots = []
    for _ in range(n_boot):
        ra = [rng.choice(a) for _ in a]
        rb = [rng.choice(b) for _ in b]
        boots.append(sum(rb) / len(rb) - sum(ra) / len(ra))
    boots.sort()
    return delta, (boots[int(0.025 * n_boot)], boots[int(0.975 * n_boot)])
```

```text
FAIL test_pass_hat_k_unbiased     AssertionError 0.25 (expected 0.1667 for 2 of 4 passes, k=2)
FAIL test_paired_ci_excludes_zero AssertionError CI (-0.027, 0.065) crosses zero
FAIL test_uses_shared_tasks_only  AssertionError delta 0.25 (expected 0.0 on shared tasks)
failures: 3
```

What they grade: reading failing tests precisely, statistical correctness, and fixes that keep the interface.

Approach: Bug 1 is a biased pass^k. Raising the sample pass rate to the power k is biased at small n. With 2 passes in 4 trials and k = 2, the chance two trials drawn without replacement both pass is C(2,2)/C(4,2) = 1/6, while the code returns 0.25. Use C(c,k)/C(n,k) and reject tasks with fewer than k trials. Bug 2 is an unpaired bootstrap. Flattening and resampling A and B independently throws away the pairing; task difficulty varies far more than a 2 point effect, so the interval always crosses zero and the gate never fires. Resample tasks and take the difference within each task, which cancels difficulty. Bug 3 is mismatched task sets: A has a hard task B never ran, so B looks better. Compare only shared tasks and report the dropped ones. Alternatives: McNemar's test is exact and instant for binary outcomes but ignores multiple trials per task; a paired t-test on task deltas matches the bootstrap at 300 or more tasks for O(n) instead of O(n x 2000). I keep the bootstrap because it also handles skewed cost metrics.

Code:

```python
import math, random

def pass_hat_k(trials, k):
    vals = []
    for task, outcomes in trials.items():
        n, c = len(outcomes), int(sum(outcomes))
        if n < k:
            raise ValueError(f"{task}: {n} trials < k={k}")
        vals.append(math.comb(c, k) / math.comb(n, k))
    return sum(vals) / len(vals)

def paired_bootstrap(scores_a, scores_b, n_boot=2000, alpha=0.05, seed=0):
    tasks = sorted(set(scores_a) & set(scores_b))
    if len(tasks) < 2:
        raise ValueError("need at least 2 shared tasks")
    mean = lambda xs: sum(xs) / len(xs)
    diffs = [mean(scores_b[t]) - mean(scores_a[t]) for t in tasks]
    n, rng = len(diffs), random.Random(seed)
    boots = sorted(mean([diffs[rng.randrange(n)] for _ in range(n)]) for _ in range(n_boot))
    lo, hi = boots[int(alpha / 2 * n_boot)], boots[int((1 - alpha / 2) * n_boot) - 1]
    return mean(diffs), (lo, hi), sorted(set(scores_a) ^ set(scores_b))

assert abs(pass_hat_k({"t": [1, 1, 0, 0]}, 2) - 1 / 6) < 1e-9
rng = random.Random(1)
a, b = {}, {}
for i in range(300):
    d = rng.random()
    a[f"t{i}"] = [d]
    b[f"t{i}"] = [d + 0.02 + rng.gauss(0, 0.03)]
a["hard"] = [0.0]
delta, (lo, hi), dropped = paired_bootstrap(a, b)
assert 0.015 < delta < 0.025 and lo > 0 and dropped == ["hard"]
print("q4 ok", round(delta, 3), round(lo, 3), round(hi, 3))
```

Verify: the 1/6 case, a paired interval that excludes zero on a 2 point effect with large difficulty spread, and the extra task reported as dropped instead of skewing the delta.

Pitfalls:
- Averaging trials before checking n >= k, which hides tasks with too few trials.
- Reporting a delta without the list of dropped tasks, so nobody notices a broken run.

Follow-ups: one-sided non-inferiority bounds, multiple slices with a correction, and stratified resampling by task family.

Why this is likely: the team ships regression gates whose correctness rests on exactly this statistics code, and a debug task with failing tests fits Perplexity's applied hands-on coding format (official format, high; topic inferred from the listing, medium).

</details>

<a id="q5"></a>

#### Q5. Review this AI-generated LLM judge harness before it merges (no AI, as at Perplexity). List every defect, rank by severity, and write the fixed version.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: LLM judge harness, security, async.

<details>
<summary>Show answer</summary>

Task: A coding agent produced this harness. Review it like a senior reviewer, then fix it.

```python
import asyncio, json, logging
log = logging.getLogger("judge")

async def grade_all(cases, call_judge, out_dir, concurrency=8):
    async def one(case):
        raw = await call_judge(case["prompt"], case["answer"])
        log.info("judge raw=%s headers=%s", raw, case["headers"])
        verdict = eval(raw)
        with open(f"{out_dir}/{case['id']}.json", "w") as f:
            json.dump(verdict, f)
        return verdict
    results = await asyncio.gather(*(one(cases[i]) for i in range(1, len(cases))),
                                   limit=concurrency)
    scores = [r["score"] for r in results]
    return {"n": len(cases), "mean": sum(scores) / len(scores)}
```

What they grade: finding the security issues first, spotting the invented API and the off-by-one, and a fix that degrades gracefully.

Approach: Defects, most severe first:
1. `eval(raw)` runs model output as Python. The judge reads agent answers, which can carry text that steers it into emitting `__import__('os').system(...)`, so prompt injection becomes code execution. Parse JSON against a schema.
2. `case["id"]` goes straight into a path, so `../../etc/x` writes outside `out_dir`. Validate ids against an allowlist pattern.
3. Logging `headers` writes tenant auth tokens into logs. Log ids and lengths only.
4. Hallucinated API: `asyncio.gather` has no `limit` argument, so the first run raises `TypeError`. Use a semaphore.
5. Off-by-one: `range(1, len(cases))` drops the first case while `n` still claims all of them.
6. Missing edge cases: one unparseable verdict kills the batch, a score of 9 on a 1 to 5 rubric is accepted, and an all-failed batch divides by zero.

For parsing, provider structured outputs with a JSON schema remove most parse failures but tie you to one provider; regex plus `json.loads` works everywhere at the cost of a small unparseable rate that you must count. I pick the portable parser and report `parsed` next to `n`.

Code:

```python
import asyncio, json, logging, os, re, tempfile
from pathlib import Path

log = logging.getLogger("judge")
VERDICT_RE = re.compile(r"\{.*\}", re.S)
SAFE_ID = re.compile(r"^[A-Za-z0-9_-]{1,64}$")

def parse_verdict(text):
    m = VERDICT_RE.search(text or "")
    if not m:
        return None
    try:
        obj = json.loads(m.group(0))
    except json.JSONDecodeError:
        return None
    score = obj.get("score")
    if not isinstance(score, (int, float)) or not 1 <= score <= 5:
        return None
    return {"score": float(score), "reason": str(obj.get("reason", ""))[:500]}

async def grade_all(cases, call_judge, out_dir, concurrency=8):
    sem, out_dir = asyncio.Semaphore(concurrency), Path(out_dir).resolve()
    async def one(case):
        if not SAFE_ID.match(case["id"]):
            raise ValueError(f"bad case id {case['id']!r}")
        async with sem:
            raw = await call_judge(case["prompt"], case["answer"])
        v = parse_verdict(raw)
        if v is None:
            log.warning("unparseable verdict case=%s len=%d", case["id"], len(raw or ""))
        (out_dir / f"{case['id']}.json").write_text(json.dumps({"id": case["id"], "verdict": v}))
        return v
    results = await asyncio.gather(*(one(c) for c in cases))
    scored = [r["score"] for r in results if r]
    return {"n": len(cases), "parsed": len(scored),
            "mean": sum(scored) / len(scored) if scored else None}

async def fake_judge(prompt, answer):
    return {"a": 'Sure. {"score": 4, "reason": "ok"}', "b": "no json", "c": '{"score": 9}'}[answer]

cases = [{"id": f"c{i}", "prompt": "p", "answer": x, "headers": {"auth": "secret"}}
         for i, x in enumerate("abc", 1)]
with tempfile.TemporaryDirectory() as d:
    assert asyncio.run(grade_all(cases, fake_judge, d)) == {"n": 3, "parsed": 1, "mean": 4.0}
    assert sorted(os.listdir(d)) == ["c1.json", "c2.json", "c3.json"]
    try:
        asyncio.run(grade_all([{"id": "../x", "prompt": "p", "answer": "a"}], fake_judge, d))
        raise AssertionError("path traversal accepted")
    except ValueError:
        pass
assert asyncio.run(grade_all([], fake_judge, ".")) == {"n": 0, "parsed": 0, "mean": None}
print("q5 ok")
```

Verify: all three cases graded including the first, an out-of-range score and a non-JSON verdict counted as unparsed rather than crashing, a traversal id rejected, and an empty batch returning `None` instead of dividing by zero.

Pitfalls:
- Fixing only the crash (`limit=`) and shipping the `eval`.
- Averaging over parsed verdicts without flagging a parse-rate drop; a fall from 99% to 80% means the mean comes from a biased subset and the run should be marked untrusted.

Follow-ups: retries for unparseable verdicts, a judge prompt that resists injection from the graded answer, and storing raw verdicts for audit without secrets.

Why this is likely: candidate reports and Perplexity's hands-on coding page describe practical debug exercises, and the listing makes this team own automated graders; the review-of-generated-code framing is inferred rather than reported (medium confidence).

</details>

<a id="q6"></a>

#### Q6. Find the root cause in this agent trace, then write a detector that flags the same pattern across a million traces (no AI, as at Perplexity).

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Agent trajectories, loop detection.

<details>
<summary>Show answer</summary>

Task: A user rated this session thumbs down. Explain why it failed, who owns each fault, and write a streaming detector.

```text
step 0 user      "Summarize Q3 churn from the finance drive"
step 1 tool_call drive.search {"q": "Q3 churn"}         200  hits=[f_91 "Q3 churn final.xlsx"]
step 2 tool_call drive.read   {"id": "f_91"}            403  {"error": "forbidden"}
step 3 thought   "Read failed, retrying"
step 4 tool_call drive.read   {"id": "f_91"}            403  {"error": "forbidden"}
step 5 tool_call drive.read   {"id": "f_91"}            403  {"error": "forbidden"}
step 6 tool_call web.search   {"q": "Q3 churn"}         200  10 public results
step 7 final     "Q3 churn was 4.1%"
tokens_in=48210 tokens_out=612 latency_ms=21400
principal=analyst_7 acl(f_91)=[finance_leads]
```

What they grade: a precise root cause, separating faults by owner, turning the failure into an eval task, and a detector that runs cheaply at scale.

Approach: Search returns a file the principal cannot read, because the search connector does not filter by ACL. The 403 body is a bare "forbidden" with no hint, so the agent treats it as transient and retries the identical call twice. Then it falls back to public web search and states an unrelated public number as the tenant's figure. Each of the three faults has a different owner: the connector (search leaks titles of unreadable files), the tool result (a non-actionable error), and the agent policy (no refusal path, unsourced claim). The user-visible harm is the confident wrong answer, so the eval task I add has the success predicate "tells the user they lack access to f_91 and makes no numeric claim". For the detector, an LLM judge over every trace catches subtler failures, but at about 50k tokens per trace that is roughly 50B tokens for a million traces. Cheap rule detectors run first, and the judge sees only flagged and sampled traces.

Code:

```python
import hashlib, json
from collections import Counter

def call_key(ev):
    args = json.dumps(ev.get("args", {}), sort_keys=True)
    return ev["tool"] + ":" + hashlib.sha1(args.encode()).hexdigest()[:12]

def find_loops(events, max_repeats=3, window=8):
    findings, recent = [], []
    for ev in events:
        if ev.get("type") != "tool_call":
            continue
        recent.append((call_key(ev), ev.get("status")))
        recent = recent[-window:]
        key, n = Counter(k for k, _ in recent).most_common(1)[0]
        if n >= max_repeats:
            errs = [s for k, s in recent if k == key and s and s >= 400]
            findings.append({"step": ev["step"], "key": key.split(":")[0], "repeats": n,
                             "kind": "error_retry_loop" if len(errs) == n else "repeat_loop"})
            recent = [r for r in recent if r[0] != key]
    return findings

trace = [
    {"step": 1, "type": "tool_call", "tool": "drive.search", "args": {"q": "Q3 churn"}, "status": 200},
    {"step": 2, "type": "tool_call", "tool": "drive.read", "args": {"id": "f_91"}, "status": 403},
    {"step": 3, "type": "thought"},
    {"step": 4, "type": "tool_call", "tool": "drive.read", "args": {"id": "f_91"}, "status": 403},
    {"step": 5, "type": "tool_call", "tool": "drive.read", "args": {"id": "f_91"}, "status": 403},
    {"step": 6, "type": "tool_call", "tool": "web.search", "args": {"q": "Q3 churn"}, "status": 200},
]
assert find_loops(trace) == [{"step": 5, "key": "drive.read", "repeats": 3, "kind": "error_retry_loop"}]
ok = [{"step": i, "type": "tool_call", "tool": "drive.read", "args": {"id": f"f_{i}"}, "status": 200}
      for i in range(20)]
assert find_loops(ok) == []
print("q6 ok")
```

Verify: exactly one `error_retry_loop` at step 5 on this trace, nothing on 20 distinct successful reads, and argument order not affecting the key because of `sort_keys`. It is O(steps x window) per trace and streams, so it runs as a map job over the trace store.

Pitfalls:
- Blaming only the model; two of the three fixes are in the connector and the tool result.
- Keying loops on tool name alone, which flags legitimate pagination.

Follow-ups: a second detector that flags a final answer containing numbers absent from every successful tool result, and an actionable 403 message that names who can grant access.

Why this is likely: the listing asks you to analyze agent trajectories and production failures and find root causes, and candidate reports describe debugging rounds built on logs (listing plus candidate reports, medium-high confidence).

</details>

<a id="q7"></a>

#### Q7. Implement a block-level prefix cache for prompt KV state (no AI, as at Perplexity): requests sharing a token prefix reuse blocks, with LRU eviction and tenant isolation.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Prefix caching, LRU, tenant isolation.

<details>
<summary>Show answer</summary>

Task: `insert(tokens, salt)` stores blocks for a processed prompt; `lookup(tokens, salt)` returns how many leading tokens are cached. Fixed `block_size` (vLLM defaults to 16), only full blocks cached, capacity `max_blocks` with LRU eviction, and a block is identified by its own tokens plus everything before it.

What they grade: why block keys must encode the whole prefix, correct LRU behavior, isolation between tenants, and complexity.

Approach: Hashes are chained. Hashing a block alone would let block B after prefix A collide with B after prefix X, but their KV values differ because attention at every position depends on everything before it. Chaining makes each key encode the full prefix, so a flat ordered dict replaces a trie, which mirrors vLLM automatic prefix caching (it also folds LoRA ids and a cache salt into the key). Lookup and insert are O(L / block_size) dict operations after O(L) hashing, and memory is O(max_blocks). I considered two alternatives. A radix tree (SGLang RadixAttention) matches at token granularity, so a 1,000 token shared prefix reuses all 1,000 tokens instead of 992, at the cost of pointer-heavy nodes and harder eviction. Hashing whole prompts is simplest but gives zero reuse across agent turns that append to the same history, where block chaining reuses nearly all of it. For agent loops I pick the chained blocks.

Code:

```python
import hashlib
from collections import OrderedDict

class PrefixCache:
    def __init__(self, block_size=16, max_blocks=1024):
        self.bs, self.max_blocks = block_size, max_blocks
        self.blocks = OrderedDict()

    def _hashes(self, tokens, salt=""):
        h, out = hashlib.sha256(salt.encode()).hexdigest(), []
        full = len(tokens) - len(tokens) % self.bs
        for i in range(0, full, self.bs):
            block = ",".join(map(str, tokens[i:i + self.bs]))
            h = hashlib.sha256(f"{h}|{block}".encode()).hexdigest()
            out.append(h)
        return out

    def lookup(self, tokens, salt=""):
        hit = 0
        for h in self._hashes(tokens, salt):
            if h not in self.blocks:
                break
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
                    self.blocks.popitem(last=False)

c = PrefixCache(block_size=4, max_blocks=3)
prompt = list(range(10))
c.insert(prompt, "tenant_a")
assert c.lookup(prompt, "tenant_a") == 8
assert c.lookup(prompt + [99, 98], "tenant_a") == 8
assert c.lookup(prompt, "tenant_b") == 0
assert c.lookup([7] + prompt[1:], "tenant_a") == 0
c.insert(list(range(100, 108)), "tenant_a")
assert len(c.blocks) == 3 and c.lookup(prompt, "tenant_a") == 0
print("q7 ok")
```

Verify: 8 of 10 tokens hit (only full blocks), an appended turn still hits its prefix, another tenant's salt misses, one changed first token misses everything, and inserting new blocks past capacity evicts the least recently used ones.

Pitfalls:
- A timestamp near the top of the prompt misses every later block (see Q14).
- No salt turns hit latency into a cross-tenant timing side channel.
- Evicting a parent while a child stays cached leaves an unreachable child; production engines evict leaves first and pin blocks used by running requests.

Follow-ups: reference counts, hit rate for a 12 turn agent loop, and cache-aware routing of a session to the replica that holds its prefix.

Why this is likely: the listing names caching and context management as efficiency levers, and a prefix cache is the data structure that maps most directly onto inference cost (listing-derived, medium confidence).

</details>

### Agentic coding (AI assistant allowed)

<a id="q8"></a>

#### Q8. Agentic coding round (AI assistant allowed): add a `gate` command to this eval harness repo that compares a candidate run with a baseline per slice, blocks on hard invariant failures, and blocks when a slice regresses beyond a margin. Show how you drive the agent.

Round: Agentic coding. AI policy: Allowed. Type: Build. Difficulty: Hard. Topic: Regression gate, per-slice statistics.

<details>
<summary>Show answer</summary>

Task: Implement the spec in this repo with a coding agent, and show your delegation, review and testing.

```text
evalkit/
  pyproject.toml
  evalkit/__init__.py
  evalkit/cli.py            # argparse; subcommands: run, report
  evalkit/store.py          # load_run(path) -> {task_id: score}, load_meta(path)
  evalkit/stats.py          # paired_bootstrap(a, b) from Q4
  evalkit/slices.py         # task_slice(task_id) -> "crm" | "docs" | ...
  evalkit/invariants.py     # failed_invariants(run) -> set[str]
  tests/test_stats.py
  tests/test_cli.py
  tests/fixtures/base.jsonl
  tests/fixtures/cand.jsonl
Spec: evalkit gate BASE CAND [--margin -0.02] exits 1 if any invariant failed
or any slice with >= 20 tasks has a lower CI bound below margin, Bonferroni
across slices; slices under 20 tasks are reported, never blocking.
```

What they grade: scope and quality above the hands-on bar, how you split work for the agent, what you check in its diffs, and your own tests.

Approach: I read `cli.py`, `store.py` and `stats.py` myself first, because prompts that name real interfaces get better code. Prompt 1: "Write `evalkit/gate.py` with `gate(base, cand, task_slice, margin, invariants_failed)` returning per-slice verdicts. Paired deltas on shared tasks only; bootstrap within each slice; one-sided Bonferroni alpha 0.05/m; slices under 20 tasks never block. Write `tests/test_gate.py` first: a regressed slice, a clean slice, an invariant failure, a tiny slice." Prompt 2: "Wire `gate` into `cli.py` with exit code 1 on block, plus a CLI test on the fixtures." Small prompts keep each diff reviewable. In the diff I check task intersection, alpha divided by the slice count, a one-sided bound, invariants short-circuiting before statistics, and unchanged `stats.py` signatures. I caught one wrong output. The agent's first version ran one bootstrap over all tasks and split the resampled means by slice, so a flat docs slice diluted a 10 point CRM drop and CRM passed. Its own test checked only the overall delta. I added a per-slice regression test, which failed, and had the agent bootstrap within each slice. Alternatives: a single pooled test is faster but hides slice regressions, as shown; Benjamini-Hochberg blocks more true regressions across 20 slices but tolerates about 5% false blocks among flagged slices.

Code:

```python
import random
from dataclasses import dataclass

@dataclass
class Verdict:
    slice: str
    delta: float
    lo: float
    blocked: bool
    reason: str

def gate(base, cand, task_slice, margin=-0.02, invariants_failed=(), n_boot=2000, seed=0):
    if invariants_failed:
        return [Verdict("*", 0.0, 0.0, True, f"invariant: {sorted(invariants_failed)[0]}")]
    groups = {}
    for t in sorted(set(base) & set(cand)):
        groups.setdefault(task_slice[t], []).append(cand[t] - base[t])
    rng, m, out = random.Random(seed), len(groups), []
    for name, d in sorted(groups.items()):
        n, mean = len(d), sum(d) / len(d)
        if n < 20:
            out.append(Verdict(name, mean, float("nan"), False, "too few tasks; report only"))
            continue
        boots = sorted(sum(d[rng.randrange(n)] for _ in range(n)) / n for _ in range(n_boot))
        lo = boots[int(0.05 / m * n_boot)]
        out.append(Verdict(name, mean, lo, lo < margin, "regressed" if lo < margin else "ok"))
    return out

rng = random.Random(7)
base, cand, sl = {}, {}, {}
for i in range(310):
    t = f"t{i}"
    sl[t] = "crm" if i < 150 else "docs" if i < 300 else "tiny"
    base[t] = rng.random()
    shift = -0.10 if sl[t] == "crm" else 0.0
    cand[t] = base[t] + shift + rng.gauss(0, 0.05)
v = {x.slice: x for x in gate(base, cand, sl)}
assert v["crm"].blocked and v["crm"].lo < -0.08
assert not v["docs"].blocked and not v["tiny"].blocked
assert gate(base, cand, sl, invariants_failed={"acl_leak"})[0].blocked
print("q8 ok", round(v["crm"].lo, 3), round(v["docs"].lo, 3))
```

Verify: `pytest -q tests/`, the CLI on the fixtures, then a mutation check where I regress one fixture slice by 10 points and confirm exit code 1. The self-test above shows CRM blocking near minus 0.10 while docs passes and a 10 task slice only reports.

Pitfalls:
- Accepting a green test suite the agent wrote without reading what it asserts.
- Letting the agent touch `stats.py` signatures other callers depend on.

Follow-ups: cost and latency margins next to success, and a JSON report for CI comments.

Why this is likely: Perplexity's agentic coding round requires frontier coding agents on a multi-file task, and a regression gate is the artifact this team ships (official round format, high confidence; low confidence that this role includes the round).

</details>

### System design

<a id="q9"></a>

#### Q9. Design a reproducible evaluation environment for enterprise agents with connected apps (mail, docs, tickets, CRM), realistic data, roles and permissions, and multi-step workflows.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Eval environments, connector fakes, snapshots.

<details>
<summary>Show answer</summary>

Task: Every trial starts from identical state in isolation, sees production tool schemas, and is graded on end state, at a scale where a full suite finishes in under 2 hours.

What they grade: framing what "reproducible" means, a fidelity versus cost decision with numbers, permission handling, and grading on state instead of claims.

Approach: The design has six components.
1. Connector fakes behind production interfaces: each app is a stateful service with the real tool schema. Contract tests replay recorded real calls against each fake on every connector release to catch drift.
2. A world generator that builds an org, documents, threads and tickets from seeds, injecting near-duplicate names, stale docs with newer replacements, long threads and partial access.
3. Identity and ACLs enforced inside the fakes, with tasks whose correct outcome is a refusal, like the 403 case in Q6.
4. Snapshot and restore: each world is a versioned database dump plus blob store, cloned copy-on-write per trial (the Q3 idea at scale), with a frozen clock and a seeded user simulator.
5. Task specs: instruction, principal, world version, success predicates over end state, forbidden actions, optional rubric.
6. A runner and trace store logging every tool call, tokens, latency and the world diff.

Numbers: about 5k tasks run 3 to 8 times for pass^k is 20k to 40k trials per suite. At 3 minutes per trial, finishing in 2 hours needs 750 to 1,000 trials in parallel, so world clones must be cheap: copy-on-write makes a clone milliseconds, while a real staging tenant reset takes 30 to 60 seconds.

Alternatives: high-fidelity fakes cost about one engineer-week per connector plus upkeep but support any trajectory. Record and replay costs hours per connector but breaks on unseen calls; in practice 20% to 40% of new trajectories fall off the recording. Real staging tenants are the most faithful but slow and leak state between trials. I use fakes for the five core apps, replay for the long tail, and a nightly canary of 100 tasks on real staging tenants to measure fake drift as a pass-rate gap.

Risks: nondeterminism (freeze what you can, measure the rest with repeated trials); graders that trust the agent's own claims, so check state diffs instead; and broken tasks, so a human solves a sample and any task no model passes gets reviewed.

Why this is likely: building realistic enterprise evaluation environments with connected applications, roles, permissions and reproducible state is the first responsibility in the listing, and Perplexity's system design guide says questions cover the team's domain (listing plus official format, high confidence).

</details>

<a id="q10"></a>

#### Q10. Design the regression gate that decides whether a model, connector, tool, skill or harness change can ship.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Regression gates, statistical power, eval cost.

<details>
<summary>Show answer</summary>

Task: Changes land daily from several teams. The gate says whether anything got worse beyond noise, on which slices, and at what cost, in under 30 minutes for normal PRs, while keeping false blocks rare so engineers keep trusting it.

What they grade: tiering by cost, a decision rule with stated power, grader health, and how overrides work.

Approach:
1. Tiered suites. Smoke: 100 tasks, 1 trial, about 10 minutes on every PR. Gate: 1.5k tasks, 3 trials, before merge or a model swap. Full: all tasks with 5 or more trials nightly. Each change type maps to the slices it can affect.
2. A pinned baseline on the same task versions, re-run when worlds or graders change.
3. Per-slice decisions: paired bootstrap lower bound against a non-inferiority margin, such as minus 2 points on success, plus 10% on p95 latency or plus 10% on cost per successful task (Q4 and Q8 code).
4. Hard invariants with zero tolerance: data sent outside an ACL, destructive calls without confirmation, schema violations.
5. Grader health: re-score a fixed calibration set each run; if judge agreement with humans drops (Q13), the result is untrusted and cannot pass.
6. Output: deltas, intervals, newly failing tasks with trace links, a cost table, and logged overrides with a named owner.

Numbers: with 1.5k tasks and a per-task delta standard deviation near 0.25, the standard error is about 0.65 points, so the gate reliably detects a 2 point drop, while the 100 task smoke tier only detects drops near 8 points. Budget: 1.5k tasks x 3 trials x 40k tokens is 180M tokens per gate run, so shared-prefix caching (Q14) matters.

Alternatives: a fixed threshold on the raw mean ("block if success falls 0.5 points") is simple but at this noise level blocks about one in five clean PRs. Sequential testing stops early on clear wins or losses and cuts average cost by roughly 40%, but is harder to explain to gated teams. I start with the fixed-sample bootstrap and add sequential stopping once trust is earned.

Risks: flaky tasks (track per-task flip rate with no code change and quarantine above 20%), saturation (rotate in fresh production failures), and slices too small to test, which are reported but never block.

Follow-ups: how to gate a change that improves success but raises cost, and who may override.

Why this is likely: the listing asks for regression gates and launch-readiness signals for model, connector, tool, skill and harness changes, and Perplexity's agent evaluation post describes regression checks (listing plus company blog, high confidence).

</details>

<a id="q11"></a>

#### Q11. Design a model routing and effort-selection system for enterprise agent tasks that minimizes cost per successful task without losing quality.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Model routing, cascades, cost per successful task.

<details>
<summary>Show answer</summary>

Task: Several model families, each with effort levels. The router chooses per task, maybe per step. Targets: success within a non-inferiority margin of an all-frontier baseline, lower cost per successful task, p95 latency budgets per surface, and routing overhead under 50 ms.

What they grade: the objective stated precisely, label sources, an escalation design, and awareness of agent-specific costs such as cache loss.

Approach:
1. Features: request embedding, a task-type classifier, connectors in scope, expected horizon, tenant tier, history so far.
2. A router that predicts success, cost and latency per option and picks the cheapest option clearing a success threshold, or maximizes success minus lambda x cost with lambda tuned on the eval set.
3. Escalation: start cheap and escalate when a verifier, a schema error, an empty search or the loop detector (Q6) says the step failed.
4. Labels from running the eval suite with every candidate model, plus production traces scored by calibrated judges.
5. Shadow mode, then an A/B on success, retries and user-visible failures, with a per-tenant kill switch.

Numbers: a learned router such as RouteLLM's matrix factorization reports cost reductions above 2x on public benchmarks at near-equal quality, with a few milliseconds of overhead. In a cascade where 30% of tasks escalate, those tasks pay the cheap model's full runtime on top, so their latency rises accordingly. With a 40k token history, switching models mid-trajectory discards the prompt cache, and re-prefilling 40k tokens can cost more than the cheaper model saves.

Alternatives: static rules by task type cost nothing to serve and are easy to audit, but capture only part of the savings. A cascade like FrugalGPT needs no trained router but adds a second call on every escalation. A learned router saves most but needs labels and drifts. I start with a cascade on the two highest-volume task types, because escalation signals in agent loops are cheap and reliable, then train a router once labels accumulate.

Risks: router calibration drifts when providers update models; different models need different tool descriptions and skills, so every route needs eval coverage; route at task start and subagent boundaries to keep caches warm. Measure against an oracle that picks the cheapest model that succeeded, which shows the headroom left.

Follow-ups: per-step effort selection inside one model, and how routing interacts with the regression gate.

Why this is likely: the listing asks you to build and evaluate model selection and routing strategies that match each task to the right model and effort level (listing, high confidence for the topic).

</details>

<a id="q12"></a>

#### Q12. Design how long-running enterprise agent tasks (10 minutes to 2 hours) execute reliably: survive worker crashes and deploys, stream progress, enforce budgets, support cancel, and leave a trajectory the eval team can replay.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Durable execution, idempotency, trajectory replay.

<details>
<summary>Show answer</summary>

Task: Production and eval runs share one executor. A task makes 40 or more calls, some with side effects such as sending mail.

What they grade: durability without duplicated side effects, budget enforcement, a progress channel that survives reconnects, and how the design feeds evals.

Approach: I use durable execution. Each task is a workflow record, and the agent loop is a sequence of steps. Before acting on a step's result, the worker appends it to an event log keyed by `(task_id, step_no)`. If a worker dies, another takes its lease (30 second heartbeat), replays the log without calling any model again, and resumes. Side-effecting tool calls carry an idempotency key of `(task_id, step_no)`, so a retried step cannot send the same email twice. Before each step it checks token, dollar, time and step budgets; on breach it writes a summary and stops. Context compaction writes its summary as an event too, so replay stays deterministic. Progress events go out over server-sent events, and a client reconnects with its last event id. Cancel is a flag checked between steps. The same log is the trajectory store: Q6 detectors run on it, graders re-score without re-running, and engineers fork a trajectory at step k to test a harness change.

Numbers: 50k concurrent tasks with a step every 15 seconds is about 3.3k event writes per second, comfortable for a partitioned database or a log plus key-value store. At about 20 KB per event (large tool outputs go to blob storage), 40 steps per task is 40 GB of hot log per 50k-task wave. Recovery replays 40 events in under a second, against about 200 seconds and real spend to re-run 40 model calls.

Alternatives: an in-memory loop with retries is simplest, but with one deploy a day a 2 hour task has about a 2 in 24, or 8%, chance of being killed. A workflow engine such as Temporal provides replay, timers and retries, but imposes determinism rules and history size limits. A custom event log on an existing database is easy to own. I use the engine if the company already runs one, otherwise the event log; both give the same semantics.

Risks: code changes between crash and resume break replay, so version the workflow; stale data after an hour, so re-read before writes; runaway loops, so run the Q6 detector online and kill on repeat.

Why this is likely: a candidate-report aggregator lists a recent Perplexity onsite system design prompt on long-running agentic query execution, and the listing names reliability, latency and cost as this team's targets (candidate reports, medium confidence).

</details>

### Core expertise: graders, cost and tools

<a id="q13"></a>

#### Q13. Design the calibration study for an LLM judge that grades whether an enterprise task was completed, and decide what numbers would let it gate launches.

Round: Core expertise. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: LLM judge calibration, agreement, bias correction.

<details>
<summary>Show answer</summary>

Task: Choose the labeling plan, the metrics, and a written rule for when the judge's verdicts can block or pass a launch.

What they grade: sound labeling design, knowing why accuracy misleads, correcting the reported pass rate for judge error, and a concrete gate rule.

Approach: Sample 300 trajectories stratified by task type and by the judge's verdict, so rare failures are covered. Two domain experts label each one blind to the judge, because annotators shown the verdict anchor on it. Write the rubric first and adjudicate disagreements, which exposes ambiguous criteria. Human-human Cohen's kappa is the ceiling: if two experts agree at 0.62, a judge at 0.60 with each behaves like a third expert, and below about 0.6 between humans the rubric needs fixing before the judge. Accuracy misleads at skewed prevalence: if 90% of runs pass, an always-pass judge scores 90% accuracy and kappa 0. The judge's true positive rate s and true negative rate t both bias the reported pass rate; the corrected rate is (p_obs + t - 1) / (s + t - 1). With s = 0.95, t = 0.80 and an observed 0.70, the true rate is 0.667, a 3.3 point overstatement. To separate grader variance from agent variance, grade the same saved trajectory 5 times.

Code:

```python
def cohen_kappa(a, b):
    n = len(a)
    po = sum(x == y for x, y in zip(a, b)) / n
    pe = sum((a.count(l) / n) * (b.count(l) / n) for l in set(a) | set(b))
    return 1.0 if pe == 1 else (po - pe) / (1 - pe)

def corrected_rate(p_obs, tpr, tnr):
    return (p_obs + tnr - 1) / (tpr + tnr - 1)

human = [1] * 9 + [0]
always_pass = [1] * 10
assert abs(cohen_kappa(human, always_pass)) < 1e-9
assert abs(cohen_kappa(human, human) - 1.0) < 1e-9
assert abs(corrected_rate(0.70, 0.95, 0.80) - 0.6667) < 1e-3
print("q13 ok")
```

Verify: an always-pass judge gets kappa 0 despite 90% accuracy, identical labels get 1, and the worked correction returns 0.667.

Pitfalls:
- One holistic 1 to 10 score; it drifts and cannot be calibrated per criterion.
- Re-calibrating only when someone complains instead of on every judge model or prompt change.

Alternatives: code checks on end state are exact and nearly free but cover only verifiable criteria; a single holistic judge score is cheap but hard to calibrate; narrow binary criteria judged separately cost about 3 to 5 times more judge tokens but give per-criterion kappa you can act on. I use code checks wherever possible and binary judged criteria for the rest. Gate rule: the judge may gate when judge-human kappa is within 0.05 of human-human kappa, the true negative rate on safety and permission criteria is at least 0.95, and repeat-grading flips stay under 3%.

Why this is likely: the listing says graders must be calibrated against human judgment, and Perplexity's agent evaluation post describes human calibration and judge consistency checks (listing plus company blog, high confidence).

</details>

<a id="q14"></a>

#### Q14. An agent harness puts the current time and user name at the top of a 30k token system prompt. Sessions average 12 model calls. Estimate the cost of this choice, choose a fix, and pick a cache lifetime.

Round: Core expertise. AI policy: Not allowed. Type: Optimize. Difficulty: Medium. Topic: Prompt caching, context management, cost.

<details>
<summary>Show answer</summary>

Task: Quantify the waste, propose a fix with an alternative, and choose between a short and a long cache lifetime.

What they grade: knowing that cache matching is exact and prefix-only, a clean estimate with stated assumptions, and a decision tied to call spacing.

Approach: Prefill computes KV state for every prompt token, and a prompt cache reuses it only for an identical prefix. A timestamp in the first line changes every call's prefix, so nothing after it is reused. Assume a stable 30k prefix (system prompt, tools, skill index) and history growing by about 2k tokens per call, so call i has 30k + 2k x i input tokens. Over 12 calls that is about 516k input tokens, all billed at the full rate today. With the time moved to the end, call 1 writes the cache and calls 2 to 12 read their prefix. Using Anthropic-style multipliers of 1.25x for a 5 minute write and 0.1x for reads, and treating each call's new 2k tokens as written, the session costs about 114k token-equivalents: roughly 78% cheaper on input, with lower time to first token.

Code:

```python
def session_cost(prefix=30_000, step=2_000, calls=12, write=1.25, read=0.10, cached=True):
    total = 0.0
    for i in range(1, calls + 1):
        n = prefix + step * i
        if not cached:
            total += n
        elif i == 1:
            total += n * write
        else:
            total += (n - step) * read + step * write
    return total

full, cached = session_cost(cached=False), session_cost()
assert full == 516_000 and round(cached) == 113_700
assert round(1 - cached / full, 2) == 0.78
print("q14 ok", full, round(cached))
```

Verify: 516k uncached, about 113.7k cached, 78% saved. Re-run with your provider's real multipliers before quoting a number.

Alternatives: option A moves the time and user fields to the last user message; it is free and keeps the whole prefix stable. Option B removes the time and gives the agent a clock tool; also stable, but adds a tool call in the maybe 5% of sessions where time matters. I pick A. On lifetime, a 5 minute write breaks even on its second use, while a 1 hour write costs 2x and needs three uses. Agent calls are seconds apart, so 5 minutes wins inside a session; the 1 hour cache pays only for prefixes shared across sessions, such as the system prompt across a tenant's users.

Pitfalls:
- Tool lists in nondeterministic order, which silently breaks the prefix just like the timestamp.
- Not tracking a cached-token fraction metric in the gate, so a harmless-looking prompt edit doubles input cost unnoticed.

Why this is likely: the listing names caching and context management as efficiency levers and cost per successful task as a target, so an estimate-and-choose cache question is a natural core expertise probe (listing-derived, medium confidence).

</details>

<a id="q15"></a>

#### Q15. Config A uses a frontier model at high effort; config B uses a mid model with escalation. Given the run summary, decide which ships, using cost per successful task with uncertainty.

Round: Core expertise. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Cost per successful task, non-inferiority, slices.

<details>
<summary>Show answer</summary>

Task: Paired run on 400 tasks:

```text
config  success  mean_cost/task  p95_cost  retries  p95_latency_s
A       0.81     0.42            1.10      0.05     38
B       0.79     0.17            0.95      0.22     41
paired delta B-A success: -0.020, 95% CI [-0.041, +0.001]
slice long_horizon (n=60): A 0.70, B 0.58, delta CI [-0.21, -0.03]
```

What they grade: computing the metric correctly, treating quality as a constraint, reading slices, and handling uncertainty in a ratio.

Approach: Cost per successful task (CPST) is total spend on all attempts, failures and retries included, divided by successes. A is 0.42 / 0.81 = 0.52 units; B is 0.17 / 0.79 = 0.215, about 58% cheaper per success. CPST is a ratio of two noisy means and agent cost is heavy-tailed, so I bootstrap over tasks, resampling the same tasks for both configs; the delta method is a cross-check but understates the tail when a few loops cost 20x the median. The overall success interval of minus 4.1 to plus 0.1 points sits inside a minus 5 point margin but not a minus 2 point one, so the answer depends on the margin agreed before the run, and I would say which I used. The long-horizon slice fails clearly, because B loses 12 points there. I would split the traffic, sending short tasks to B and long-horizon ones to A.

Code:

```python
def cpst(mean_cost, success):
    return mean_cost / success

a, b = cpst(0.42, 0.81), cpst(0.17, 0.79)
long_share = 60 / 400
b_short_success = (0.79 - long_share * 0.58) / (1 - long_share)
blend_cost = (1 - long_share) * 0.17 + long_share * 0.42
blend_success = (1 - long_share) * b_short_success + long_share * 0.70
blend = blend_cost / blend_success
assert round(a, 3) == 0.519 and round(b, 3) == 0.215 and abs((1 - b / a) - 0.585) < 0.005
assert 0.25 < blend < 0.27
print("q15 ok", round(a, 3), round(b, 3), round(blend, 3))
```

Verify: the split gives a blended CPST near 0.26 with no slice regression, assuming long-horizon tasks cost the mean per task; real per-slice costs will push it higher, so recompute with them.

Alternatives: ship A everywhere, safe but about 2x the cost; ship B everywhere, cheapest but a real regression on the hardest, highest-value tasks.

Pitfalls:
- Comparing configs with different cache warmth or different judges.
- Ignoring B's 0.22 retry rate; run the Q6 detector over its traces to explain it before trusting the cost.

Why this is likely: the listing names cost per successful task as a metric the team moves and asks for sound quality, latency and cost tradeoff judgment (listing, high confidence for the topic).

</details>

<a id="q16"></a>

#### Q16. Design the eval suite that tells you whether a rewrite of tool descriptions and skill loading improved tool selection, and how you would avoid breaking tools you did not touch.

Round: Core expertise or system design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Tool selection evals, skills, regression.

<details>
<summary>Show answer</summary>

Task: Measure selection quality directly, connect it to end-to-end success, and protect untouched tools.

What they grade: treating selection as classification, a clear ship rule, cost-aware tiering, and concrete description fixes.

Approach:
1. Selection set: for each tool or skill, 20 to 40 positive queries where it should be chosen, negatives drawn from adjacent tools, and forbidden cases. Perplexity describes skill suites that check precision, recall and forbidden loads, plus whether the agent reads accessory files after loading. Track the full confusion matrix; most bugs are two tools with overlapping descriptions.
2. Call correctness: schema-valid arguments, semantically right arguments (correct user id, date range), and only the calls needed to reach the goal.
3. End-to-end tasks with the full loop and a rubric judge, to confirm that a selection gain also improves task success.
4. Every orchestrator model separately, because model families react differently to the same description.

Numbers: a selection-only suite of 40 queries x 50 tools is 2k cheap single-turn calls, under a minute, and points at the exact confused pair. Seeing a 2 point change end to end needs about 1.5k tasks and blames nothing specific. Ship rule: macro recall rises with a paired interval above zero, forbidden-load rate is flat or lower, calls per successful task rise by at most 5%, and every tool's recall stays within 5 points of baseline.

Alternatives: end-to-end only is faithful but slow and vague; selection only is fast and precise but misses argument errors. I run selection on every PR and end-to-end in the nightly tier.

Risks: adding one tool can steal selections from an untouched one, so every addition re-runs the full selection suite and reports per-tool recall deltas next to the average. Fixes that usually move the numbers: descriptions written as routing triggers ("use when the user asks about X; not for Y"), merging overlapping tools into one with filters, enums instead of free strings, concise paginated responses, and actionable errors. In the Q6 trace a bare "forbidden" caused three wasted calls; "no access to f_91; ask an owner in finance_leads" ends the loop in one.

Why this is likely: the listing asks you to improve tool descriptions, schemas and result handling so agents select the right tools and recover with fewer calls, and Perplexity's agent skills post describes these suites (listing plus company blog, high confidence).

</details>

### Behavioral

<a id="q17"></a>

#### Q17. Tell me how you use coding agents day to day, including a time an agent's output looked right but was wrong, how you caught it, and what you changed in how you check agent work. Tie it to a project where you defined a metric across teams.

Round: Hiring manager deep dive. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI-assisted engineering, verification, cross-team metrics.

<details>
<summary>Show answer</summary>

Task: Show real speed from agents and a systematic way of checking their work, inside one past project with measurable impact.

What they grade: specificity, ownership of correctness, and leading an ambiguous cross-team effort to a measured result.

Approach: A strong structure has six parts.
1. Your workflow in concrete terms: what you delegate (test scaffolding, migrations, boilerplate, first drafts of analysis scripts), what you keep (metric definitions, security-sensitive parsing, statistical decision rules), and how you size prompts so each diff reviews in a few minutes.
2. The miss: a specific case, such as an agent-written eval aggregation that averaged trials before tasks and moved a headline number by 3 points, or a judge parser that used `eval`. Say how long it went unnoticed and what it cost.
3. How you caught it: a test with hand-computed answers, a mutation check, reading 20 raw cases, or a number that disagreed with intuition.
4. What you changed: tests first with known expected values, a checklist for generated code (invented APIs, unvalidated input, silent drops), and a rule that metric code gets a second human reviewer.
5. The cross-team link: the metric you defined with product, research and platform, how you got agreement (a one-page spec and 100 jointly labeled cases), and the measured result with uncertainty, such as success up 11 points with a paired interval of plus 7 to plus 15 and cost per success down 30%.
6. What persisted: the gate or checklist other teams adopted, or someone you mentored who now owns it.

Verify: before the interview, check every number in the story against your own notes, and be ready to say who wrote the tests when you claim the tests passed.

Pitfalls:
- A story where the agent was wrong and nobody can tell how it was caught.
- Claiming agents are not part of your work; Perplexity's careers guide says knowledge of how to use AI is required.
- Only one story; keep a backup about an eval of yours that was wrong.

Follow-ups: what you would never delegate to an agent, and how you would review a teammate's agent-written metric code.

Why this is likely: Perplexity's careers interview guide says knowledge of how to use AI is required, its agentic coding round scores agent use when included, and candidates report a long hiring manager deep dive on past projects (official guide plus candidate reports, medium-high confidence).

</details>

## Prep checklist

- [ ] Solve Perplexity's public todo list practice exercise in Python under 60 minutes without AI, keeping earlier parts' tests green.
- [ ] Build the Q3 in-memory file system from a blank file in 45 minutes, then add snapshots without breaking part 1.
- [ ] Write the Q1 runner and Q2 provider pool from memory, then the heap variant for 1e5 providers.
- [ ] Hand-compute unbiased pass^k for a few tasks and code a paired bootstrap that intersects task sets.
- [ ] Review generated code for invented APIs, off-by-ones, missing edge cases and injection, using Q5 as a template.
- [ ] Debug three agent traces by hand and write a loop detector that streams over JSONL.
- [ ] Do one timed multi-file task with a coding agent, writing tests first and logging each wrong output you caught.
- [ ] Compute Cohen's kappa, TPR, TNR and a prevalence-corrected pass rate on a small labeled set.
- [ ] Practice 45 minute designs for an eval environment, a regression gate, a cost-aware router and durable long-running execution, each with numbers.
- [ ] Redo the Q14 cache estimate with your provider's real multipliers.
- [ ] Read Perplexity's posts on agent evaluation and agent skills, plus the tau-bench paper and Anthropic's agent eval and tool-writing guides.
- [ ] Prepare a 90 minute deep dive on one past system and a behavioral story with numbers about catching a wrong agent output.

## Sources

- https://interviewresources.perplexity.ai
- https://interviewresources.perplexity.ai/hands-on-coding
- https://interviewresources.perplexity.ai/hands-on-coding/examples/todo-list/
- https://interviewresources.perplexity.ai/practical-assessment/
- https://interviewresources.perplexity.ai/agentic-coding/
- https://interviewresources.perplexity.ai/spike-round/instructions/
- https://interviewresources.perplexity.ai/system-design/
- https://interviewresources.perplexity.ai/core-expertise/
- https://www.perplexity.ai/hub/careers/interview-guide
- https://www.perplexity.ai/hub/blog/ai-agent-evaluation
- https://www.perplexity.ai/hub/blog/designing-refining-and-maintaining-agent-skills-at-perplexity
- https://www.glassdoor.com/Interview/Perplexity-AI-Software-Engineer-Interview-Questions-EI_IE8515634.0,13_KO14,31.htm
- https://www.glassdoor.com/Interview/Perplexity-AI-Member-Of-Technical-Staff-Interview-Questions-EI_IE8515634.0,13_KO14,39.htm
- https://www.reddit.com/r/Hack2Hire/comments/1us3j19/perplexity_interview_process_experience
- https://darkinterview.com/collections/perplexity
- https://prachub.com/resources/perplexity-technical-interview-guide-2026-practical-assessment-spike-round-agentic-coding-and-system-design
- https://www.teamblind.com/post/perplexity-swe-interview-ps2ierct
- https://www.1point3acres.com/interview/guides/interview-trend-perplexity-2026q2
- https://www.fastprep.io/problems/perplexity-llm-provider-pool-failover
- https://github.com/sierra-research/tau-bench
- https://www.anthropic.com/research/demystifying-evals-for-ai-agents
- https://www.anthropic.com/research/writing-tools-for-agents
- https://arxiv.org/abs/2406.18665
- https://arxiv.org/abs/2305.05176
- https://github.com/vllm-project/vllm/blob/main/docs/design/prefix_caching.md
- https://docs.temporal.io/workflows
