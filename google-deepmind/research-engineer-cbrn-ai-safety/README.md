# Google DeepMind: Research Engineer, Chemical, Biological and Nuclear AI Safety

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Google DeepMind publishes four stages: initial interviews, two or three skills interviews, final interviews with team leads and leadership, then a decision. Candidate reports for Research Engineer roles fill in the skills stage with coding, ML fundamentals and team-specific ML design. Most interviews are virtual. The exact mix is set per role, so confirm it with the recruiter.

1. Recruiter call (about 30 minutes, video). Background, motivation, and whether you lean toward the research side or the applied side of the Research Engineer role. AI: not allowed.
2. Hiring manager or team lead conversation (30 to 45 minutes). What the CBRNE team does and whether you fit its working conditions: limited publishing, strict data handling, tight release timelines, an on-call rotation, and reading technical material about mass-casualty harm. AI: not allowed.
3. Coding (one or two rounds, 45 to 60 minutes each, shared editor). Medium-level algorithm problems from Google's question bank, with follow-ups on tradeoffs. Candidates report that the code is expected to run by the end. AI: not allowed.
4. ML coding (some loops, about 60 minutes). Implement part of an ML or evaluation pipeline by hand, such as a metric, an estimator, a sampling routine or a small training loop. AI: not allowed.
5. ML fundamentals (45 to 60 minutes). A conversational mix of quick checks and deeper "how would you train or optimize this" questions, often with a derivation. AI: not allowed.
6. ML system design or evaluation infrastructure design (about 60 minutes). Run by the hiring team, so expect a team-specific prompt: an evaluation harness, a safeguard, a red-teaming pipeline, or the reliability of a safeguard in production. AI: not allowed.
7. Final interviews (two or three conversations of 30 to 45 minutes). A technical deep dive on your past work and ML experiments with team leads, plus a behavioral round with a people partner. AI: not allowed.
8. Hiring committee review of the written feedback, then the offer.

