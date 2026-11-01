# Meta: Software Engineer, Machine Learning

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This listing asks for 12+ years of large-scale ML systems work and org-wide technical strategy, so read it as a very senior (roughly E7) loop. No level is stated, and the round mix at Meta depends on level, so treat this as an estimate.

1. Prescreen. A recruiter call, and for some roles a written prescreen form. Reported prompts include which AI tools you use every day and how you check the quality of AI output. Virtual.
2. Technical screen. 45 minutes in CoderPad, two LeetCode-style medium problems, code not executed, no AI. Senior candidates sometimes skip it, and some now get a 60 minute "AI-native" screen instead (one coding problem plus questions on how you use AI). Virtual.
3. AI-enabled coding. 60 minutes in a three-panel CoderPad (file tree, editor with test runner, AI chat). One multi-file project in phases: read the code and fix a failing test, build the core feature, then optimize against larger inputs. Meta's careers FAQ says the assistant is built in and candidates are expected to use it; outside AI tools are not allowed. Prep guides report this is the only coding round at E7 and above.
4. ML system design, one to three rounds of 45 minutes. Business goal, data, features, model, evaluation, serving. Prompts are now sub-domain focused (GenAI and LLM, ranking and recommendation, computer vision). Meta says design interviews also run in CoderPad with Mermaid diagrams and the same AI assistant. Design performance drives the level decision.
5. AI-enabled system design (newer variant). An AI-assisted design round with AI-flavored prompts such as a RAG pipeline or an agent with tools and guardrails. Reported for AI-native loops; it may replace or sit beside one ML design round. Lower confidence for this track.
6. Behavioral. 45 minutes with new AI-oriented signal areas (AI-driven impact, continuous AI learning, partnering, ambiguity, communication). Expect many follow-ups.
7. Cross-functional or project retrospective (senior loops). You walk through one large project in detail: its scope, the trade-offs, how you aligned several teams, and the measured impact.
8. Hiring committee and team match. Meta says the process takes a few months end to end.

Every report found ran the loop virtually. The questions below follow that mix, with six coding tasks (five in the AI-enabled format, one classic screen problem), three design rounds, four ML depth questions, one cross-functional question and one behavioral question.

## Questions and answers

### AI-enabled coding (AI assistant allowed, as in Meta's AI-enabled round)

#### Q1. Fix the leaking point-in-time join in this training data repo, then add a feature TTL

You get this repo in CoderPad. `test_join.py::test_no_future_leak` fails.

```text
training_data/
  features.py      # FeatureRow(user_id, ts, value) loaders
  labels.py        # LabelRow(user_id, label_ts, y)
  join.py          # point_in_time_join(labels, feature_rows)
  test_join.py     # 6 tests, 1 failing

# join.py (current)
import pandas as pd
def point_in_time_join(labels, feats):
    labels = labels.sort_values("label_ts")
    feats = feats.sort_values("ts")
    return pd.merge_asof(labels, feats, left_on="label_ts", right_on="ts",
                         by="user_id", direction="backward")

FAILED test_no_future_leak: expected value 0.3 for u1, got 0.9
```

Phase 2: drop any feature older than 7 days. Phase 3: make it fast for 50M labels.

The bug is leakage. `merge_asof` with `direction="backward"` allows exact matches by default, so a feature written at the same timestamp as the label (often the label event itself updating the feature) joins in. The fix is "strictly before", which in pandas is `allow_exact_matches=False`. The function also returns rows sorted by time instead of input order, which breaks any caller that zips results back to labels.

With the assistant, I would first ask it to explain `join.py` and list every caller of `point_in_time_join`, and check that against the file tree myself. Then ask for the minimal fix plus a test for the equal-timestamp case. Here is a wrong AI output I'd expect to catch. Asked for the TTL, it added `tolerance=pd.Timedelta("7D")` and also changed the direction to `"nearest"`, which would pull future features again. I reject that hunk and keep `"backward"`.

A dependency-free version of the fixed logic (what I would put in the reference test helper):

```python
from bisect import bisect_left
from datetime import datetime, timedelta

TTL = timedelta(days=7)

def build_index(feature_rows):
    idx = {}
    for user, ts, value in sorted(feature_rows, key=lambda r: (r[0], r[1])):
        times, values = idx.setdefault(user, ([], []))
        times.append(ts)
        values.append(value)
    return idx

def point_in_time_join(labels, feature_rows, default=None):
    idx = build_index(feature_rows)
    out = []
    for user, label_ts, y in labels:
        times, values = idx.get(user, ([], []))
        i = bisect_left(times, label_ts) - 1      # strictly before label_ts
        if i >= 0 and label_ts - times[i] <= TTL:
            out.append((user, label_ts, values[i], y))
        else:
            out.append((user, label_ts, default, y))
    return out

t = datetime(2025, 1, 10, 12)
feats = [("u1", t - timedelta(hours=2), 0.3), ("u1", t, 0.9),
         ("u2", t - timedelta(days=8), 0.5), ("u3", t - timedelta(days=7), 0.4)]
got = point_in_time_join([("u1", t, 1), ("u2", t, 0), ("u3", t, 1), ("u4", t, 0)], feats)
assert [g[2] for g in got] == [0.3, None, 0.4, None]
```

Tests I run: equal timestamp (no leak), TTL boundary at exactly 7 days (kept), just over (dropped), unknown user, and input order preserved. For phase 3 there are two options. A per-label bisect costs O((L + F) log F) and is easy to reason about. A single sorted merge per user (what `merge_asof` does) is O(L log L + F log F) but vectorized. For 50M labels I keep `merge_asof` with `allow_exact_matches=False` and `tolerance`, after a `reset_index` so I can restore input order, because the vectorized merge is about 10x to 50x faster than a Python loop at that size.

