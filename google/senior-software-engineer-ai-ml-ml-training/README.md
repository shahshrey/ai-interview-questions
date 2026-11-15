# Google: Senior Software Engineer, AI/ML, ML Training

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

No interview report specific to the JAX Models and Performance team was found. Google hires into a general pool and then runs team matching, so this is the L5 AI/ML track loop from Google's own hiring pages, a living L5 guide and recent candidate reports, with topics fitted to a team that tunes open-source JAX LLM and diffusion implementations (MaxText, MaxDiffusion) on TPUs and GPUs. Ask your recruiter which variant you get.

1. Google Hiring Assessment (about 30 minutes, online). A work-style questionnaire; senior candidates sometimes skip it. AI: not relevant.
2. Recruiter screen (30 minutes). Background, level calibration, and on AI/ML tracks you choose your ML domain here. AI: not relevant.
3. Technical phone screen (2 x 45 minutes). One algorithm problem each in Google's interview editor, which cannot run code. Many L5 candidates report one of the two swapped for a behavioral talk. AI: not allowed.
4. Onsite coding (1 to 2 rounds, 45 minutes each). LC medium to hard with follow-ups on complexity and scale, often on a whiteboard in person. AI: not allowed.
5. Code comprehension round with Gemini (45 to 60 minutes, possible). Read, debug and optimize an existing codebase with Gemini as the provided assistant; graded on prompting and on validating the output. Google calls it a pilot; a press report says it has rolled out more broadly and recruiters are naming it ahead of loops. Where it runs it replaces one classic coding round. AI: provided (Gemini only).
6. ML domain round (role-related knowledge, 45 minutes). Fundamentals with depth. For this team expect performance: rooflines, MFU, parallelism, numerics. AI: not allowed.
7. System design (45 minutes). Vague prompt, trade-offs and capacity math; for ML infra expect training infrastructure. AI: not allowed.
8. Googleyness and Leadership (45 minutes). Ambiguity, conflict, influence without authority. AI: not allowed.
9. Hiring committee, then team matching, then offer.

