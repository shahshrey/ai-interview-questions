# Perplexity: Member of Technical Staff (Software Engineer, Enterprise Agent Evals & Efficiency)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Perplexity publishes a technical interviewing portal that lists its round types. Each role uses a subset, and the recruiter tells you which. The stages below combine that portal with recent candidate reports. Treat the exact order and count as a guess. Candidate reports from 2026 describe AI-free coding rounds, so AI-free Python is the main thing to prepare.

1. Recruiter call (20 to 30 minutes). Background, why Perplexity, and the team's work. Some candidates report a light technical question here.
2. Practical assessment or hands-on coding screen. The practical assessment runs in CoderPad or CodeSignal, is designed for 45 to 90 minutes, and comes with a 2 or 4 hour window. The live screen is a 30 to 60 minute multi-part Python exercise where each part adds tests and earlier tests must keep passing. Perplexity's public example is an agent todo list with dependencies. Candidates report an in-memory file system, a byte tokenizer, and an LLM provider pool with failover. In both formats you may read documentation and ask about library usage, and AI assistance is off. The portal says there are no LeetCode banks, dynamic programming recipes or advanced graph algorithms.
3. Onsite of about 4 to 5 rounds:
   - Hands-on coding, no AI: another applied multi-part component. For this team expect an eval runner, statistics code, a cache, or trace analysis.
   - Agentic coding (possible; no first-hand report for this role yet): a 60 to 120 minute live session outside CoderPad, or a 3 hour async version, where you are expected to drive coding agents on a multi-file task. Your use of the agent is part of what is scored.
   - System design (45 to 60 minutes): problem framing, requirements, architecture, quantitative scaling and tradeoffs. Expect eval environments, regression gates or routing.
   - Core expertise (30 to 60 minutes): eval statistics, grader calibration, routing and inference efficiency, and your own past systems in depth.
   - Hiring manager deep dive (candidates report about 90 minutes): a past project end to end, plus ownership and ambiguity. Some SWE loops add an infra round on an overloaded service.
4. A final conversation with a founder or senior leader.

## Questions and answers

### Coding, no AI, as at Perplexity

#### Q1. Build an async eval runner in three parts (no AI, as at Perplexity): part 1 runs N agent tasks with bounded concurrency and returns results in input order; part 2 adds retries with backoff for 429 and 503 plus a per-call timeout; part 3 adds a requests-per-second limit.

Spec. Input: a list of `(task_id, payload)` and an async `call(payload)` that may raise `RetryableError`, raise a non-retryable error (bad schema), or hang. Output: one `Result` per task in input order with ok, output or error, attempts and latency. Constraints: at most `max_concurrency` in flight, at most `rate_per_s` starts per second, at most `max_retries` retries, each attempt bounded by `timeout_s`. Each part keeps earlier tests green.

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
```

Complexity is O(N x retries) calls and O(N) memory. `gather` returns results in input order, so the runner skips the sort.

I compared two other limiters. A semaphore alone caps in-flight calls but leaves starts unbounded. With 16 slots and 200 ms calls that allows up to 80 starts per second, which breaks a 10 RPS quota. A fixed sleep of 1 / rate between starts honors the quota but wastes throughput after any idle gap, since it cannot burst. The token bucket allows a burst of `capacity` and then holds the long-run rate, so I would pick it and keep the semaphore to bound memory.

For retries, full jitter spreads retries after a provider blip, where fixed backoff would make all 16 workers retry in the same millisecond. Timeouts count as retryable, while schema errors fail on the first attempt, which leaves the retry budget for transient errors. `CancelledError` propagates because it is not an `Exception` subclass on Python 3.8 and later. An empty task list returns `[]`.

Follow-ups to expect: limit tokens per minute with `acquire(cost)` and refund after the response; honor `Retry-After`; release the semaphore during backoff sleeps so fresh tasks can start while retries wait; checkpoint results keyed by `(run_id, task_id, trial)` so a crashed run resumes; and record infrastructure failures such as a 429 separately so the score counts only agent failures.

Why this is likely: Perplexity's hands-on coding guide says exercises build practical multi-part components of its systems, and this team owns eval infrastructure, so a rate-limited runner is the most direct component to ask for.

#### Q2. Build an LLM provider pool (no AI, as at Perplexity): route each request to the highest-priority healthy provider, fail over on retryable errors, eject a provider after repeated failures, and bring it back after a cooldown.

Spec. Input: providers in priority order and an async `call(provider, prompt)` that can raise `RetryableError`. Output: `(provider_used, response)`, or raise `UNAVAILABLE` when nothing healthy remains. After `failure_threshold` consecutive failures a provider is ejected for `cooldown_s`. After the cooldown it is half-open, and one more failure ejects it again. A success resets it. The clock is injected so tests are deterministic.

```python
import time

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
            self.fails[p] = 0
            self.open_until[p] = 0.0
        else:
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
                self.record(p, True)
                return p, out
            except RetryableError:
                self.record(p, False)