Why this is likely: Meta's careers FAQ describes the built-in assistant, and candidate reports and prep guides describe the fix-a-failing-test, build, then optimize phases; the leakage theme comes from the listing's focus on data integrity and feature consistency.

#### Q2. Review this AI-generated model evaluation module before it gates launches

The assistant wrote this to compute normalized entropy (NE) and calibration for a click model trained with negative downsampling. Review it like a PR.

```python
from sklearn.metrics import normalized_entropy   # used for a cross-check
import math

def ne(preds, labels):
    base = sum(preds) / len(preds)
    ll = -sum(y * math.log(p) + (1 - y) * math.log(1 - p)
              for p, y in zip(preds, labels)) / len(labels)
    return ll / -(base * math.log(base) + (1 - base) * math.log(1 - base))

def recalibrate(p, neg_keep_rate):
    return p * neg_keep_rate

def calibration_by_bucket(preds, labels, n=10):
    sums = [[0, 0, 0] for _ in range(n)]
    for p, y in zip(preds, labels):
        b = int(p * n)
        sums[b][0] += p; sums[b][1] += y; sums[b][2] += 1
    return [(s[0] / s[2], s[1] / s[2]) for s in sums]
```

Findings, in order of how much damage they do:

1. Hallucinated API. scikit-learn has no `normalized_entropy`, so the module fails on import. Delete the line, and cross-check against `log_loss` instead.
2. Wrong baseline. NE divides by the entropy of the base rate from the labels. Using the mean prediction makes a badly calibrated model look better, so a launch gate built on this would pass models it should block.
3. Wrong recalibration. If negatives were kept at rate w, the correct inverse is p / (p + (1 - p) / w). Multiplying by w is only roughly right for tiny p and is badly off at p = 0.5.
4. Missing edge cases. `log(0)` when a prediction is exactly 0 or 1, and a divide by zero when every label is the same.
5. Off-by-one. `int(p * n)` is 10 for p = 1.0, so the code raises IndexError, and empty buckets divide by zero.

Verdict: do not merge. Fixed version:

```python
import math

EPS = 1e-7

def recalibrate(p, neg_keep_rate):
    return p / (p + (1.0 - p) / neg_keep_rate)

def normalized_entropy(preds, labels):
    n = len(labels)
    if len(preds) != n or n == 0:
        raise ValueError("preds and labels must be non-empty and the same length")
    base = sum(labels) / n
    if base in (0.0, 1.0):
        raise ValueError("NE undefined when all labels are equal")
    ll = 0.0
    for p, y in zip(preds, labels):
        p = min(max(p, EPS), 1 - EPS)
        ll -= y * math.log(p) + (1 - y) * math.log(1 - p)
    h = -(base * math.log(base) + (1 - base) * math.log(1 - base))
    return (ll / n) / h

def calibration_by_bucket(preds, labels, n_buckets=10):
    sums = [[0.0, 0.0, 0] for _ in range(n_buckets)]
    for p, y in zip(preds, labels):
        b = min(int(p * n_buckets), n_buckets - 1)
        sums[b][0] += p; sums[b][1] += y; sums[b][2] += 1
    return [(s[0] / s[2], s[1] / s[2], s[2]) for s in sums if s[2]]

labels = [1, 0, 0, 0, 1, 0, 0, 0, 0, 0]
assert abs(normalized_entropy([0.2] * 10, labels) - 1.0) < 1e-9
assert math.isfinite(normalized_entropy([1.0, 0.0] + [0.0] * 8, labels))
assert calibration_by_bucket([1.0, 0.0], [1, 0])[-1][2] == 1
assert abs(recalibrate(0.2, 1.0) - 0.2) < 1e-12
```

Tests that would have caught each defect: a predictor that always outputs the base rate must score NE = 1.0; inputs with p = 0 and p = 1; a recalibration round trip (sample, train-rate, invert, compare to the true rate); and an all-negative slice. An alternative is to drop the hand-rolled code and wrap `sklearn.metrics.log_loss`. I would keep a small pure function here, because the gate runs inside a training job where an extra dependency costs more than 20 lines of tested code.

Why this is likely: candidate reports and Hello Interview say the AI-enabled round grades verification of assistant output, and the listing asks for automated evaluation pipelines.

#### Q3. Extend the maze solver to handle keys and doors, then make it fast on large grids

The repo has `grid.py` (parses a text grid), `solver.py` (plain BFS from `S` to `X` that treats letters as walls) and `test_solver.py`, where 4 of 9 tests fail. Spec: lowercase `a` to `f` are keys, uppercase `A` to `F` are doors that need the matching key, and keys persist once picked up. Return the fewest moves, or -1. Phase 3 runs a 200 x 200 grid with 6 keys under a time limit.

The BFS state has to include the keys you hold, because standing on a cell with key `a` lets you reach places you could not reach from that cell without it. So the state is (row, col, keys bitmask). That is at most 40,000 cells x 64 masks = 2.56M states, which BFS handles in well under a second in Python.

With the assistant, I would ask it to summarize `grid.py` and the test fixtures, then ask for the state change only. I check three things in the diff: the visited set includes the mask, a door test reads the key bit before moving, and picking up a key updates the mask for the new state, not the current one. One wrong output to watch for is an assistant draft that marks a cell visited by (row, col) only. That passes the small tests but returns -1 when the route has to go back through a cell after collecting a key. The fixture `["S.b", "B##", "X.."]` catches it.