Every question below is labelled "no AI, as at Google DeepMind", because the company's candidate guide asks for no AI tools in live interviews or interview tasks unless you are told otherwise. Classic algorithm questions are limited to two, matching the coding round in recent Research Engineer reports. The domain questions stay on evaluations, classifiers, red-team infrastructure and reliability. They use placeholder labels such as "hazard-class item" and "benign science item" and never touch hazardous technical content.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | A safeguard classifier emits (timestamp, flagged) events in time order. Implement a... | Classic algorithm | Coding | Not allowed | Medium |
| [Q2](#q2) | Eval jobs have durations and dependencies. Return a valid run order and the earliest... | Classic algorithm | Coding | Not allowed | Medium |
| [Q3](#q3) | Implement the unbiased pass@k estimator and a confidence interval for attack success... | Concept plus code | ML coding | Not allowed | Medium |
| [Q4](#q4) | An LLM autograder will replace expert grading on a capability eval. Write the code that... | Concept plus code | ML coding | Not allowed | Medium |
| [Q5](#q5) | This eval harness reports that a new checkpoint scores far lower on hazard-class items... | Debug | ML coding or ML design | Not allowed | Medium |
| [Q6](#q6) | After a prompt-template launch, the output classifier's miss rate on red-team... | Trace debug | ML system design | Not allowed | Hard |
| [Q7](#q7) | A teammate's coding assistant wrote this automated red-teaming loop. Review it like a... | Review | Coding or ML coding | Not allowed | Medium |
| [Q8](#q8) | Design an evaluation harness that measures chemical, biological, radiological and... | Design | ML system design | Not allowed | Hard |
| [Q9](#q9) | Design a layered safeguard that blocks hazardous assistance while keeping refusals on... | Design | ML system design | Not allowed | Hard |
| [Q10](#q10) | Design an automated red-teaming system that keeps finding new attacks against your... | Design | ML system design | Not allowed | Hard |
| [Q11](#q11) | The safeguard classifier service starts timing out at peak load. You are on call.... | Design | ML system design or final interviews | Not allowed | Hard |
| [Q12](#q12) | Experts say a model should complete fewer than 5% of a hard task set. How many... | Concept plus code | ML fundamentals | Not allowed | Medium |
| [Q13](#q13) | Your classifier flags hazard-class requests. Missing one costs far more than a false... | Concept plus code | ML fundamentals | Not allowed | Medium |
| [Q14](#q14) | You have 800 expert-labelled examples and a large pool of unlabelled traffic.... | Concept plus code | ML fundamentals | Not allowed | Hard |
| [Q15](#q15) | Design the data pipeline for sensitive expert-written eval items: who can read them,... | Design | ML system design or final interviews | Not allowed | Medium |
| [Q16](#q16) | Tell me about a time you had to report a negative or uncertain result to leadership... | Behavioral | Final interviews | Not allowed | Medium |

### Coding: classic rounds (no AI, as at Google DeepMind)

<a id="q1"></a>

#### Q1. (Coding round, no AI) A safeguard classifier emits (timestamp, flagged) events in time order. Implement a monitor that reports, after each event, the largest number of flagged events seen in any W-second window so far, and whether the current window breaches an alert threshold.

Round: Coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Sliding window, streaming.
Related practice: [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) (Hard), [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) (Medium), [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) (Hard).

<details>
<summary>Show answer</summary>

Task: build `FlagMonitor(window, threshold)` with `add(ts, flagged)` returning `(peak, breach)`. Timestamps are integers in seconds and never decrease. A window ending at time t covers the half-open interval (t - window, t]. `peak` is the most flagged events in any window ending at an event seen so far. `breach` is true when the window ending now holds at least `threshold` flagged events.

Approach: only flagged events matter for the count, so keep a deque of their timestamps. On each event, pop from the left while the oldest timestamp is at or before `ts - window`, then append `ts` if the event is flagged. The deque length is the count for the current window. Each timestamp enters and leaves the deque once, so the cost is amortized O(1) per event and memory is bounded by the busiest window. The peak can only change when the window ends at a flagged event, but updating it on every call is simpler and still correct.

Alternatives: recounting the whole history per event is O(n) per call and fails at production volume. Fixed one-second buckets in a ring buffer give O(1) worst case and fixed memory, which is better when a single second can hold a million events, at the cost of coarse boundaries. For an interview, the deque is the clean answer, and the bucket version is a good follow-up.

```python
from collections import deque
import random

class FlagMonitor:
    def __init__(self, window, threshold):
        self.window, self.threshold = window, threshold
        self.flags = deque()
        self.peak = 0

    def add(self, ts, flagged):
        while self.flags and self.flags[0] <= ts - self.window:
            self.flags.popleft()
        if flagged:
            self.flags.append(ts)
        self.peak = max(self.peak, len(self.flags))
        return self.peak, len(self.flags) >= self.threshold

def brute(events, window, threshold):
    out, peak = [], 0
    for i, (t, _) in enumerate(events):
        cur = sum(1 for s, f in events[: i + 1] if f and t - window < s <= t)
        peak = max(peak, cur)
        out.append((peak, cur >= threshold))
    return out

for _ in range(500):
    w, k = random.randint(1, 6), random.randint(1, 4)
    t, events = 0, []
    for _ in range(random.randint(0, 40)):
        t += random.randint(0, 3)
        events.append((t, random.random() < 0.5))
    m = FlagMonitor(w, k)
    assert [m.add(*e) for e in events] == brute(events, w, k)
print("Q1 ok")
```

Verify: equal timestamps, an event exactly `window` seconds after a flag (it must drop out), a window of 1, threshold 1, and a long run with no flags. The randomized test compares against a brute-force recount.

Pitfalls: using a closed interval [t - window, t], which wrongly keeps a flag from exactly `window` seconds ago and overcounts at the boundary, and counting unflagged events in the deque.

Follow-ups: out-of-order events within a lateness bound (buffer and sort a small horizon), one monitor per safeguard version, and alerting on a rate ratio against the previous day instead of a fixed count.

Why this is likely: Research Engineer candidates report one or two unaided coding rounds at a medium level, with code that has to run, and this problem fits the posting's focus on safeguard reliability and incident response (medium confidence: Glassdoor reports plus prep guides).

</details>

<a id="q2"></a>

#### Q2. (Coding round, no AI) Eval jobs have durations and dependencies (for example, a grader calibration must finish before the graded run). Return a valid run order and the earliest finish time with unlimited workers, or report a dependency cycle.

Round: Coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Topological sort, critical path.
Related practice: [Course Schedule](https://leetcode.com/problems/course-schedule/) (Medium), [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) (Medium), [Parallel Courses III](https://leetcode.com/problems/parallel-courses-iii/) (Hard).

<details>
<summary>Show answer</summary>

Task: `plan(durations, deps)` takes a dict from job name to duration in minutes and a list of `(before, after)` pairs. It returns `(order, finish)`, where `order` lists every job after all of its prerequisites and `finish` is the earliest time the last job can end when any number of jobs can run at once. If the dependencies contain a cycle, it raises `ValueError` and names the jobs that are stuck.

Approach: Kahn's algorithm. Count incoming edges, start with every job that has none, and repeatedly pop a ready job, append it to the order, and release its children. While doing this, track `start[child] = max(start[child], start[job] + duration[job])`, which is the longest path into each job. With unlimited workers, the finish time is the longest path through the graph, the critical path. If the order ends up shorter than the job list, the remaining jobs sit on or behind a cycle. Cost is O(V + E).

Alternatives: depth-first search with three colors also gives a topological order and finds the exact cycle edges, which is nicer for an error message, but recursion depth can hit Python's limit on long chains. Kahn's version is iterative and computes the critical path in the same pass. With a fixed worker count, the problem becomes scheduling with precedence constraints, which is NP-hard in general, so the follow-up answer is a list-scheduling heuristic that runs the ready job with the longest remaining path first.

```python
from collections import defaultdict, deque
import itertools, random

def plan(durations, deps):
    children, indeg = defaultdict(list), {j: 0 for j in durations}
    for a, b in deps:
        children[a].append(b)
        indeg[b] += 1
    start = {j: 0 for j in durations}
    ready = deque(sorted(j for j in durations if indeg[j] == 0))
    order = []
    while ready:
        j = ready.popleft()
        order.append(j)
        for c in children[j]:
            start[c] = max(start[c], start[j] + durations[j])
            indeg[c] -= 1
            if indeg[c] == 0:
                ready.append(c)
    if len(order) < len(durations):
        raise ValueError(f"cycle among {sorted(set(durations) - set(order))}")
    return order, max((start[j] + durations[j] for j in durations), default=0)

order, finish = plan({"calibrate": 30, "graded_run": 120, "build_items": 45, "report": 10},
                     [("build_items", "graded_run"), ("calibrate", "graded_run"), ("graded_run", "report")])
assert finish == 175 and order.index("graded_run") > order.index("calibrate")
try:
    plan({"a": 1, "b": 1, "c": 1}, [("a", "b"), ("b", "a")])
    raise AssertionError("cycle missed")
except ValueError as e:
    assert "'a'" in str(e) and "'c'" not in str(e)

def longest_path(d, deps):
    best = {}
    def f(j):
        if j not in best:
            best[j] = d[j] + max((f(a) for a, b in deps if b == j), default=0)
        return best[j]
    return max((f(j) for j in d), default=0)

for _ in range(300):
    names = [f"j{i}" for i in range(random.randint(0, 8))]
    d = {n: random.randint(1, 9) for n in names}
    deps = [(a, b) for a, b in itertools.combinations(names, 2) if random.random() < 0.3]
    order, finish = plan(d, deps)
    pos = {j: i for i, j in enumerate(order)}
    assert all(pos[a] < pos[b] for a, b in deps) and finish == longest_path(d, deps)
print("Q2 ok")
```

Verify: an empty job set, a single job, a diamond where two paths meet, a cycle that leaves an unrelated job runnable (the error must name only the stuck jobs), and random acyclic graphs checked against a memoized longest path.

Pitfalls: summing all durations instead of taking the longest path, and reporting a cycle without naming the jobs, which costs the on-call engineer time.

Follow-ups: a fixed worker pool, retrying a failed job without rerunning its finished parents, and caching job outputs by a hash of code version plus inputs.

Why this is likely: Research Engineer coding rounds are reported to draw medium problems from Google's question bank, graph and ordering problems appear often in Google's recent company-tagged lists, and eval pipelines with dependent steps are the daily work in this role (medium confidence).

</details>

### ML coding: evaluation statistics and graders (no AI, as at Google DeepMind)

<a id="q3"></a>

#### Q3. (ML coding, no AI) Implement the unbiased pass@k estimator and a confidence interval for attack success rate, then use them to decide whether a safeguard change really lowered attack success from 6% to 4%.

Round: ML coding. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Eval statistics.

<details>
<summary>Show answer</summary>

Task: write `pass_at_k(n, c, k)` for n samples of which c succeeded, a Wilson interval for a success rate, and a two-proportion test. Then answer the product question: red-teamers ran 500 attacks before the change and 500 after, with 30 and 20 successes. Did the safeguard improve?

Approach: the naive pass@k, "did any of the first k samples succeed", has high variance and wastes the other n - k samples. The unbiased estimator averages over every k-subset of the n samples: 1 - C(n - c, k) / C(n, k). For a red team this is "probability that an attacker with k tries gets through at least once", which is the number leadership cares about. For a single rate, the Wilson interval behaves well near 0 and at small n, where the normal approximation can produce negative lower bounds.

Here 30/500 = 6.0% and 20/500 = 4.0%. A pooled two-proportion z-test gives z = 1.45 and p = 0.15, so the drop could be noise. The Wilson intervals, 4.2% to 8.4% and 2.6% to 6.1%, overlap. To detect a 6% to 4% drop with 80% power at a 5% two-sided level, you need about 1,863 attacks per arm. The honest report is "no detectable change at this sample size, here is the run size that would settle it".

```python
from math import comb, sqrt, erf, ceil
from itertools import combinations
import random

def pass_at_k(n, c, k):
    if n - c < k:
        return 1.0
    return 1.0 - comb(n - c, k) / comb(n, k)

def wilson(x, n, z=1.959964):
    if n == 0:
        return 0.0, 1.0
    p, d = x / n, 1 + z * z / n
    centre = (p + z * z / (2 * n)) / d
    half = z * sqrt(p * (1 - p) / n + z * z / (4 * n * n)) / d
    return max(0.0, centre - half), min(1.0, centre + half)

def two_prop(x1, n1, x2, n2):
    p = (x1 + x2) / (n1 + n2)
    z = (x1 / n1 - x2 / n2) / sqrt(p * (1 - p) * (1 / n1 + 1 / n2))
    return z, 2 * (1 - 0.5 * (1 + erf(abs(z) / sqrt(2))))

def n_per_arm(p1, p2, za=1.959964, zb=0.841621):
    pbar = (p1 + p2) / 2
    num = (za * sqrt(2 * pbar * (1 - pbar)) + zb * sqrt(p1 * (1 - p1) + p2 * (1 - p2))) ** 2
    return ceil(num / (p1 - p2) ** 2)

for _ in range(200):
    n = random.randint(1, 8); c = random.randint(0, n); k = random.randint(1, n)
    outcomes = [1] * c + [0] * (n - c)
    subsets = list(combinations(range(n), k))
    exact = sum(any(outcomes[i] for i in s) for s in subsets) / len(subsets)
    assert abs(pass_at_k(n, c, k) - exact) < 1e-12
z, p = two_prop(30, 500, 20, 500)
lo1, hi1 = wilson(30, 500); lo2, hi2 = wilson(20, 500)
assert round(z, 2) == 1.45 and round(p, 2) == 0.15 and hi2 > lo1
assert n_per_arm(0.06, 0.04) == 1863
print("Q3 ok", round(z, 2), round(p, 3), [round(v, 3) for v in (lo1, hi1, lo2, hi2)])
```

Verify: the estimator matches brute-force enumeration over all k-subsets, and it returns 1.0 when fewer than k failures exist.

Pitfalls: treating several attempts on the same seed prompt as independent. Attempts cluster by prompt, so bootstrap over prompts, not attempts, or the interval comes out too narrow. Comparing runs that used different judges or attacker temperatures also breaks the comparison.

Alternatives: for the interval, the normal-approximation (Wald) interval is simplest but can go below zero at small rates, Clopper-Pearson is exact but conservative (too wide), and Wilson keeps close to nominal coverage, so Wilson wins here. For the comparison, Fisher's exact test is the safe choice at small counts, but with 30 and 20 successes out of 500 it gives p = 0.19 against the z-test's 0.15, the same conclusion, and the z-test is easier to explain.

Follow-ups: one-sided tests when only improvement matters, multiple comparisons across many risk categories, and pass@k for agentic tasks where samples are expensive.

Why this is likely: candidate guides for this Research Engineer track describe ML coding rounds that include eval metrics such as pass@k, and the posting asks for statistics applied to ML evaluation and for communicating uncertainty (medium confidence).

</details>

<a id="q4"></a>

#### Q4. (ML coding, no AI) An LLM autograder will replace expert grading on a capability eval. Write the code that measures its agreement with expert labels and decide whether it is good enough to ship.

Round: ML coding. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Grader calibration.

<details>
<summary>Show answer</summary>

Task: you have 400 eval transcripts. Two domain experts labelled each one as `none`, `partial` or `uplift`, meaning how much the response would help with a hazard-class task. The autograder labelled the same 400. Decide whether the autograder can replace the experts on the next model's run.

Approach: raw accuracy hides class imbalance. If 85% of transcripts are `none`, a grader that always says `none` scores 85%. Cohen's kappa corrects for chance agreement: (observed - expected) / (1 - expected). The right bar is the experts' agreement with each other, because the grader cannot be held to a ceiling the humans do not reach. My ship rule has three parts. The lower bound of the bootstrap interval for grader-versus-expert kappa must be within 0.05 of the expert-versus-expert kappa. Recall on the `uplift` class must be at least 0.9, because a missed `uplift` is the expensive error. And there must be no subgroup, such as agentic transcripts or long transcripts, where agreement collapses.

```python
import random

def kappa(a, b):
    n = len(a)
    po = sum(x == y for x, y in zip(a, b)) / n
    pe = sum((a.count(l) / n) * (b.count(l) / n) for l in set(a) | set(b))
    return 1.0 if pe == 1 else (po - pe) / (1 - pe)

def recall(truth, pred, label):
    idx = [i for i, t in enumerate(truth) if t == label]
    return sum(pred[i] == label for i in idx) / len(idx) if idx else float("nan")

def bootstrap(a, b, stat, iters=2000, seed=0):
    rng, n = random.Random(seed), len(a)
    vals = []
    for _ in range(iters):
        ix = [rng.randrange(n) for _ in range(n)]
        vals.append(stat([a[i] for i in ix], [b[i] for i in ix]))
    vals.sort()
    return vals[int(0.025 * iters)], vals[int(0.975 * iters) - 1]

assert abs(kappa([1, 1, 0, 0], [1, 0, 0, 0]) - 0.5) < 1e-12
assert kappa(["none"] * 9 + ["uplift"], ["none"] * 10) == 0.0

rng = random.Random(1)
labels = ["none"] * 340 + ["partial"] * 40 + ["uplift"] * 20
def noisy(truth, p):
    return [t if rng.random() > p else rng.choice(["none", "partial", "uplift"]) for t in truth]
expert_b, grader = noisy(labels, 0.10), noisy(labels, 0.12)
k_ee, k_ge = kappa(labels, expert_b), kappa(labels, grader)
lo, hi = bootstrap(labels, grader, kappa)
ship = lo >= k_ee - 0.05 and recall(labels, grader, "uplift") >= 0.9
print("Q4 ok", round(k_ee, 3), round(k_ge, 3), (round(lo, 3), round(hi, 3)), "ship" if ship else "do not ship")
```

Verify: kappa is 0.5 on a hand-worked case and 0 for a constant grader on imbalanced data. On the synthetic run above, the experts agree at kappa 0.80, the grader reaches 0.74, and its bootstrap lower bound of 0.66 misses the bar, so the verdict is do not ship yet. Read 30 disagreements by hand before trusting any number, because they show whether the grader fails on style (long answers, hedged answers) or on substance.

Pitfalls: letting the grader see the reference answer in a way it can copy, grading only on transcripts from the old model and then running it on a new model whose outputs look different, and using the same 400 items to tune the grader prompt and to measure it.

Alternatives: raw accuracy and per-class F1 ignore chance agreement, so they flatter a grader on imbalanced labels. Cohen's kappa corrects for chance. Because `none`, `partial` and `uplift` are ordered, weighted kappa or Krippendorff's alpha (which also handles missing labels and more than two raters) can be better. They count a grader picking `partial` for `uplift` as a smaller error than picking `none` for `uplift`.

Follow-ups: weighted kappa for ordinal labels, re-validating the grader on every new model family, and sending low-confidence grades back to experts.

Why this is likely: the posting says the team turns expert judgment into datasets and automated graders, and the evaluation-infrastructure round in Research Engineer guides asks how you make eval results trustworthy (medium confidence).

</details>

### Coding: debugging, traces and review (no AI, as at Google DeepMind)

<a id="q5"></a>

#### Q5. (ML coding or ML design, no AI) This eval harness reports that a new checkpoint scores far lower on hazard-class items than the last one. The result looks too good to be true. Find the bugs.

Round: ML coding or ML design. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Eval harness bugs.

<details>
<summary>Show answer</summary>

Task: the old checkpoint scored 0.42 on the hazard-class set and the new one scores 0.03. Nobody changed the safeguards. Here is the harness.

```python
REFUSALS = ("i can't", "i cannot", "i won't")

def score_buggy(items, model, grader):
    total = 0.0
    for it in items:
        text = model(it["prompt"])["text"]
        if any(r in text.lower() for r in REFUSALS):
            continue  # refusal scores 0
        try:
            total += grader(it, text)
        except Exception:
            pass  # flaky grader, skip
    return total / len(items)
```

Approach: a big drop with no cause is a measurement problem until proven otherwise, so diff the harness behaviour before believing the model. Two bugs fail toward "safe". First, the refusal check is a substring match. A newer model often opens with a partial disclaimer ("I can't share everything, but here is an outline") and then helps anyway, and the harness scores that as a refusal. Second, every grader exception becomes a silent 0. If the new checkpoint writes longer answers that overflow the grader's context, every long answer quietly turns into a 0. The fix is to let the rubric grader judge refusals, record errors separately, and refuse to report a score when the error rate passes a small budget.

```python
def score_fixed(items, model, grader, max_error_rate=0.02):
    scores, errors = [], []
    for it in items:
        out = model(it["prompt"])
        try:
            scores.append(grader(it, out["text"]))
        except Exception as e:
            errors.append((it["id"], type(e).__name__))
    err = len(errors) / len(items)
    valid = err <= max_error_rate
    return {"score": sum(scores) / len(scores) if valid and scores else None,
            "error_rate": err, "errors": errors[:5], "valid": valid}

items = [{"id": i, "prompt": f"hazard-class item {i}"} for i in range(50)]
old = lambda p: {"text": "Here is an outline. DETAIL DETAIL", "finish_reason": "stop"}
new = lambda p: {"text": "I can't share everything, but here is an outline. " + "DETAIL " * 40, "finish_reason": "stop"}
def grader_v1(it, text):
    if len(text) > 250:
        raise ValueError("grader context exceeded")
    return 1.0 if "DETAIL" in text else 0.0
grader_v2 = lambda it, text: 1.0 if "DETAIL" in text else 0.0

assert score_buggy(items, old, grader_v1) == 1.0 and score_buggy(items, new, grader_v1) == 0.0
r = score_fixed(items, new, grader_v1)
assert r["valid"] is False and r["score"] is None and r["error_rate"] == 1.0
assert score_fixed(items, new, grader_v2)["score"] == 1.0
print("Q5 ok")
```

Verify: on the toy data the buggy harness reports a perfect safety gain, the fixed harness marks the run invalid with the old grader, and with a grader that fits the longer answers the new checkpoint is no safer at all.

Pitfalls: other "too good" causes to check in the same pass include a generation length cap that cuts answers before the substance, few-shot examples drawn from the test set, a cached result from the old checkpoint, and items silently dropped by a filter.

Follow-ups: add a canary item with a known score to every run, alert on error-rate changes between runs, and store raw outputs so a score can be regraded later.

Why this is likely: the posting says you will "find out why an evaluation result looked too good to be true", and the Research Engineer evaluation-infrastructure round asks about reproducibility and trust in results (high confidence on topic).

</details>

<a id="q6"></a>

#### Q6. (ML system design, no AI) After a prompt-template launch, the output classifier's miss rate on red-team transcripts tripled while its scores on the standard eval set did not move. Here are the logs. Find the root cause and fix it.

Round: ML system design. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Classifier input truncation.

<details>
<summary>Show answer</summary>

Task: explain the gap between the red-team results and the standard eval, then fix the classifier so it sees what the model actually said. The classifier reads a 512-token input.

```text
time      template  conv_turns  input_tokens  truncated  clf_score  red_team_label
09:01:12  v14       2           388           false      0.97       violating
09:14:40  v14       6           701           true       0.91       violating
09:22:05  v14       1           142           false      0.02       benign
11:02:51  v15       2           812           true       0.04       violating
11:03:30  v15       1           566           true       0.03       violating
11:40:03  v15       7           1204          true       0.05       violating
11:41:17  v15       1           561           true       0.01       benign
eval-set  v14       1           96 (median)   false      recall 0.93 (unchanged)
release note v15: adds a 420-token system preamble before the conversation
```

Approach: the misses sit only on v15 and all of them are truncated. The classifier input is "system preamble + conversation", and the tokenizer truncates from the right. Under v14 a short conversation fit in 512 tokens. Under v15 the 420-token preamble uses most of the budget, so the model's reply, the part being judged, is cut off. Scores near 0 on clearly violating transcripts fit that. The standard eval set never moved because its items are single short prompts built without the preamble, so it no longer represents production inputs.

Fix: build the classifier input from the user and model turns only, and score long conversations in overlapping chunks, taking the maximum score. Chunking costs ceil((L - overlap) / (512 - overlap)) classifier calls per conversation: 3 calls for the full 1,204-token input with a 64-token overlap, or 2 once the 420-token preamble is dropped and about 784 tokens remain, so check the latency budget.

```python
def chunks(tokens, size=512, overlap=64):
    step = size - overlap
    for i in range(0, max(len(tokens) - overlap, 1), step):
        yield tokens[i:i + size]

def classify(turns, score_fn, size=512, overlap=64):
    toks = [t for turn in turns if turn["role"] in ("user", "model") for t in turn["text"].split()]
    return max(score_fn(c) for c in chunks(toks, size, overlap))

score_fn = lambda toks: 1.0 if "FLAGGED_SPAN" in toks else 0.0
preamble = {"role": "system", "text": "rule " * 420}
conv = [{"role": "user", "text": "question " * 50}, {"role": "model", "text": "answer " * 120 + "FLAGGED_SPAN"}]
old_input = [t for turn in [preamble] + conv for t in turn["text"].split()][:512]
assert score_fn(old_input) == 0.0
assert classify([preamble] + conv, score_fn) == 1.0
for n in range(0, 2000, 37):
    covered = set()
    for i, c in enumerate(chunks(list(range(n)))):
        covered.update(c)
        assert len(c) <= 512
    assert covered == set(range(n))
print("Q6 ok")
```

Verify: the old input drops the flagged span and the fixed one catches it. The coverage loop proves every token lands in some chunk for lengths up to 2,000.

Pitfalls: raising the context length alone, which only moves the point where truncation starts, and fixing the classifier without adding long, preamble-wrapped transcripts to the standard eval set. Leave that gap open and the next template change will repeat this incident.

Follow-ups: a monitor on the truncation rate per template version, attacks that split content across chunk boundaries (hence the overlap), and conversation-level classifiers.

Why this is likely: the posting describes digging into "small discrepancies that hint at big problems" and owning safeguard reliability, and team-run ML design rounds for Research Engineers are reported to use the team's own problems (medium confidence).

</details>

<a id="q7"></a>

#### Q7. (Coding or ML coding, no AI) A teammate's coding assistant wrote this automated red-teaming loop. Review it like a pull request: find the defects, then fix them.

Round: Coding or ML coding. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: Reviewing AI-generated code.

<details>
<summary>Show answer</summary>

Task: review this diff. `attacker`, `target` and `judge` are callables provided by the team's libraries.

```python
import hashlib, json, random

def run_red_team(attacker, target, judge, seeds, attempts_per_seed=5, log_path="/tmp/redteam.log"):
    random.seed_everything(0)
    seen, verdicts = set(), []
    for seed in seeds:
        for i in range(1, attempts_per_seed):
            prompt = attacker(seed, temperature=0.0)
            h = hashlib.md5(prompt.encode()).hexdigest()
            if h in seen:
                continue
            seen.add(h)
            response = target(prompt)
            try:
                verdict = judge(prompt, response)
            except Exception:
                verdict = "safe"
            verdicts.append(verdict)
            with open(log_path, "a") as f:
                f.write(json.dumps({"prompt": prompt, "response": response}) + "\n")
    return verdicts.count("unsafe") / len(seeds)
```

Approach: read it top to bottom and check every call against the real API. Findings, worst first:
1. Security: raw attack prompts and target responses are written to a world-readable file in /tmp. On this team that is sensitive data under strict controls. Log ids, hashes and verdicts, and keep raw text in the access-controlled store.
2. Fail-open judge: any judge exception becomes "safe", which lowers the attack success rate exactly when the judge breaks. Count errors and report them.
3. Hallucinated API: `random.seed_everything` does not exist in Python's `random` module. Use a local `random.Random(seed)` passed to the attacker.
4. Off-by-one: `range(1, attempts_per_seed)` runs 4 attempts, not 5.
5. Temperature 0 plus hash dedupe means every attempt after the first is identical and skipped, so each seed effectively gets one attempt.
6. Wrong denominator: successes are divided by seeds, not attempts, which mixes two different metrics. It also crashes on an empty seed list.

```python
import hashlib, random

def run_red_team(attacker, target, judge, seeds, attempts_per_seed=5, temperature=1.0, rng_seed=0, log=None):
    if not seeds:
        raise ValueError("no seeds")
    rng = random.Random(rng_seed)
    attempts = successes = judge_errors = 0
    seed_hit = {}
    for seed in seeds:
        seed_hit[seed["id"]] = False
        for i in range(attempts_per_seed):
            prompt = attacker(seed, temperature=temperature, rng=rng)
            response = target(prompt)
            attempts += 1
            try:
                verdict = judge(prompt, response)
            except Exception:
                judge_errors += 1
                verdict = "error"
            if verdict == "unsafe":
                successes += 1
                seed_hit[seed["id"]] = True
            if log is not None:
                log.append({"seed": seed["id"], "attempt": i, "verdict": verdict,
                            "prompt_sha256": hashlib.sha256(prompt.encode()).hexdigest()})
    return {"attempt_asr": successes / attempts, "seed_asr": sum(seed_hit.values()) / len(seeds),
            "attempts": attempts, "judge_errors": judge_errors}

attacker = lambda seed, temperature, rng: f"{seed['text']} variant {rng.random():.6f}"
target = lambda p: "COMPLIED" if p.endswith(("1", "3")) else "refused"
def judge(p, r):
    if p.endswith("7"):
        raise TimeoutError
    return "unsafe" if r == "COMPLIED" else "safe"
log = []
out = run_red_team(attacker, target, judge, [{"id": k, "text": f"seed {k}"} for k in range(20)], log=log)
assert out["attempts"] == 100 and len(log) == 100
assert all("prompt" not in row for row in log) and out["judge_errors"] > 0
assert 0 < out["attempt_asr"] <= out["seed_asr"] <= 1
print("Q7 ok", out)
```

Verify: exactly 5 attempts per seed, judge errors counted rather than hidden, no raw text in the log, and per-seed success never below per-attempt success.

Pitfalls: approving the diff because it runs. Most of these defects produce a plausible number rather than a crash.

Follow-ups: deduplicate attacks by semantic similarity after judging, not before; add a judge-error budget that fails the run; and report results per risk category.

Why this is likely: the posting asks you to attack safeguards with automated red teaming and to handle sensitive data under strict controls, and Research Engineer coding rounds expect working, reviewed code without AI help (medium confidence).

</details>

### ML system design: evaluations, safeguards and red teaming (no AI)

<a id="q8"></a>

#### Q8. (ML system design, no AI) Design an evaluation harness that measures chemical, biological, radiological and nuclear risk in a frontier model, including agentic tasks with tools, and gates launches on the result.

Round: ML system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Dangerous-capability evals.

<details>
<summary>Show answer</summary>

Task: every new checkpoint and every launch candidate must get a risk read before release. Experts write the items. Results feed a decision: below an early-warning threshold, at it, or past a critical level that triggers mitigations.

Approach: I would split the harness into four layers. First, an item store: expert-written items in tiers (knowledge questions, multi-step protocol-style tasks described only abstractly in the interview, and agentic tasks where the model uses tools such as search or a code sandbox). Items are versioned, access-controlled and never leave the secure environment. Second, a runner that pins everything that changes a score: checkpoint, system prompt, sampling settings, tool versions, and seeds. Each run records a manifest hash so any score can be reproduced. Third, graders: rubric-based autograders checked against expert labels (see Q4), with experts grading a random 10% plus every borderline case. Fourth, a reporting layer that shows rates with intervals, compares against a reference model and a human-with-internet baseline, and states what the result does and does not show.

For agentic tasks, the tool sandbox has no real network access to sensitive resources and logs every tool call. The grader scores the whole trajectory, because a model can make real progress and still fail at the last step.

Numbers: with about 300 items per tier, a rate near 20% has a 95% interval about plus or minus 4.5 points, which is enough to separate a model from its predecessor only on large shifts. A full run with agentic tasks at 50 tool steps each, 300 tasks and 5 samples is 75,000 model calls, so plan for a run of several hours, and run the cheap knowledge tier on every checkpoint and the full suite on launch candidates.

Alternatives: public benchmarks only (cheap and comparable, but saturated and possibly in the training data), human red-team uplift studies (the most valid evidence, but slow and costly, so I keep them for threshold decisions), and automated tiers in between.

Risks: contamination from items leaking into training data (canary strings and held-out private items), elicitation gaps where the model can do more than the harness asks (try scaffolds and fine-tuned elicitation before calling a result safe), and grader drift across model generations.

Why this is likely: the posting asks you to design, build and run evaluations for this risk area including agentic systems risk, Google DeepMind's published frontier safety reports describe early-warning alert thresholds for this domain, and Research Engineer guides describe an evaluation-infrastructure design round (high confidence on topic).

</details>

<a id="q9"></a>

#### Q9. (ML system design, no AI) Design a layered safeguard that blocks hazardous assistance while keeping refusals on legitimate science questions low, and explain how you would measure its cost to real researchers.

Round: ML system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Safeguard design and over-refusal.

<details>
<summary>Show answer</summary>

Task: Gemini answers millions of science questions a day. A tiny fraction are hazard-class requests. Build a safeguard that catches those, and prove that chemists, biologists and physicists can still get their work done.

Approach: layers, cheapest first. Model-level training (refusal and safe-completion behaviour) handles most cases at no serving cost. A fast input classifier scores every request. An output classifier scores the model's reply, because the harm is in what the answer says. A conversation-level classifier catches requests split across many harmless-looking turns. Only the small slice scored high by the fast layer goes to a larger, slower classifier. Actions are graded too: answer, answer at a general level, or decline with a pointer to legitimate resources.

Measuring cost to research is the hard part and the one interviewers push on. I would build a benign science set with domain experts: real questions from coursework, lab safety, clinical work and published research, weighted toward topics that sit near the hazard boundary, because that is where over-refusal happens. Track refusal rate and partial-answer rate on that set for every safeguard change, next to recall on hazard-class items. Add an online signal: sampled refusals reviewed by experts each week, and user reports of wrong refusals.

Numbers: at 10 million science requests a day, a 0.5% false-positive rate on the first layer means 50,000 flagged requests a day. If the second stage costs 50 ms and runs only on those, it adds little total compute, while running it on everything would add 50 ms to every request. A refusal rate on the benign set that rises from 1% to 3% is a tripling that researchers will notice, even if it looks small.

Alternatives: one large classifier on everything (simpler, but the latency and cost scale with all traffic), keyword or topic blocklists (cheap but they refuse whole fields of science and are easy to evade), and model-only refusals (no serving cost, but they can be jailbroken and are harder to update fast).

Risks: attackers probing the boundary, so red-team every layer (Q10); thresholds tuned on stale data; and fail-open behaviour when a classifier times out (Q11).

Why this is likely: the posting asks you to build safeguards and measure their cost to legitimate scientific research, and the team says it ships safeguards to billions of users (high confidence on topic).

</details>

<a id="q10"></a>

#### Q10. (ML system design, no AI) Design an automated red-teaming system that keeps finding new attacks against your safeguards instead of rediscovering the same ones.

Round: ML system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Automated red teaming.

<details>
<summary>Show answer</summary>

Task: manual red-teamers find strong attacks but cannot run every night. Build a system that attacks the current safeguard stack continuously, reports what got through, and feeds training data back to the classifiers.

Approach: an attacker model generates attempts from seeds written by experts. Seeds describe goals at a placeholder level, never real hazardous content. Each attempt goes through the full production stack, because attacks often succeed in the gap between two layers. A judge scores the outcome, and experts review a sample of successes and every new attack family. To avoid rediscovering the same attack, reward novelty: embed each successful attack, cluster them, and give the attacker a bonus for landing outside known clusters. Track coverage as the number of distinct attack families found per week and the share of risk categories with at least one successful family.

Successful attacks also feed the defence. They become hard negatives for classifier training, but hold out whole attack families for evaluation, otherwise the classifier memorizes the exact strings and the next eval looks great for the wrong reason.

Numbers: if 1 in 1,000 attempts succeeds, finding 50 successes a night takes 50,000 attempts through the stack. At about 1 second per attempt across 100 parallel workers, that is around 8 minutes of wall time plus judge cost, which fits a nightly job.

Alternatives: manual red teaming only (highest quality per attack, lowest coverage), fixed jailbreak libraries replayed every night (good for regression testing but they find nothing new), and gradient-based attacks on open classifiers (strong, but they need white-box access and often produce strings that fail on the full system).

Risks: the attacker collapsing onto one trick (watch cluster counts), the judge being fooled by the same trick as the classifier (use a different judge model and expert review), and sensitive outputs from successful attacks, which must stay in the access-controlled store (Q15).

Why this is likely: the posting asks you to attack safeguards with automated and manual red teaming, and team-run ML design rounds for Research Engineers are reported to use the team's own problems (high confidence on topic).

</details>

<a id="q11"></a>

#### Q11. (ML system design or final interviews, no AI) The safeguard classifier service starts timing out at peak load. You are on call. Decide between failing open and failing closed, and design the system so the decision is not made at 3 a.m.

Round: ML system design or final interviews. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Safeguard reliability and incident response.

<details>
<summary>Show answer</summary>

Task: p99 latency on the output classifier jumped from 80 ms to 2 s and 4% of calls time out. The serving path currently treats a timeout as "allow". Handle the incident, then fix the design.

Approach: during the incident, first limit the damage while keeping the safety check in place. Fail-open means hazard-class replies can ship unchecked while the classifier is down, so for this surface I would switch to a degraded but safe mode: route requests whose fast first-layer score is above a low threshold to a fail-closed path (decline with a retry message), and let clearly benign traffic through. That confines the cost to a small share of users, while everyone else is still served and checked. Then find the cause: a traffic spike, a bad deploy, a dependency, or a long-input pattern (see Q6). Roll back if a deploy lines up with the start.

Design fix: decide the failure policy per surface in advance and write it in the runbook, with sign-off from safety and product. Add capacity headroom and autoscaling on queue depth, load shedding that drops low-risk traffic first, a smaller fallback classifier that is always warm, and alerts on timeout rate, not just latency. Every timeout is logged with its decision, so the postmortem can count what went through unchecked.

Numbers: at 2,000 requests per second, a 4% timeout rate under fail-open lets 80 unchecked replies through every second. If the fast layer sends 2% of traffic down the fail-closed path, that path declines about 40 requests a second, which is a visible but bounded cost.

Alternatives: always fail closed (safest, but a classifier outage becomes a full product outage, which pushes teams to weaken the safeguard later), and always fail open (best availability, but unacceptable for this risk class).

Risks: the degraded mode itself becoming the attack, since an attacker who can trigger slow classification gets the open path. Treat a timeout on a high-score input as a decline.

Why this is likely: the posting lists reliability of safeguards, incident response and an on-call rotation as part of the job (high confidence on topic).

</details>

### ML fundamentals: statistics, thresholds and training (no AI)

<a id="q12"></a>

#### Q12. (ML fundamentals, no AI) Experts say a model should complete fewer than 5% of a hard task set. How many expert-graded items do you need to show that with 95% confidence, and what changes if you check results after every batch?

Round: ML fundamentals. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Sample size and sequential testing.

<details>
<summary>Show answer</summary>

Task: the claim to support is "true success rate below 5%". Items are expensive because experts write and grade them, so the number matters.

Approach: use an exact one-sided binomial test. The claim holds at 95% confidence if, assuming the true rate were exactly 5%, seeing as few successes as we saw would happen at most 5% of the time. With zero successes, the condition is 0.95^n at or below 0.05, which gives n = 59 (0.95^58 is 0.051 and 0.95^59 is 0.048). This is close to the "rule of three", 3/n as an upper bound when nothing succeeds. But zero successes is a lucky case. If the model's true rate is 2%, we need enough items that we usually see few enough successes. An exact power calculation gives 234 items for 80% power, with up to 6 successes allowed.

Checking after every batch and stopping as soon as the bound passes inflates the false "safe" rate. The simulation below, with a true rate of exactly 5% and a check after each batch of 20 up to 400 items, declares "below 5%" in about 13% of runs instead of at most 5%. Fixes: fix n in advance, or use a sequential design with a stricter per-look threshold such as an alpha-spending rule.

```python
from math import comb
import random
from functools import lru_cache

def cdf(x, n, p):
    return sum(comb(n, i) * p ** i * (1 - p) ** (n - i) for i in range(x + 1))

@lru_cache(None)
def max_allowed(n, bound=0.05, alpha=0.05):
    x = -1
    while x + 1 <= n and cdf(x + 1, n, bound) <= alpha:
        x += 1
    return x

def power(n, true_p):
    x = max_allowed(n)
    return cdf(x, n, true_p) if x >= 0 else 0.0

assert next(n for n in range(1, 500) if max_allowed(n) >= 0) == 59
n80 = next(n for n in range(1, 2000) if power(n, 0.02) >= 0.8)
assert n80 == 234 and max_allowed(234) == 6

rng, trials, false_safe = random.Random(0), 4000, 0
for _ in range(trials):
    hits = 0
    for look in range(1, 21):
        hits += sum(rng.random() < 0.05 for _ in range(20))
        if hits <= max_allowed(20 * look):
            false_safe += 1
            break
rate = false_safe / trials
assert 0.10 < rate < 0.17
print("Q12 ok", n80, round(rate, 3))
```

Verify: 59 matches the closed form, the power search returns 234 with at most 6 successes, and the peeking rate is well above the nominal 5%.

Pitfalls: treating items as independent when many come from a few expert-written templates (the effective sample size is smaller), and pooling task tiers with very different difficulty into one rate.

Alternatives: the rule of three (zero successes in n items gives a 95% upper bound of about 3/n, so 60 items) is a quick check but only covers zero successes. The exact binomial test handles any count, so I use it here. A Bayesian Beta posterior bound is easier to explain to leadership but depends on the prior. For checking after every batch, a fixed-n design is simplest, while an alpha-spending sequential design (for example O'Brien-Fleming) allows early stopping without inflating the error rate.

Follow-ups: how the answer changes with 5 samples per item, and how to report "we could not rule out 5%" without it reading as "the model is dangerous".

Why this is likely: the posting asks for statistics applied to ML evaluation and for communicating uncertainty and negative findings, and Research Engineer candidates report ML fundamentals rounds with probability questions and derivations (medium confidence).

</details>

<a id="q13"></a>

#### Q13. (ML fundamentals, no AI) Your classifier flags hazard-class requests. Missing one costs far more than a false refusal, and true hazard requests are about 1 in 10,000. Choose the operating threshold and show the math.

Round: ML fundamentals. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Thresholds under asymmetric costs.

<details>
<summary>Show answer</summary>

Task: assume a miss costs 1,000 times a false refusal. On validation data, benign scores look like a normal distribution with mean 0 and hazard scores like a normal with mean 4, both with standard deviation 1. Pick the threshold and say what it means in production.

Approach: with a calibrated probability, the expected-cost rule is to flag when P(hazard | x) is at least C_FP / (C_FP + C_FN), here 1/1001. In likelihood-ratio form, flag when the ratio of hazard to benign density exceeds C_FP (1 - pi) / (C_FN pi), which is 0.9999 / 0.1, about 10. For these two normals the ratio is exp(4t - 8), so t = (8 + ln 10.0) / 4, about 2.58. At that threshold the false-positive rate is about 0.50% and recall is about 92%.

Now the production view. Precision is pi times recall divided by that plus (1 - pi) times the false-positive rate, about 1.8%. At 10 million requests a day, that is roughly 50,000 false flags to catch about 920 of the 1,000 hazard requests. So the design needs more than one threshold. A second-stage classifier or a softer action (general answer instead of refusal) on the flagged slice brings the user cost down (see Q9).

```python
from math import erf, exp, log, sqrt

phi = lambda x: 0.5 * (1 + erf(x / sqrt(2)))
pi_h, c_fn, c_fp, mu = 1e-4, 1000.0, 1.0, 4.0

def stats(t):
    fpr, tpr = 1 - phi(t), 1 - phi(t - mu)
    cost = c_fn * pi_h * (1 - tpr) + c_fp * (1 - pi_h) * fpr
    return fpr, tpr, cost

grid = [i / 1000 for i in range(0, 6001)]
t_best = min(grid, key=lambda t: stats(t)[2])
t_closed = (mu * mu / 2 + log(c_fp * (1 - pi_h) / (c_fn * pi_h))) / mu
assert abs(t_best - t_closed) < 0.002 and abs(t_closed - 2.576) < 0.001
fpr, tpr, _ = stats(t_closed)
precision = pi_h * tpr / (pi_h * tpr + (1 - pi_h) * fpr)
assert abs(fpr - 0.0050) < 0.0002 and abs(tpr - 0.923) < 0.001 and abs(precision - 0.018) < 0.001
print("Q13 ok", round(t_closed, 3), round(fpr, 4), round(tpr, 3), round(precision, 4),
      "false flags per 10M:", round(1e7 * (1 - pi_h) * fpr))
```

Verify: the grid search agrees with the closed form, and the precision, recall and false-flag numbers match the text.

Pitfalls: picking a threshold on a balanced validation set and forgetting the base rate, using an uncalibrated score as a probability (Q14), and treating the 1,000 cost ratio as known. Show how the threshold moves when the ratio is 100 or 10,000, because that judgment belongs to safety and product leads, not to the engineer alone.

Follow-ups: choosing a threshold when the base rate shifts during an attack wave, and setting different thresholds per surface.

Why this is likely: Research Engineer candidates report ML fundamentals questions on precision and recall with deeper follow-ups, and the posting's core job is building safeguards with a measured cost to legitimate users (medium confidence).

</details>

<a id="q14"></a>

#### Q14. (ML fundamentals, no AI) You have 800 expert-labelled examples and a large pool of unlabelled traffic. Fine-tune a classifier for a new hazard category and make its scores trustworthy enough to threshold.

Round: ML fundamentals. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Low-data fine-tuning and calibration.

<details>
<summary>Show answer</summary>

Task: 800 labels, about 120 of them positive. Labels are sensitive and expensive. The model will run on production traffic where positives are far rarer than in the labelled set.

Approach: start from a strong pretrained model and train a small head or a parameter-efficient adapter (for example LoRA) instead of full fine-tuning, because 800 examples will overfit a full model. Split by source, not by row, so near-duplicates from one expert do not sit on both sides. Hold out 200 examples for calibration and thresholding. Use the unlabelled pool in two ways. Active learning: score the pool, send the most uncertain and the highest-scored items to experts, and add those labels, which usually beats random labelling for rare classes. Hard negatives: benign science text near the boundary, so the classifier learns the difference between the topic and the harm.

Calibration: classifiers fine-tuned with class weighting or oversampling are often overconfident and their scores do not match frequencies. Temperature scaling fits one parameter T on the held-out set to minimize log loss, dividing logits by T. It keeps the ranking (so recall at a fixed false-positive rate does not change) and fixes the probabilities, which Q13's threshold rule needs. Then correct for the base rate: the labelled set has 15% positives and production has far fewer, so shift the logit by log of the production prior odds over the training prior odds.

```python
import math, random

sig = lambda z: 1 / (1 + math.exp(-z))
rng = random.Random(0)
data = []
for _ in range(4000):
    z = rng.gauss(0, 4)
    data.append((z, 1 if rng.random() < sig(z / 2.0) else 0))
val, test = data[:2000], data[2000:]

def nll(rows, T):
    return -sum(y * math.log(max(sig(z / T), 1e-12)) + (1 - y) * math.log(max(1 - sig(z / T), 1e-12)) for z, y in rows) / len(rows)

def ece(rows, T, bins=10):
    total, err = len(rows), 0.0
    for b in range(bins):
        sel = [(sig(z / T), y) for z, y in rows if b / bins <= sig(z / T) < (b + 1) / bins or (b == bins - 1 and sig(z / T) == 1.0)]
        if sel:
            err += len(sel) / total * abs(sum(p for p, _ in sel) / len(sel) - sum(y for _, y in sel) / len(sel))
    return err

T = min((t / 100 for t in range(50, 501)), key=lambda t: nll(val, t))
assert 1.7 < T < 2.3 and ece(test, T) < ece(test, 1.0) / 2
prior_shift = math.log((1e-3 / (1 - 1e-3)) / (0.15 / 0.85))
print("Q14 ok", T, round(ece(test, 1.0), 3), round(ece(test, T), 3), round(prior_shift, 2))
```

Verify: on synthetic logits that are twice too confident, the fit recovers T near 2 and cuts the calibration error on a separate test split by more than half.

Pitfalls: calibrating on the training split, reporting accuracy on a set with 15% positives as if it held in production, and letting active learning sample only from one traffic source.

Follow-ups: when to prefer isotonic regression (enough held-out data, roughly 1,000 or more examples, and a miscalibration shape that a single temperature cannot fix, since isotonic fits any monotone mapping), how to detect drift after launch, and how to keep labelled examples secure (Q15).

Why this is likely: the posting asks for experience building classifiers for large language models and for training, fine-tuning or evaluating models, and candidates report ML fundamentals rounds that ask how you would train and optimize a model (medium confidence).

</details>

### Data security and working style

<a id="q15"></a>

#### Q15. (ML system design or final interviews, no AI) Design the data pipeline for sensitive expert-written eval items: who can read them, how graders use them, how you stop leakage into training data, and how you audit it.

Round: ML system design or final interviews. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Sensitive data handling.

<details>
<summary>Show answer</summary>

Task: domain experts write eval items and reference answers that are sensitive by design. Engineers need to run evals and debug graders. The items have to stay out of training data and logs, and only the people who need them can read them.

Approach: treat the items like secrets with a lifecycle. Store them encrypted in a dedicated store with access granted per project and per person, time-limited, and approved by the data owner. Engineers work through tooling that runs evals inside the secure environment and returns aggregate scores, item ids and grader verdicts, not raw text. Raw text access for debugging goes through a separate, logged path with a stated reason. Graders and models that read the items run in the same environment, and their outputs are stored there too, because a model's answer to a sensitive item can be as sensitive as the item.

Leakage into training: add unique canary strings to every item and scan training corpora and model outputs for them. Keep a hash index of item text and filter training data against it with fuzzy matching, since paraphrases leak too. Send items only to models and graders that run inside the secure environment.

Audit: every read is logged with who, what, when and why. A weekly review checks for unusual access, and an alert fires on bulk reads. Items carry a version and an owner, and retired items are deleted on a schedule.

Numbers: with 2,000 items and 30 people on the project, most engineers should need zero raw reads in a normal week, so even 50 raw reads by one person is worth a question.

Alternatives: a fully air-gapped environment (strongest, but it slows iteration and pushes people toward workarounds), and redacted copies for debugging (useful, but redaction can remove exactly the part the grader got wrong).

Risks: secrets leaking through debugging screenshots, notebooks and chat threads more often than through the store itself. Give engineers good debugging tools inside the secure environment so they have no reason to copy items out.

Why this is likely: the posting says the team manages sensitive data under strict security controls and works with expert-written datasets, and the hiring manager stage is reported to cover whether candidates fit the team's working conditions (high confidence on topic).

</details>

<a id="q16"></a>

#### Q16. (Final interviews) Tell me about a time you had to report a negative or uncertain result to leadership when people wanted a clear answer.

Round: Final interviews. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Communicating uncertainty.

<details>
<summary>Show answer</summary>

Task: tell one specific story in STAR form (situation, task, action, result), which Google DeepMind's candidate guide recommends. Interviewers want evidence that you will say "we do not know yet" under release pressure, and that you will say it in a way that still helps a decision get made.

Approach: pick a story where the stakes were real and you could have rounded the result in the convenient direction. A good shape:
- Situation: a launch was planned, and your eval showed a borderline or noisy result, for example an improvement that sat inside the confidence interval, or a metric that moved only because the eval changed.
- Task: leadership needed a go or no-go recommendation by a date.
- Action: describe what you did to understand the result before reporting it (reran with more samples, checked the harness, as in Q5), then how you reported it: the finding in one sentence, the interval, what would change the conclusion, and a concrete option such as "launch with this mitigation and rerun with 2,000 items by Friday".
- Result: what was decided and what happened afterwards, including whether you were right.

Pitfalls: a story where the uncertainty was someone else's fault, a story without numbers, or a heroic ending where you were simply proven right. Interviewers value what you would do differently.

Verify: before the interview, check the story against the posting's line about communicating results, "including uncertainty and negative findings", and make sure it shows both technical care and judgment about the decision.

Follow-ups: what you do when a senior person pushes back, how you write it up for people who were not in the room, and a time you reported a result that later turned out to be wrong.

Why this is likely: the posting explicitly asks for communicating uncertainty and negative findings to leadership, and Research Engineer candidates report a final behavioral round with standard "tell me about a time" questions (high confidence).

</details>

## Prep checklist

- [ ] Solve 15 to 20 medium Google-tagged problems in a plain editor with no autocomplete, and make each solution run with a small test before you call it done.
- [ ] Practice sliding windows, topological sort and heaps, and say the complexity out loud before coding.
- [ ] Implement from memory: unbiased pass@k, a Wilson interval, Cohen's kappa, a bootstrap interval and temperature scaling.
- [ ] Be able to compute precision at a 1 in 10,000 base rate and a sample size for "rate below 5%" on paper.
- [ ] Prepare a 45 minute design for a risk evaluation harness that includes agentic tasks, with run-cost numbers.
- [ ] Prepare designs for a layered safeguard with an over-refusal metric and for an automated red-teaming loop with novelty and coverage metrics.
- [ ] Practice finding bugs in eval harness and classifier code that fail toward "safe": silent grader errors, truncation and substring refusal checks.
- [ ] Write a one-page incident runbook for a safeguard outage, including when to fail open or closed.
- [ ] Read Google DeepMind's Frontier Safety Framework and a recent model's frontier safety report, and be able to explain alert thresholds and critical capability levels.
- [ ] Prepare two stories: reporting an uncertain or negative result, and a production incident you owned end to end.
- [ ] Ask the recruiter how many coding rounds you will have, whether there is an ML coding round, and who runs the design round.

## Sources

- https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf
- https://deepmind.google/careers/
- https://www.google.com/about/careers/applications/jobs/results/105041602358977222-research-engineer-chemical-biological-and-nuclear-ai-safety-deepmind
- https://www.glassdoor.com/Interview/Google-DeepMind-Research-Engineer-Interview-Questions-EI_IE1596815.0,15_KO16,33.htm
- https://www.techinterview.org/post/3233474918/deepmind-interview-process-2026
- https://www.datainterview.com/blog/google-deepmind-research-engineer-interview
- https://www.algomindset.com/blog/google-deepmind-research-engineer-interview-guide (low confidence, prep vendor)
- https://www.teamblind.com/post/deepmind-research-engineer-interview-process-wp1gjgyb (older first-hand report)
- https://www.hackingthecaseinterview.com/pages/google-deepmind-gemini-interview (low confidence, prep vendor)
- https://storage.googleapis.com/deepmind-media/gemini/gemini_3_pro_fsf_report.pdf
- https://deepmind.google/models/model-cards/gemini-3-1-pro
- https://www.businessinsider.com/google-interview-process-ai-gemini-coding-comprehension-software-engineer-2026-10
- https://github.com/liquidslr/leetcode-company-wise-problems