```

Setting `fails` to `threshold - 1` on ejection makes the half-open state cheap, because the first failure after cooldown ejects again without a separate state enum. Each request tries each provider at most once, so an all-down pool fails fast after one pass.

The linear scan is O(P) per request, fine for 3 to 10 providers. The large variant candidates report, up to about 1e5 providers with UP and DOWN events, needs a min-heap of healthy priority indices with lazy deletion, O(log P) per event, or a sorted container. Consecutive-failure counting reacts in 3 calls but flaps on a provider failing 30% of calls; an error rate over a sliding window of 50 calls is steadier at the cost of slower ejection.

Edge cases: non-retryable errors such as a 400 propagate and leave health untouched, since the request is at fault. UP and DOWN are idempotent. A recovered higher-priority provider wins the next request. Under concurrency, wrap `record` in an `asyncio.Lock` or accept benign races.

Expect the interviewer to push on the eval side. Failing over from one model to another changes behavior, so every eval call must log which provider served it, or a silent failover looks like a quality regression in the gate.

Why this is likely: an LLM provider pool with failover appears in several candidate reports of Perplexity coding rounds, and the listing makes this team own multi-model reliability and routing.

#### Q3. Debug this eval statistics module (no AI, as at Perplexity). A teammate says the regression gate never flags anything and pass^k looks too high. Find the bugs from the failing tests and fix them.

Artifact: the module and its test output.

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
FAIL test_paired_ci_excludes_zero AssertionError (0.019, -0.016, 0.055)
FAIL test_uses_shared_tasks_only  AssertionError 0.25 (expected 0.0)
failures: 3
```

Bug 1 is a biased pass^k. The code raises the sample pass rate to the power k, which is biased when n is small. With 2 passes in 4 trials and k = 2, the chance that 2 trials drawn without replacement both pass is C(2,2)/C(4,2) = 1/6, while the code returns 0.25. Use the unbiased C(c,k)/C(n,k), and reject tasks with fewer than k trials.

Bug 2 is an unpaired bootstrap. The code flattens trials and resamples A and B independently, which throws away the pairing. Task difficulty varies far more than the 2 point effect, so the interval is about 7 points wide and always crosses zero, which is why the gate never fires. Resample tasks and take the difference within each task. That cancels difficulty and narrows the interval, typically by a quarter to a half when task difficulty dominates the variance. Here it narrows enough that the interval excludes zero.

Bug 3 is mismatched task sets. A has a hard task B never ran, so B looks 25 points better. Compare only shared tasks and report the dropped ones.

```python
import math, random

def pass_hat_k(trials, k):
    vals = []
    for task, outcomes in trials.items():
        n, c = len(outcomes), sum(outcomes)
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
    lo = boots[int(alpha / 2 * n_boot)]
    hi = boots[int((1 - alpha / 2) * n_boot) - 1]
    dropped = sorted(set(scores_a) ^ set(scores_b))
    return mean(diffs), (lo, hi), dropped
```

Alternatives: McNemar's test on binary per-task outcomes is exact and instant but ignores multiple trials per task; a paired t-test on task deltas is close to the bootstrap at 300 or more tasks and costs O(n) instead of O(n x 2000). I keep the bootstrap because it also handles skewed cost metrics.

Why this is likely: the team ships regression gates whose correctness rests on exactly this statistics code, and a debug task with failing tests fits Perplexity's applied hands-on coding format.

#### Q4. Review this AI-generated LLM judge harness before it merges (no AI, as at Perplexity). List every defect, rank by severity, and write the fixed version.

Artifact: the generated code.

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

Defects, most severe first:
1. Security: `eval(raw)` runs model output as Python. The judge reads agent answers, which can carry text that steers the judge into emitting `__import__('os').system(...)`. That is code execution through prompt injection. Parse JSON with a schema instead.
2. Security: `case["id"]` goes straight into a path, so an id of `../../etc/x` writes outside `out_dir`. Validate ids against an allowlist pattern and resolve under the directory.
3. Security: logging `headers` writes tenant auth tokens into logs. Log ids and lengths only.
4. Hallucinated API: `asyncio.gather` has no `limit` argument, so this raises `TypeError` on the first run. Concurrency needs a semaphore.
5. Off-by-one: `range(1, len(cases))` silently drops the first case, so every reported mean is computed on N minus 1 cases while `n` claims N.
6. Missing edge cases: an unparseable verdict raises and kills the whole batch, a score of 9 on a 1 to 5 rubric is accepted, and an all-failed batch divides by zero.