```python
from collections import deque

def shortest_path(grid):
    rows, cols = len(grid), len(grid[0])
    start = next((r, c) for r in range(rows) for c in range(cols) if grid[r][c] == "S")
    seen = {(start[0], start[1], 0)}
    q = deque([(start[0], start[1], 0, 0)])
    while q:
        r, c, keys, d = q.popleft()
        if grid[r][c] == "X":
            return d
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if not (0 <= nr < rows and 0 <= nc < cols) or grid[nr][nc] == "#":
                continue
            ch = grid[nr][nc]
            if "A" <= ch <= "F" and not keys >> (ord(ch) - ord("A")) & 1:
                continue
            nkeys = keys | (1 << (ord(ch) - ord("a"))) if "a" <= ch <= "f" else keys
            if (nr, nc, nkeys) not in seen:
                seen.add((nr, nc, nkeys))
                q.append((nr, nc, nkeys, d + 1))
    return -1

assert shortest_path(["S.X"]) == 2
assert shortest_path(["SAX", ".##", "a.."]) == 6
assert shortest_path(["SAX"]) == -1
assert shortest_path(["Sa.A.X"]) == 5
assert shortest_path(["S.b", "B##", "X.."]) == 6
```

There are two alternatives for the large tier. A* with a Manhattan heuristic cuts the explored states on open grids, but the heuristic ignores doors, so the gain is small when keys force detours. Compressing the grid to its points of interest (start, keys, doors, exit), running one BFS from each to get pairwise distances, then running Dijkstra over (point, mask) wins when the grid is huge and the points are few: 8 BFS runs over 40,000 cells, then a search over 8 x 64 states. I would ship plain state BFS first because it meets the limit and is easy to verify, and mention compression as the next step if the grid grows.

Why this is likely: a maze with keys, doors and portals is one of the AI-enabled round prompts reported in 1point3acres's quarterly Meta trend summary, and candidate reports describe the same failing-tests-then-extend structure.

#### Q4. Read this training log, find why the job diverged, and add a guard that catches it earlier

A ranking model retrain diverged overnight. You get the log and the recent change list.

```text
step   loss    grad_norm  lr       amp_scale  max|emb|  note
11800  0.4121  1.03       2.0e-3   65536      3.9
11900  0.4118  1.10       2.0e-3   65536      4.0
12000  0.4115  1.07       2.0e-3   65536      4.0       data: new daily shard enters
12050  0.4140  2.85       2.0e-3   65536      7.8
12100  0.4392  19.6       2.0e-3   32768      31.0
12150  0.5810  48.2       2.0e-3   16384      122.5
12200  nan     inf        2.0e-3   8192       inf
skipped 412 steps (overflow), restarting from ckpt 12000
changes since last good run: +feature watch_time (new upstream table),
  embedding lr 2e-3 -> unchanged, warmup unchanged
```

The divergence starts at step 12050, well before the NaN at 12200. The grad norm jumps about 2.7x, then 18x, and the embedding max grows 30x. The AMP scale halving only reacts to overflow; it is a symptom. The timing lines up with the new data shard and the new `watch_time` feature. A raw count or duration feature fed in without normalization, or a unit change upstream (seconds to milliseconds is 1000x), produces huge activations in the first layer. That pushes large gradients into the embeddings that interact with it, and in fp16 those overflow.

To confirm, I would compare the feature's distribution on the new shard against the previous day (p50, p99, max), and rerun from checkpoint 12000 with that one feature zeroed. If the run is stable, the cause is confirmed. Fixes, in order of preference: normalize at the feature definition (log1p plus clipping at the training p99.9), which fixes every consumer at once; or add gradient clipping (for example a global norm of 1.0) to this job, which hides the symptom for this model only. I would do both. Clipping is the safety net, but on its own it lets a broken feature train quietly.

The guard I would add, with the assistant writing the first draft and me checking the median window and the ordering of the two checks:

```python
import math
from statistics import median

def first_anomaly(steps, window=20, factor=10.0):
    hist = []
    for s in steps:
        if not (math.isfinite(s["loss"]) and math.isfinite(s["grad_norm"])):
            return s["step"], "non-finite loss or grad"
        if len(hist) >= 5 and s["grad_norm"] > factor * median(hist[-window:]):
            return s["step"], "grad norm spike"
        hist.append(s["grad_norm"])
    return None, "clean"

log = [{"step": i, "loss": 0.62, "grad_norm": 1.0 + 0.05 * (i % 3)} for i in range(40)]
assert first_anomaly(log) == (None, "clean")
log[30]["grad_norm"], log[31]["loss"] = 48.0, float("nan")
assert first_anomaly(log) == (30, "grad norm spike")
```

The assistant's first draft compared each step against the mean of all history. One early spike then raised the threshold for the rest of the run, so I switched to a rolling median. On a hit, the trainer stops, saves the batch IDs and alerts the feature owner.

Why this is likely: the listing asks for resolving training stability and data integrity failures, and Meta's guides say debugging and verification are graded in the AI-enabled round; the log format is illustrative.

#### Q5. Find the train-serve skew between these two feature implementations and make it impossible to reintroduce

Offline NE says the new model wins, but online it loses. Feature parity logging shows mismatches on 3 features. Here are the two implementations.

