# NVIDIA: Senior Software Engineer, Agentic Systems

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

NVIDIA publishes the outline of its hiring process, and each team designs its own rounds inside it. The listing describes a senior Python engineer building the platform side of agent evaluation: SDKs, plugins and CLIs, typed trace schemas with Pydantic, distributed job orchestration on Kubernetes, and testing from unit level up to Docker and Kubernetes. NVIDIA's open-source NeMo Evaluator ("everything is an environment", `@benchmark` and `@scorer` plugins, local, Docker and SLURM backends, resumable runs, and statistical quality gates) matches that scope, so the domain rounds below lean on that kind of work. That match is an inference. Candidates on Blind say there is no standard process across NVIDIA teams, so treat this as a likely shape and confirm it with the recruiter.

1. Recruiter or application review. A short call about background and team fit. Virtual. AI: not relevant.
2. Hiring manager screen. 30 to 60 minutes by video. A close look at past projects and your own contribution. A NeMo team candidate described this round first, followed by coding, a live pull request review and an ML pipeline design. AI: not allowed.
3. Technical screen or online assessment. 45 to 60 minutes, live in HackerRank or a shared editor, in Python. Recent reports describe practical, team-written problems: a REST API task with JSON, retries and tests, a multi-part Python parsing task, or a task DAG executor. Some teams send a short online assessment with a LeetCode-style question instead. AI: not allowed.
4. Onsite coding, two or three rounds. 45 to 60 minutes each, on a whiteboard or an NVIDIA laptop, at least one round in person. One guide reports two LeetCode medium problems at the senior onsite; other reports describe async and concurrency work, plugin or API design in code, and a live code review where you find the bugs. A recent Blind invite listed rounds named "llm problem solving" and "bug bash". AI: not allowed.
5. Domain deep dive. 45 to 60 minutes with team engineers. For this team, expect agent evaluation internals: reading agent traces, statistical regression gates, Pydantic schemas for traces and their versioning, and ingestion performance. AI: not allowed.
6. System design, one or two rounds. 45 to 60 minutes. An ML or evaluation pipeline is the reported theme: a distributed eval platform with checkpoint and resume, the public SDK and CLI of an eval library, or a test strategy that spans Docker and Kubernetes. AI: not allowed.
7. Hiring manager or behavioral round, about 45 minutes, on ownership, ambiguity, technical leadership and how you work with AI tools day to day. AI: not allowed.
8. Decision. NVIDIA says it will get back to you "in a few short weeks". AI: not relevant.

AI tools: NVIDIA's How We Hire page says that using unapproved outside tools such as ChatGPT during an interview "will result in the disqualification of candidacy", and that an in-person onsite at an NVIDIA office comes before any offer. A reply in a recent Blind thread said the "llm problem solving" and "bug bash" rounds were also without AI assistance. No source shows an AI-assisted round at NVIDIA. Every question below assumes no AI, as at NVIDIA, so you write and check all the code yourself.