```python
import asyncio, json, logging, re
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
    sem = asyncio.Semaphore(concurrency)
    out_dir = Path(out_dir).resolve()
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
```

Report `parsed` next to `n`. If the parse rate drops from 99% to 80% after a judge prompt change, the mean comes from a biased subset and the gate should mark the run untrusted. For parsing, provider structured outputs with a JSON schema remove most parse failures but tie you to one provider; regex plus `json.loads` works across all of them at a cost of roughly 1% unparseable verdicts that you must count.

Why this is likely: candidate reports (r/Hack2Hire) and Perplexity's official hands-on coding page describe onsite log processing and debug-failure exercises, and the listing makes this team own judges and graders; the review-of-generated-code framing is inferred, not reported.

#### Q5. Find the root cause in this agent trace and write a detector that flags the pattern across a million traces (no AI, as at Perplexity).

Artifact: the trace for a session the user rated thumbs down.

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

The root cause is that search returns a file the principal cannot read, because the search connector does not filter by ACL. The 403 body is a bare "forbidden" with no hint, so the agent treats it as transient and retries the identical call twice. Then it falls back to public web search and states a number from an unrelated public company as if it were the tenant's figure. There are three faults with three owners: the connector (search leaks titles of unreadable files), the tool result (non-actionable error), and the agent policy (no refusal path, unsourced claim). The user-visible harm is the last one, a confident wrong answer, so the eval task I would add has a success predicate of "tells the user they lack access to f_91 and makes no numeric claim".

```python
import hashlib, json
from collections import Counter

def call_key(ev):
    args = json.dumps(ev.get("args", {}), sort_keys=True)
    return ev["tool"] + ":" + hashlib.sha1(args.encode()).hexdigest()[:12]

def find_loops(events, max_repeats=3, window=8):
    findings, recent = [], []
    for i, ev in enumerate(events):
        if ev.get("type") != "tool_call":
            continue
        recent.append((i, call_key(ev), ev.get("status")))
        recent = recent[-window:]
        counts = Counter(k for _, k, _ in recent)
        key, n = counts.most_common(1)[0]
        if n >= max_repeats:
            errs = [s for _, k, s in recent if k == key and s and s >= 400]
            findings.append({"step": i, "key": key, "repeats": n,
                             "kind": "error_retry_loop" if len(errs) == n else "repeat_loop"})
            recent = [r for r in recent if r[1] != key]
    return findings
```

On this trace it returns one `error_retry_loop` at step 5. It is O(steps x window) per trace and streams, so it runs as a map job over the trace store. An LLM judge over every trace catches subtler failures but at about 50k tokens per trace costs roughly 50B tokens for a million traces, so I run cheap detectors first and judge only flagged and sampled traces. A second detector would flag a final answer containing numbers when no successful tool result contained them.

Why this is likely: the listing asks you to analyze agent trajectories and production failures and identify root causes, and candidate reports describe debugging rounds built on logs.

#### Q6. Implement a block-level prefix cache for prompt KV state (no AI, as at Perplexity): requests sharing a token prefix reuse blocks, with LRU eviction and tenant isolation.

Spec. `insert(tokens, salt)` stores blocks for a processed prompt; `lookup(tokens, salt)` returns how many leading tokens are cached. Fixed `block_size` (vLLM defaults to 16), only full blocks cached, capacity `max_blocks` with LRU eviction, and a block is identified by its own tokens plus everything before it.

```python
import hashlib
from collections import OrderedDict

class PrefixCache:
    def __init__(self, block_size=16, max_blocks=1024):
        self.bs, self.max_blocks = block_size, max_blocks
        self.blocks = OrderedDict()

    def _hashes(self, tokens, salt=""):
        h = hashlib.sha256(salt.encode()).hexdigest()
        out = []
        full = len(tokens) - len(tokens) % self.bs
        for i in range(0, full, self.bs):
            block = tokens[i:i + self.bs]
            h = hashlib.sha256((h + "|" + ",".join(map(str, block))).encode()).hexdigest()
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
```

Lookup and insert are O(L / block_size) dict operations after O(L) hashing; memory is O(max_blocks).

The hashes are chained because hashing a block alone would let block B after prefix A collide with B after prefix X, but their KV values differ because attention at every position depends on everything before it. Chaining makes the key encode the full prefix, so a flat dict replaces a trie. This mirrors vLLM automatic prefix caching, which also folds LoRA ids and a cache salt into the key.