```python
from datetime import timedelta

# offline/features.py (batch backfill, runs in UTC)
def offline_features(events, now):
    recent = [e for e in events if now - e["ts"] <= timedelta(hours=24)]
    return {
        "hour_of_day": now.hour,
        "clicks_24h": sum(e["type"] == "click" for e in recent),
        "avg_watch_s": (sum(e.get("watch_ms", 0) for e in recent) / len(recent) / 1000) if recent else -1.0,
    }

# serving/features.py (online, written by another team)
def online_features(events, now):
    local = now.astimezone()
    recent = [e for e in events if now - e["ts"] < timedelta(hours=24)]
    watch = [e["watch_ms"] for e in recent if "watch_ms" in e]
    return {
        "hour_of_day": local.hour,
        "clicks_24h": sum(e["type"] == "click" for e in recent),
        "avg_watch_s": (sum(watch) / len(watch) / 1000) if watch else 0.0,
    }
```

There are four bugs. `hour_of_day` uses the server's local time online and UTC offline, so the value depends on which data center serves the request. The window is `<=` offline and `<` online, so events exactly 24 hours old count differently. Offline counts events without `watch_ms` as 0 watch time, which drags the average down, while online skips them. And the empty default is -1.0 offline but 0.0 online, a value the model learned to mean "a 0 second watch".

The durable fix is one definition imported by both paths, not two patched copies:

```python
from datetime import datetime, timedelta, timezone

WINDOW = timedelta(hours=24)

def compute_features(events, now):
    if now.tzinfo is None:
        raise ValueError("now must be timezone-aware")
    now = now.astimezone(timezone.utc)
    recent = [e for e in events if timedelta(0) <= now - e["ts"] < WINDOW]
    watch = [e["watch_ms"] for e in recent if "watch_ms" in e]
    return {
        "hour_of_day": now.hour,
        "clicks_24h": sum(e["type"] == "click" for e in recent),
        "avg_watch_s": sum(watch) / len(watch) / 1000 if watch else -1.0,
    }

now = datetime(2025, 3, 1, 23, 30, tzinfo=timezone.utc)
ev = [{"ts": now - timedelta(hours=24), "type": "click"},
      {"ts": now - timedelta(hours=1), "type": "view"},
      {"ts": now - timedelta(minutes=5), "type": "view", "watch_ms": 4000}]
assert compute_features(ev, now) == {"hour_of_day": 23, "clicks_24h": 0, "avg_watch_s": 4.0}
assert compute_features(ev, now.astimezone(timezone(timedelta(hours=-8)))) == compute_features(ev, now)
```

Alternatives: (a) log features at serving time and train on the logged values. That removes skew by construction but costs storage, and you cannot backfill new features. (b) A shared definition plus a nightly parity job that recomputes a 0.1% sample offline and alerts when the mismatch rate goes above 0.1% per feature. At platform scale I would use (a) for the top ranking models and (b) everywhere else. Logging every feature for every request is too expensive for long-tail models, and (b) catches drift in a day. With the assistant I would ask for the parity test and the boundary fixtures first. Its first version built `now` with `datetime.now()` (naive), which hides the time zone bug, so I made naive timestamps an error.

Why this is likely: the listing names feature consistency and serving correctness among the cross-system failures this role resolves, and a Meta E6 MLE candidate report (Reddit, early 2026) describes ML design follow-ups on offline metric ground truth.

### Technical screen (no AI, as in Meta's classic screen)

#### Q6. Dot product of two sparse vectors, with a follow-up when one vector is much larger

Design a `SparseVector` built from a dense list, with `dot(other)`. Most entries are zero. Follow-up: one vector has 100 nonzeros and the other has 10 million. Code is not executed in this screen, so walk through test cases by hand.

Storing a dict from index to value is the first idea, and it is fine for lookups. The interviewer then asks what happens if a hash map is too costly to build, or if both vectors are stored on disk sorted by index. That points to a sorted list of (index, value) pairs. With two sorted lists of similar size, a two-pointer merge costs O(a + b). When one side is tiny, binary searching the big side for each entry costs O(a log b): for a = 100 and b = 10M that is about 2,300 comparisons instead of 10M.

```python
from bisect import bisect_left

class SparseVector:
    def __init__(self, dense):
        self.pairs = [(i, v) for i, v in enumerate(dense) if v != 0]

    def dot(self, other):
        a, b = self.pairs, other.pairs
        if len(a) > len(b):
            a, b = b, a
        if len(b) > 8 * len(a):
            idx = [i for i, _ in b]
            total = 0
            for i, v in a:
                j = bisect_left(idx, i)
                if j < len(idx) and idx[j] == i:
                    total += v * b[j][1]
            return total
        i = j = total = 0
        while i < len(a) and j < len(b):
            if a[i][0] == b[j][0]:
                total += a[i][1] * b[j][1]; i += 1; j += 1
            elif a[i][0] < b[j][0]:
                i += 1
            else:
                j += 1
        return total

assert SparseVector([1, 0, 0, 2, 3]).dot(SparseVector([0, 3, 0, 4, 0])) == 8
assert SparseVector([0, 0]).dot(SparseVector([5, 5])) == 0
assert SparseVector([]).dot(SparseVector([])) == 0
```

Building `idx` in the binary search branch costs O(b) per call. In production I would store the index array once in the constructor so repeated dots against the large vector stay O(a log b). The cutoff of 8 is a rough point where a log b drops below a + b for typical sizes; the exact value matters less than having both paths. Hand-traced cases: overlapping indexes, no overlap, empty vectors, negative values, and the size-swap path. To trace the binary search branch by hand, take a = [(3, 2)] and b = [(0, 1), (3, 4), (4, 1), (5, 1), (6, 1), (7, 1), (8, 1), (9, 1), (10, 1)]. Since b has 9 pairs, more than 8 times 1, the code builds idx = [0, 3, 4, ..., 10]. Then bisect_left(idx, 3) returns 1, idx[1] equals 3, and the total is 2 times 4, which is 8.