Classic algorithm questions are limited to two (Q3 and Q4), because reports show a LeetCode-style online assessment on some teams and two medium problems at some senior onsites, while most rounds for platform roles use practical problems.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a client that pulls every result of an eval run from a paginated, flaky API | Build | Technical screen | Not allowed | Medium |
| [Q2](#q2) | Parse a JSONL eval event log in three parts: durations, retries, then per-benchmark scores | Build | Technical screen | Not allowed | Medium |
| [Q3](#q3) | Order the stages of an eval pipeline, run independent stages in parallel waves, and reject cycles | Classic algorithm | Technical screen or online assessment | Not allowed | Medium |
| [Q4](#q4) | Return the K most common failure signatures across millions of eval samples | Classic algorithm | Onsite coding | Not allowed | Medium |
| [Q5](#q5) | Build an async eval runner with bounded concurrency, timeouts, retries and resume from a checkpoint | Build | Onsite coding | Not allowed | Hard |
| [Q6](#q6) | This async eval runner reports the wrong accuracy. Find the bugs | Debug | Onsite coding | Not allowed | Medium |
| [Q7](#q7) | A teammate's coding assistant wrote this Pydantic trace normalizer. Review it before it merges | Review | Onsite coding | Not allowed | Medium |
| [Q8](#q8) | Build the `@benchmark` and `@scorer` plugin registry for an eval SDK | Build | Onsite coding | Not allowed | Medium |
| [Q9](#q9) | Tool-calling success fell from 71% to 58% after an upgrade. Read the traces and find the cause | Trace debug | Domain deep dive | Not allowed | Medium |
| [Q10](#q10) | Design the statistical gate that says GO, NO-GO or INCONCLUSIVE for a new model or agent version | Concept plus code | Domain deep dive | Not allowed | Hard |
| [Q11](#q11) | Your trace schema needs a breaking change. Version it in Pydantic without breaking old readers or old data | Concept plus code | Domain deep dive | Not allowed | Hard |
| [Q12](#q12) | Trace ingestion spends most of its time in Pydantic validation. Make it faster | Optimize | Domain deep dive | Not allowed | Medium |
| [Q13](#q13) | Design a distributed eval platform that runs on a laptop, Docker, SLURM and Kubernetes, and resumes after failures | Design | System design | Not allowed | Hard |
| [Q14](#q14) | Design the public Python SDK and CLI of an open-source eval library | Design | System design | Not allowed | Hard |
| [Q15](#q15) | Design the test strategy for an eval platform, from unit tests to Kubernetes | Design | System design | Not allowed | Medium |
| [Q16](#q16) | How do you use AI coding tools in your daily work, and how do you check what they produce? | Behavioral | Hiring manager | Not allowed | Medium |

### Technical screen coding (no AI, as at NVIDIA)

<a id="q1"></a>

#### Q1. Build a client that pulls every result of an eval run from a paginated, flaky API

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Cursor pagination, retries, deduplication.

<details>
<summary>Show answer</summary>

Task: spec given in the round. `GET /runs/{id}/results?cursor=C` returns `{"items": [{"benchmark", "sample_id", "score"}], "next": C2}`, where `next` is null on the last page. The service sometimes returns 503, drops connections, or returns 429 with a `Retry-After` header. Items can repeat across pages when results arrive during the scan. Write `collect(get, run_id)` and `accuracy_by_benchmark(items)`.

What they grade: which errors you retry, how you stop on a bad cursor, tests without a network, and correct numbers when the data repeats.

Approach: keep I/O and logic apart. `fetch_page` owns the retry policy and takes the transport and the sleep function as parameters, so a test can script responses and record delays. `collect` owns pagination. It follows `next`, deduplicates by `(benchmark, sample_id)` keeping the last copy, and raises if a cursor repeats, because a server bug that loops would otherwise spin forever. Retry 429, 5xx and connection errors; fail fast on other 4xx. Backoff doubles from 0.5 seconds with a cap of 8, and on 429 the server's `Retry-After` wins when it is longer.

Code:

```python
import json

RETRYABLE = {429, 500, 502, 503, 504}

class HTTPError(Exception):
    def __init__(self, status):
        super().__init__(f"HTTP {status}")
        self.status = status

def fetch_page(get, url, sleep, max_attempts=4, base=0.5, cap=8.0):
    for attempt in range(max_attempts):
        try:
            status, headers, body = get(url)
        except ConnectionError:
            status, headers, body = None, {}, ""
        if status == 200:
            return json.loads(body)
        if status is not None and status not in RETRYABLE:
            raise HTTPError(status)
        if attempt == max_attempts - 1:
            raise HTTPError(status or 0)
        delay = min(cap, base * 2 ** attempt)
        retry_after = headers.get("Retry-After", "")
        if status == 429 and retry_after.isdigit():
            delay = max(delay, float(retry_after))
        sleep(delay)

def collect(get, run_id, sleep, cursor=None):
    seen_cursors, items = set(), {}
    while True:
        url = f"/runs/{run_id}/results" + (f"?cursor={cursor}" if cursor else "")
        page = fetch_page(get, url, sleep)
        for it in page["items"]:
            items[(it["benchmark"], it["sample_id"])] = it
        cursor = page.get("next")
        if not cursor:
            return list(items.values())
        if cursor in seen_cursors:
            raise RuntimeError(f"cursor {cursor} repeated")
        seen_cursors.add(cursor)

def accuracy_by_benchmark(items):
    totals = {}
    for it in items:
        n, s = totals.get(it["benchmark"], (0, 0.0))
        totals[it["benchmark"]] = (n + 1, s + float(it["score"]))
    return {b: round(s / n, 4) for b, (n, s) in sorted(totals.items())}

def scripted(pages):
    it = iter(pages)
    def get(url):
        r = next(it)
        if isinstance(r, Exception):
            raise r
        status, payload = r
        headers = {"Retry-After": "3"} if status == 429 else {}
        return status, headers, json.dumps(payload)
    return get

p1 = {"items": [{"benchmark": "mmlu", "sample_id": 1, "score": 1}, {"benchmark": "gsm8k", "sample_id": 7, "score": 0}], "next": "a"}
p2 = {"items": [{"benchmark": "mmlu", "sample_id": 1, "score": 1}, {"benchmark": "mmlu", "sample_id": 2, "score": 0}], "next": None}
slept = []
get = scripted([(503, {}), ConnectionError(), (200, p1), (429, {}), (200, p2)])
items = collect(get, "r1", slept.append)
assert len(items) == 3
assert accuracy_by_benchmark(items) == {"gsm8k": 0.0, "mmlu": 0.5}
assert slept == [0.5, 1.0, 3.0]
try:
    collect(scripted([(404, {})]), "r1", slept.append)
    raise AssertionError("404 must not retry")
except HTTPError as e:
    assert e.status == 404
loop = {"items": [], "next": "x"}
try:
    collect(scripted([(200, loop), (200, loop)]), "r1", slept.append)
    raise AssertionError("loop not detected")
except RuntimeError:
    pass
```

Verify: the scripted run covers a 503, a dropped connection, a 429 whose `Retry-After` beats the backoff, a repeated item across pages, a 404 that fails at once, and a cursor loop. The attempt counter resets for each page, so one bad page cannot use up the budget of the whole scan.

Pitfalls:
- Counting the repeated sample twice, which makes `mmlu` look like 0.67 instead of 0.5.
- No guard on a repeating cursor, so a server bug becomes an infinite loop.
- Calling `time.sleep` directly, which makes tests slow.

Follow-ups: offset pagination is simpler but skips or repeats rows when results are inserted during the scan, which is why cursors win here; a server-side snapshot id would remove the duplicates at the source. Full jitter (a random delay between 0 and the current backoff, min(cap, base * 2^attempt)) spreads a fleet of clients better than plain doubling when 500 workers retry at once. To resume a crashed scan, persist the last cursor and pass it as `cursor`.

Why this is likely: a 2026 candidate report describes an NVIDIA practical screen built around a REST API with JSON, error handling and tests, and the listing asks for Python API and SDK work with strong testing; medium confidence in the shape, the exact API is illustrative.

</details>

<a id="q2"></a>

#### Q2. Parse a JSONL eval event log in three parts: durations, retries, then per-benchmark scores

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Multi-part parsing, malformed input, ordering.

<details>
<summary>Show answer</summary>

Task: the interviewer reveals the parts one at a time. Each line is JSON: `{"ts": int, "task": str, "benchmark": str, "attempt": int, "event": "start" | "end", "status": "pass" | "fail" | "error"}` (status only on `end`). Part 1: return the duration of each task. Part 2: tasks can be retried; only the highest attempt counts, and lines can arrive out of order. Part 3: report pass rate per benchmark, with `error` excluded from the denominator and counted separately, plus a count of malformed lines.

What they grade: a data model that survives the next part without a rewrite, explicit rules for bad input, and tests per part.

Approach: parse each line into a record or count it as malformed (bad JSON, missing field, wrong type). Key state by `(task, attempt)` so part 2 changes only the reduce step. Sort by timestamp only where needed, since one sort of all lines costs O(n log n) and grouping first is O(n). After grouping, pick the highest attempt that has both a start and an end; a lone start means the task is still running or crashed, so part 3 lists it as incomplete and leaves it out of the pass rate.

Code:

```python
import json
from collections import defaultdict

REQUIRED = {"ts": int, "task": str, "benchmark": str, "attempt": int, "event": str}

def parse(lines):
    recs, bad = [], 0
    for line in lines:
        try:
            r = json.loads(line)
            if not all(isinstance(r.get(k), t) for k, t in REQUIRED.items()) or r["event"] not in ("start", "end"):
                raise ValueError
            if r["event"] == "end" and r.get("status") not in ("pass", "fail", "error"):
                raise ValueError
            recs.append(r)
        except ValueError:
            bad += 1
    return recs, bad

def final_attempts(recs):
    by = defaultdict(dict)
    for r in recs:
        by[(r["task"], r["attempt"])][r["event"]] = r
    best = {}
    for (task, attempt), ev in by.items():
        if "start" in ev and "end" in ev and attempt >= best.get(task, (-1, None))[0]:
            best[task] = (attempt, ev)
    return {t: {"benchmark": ev["end"]["benchmark"], "attempt": a, "status": ev["end"]["status"],
                "duration": ev["end"]["ts"] - ev["start"]["ts"]} for t, (a, ev) in best.items()}

def report(lines):
    recs, bad = parse(lines)
    final = final_attempts(recs)
    out = defaultdict(lambda: {"pass": 0, "fail": 0, "error": 0})
    for f in final.values():
        out[f["benchmark"]][f["status"]] += 1
    started = {r["task"] for r in recs}
    scores = {b: {"pass_rate": round(c["pass"] / (c["pass"] + c["fail"]), 3) if c["pass"] + c["fail"] else None,
                  "errors": c["error"]} for b, c in sorted(out.items())}
    return {"scores": scores, "malformed": bad, "incomplete": sorted(started - set(final))}

def ev(ts, task, attempt, event, status=None, bench="mmlu"):
    d = {"ts": ts, "task": task, "benchmark": bench, "attempt": attempt, "event": event}
    if status:
        d["status"] = status
    return json.dumps(d)

log = [
    ev(5, "t1", 0, "end", "fail"), ev(1, "t1", 0, "start"),
    ev(6, "t1", 1, "start"), ev(9, "t1", 1, "end", "pass"),
    ev(2, "t2", 0, "start"), ev(4, "t2", 0, "end", "fail"),
    ev(3, "t3", 0, "start", bench="gsm8k"), ev(8, "t3", 0, "end", "error", bench="gsm8k"),
    ev(7, "t4", 0, "start"),
    "{not json", json.dumps({"ts": "x", "task": "t5"}), ev(1, "t6", 0, "end", "maybe"),
]
recs, bad = parse(log)
final = final_attempts(recs)
assert final["t1"] == {"benchmark": "mmlu", "attempt": 1, "status": "pass", "duration": 3}
assert final["t2"]["duration"] == 2
r = report(log)
assert r["scores"] == {"gsm8k": {"pass_rate": None, "errors": 1}, "mmlu": {"pass_rate": 0.5, "errors": 0}}
assert r["malformed"] == 3 and r["incomplete"] == ["t4"]
```

Verify: the log has an end before its start, a retry that turns a fail into a pass, a benchmark with only an infrastructure error, a task that never ended, and three kinds of malformed line. Each part's assertion stays valid as the next part arrives, because state is keyed by `(task, attempt)`.

Pitfalls:
- Pairing start and end by line order, which breaks on the first out-of-order line.
- Counting `error` as a failure, which blames the model for an infrastructure problem.
- Crashing on the first bad line instead of counting it, or dropping bad lines without saying how many.

Follow-ups: for a 50 GB log, stream it and keep only open `(task, attempt)` pairs in memory; the pass and fail counts are small. What if two `end` lines exist for the same attempt? Pick a rule (first or last by `ts`) and test it.

Why this is likely: a 2026 senior SWE candidate report (LeetCode Discuss) describes a multi-part Python parsing problem in an NVIDIA screen, and this team's product is evaluation results; medium confidence in the format and the domain.

</details>

<a id="q3"></a>

#### Q3. (Technical screen or OA, no AI) Order the stages of an eval pipeline, run independent stages in parallel waves, and reject cycles

Round: Technical screen or online assessment. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Topological sort, cycle detection.

Related practice: [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) (Medium)

<details>
<summary>Show answer</summary>

Task: an eval pipeline is a set of stages (`download`, `build_env`, `serve_model`, `run_mmlu`, `run_gsm8k`, `score`, `report`) with "runs after" dependencies. Return the stages as waves. Each wave holds stages whose dependencies are all in earlier waves, so a wave can run in parallel. If there is a cycle, raise an error that names the stages in it. Unknown dependencies are an error too.

What they grade: Kahn's algorithm or DFS without help, the cycle case, deterministic output, and the complexity.

Approach: Kahn's algorithm by levels. Count incoming edges for each stage; the first wave is every stage with zero. Removing a wave lowers its children's counts, and the children that reach zero form the next wave. If some stages are never emitted, they are in or behind a cycle. Sort each wave so the output is stable for tests and logs. Time is O(V + E) plus the sort of each wave; space O(V + E).

Code:

```python
from collections import defaultdict

def waves(deps):
    for s, ds in deps.items():
        unknown = set(ds) - set(deps)
        if unknown:
            raise ValueError(f"{s} depends on unknown stage(s) {sorted(unknown)}")
    indeg = {s: len(set(ds)) for s, ds in deps.items()}
    children = defaultdict(list)
    for s, ds in deps.items():
        for d in set(ds):
            children[d].append(s)
    out, wave = [], sorted(s for s, n in indeg.items() if n == 0)
    while wave:
        out.append(wave)
        nxt = []
        for s in wave:
            for c in children[s]:
                indeg[c] -= 1
                if indeg[c] == 0:
                    nxt.append(c)
        wave = sorted(nxt)
    stuck = sorted(s for s, n in indeg.items() if n > 0)
    if stuck:
        raise ValueError(f"cycle among {stuck}")
    return out

pipeline = {
    "download": [], "build_env": [], "serve_model": ["build_env"],
    "run_mmlu": ["download", "serve_model"], "run_gsm8k": ["download", "serve_model"],
    "score": ["run_mmlu", "run_gsm8k"], "report": ["score"],
}
assert waves(pipeline) == [["build_env", "download"], ["serve_model"], ["run_gsm8k", "run_mmlu"], ["score"], ["report"]]
assert waves({}) == []
assert waves({"a": ["b", "b"], "b": []}) == [["b"], ["a"]]
for bad, msg in [({"a": ["b"], "b": ["a"], "c": []}, "cycle among ['a', 'b']"), ({"a": ["zzz"]}, "unknown")]:
    try:
        waves(bad)
        raise AssertionError
    except ValueError as e:
        assert msg in str(e), e
```

Verify: the sample pipeline, an empty graph, a duplicate dependency (which would double count the in-degree without the `set`), a two-stage cycle, and an unknown stage. The cycle report lists stages in a cycle plus any stage downstream of it; to print only the cycle, run a DFS from a stuck stage.

Pitfalls:
- DFS recursion on a 10,000-stage graph hitting Python's default recursion limit of 1,000.
- Forgetting duplicates or unknown names, which are common in hand-written configs.
- Returning one flat order when the question asks what can run in parallel.

Follow-ups: DFS with three colors gives the same order in O(V + E) and finds the exact cycle path, but it gives no waves. Waves leave work on the table when one stage in a wave takes 40 minutes and the rest take 1; a real executor starts each stage as soon as its own count reaches zero, which is the same algorithm driven by completion events. How would you add retries and skip the children of a failed stage?

Why this is likely: one 2026 report, posted by a proxy-interview service, describes a dependency-aware task execution engine in an NVIDIA screen, and eval pipelines with dependent stages are this team's daily work; the LeetCode link is practice for the same idea, not a reported question. Medium-low confidence.

</details>

### Onsite coding (no AI, as at NVIDIA)

<a id="q4"></a>

#### Q4. (Onsite coding, no AI) Return the K most common failure signatures across millions of eval samples

Round: Onsite coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Normalization, counting, heaps.

Related practice: [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) (Medium)

<details>
<summary>Show answer</summary>

Task: each failed sample carries an error message such as `Timeout after 30.5s calling tool search (req 9f3a2c)` or `KeyError: 'answer' in scorer gsm8k_v2`. Group messages that differ only in numbers, hex ids or quoted values into one signature, and return the K most common signatures with counts. Ties break alphabetically. Then discuss the version where messages arrive as a stream across 64 workers.

What they grade: a sensible normalization, the right data structure for top K, tie handling, and the complexity of each choice.

Approach: normalize with a few ordered regex rules (quoted strings, hex ids of six or more characters with at least one letter, then numbers not glued to a letter), count with a dictionary, then select. A heap of size K gives O(n + m log K) for n messages and m distinct signatures. Sorting all signatures is O(m log m), which is fine when m is small. Bucket sort by count is O(n + m), which is the LeetCode answer, but it needs care for ties.

Code:

```python
import heapq, re
from collections import Counter

RULES = [
    (re.compile(r"'[^']*'|\"[^\"]*\""), "<str>"),
    (re.compile(r"\b(?=[0-9a-f]*[a-f])[0-9a-f]{6,}\b"), "<hex>"),
    (re.compile(r"(?<![A-Za-z_])\d+(\.\d+)?"), "<n>"),
]

def signature(msg):
    for pat, repl in RULES:
        msg = pat.sub(repl, msg)
    return msg.strip()

def top_k(messages, k):
    counts = Counter(signature(m) for m in messages)
    return heapq.nsmallest(k, counts.items(), key=lambda kv: (-kv[1], kv[0]))

def merge(partials, k):
    total = Counter()
    for p in partials:
        total.update(p)
    return heapq.nsmallest(k, total.items(), key=lambda kv: (-kv[1], kv[0]))

msgs = [
    "Timeout after 30.5s calling tool search (req 9f3a2c)",
    "Timeout after 12s calling tool search (req 77aa01)",
    "KeyError: 'answer' in scorer gsm8k_v2",
    "KeyError: 'final' in scorer gsm8k_v2",
    "KeyError: 'answer' in scorer gsm8k_v2",
    "HTTP 429 from endpoint",
    "HTTP 503 from endpoint",
    "context length 131072 exceeded",
]
assert signature(msgs[0]) == "Timeout after <n>s calling tool search (req <hex>)"
assert top_k(msgs, 2) == [("KeyError: <str> in scorer gsm8k_v2", 3), ("HTTP <n> from endpoint", 2)]
assert top_k(msgs, 10)[2] == ("Timeout after <n>s calling tool search (req <hex>)", 2)
assert signature(msgs[-1]) == "context length <n> exceeded"
assert top_k([], 3) == []
parts = [Counter(signature(m) for m in msgs[:4]), Counter(signature(m) for m in msgs[4:])]
assert merge(parts, 2) == top_k(msgs, 2)
```

Verify: the samples cover a float, a hex id, quoted values, a name with digits inside it (`gsm8k_v2` stays intact because a digit after a letter is not replaced), an all-digit number that must not look like a hex id, a tie between two signatures with count 2, and an empty input. The merge test shows the distributed version gives the same answer as one machine.

Pitfalls:
- Merging only each worker's local top K. A signature that is fourth on every worker can be first overall, so workers must send full counts, or a count-min sketch if the signature set is huge.
- Normalizing too much, for example all identifiers, which merges different bugs into one bucket.
- A heap keyed only on count, which makes ties depend on dictionary order.

Follow-ups: for a stream with unbounded distinct signatures, a count-min sketch with a heap of K candidates uses fixed memory (for example 4 rows of 2,048 counters) and overestimates counts slightly. Space-Saving or Misra-Gries give a guaranteed error bound (at most n/K per count) with K counters. Which would you ship for a dashboard that refreshes every minute?

Why this is likely: one prep guide reports two LeetCode medium problems at NVIDIA senior onsites, and Top K Frequent Elements is in the crowd-reported NVIDIA LeetCode list for the last three to six months; this version ties the classic problem to failure triage on an eval platform. Medium-low confidence.

</details>

<a id="q5"></a>

#### Q5. Build an async eval runner with bounded concurrency, timeouts, retries and resume from a checkpoint

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: asyncio, failure handling, checkpointing.

<details>
<summary>Show answer</summary>

Task: write `run_eval(samples, solve, ckpt_path)`. `solve` is an async function that calls a model endpoint for one sample. Requirements: at most N samples in flight; a timeout per sample; retry transient errors (a `Transient` exception or a timeout) with backoff; record hard errors without retrying; append each result to a JSONL checkpoint as soon as it finishes; and on restart, skip samples that already succeeded but run errored ones again.

What they grade: correct use of a semaphore, timeouts and exceptions in asyncio, a checkpoint that survives a crash in the middle of a write, and tests that run in milliseconds.

Approach: the semaphore wraps the whole sample, including its retries, so the limit means "endpoint calls in flight". Results are written as one line per sample and flushed, so a crash loses at most the samples in flight. On load, the last record for each id wins, and a torn final line (half written at the crash) is skipped. Before appending, add a newline if the file does not end with one, or the next record glues onto the torn line. Writes need no lock, because each `write` is a synchronous call inside one event loop, so two coroutines cannot interleave inside it.

Code:

```python
import asyncio, json, os, tempfile

class Transient(Exception):
    pass

def load_checkpoint(path):
    done = {}
    if os.path.exists(path):
        with open(path) as f:
            for line in f:
                try:
                    rec = json.loads(line)
                    done[rec["id"]] = rec
                except (json.JSONDecodeError, KeyError):
                    pass
    return done

async def run_eval(samples, solve, ckpt_path, concurrency=4, timeout=5.0, retries=2, backoff=0.1):
    prior = load_checkpoint(ckpt_path)
    todo = [s for s in samples if prior.get(s["id"], {}).get("status") != "ok"]
    if os.path.exists(ckpt_path) and os.path.getsize(ckpt_path):
        with open(ckpt_path, "rb") as f:
            f.seek(-1, os.SEEK_END)
            torn = f.read(1) != b"\n"
    else:
        torn = False
    sem = asyncio.Semaphore(concurrency)
    with open(ckpt_path, "a") as out:
        if torn:
            out.write("\n")

        async def one(s):
            async with sem:
                for attempt in range(retries + 1):
                    try:
                        rec = {"id": s["id"], "status": "ok", "output": await asyncio.wait_for(solve(s), timeout)}
                        break
                    except (Transient, asyncio.TimeoutError) as e:
                        rec = {"id": s["id"], "status": "error", "error": type(e).__name__, "attempts": attempt + 1}
                        if attempt < retries:
                            await asyncio.sleep(backoff * 2 ** attempt)
                    except Exception as e:
                        rec = {"id": s["id"], "status": "error", "error": type(e).__name__, "attempts": attempt + 1}
                        break
            out.write(json.dumps(rec) + "\n")
            out.flush()
            return rec

        fresh = await asyncio.gather(*(one(s) for s in todo))
    results = dict(prior)
    results.update({r["id"]: r for r in fresh})
    return results

calls, in_flight, peak = {}, 0, 0
flaky = {"s1": 1}

async def solve(s):
    global in_flight, peak
    calls[s["id"]] = calls.get(s["id"], 0) + 1
    in_flight += 1
    peak = max(peak, in_flight)
    try:
        await asyncio.sleep(0.001)
        if s["id"] == "s2":
            await asyncio.sleep(10)
        if s["id"] == "s3":
            raise ValueError("bad prompt template")
        if flaky.get(s["id"], 0) > 0:
            flaky[s["id"]] -= 1
            raise Transient()
        return s["id"].upper()
    finally:
        in_flight -= 1

samples = [{"id": f"s{i}"} for i in range(8)]
with tempfile.TemporaryDirectory() as d:
    path = os.path.join(d, "ckpt.jsonl")
    res = asyncio.run(run_eval(samples, solve, path, concurrency=2, timeout=0.05, retries=1, backoff=0))
    assert res["s1"]["output"] == "S1" and calls["s1"] == 2
    assert res["s2"] == {"id": "s2", "status": "error", "error": "TimeoutError", "attempts": 2}
    assert res["s3"]["attempts"] == 1 and calls["s3"] == 1
    assert peak <= 2
    with open(path, "a") as f:
        f.write('{"id": "s7", "sta')
    calls.clear()
    res2 = asyncio.run(run_eval(samples, solve, path, concurrency=2, timeout=0.05, retries=0))
    assert sorted(calls) == ["s2", "s3"]
    assert sum(r["status"] == "ok" for r in res2.values()) == 6
    assert load_checkpoint(path)["s4"]["output"] == "S4"
```

Verify: the test runs in well under a second. It checks a transient error that succeeds on retry, a hang that becomes a timeout after two attempts, a hard error that is not retried, the concurrency peak, and a resume after a torn write where only the two errored samples run again.

Pitfalls:
- Catching the builtin `TimeoutError` on Python 3.10 or older, where `asyncio.wait_for` raises `asyncio.TimeoutError`, a different class until 3.11.
- Putting the semaphore around task creation instead of the call, which limits nothing.
- Writing the checkpoint only at the end, so a crash at sample 9,999 of 10,000 loses the whole run.

Follow-ups: `asyncio.gather` holds all coroutines in memory; for 1 million samples, use a fixed pool of N workers that pull from a queue, which keeps memory flat. A `TaskGroup` (3.11 and later) cancels siblings on the first unhandled error, which is the wrong default for an eval where one bad sample should not stop 10,000. Retrying non-idempotent tool calls needs an idempotency key.

Why this is likely: the listing asks for distributed job orchestration and reliability, NeMo Evaluator advertises per-benchmark checkpointing with `--resume`, and one 2026 report (posted by a proxy-interview service, low confidence) describes a concurrent task execution engine in an NVIDIA screen; medium confidence.

</details>

<a id="q6"></a>

#### Q6. This async eval runner reports the wrong accuracy. Find the bugs

Round: Onsite coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: asyncio ordering, exceptions, shared state.

<details>
<summary>Show answer</summary>

Task: a teammate says model A, which looks right on every sample when checked by hand, scores 0.0, while model B, which always answers "wrong", sometimes scores above zero in the next call. Below are the runner and a small run that prints B's score. The score is 0.0 or 0.25, depending on the run and the Python version. Find the bugs, explain each one, and fix them.

```python
import asyncio

async def run_eval(samples, solve, score, concurrency=8, cache={}):
    sem = asyncio.Semaphore(concurrency)

    async def one(s):
        if s["id"] in cache:
            return cache[s["id"]]
        async with sem:
            out = await solve(s)
        cache[s["id"]] = out
        return out

    results = {}
    for i, fut in enumerate(asyncio.as_completed([one(s) for s in samples])):
        try:
            out = await fut
        except Exception as e:
            out = e
        results[samples[i]["id"]] = score(samples[i], out) if out else 0
    return sum(results.values()) / len(samples)

samples = [{"id": i, "answer": str(i)} for i in range(4)]
score = lambda s, out: int(out == s["answer"])

async def model_a(s):
    await asyncio.sleep(0.01 * (4 - s["id"]))
    if s["id"] == 2:
        raise RuntimeError("endpoint 503")
    return s["answer"]

calls_b = []
async def model_b(s):
    calls_b.append(s["id"])
    return "wrong"

assert asyncio.run(run_eval(samples, model_a, score)) == 0.0
print("model B accuracy:", asyncio.run(run_eval(samples, model_b, score)))
assert calls_b == [2]
```

Approach: read the symptoms first. "A model that always answers wrong scores above zero" points at shared state; "changes from run to run" points at ordering. Then walk the code with the small run.

1. `asyncio.as_completed` yields futures in completion order, not input order, so `samples[i]` is the wrong sample. Model A gets every non-failing sample right, yet scores 0.0, because every result is matched to the wrong sample.
2. `cache={}` is a mutable default, created once per process. The second call with model B reads model A's cached outputs, so model B is called only for the sample that errored (`calls_b == [2]`), and its score comes from model A's answers matched to the wrong samples. The cache key also ignores the model.
3. An exception is truthy, so `score(sample, exception)` runs and counts an endpoint 503 as a wrong answer. Report infrastructure errors in their own count, apart from accuracy.
4. `if out else 0` skips the scorer for an empty string, which some scorers treat as a valid answer (for example "no tool call needed").

Code:

```python
import asyncio

async def run_eval(samples, solve, score, concurrency=8, cache=None):
    cache = {} if cache is None else cache
    sem = asyncio.Semaphore(concurrency)

    async def one(s):
        if s["id"] in cache:
            return cache[s["id"]]
        async with sem:
            out = await solve(s)
        cache[s["id"]] = out
        return out

    outs = await asyncio.gather(*(one(s) for s in samples), return_exceptions=True)
    scores, errors = [], []
    for s, out in zip(samples, outs):
        if isinstance(out, Exception):
            errors.append((s["id"], type(out).__name__))
        else:
            scores.append(score(s, out))
    acc = sum(scores) / len(scores) if scores else None
    return {"accuracy": acc, "scored": len(scores), "errors": errors}

samples = [{"id": i, "answer": str(i)} for i in range(4)]
score = lambda s, out: int(out == s["answer"])

async def model_a(s):
    await asyncio.sleep(0.01 * (4 - s["id"]))
    if s["id"] == 2:
        raise RuntimeError("endpoint 503")
    return s["answer"]

async def model_b(s):
    return "wrong"

ra = asyncio.run(run_eval(samples, model_a, score))
rb = asyncio.run(run_eval(samples, model_b, score))
assert ra == {"accuracy": 1.0, "scored": 3, "errors": [(2, "RuntimeError")]}
assert rb == {"accuracy": 0.0, "scored": 4, "errors": []}
```

Verify: the same small run now gives model A 1.0 on the 3 scored samples with one endpoint error listed, and model B 0.0. Add a test that runs two models in a row in one process, since that is exactly what the mutable default broke, and a test where completion order is the reverse of input order.

Pitfalls:
- Fixing only the ordering bug and missing the cache, because the first symptom disappears.
- Dividing by `len(samples)` when some failed, which mixes a model's quality with the endpoint's uptime.
- Assuming `as_completed` is ordered because the test endpoint always answered in order.

Follow-ups: `as_completed` is still useful for progress bars and early checkpointing; carry the sample with the result (return `(s, out)`) instead of relying on order. How do you report a run where 30% of samples errored? (Mark it invalid above a threshold, for example 2%, instead of publishing an accuracy.)

Why this is likely: a NeMo team candidate described a live pull request review with bug finding, a recent Blind invite listed a "bug bash" round, and this team's runner is async Python; medium confidence in the format and the content.

</details>

<a id="q7"></a>

#### Q7. A teammate's coding assistant wrote this Pydantic trace normalizer. Review it before it merges

Round: Onsite coding. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: Pydantic v2, hallucinated APIs, data safety.

<details>
<summary>Show answer</summary>

Task: the pull request says "normalize raw spans from agent runs into a tree and redact secrets; generated with an assistant, tests pass locally". The project uses Pydantic v2, and spans come from several exporters, some of which sample or drop spans. Review it out loud: list the defects by severity, then write the fix.

```python
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel, validator

class Span(BaseModel):
    span_id: str
    trace_id: str
    parent_id: Optional[str]
    name: str
    start: datetime
    end: datetime
    attributes: dict = {}
    children: List["Span"] = []

    @validator("end")
    def end_after_start(cls, v, values):
        if v < values["start"]:
            raise ValueError("end before start")
        return v

def normalize(raw_spans):
    spans = Span.model_validate_many(raw_spans)
    by_id = {s.span_id: s for s in spans}
    roots = []
    for s in spans:
        if s.parent_id:
            by_id[s.parent_id].children.append(s)
        else:
            roots.append(s)
    for r in roots:
        redact(r)
    return roots

def redact(span):
    for k in span.attributes:
        if "key" in k:
            span.attributes[k] = "***"
```

Approach: check that it runs, then correctness, then safety, then style. Findings, most severe first:

1. Hallucinated API. `BaseModel.model_validate_many` does not exist in Pydantic v2, so every call raises `AttributeError`. "Tests pass locally" means the tests never called `normalize`. Use `TypeAdapter(List[Span])`.
2. Secrets leak. `"key" in k` misses `authorization`, `api_token` and `password`, flags harmless keys such as `monkey_count`, ignores nested dicts, and only visits roots, so every child span keeps its secrets.
3. Orphans crash the run. Exporters drop or sample spans, so `by_id[s.parent_id]` raises `KeyError` and one missing parent loses the whole trace. Keep orphans as roots and count them.
4. Root spans are rejected. In v2, `Optional[str]` with no default is a required field, so an exporter that omits `parent_id` on the root fails validation.
5. Mixed time zones. Plain `datetime` accepts naive and aware values; comparing them raises `TypeError`. Use `AwareDatetime`.
6. The v1 `@validator` is deprecated in v2, and `values["start"]` raises `KeyError` when `start` itself failed, hiding the original validation error. Use a `model_validator(mode="after")`.
7. Unknown fields are dropped silently by default, so an exporter typo loses data. Decide on `extra="forbid"` for this internal schema.

Not a defect: `attributes: dict = {}` looks like the mutable default bug, but Pydantic copies field defaults per instance. Saying so shows you know how the library handles defaults.

Code:

```python
import re
from typing import Any, Dict, List, Optional
from pydantic import AwareDatetime, BaseModel, ConfigDict, Field, TypeAdapter, ValidationError, model_validator

SECRET = re.compile(r"(^|_)(api_?key|token|secret|password|authorization|cookie)($|_)", re.I)

class Span(BaseModel):
    model_config = ConfigDict(extra="forbid")
    span_id: str
    trace_id: str
    parent_id: Optional[str] = None
    name: str
    start: AwareDatetime
    end: AwareDatetime
    attributes: Dict[str, Any] = Field(default_factory=dict)

    @model_validator(mode="after")
    def end_after_start(self):
        if self.end < self.start:
            raise ValueError(f"span {self.span_id}: end before start")
        return self

class Node(BaseModel):
    span: Span
    children: List["Node"] = Field(default_factory=list)

SPANS = TypeAdapter(List[Span])

def redact(value):
    if isinstance(value, dict):
        return {k: "***" if SECRET.search(k) else redact(v) for k, v in value.items()}
    if isinstance(value, list):
        return [redact(v) for v in value]
    return value

def normalize(raw_spans):
    nodes = {}
    for s in SPANS.validate_python(raw_spans):
        if s.span_id in nodes:
            raise ValueError(f"duplicate span_id {s.span_id}")
        nodes[s.span_id] = Node(span=s.model_copy(update={"attributes": redact(s.attributes)}))
    roots, orphans = [], 0
    for n in nodes.values():
        parent = n.span.parent_id
        if parent in nodes:
            nodes[parent].children.append(n)
        else:
            orphans += parent is not None
            roots.append(n)
    return roots, orphans

T = "2026-01-01T00:00:0{}Z"
raw = [
    {"span_id": "a", "trace_id": "t", "name": "agent", "start": T.format(0), "end": T.format(9),
     "attributes": {"Authorization": "Bearer x", "monkey_count": 3, "tokens_used": 812}},
    {"span_id": "b", "trace_id": "t", "parent_id": "a", "name": "tool", "start": T.format(1), "end": T.format(2),
     "attributes": {"request": {"openai_api_key": "sk-1", "q": "gpu"}}},
    {"span_id": "c", "trace_id": "t", "parent_id": "lost", "name": "llm", "start": T.format(3), "end": T.format(4)},
]
roots, orphans = normalize(raw)
assert [r.span.span_id for r in roots] == ["a", "c"] and orphans == 1
assert roots[0].span.attributes == {"Authorization": "***", "monkey_count": 3, "tokens_used": 812}
assert roots[0].children[0].span.attributes["request"] == {"openai_api_key": "***", "q": "gpu"}
assert not hasattr(Span, "model_validate_many")
for bad in [dict(raw[2], start="2026-01-01T00:00:03"), dict(raw[2], end=T.format(0)), dict(raw[2], colour="x")]:
    try:
        normalize([bad])
        raise AssertionError(bad)
    except ValidationError:
        pass
```

Verify: the tests cover a root with no `parent_id`, a nested secret in a child, harmless keys that contain "key" or "token" as part of a word, an orphan, a naive timestamp, an end before its start, and an unknown field. Each of findings 1 to 6 has a test that fails on the original code.

Pitfalls:
- Commenting on style while the code cannot run at all.
- Trusting "tests pass" in the description without looking for a test that calls the function.
- Flagging the Pydantic mutable default as a bug, which costs credibility for the other findings.

Follow-ups: should redaction live in the exporter instead, so secrets never leave the process? (Yes for defense in depth, but keep it here as a second line, since you do not control every exporter.) A regex denylist misses new key names; an allowlist of known attribute keys is safer but drops new fields until someone adds them.

Why this is likely: a NeMo team candidate described a live pull request review where the task was to find bugs, and the listing asks for Pydantic and typed trace schemas; the AI-written framing is illustrative, since teams now review AI-generated pull requests at work. Medium confidence.

</details>

<a id="q8"></a>

#### Q8. Build the `@benchmark` and `@scorer` plugin registry for an eval SDK

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Decorators, plugin discovery, API errors.

<details>
<summary>Show answer</summary>

Task: users add benchmarks and scorers to the SDK by decorating functions, `@benchmark("gsm8k")` and `@scorer("exact_match")`, in their own packages. Build the registry: registration, lookup by name, a clear error for duplicates, a check that the function has the right parameters, and lazy loading of third-party plugins so `import sdk` stays fast.

What they grade: decorator mechanics, error messages a user can act on, import-time cost, and tests that do not depend on installed packages.

Approach: one `Registry` class per plugin kind holds a name to function map. The decorator checks the function's leading parameters with `inspect.signature` at registration time, so a wrong signature fails when the plugin loads, before a 3-hour run starts. A duplicate name from a different function raises an error naming both modules; registering the same function twice (a module imported twice in a notebook, for example) is allowed, judged by module and qualified name. Third-party plugins load lazily. An entry point group (for example `myeval.benchmarks`) maps names to `module:attr` strings, and `get` imports the module only on first use. The loader is injected so tests can fake it.

Code:

```python
import inspect

class RegistryError(Exception):
    pass

class Registry:
    def __init__(self, kind, required, loader=None, lazy=None):
        self.kind, self.required = kind, tuple(required)
        self.entries, self.lazy, self.loader = {}, dict(lazy or {}), loader

    def __call__(self, name=None):
        def deco(fn):
            key = name or fn.__name__
            params = tuple(inspect.signature(fn).parameters)[:len(self.required)]
            if params != self.required:
                raise RegistryError(f"{self.kind} {key!r} must take {self.required}, got {params}")
            old = self.entries.get(key)
            if old is not None and (old.__module__, old.__qualname__) != (fn.__module__, fn.__qualname__):
                raise RegistryError(f"{self.kind} {key!r} is defined in both {old.__module__} and {fn.__module__}")
            self.entries[key] = fn
            return fn
        return deco

    def get(self, name):
        if name not in self.entries and name in self.lazy:
            self.loader(self.lazy.pop(name))
        if name not in self.entries:
            known = sorted(set(self.entries) | set(self.lazy))
            raise RegistryError(f"unknown {self.kind} {name!r}; known: {known}")
        return self.entries[name]

loaded = []
def fake_loader(target):
    loaded.append(target)
    @benchmark("humaneval")
    def humaneval(config):
        return ["def f(): ..."]

benchmark = Registry("benchmark", ["config"], loader=fake_loader, lazy={"humaneval": "acme_evals.code:humaneval"})
scorer = Registry("scorer", ["sample", "output"])

@benchmark("gsm8k")
def gsm8k(config, split="test"):
    return [{"q": "2+2", "a": "4"}]

@scorer()
def exact_match(sample, output):
    return float(output.strip() == sample["a"])

assert benchmark.get("gsm8k")({}) == [{"q": "2+2", "a": "4"}]
assert scorer.get("exact_match")({"a": "4"}, " 4 ") == 1.0
assert loaded == [] and benchmark.get("humaneval")({}) and loaded == ["acme_evals.code:humaneval"]
benchmark("gsm8k")(gsm8k)

def clash(config):
    return []
clash.__module__ = "other_pkg.evals"
cases = [
    (lambda: benchmark("gsm8k")(clash), "defined in both __main__ and other_pkg.evals"),
    (lambda: scorer("bad")(lambda output, sample: 0), "must take ('sample', 'output')"),
    (lambda: scorer.get("bleu"), "known: ['exact_match']"),
]
for call, msg in cases:
    try:
        call()
        raise AssertionError(msg)
    except RegistryError as e:
        assert msg in str(e), e
```

Verify: the tests cover lookup, default names, lazy loading that happens once and only on use, re-registering the same function, a name clash between two modules, swapped parameters, and an unknown name that lists what exists.

Pitfalls:
- Importing every installed plugin at `import sdk` time; ten plugins that each import a large library can add seconds to every CLI call.
- A silent overwrite on duplicate names, so the benchmark you get depends on import order.
- Checking signatures only when the benchmark runs, after the cluster job has been scheduled.

Follow-ups: a base class with abstract methods gives type checkers more to work with than a decorated function, at the cost of boilerplate for a 5-line scorer; many libraries offer both. Entry points need `pip install` to register, which is awkward for a notebook, so also accept a `module:attr` string on the command line. How would you version the plugin interface when the scorer signature gains a parameter?

Why this is likely: NeMo Evaluator documents `@benchmark` and `@scorer` decorators and pluggable solvers, and the listing asks for SDK, plugin and CLI design in Python; medium-high confidence.

</details>

### Domain deep dive (no AI, as at NVIDIA)

<a id="q9"></a>

#### Q9. Tool-calling success fell from 71% to 58% after an upgrade. Read the traces and find the cause

Round: Domain deep dive. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Agent traces, root cause, eval hygiene.

<details>
<summary>Show answer</summary>

Task: the nightly tool-calling benchmark (500 tasks, same model, same seed, same scorer version) dropped from 71% to 58% after the eval SDK went from 2.3 to 2.4. You get the failure breakdown and one failing trace. Find the cause, prove it, and say what you would change so this is caught before a release.

| Failure reason | SDK 2.3 | SDK 2.4 |
|---|---|---|
| wrong final answer | 21% | 20% |
| hit max_turns (10) | 4% | 18% |
| tool error | 3% | 3% |
| endpoint error, excluded | 1% | 1% |

```text
step 3  tool_call    search_docs {"q": "cuda graph capture limits"}
step 3  tool_result  search_docs chars=4096 tail='..."score": 0.8'
step 4  llm          "The search result seems cut off. Let me search again."
step 5  tool_call    search_docs {"q": "cuda graph capture limits"}
step 5  tool_result  search_docs chars=4096 tail='..."score": 0.8'
...
step 10 end          status=fail reason=max_turns
```

Approach: narrow before guessing. Wrong answers and tool errors are flat, so the model's reasoning and the tools look unchanged; the whole drop is in max_turns. The trace shows the agent calling the same tool with the same arguments after a result of exactly 4,096 characters that ends in the middle of the JSON. A result of exactly 4,096 characters points to a size limit. Hypothesis: 2.4 truncates tool results at 4,096 characters (the old limit was larger), the model sees broken JSON, and it retries until the turn limit.

Prove it across all the traces. Compare the share of tool results at exactly the limit, and the share of identical repeated calls that follow a truncated result, across both runs. Then confirm causally: rerun only the 65 newly failing tasks on 2.4 with the old limit. If most of them pass, that is the cause.

Code:

```python
from collections import Counter

def analyze(events, limit=4096):
    by_task = {}
    for e in events:
        by_task.setdefault(e["task"], []).append(e)
    stats = Counter()
    for evs in by_task.values():
        evs.sort(key=lambda e: e["step"])
        last_call, truncated = None, False
        for e in evs:
            if e["type"] == "tool_result":
                stats["results"] += 1
                truncated = e["chars"] == limit
                stats["truncated"] += truncated
            elif e["type"] == "tool_call":
                key = (e["tool"], tuple(sorted(e["args"].items())))
                if key == last_call:
                    stats["repeat_calls"] += 1
                    stats["repeat_after_truncation"] += truncated
                last_call = key
            elif e["type"] == "end" and e["status"] == "fail":
                stats["fail_" + e["reason"]] += 1
    return {
        "truncated_share": round(stats["truncated"] / max(stats["results"], 1), 3),
        "repeats_after_truncation": round(stats["repeat_after_truncation"] / max(stats["repeat_calls"], 1), 3),
        "max_turn_fails": stats["fail_max_turns"],
    }

def task(tid, sizes, status, reason=None):
    evs, step = [], 0
    for chars in sizes:
        evs.append({"task": tid, "step": step, "type": "tool_call", "tool": "search_docs", "args": {"q": "cuda graphs"}})
        evs.append({"task": tid, "step": step, "type": "tool_result", "chars": chars})
        step += 1
    evs.append({"task": tid, "step": step, "type": "end", "status": status, "reason": reason})
    return evs

old = task("a", [6100], "pass") + task("b", [900, 900], "fail", "wrong_answer")
new = task("a", [4096] * 5, "fail", "max_turns") + task("b", [900, 900], "fail", "wrong_answer")
assert analyze(old) == {"truncated_share": 0.0, "repeats_after_truncation": 0.0, "max_turn_fails": 0}
assert analyze(new) == {"truncated_share": 0.714, "repeats_after_truncation": 0.8, "max_turn_fails": 1}
```

Verify: the toy runs show the pattern. On the new SDK, most tool results sit at the limit and most repeated calls follow a truncated result, while the old run has none. On the real data, the rerun with the old limit proves the cause; the trace statistics alone only show a correlation.

Pitfalls:
- Blaming the model because "the model looped". The model behaved sensibly on broken input.
- Fixing it by raising max_turns, which hides the problem and costs more tokens per task.
- Comparing the two runs without checking that the model, seed, scorer and dataset versions match first.

Follow-ups: the fix has options. Restore the old limit (simple, but long results cost context), truncate on a JSON boundary and add a `"truncated": true` marker the model can read, or page large results. Then add a release check that runs a 100-task canary on both SDK versions with the same model and blocks on a significant drop (see Q10), plus a trace metric for "results at exactly the limit". Why are 1% endpoint errors excluded from the score, and what if they rose to 10%?

Why this is likely: the listing asks for trace analysis and debugging of agent systems, NVIDIA's agent evaluation blog names tool call accuracy and trajectory efficiency as core metrics, and NeMo Agent Toolkit saves intermediate steps for this kind of review; medium-high confidence.

</details>

<a id="q10"></a>

#### Q10. Design the statistical gate that says GO, NO-GO or INCONCLUSIVE for a new model or agent version

Round: Domain deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Paired tests, McNemar, sample size.

<details>
<summary>Show answer</summary>

Task: a candidate agent version scored 70.4% against the baseline's 72.0% on the same 500 tasks. Product wants a yes or no. Design a gate that a CI job can run on every candidate, explain the statistics, and write the decision function.

What they grade: using the pairing, a test that fits pass or fail data, a margin set before the run, an honest third outcome, and sample size math.

Approach: both versions ran on the same tasks, so compare per task. Only tasks that flip carry information: b tasks went from fail to pass, c went from pass to fail. McNemar's exact test asks whether b and c differ more than a fair coin would explain; it is a binomial test on b out of b + c. Two independent proportions would ignore the pairing and, when most tasks agree, need several times more tasks for the same power (about 4 times here: 2 x 0.72 x 0.28 = 0.40 against a flip rate of about 0.10). For the decision, use a confidence interval on the paired difference and a non-inferiority margin agreed in advance, for example 2 points:

- NO-GO when McNemar says the drop is significant (p below 0.05 and c greater than b).
- GO when the whole interval lies above minus 2 points.
- INCONCLUSIVE otherwise, with the number of tasks needed to decide.

Code:

```python
from math import comb, sqrt

def mcnemar_exact(b, c):
    n = b + c
    if n == 0:
        return 1.0
    tail = sum(comb(n, i) for i in range(min(b, c) + 1)) / 2 ** n
    return min(1.0, 2 * tail)

def gate(base, cand, margin=0.02, alpha=0.05, z=1.96):
    assert len(base) == len(cand) and base, "runs must be paired on the same tasks"
    n = len(base)
    b = sum(1 for x, y in zip(base, cand) if not x and y)
    c = sum(1 for x, y in zip(base, cand) if x and not y)
    d = (b - c) / n
    se = sqrt(max(b + c - (b - c) ** 2 / n, 0)) / n
    lo, hi = d - z * se, d + z * se
    p = mcnemar_exact(b, c)
    if p < alpha and c > b:
        verdict = "NO-GO"
    elif lo >= -margin:
        verdict = "GO"
    else:
        verdict = "INCONCLUSIVE"
    return {"verdict": verdict, "diff": round(d, 4), "ci": (round(lo, 4), round(hi, 4)), "p": round(p, 4), "b": b, "c": c}

def paired(n, both, b, c):
    base = [1] * both + [0] * b + [1] * c + [0] * (n - both - b - c)
    cand = [1] * both + [1] * b + [0] * c + [0] * (n - both - b - c)
    return base, cand

r = gate(*paired(500, 332, 20, 28))
assert r["verdict"] == "INCONCLUSIVE" and r["diff"] == -0.016 and r["ci"] == (-0.0431, 0.0111)
assert 0.3 < r["p"] < 0.32
assert gate(*paired(2000, 1360, 80, 80))["verdict"] == "GO"
assert gate(*paired(500, 330, 10, 40))["verdict"] == "NO-GO"
assert mcnemar_exact(0, 0) == 1.0 and mcnemar_exact(5, 5) == 1.0
```

Verify: the 500-task case is the prompt: 20 tasks improved, 28 regressed, a 1.6 point drop, p about 0.31 and an interval from minus 4.3 to plus 1.1 points. That interval crosses the 2 point margin, so the gate returns INCONCLUSIVE. With 2,000 tasks and balanced flips the interval is about plus or minus 1.2 points, so GO. With 10 against 40 flips, NO-GO.

Pitfalls:
- Comparing 72.0 and 70.4 as two unrelated proportions, which throws away the pairing.
- Picking the margin after seeing the result.
- Running the gate on every one of 30 benchmarks at 0.05 and expecting no false alarms; with 30 tests, about 1.5 will trip by chance, so correct for it (Holm) or gate on a few primary suites.

Follow-ups: sample size. With a flip rate around 10% and a 2 point effect to detect at 80% power, n is about (1.96 + 0.84) squared x 0.10 / 0.02 squared, roughly 2,000 tasks. Agents are not deterministic at nonzero temperature, so run each task 3 times and use the per-task majority, or a paired bootstrap over tasks with per-task pass rates. The interval here is the simple Wald form; at small flip counts a bootstrap or exact interval is safer.

Why this is likely: NeMo Evaluator documents McNemar's exact test, paired flip analysis, confidence intervals and a quality gate that returns GO, NO-GO or INCONCLUSIVE, and the listing asks for evaluation and reliability work; high confidence in the topic.

</details>

<a id="q11"></a>

#### Q11. Your trace schema needs a breaking change. Version it in Pydantic without breaking old readers or old data

Round: Domain deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Schema evolution, discriminated unions.

<details>
<summary>Show answer</summary>

Task: version 1 trace events store tool arguments as a JSON string and latency in milliseconds, with a `kind` field. Version 2 needs typed arguments, durations in seconds, a `type` field and a usage object for LLM calls. Months of v1 data sit in storage, and third-party exporters keep writing v1 for a while. Design the versioning rules and write the reader.

What they grade: a clear rule for what needs a new version, upgrade on read instead of a risky rewrite, discriminated unions, and behavior on data from the future.

Approach: rules first. Additive changes (a new optional field) do not bump the version; readers ignore unknown fields, so an old reader keeps working. Renames, type changes and unit changes get a new `schema_version` and an upgrader from the previous version. Readers upgrade stored data when they read it, and a backfill job can rewrite cold data later while keeping the raw copy. Writers always emit the latest version. A reader that sees a version newer than it knows fails loudly with "upgrade the reader", because guessing would corrupt data silently. Inside a version, a discriminated union on `type` gives a fast parse and clear errors. Publish the JSON Schema of each version, and keep golden files from each version as contract tests.

Code:

```python
import json
from typing import Any, Dict, Literal, Union
from typing_extensions import Annotated
from pydantic import BaseModel, ConfigDict, Field, TypeAdapter, ValidationError

class Lenient(BaseModel):
    model_config = ConfigDict(extra="ignore")

class V1Tool(Lenient):
    kind: Literal["tool"]
    tool: str
    args: str
    latency_ms: int

class V1LLM(Lenient):
    kind: Literal["llm"]
    model: str
    prompt_tokens: int
    completion_tokens: int
    latency_ms: int

class ToolCall(Lenient):
    schema_version: Literal[2] = 2
    type: Literal["tool_call"] = "tool_call"
    tool_name: str
    arguments: Dict[str, Any]
    duration_s: float

class LLMCall(Lenient):
    schema_version: Literal[2] = 2
    type: Literal["llm_call"] = "llm_call"
    model: str
    usage: Dict[str, int]
    duration_s: float

V1 = TypeAdapter(Annotated[Union[V1Tool, V1LLM], Field(discriminator="kind")])
V2 = TypeAdapter(Annotated[Union[ToolCall, LLMCall], Field(discriminator="type")])

def upgrade_v1(e):
    if isinstance(e, V1Tool):
        return ToolCall(tool_name=e.tool, arguments=json.loads(e.args), duration_s=e.latency_ms / 1000)
    return LLMCall(model=e.model, usage={"input": e.prompt_tokens, "output": e.completion_tokens},
                   duration_s=e.latency_ms / 1000)

READERS = {1: lambda raw: upgrade_v1(V1.validate_python(raw)), 2: V2.validate_python}

def read_event(raw):
    version = raw.get("schema_version", 1)
    if version not in READERS:
        raise ValueError(f"schema_version {version} is newer than this reader; upgrade the SDK")
    return READERS[version](raw)

t = read_event({"kind": "tool", "tool": "search", "args": '{"q": "gpu"}', "latency_ms": 250})
assert t == ToolCall(tool_name="search", arguments={"q": "gpu"}, duration_s=0.25)
m = read_event({"kind": "llm", "model": "m1", "prompt_tokens": 900, "completion_tokens": 40, "latency_ms": 1200})
assert m.usage == {"input": 900, "output": 40} and m.type == "llm_call"
stored = json.loads(t.model_dump_json())
assert stored["schema_version"] == 2 and read_event(stored) == t
assert read_event(dict(stored, cache_hit=True)) == t
for bad, err in [(dict(stored, schema_version=3), ValueError), (dict(stored, type="retrieval"), ValidationError),
                 ({"kind": "tool", "tool": "s", "args": "{oops", "latency_ms": 1}, ValueError)]:
    try:
        read_event(bad)
        raise AssertionError(bad)
    except err:
        pass
assert "discriminator" in V2.json_schema()
```

Verify: the tests read both v1 kinds and upgrade them, round-trip a v2 event through storage, accept an unknown additive field, reject a future version and an unknown event type, and catch a v1 record whose argument string is not valid JSON. That last case is real in old data, so the backfill job must count and quarantine those rows and keep going.

Pitfalls:
- Bumping the version for every additive field, so readers need a new release for every exporter change.
- `extra="forbid"` on a stored schema, which makes every new field a breaking change for old readers.
- Rewriting all stored data in one migration with no raw copy, so an upgrader bug becomes permanent.

Follow-ups: upgraders chain (v1 to v2 to v3), so each step stays small and testable, at the cost of read time on old data; a backfill removes that cost for hot data. OpenTelemetry semantic conventions solve a similar problem with attribute names and stability levels. Would you map these events onto OpenTelemetry spans instead of a custom schema? (Interop with existing tools, against less control over the shape.)

Why this is likely: the listing names Pydantic and typed trace schemas, and an evaluation platform that stores months of traces must evolve its schema without breaking users; medium-high confidence.

</details>

<a id="q12"></a>

#### Q12. Trace ingestion spends most of its time in Pydantic validation. Make it faster

Round: Domain deep dive. AI policy: Not allowed. Type: Optimize. Difficulty: Medium. Topic: Profiling, Pydantic v2, batching.

<details>
<summary>Show answer</summary>

Task: a worker ingests JSONL spans from eval runs at about 80,000 spans per second per core, and the target is 150,000. A profile says most of the time is inside Pydantic. Here is the current path: parse each line, validate, change one field by rebuilding the model, and validate again "to be safe" before writing.

What they grade: reading the profile before changing code, knowing where Pydantic v2 is fast (its Rust core) and where it is slow (Python validators, repeated round trips), keeping behavior identical, and handling bad lines.

Approach: three findings, all from the profile. First, every span is validated three times: once on parse, once when the model is rebuilt after the `name` change, and once before the write. Validate once at the boundary; inside the process the data is already trusted, so update with `model_copy`. Second, a wildcard `field_validator("*", mode="before")` runs Python code for every field of every span to strip strings; the built-in `str_strip_whitespace` setting does the same in the Rust core. Third, `json.loads` followed by validation builds Python dicts that are thrown away; `validate_json` parses and validates in one pass. Batching a whole chunk through one `TypeAdapter` call keeps the hot loop out of Python, but one bad line fails the whole batch, so fall back to per-line validation for that chunk and count the bad lines.

Code:

```python
import json, time
from typing import Any, Dict, List
from pydantic import BaseModel, ConfigDict, TypeAdapter, ValidationError, field_validator

class SlowSpan(BaseModel):
    span_id: str
    trace_id: str
    name: str
    start_ns: int
    end_ns: int
    attributes: Dict[str, Any]

    @field_validator("*", mode="before")
    @classmethod
    def strip(cls, v):
        return v.strip() if isinstance(v, str) else v

def ingest_slow(lines):
    out = []
    for line in lines:
        span = SlowSpan(**json.loads(line))
        span = SlowSpan(**{**span.model_dump(), "name": span.name.lower()})
        out.append(SlowSpan.model_validate(span.model_dump()).model_dump())
    return out

class Span(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)
    span_id: str
    trace_id: str
    name: str
    start_ns: int
    end_ns: int
    attributes: Dict[str, Any]

BATCH = TypeAdapter(List[Span])

def ingest_fast(lines):
    try:
        spans, bad = BATCH.validate_json("[" + ",".join(lines) + "]"), 0
    except ValidationError:
        spans, bad = [], 0
        for line in lines:
            try:
                spans.append(Span.model_validate_json(line))
            except ValidationError:
                bad += 1
    rows = [s.model_copy(update={"name": s.name.lower()}).model_dump() for s in spans]
    return rows, bad

lines = [json.dumps({"span_id": f"s{i}", "trace_id": f"t{i // 50}", "name": " Tool.Search ", "start_ns": i,
                     "end_ns": i + 5, "attributes": {"tool": "search", "tokens": i % 900}}) for i in range(20000)]
fast, bad = ingest_fast(lines)
assert fast == ingest_slow(lines) and bad == 0
assert fast[0]["name"] == "tool.search"
rows, bad = ingest_fast(lines[:3] + ['{"span_id": 1}'])
assert len(rows) == 3 and bad == 1
t0 = time.perf_counter(); ingest_slow(lines); t1 = time.perf_counter(); ingest_fast(lines); t2 = time.perf_counter()
print(f"slow {t1 - t0:.3f}s, fast {t2 - t1:.3f}s, speedup {(t1 - t0) / (t2 - t1):.1f}x")
```

Verify: the first assertion proves the fast path returns exactly what the slow path returned on 20,000 spans; check that before any timing. The second shows that a bad line costs only that span, and the rest of the chunk is kept. On a laptop with Pydantic 2.13 this sample runs about 2.2x faster (roughly 80,000 to 175,000 spans per second on one core); measure on your own data, since wide attribute maps change the ratio.

Pitfalls:
- Switching to `model_construct` everywhere to skip validation. It skips the checks that protect the store, and in our measurement it was slower (0.69 seconds against 0.54 for 100,000 spans with plain validation), because building the model in Python costs more than the Rust validation it skips.
- Optimizing before profiling; if the time were in gzip or the network, none of this would matter.
- Changing behavior silently, for example dropping whitespace stripping.

Follow-ups: past one core, run N worker processes that each take a file chunk; threads do not help much because validation holds the GIL. `orjson` speeds up the dump side. If the store accepts JSON, `model_dump_json` skips building Python dicts too. At what point would you move ingestion to a compiled service, and what would you lose?

Why this is likely: the listing asks for Python performance and Pydantic trace schemas at platform scale, and a platform that ingests agent traces at volume pays for validation cost; this is inferred from the listing, not a reported question. Medium-low confidence, numbers measured on the sample.

</details>

### System design (no AI, as at NVIDIA)

<a id="q13"></a>

#### Q13. Design a distributed eval platform that runs on a laptop, Docker, SLURM and Kubernetes, and resumes after failures

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Job orchestration, executors, checkpointing.

<details>
<summary>Show answer</summary>

Task: researchers run one benchmark on a laptop against a hosted endpoint; release engineering runs 40 benchmarks against a model served on a GPU cluster, some sites on SLURM and some on Kubernetes. The same config must work everywhere, results must be comparable, and a run that dies at 90% must not start from zero.

What they grade: one execution model behind several backends, capacity math, where state lives, how resume works, and what makes two runs comparable.

Numbers: a release run is 40 benchmarks and about 150,000 samples, each about 2,000 input and 300 output tokens, so about 345M tokens. One 8-GPU node serving at an aggregate 20,000 output tokens per second and about 100,000 prefill tokens per second needs about 38 minutes of decode for the 45M output tokens plus about 50 minutes of prefill, roughly 1.5 hours. Agentic benchmarks with about 10 model calls per sample multiply that by 10, to about 15 hours on one node or about 2 hours on 8 nodes. The endpoint is the bottleneck; the runners are cheap CPU work, and their concurrency must match the endpoint's capacity, not the cluster's size.

Approach: separate what to run from where to run it. A run is a resolved, immutable spec (benchmarks with versions, model endpoint, sampling settings, seed, SDK version) with a content hash as its id. A planner splits it into shards of about 500 samples. An executor interface (`submit(shard)`, `status`, `cancel`, `logs`) has four implementations: local processes, Docker, a SLURM job array, and Kubernetes Jobs with indexed completions. Each shard writes results as append-only JSONL to object storage under `run_id/benchmark/shard`, plus a done marker. The planner handles resume, so every backend gets it. It lists done markers and resubmits only missing shards, and inside a shard the runner skips samples already in its file (the Q5 runner). A shared token-bucket limiter per endpoint (for example in Redis) keeps 400 workers from sending 429s to a model that serves 64 concurrent requests well.

Alternatives: Ray gives scheduling and retries in one framework, but it adds a cluster to operate and does not run on SLURM sites without extra setup. Airflow or Argo give DAG retries and UIs, but per-sample checkpointing still has to be written. A single controller process that holds all state is simplest, but it is a single point of failure for a 12-hour run. The executor interface plus object storage plus planner-side resume costs about 4 small adapters and works with every scheduler.

Risks: comparability drifts when the endpoint changes underneath; record the served model's checksum and the sampling settings in the results and refuse to merge shards from different ones. Preempted SLURM jobs and evicted pods are normal, so make shards idempotent and keep them short (under 15 minutes). Secrets for hosted endpoints must reach Docker, SLURM and Kubernetes in different ways; keep them out of the run spec. Benchmarks that execute model-written code need a sandbox with no network on every backend.

Follow-ups: how do you show progress across 300 shards? (Count done markers and sample lines; no central database needed.) How do you rerun one benchmark with a new scorer without new model calls? (Store raw outputs separately from scores, and rescore offline.)

Why this is likely: NeMo Evaluator documents local, Docker and SLURM backends with per-benchmark checkpointing and distributed evaluation on SLURM, Kubernetes or Ray, a NeMo team candidate reported an ML pipeline design round, and the listing asks for distributed job orchestration on Kubernetes; high confidence in the theme.

</details>

<a id="q14"></a>

#### Q14. Design the public Python SDK and CLI of an open-source eval library

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: API design, typing, compatibility.

<details>
<summary>Show answer</summary>

Task: the library is used three ways: researchers call it from notebooks, CI jobs call the CLI, and partner teams write plugins. Design the public surface, the config model, typing, errors, and the rules for changing it without breaking users.

What they grade: a small core API, one config model shared by the SDK and CLI, typed and documented contracts, explicit stability rules, and the cost of each choice.

Approach: three layers. The config layer is Pydantic models (`RunConfig`, `BenchmarkConfig`, `EndpointConfig`) that load from YAML, environment and arguments in one documented order, and export JSON Schema for editor completion. The SDK is a few functions: `run(config) -> RunResult`, `compare(a, b) -> Comparison`, `load_results(path)`, plus the `@benchmark` and `@scorer` decorators from Q8. Everything else lives under `_internal`. The CLI is a thin wrapper that maps subcommands (`run`, `compare`, `gate`, `ls`) onto the same functions, so both always give the same result. Results are typed models with a stable JSON form, since CI scripts parse them. Errors are a small hierarchy (`ConfigError`, `EndpointError`, `PluginError`) with messages that say what to do; the CLI maps them to exit codes (2 for config, 3 for endpoint), which CI uses. The package ships `py.typed`, and the public API is checked in CI by a type checker and a snapshot of its signatures, so an accidental change fails the build.

Numbers: `import` time under 300 ms, because the CLI pays it on every call; plugins and heavy libraries load lazily. Deprecations warn for at least two minor releases. The config schema has a version number and an upgrader, as in Q11.

Alternatives: a class-based API (`Evaluator(...).run()`) groups state, but it tends to grow 40 methods; plain functions plus config objects keep the surface small. For the CLI, Typer derives the CLI from type hints with less code, Click gives finer control; either works if the CLI is thin. Keeping SDK and CLI in separate packages lets the CLI move faster, but doubles release work for a small team.

Risks: every public name is a promise, so start small; it is easy to add later and hard to remove. Notebooks often already run an event loop, so a sync `run` that calls `asyncio.run` breaks there; offer `arun` and detect the case with a clear error. Optional extras (`pip install lib[k8s]`) keep the base install light but need tests per extra.

Follow-ups: how do you know which functions users rely on? (Search public code that imports the package, and read issue reports, before deprecating.) How would you add a second backend for results storage without changing `run`?

Why this is likely: the listing asks for Python API design, typing, SDKs, plugins and CLIs, and NeMo Evaluator's documentation shows a CLI (`nel`) with `compare` and `gate` commands next to a Python API; high confidence in the theme.

</details>

<a id="q15"></a>

#### Q15. Design the test strategy for an eval platform, from unit tests to Kubernetes

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Test pyramid, fakes, CI cost.

<details>
<summary>Show answer</summary>

Task: the platform from Q13 has a planner, four executors, a runner that calls model endpoints, scorers, plugins and a CLI. Real model calls are slow, cost money and are not deterministic. Design the tests and the CI pipeline.

What they grade: the right level for each risk, fakes that keep tests fast and deterministic, real infrastructure where it matters, and a CI budget.

Approach: decide what each layer must prove. Unit tests (thousands, under a minute in total) cover the planner, the retry and resume logic, the statistics in Q10, the schema upgraders in Q11 and the scorers, with injected clocks and transports, as in Q1 and Q5. Contract tests run every registered benchmark and scorer against golden inputs and outputs, so a plugin change that alters scores fails before release. Integration tests start a fake OpenAI-compatible server (a few dozen lines that return scripted responses, delays, 429s and malformed JSON) and run real CLI commands against it, inside Docker, to cover packaging, config loading and exit codes. Executor tests run the same small run on each backend: Docker on every pull request, a `kind` Kubernetes cluster in CI, and SLURM in a container image with a single-node SLURM install, nightly. One end-to-end test per night runs a 200-sample suite against a real small model on a GPU runner and compares scores to a stored baseline with the Q10 gate, which catches changes that fakes cannot.

Numbers: a pull request run budget of about 10 minutes: unit and contract tests 2 minutes, Docker integration 4 minutes, `kind` 4 minutes in parallel. Nightly: SLURM and GPU end-to-end, about 1 hour. Flaky tests are quarantined within a day and fixed or deleted within a week.

Alternatives: mocking the HTTP client in unit tests is faster than a fake server, but it misses serialization and streaming bugs; use mocks in unit tests and the fake server in integration. Recording real responses (VCR style) gives realistic data but goes stale when the API changes; refresh recordings monthly. Running every test against a real model would cost GPU hours on each pull request and still be flaky.

Risks: fakes drift from real servers; run the same integration suite against a real endpoint nightly so differences surface. Tests that sleep make CI slow and flaky; inject time. A green test suite can hide a scoring change, which is why the golden contract tests and the nightly baseline exist.

Follow-ups: how do you test resume? (Kill the runner at a random point in an integration test and assert the final results equal an uninterrupted run.) How do you keep the SLURM image maintained?

Why this is likely: the listing asks for testing across unit, integration and end-to-end levels with Docker and Kubernetes, and an eval platform's value depends on trustworthy numbers; high confidence in the theme.

</details>

### Hiring manager

<a id="q16"></a>

#### Q16. How do you use AI coding tools in your daily work, and how do you check what they produce?

Round: Hiring manager. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Judgment with AI tools, ownership.

<details>
<summary>Show answer</summary>

Task: give a concrete picture with one specific story: where you use AI tools, where you do not, how you check the output, and a time the checking caught something.

What they grade: judgment about when to use the tools. The interviewer wants to hear that you own every line you merge, that you know the failure modes, and that you have a repeatable way to check. NVIDIA bans AI tools in the interview itself; this question is about daily work.

Approach: example structure. Where I use them: first drafts of tests, boilerplate such as CLI wiring and Pydantic models from a spec, explaining unfamiliar code, and reviewing my own diff for missed cases. Where I do not: statistics, concurrency and security-sensitive code, where a plausible mistake is expensive, unless I can test the result well.

How I check: I read every line before it reaches a pull request. I run it, and I write or extend a test that would fail if the code were wrong, ideally before I look at the generated code. I check every library call against the documentation for the version we pin, because invented methods and old APIs are the most common failure. For anything that touches results, I compare outputs with the old path on real data.

Story: I used an assistant to draft the paired comparison for our regression gate. The draft ran and looked right, but it used an independent two-proportion test, which ignores that both versions ran on the same tasks. On our 600-task suite that made the gate too conservative, so it called a real 3 point regression "not significant". I rewrote it as McNemar's exact test with a paired interval, added a test with known flip counts, and documented why in the code. The gate has since caught two real regressions that the old version would have passed.

Pitfalls:
- "I never use AI" or "I use it for everything"; both answers suggest no method.
- No example of a caught error, which makes the checking sound theoretical.

Follow-ups: how do you review a teammate's AI-written pull request? (The same bar as any pull request, plus a direct question: which parts did you check against the docs?) How would you set team rules for AI tools? What do you do when a generated test passes against generated code? (Write at least one test independently, or the two can share the same wrong idea.)

Why this is likely: NVIDIA's hiring page bans unapproved AI tools in interviews, which makes how candidates use AI at work a natural question for a senior role building tools for AI agents; the listing also asks for senior judgment and leadership. Medium confidence.

</details>

## Prep checklist

- [ ] Write a paginated REST client with retries, backoff, `Retry-After` and fail-fast 4xx handling from memory, with tests that need no network.
- [ ] Practice multi-part Python parsing tasks under time, with malformed lines, retries and out-of-order events.
- [ ] Solve Course Schedule II and Top K Frequent Elements without help, and explain the complexity and the distributed version.
- [ ] Rehearse asyncio: semaphores, `wait_for`, `gather` with `return_exceptions`, cancellation, and the `asyncio.TimeoutError` difference before Python 3.11.
- [ ] Review a pull request by hand for hallucinated library calls, secret leaks, crash paths and Pydantic v1 patterns in v2 code.
- [ ] Know Pydantic v2 well: `TypeAdapter`, discriminated unions, `model_validator`, `validate_json`, config options and what each costs.
- [ ] Be able to run McNemar's exact test by hand, build a paired confidence interval, and size an eval for a 2 point effect.
- [ ] Read an agent trace and name the cause of a regression, with the experiment that proves it.
- [ ] Prepare the design of an eval platform with capacity math, an executor interface for SLURM and Kubernetes, and resume.
- [ ] Know Kubernetes Jobs, indexed completions, SLURM job arrays and how preemption works on each.
- [ ] Prepare one story about how you use and check AI tools, with an error you caught.
- [ ] Plan to do every round without AI tools, and expect an in-person onsite.

## Sources

- NVIDIA, How We Hire (official process, AI tool rule, in-person onsite): https://www.nvidia.com/en-us/about-nvidia/careers/how-we-hire/
- NeMo Evaluator documentation: https://docs.nvidia.com/nemo/evaluator/latest/
- NeMo Evaluator repository: https://github.com/NVIDIA-NeMo/Evaluator
- NeMo Agent Toolkit, evaluating workflows: https://docs.nvidia.com/nemo/agent-toolkit/latest/improve-workflows/evaluate.html
- NVIDIA technical blog, AI agent evaluation: https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-evaluation
- Candidate report, NeMo team interview rounds (LinkedIn post): https://www.linkedin.com/posts/prashant-shihora-794a881b5_machinelearning-softwareengineering-mlops-activity-7299846808346726402-Od_f
- Candidate report, NVIDIA practical coding round (Reddit): https://www.reddit.com/r/AskProgrammers/comments/1ve7bzw/nvidia_swe_interview_2026_a_practical_coding
- Candidate report, NVIDIA senior software engineer loop (LeetCode Discuss post 7542688)
- Blind, NVIDIA interview format thread: https://www.teamblind.com/post/nvidia-interview-format-y4cpgjpa
- Blind, NVIDIA interview posts: https://www.teamblind.com/company/NVIDIA/posts/nvidia-interview
- Glassdoor, NVIDIA senior software engineer interviews: https://www.glassdoor.com/Interview/NVIDIA-Senior-Software-Engineer-Interview-Questions-EI_IE7633.0,6_KO7,31.htm
- spacecomplexity, NVIDIA senior software engineer interview: https://spacecomplexity.ai/blog/nvidia-senior-software-engineer-interview
- calibrd, NVIDIA interview guide: https://www.calibrd.com/interview-prep/nvidia-interview
- techscreen, NVIDIA technical interview process: https://techscreen.app/articles/nvidia-technical-interview-process-2026
- Medium, NVIDIA SDE interview experience: https://medium.com/@interviewshow/nvidia-sde-interview-experience-2026-full-software-engineer-interview-process-6b4f8c1c16f1
- Crowd-reported LeetCode company tags, NVIDIA (low confidence, practice links only): https://github.com/liquidslr/leetcode-company-wise-problems