A radix tree (SGLang RadixAttention) matches at token granularity, so a 1,000 token shared prefix reuses all 1,000 tokens instead of 992 with 16 token blocks, at the cost of pointer-heavy nodes and harder eviction. Hashing whole prompts is simplest but gives zero reuse across agent turns that append to the same prefix, where block chaining gives close to 100% reuse of the history.

Edge cases:
- One changed token near the top misses every later block, which is why timestamps belong at the end of the prompt.
- Without a salt, hit latency becomes a cross-tenant timing side channel.
- Evicting a parent while a child stays cached leaves an unreachable child. Production engines evict leaves first and never evict blocks referenced by running requests.

Follow-ups: refcounts, hit rate for a 12 turn agent loop, and cache-aware routing of sessions to the replica holding their prefix.

Why this is likely: the listing names caching and context management as efficiency levers, and a prefix cache is the data structure that maps directly onto inference cost.

### Agentic coding, AI assistant allowed

#### Q7. Agentic coding round (AI assistant allowed): add a `gate` command to this eval harness repo that compares a candidate run with a baseline per slice, blocks on hard invariant failures, and blocks when a slice regresses beyond a margin. Show how you drive the agent.

Artifact: the repo and the spec.

```text
evalkit/
  pyproject.toml
  evalkit/__init__.py
  evalkit/cli.py            # argparse; subcommands: run, report
  evalkit/store.py          # load_run(path) -> {task_id: score}, load_meta(path)
  evalkit/stats.py          # paired_bootstrap(a, b) from Q3
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

Delegation plan. First I read `cli.py`, `store.py` and `stats.py` myself for five minutes, since the agent writes better code when my prompt names the real interfaces. Then two prompts. Prompt 1: "Write `evalkit/gate.py` with `gate(base, cand, task_slice, margin, invariants_failed)` returning per-slice verdicts. Reuse nothing from stats.py; paired deltas on shared tasks only; Bonferroni alpha 0.05/m; slices under 20 tasks never block. Write tests first in `tests/test_gate.py` covering a regressed slice, a clean slice, an invariant failure and a tiny slice." Prompt 2: "Wire `gate` into `cli.py` as a subcommand with exit code 1 on block, plus a CLI test using the fixtures." Small prompts keep each diff reviewable.

In the diff I check that tasks are intersected, alpha is divided by the number of slices, the lower bound uses alpha rather than alpha / 2 because the test is one-sided, invariants short-circuit before statistics, and `stats.py` signatures that other callers use stay the same. Tests I run: `pytest -q tests/`, then the CLI against the fixtures, then a mutation check where I edit the fixture to regress one slice by 10 points and confirm exit code 1.

Here is a wrong output I caught. The agent's first version computed one bootstrap over all tasks and then split the resampled means by slice, which mixes slices. It reported the CRM slice as passing because a flat docs slice diluted a 10 point CRM drop. Its own test checked only the overall delta, so it passed. I added the per-slice regression test, which failed, and told the agent to bootstrap within each slice. The core after the fix:

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
```

On a 300 task fixture with CRM down 10 points, CRM returns a lower bound near minus 0.10 and blocks while docs passes. Alternative designs: a single pooled test is faster but hides slice regressions, as shown; Benjamini-Hochberg instead of Bonferroni blocks more true regressions across 20 slices but tolerates about 5% false blocks among flagged slices.

Why this is likely: Perplexity's agentic coding round requires coding agents on a multi-file task, and a regression gate is the artifact this team ships (official round format, high confidence; low confidence that this specific role includes it, since no first-hand MTS-evals report was found).

### System design

#### Q8. Design a reproducible evaluation environment for enterprise agents with connected apps (mail, docs, tickets, CRM), realistic data, roles and permissions, and multi-step workflows.

Assume about 5k tasks, each run 3 to 8 times for pass^k, so 20k to 40k trials per full suite. Every trial starts from identical state in isolation, sees production tool schemas, and is graded on end state. A full suite finishes in under 2 hours, which at 3 minutes per trial means 750 to 1,000 trials in parallel.

Components:
1. Connector fakes behind production interfaces. Each app is a stateful service implementing the same tool schema as the real connector. Contract tests replay recorded real calls against each fake on every connector release to catch drift.
2. A world generator that builds an org, documents, threads and tickets from seeds, injecting near-duplicate names, stale docs with newer replacements, long threads and partial access.
3. Identity and ACLs enforced inside the fakes, with tasks whose correct outcome is a refusal, such as the 403 case in Q5.
4. Snapshot and restore. Each world is a versioned database dump plus blob store, cloned copy-on-write per trial, with a frozen clock and a seeded user simulator.
5. Task specs: instruction, principal, world version, success predicates over end state, forbidden actions, optional rubric.
6. A runner and trace store logging every tool call, tokens and latency plus the world diff.