Why this is likely: the classic no-AI screen is still reported for Meta in recent prep guides and the 1point3acres quarterly summary, and sparse vector dot product is listed among Meta MLE screen questions in the Prepfully MLE guide (vendor source, medium confidence).

### ML system design (assistant and Mermaid available in CoderPad)

#### Q7. Design Reels ranking end to end, and show where the compute goes

Design the system that picks which short videos a user sees next. Cover objectives, the candidate funnel, features, models, evaluation and serving, with capacity numbers.

Start with the objective. Watch time alone rewards clickbait and long videos. I would rank on a weighted combination of predicted events: P(watch past 50%), P(like), P(share), P(follow), P(skip within 2s) as a negative, plus integrity downranking. The weights are a product decision, tuned with online experiments against long-term retention as well as session watch time.

The funnel exists because compute forces it:

```python
users, req_per_user, peak = 500e6, 20, 3.0
qps = users * req_per_user / 86400 * peak
gpu_flops = 989e12 * 0.30
for name, items, flop in [("retrieval", 10_000, 0.002e9), ("early ranker", 10_000, 0.02e9),
                          ("final ranker", 500, 2e9), ("final ranker on all 10k", 10_000, 2e9)]:
    print(f"{name}: {qps * items * flop / gpu_flops:,.0f} GPUs")
```

That prints about 23, 234, 1,170 and 23,406 GPUs at a peak of about 347k QPS. Running the heavy multitask model on every retrieved candidate needs 23,406 GPUs, about 16x the roughly 1,430 GPUs of the whole cascade. So I would use several retrieval sources (two-tower embedding ANN, follow graph, trending, creator affinity) merged to about 10k candidates; a small early-stage ranker down to about 500; a large multitask final ranker with user history sequence features; then a rules and diversity layer (creator caps, no repeats, integrity filters).

Alternatives for the final model: separate models per event (simple, but features and compute are duplicated), versus one shared-bottom multitask model with per-task heads. A multi-gate mixture of experts beats a plain shared bottom when tasks conflict, such as like versus skip. I'd pick the MMoE because one forward pass serves all heads, which cuts serving cost about 4x against four separate models.

Evaluation: offline NE and calibration per head, and recall at k for retrieval. Online: A/B tests on time spent and retention at 7 and 28 days, with guardrails on integrity reports and creator diversity. Cold start: new videos get an exploration slot (a few percent of impressions) and content embeddings from the video encoder until engagement data arrives. In the AI-assisted CoderPad I would draw the funnel in Mermaid myself and use the assistant to check my arithmetic, while I make the architecture choices.

Why this is likely: ML design is the round that sets the level at Meta per prep guides and candidate reports, a senior MLE loop reported ML product design rounds on Meta surfaces, and Meta's engineering blog describes this multi-stage recommendation funnel.

#### Q8. Design the training platform for a recommendation foundation model with 20B embedding rows

The ads org wants one large model trained on all surfaces, refreshed daily, with 20B sparse ID rows at dimension 128 and a 50 GFLOP dense tower per example. 20B examples per day. Size the cluster, choose the parallelism, and say what limits MFU.

Memory comes first, because the embeddings dominate:

```python
rows, dim = 20e9, 128
for name, b in {"fp32 + Adam": 12, "fp16 rows + rowwise Adagrad": 2}.items():
    tb = rows * dim * b / 1e12
    print(f"{name}: {tb:.1f} TB, {tb * 1e12 / 60e9:.0f} GPUs at 60 GB usable")
need = 20e9 * 50e9 * 3 / 86400
for mfu in (0.2, 0.4):
    print(f"MFU {mfu:.0%}: {need / (989e12 * mfu):.0f} GPUs")
```

That gives 30.7 TB (512 GPUs) versus 5.1 TB (85 GPUs) for the tables, and 176 versus 88 GPUs for the dense compute. So the choice of optimizer and precision for the tables is worth about 6x in hardware, and the gap between 20% and 40% MFU doubles the dense cost.

Parallelism. Embedding tables use model parallelism, with a planner choosing per table: table-wise for small tables, row-wise for huge ones, column-wise for very wide ones. Lookups use all-to-all. Dense layers use data parallelism, and FSDP once the dense tower no longer fits. The alternatives are keeping tables in host memory with a GPU cache (cheaper, but the cache miss rate sets throughput) or hashing IDs into fewer rows (cheaper, but collisions cost quality for tail IDs). I would shard across HBM for the hot tables and put the long tail in a host-memory cache tier, because ID frequency is heavily skewed.

What limits MFU here: the all-to-all embedding exchange (overlap it with the dense compute of the previous batch), the data loader (preprocessing on separate CPU hosts so GPUs never wait), and stragglers at thousands of GPUs. I'd measure step time broken down into compute, exposed communication and input wait before tuning anything. Reliability: checkpoint every 30 to 60 minutes with async sharded writes, and expect daily hardware failures at this scale, so restart must be automatic and take minutes.

Why this is likely: the listing names distributed training, training efficiency and hardware utilization, and Meta's engineering blog describes training its ads recommendation foundation model at LLM scale; numbers here are illustrative.

#### Q9. Design an LLM assistant that lets advertisers create and edit campaigns by chat (AI-enabled system design)