Two questions below are classic algorithm problems because the phone screen and onsite coding still use them. Five are code comprehension tasks in the Gemini round format; if your loop has no Gemini round, practise them without AI as onsite coding.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | A model has n layers with per-layer step costs costs[i]. Split the layers, in order... | Classic algorithm | Phone screen or onsite coding | Not allowed | Hard |
| [Q2](#q2) | Given profiler events (device, op_name, start_us, end_us) for one training step... | Classic algorithm | Phone screen or onsite coding | Not allowed | Medium |
| [Q3](#q3) | Step time is 40 seconds instead of 0.4 seconds, and the log shows a compile on every... | Debug | Code comprehension (Gemini) | Provided (Gemini) | Medium |
| [Q4](#q4) | A profile shows the TPU idle 55 percent of each step. Find the host-side bottlenecks... | Optimize | Code comprehension (Gemini) | Provided (Gemini) | Medium |
| [Q5](#q5) | After switching to bf16 and 8-way gradient accumulation, loss plateaus higher than... | Debug | Code comprehension (Gemini) | Provided (Gemini) | Hard |
| [Q6](#q6) | Gemini generated this sharded MoE dispatch for a 16 x 16 TPU mesh. Review it before... | Review | Code comprehension (Gemini) | Provided (Gemini) | Hard |
| [Q7](#q7) | Add sliding-window attention to this MaxText-style model behind a config flag. The... | Build | Code comprehension (Gemini) | Provided (Gemini) | Medium |
| [Q8](#q8) | A bf16 matmul multiplies a [B, 8192] activation by an [8192, 8192] weight on one TPU... | Concept plus code | ML domain | Not allowed | Medium |
| [Q9](#q9) | An 8B dense model trains on 256 v5e chips with 1M tokens per step and a 2.0 second... | Concept plus code | ML domain | Not allowed | Medium |
| [Q10](#q10) | Plan the parallelism for training a 47B-parameter MoE (8 experts, top-2, about 13B... | Design | ML domain | Not allowed | Hard |
| [Q11](#q11) | A text-to-image diffusion model takes about 6 seconds per image: 50 sampling steps... | Optimize | ML domain | Not allowed | Medium |
| [Q12](#q12) | Design a service that benchmarks 30 model configurations nightly on TPU and GPU and... | Design | System design | Not allowed | Hard |
| [Q13](#q13) | A training job runs for weeks on preemptible TPU slices. A checkpoint takes 60... | Design | System design | Not allowed | Hard |
| [Q14](#q14) | Design the release pipeline that ships the team's pip wheels and docker images so a... | Design | System design | Not allowed | Medium |
| [Q15](#q15) | After a library upgrade, step time went from 1.0 to 1.5 seconds with the same model... | Trace debug | ML domain | Not allowed | Hard |
| [Q16](#q16) | You replaced the attention kernel with a faster fused one. Design the evaluation... | Design | ML domain | Not allowed | Medium |
| [Q17](#q17) | Tell me about a time an AI tool suggested a performance change that looked right but... | Behavioral | Googleyness and Leadership | Not allowed | Medium |

### Classic coding (no AI, as at Google's phone screen and onsite)

<a id="q1"></a>

#### Q1. (Phone screen or onsite coding, no AI, cannot run code) A model has n layers with per-layer step costs costs[i]. Split the layers, in order, into at most k contiguous pipeline stages so the slowest stage is as fast as possible. Return that cost and one valid split.

Round: Phone screen or onsite coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Hard. Topic: Binary search on the answer, pipeline parallelism.
Related practice: [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) (Hard)

<details>
<summary>Show answer</summary>

Task: given costs = [7, 2, 5, 10, 8] and k = 2, return 18 with stages [[0, 1, 2], [3, 4]].

What they grade: spotting the monotone predicate, a correct greedy check, clean bounds, and complexity stated without being asked.

Approach: The answer lies between max(costs), since one layer cannot be split, and sum(costs), one stage. The predicate "can we finish with at most k stages if no stage exceeds limit" is monotone: a larger limit never needs more stages. So binary search the smallest limit where the greedy check passes. The greedy check walks layers left to right and opens a new stage only when adding the next layer would exceed the limit; greedy is optimal here because packing a stage as full as possible never forces more stages later. Each check is O(n) and the search is O(log(sum)), so the total is O(n log S) time and O(1) extra space.

Alternatives: dynamic programming over (layer, stages) is O(k n squared) and only worth it if stage cost is not additive, for example when a stage boundary adds a fixed activation transfer cost. Then I would add the boundary cost inside the DP and drop the binary search.

Code:

```python
def min_max_stage(costs, k):
    """Split layers (in order) into at most k contiguous stages; minimize the slowest stage."""
    if not costs or k <= 0:
        raise ValueError("need at least one layer and one stage")

    def stages_needed(limit):
        count, cur = 1, 0
        for c in costs:
            if cur + c > limit:
                count, cur = count + 1, 0
            cur += c
        return count

    lo, hi = max(costs), sum(costs)
    while lo < hi:
        mid = (lo + hi) // 2
        if stages_needed(mid) <= k:
            hi = mid
        else:
            lo = mid + 1
    return lo


def partition(costs, limit):
    stages, cur = [[]], 0
    for i, c in enumerate(costs):
        if cur + c > limit:
            stages.append([])
            cur = 0
        stages[-1].append(i)
        cur += c
    return stages


layers = [7, 2, 5, 10, 8]
assert min_max_stage(layers, 2) == 18
assert partition(layers, 18) == [[0, 1, 2], [3, 4]]
assert min_max_stage(layers, 5) == 10
assert min_max_stage(layers, 1) == 32
assert min_max_stage([4, 4, 4, 4], 3) == 8
big = [3] * 96
assert min_max_stage(big, 8) == 36
print("q1 ok")
```

Verify: k = 1 returns the sum; k = n returns the max; equal costs split evenly; the returned split has at most k stages and no stage above the limit.

Pitfalls: starting lo below max(costs), where the greedy check must then handle a layer that fits in no stage; off by one in the while condition; claiming the greedy split uses exactly k stages when it may use fewer.

Follow-ups: costs are floats in milliseconds (search on a tolerance, or binary search over the sorted set of candidate prefix sums); the pipeline bubble adds (k minus 1) idle stage slots per batch, a fraction (k minus 1) / (m + k minus 1) with m micro-batches, so how many micro-batches make the split worth it?

Why this is likely: the L5 guide and 2026 candidate reports describe classic algorithm problems in the phone screen and onsite (no AI, cannot run code); binary search on the answer appears on crowd-reported Google practice lists, and this team works on pipeline sharding. Low to medium confidence on this exact problem.

</details>

<a id="q2"></a>

#### Q2. (Phone screen or onsite coding, no AI, cannot run code) Given profiler events (device, op_name, start_us, end_us) for one training step, return each device's idle fraction and the top k ops by total time. Events on one device can overlap.

Round: Phone screen or onsite coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Interval merging, top-k.
Related practice: [Merge Intervals](https://leetcode.com/problems/merge-intervals/) (Medium), [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) (Medium)

<details>
<summary>Show answer</summary>

Task: busy time is the union of a device's intervals clipped to the step window, so overlapping compute and communication count once. Op time is the plain sum, because it answers "where did the work go".

What they grade: noticing that overlap breaks a naive sum, sorting then merging, a heap for top k, and the memory follow-up.

Approach: Clip each event to [step_start, step_end), group by device, sort by start and merge: if the next start is past the current end, close the run and add its length, otherwise extend the end. Idle fraction is 1 minus busy over step length. For ops, sum durations into a dict and take heapq.nlargest(k), which is O(m log k) for m distinct ops. Total time is O(n log n) for the sorts.

Alternatives: a sweep line over all start and end points gives the same union and also concurrency counts; a full sort of the op dict is fine when m is small, and the heap wins when m is large and k is 10.

Code:

```python
import heapq
from collections import defaultdict


def busy_time(intervals):
    """Union length of [start, end) intervals."""
    total, cur_s, cur_e = 0, None, None
    for s, e in sorted(intervals):
        if cur_e is None or s > cur_e:
            if cur_e is not None:
                total += cur_e - cur_s
            cur_s, cur_e = s, e
        else:
            cur_e = max(cur_e, e)
    if cur_e is not None:
        total += cur_e - cur_s
    return total


def summarize(events, step_start, step_end, k):
    per_dev, per_op = defaultdict(list), defaultdict(int)
    for dev, op, s, e in events:
        s, e = max(s, step_start), min(e, step_end)
        if s < e:
            per_dev[dev].append((s, e))
            per_op[op] += e - s
    idle = {d: 1 - busy_time(iv) / (step_end - step_start) for d, iv in per_dev.items()}
    top = heapq.nlargest(k, per_op.items(), key=lambda kv: (kv[1], kv[0]))
    return idle, top


events = [
    ("tpu0", "fusion.matmul", 0, 400), ("tpu0", "all-gather", 350, 500),
    ("tpu0", "fusion.matmul", 600, 900), ("tpu0", "copy", 880, 950),
    ("tpu1", "fusion.matmul", 0, 300), ("tpu1", "all-gather", 700, 1100),
]
idle, top = summarize(events, 0, 1000, 2)
assert idle == {"tpu0": 1 - 850 / 1000, "tpu1": 1 - 600 / 1000}
assert top == [("fusion.matmul", 1000), ("all-gather", 450)]
assert busy_time([]) == 0 and busy_time([(0, 5), (5, 7)]) == 7
print("q2 ok")
```

Verify: touching intervals (end equals next start) merge without double counting; an event crossing the step boundary is clipped; an empty device list returns no entries; ties in top k are deterministic.

Pitfalls: summing raw durations for busy time (tpu0 above would show 920 busy microseconds instead of the true 850, because the overlaps are counted twice); forgetting the clip; sorting by end instead of start.

Follow-ups: the trace does not fit in memory. Ask whether each device's events can be read sorted by start; if not, externally sort each device's events in chunks first. Then merge each device stream with O(1) state. For op totals, aggregate per shard and merge the dicts, or keep a count-min sketch plus a small heap if op names are unbounded. Across 256 chips, map per host and reduce totals.

Why this is likely: a recent AI/ML track candidate report describes a top-k problem with a "data cannot fit in memory" follow-up, and trace analysis is the daily work named in the listing. Medium confidence.

</details>

### Code comprehension with Gemini (AI provided, as in Google's Gemini round)

<a id="q3"></a>

#### Q3. (Code comprehension round, Gemini provided) Step time is 40 seconds instead of 0.4 seconds, and the log shows a compile on every step. Find why with Gemini's help, fix it, and prove it compiles once.

Round: Code comprehension (Gemini). AI policy: Provided (Gemini). Type: Debug. Difficulty: Medium. Topic: JAX jit recompilation.

```text
# train.py (excerpt)
jax.config.update("jax_log_compiles", True)

@functools.partial(jax.jit, static_argnums=(3,))
def train_step(params, opt_state, batch, step):
    lr = schedule(step)                       # warmup then cosine
    loss, grads = jax.value_and_grad(loss_fn)(params, batch)
    updates, opt_state = opt.update(grads, opt_state, params)
    updates = jax.tree.map(lambda u: -lr * u, updates)
    return optax.apply_updates(params, updates), opt_state, loss

for step, batch in enumerate(loader):        # drop_remainder=False
    params, opt_state, loss = train_step(params, opt_state, batch, step)

# log
Compiling train_step with global shapes and types [... (256, 2048) int32 ...]
Compiling train_step with global shapes and types [... (256, 2048) int32 ...]
Compiling train_step with global shapes and types [... (37, 2048) int32 ...]
```

<details>
<summary>Show answer</summary>

Task: two causes. static_argnums=(3,) makes step part of the cache key, so every new integer compiles a new program. The last partial batch has a new shape, so it compiles again even after the first fix.

What they grade: reading the code before prompting, a narrow question to Gemini, checking its claim against the log, and a test that counts compiles.

Approach: I read the decorator and the loop first; the log already shows identical shapes compiling twice, which points at a static argument. My prompt: "In this file, list every input to train_step that is part of the jit cache key and explain which ones change per step. Do not rewrite the file." Gemini named step and also suggested wrapping the loop in jax.jit with donate_argnums. Donation saves memory but does not stop recompiles, and jitting a Python loop over a data iterator does not work, so I declined that part.

Fix: drop static_argnums and pass step as a traced scalar; schedules built from jnp ops accept a traced step, and the cleaner version reads the count from opt_state. For the last batch, pad to the full batch size and pass a mask so padded tokens add zero loss and the mean divides by real tokens, or drop the remainder in training.

Code:

```python
import numpy as np


class JitCacheModel:
    """jit keys its cache on input shapes/dtypes plus the values of static args."""

    def __init__(self, static_argnums=()):
        self.static_argnums = set(static_argnums)
        self.keys = set()

    def call(self, *args):
        key = tuple(
            ("static", a) if i in self.static_argnums
            else ("array", np.shape(a), np.asarray(a).dtype.str)
            for i, a in enumerate(args)
        )
        self.keys.add(key)
        return len(self.keys)


def padded_batches(n, bs, d):
    x_all = np.random.default_rng(0).normal(size=(n, d)).astype(np.float32)
    for i in range(0, n, bs):
        x = x_all[i:i + bs]
        mask = np.ones(bs, np.float32)
        if len(x) < bs:
            mask[len(x):] = 0.0
            x = np.concatenate([x, np.zeros((bs - len(x), d), np.float32)])
        yield x, mask


w = np.zeros(4, np.float32)
bad = JitCacheModel(static_argnums=(2,))
for step, i in enumerate(range(0, 100, 32)):
    x = np.ones((min(32, 100 - i), 4), np.float32)
    bad.call(w, x, step)
assert len(bad.keys) == 4  # new step value and a short last batch: a compile every step

good = JitCacheModel()
for step, (x, mask) in enumerate(padded_batches(100, 32, 4)):
    good.call(w, x, mask, np.float32(step))
assert len(good.keys) == 1
assert sum(m.sum() for _, m in padded_batches(100, 32, 4)) == 100  # padding is masked out

try:
    import jax
    import jax.numpy as jnp
except ImportError:
    jax = None

if jax is not None:
    traces = []

    @jax.jit
    def train_step(w, x, y, mask, step):
        traces.append(1)  # Python side effects run only while tracing
        def loss(w):
            err = (x @ w - y) ** 2
            return jnp.sum(err * mask) / jnp.maximum(jnp.sum(mask), 1.0)
        return w - 0.1 / (1.0 + step) * jax.grad(loss)(w)

    wj = jnp.zeros(4)
    for step, (x, mask) in enumerate(padded_batches(100, 32, 4)):
        wj = train_step(wj, x, x.sum(axis=1), mask, jnp.float32(step))
    assert len(traces) == 1
print("q3 ok")
```

Verify: the model and the JAX block both assert one trace across uneven batches; the masked loss equals the unpadded loss on real tokens. In the real job, keep jax_log_compiles on in a smoke test and fail CI if more than one compile appears after step 2.

Pitfalls: Python if on a traced value, which errors or forces static; a changing dict key order in batch; padding without the mask, which biases the loss toward zeros.

Follow-ups: what else enters the cache key (shardings, dtypes, pytree structure, static config)? How would you cache compiles across job restarts?

Why this is likely: a press report on Google's Gemini round and a candidate's recruiter note both describe reading, debugging and optimizing existing code with Gemini, and recompilation is the first JAX performance bug most teams hit. Medium confidence.

</details>

<a id="q4"></a>

#### Q4. (Code comprehension round, Gemini provided) A profile shows the TPU idle 55 percent of each step. Find the host-side bottlenecks in this input loop and fix them without changing which examples the model sees or their order.

Round: Code comprehension (Gemini). AI policy: Provided (Gemini). Type: Optimize. Difficulty: Medium. Topic: Input pipeline, host bound training.

```text
def load_batches(files, batch_size):
    for path in files:
        with open(path, "rb") as f:
            records = parse_records(f.read())          # whole shard in memory
        for i in range(0, len(records), batch_size):
            batch = [decode_and_augment(r) for r in records[i:i + batch_size]]
            yield np.stack(batch)

for step, batch in enumerate(load_batches(files, 256)):
    batch = jax.device_put(batch, data_sharding)
    state, metrics = train_step(state, batch)
    print(step, float(metrics["loss"]))                # every step

# profile, one step: device busy 450 ms, host decode 410 ms, device idle 550 ms
```

<details>
<summary>Show answer</summary>

Task: three serial waits. Decode runs on one thread, the next batch starts only after the device step returns, and float(metrics["loss"]) blocks the host on every step so nothing overlaps.

What they grade: reading the profile numbers, ordered fixes with expected gains, keeping data order deterministic, and verifying Gemini's suggestions.

Approach: First remove the per-step sync: log the loss every 100 steps, or keep metrics on device and fetch them asynchronously. Then decode in a thread or process pool; image decode in C libraries releases the GIL, so threads often suffice, and Python-heavy augmentation needs processes. Then prefetch two batches so decode for step n plus 1 overlaps the device step n. The order stays fixed because the pool map returns results in input order and the prefetch queue is FIFO.

Gemini proposed shuffling records inside the pool for speed. That changes which examples land in each batch and breaks the reproducibility requirement, so I rejected it. It also suggested a prefetch depth of 32; at 256 large examples per batch that is a lot of host memory for no gain once the device is busy, so I kept 2.

Alternatives: tf.data with parallel map and prefetch, or Grain, which gives deterministic, checkpointable iteration. For a long job I would pick Grain because resuming mid-epoch reproduces the same order.

Code:

```python
import queue
import threading
import time
from concurrent.futures import ThreadPoolExecutor

DECODE_S, STEP_S, N = 0.004, 0.010, 12  # per example decode, device step, steps
BATCH = 8


def decode(i):
    time.sleep(DECODE_S)  # stands in for JPEG decode / tokenization (releases the GIL)
    return i


def batches_serial():
    for s in range(N):
        yield [decode(s * BATCH + j) for j in range(BATCH)]


def batches_parallel(pool):
    for s in range(N):
        yield list(pool.map(decode, range(s * BATCH, (s + 1) * BATCH)))


def prefetch(it, depth=2):
    q, done = queue.Queue(maxsize=depth), object()

    def worker():
        for b in it:
            q.put(b)
        q.put(done)

    threading.Thread(target=worker, daemon=True).start()
    while True:
        b = q.get()
        if b is done:
            return
        yield b


def run(it):
    t0, seen = time.perf_counter(), []
    for b in it:
        seen.extend(b)
        time.sleep(STEP_S)  # device step
    return time.perf_counter() - t0, seen


t_serial, s1 = run(batches_serial())
with ThreadPoolExecutor(8) as pool:
    t_fast, s2 = run(prefetch(batches_parallel(pool)))
assert s1 == s2 == list(range(N * BATCH))  # same data, same order
ideal_serial = N * (BATCH * DECODE_S + STEP_S)  # 0.504 s
assert t_serial >= ideal_serial * 0.95
assert t_fast < t_serial * 0.6, (t_fast, t_serial)
print(f"q4 ok serial={t_serial:.3f}s overlapped={t_fast:.3f}s")
```

Verify: the self-test checks identical examples in identical order and at least a 40 percent wall-time cut. On the real job, the device idle fraction in the profile should drop toward zero; if it stays high, the host is CPU bound and needs more workers or offline preprocessing.

Pitfalls: one worker per core plus a framework thread pool oversubscribing the CPU; an unbounded queue; per-step device_get hidden inside a metrics logger.

Follow-ups: how many decode workers do you need if decode is 410 ms per batch and the step is 450 ms?

Why this is likely: the Gemini round tests optimizing existing code, and the job listing asks for triage of hardware and system bottlenecks; input stalls are a standard goodput loss in Google's TPU docs. Medium confidence.

</details>

<a id="q5"></a>

#### Q5. (Code comprehension round, Gemini provided) After switching to bf16 and 8-way gradient accumulation, loss plateaus higher than the fp32 baseline. Find the numeric bugs and explain why loss scaling is or is not the fix.

Round: Code comprehension (Gemini). AI policy: Provided (Gemini). Type: Debug. Difficulty: Hard. Topic: Mixed precision, gradient accumulation.

```text
params = jax.tree.map(lambda p: p.astype(jnp.bfloat16), params)   # only copy kept
acc = jax.tree.map(lambda p: jnp.zeros_like(p), params)            # bf16

for micro in jnp.split(batch, 8):
    # loss_fn returns the mean over the micro-batch's real tokens
    loss, g = jax.value_and_grad(loss_fn)(params, micro)
    acc = jax.tree.map(lambda a, x: a + x, acc, g)                  # bf16 sum

grads = jax.tree.map(lambda a: a / 8, acc)                          # mean of means
params = jax.tree.map(lambda p, g: p - lr * g, params, grads)       # bf16 update

# micro-batches are packed sequences; real token counts range from 37 to 512
```

<details>
<summary>Show answer</summary>

Task: three bugs. The accumulator is bf16, so once it is large, small micro-gradients round away. Dividing by 8 averages per-micro means, so a 37-token micro-batch weighs as much as a 512-token one. The only parameter copy is bf16, so updates smaller than about 1/256 of a weight vanish.

What they grade: naming each bug with a mechanism, a numeric demonstration, and knowing why bf16 differs from fp16.

Approach: Keep fp32 master weights and an fp32 accumulator; cast to bf16 only for the forward and backward matmuls. Accumulate token-weighted sums (gradient of the summed loss) and divide once by the total real tokens. bf16 has 8 bits of mantissa precision and the same exponent range as fp32, so underflow is rare and loss scaling, which is the fix for fp16 gradients, does not help here. The problem is lost precision, and the exponent range is already wide enough.

How I used Gemini: "Simulate this accumulation for 64 micro-gradients of 1e-3 in bf16 and fp32 and show the relative error." It wrote a numpy version that cast with astype(np.float16), which tests the wrong format. I replaced it with explicit bf16 rounding of the float32 bits, which is the code below.

Code:

```python
import numpy as np


def to_bf16(x):
    """Round float32 to bfloat16 precision (round to nearest even), kept as float32."""
    b = np.asarray(x, np.float32).view(np.uint32).astype(np.uint64)
    rounded = (b + 0x7FFF + ((b >> 16) & 1)) & 0xFFFF0000
    return rounded.astype(np.uint32).view(np.float32)


def accumulate(micro_grads, micro_tokens, acc_dtype):
    acc = np.float32(0.0)
    for g, t in zip(micro_grads, micro_tokens):
        acc = acc + np.float32(g) * np.float32(t)  # token-weighted sum
        if acc_dtype == "bf16":
            acc = to_bf16(acc)
    return acc / np.float32(sum(micro_tokens))


def accumulate_buggy(micro_grads, micro_tokens):
    acc = np.float32(0.0)
    for g in micro_grads:
        acc = to_bf16(acc + np.float32(g))
    return acc / len(micro_grads)  # mean of means ignores token counts


assert to_bf16(np.float32(1.0)) == 1.0
assert to_bf16(np.float32(256.0) + np.float32(1.0)) == 256.0  # 8 bit mantissa swallows +1

grads = [1e-3] * 64
tokens = [512] * 63 + [37]
exact = sum(g * t for g, t in zip(grads, tokens)) / sum(tokens)
fp32 = accumulate(grads, tokens, "fp32")
bf16 = accumulate(grads, tokens, "bf16")
assert abs(fp32 - exact) / exact < 1e-5
assert abs(bf16 - exact) / exact > 1e-2  # bf16 accumulator stalls

g2 = [1.0] * 3 + [5.0]
t2 = [512, 512, 512, 8]
weighted = accumulate(g2, t2, "fp32")
assert abs(weighted - (3 * 512 + 40) / 1544) < 1e-6
assert abs(accumulate_buggy(g2, t2) - 2.0) < 1e-6  # short batch gets 64x weight

master = np.float32(1.0)
update = np.float32(1e-4)
w_bf16 = to_bf16(master)
for _ in range(100):
    w_bf16 = to_bf16(w_bf16 - update)
    master = master - update
assert w_bf16 == 1.0  # every update rounds away
assert abs(master - 0.99) < 1e-5
print("q5 ok")
```

Verify: fp32 accumulation matches the exact weighted mean within 1e-5; bf16 accumulation is off by more than 1 percent; 100 updates of 1e-4 leave a bf16 weight of 1.0 unchanged while fp32 master weights move to 0.99.

Pitfalls: fixing averaging but leaving the bf16 accumulator; treating loss scaling as a universal mixed precision fix; computing the softmax or the loss reduction in bf16.

Follow-ups: what does stochastic rounding buy if you must keep bf16 weights? Where would you still keep fp32 inside the model?

Why this is likely: numerics bugs are a core part of the performance work in the listing, and candidate reports describe the ML domain round probing fundamentals with depth. Medium confidence.

</details>

<a id="q6"></a>

#### Q6. (Code comprehension round, Gemini provided) Gemini generated this sharded MoE dispatch for a 16 x 16 TPU mesh. Review it before it lands: find the defects, rank them by impact, and give the fixes.

Round: Code comprehension (Gemini). AI policy: Provided (Gemini). Type: Review. Difficulty: Hard. Topic: MoE routing, sharding annotations.

```text
mesh = Mesh(np.array(jax.devices()).reshape(16, 16), ("data", "expert"))
E, cf = 16, 1.0

def moe_layer(x, router_w, expert_w):          # x: [tokens, d], expert_w: [E, d, f]
    logits = x @ router_w                       # [tokens, E], bf16
    expert = jnp.argmax(logits, axis=-1)        # top-1
    capacity = int(x.shape[0] * cf / E)         # tokens per expert
    buf, pos = dispatch(x, expert, capacity)    # [E, capacity, d]; overflow tokens skipped
    buf = jax.experimental.shard_constraint(buf, P("data", None, None))
    out = jnp.einsum("ecd,edf->ecf", buf, expert_w)
    return combine(out, expert, pos)            # skipped tokens come back as zeros
```

<details>
<summary>Show answer</summary>

Task: four planted defects, ranked by impact.

1. Hallucinated API. jax.experimental.shard_constraint does not exist. The real call is jax.lax.with_sharding_constraint(buf, NamedSharding(mesh, spec)). This raises an AttributeError on the first call, so it is caught fast, but it shows the code was never run.
2. Wrong mesh axis. buf is [E, capacity, d] and its leading dimension is experts, so it must be sharded on "expert", matching expert_w sharded P("expert", None, None). Sharding it on "data" makes XLA gather every expert's weights or reshuffle buffers each layer, which slows every step and raises no error.
3. Floor capacity. int() floors, so 30 tokens over 8 experts gives 3 slots instead of 4 and drops more tokens than the capacity factor promises. Use math.ceil.
4. Silent drops. Overflow tokens come back as zeros with no count. With a residual connection, the layer becomes the identity for them, and quality degrades with no signal. Count drops per expert and log the drop rate; alert above about 1 percent.

Also check: router logits in bf16 then argmax and softmax should be fp32 for stability; there is no load-balancing auxiliary loss, so a collapsed router sends everything to one expert.

Approach: I read the shapes first, then asked Gemini: "List every JAX API this file calls and link to its documentation." It cited a page for shard_constraint that does not exist, which confirmed defect 1. Before I accept a sharding fix, I check the compiled HLO for unexpected all-gathers.

Code:

```python
import math
import numpy as np


def route_top1(logits, num_experts, capacity_factor):
    """Top-1 routing with a per-expert capacity. Returns expert id per token (-1 = dropped)."""
    tokens = logits.shape[0]
    capacity = math.ceil(tokens * capacity_factor / num_experts)  # ceil, not floor
    expert = logits.argmax(axis=1)
    slot = np.full(tokens, -1)
    fill = np.zeros(num_experts, dtype=int)
    for t in range(tokens):  # token order = priority order
        e = expert[t]
        if fill[e] < capacity:
            slot[t] = e
            fill[e] += 1
    dropped = int((slot == -1).sum())
    return slot, capacity, dropped


rng = np.random.default_rng(0)
logits = rng.normal(size=(30, 8))
slot, cap, dropped = route_top1(logits, 8, 1.0)
assert cap == 4  # floor(30/8) would give 3
assert dropped == int((slot == -1).sum())
assert all((slot == e).sum() <= cap for e in range(8))

skewed = np.zeros((16, 4))
skewed[:, 0] = 1.0  # every token wants expert 0
slot, cap, dropped = route_top1(skewed, 4, 1.25)
assert cap == 5 and dropped == 11  # drops must be counted and logged, never silent
assert list(slot[:5]) == [0] * 5 and (slot[5:] == -1).all()

try:
    import jax
    import jax.numpy as jnp
    from jax.sharding import Mesh, NamedSharding, PartitionSpec as P
except ImportError:
    jax = None

if jax is not None:
    mesh = Mesh(np.array(jax.devices()[:1]).reshape(1, 1), ("data", "expert"))

    @jax.jit
    def dispatch(x):
        # [experts, capacity, d]: experts split over the "expert" mesh axis
        return jax.lax.with_sharding_constraint(x, NamedSharding(mesh, P("expert", None, None)))

    out = dispatch(jnp.ones((4, 5, 16)))
    assert out.shape == (4, 5, 16)
print("q6 ok")
```

Verify: capacity uses ceil; a fully skewed router drops exactly the overflow and reports it; the JAX block runs with_sharding_constraint on a small mesh when JAX is installed. On TPU, compare step time and the collective ops in the profile before and after the axis fix.

Pitfalls: trusting that code which compiles is sharded as intended; fixing the API name but keeping the wrong axis.

Why this is likely: the Gemini round grades validating AI output, and the listing names MoE performance work. Medium confidence.

</details>

<a id="q7"></a>

#### Q7. (Code comprehension round, Gemini provided) Add sliding-window attention to this MaxText-style model behind a config flag. The default path must stay bit-identical, and the new path needs a reference test.

Round: Code comprehension (Gemini). AI policy: Provided (Gemini). Type: Build. Difficulty: Medium. Topic: Attention variants, config flags, tests.

```text
model/
  configs/base.yml            # attention: "dot_product"; add sliding_window: 0
  layers/attentions.py        # Attention.__call__: builds causal mask, calls dot_product()
  layers/models.py            # Decoder stacks N blocks; passes config down
  kernels/flash.py            # blocked attention kernel, block_q = block_kv = 512
  tests/attention_test.py     # compares dot_product() with a numpy reference
  tests/train_smoke_test.py   # 20 steps on a tiny config, checks loss decreases
  pyconfig.py                 # validates config keys and types
```

```text
# configs/base.yml (excerpt)
attention: "dot_product"
max_target_length: 2048
# add: sliding_window: 0   (0 = off)

# layers/attentions.py (excerpt)
class Attention(nn.Module):
  config: Config

  def __call__(self, q, k, v):
    # q, k, v: [batch, t, heads, head_dim]
    t = q.shape[1]
    mask = jnp.tril(jnp.ones((t, t), dtype=bool))
    if self.config.attention == "flash":
      return flash.attention(q, k, v, causal=True,
                             block_q=512, block_kv=512)
    return dot_product(q, k, v, mask)

def dot_product(q, k, v, mask):
  scores = jnp.einsum("bqhd,bkhd->bhqk", q, k) / jnp.sqrt(q.shape[-1])
  scores = jnp.where(mask[None, None], scores, -jnp.inf)
  return jnp.einsum("bhqk,bkhd->bqhd", jax.nn.softmax(scores, -1), v)
```

<details>
<summary>Show answer</summary>

Task: add sliding_window to base.yml with 0 meaning off, validate it in pyconfig, mask keys older than the window in the reference path, and skip whole key blocks outside the window in the blocked kernel.

What they grade: a plan before prompting, scoping Gemini to one file at a time, catching an off-by-one, and tests that pin the default path.

Approach: Semantics first, written in the test: with window w, query i sees keys j where i minus w is less than j and j is at most i, so exactly w keys. Prompt 1: "In layers/attentions.py, extend the causal mask with an optional window. Do not change the code path when sliding_window is 0." Prompt 2: "In kernels/flash.py, start the key loop at the first block that intersects the window." Prompt 3: "Add parameterized tests for windows 1, block size minus 1, block size, and longer than the sequence."

Wrong output caught: Gemini's mask used j >= i - w, which lets w plus 1 keys through. The reference test with window 1 caught it, because output must equal v exactly when each query sees only itself.

Alternatives: masking only, which is simple but still pays full quadratic FLOPs, versus block skipping, which makes cost linear in sequence length for a fixed window. I ship both. The mask is the reference and the blocked path is the fast one.

Code:

```python
import numpy as np


def softmax(x, axis=-1):
    x = x - x.max(axis=axis, keepdims=True)
    e = np.exp(x)
    return e / e.sum(axis=axis, keepdims=True)


def reference_attention(q, k, v, window=None):
    """Causal attention; with window w, query i sees keys i-w+1 .. i."""
    t = q.shape[0]
    scores = q @ k.T / np.sqrt(q.shape[1])
    i, j = np.arange(t)[:, None], np.arange(t)[None, :]
    allowed = j <= i
    if window is not None:
        allowed &= j > i - window
    scores = np.where(allowed, scores, -np.inf)
    return softmax(scores) @ v


def sliding_window_attention(q, k, v, window, block=4):
    """Blocked version: each query block reads only the key blocks it can see."""
    t, d = q.shape
    out = np.zeros_like(v)
    for qs in range(0, t, block):
        qe = min(qs + block, t)
        ks = max(0, qs - window + 1)
        s = q[qs:qe] @ k[ks:qe].T / np.sqrt(d)
        i = np.arange(qs, qe)[:, None]
        j = np.arange(ks, qe)[None, :]
        s = np.where((j <= i) & (j > i - window), s, -np.inf)
        out[qs:qe] = softmax(s) @ v[ks:qe]
    return out


def attention(q, k, v, cfg):
    if cfg.get("sliding_window"):
        return sliding_window_attention(q, k, v, cfg["sliding_window"])
    return reference_attention(q, k, v)


rng = np.random.default_rng(0)
q, k, v = (rng.normal(size=(19, 8)) for _ in range(3))
for w in (1, 3, 4, 7, 19, 50):
    assert np.allclose(sliding_window_attention(q, k, v, w), reference_attention(q, k, v, w))
assert np.allclose(attention(q, k, v, {}), reference_attention(q, k, v))  # flag off: unchanged
assert np.allclose(attention(q, k, v, {"sliding_window": 19}), reference_attention(q, k, v))
assert np.allclose(sliding_window_attention(q, k, v, 1), v)  # window 1 = attend to self
print("q7 ok")
```

Verify: the blocked version matches the reference for windows 1, 3, 4, 7, 19 and 50 on a 19-token sequence, which is not a multiple of the block; the flag-off path equals the original; then run the training smoke test with the flag on.

Pitfalls: a window larger than the sequence; decode with a KV cache, where the cache can drop entries older than the window; forgetting to validate negative values.

Why this is likely: the listing asks for writing and testing model code, and the Gemini round includes extending an existing codebase. Medium confidence.

</details>

### TPU performance (ML domain round)

<a id="q8"></a>

#### Q8. (ML domain round, no AI) A bf16 matmul multiplies a [B, 8192] activation by an [8192, 8192] weight on one TPU v5e chip. Is it compute or memory bound at B = 1, 64 and 4096, and at what B does it flip?

Round: ML domain. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Roofline analysis.

<details>
<summary>Show answer</summary>

Task: compute the arithmetic intensity (FLOPs per byte moved from HBM) and compare it with the chip's critical intensity.

What they grade: correct FLOP and byte counts, the critical ratio from hardware specs, and the practical meaning for training versus decode.

Approach: FLOPs are 2 x B x D x F. Bytes in bf16 are 2 x (B x D + D x F + B x F). When B is much smaller than D and F, the weight dominates the bytes, so intensity is close to B. A v5e chip does about 1.97e14 bf16 FLOP/s with about 8.1e11 bytes/s of HBM bandwidth, so the critical intensity is about 243 FLOPs per byte. Below that the chip waits on memory; above it the MXUs are the limit.

Numbers: B = 1 and B = 64 are memory bound; B = 4096 is compute bound. With activations counted too, the flip lands at B = 259, slightly above 243. In practice, going from batch 1 to batch 64 in decode costs almost no extra time because the weight load dominates, which is why serving batches requests.

Code:

```python
PEAK_FLOPS = 1.97e14  # TPU v5e bf16 FLOP/s per chip
HBM_BW = 8.1e11       # TPU v5e HBM bytes/s per chip


def matmul_time(b, d, f, bytes_per=2):
    flops = 2 * b * d * f
    bytes_moved = bytes_per * (b * d + d * f + b * f)
    t_compute, t_memory = flops / PEAK_FLOPS, bytes_moved / HBM_BW
    return max(t_compute, t_memory), flops / bytes_moved, t_compute >= t_memory


critical = PEAK_FLOPS / HBM_BW
assert 240 < critical < 245

for b, expect_compute_bound in [(1, False), (64, False), (128, False), (512, True), (4096, True)]:
    _, intensity, compute_bound = matmul_time(b, 8192, 8192)
    assert compute_bound == expect_compute_bound, (b, intensity)

flip = next(b for b in range(1, 2048) if matmul_time(b, 8192, 8192)[2])
assert 240 <= flip <= 260  # slightly above 243 because activations also move bytes

t1, _, _ = matmul_time(1, 8192, 8192)
t64, _, _ = matmul_time(64, 8192, 8192)
assert t64 / t1 < 1.1  # decode batch 1 to 64 costs almost nothing extra: weight-load bound
print(f"q8 ok critical={critical:.0f} flip={flip}")
```

Verify: the critical ratio sits between 240 and 245; the flip point is found by search rather than assumed; the batch 64 time is within 10 percent of batch 1.

Pitfalls: counting a multiply-add as one FLOP; using per-pod bandwidth with per-chip FLOPs; forgetting that in training B is tokens per chip, so with sequence length 2048 even a per-chip batch of 1 sequence is far above the flip point.

Follow-ups: what changes with int8 weights (bytes halve, so the flip point halves)? How does sharding the weight over 4 chips with tensor parallelism change per-chip intensity and add communication?

Alternatives: measure instead of estimate. A profile gives achieved FLOP/s and bandwidth; the estimate tells you which one to look at.

Why this is likely: the team's own scaling book teaches rooflines first, and the listing centers on TPU performance tuning. High confidence that roofline reasoning comes up in the domain round.

</details>

<a id="q9"></a>

#### Q9. (ML domain round, no AI) An 8B dense model trains on 256 v5e chips with 1M tokens per step and a 2.0 second step. Compute MFU, explain why it differs from hardware FLOP utilization, and name the first three things you would profile.

Round: ML domain. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: MFU, step time math.

<details>
<summary>Show answer</summary>

Task: MFU is model FLOPs per second over peak FLOPs per second. Training FLOPs are about 6 x N x D: 2ND forward and 4ND backward.

What they grade: the 6ND rule and where it comes from, clean arithmetic, and a profiling plan ordered by likely payoff.

Approach: 6 x 8e9 x 1e6 = 4.8e16 FLOPs per step. Peak is 256 x 1.97e14 = 5.04e16 FLOP/s, so an ideal step is 0.952 seconds. At 2.0 seconds, MFU is 47.6 percent. With full rematerialization the hardware also reruns the forward pass, about 8ND, so hardware FLOP utilization is 63.5 percent while MFU stays at 47.6. MFU is the honest number because recompute is overhead you chose.

Numbers: to reach 60 percent MFU the step must drop to 1.59 seconds. The 6ND rule ignores attention FLOPs, which grow with sequence length; at 8K context on an 8B model they add a noticeable share, so state that you are ignoring them.

Profile order: first, exposed collectives (all-gathers and reduce-scatters not overlapped with compute) in the trace; second, remat policy, because saving more activations trades HBM for fewer FLOPs; third, host stalls and input wait between steps. Then fusion quality of the biggest ops.

Code:

```python
PEAK_FLOPS = 1.97e14  # TPU v5e bf16 per chip


def mfu(params, tokens_per_step, chips, step_s, peak=PEAK_FLOPS):
    model_flops = 6 * params * tokens_per_step  # forward 2ND plus backward 4ND
    return model_flops / (chips * peak * step_s)


ideal = 6 * 8e9 * 1e6 / (256 * PEAK_FLOPS)
assert abs(ideal - 0.952) < 0.001
m = mfu(8e9, 1e6, 256, 2.0)
assert abs(m - 0.476) < 0.001

# Rematerialization redoes the forward pass: hardware FLOPs rise, MFU does not.
hfu = 8 * 8e9 * 1e6 / (256 * PEAK_FLOPS * 2.0)
assert hfu > m and abs(hfu - 0.635) < 0.001

# Step needed to hit 60% MFU on the same job
target_step = 6 * 8e9 * 1e6 / (256 * PEAK_FLOPS * 0.60)
assert abs(target_step - 1.586) < 0.001
print(f"q9 ok ideal={ideal:.3f}s mfu={m:.3f} hfu={hfu:.3f}")
```

Verify: the ideal step and MFU match hand math; HFU is above MFU only when remat is on.

Pitfalls: counting tokens per chip instead of global tokens; quoting int8 peak for a bf16 run; reporting HFU as MFU.

Alternatives: tokens per second per chip is easier to compare across runs of one model; MFU compares across model sizes and chips.

Why this is likely: Google's TPU docs present MFU and goodput as the key MaxText metrics, and the team's job is raising them. High confidence that this math is in scope.

</details>

<a id="q10"></a>

#### Q10. (ML domain round, no AI) Plan the parallelism for training a 47B-parameter MoE (8 experts, top-2, about 13B active) on 256 v5e chips with 16 GB HBM each. Choose among data, FSDP, tensor and expert parallelism and show the memory and communication math.

Round: ML domain. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Parallelism, MoE, memory budget.

<details>
<summary>Show answer</summary>

Task: fit the training state, keep communication overlapped, and say what you would measure.

What they grade: a memory budget with bytes per parameter, why each axis is chosen, the all-to-all cost of experts, and a fallback.

Approach: Training state with Adam is about 16 bytes per parameter: bf16 weights and gradients (2 + 2) plus fp32 master weights and two moments (4 + 4 + 4). 46.7e9 x 16 bytes is about 747 GB, so at least 47 chips just to hold state, before activations. On 256 chips I shard all state with FSDP so each chip holds about 2.9 GB, leaving room for activations with remat.

Mesh: 2D, 32 (data and FSDP) by 8 (expert). Expert parallelism places the 8 experts on 8 groups of chips; each token is sent to its top-2 experts with an all-to-all and comes back with another. FSDP shards the dense layers and each expert's weights inside its group. I leave tensor parallelism off at this size because it needs fast collectives on every matmul and pays off mainly when a single layer does not fit.

Numbers: with 4096 tokens per chip and d_model 4096, top-2 dispatch plus combine moves about 128 MiB per chip per MoE layer in the forward pass. Compare that with the expert matmul time per layer; if the all-to-all is not hidden behind compute, increase tokens per chip or overlap dispatch of the next micro-batch.

Code:

```python
import math

HBM_GB = 16  # per TPU v5e chip


def state_gb(total_params, bytes_per_param=16):
    # bf16 params 2 + bf16 grads 2 + fp32 master 4 + Adam m 4 + Adam v 4
    return total_params * bytes_per_param / 1e9


def plan(total_params, chips, act_gb_per_chip):
    per_chip = state_gb(total_params) / chips  # FSDP plus expert sharding spread all state
    return per_chip, per_chip + act_gb_per_chip <= HBM_GB * 0.9  # keep 10% headroom


total = 46.7e9  # 8 experts, top-2, about 12.9e9 active per token
assert abs(state_gb(total) - 747.2) < 0.1
assert math.ceil(state_gb(total) / HBM_GB) == 47  # floor on chips, before activations
per_chip, fits = plan(total, 64, act_gb_per_chip=4.0)
assert abs(per_chip - 11.675) < 0.001 and not fits
per_chip, fits = plan(total, 256, act_gb_per_chip=4.0)
assert abs(per_chip - 2.919) < 0.001 and fits

# All-to-all volume per MoE layer, per chip, forward only: tokens leave and come back.
tokens_per_chip, d_model, top_k = 4096, 4096, 2
a2a_bytes = 2 * tokens_per_chip * top_k * d_model * 2  # dispatch + combine, bf16
assert a2a_bytes == 134_217_728  # 128 MiB per layer per chip
print(f"q10 ok state={state_gb(total):.0f}GB per_chip@256={per_chip:.2f}GB")
```

Alternatives: pure FSDP with experts replicated is simple but all-gathers every expert's weights each layer, much more traffic than moving tokens. Expert parallelism over all 256 chips needs more experts than 8.

Risks: router collapse makes one expert group the bottleneck; use a load-balancing loss and track tokens per expert.

Why this is likely: the listing names MoE performance work, and the scaling book covers sharding and collective costs. Medium confidence.

</details>

<a id="q11"></a>

#### Q11. (ML domain round, no AI) A text-to-image diffusion model takes about 6 seconds per image: 50 sampling steps with classifier-free guidance at 60 ms per forward pass. Where do you cut latency, by how much, and how do you check quality held?

Round: ML domain. AI policy: Not allowed. Type: Optimize. Difficulty: Medium. Topic: Diffusion inference cost.

<details>
<summary>Show answer</summary>

Task: count forward passes, then rank the levers by speedup and quality risk.

What they grade: knowing that guidance doubles the passes, ranked options with numbers, and a quality gate.

Approach: 50 steps with classifier-free guidance run a conditional and an unconditional pass each, so 100 passes x 60 ms plus about 40 ms of encode and decode overhead is about 6.0 seconds.

Levers in order. First, batch the conditional and unconditional passes into one batch-2 call, which keeps the FLOPs and uses the chip better. If the batch-2 call costs 1.6x a single pass, latency drops to about 4.8 seconds with zero quality risk. Second, a faster solver: modern samplers reach similar quality in 20 to 30 steps instead of 50, cutting latency roughly in proportion. Third, step and guidance distillation: a student that runs 4 steps without a separate unconditional pass costs about 280 ms, over 20x faster, but needs training and a careful quality gate. Kernel work (flash attention, fused norms) and sharding the batch over chips stack on top.

Code:

```python
def latency_ms(steps, cfg, pass_ms, batched_cfg=True, overhead_ms=40):
    passes = steps * (2 if cfg else 1)
    if cfg and batched_cfg:
        # cond and uncond in one batch-2 call: same FLOPs, better utilization
        return steps * pass_ms * 1.6 + overhead_ms
    return passes * pass_ms + overhead_ms


base = latency_ms(50, cfg=True, pass_ms=60, batched_cfg=False)
assert base == 6040  # 100 forward passes
batched = latency_ms(50, cfg=True, pass_ms=60)
assert batched == 4840
distilled = latency_ms(4, cfg=False, pass_ms=60)  # step and guidance distilled student
assert distilled == 280
assert base / distilled > 20

# Quality gate: ship only if the metric stays within budget of the teacher.
def gate(teacher_score, student_score, max_drop=0.01):
    return teacher_score - student_score <= max_drop

assert gate(0.320, 0.315) and not gate(0.320, 0.300)
print(f"q11 ok base={base}ms batched={batched}ms distilled={distilled}ms")
```

Numbers: 6040 ms baseline, 4840 ms batched, 280 ms distilled, as computed above.

Alternatives: lower resolution then upscale saves FLOPs but changes the output; caching text embeddings for repeated prompts helps only some traffic.

Risks: distilled students lose diversity and prompt adherence; a single aggregate metric hides that. Gate on an image-text alignment score plus a diversity metric on a fixed prompt set, and run a blind side-by-side human review before shipping.

Verify: assert the pass counts per configuration, and that the gate rejects a 2 point drop.

Why this is likely: the listing names diffusion model performance, and the team maintains MaxDiffusion. Medium confidence.

</details>

### ML infrastructure design (system design round)

<a id="q12"></a>

#### Q12. (System design round, no AI) Design a service that benchmarks 30 model configurations nightly on TPU and GPU and flags performance regressions per commit without flooding the team with false alarms.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Benchmarking, regression detection.

<details>
<summary>Show answer</summary>

Task: scheduler, runners, a results store, a detector, and triage output that points at a commit range.

What they grade: noise handling with numbers, capacity planning for scarce accelerators, and bisection.

Approach: A scheduler reads a config matrix (model, size, chip type, topology) and queues jobs per accelerator pool. Each runner pins the exact wheel and image, warms up, discards compile and the first steps, then records 50 step times, MFU, HBM peak and compile time with full metadata: commit, library versions, chip, topology. Results go to a columnar store keyed by config and commit.

Detection: compare the candidate median with a rolling baseline of the last 10 good runs. Use a robust noise estimate (median absolute deviation) and flag only if the slowdown exceeds max(3 percent, 4 x noise). One outlier step cannot trigger it because medians ignore it. Confirmed flags trigger an automatic rerun, then bisection across the commit range.

Numbers: 30 configs x about 20 minutes each is 10 slice hours per night; with 4 slices per pool that fits in about 2.5 hours. Bisection over 16 commits needs 4 extra runs.

Code:

```python
import statistics


def is_regression(baseline, candidate, min_rel=0.03, k=4.0):
    """Flag if candidate median step time is slower by more than max(min_rel, k*noise)."""
    base_med = statistics.median(baseline)
    mad = statistics.median(abs(x - base_med) for x in baseline)
    noise = 1.4826 * mad / base_med  # MAD to sigma, relative
    delta = (statistics.median(candidate) - base_med) / base_med
    threshold = max(min_rel, k * noise)
    return delta > threshold, round(delta, 4), round(threshold, 4)


base = [1.000, 1.004, 0.998, 1.002, 0.999, 1.001, 1.003, 0.997, 1.000, 1.002]
assert is_regression(base, [1.001, 1.003, 0.999, 1.002, 1.000])[0] is False
assert is_regression(base, [1.050, 1.048, 1.052, 1.049, 1.051])[0] is True
assert is_regression(base, [1.020, 1.021, 1.019, 1.020, 1.022])[0] is False  # under 3% floor
noisy = [1.0, 1.08, 0.93, 1.05, 0.96, 1.02, 0.95, 1.07, 0.98, 1.03]
flag, delta, thr = is_regression(noisy, [1.05, 1.06, 1.04, 1.05, 1.05])
assert flag is False and thr > 0.05  # noisy baseline raises the bar
assert is_regression(base, [1.000, 1.000, 9.0, 1.001, 0.999])[0] is False  # one outlier
print("q12 ok")
```

Alternatives: a fixed percent threshold is simple but either misses real 2 percent regressions on quiet configs or fires daily on noisy ones; change-point detection over the time series catches gradual drift but needs more history. I start with the MAD rule and add change points for drift.

Risks: shared hosts add noise, so pin runners to dedicated slices for the core configs; library upgrades shift every baseline at once, so tag runs with versions and rebaseline explicitly.

Verify: the self-test covers a 5 percent regression, a 2 percent change under the floor, a noisy baseline and a single outlier.

Why this is likely: the listing asks for triaging performance issues across TPU and GPU, and the design round uses capacity math. Medium confidence.

</details>

<a id="q13"></a>

#### Q13. (System design round, no AI) A training job runs for weeks on preemptible TPU slices. A checkpoint takes 60 seconds to write and the slice fails about every 4 hours. Design checkpointing and elastic recovery, and choose the checkpoint interval.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Checkpointing, goodput, fault tolerance.

<details>
<summary>Show answer</summary>

Task: pick the interval with math, cut the blocking cost, and restart fast, possibly on a different slice shape.

What they grade: Young's formula, goodput accounting, async writes, and resharding on restore.

Approach: Young's approximation gives the interval T = sqrt(2 x C x M) with C = 60 seconds and M = 14400 seconds, so about 1314 seconds, about 22 minutes. Lost time per failure is about half an interval of redone work plus restart time. With a 300 second restart, goodput is about 88.8 percent. Hourly checkpoints or every 5 minutes both score lower.

The bigger lever is making the checkpoint cheap. Async checkpointing (Orbax supports it) copies state to host memory, which blocks for seconds, then writes to storage in the background. If the blocking part is 5 seconds, the optimal interval drops to about 380 seconds and goodput rises above 95 percent.

Recovery: a controller detects the failure, gets a new slice, and restores the latest complete checkpoint. Store checkpoints in a sharding-independent layout so a restore can reshard onto a smaller or larger mesh; then rescale the batch or gradient accumulation so the global batch stays the same. Write a commit marker last, and restore only checkpoints that have one. Save the data iterator state too, or the restart sees repeated examples.

Code:

```python
import math


def young_interval(ckpt_cost_s, mtbf_s):
    return math.sqrt(2 * ckpt_cost_s * mtbf_s)


def goodput(interval_s, ckpt_cost_s, mtbf_s, restart_s):
    """First-order fraction of wall time that is useful training."""
    lost = ckpt_cost_s / interval_s + (interval_s / 2 + restart_s) / mtbf_s
    return 1 - lost


t = young_interval(60, 14400)
assert abs(t - 1314.5) < 0.1
g_opt = goodput(t, 60, 14400, 300)
assert abs(g_opt - 0.888) < 0.001
assert goodput(3600, 60, 14400, 300) < g_opt  # hourly checkpoints lose more
assert goodput(300, 60, 14400, 300) < g_opt   # every 5 min pays too much write cost

# Async checkpointing: the blocking part drops from 60 s to 5 s.
t_async = young_interval(5, 14400)
assert abs(t_async - 379.5) < 0.1
assert goodput(t_async, 5, 14400, 300) > 0.95
print(f"q13 ok T={t:.0f}s goodput={g_opt:.3f} async_T={t_async:.0f}s")
```

Alternatives: in-memory peer checkpoints restore fastest but are lost with the slice; storage-only is durable but slow to read. Use both tiers: frequent local or peer copies and less frequent durable copies.

Risks: two writers after a split-brain restart; checkpoints that restore but give different loss because the RNG or data state was not saved.

Why this is likely: Google's TPU docs and the MaxText talk discuss goodput and elastic training for long jobs. Medium confidence.

</details>

<a id="q14"></a>

#### Q14. (System design round, no AI) Design the release pipeline that ships the team's pip wheels and docker images so a user who pins one image gets the exact jax, jaxlib and libtpu that were benchmarked.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Release engineering, reproducibility.

<details>
<summary>Show answer</summary>

Task: hermetic builds, a lock file with hashes, compatibility checks, and promotion only after benchmarks pass.

What they grade: understanding that jaxlib and libtpu must be the versions that jax itself declares, gating on performance as well as tests, and rollback.

Approach: Each release starts from a lock file with exact pins and sha256 hashes. Pin jaxlib and libtpu to the versions that jax's own [tpu] extra resolves to (for jax 0.6.2 that is jaxlib 0.6.2 and libtpu 0.0.17.*; a jax patch release such as 0.4.37 pairs with jaxlib 0.4.36), so a script generates the lock from jax's requirements. CI validates the lock, builds the wheel and an image per accelerator, tags the image by digest, and runs unit tests, a smoke training run on each chip type, and the Q12 benchmark set. Only a candidate within the regression threshold is promoted to the stable tag. The release notes record the version matrix and MFU per config.

Code:

```python
import re

EXPECTED_JAXLIB = {"0.6.2": "0.6.2", "0.4.37": "0.4.36"}


def check_lock(lines):
    """Validate a requirements lock for a release image."""
    errors, pins = [], {}
    for raw in lines:
        line = raw.split("#")[0].strip()
        if not line:
            continue
        m = re.match(r"^([A-Za-z0-9_.\-\[\]]+)==([^\s;\\]+)", line)
        if not m:
            errors.append(f"not pinned exactly: {line}")
            continue
        name = m.group(1).lower().split("[")[0]
        pins[name] = m.group(2)
        if "--hash=sha256:" not in raw:
            errors.append(f"missing hash: {name}")
    for need in ("jax", "jaxlib"):
        if need not in pins:
            errors.append(f"missing {need}")
    if "jax" in pins and "jaxlib" in pins:
        want = EXPECTED_JAXLIB.get(pins["jax"])
        if want is None:
            errors.append(f"jax {pins['jax']} not in the expected map")
        elif pins["jaxlib"] != want:
            errors.append(f"jax {pins['jax']} expects jaxlib {want}, got {pins['jaxlib']}")
    return errors


good = [
    "jax==0.6.2 --hash=sha256:aa",
    "jaxlib==0.6.2 --hash=sha256:bb",
    "libtpu==0.0.17 --hash=sha256:cc  # pinned to the jax release notes",
    "orbax-checkpoint==0.11.0 --hash=sha256:dd",
]
assert check_lock(good) == []
bad = [
    "jax>=0.6",
    "jaxlib==0.6.1 --hash=sha256:bb",
    "libtpu==0.0.17",
]
errs = check_lock(bad)
assert "not pinned exactly: jax>=0.6" in errs
assert "missing hash: libtpu" in errs
assert "missing jax" in errs
mismatch = ["jax==0.6.2 --hash=sha256:a", "jaxlib==0.6.1 --hash=sha256:b"]
assert check_lock(mismatch) == ["jax 0.6.2 expects jaxlib 0.6.2, got 0.6.1"]
patch = ["jax==0.4.37 --hash=sha256:a", "jaxlib==0.4.36 --hash=sha256:b"]
assert check_lock(patch) == []
print("q14 ok")
```

Alternatives: floating ranges (jax>=0.6) make images rebuild differently over time and break benchmark comparability; vendoring every dependency is fully reproducible but slow to patch. Exact pins with hashes plus a weekly automated bump job is the middle path.

Numbers: if the smoke and benchmark stage takes 3 hours on shared slices, cap release candidates at two per day and batch commits.

Risks: an image tag that moves silently breaks users who pinned the tag, so publish immutable digests and keep old ones; a libtpu update can change performance without changing jax, so the benchmark gate must cover it.

Verify: the self-test rejects ranges, missing hashes, missing jax and a jax and jaxlib pair that jax does not declare, and accepts the 0.4.37 and 0.4.36 patch pair.

Pitfalls: testing on CPU only; promoting on unit tests alone.

Why this is likely: the listing names building pip wheels and docker images as preferred experience. Medium confidence.

</details>

### Debugging and evaluation

<a id="q15"></a>

#### Q15. (ML domain round, no AI) After a library upgrade, step time went from 1.0 to 1.5 seconds with the same model and FLOPs. Here are the trace summaries. Find the cause and the fix.

Round: ML domain. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Profiling, collective overlap.

```text
before (ms, chip 0, one step)            after (ms, chip 0, one step)
compute  fusion.1   0    -> 700          compute  fusion.1   0    -> 700
coll     all-gather 100  -> 600          coll     all-gather 700  -> 1200
compute  fusion.2   700  -> 980          compute  fusion.2   1200 -> 1480
step end            1000                 step end            1500
HLO: all-gather-start / all-gather-done  HLO: all-gather (synchronous)
compute total 980 ms, MXU util same      compute total 980 ms, MXU util same
```

<details>
<summary>Show answer</summary>

Task: compute time is unchanged, so the regression is lost overlap. The all-gather now runs after fusion.1 instead of during it.

What they grade: separating compute from exposed communication, reading HLO for async collectives, and a fix plus a guard.

Approach: Union the compute intervals and the collective intervals, then subtract their intersection from collective time. Before: 500 ms of all-gather fully hidden, 0 exposed. After: 500 ms exposed, exactly the regression. The HLO confirms it. The all-gather-start and all-gather-done pairs became a single synchronous all-gather, so XLA's latency hiding scheduler could not move it earlier.

Fix: diff the XLA flags and the HLO dumps (XLA_FLAGS=--xla_dump_to) between versions. Likely causes are a changed default for async collectives, a new dependency that forces the gather after fusion.1 (for example, a sharding change that makes fusion.1 produce the gathered tensor's input), or a scheduler memory limit that now blocks the early start. Restore the flag or the sharding, rerun, and confirm exposed time returns to near zero.

Code:

```python
def union(iv):
    out = []
    for s, e in sorted(iv):
        if out and s <= out[-1][1]:
            out[-1][1] = max(out[-1][1], e)
        else:
            out.append([s, e])
    return out


def length(iv):
    return sum(e - s for s, e in iv)


def intersect(a, b):
    i = j = 0
    out = []
    while i < len(a) and j < len(b):
        s, e = max(a[i][0], b[j][0]), min(a[i][1], b[j][1])
        if s < e:
            out.append([s, e])
        if a[i][1] < b[j][1]:
            i += 1
        else:
            j += 1
    return out


def step_breakdown(ops, step_ms):
    compute = union([(s, e) for kind, s, e in ops if kind == "compute"])
    comm = union([(s, e) for kind, s, e in ops if kind == "collective"])
    exposed = length(comm) - length(intersect(compute, comm))
    idle = step_ms - length(union(compute + comm))
    return {"compute": length(compute), "exposed_comm": exposed, "idle": idle}


# Times in ms for one step on one chip, read from the profiler trace.
good = [("compute", 0, 700), ("collective", 100, 600), ("compute", 700, 980)]
bad = [("compute", 0, 700), ("collective", 700, 1200), ("compute", 1200, 1480)]
g, b = step_breakdown(good, 1000), step_breakdown(bad, 1500)
assert g == {"compute": 980, "exposed_comm": 0, "idle": 20}
assert b == {"compute": 980, "exposed_comm": 500, "idle": 20}
assert b["compute"] == g["compute"]  # same math: the regression is lost overlap
print("q15 ok", g, b)
```

Verify: same compute in both traces, 0 versus 500 ms exposed, idle unchanged.

Pitfalls: looking only at the top op by time; blaming the network when the bytes moved did not change; profiling one chip when a straggler elsewhere sets the pace.

Follow-ups: add exposed collective time as a tracked metric in the Q12 benchmark service so this fails CI next time.

Why this is likely: the listing asks you to triage product or system issues and debug them by analyzing the impact on hardware and network, and the team's scaling book covers collective overlap. No candidate report describes this exact task; low confidence.

</details>

<a id="q16"></a>

#### Q16. (ML domain round, no AI) You replaced the attention kernel with a faster fused one. Design the evaluation that proves training quality did not change before it becomes the default.

Round: ML domain. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Numerics parity, eval design.

<details>
<summary>Show answer</summary>

Task: three layers of evidence: op-level numerics, training curve parity, and downstream eval parity with stated tolerances.

What they grade: tolerances justified by dtype, paired statistics, and knowing that bitwise equality is the wrong bar.

Approach: Layer 1, op level: run old and new kernels on the same random and real inputs, including long sequences, padding masks and extreme logits. Compare against an fp32 reference, not against each other; the new kernel passes if its error is no worse than the old kernel's error. Bitwise equality is impossible because fused kernels reorder sums.

Layer 2, training: train the same seed and data for a few thousand steps both ways. Compare moving averages of loss after warmup; pass if the relative gap stays within 1 percent. Also compare gradient norms, which drift earlier than loss.

Layer 3, evals: run the same checkpoint step from both runs on a fixed eval set, scoring item by item. Use a paired test because the same items are scored twice: pass if the 95 percent lower bound of new minus old accuracy is above minus 0.5 points.

Code:

```python
import math
import random


def loss_curve_parity(base, cand, warmup=100, rel_tol=0.01, window=50):
    """Compare moving averages of training loss after warmup."""
    def ma(xs, i):
        seg = xs[max(0, i - window + 1):i + 1]
        return sum(seg) / len(seg)

    worst = max(abs(ma(cand, i) - ma(base, i)) / ma(base, i) for i in range(warmup, len(base)))
    return worst <= rel_tol, worst


def accuracy_parity(base_correct, cand_correct, margin=0.005, z=1.96):
    """Paired check on the same eval items: pass if the lower bound of the
    candidate minus baseline difference stays above -margin."""
    n = len(base_correct)
    diffs = [c - b for b, c in zip(base_correct, cand_correct)]
    mean = sum(diffs) / n
    var = sum((d - mean) ** 2 for d in diffs) / (n - 1)
    lower = mean - z * math.sqrt(var / n)
    return lower >= -margin, round(mean, 4), round(lower, 4)


rng = random.Random(0)
base = [3.0 * math.exp(-i / 400) + 1.5 + rng.gauss(0, 0.02) for i in range(1000)]
same = [x + rng.gauss(0, 0.02) for x in base]
drift = [x * (1 + 0.03 * i / 1000) for i, x in enumerate(base)]
assert loss_curve_parity(base, same)[0] is True
assert loss_curve_parity(base, drift)[0] is False

items = 20000
b = [1 if rng.random() < 0.70 else 0 for _ in range(items)]
c_same = [(0 if rng.random() < 0.01 else 1) if x else (1 if rng.random() < 0.0233 else 0) for x in b]  # balanced flips
c_worse = [0 if (x == 1 and rng.random() < 0.03) else x for x in b]
assert accuracy_parity(b, c_same)[0] is True
assert accuracy_parity(b, c_worse)[0] is False  # about 2 points lost on hard data
print("q16 ok")
```

Numbers: with 20000 items the paired standard error is well under 0.5 points, which a single unpaired comparison would not give.

Alternatives: run seed sweeps to measure natural run-to-run variance first; if two seeds of the old kernel differ by 1 percent in loss, a 1 percent tolerance is meaningless.

Risks: parity on short runs hides divergence that appears late; schedule one long run before deprecating the old kernel.

Verify: the self-test passes a noise-only change and fails a slow 3 percent drift and a 2 point accuracy loss.

Why this is likely: the team's optimizations must keep model quality, and the listing's minimum qualifications ask for 3 years of experience with end-to-end machine learning, including model evaluation. No candidate report describes this question; low confidence.

</details>

### Behavioral

<a id="q17"></a>

#### Q17. (Googleyness and Leadership, no AI) Tell me about a time an AI tool suggested a performance change that looked right but was wrong. How did you find out, and what did you change in how your team uses AI?

Round: Googleyness and Leadership. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI use, verification, team practice.

<details>
<summary>Show answer</summary>

Task: one specific story in situation, action, result form, with a number and a process change.

What they grade: ownership, evidence over intuition, how you check AI output, and influence on the team without blame.

Approach: Example shape. Situation: an AI assistant suggested raising the prefetch depth and adding donate_argnums to the train step to fix a step-time regression, and a teammate had already opened the change. Action: before approving, I asked for a profile. The trace showed the device was compute bound with zero input wait, so prefetch could not help, and donation only saves memory. The real cause was a sharding change that exposed an all-gather. I wrote that up with both traces side by side, kept the write-up about the traces, and we reverted the sharding change. Result: step time returned to baseline the same day, and we avoided a change that would have added host memory pressure for nothing.

What changed: we added a rule to our review template that any performance change cites a before and after profile, and we added exposed collective time to our nightly benchmark. AI suggestions are still welcome; they now come with a measurement.

Verify: have the numbers ready (step time before and after, how long it took) and be ready for "what would you do if the teammate disagreed?"

Pitfalls: a story where the AI is simply bad and you are the hero; no number; no lasting change. Interviewers want to hear that you still use AI and verify it.

Why this is likely: a press report says Google now treats AI fluency as a requirement for all software engineering jobs, and the listing prefers experience with AI productivity tools. Medium confidence on this exact framing.

</details>

## Prep checklist

- [ ] Solve binary search on the answer and interval merge problems on a whiteboard in 25 minutes, saying complexity out loud.
- [ ] Practise one medium problem a day in a plain editor that cannot run code.
- [ ] Do one timed read, debug and optimize session on an unfamiliar JAX repo with an AI assistant, and write down every claim you checked.
- [ ] Read the rooflines and sharding chapters of the scaling book and redo the critical intensity math for one TPU generation.
- [ ] Compute MFU for a model you know with 6ND and explain the gap to hardware FLOP utilization.
- [ ] Profile a small JAX training loop: find compile count, input wait and exposed collectives in the trace viewer.
- [ ] Write a bf16 gradient accumulation bug and its fix from memory, including token weighting.
- [ ] Sketch an MoE mesh with expert and FSDP axes and its all-to-all bytes per layer.
- [ ] Prepare one design each for benchmarking with regression detection and for checkpointing with elastic restore, with numbers.
- [ ] Prepare four Googleyness stories, one about checking AI output with data.
- [ ] Ask your recruiter whether your loop includes the Gemini code comprehension round and which rounds are in person.

## Sources

- [Google careers: Senior Software Engineer, AI/ML, ML Training](https://www.google.com/about/careers/applications/jobs/results/100216581069382342-senior-software-engineer-aiml-ml-training): official job listing, team and qualifications.
- [Google careers: how we hire](https://www.google.com/about/careers/applications/how-we-hire/): official hiring process overview.
- [Google careers: interview tips](https://www.google.com/about/careers/applications/interview-tips/): official interview preparation page.
- [Business Insider: Google's Gemini code comprehension interview](https://www.businessinsider.com/google-interview-process-ai-gemini-coding-comprehension-software-engineer-2026-10): press report on the AI-assisted round and AI fluency requirements.
- [Blind: Google AI-assisted coding interview, recent experiences](https://www.teamblind.com/post/google-ai-assisted-coding-interview-recent-experiences-24dvwz4z): candidate thread on the Gemini round.
- [Hello Interview: Google L5 guide](https://www.hellointerview.com/guides/google/l5): loop format, phone screen and onsite details.
- [1point3acres: Google interview trend report](https://www.1point3acres.com/interview/guides/google-interview-trend-2026q2): aggregated candidate reports.
- [1point3acres: Google AI infra SDE loop thread](https://www.1point3acres.com/bbs/thread-1161287-1-1.html): first-hand TPU infra loop report.
- [fastprep: Google SWE AI/ML coding and ML domain interviews](https://www.fastprep.io/experience/google-2026-new-grad-swe-ai-ml-coding-and-ml-domain-interviews-tngfafg5sk): first-hand AI/ML track report.
- [Glassdoor: Google Software Engineer, Machine Learning interviews](https://www.glassdoor.com/Interview/Google-Software-Engineer-Machine-Learning-Interview-Questions-EI_IE9079.0,6_KO7,41.htm): older candidate reviews.
- [Computerworld: companies bring back in-person interviews](https://www.computerworld.com/article/4044734/to-counter-ai-cheating-companies-bring-back-in-person-job-interviews.html): press report on Google's in-person rounds.
- [How to Scale Your Model (scaling book)](https://jax-ml.github.io/scaling-book/): Google DeepMind book on rooflines, sharding and TPU performance.
- [MaxText on GitHub](https://github.com/AI-Hypercomputer/maxtext): the team's open-source JAX LLM implementation.
- [Google Cloud: JAX AI stack on TPU](https://docs.cloud.google.com/tpu/docs/jax-ai-stack): official docs on MaxText, MFU and goodput.
- [MaxText 2.0 talk](https://www.youtube.com/watch?v=SPTcZhUBVz8): Google talk on MaxText features including elastic training.