High-fidelity fakes cost perhaps one engineer-week per connector to build and ongoing upkeep, but support any trajectory. Record and replay costs hours per connector but breaks on any unseen call, so in practice 20% to 40% of new trajectories fall off the recording. Real staging tenants are the most faithful but run at 30 to 60 seconds per reset and leak state between trials. I would use fakes for the five core apps, replay for the long tail, and a nightly canary of 100 tasks on real staging tenants to measure fake drift as a pass-rate gap.

Hard parts: nondeterminism (freeze what you can, measure the rest with repeated trials); graders that check state diffs, so an agent that claims success without acting fails; and task validity, where a human solves a sample and any task no model passes is reviewed as possibly broken.

Why this is likely: building realistic enterprise evaluation environments with connected applications, user roles, permissions and reproducible state is the first responsibility in the listing, and Perplexity's system design guide says questions cover the team's domain.

#### Q9. Design the regression gate that decides whether a model, connector, tool, skill or harness change can ship.

Requirements. Changes land daily from several teams. The gate answers three questions: did anything get worse beyond noise, on which slices, and at what cost. Verdicts in under 30 minutes for normal PRs, deeper runs nightly. Keep false blocks rare so engineers keep trusting it.

Design:
1. Tiered suites. Smoke: 100 tasks, 1 trial, about 10 minutes on every PR. Gate: 1.5k tasks, 3 trials, before merge to main or a model swap. Full: all tasks with 5 or more trials nightly. Each change type maps to the slices it can touch.
2. A pinned baseline on the same task versions, re-run when worlds or graders change.
3. A statistical decision per slice: paired bootstrap lower bound against a non-inferiority margin, such as minus 2 points on success, plus 10% on p95 latency, or plus 10% on cost per successful task (Q3 and Q7 code).
4. Hard invariants with zero tolerance: data sent outside an ACL, destructive calls without confirmation, schema violations.
5. Grader health: re-score a fixed calibration set each run; if judge agreement with humans drops, the result is marked untrusted and cannot pass.
6. Output: deltas, intervals, newly failing tasks with trace links, a cost table, and logged overrides with a named owner.

Start with power. With 1.5k tasks and a per-task delta standard deviation near 0.25, the standard error is about 0.65 points, so the gate detects a 2 point drop reliably, while the 100 task smoke tier only detects drops near 8 points. For budget, 1.5k tasks x 3 trials x 40k tokens is 180M tokens per gate run, so caching shared prefixes (Q12) matters. Among other decision rules, a fixed threshold on the raw mean ("block if success falls 0.5 points") is simple but at this noise level blocks about one in five clean PRs; sequential testing stops early on clear wins or losses and cuts average cost by roughly 40%, but is harder to explain to the teams being gated. I would start with the fixed-sample bootstrap and add sequential stopping once trust is earned.

Hard parts: flaky tasks (track per-task flip rate with no code change and quarantine above 20%), saturation (rotate in fresh production failures), and slices too small to test, which are reported but never block.

Why this is likely: the listing asks for regression gates and launch-readiness signals for model, connector, tool, skill and harness changes, and Perplexity's agent evaluation post describes the same workflow.

#### Q10. Design a model routing and effort-selection system for enterprise agent tasks that minimizes cost per successful task without losing quality.

There are several model families, each with effort levels. The router chooses per task, and maybe per step. Targets: success within a non-inferiority margin of an all-frontier baseline, lower cost per successful task, p95 latency budgets per surface, and routing overhead under 50 ms.

Components:
1. Features: request embedding, a task-type classifier, connectors in scope, expected horizon, tenant tier, history so far.
2. A router that predicts success, cost and latency per option and picks the cheapest option clearing a success threshold, or maximizes success minus lambda x cost, with lambda tuned on the eval set.
3. Escalation: start cheap and escalate when a verifier, schema error, empty search or loop detector (Q5) says the step failed.
4. Training labels from running the eval suite with every candidate model, plus production traces scored by calibrated judges.
5. Shadow mode, then an A/B on success, retries and user-visible failures, with a per-tenant kill switch.