The assistant reads the advertiser's account, proposes campaigns, and can call tools: `get_performance`, `create_campaign`, `update_budget`, `generate_ad_text`. Design it, including guardrails and how you'd know it works.

Architecture: a planner LLM with a fixed tool schema; a retrieval layer over the advertiser's own account data and the ads policy docs; a tool executor that enforces permissions; and a confirmation step for every write. Reads run automatically. Writes (create, budget change) produce a diff that the advertiser approves. That one rule removes most of the risk of money-moving mistakes.

Alternatives for the core loop: (a) a single agent with all tools, simple but harder to constrain; (b) a router that classifies intent and hands off to small task-specific flows with fewer tools each. I would pick (b) for writes and (a) for open-ended performance questions. Fewer tools per step raises tool-choice accuracy, and a fixed flow is easier to audit.

Guardrails. Prompt injection arrives through ad text or landing-page content that the assistant reads. The executor wraps tool outputs as quoted data with a clear boundary, and a write tool only accepts arguments that come from the user's turn or the confirmed plan. Policy: generated ad text goes through the same ads integrity classifiers as human-written ads before it is shown. Limits: a budget change above a set multiple of the current daily budget always needs explicit confirmation.

Latency and cost: a planner call of about 1.5k input and 200 output tokens per turn, about 3 turns per task, plus retrieval. That keeps a typical task under 5 seconds with a mid-size model; the largest model is reserved for the plan step.

Evaluation: an offline set of a few hundred real task transcripts with expected tool calls, scored on task success, tool-argument exactness and policy violations, plus an LLM judge for the quality of explanations, calibrated against human labels until agreement is above about 85%. Online: task completion rate, the share of proposed changes the user edits or rejects, and the performance of campaigns created with the assistant versus without.

Why this is likely: candidate and vendor reports describe a newer AI-enabled system design round with RAG and agent-with-guardrails prompts, and Meta's FAQ says the AI assistant is available in design interviews (vendor-sourced for this track, lower confidence).

### ML depth

#### Q10. Design the automated evaluation pipeline that decides whether a retrained ranking model can ship

Thirty teams retrain models daily. Today each team checks launches by hand in notebooks. Design a shared evaluation and launch gate.

Stages, cheapest first, so a bad model fails fast:

1. Data checks on the eval set: label rate, feature null rates and distribution drift against the last 7 days. A broken eval set must fail the whole run.
2. Offline quality: NE and calibration per head, overall and on fixed slices (new users, new content, each country tier, each device). Compare against the production model on the same day's held-out data, with confidence intervals from bootstrap over users, not over impressions, because impressions from one user are correlated.
3. Behavior checks: score distribution shift, the share of top-k items that changed, and a replay of a small fixed set of sensitive cases (integrity, minors' content) that must score at least as well as production.
4. Online: an automatic small-percentage A/B with pre-registered success and guardrail metrics. The ramp only continues if guardrails hold.

The gate rule: ship if NE improves by at least a set threshold (for example 0.05% relative, roughly the size that tends to show up online for a mature model), no slice regresses by more than its own threshold, and calibration on every head stays within 0.95 to 1.05.

Alternatives: (a) one global threshold, which is simple but lets a model trade a large regression on a small slice for an average gain; (b) per-slice thresholds learned from history (how much offline movement predicted online movement). I would start with (a) plus hard slice floors and move to (b) once there are a few hundred launches to fit on.

Where AI tools help: the pipeline writes a launch summary, and an LLM drafts the explanation of which slices moved and why. The threshold rule makes the ship decision, and the draft only explains it. I check it by having the summary cite the metric table rows it is based on, and the review fails if a cited number does not match. Measuring the gate: track how often offline pass and online fail disagree. If more than about 20% of offline passes lose online, the offline metrics or slices are wrong and need fixing first.

Why this is likely: the listing asks for AI-native workflows that automate evaluation pipelines, and a Meta E6 MLE candidate report (Reddit, early 2026) describes ML design deep dives on offline metric ground truth.

#### Q11. Your click model is trained on downsampled negatives and delayed conversions. Estimate the calibration error and fix it

Negatives are kept at 5%, and the true CTR is 1%. Conversions can arrive up to 7 days after the click, but training uses labels joined 1 hour after the impression, and about 30% of conversions arrive later than that.

```python
def recalibrate(p, w):
    return p / (p + (1 - p) / w)

true_ctr, w = 0.01, 0.05
p_train = true_ctr / (true_ctr + (1 - true_ctr) * w)
print(f"{p_train:.3f} -> {recalibrate(p_train, w):.4f}")
assert abs(recalibrate(p_train, w) - true_ctr) < 1e-12
```

On the sampled data the model learns a rate of about 16.8%, not 1%. That is a 17x overestimate, which would wreck an auction that multiplies the prediction by the bid. The inverse transform p / (p + (1 - p) / w) recovers 1%. It must run in serving before the auction, and the eval module must apply the same transform, or offline calibration will look broken while online is fine (or the other way round).

Delayed labels are the harder one. With a 1 hour cutoff, 30% of positives are labeled negative at training time, so the conversion model underpredicts by about 30% (calibration of 0.70). Options:

1. Wait for the full window before training. Correct, but the model is 7 days stale, which costs more than the bias in fast-changing ads.
2. Train on fast labels and correct with a delay model: estimate P(conversion arrives after the cutoff | features) and reweight, or use a fake-negative correction that re-adds the positive when it arrives.
3. Train on fast labels, then fit a per-segment calibration layer daily on matured data (7+ days old).

I would combine 2 for the model and 3 as a safety layer, because the calibration layer is cheap, easy to audit, and catches drift in the delay distribution itself. To check: calibration by bucket on matured data, split by advertiser vertical, since delay patterns differ a lot (app installs convert fast, high-value purchases slowly).

Why this is likely: the listing asks for resolving failures in model quality and data integrity, and a Meta E6 MLE candidate report (Reddit, early 2026) describes an ML design deep dive on offline metric ground truth; the recalibration formula is the standard one from Meta's 2014 ads CTR paper (background knowledge, not an interview report).

#### Q12. An upstream logging change silently broke a feature used by 300 models. Design the safeguard so it is caught within an hour

Last quarter a client release changed how dwell time is logged. The feature's mean dropped 40% and 300 models trained on it for 9 days before anyone noticed.

The root problem is that no one owns the contract between the logger and the feature consumers. I would build three layers.

Layer 1, contracts at the source. Every logged field used as a feature gets a schema with type, unit and expected range, and client releases run a contract test in CI against a fixed event replay. This catches unit changes before they ship, at near-zero cost.

Layer 2, distribution monitoring on the feature store. For each feature, hourly statistics: null rate, mean, p50, p99, and the share of zeros. Compare against the same hour last week so daily cycles cancel out, and alert when the population stability index rises above about 0.2 or the null rate moves by more than a few points. With 50k features and hourly checks, that is 1.2M comparisons a day, cheap enough as a streaming job. The alternative of retraining canaries (train a small model hourly and watch its NE) catches problems that matter to models but is slower and costs GPU time, so I'd keep it only for the top 20 features by usage.

Layer 3, blast-radius control. Lineage from feature to every model that consumes it, so an alert lists the affected models. Training jobs refuse to start when an input feature is in alert. Serving falls back to the last good feature version for an hour, until the owner acknowledges.

Alert fatigue is the failure mode. With 50k features, a 1% false alarm rate is 500 alerts a day, and people will mute them. So alerts page only for features used by more than some number of models, or by any top-revenue model. Everything else goes to a daily digest. I would validate thresholds by replaying the last year of known incidents. The goal is to catch all of them within an hour with fewer than 10 pages a week.

Why this is likely: the listing asks for safeguards for reliability and integrity risks and for resolving data integrity failures across systems; no candidate report found names this prompt, so this is inferred from the listing (lower confidence).

#### Q13. You have a 2x larger ranking model that wins offline. Serve it without doubling latency or cost

Current final ranker: 18 ms p50, 1,000 GPUs. The new model has twice the FLOPs. Product will not accept more than about 20 ms or more than about 1,100 GPUs.

```python
base_ms, base_gpus = 18.0, 1000
for name, f in {"2x, nothing": 2.0, "2x + FP8": 2.0 / 1.6,
                "2x + cached user tower": 2.0 * (1 - 0.6 * 0.8), "distilled student": 1.0}.items():
    print(f"{name}: {base_ms * f:.1f} ms, {base_gpus * f:,.0f} GPUs")
```

That prints 36 ms and 2,000 GPUs; 22.5 ms and 1,250; 18.7 ms and 1,040; and 18 ms and 1,000. The options:

1. Lower precision (FP8 or INT8). About 1.6x faster on the matrix-heavy parts, but on its own it misses both budgets. It needs a calibration check, because quantization can shift the scores.
2. Cache the user tower. The user side of the model does not depend on the candidate, so I cache its output per user across requests for a few minutes. If 60% of FLOPs are user-side and 80% of requests find a fresh cached user embedding, the cost lands at 18.7 ms and 1,040 GPUs. This changes the model's structure (user and item interactions must happen after the cached part), so it needs a retrain and loses a little accuracy.
3. Distillation. Train a 1x student on the 2x teacher's outputs. It costs nothing extra in serving, but students usually keep only part of the teacher's gain.
4. Cut the candidate count from 500 to 300 by improving the early ranker. That saves 40% of final-ranker compute, but it risks recall.

My choice: caching plus FP8, because together they fit both budgets with room to spare and keep most of the teacher's gain. I'd run distillation in parallel as a fallback. Verification: an A/B of the cached FP8 model against the uncached FP32 2x model on a small slice, so the quality loss from the serving tricks is measured by itself, separately from the model gain.

Why this is likely: the listing names serving latency, inference infrastructure and hardware utilization as the problems this role drives, so this prompt is inferred from the listing (lower confidence); numbers are illustrative.

### Cross-functional and behavioral

#### Q14. Four orgs each run their own ranking stack. Make the case for, or against, one shared framework

You are asked to write the strategy. Ads, Feed, Reels and Marketplace each have their own training and serving stacks. The proposal is a shared modeling and training framework over the next two years. Decide, and say how you would get alignment.

I would start by measuring what duplication costs today: how many engineers maintain parallel versions of the same thing (data loaders, embedding sharding, eval tooling), how long a modeling idea takes to move from one org to another, and how many incidents came from divergent implementations. Suppose 4 stacks each spend about 15 engineers on infrastructure that is mostly the same, and moving a proven idea across orgs takes about 2 quarters.

Options:

1. Full unification: one framework, everyone migrates. This brings the largest long-term savings, but migration of every production model is risky, and the org with the strictest latency needs (often Ads) will block it.
2. Shared core, owned surfaces: unify the layers that are truly the same (data loading, embedding sharding, distributed training, evaluation and launch gating), and leave model architectures and serving tuned per org.
3. Standards only: shared interfaces and an eval spec, separate code.

I would recommend 2. It captures most of the savings, because infrastructure duplication is where most of the cost sits, while each org keeps control over the part that sets its business metrics. Option 3 tends to drift back to four stacks within a year.

Getting alignment: name one pilot in the org with the most pain and the least latency risk, and agree on success criteria up front (training throughput at least equal, migration under a quarter for a typical model, no regression in NE). Each org's tech lead co-owns a part of the core so it is not "the platform team's project". Write the decision doc with the alternatives and the reasons the other two lost, so the debate does not restart every quarter. Review progress against the criteria, and stop if the pilot misses them.

Why this is likely: senior loops include a cross-functional or project retrospective round per candidate reports and prep guides, and the listing asks for multi-year ML strategy across research, infrastructure and product teams.

#### Q15. Tell me about a time AI tools changed how fast your team shipped ML work, and how you made sure the output was correct

A strong answer picks one concrete workflow, gives before-and-after numbers, and spends most of its time on verification.

Example structure. Situation: model launch reviews took about 3 days each, because an engineer hand-assembled metric tables and wrote the analysis. Task: cut that time without lowering the bar, since a bad launch costs far more than a slow one. Action: I built a pipeline where an LLM drafts the launch report from the eval tables, and an agent pulls slice metrics on request. I set three rules. Every number in the draft must cite a table cell, and a checker fails the report if any cited value does not match. The ship decision stays with the same threshold rule as before. And for the first month, a human writes the report in parallel and we compare.

In week two the draft said a slice "improved 2%", but the number was for a different slice with a similar name. The citation checker caught it, and I changed the prompt to include the slice ID next to the name. We also found the LLM described every regression as "within noise". So I added the bootstrap interval to the input and required the draft to quote it.

Result: review time went from about 3 days to under 1, with the same number of rollbacks over the next two quarters. Two other teams adopted it. Next time I would build the citation checker first, before the drafting step.

Interviewers in this round tend to ask follow-ups: how you choose what not to automate, how you keep your own skills sharp, and what you do when the AI is confidently wrong. Have a real answer for each, and describe only AI-written code you reviewed yourself.

Why this is likely: coach reports say Meta's behavioral signal areas now include AI-driven impact and continuous AI learning, Meta's prescreen asks how candidates assess AI output, and the listing lists AI-native workflows with measurable impact.

## Prep checklist

- [ ] Practice a 60 minute multi-file task with an AI assistant: read the repo, fix a failing test, extend, optimize, and say out loud what you check in each diff.
- [ ] Practice reviewing AI-generated ML code for hallucinated APIs, off-by-one errors, leakage and wrong metric baselines.
- [ ] Do a few timed two-problem LeetCode medium sets without running code, in case the classic screen still appears.
- [ ] Draw two full ranking designs in Mermaid (feed or Reels, and ads) with QPS, funnel sizes and GPU counts.
- [ ] Prepare one GenAI or agent design with tools, guardrails, prompt-injection handling and an eval plan.
- [ ] Be able to size embedding tables, choose sharding, and break down MFU losses for a recommendation model.
- [ ] Rehearse NE, calibration, negative-downsampling correction and delayed-label handling with numbers.
- [ ] Prepare an evaluation and launch-gate design with slices, confidence intervals and online guardrails.
- [ ] Write one strategy story with alternatives, numbers and how you got several teams to agree.
- [ ] Prepare one AI-impact story with before-and-after numbers and a specific AI mistake you caught.

## Sources

- Meta careers, hiring process and FAQ on the AI assistant in interviews: https://www.metacareers.com/hiring-process
- Meta post announcing the AI-enabled coding interview: https://www.linkedin.com/posts/meta_metacareers-interviewing-softwareengineering-activity-7386046074240737280-wnwA
- Meta engineering, training the ads recommendation foundation model at LLM scale: https://engineering.fb.com/2026/08/03/ml-applications/training-gem-at-llm-scale-meta-ads-recommendation-foundation-model
- Meta engineering, scaling Instagram Explore recommendations: https://engineering.fb.com/2023/08/09/ml-applications/scaling-instagram-explore-recommendations-system
- Candidate report, Meta E6 MLE loop: https://www.reddit.com/r/leetcode/comments/1qxwwh5/sharing_my_meta_e6_mle_interview_experience
- Candidate report, Meta ML research scientist loop: https://www.reddit.com/r/leetcode/comments/1r37w7q/meta_ml_research_scientist_interview_experience
- Candidate report, Meta E7 MLE loop and offer: https://www.teamblind.com/post/meta-l7e7-interview-experience-offer-wqnypf12
- 1point3acres, Meta interview trend summary: https://www.1point3acres.com/interview/guides/meta-interview-trend-2026q2
- Behavioral signal areas and AI: https://thebehavioral.substack.com/p/how-to-talk-about-ai-in-your-behavioral
- Prepfully, Meta MLE guide: https://prepfully.com/interview-guides/meta-machine-learning-engineer
- Prepfully, Meta software engineer guide: https://prepfully.com/interview-guides/meta-software-engineer
- Hello Interview, Meta AI-enabled coding: https://www.hellointerview.com/blog/meta-ai-enabled-coding
- applr.ai, Meta AI-enabled coding interview: https://applr.ai/en/blog/2026-10-27-meta-ai-enabled-coding-interview
- aonecode, Meta AI-native engineer loop (vendor, low confidence): https://aonecode.com/iq/docs/meta/ai-native-engineer/how-it-works
- IGotAnOffer, Meta project retrospective (vendor, low confidence): https://igotanoffer.com/en/advice/meta-project-retrospective