Static rules by task type cost nothing to serve and are easy to audit, but typically capture only part of the available savings. A learned router like RouteLLM's matrix factorization reports cost reductions above 2x on public benchmarks at near-equal quality, with a few milliseconds of overhead. A cascade like FrugalGPT gets similar savings without a trained router but adds a second model call on every escalation; if 30% of tasks escalate, latency on those rises by the cheap model's full runtime. I would start with a cascade on the two highest-volume task types, because escalation signals in agent loops are cheap and reliable, then train a router once there are enough labels.

Hard parts:
- Switching models mid-trajectory throws away the prompt cache, so with a 40k token history a switch can cost more than it saves. Route at task start and at subagent boundaries.
- Router calibration drifts when providers update models.
- Different models need different tool descriptions and skills, so every route needs its own eval coverage.
- Evaluate the router against an oracle that picks the cheapest model that succeeded, which shows the headroom left.

Why this is likely: the listing asks you to build and evaluate model selection and routing strategies that match each task to the right model and effort level.

### Domain: graders, cost and tools

#### Q11. Design the calibration study for an LLM judge that grades whether an enterprise task was completed, and decide what numbers would let it gate launches.

Start with labels. Sample 300 trajectories stratified by task type and by the judge's verdict, so rare failures are covered. Two domain experts label each one blind to the judge, because annotators shown the verdict anchor on it. Write the rubric first and adjudicate disagreements, which exposes ambiguous criteria.

Metrics and the decision:
- Human-human Cohen's kappa is the ceiling. If two experts agree at 0.62, a judge at 0.60 with each behaves like a third expert, and demanding 0.9 asks more than the task allows. Below about 0.6 between humans, fix the rubric before tuning the judge.
- Judge-human kappa plus the confusion matrix. Accuracy misleads at skewed prevalence: if 90% of runs pass, an always-pass judge scores 90% accuracy and kappa of 0.
- True positive rate s and true negative rate t, because both bias the reported pass rate. The corrected rate is (p_obs + t - 1) / (s + t - 1). With s = 0.95, t = 0.80 and an observed 0.70, the true rate is 0.667, a 3.3 point overstatement.
- Consistency: grade the same saved trajectory 5 times without re-running the agent, to separate grader variance from agent variance.

```python
def cohen_kappa(a, b):
    n = len(a)
    po = sum(x == y for x, y in zip(a, b)) / n
    pe = sum((a.count(l) / n) * (b.count(l) / n) for l in set(a) | set(b))
    return 1.0 if pe == 1 else (po - pe) / (1 - pe)

def corrected_rate(p_obs, tpr, tnr):
    return (p_obs + tnr - 1) / (tpr + tnr - 1)
```

My gate rule: the judge may gate launches when judge-human kappa is within 0.05 of human-human kappa, the true negative rate on safety and permission criteria is at least 0.95, and repeat-grading flips stay under 3%. Otherwise its verdicts are reported but cannot block or pass.

Among grading methods, code checks on end state are exact and nearly free but cover only verifiable criteria. A single holistic 1 to 10 judge score is cheap but drifts and is hard to calibrate. Narrow binary criteria judged separately cost about 3 to 5 times more judge tokens but give per-criterion kappa you can act on. I would use code checks wherever possible, binary judged criteria for the rest, and re-run the calibration set on every judge model or prompt change.

Why this is likely: the listing says graders must be calibrated against human judgment, and Perplexity's agent evaluation post describes a human calibration process and a judge consistency check.

#### Q12. An agent harness puts the current time and user name at the top of a 30k token system prompt. Sessions average 12 model calls. Estimate the cost of this choice, choose a fix, and pick a cache lifetime.

Prefill computes KV state for every prompt token; a prompt cache reuses it for an identical prefix. Matching is exact and prefix-only, so a timestamp in the first line changes every call's prefix and nothing after it is ever reused.

To estimate, assume the stable prefix (system prompt, tools, skill index) is 30k tokens and history grows by about 2k tokens per call, so call i has 30k + 2k x i input tokens. Over 12 calls that is 360k + 2k x 78, about 516k input tokens. Today all of it is billed at the full input rate. With the timestamp moved to the end, call 1 writes the cache and calls 2 to 12 read nearly all their prefix. Using Anthropic-style multipliers of 1.25x for a 5 minute write and 0.1x for reads, and treating each call's new 2k tokens as written, the cost is 32k x 1.25 for the first call, plus 11 x 2k x 1.25 written, plus 462k read at 0.1x: 40k + 27.5k + 46.2k, or about 114k token-equivalents. That is about 78% cheaper on input, and time to first token drops because prefill shrinks.

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

print(round(session_cost(cached=False)), round(session_cost()))
```

Option A moves the time and user fields to the last user message. It is free and keeps the whole prefix stable. Option B drops the time and gives the agent a clock tool, which is also stable but adds a tool call when time matters, maybe 5% of sessions. I pick A and add B only if prompts grow.

Cache lifetime. A 5 minute write breaks even on the second use. A 1 hour write costs 2x and needs three uses. Agent calls are seconds apart, so 5 minutes wins inside a session; a 1 hour cache pays only for prefixes shared across sessions, such as the 30k system prompt across a tenant's users. I would add a cached-token fraction metric to the gate, since a harmless-looking prompt edit can silently double input cost.

Why this is likely: the listing names caching and context management as efficiency levers and makes cost per successful task a target, so an estimate-and-choose cache question is a natural core expertise probe.

#### Q13. Config A uses a frontier model at high effort; config B uses a mid model with escalation. Given the run summary, decide which ships, using cost per successful task with uncertainty.

Data from a paired run on 400 tasks:

```text
config  success  mean_cost/task  p95_cost  retries  p95_latency_s
A       0.81     0.42            1.10      0.05     38
B       0.79     0.17            0.95      0.22     41
paired delta B-A success: -0.020, 95% CI [-0.041, +0.001]
slice long_horizon (n=60): A 0.70, B 0.58, delta CI [-0.21, -0.03]
```

Cost per successful task is total spend on all attempts, including failed ones and retries, divided by the number of successes. A: 0.42 / 0.81 = 0.52 units. B: 0.17 / 0.79 = 0.215 units. B is about 58% cheaper per success.

```python
def cpst(mean_cost, success):
    return mean_cost / success

a, b = cpst(0.42, 0.81), cpst(0.17, 0.79)
print(round(a, 3), round(b, 3), round(1 - b / a, 2))
```

Uncertainty. CPST is a ratio of two noisy means, and agent cost is heavy-tailed, so I would bootstrap over tasks rather than use a normal approximation, resampling the same tasks for both configs. The delta method works as a cross-check but understates the tail when a few loops cost 20x the median.

Decision. Quality is a constraint, cost is the objective. The overall success CI of minus 4.1 to plus 0.1 points sits inside a minus 5 point non-inferiority margin but not a minus 2 point one, so the answer depends on the margin agreed before the run, and I would say which one I used. The long-horizon slice fails clearly: B loses 12 points there. The answer is a split. Ship B for short tasks and route long-horizon tasks to A, which on this mix (15% long-horizon) gives a blended CPST near 0.26 with no slice regression, assuming long-horizon tasks cost the same per task as the overall mean; with real per-slice costs it will be higher. Alternatives considered: ship A everywhere, safe but about 2x the cost; ship B everywhere, cheapest but a real regression on the hardest, highest-value tasks.

Checks before trusting it: both configs measured with the same cache warmth; success graded by the same calibrated judge; p95 latency of 41 seconds still inside the product budget; and the Q5 detector run over B's traces to explain its 0.22 retry rate.

Why this is likely: the listing names cost per successful task as a metric the team moves and asks for sound quality, latency and cost tradeoff judgment.

#### Q14. Design the eval suite that tells you whether a rewrite of tool descriptions and skill loading improved tool selection, and how you would avoid breaking tools you did not touch.

Treat selection as classification with its own eval, separate from end-to-end success.

1. Selection set. For each tool or skill, 20 to 40 positive queries where it should be chosen, negatives drawn from adjacent tools, and forbidden cases. Perplexity describes suites that check precision, recall and forbidden loads for skills, plus whether the agent reads the accessory files after loading. Track a full confusion matrix; most bugs are two tools with overlapping descriptions.
2. Call correctness: schema-valid arguments, semantically right arguments (correct user id, date range), and calls needed to reach the goal.
3. End-to-end tasks with the full loop and a rubric judge, to confirm that a selection gain also improves task success.
4. Every orchestrator model separately, because families react differently to the same description.

Decision rule: ship the rewrite if macro recall rises with a paired CI above zero, forbidden-load rate stays flat or falls, calls per successful task rise by at most 5%, and every tool's recall stays within 5 points of its baseline.

Testing only end-to-end success needs about 1.5k tasks to see a 2 point change and blames nothing specific. A selection-only suite of 40 queries x 50 tools is 2k cheap single-turn calls, under a minute, and points at the exact confused pair, but misses argument errors. I would run selection on every PR and end-to-end in the nightly tier.

Improvements that usually move the numbers: descriptions written as routing triggers ("use when the user asks about X; not for Y"); consolidating overlapping tools into one with filters; enums instead of free strings; concise response formats with pagination; and actionable errors. The Q5 trace shows the difference. A bare "forbidden" with no guidance caused three wasted calls, while "no access to f_91; ask an owner in finance_leads" ends the loop in one.

Adding one tool can steal selections from an untouched one, so every addition re-runs the full selection suite and reports per-tool recall deltas alongside the average.

Why this is likely: the listing asks you to improve tool descriptions, schemas and result handling so agents select the right tools and recover with fewer calls, and Perplexity's skills guide describes these suites.

### Behavioral

#### Q15. Tell me how you use coding agents day to day, including a time an agent's output looked right but was wrong, how you caught it, and what you changed in how you check agent work. Tie it to a project where you defined a metric across teams.

The interviewer wants evidence that you get real speed from agents and that you check their work systematically. Perplexity scores agent use directly in its agentic round, and its portal lists rigorous thinking, controlling complexity and principled tradeoffs as traits, so this question links both.

A strong structure:
1. Your workflow in concrete terms. What you delegate (test scaffolding, migrations, boilerplate, first drafts of analysis scripts), what you keep for yourself (metric definitions, security-sensitive parsing, statistical decision rules), and how you size prompts so each diff is reviewable in a few minutes.
2. The miss. A specific case, for example an agent-written eval aggregation that averaged trials before tasks and shifted a headline number by 3 points, or a judge parser that used `eval`. Say how long it went unnoticed and what it cost.
3. How you caught it: a test with known answers, a mutation check, reading 20 raw cases, or a number that disagreed with intuition.
4. What you changed: tests first with hand-computed expected values, a checklist for generated code (invented APIs, unvalidated input, silent drops), and a rule that metric code needs a second human reviewer.
5. The cross-team link. The metric you defined with product, research and platform, how you got agreement (a one-page spec and 100 jointly labeled cases), and the measured result with uncertainty, such as success up 11 points with a paired interval of plus 7 to plus 15 and cost per success down 30%.
6. What persisted: the gate or checklist other teams adopted, or someone you mentored who now owns it.

Show that agents are part of your daily work, since Perplexity's careers guide says knowledge of how to use AI is required and its agentic round, if your loop includes it, scores agent use. When you say the tests passed, say who wrote the tests. Keep a backup story about an eval of yours that was wrong.

Why this is likely: Perplexity's careers interview guide says "knowledge of how to use AI is required", its agentic coding round, if your loop includes it, scores agent use, and Perplexity runs a long hiring manager deep dive on past projects, so one question that covers both is the most likely behavioral probe.

## Prep checklist

- [ ] Solve Perplexity's public todo list practice exercise in Python under 60 minutes without AI, keeping earlier parts' tests green.
- [ ] Write the Q1 runner and Q2 provider pool from memory, then the heap variant for 1e5 providers.
- [ ] Hand-compute unbiased pass^k for a few tasks and code a paired bootstrap that intersects task sets.
- [ ] Practice reviewing generated code for invented APIs, off-by-ones, missing edge cases and injection, using Q4 as a template.
- [ ] Debug three agent traces by hand and write a loop detector that streams over JSONL.
- [ ] Do one timed multi-file task with a coding agent, writing tests first and logging each wrong output you caught.
- [ ] Compute Cohen's kappa, TPR, TNR and a prevalence-corrected pass rate on a small labeled set.
- [ ] Practice 45 minute designs for an eval environment, a regression gate with non-inferiority margins, and a cost-aware router, each with numbers.
- [ ] Learn prompt caching rules for two providers and redo the Q12 session estimate with your own numbers.
- [ ] Read Perplexity's posts on agent evaluation and agent skills, plus the tau-bench paper and Anthropic's agent eval and tool-writing guides.
- [ ] Prepare a 90 minute deep dive on one past system and two behavioral stories with numbers, one about catching a wrong agent output.

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
- https://www.glassdoor.com/Interview/Perplexity-AI-Interview-Questions-E8515634.htm
- https://www.1point3acres.com/interview/problems/company/perplexity
- https://aceoffer.app/interviews/perplexity
- https://www.reddit.com/r/Hack2Hire/comments/1us3j19/perplexity_interview_process_experience
- https://www.fastprep.io/problems/perplexity-llm-provider-pool-failover
- https://www.techprep.app/blog/perplexity-interview-process
- https://arxiv.org/pdf/2406.12045
- https://github.com/sierra-research/tau-bench
- https://www.anthropic.com/research/demystifying-evals-for-ai-agents
- https://www.anthropic.com/research/writing-tools-for-agents
- https://arxiv.org/abs/2406.18665
- https://arxiv.org/abs/2305.05176
- https://github.com/vllm-project/vllm/blob/main/docs/design/prefix_caching.md
- https://cameronrwolfe.substack.com/p/stats-llm-evals
