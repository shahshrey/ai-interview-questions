# NVIDIA: Senior AI Research Engineer, Agentic Systems Intelligence

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

NVIDIA publishes the frame of its process (phone and video rounds, then an in-person onsite at an NVIDIA office before any offer, interviews of 30 to 60 minutes, coding usually in HackerRank on a whiteboard or an NVIDIA laptop), and each hiring manager writes the rounds inside it. This team builds agents that write and tune CUDA code: agent architectures, multi-agent workflows, coding agents, RL environments, verifiers and rewards, and evaluation frameworks, co-designed down to kernels and GPUs. Research loops at NVIDIA center on a research talk plus researcher interviews; 2026 deep learning engineer loops add LeetCode-style coding and a distributed training deep dive. This role sits between the two. No first-hand report from this exact team was found, so treat the shape below as an estimate and confirm it with the recruiter.

1. Recruiter screen. About 30 minutes by phone or video: background, level and which areas the loop will cover. AI: not relevant.
2. Hiring manager screen. 30 to 60 minutes by video. A deep dive into your projects (coding agents, RL or evaluation work you owned and how you measured it), often with technical questions; a 2026 deep learning engineer report says the first round was distributed training plus past projects. AI: not allowed.
3. Technical screen. 45 to 60 minutes, live, in Python or C++. A LeetCode easy or medium problem or a practical task; interviewing.io says LeetCode practice helps but practical questions come too. AI: not allowed.
4. Research talk or project presentation. About 60 minutes to a mixed panel of researchers and engineers, then questions on choices, ablations and next steps. For a research engineer this may be a project walkthrough instead of a formal talk. AI: not allowed.
5. Onsite coding. 45 to 60 minutes. 2026 deep learning engineer reports still describe LeetCode-style rounds ("Mostly leetcode related interview questions"), and a 2026 Blind invite listed rounds named "llm problem solving" and "bug bash". Expect one algorithm problem plus practical code, including reviewing code you did not write. AI: not allowed.
6. CUDA and GPU performance. 45 to 60 minutes with a senior engineer: write or fix a reduction, softmax or tiled kernel, reason with a roofline, and explain how you would profile with Nsight Compute. AI: not allowed.
7. ML and agent systems deep dive. 45 to 60 minutes with researchers: transformer internals (a 2025 report asked to implement multi-head self-attention), RL for agents (verifiable rewards, GRPO, reward hacking), benchmark design and inference-time compute. AI: not allowed.
8. System design. 45 to 60 minutes. Likely prompt: an agent or RL environment that generates CUDA kernels, verifies them, measures speedup on real GPUs and feeds rewards back, at cluster scale, or how to serve the agents on vLLM, SGLang or TensorRT-LLM. AI: not allowed.
9. Behavioral and Insider Chat. 30 to 45 minutes with the manager or a skip level on ownership from prototype to product and work with hardware and model teams. The optional Insider Chat with a community resource group member does not decide the outcome. AI: not allowed.
10. Decision. NVIDIA says it will get back to you "in a few short weeks". AI: not relevant.

AI tools: NVIDIA's How We Hire page says "The use of unapproved outside tools, such as ChatGPT, during the interview will result in the disqualification of candidacy." interviewing.io (2026) says AI use in NVIDIA interviews is strictly prohibited, and a reply in a 2026 Blind thread said the "llm problem solving" and "bug bash" rounds were a "technical round without AI assistance". No source shows an AI-assisted round at NVIDIA. Every question below assumes no AI, as at NVIDIA: you write, run in your head and check all the code yourself.

Classic algorithm questions are limited to two (Q1 and Q2), because 2026 Glassdoor reports for NVIDIA deep learning engineers still describe LeetCode rounds, while research loops lean on the talk and domain depth.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a byte-bounded LRU cache for compiled kernels with O(1) get and put | Classic algorithm | Technical screen | Not allowed | Medium |
| [Q2](#q2) | Merge sorted result streams from many GPU workers and return the K fastest correct kernels | Classic algorithm | Onsite coding | Not allowed | Medium |
| [Q3](#q3) | Implement the unbiased pass@k estimator and score a code generation run | Build | Technical screen | Not allowed | Medium |
| [Q4](#q4) | A coding assistant wrote this kernel verifier and reward function. Review it before RL uses it | Review | Onsite coding | Not allowed | Hard |
| [Q5](#q5) | This CUDA block reduction gives a wrong sum that changes from run to run. Find and fix the bugs | Debug | CUDA and GPU performance | Not allowed | Medium |
| [Q6](#q6) | Write a fused row softmax that reads each row once, with one warp per row | Build | CUDA and GPU performance | Not allowed | Hard |
| [Q7](#q7) | Use a roofline to decide whether three kernels are memory or compute bound, then fix the slow one | Optimize | CUDA and GPU performance | Not allowed | Medium |
| [Q8](#q8) | Implement causal multi-head self-attention from scratch and prove the mask works | Concept plus code | ML and agent systems deep dive | Not allowed | Medium |
| [Q9](#q9) | Reward climbs during kernel RL but held-out correctness does not. Read the trace and find the cause | Trace debug | ML and agent systems deep dive | Not allowed | Hard |
| [Q10](#q10) | Design a tiered reward for kernel RL and compute GRPO advantages for a group of rollouts | Concept plus code | ML and agent systems deep dive | Not allowed | Medium |
| [Q11](#q11) | Design a benchmark for CUDA code generation agents that still means something next year | Design | ML and agent systems deep dive | Not allowed | Hard |
| [Q12](#q12) | Spend a fixed inference budget across 100 kernel problems: best-of-N or sequential refinement | Concept plus code | ML and agent systems deep dive | Not allowed | Medium |
| [Q13](#q13) | Design the closed-loop agent that generates, verifies, profiles and refines CUDA kernels | Design | System design | Not allowed | Hard |
| [Q14](#q14) | Design the RL environment and sandbox that run a million coding agent rollouts safely | Design | System design | Not allowed | Hard |
| [Q15](#q15) | Size the KV cache and pick a serving engine for eight agents sharing a long prompt on a 70B model | Design | System design | Not allowed | Medium |
| [Q16](#q16) | Design a multi-agent workflow for CUDA work, and decide when one agent is better | Design | System design | Not allowed | Medium |
| [Q17](#q17) | Explain how you would train a 70B model across 64 GPUs, with the memory and communication math | Concept plus code | Hiring manager screen | Not allowed | Medium |
| [Q18](#q18) | Walk us through a result that looked better than it was, and how you check AI-written code | Behavioral | Research talk or project presentation | Not allowed | Medium |

### Coding rounds (no AI, as at NVIDIA)

<a id="q1"></a>

#### Q1. Build a byte-bounded LRU cache for compiled kernels with O(1) get and put

Round: Technical screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Hash map plus doubly linked list, eviction by size.
Related practice: [LRU Cache](https://leetcode.com/problems/lru-cache/) (Medium)

<details>
<summary>Show answer</summary>

Task: a kernel agent recompiles the same candidates many times. Cache compiled binaries keyed by `(source_hash, sm_arch, flags)`. `get(key)` returns the binary or None and marks it recently used. `put(key, binary)` stores it and evicts least recently used entries until total bytes fit in `max_bytes`. Both calls must be O(1) on average. Do it without `OrderedDict`.

What they grade: the hash map plus doubly linked list design, correct pointer updates, the size budget (not an entry count), and the edge cases.

Approach: a dict maps each key to a node; nodes sit in a doubly linked list with two sentinels, most recent next to `head`. `get` unlinks the node and puts it at the front. `put` on an existing key replaces the value and fixes the byte count; a new key goes to the front; then evict from the tail until `used <= max_bytes`. An item larger than the whole budget is refused, so one oversized binary cannot flush the cache. Sentinels remove every special case for an empty or one-node list.

Code:

```python
class Node:
    __slots__ = ("key", "val", "size", "prev", "next")
    def __init__(self, key=None, val=None, size=0):
        self.key, self.val, self.size = key, val, size
        self.prev = self.next = None

class KernelCache:
    def __init__(self, max_bytes):
        self.max_bytes, self.used, self.map = max_bytes, 0, {}
        self.head, self.tail = Node(), Node()
        self.head.next, self.tail.prev = self.tail, self.head
        self.hits = self.misses = 0

    def _unlink(self, n):
        n.prev.next, n.next.prev = n.next, n.prev

    def _push_front(self, n):
        n.prev, n.next = self.head, self.head.next
        self.head.next.prev = n
        self.head.next = n

    def get(self, key):
        n = self.map.get(key)
        if n is None:
            self.misses += 1
            return None
        self._unlink(n)
        self._push_front(n)
        self.hits += 1
        return n.val

    def put(self, key, binary):
        size = len(binary)
        if size > self.max_bytes:
            return False
        old = self.map.pop(key, None)
        if old is not None:
            self._unlink(old)
            self.used -= old.size
        n = Node(key, binary, size)
        self.map[key] = n
        self._push_front(n)
        self.used += size
        while self.used > self.max_bytes:
            lru = self.tail.prev
            self._unlink(lru)
            del self.map[lru.key]
            self.used -= lru.size
        return True

c = KernelCache(max_bytes=10)
k = lambda h: (h, "sm_90", "-O3")
assert c.put(k("a"), b"aaaa") and c.put(k("b"), b"bbbb")
assert c.get(k("a")) == b"aaaa"
c.put(k("c"), b"cccc")
assert c.get(k("b")) is None
assert c.get(k("a")) == b"aaaa" and c.get(k("c")) == b"cccc"
c.put(k("a"), b"aaaaaa")
assert c.used == 10 and c.get(k("c")) == b"cccc"
assert not c.put(k("huge"), b"x" * 11) and c.used == 10
assert c.get(k("a")) == b"aaaaaa"
c.put(k("d"), b"dddddddddd")
assert list(c.map) == [k("d")] and c.used == 10
assert (c.hits, c.misses) == (5, 1)
```

Verify: the tests cover recency changing the eviction order, replacing a key with a bigger value, refusing an item larger than the budget, and one big put that evicts several entries. `used` must always equal the sum of sizes in the map.

Pitfalls:
- Forgetting to subtract the old size on replace, so `used` drifts and the cache evicts too early.
- Evicting before inserting, which can evict the key you are about to replace.
- Including the full compiler command line in the key but not the GPU architecture, so an sm_80 binary is served on sm_90.

Follow-ups: thread safety (one lock around both structures, or shard by key hash); a disk tier with the same keys; why LRU can be wrong here (a scan of one-off candidates flushes hot kernels, which LFU or a small admission filter avoids).

Why this is likely: LRU Cache is the top entry in the crowd-reported NVIDIA LeetCode list for the last three months, and 2026 Glassdoor reports for NVIDIA deep learning engineers describe LeetCode-style rounds; the kernel-cache framing is an illustration, medium confidence in the problem, low in the framing.

</details>

<a id="q2"></a>

#### Q2. Merge sorted result streams from many GPU workers and return the K fastest correct kernels

Round: Onsite coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: K-way merge with a heap, deduplication, early stop.
Related practice: [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) (Hard)

<details>
<summary>Show answer</summary>

Task: S benchmark workers each return a list of results sorted by latency, ascending: `(latency_us, kernel_hash, correct)`. The same kernel can appear on several workers. Return the K fastest distinct kernels that passed correctness, in order, without concatenating and sorting everything.

What they grade: the heap-based merge, stopping as soon as K are found, handling duplicates and failed kernels, and the complexity argument.

Approach: push the head of each stream into a min-heap keyed by latency. Pop the smallest, skip it if it failed or if its hash was already taken, otherwise take it. Push the next element from the same stream. Stop at K results or when the heap is empty. Each pop costs O(log S), and you touch only the elements up to the K-th winner plus skipped ones, so the cost is O((K + skipped) log S) rather than O(N log N) for a full sort. Ties break on the stream index so the heap never compares unrelated payloads.

Code:

```python
import heapq

def k_fastest(streams, k):
    heap = [(s[0][0], i, 0) for i, s in enumerate(streams) if s]
    heapq.heapify(heap)
    seen, out = set(), []
    while heap and len(out) < k:
        lat, i, j = heapq.heappop(heap)
        _, h, ok = streams[i][j]
        if ok and h not in seen:
            seen.add(h)
            out.append((lat, h))
        if j + 1 < len(streams[i]):
            heapq.heappush(heap, (streams[i][j + 1][0], i, j + 1))
    return out

w0 = [(10.0, "a", True), (12.5, "b", False), (20.0, "c", True)]
w1 = [(9.0, "x", False), (11.0, "a", True), (13.0, "d", True)]
w2 = [(12.0, "e", True), (30.0, "f", True)]
assert k_fastest([w0, w1, w2], 3) == [(10.0, "a"), (12.0, "e"), (13.0, "d")]
assert k_fastest([w0, w1, w2], 10) == [(10.0, "a"), (12.0, "e"), (13.0, "d"), (20.0, "c"), (30.0, "f")]
assert k_fastest([[], w2], 1) == [(12.0, "e")]
assert k_fastest([], 5) == [] and k_fastest([w0], 0) == []

import random
rng = random.Random(0)
streams = [sorted((rng.uniform(1, 100), f"k{rng.randrange(40)}", rng.random() > 0.3) for _ in range(50)) for _ in range(8)]
flat = sorted(x for s in streams for x in s)
best, seen = [], set()
for lat, h, ok in flat:
    if ok and h not in seen:
        seen.add(h)
        best.append((lat, h))
assert k_fastest(streams, 7) == best[:7]
```

Verify: a hand case with a failed kernel at the global minimum, a duplicate hash on two workers, an empty stream, K of 0, K larger than the answer, and a random case checked against a full sort.

Pitfalls:
- Deduplicating before the correctness filter, so a failed copy of a kernel hides a passing copy.
- Pushing all N elements into the heap first, which throws away the early stop.
- Heap entries that hold tuples with unorderable payloads, which crash on equal latencies.

Follow-ups: if streams arrive over the network, keep one outstanding request per worker; Top K Frequent Elements is the variant when you must rank kernels by how often they win across problems; with timing noise, rank by the median of repeated runs.

Why this is likely: Merge k Sorted Lists and Top K Frequent Elements both appear in the crowd-reported NVIDIA LeetCode list for the last three months, and 2026 Glassdoor reports describe a second round made only of LeetCode problems; medium confidence in the pattern, the benchmark framing is illustrative.

</details>

<a id="q3"></a>

#### Q3. Implement the unbiased pass@k estimator and score a code generation run

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: pass@k, numerical stability, per-problem aggregation.

<details>
<summary>Show answer</summary>

Task: for each problem you sampled n kernels and c of them passed. Write `pass_at_k(n, c, k)` and `score(results, k)` that averages it over problems. Explain why 1 - (1 - c/n)^k is the wrong formula.

What they grade: the right estimator, a form that does not overflow, edge cases (c = 0, n - c < k), and the reason the naive version is biased.

Approach: pass@k is the chance that at least one of k samples drawn without replacement from the n is correct: 1 - C(n - c, k) / C(n, k). The binomial coefficients get huge for n = 200, so use the product form 1 - prod over i from n - c + 1 to n of (1 - k / i), which stays in float range. If n - c < k, every draw of k includes a correct sample, so the answer is 1. The naive formula treats the n samples as an exact success rate and samples with replacement, which underestimates pass@k: for n = 10, c = 1, k = 5 it gives 0.41 against the true 0.5. ComputeEval, NVIDIA's CUDA benchmark, reports pass@1 with this estimator, where it reduces to the mean of c / n.

Code:

```python
import math

def pass_at_k(n, c, k):
    if not 0 <= c <= n or not 1 <= k <= n:
        raise ValueError("need 0 <= c <= n and 1 <= k <= n")
    if n - c < k:
        return 1.0
    return 1.0 - math.prod(1.0 - k / i for i in range(n - c + 1, n + 1))

def score(results, k):
    vals = [pass_at_k(n, c, k) for n, c in results]
    return sum(vals) / len(vals)

def exact(n, c, k):
    return 1.0 - math.comb(n - c, k) / math.comb(n, k)

for n, c, k in [(10, 1, 5), (20, 3, 1), (200, 7, 10), (5, 0, 3), (5, 4, 2)]:
    assert abs(pass_at_k(n, c, k) - exact(n, c, k)) < 1e-12
assert pass_at_k(10, 1, 5) == 0.5
assert abs((1 - (1 - 1 / 10) ** 5) - 0.40951) < 1e-9
assert abs(pass_at_k(20, 3, 1) - 3 / 20) < 1e-12
assert pass_at_k(5, 0, 3) == 0.0 and pass_at_k(5, 3, 3) == 1.0
assert 0 < pass_at_k(10000, 1, 100) < 1
assert abs(score([(10, 1), (10, 10), (10, 0)], 1) - 11 / 30) < 1e-12
try:
    pass_at_k(5, 6, 1)
    raise AssertionError("c > n must fail")
except ValueError:
    pass
```

Verify: the product form matches the exact binomial form on small cases, the n = 10, c = 1, k = 5 case shows the naive bias, and n = 10,000 runs without overflow.

Pitfalls:
- Computing `comb(n, k)` as floats for large n, which overflows or loses precision.
- Averaging the pooled c and n across problems instead of averaging per-problem pass@k.
- Reporting pass@10 from only 10 samples, which has high variance; sample more than k.

Follow-ups: the confidence interval on the mean (see Q11); why pass@k with large k rewards diversity and can mislead when a verifier picks the final answer anyway; how to report pass@1 when samples cost minutes of GPU time each.

Why this is likely: NVIDIA's ComputeEval benchmark for CUDA code reports pass@1 per model and the listing asks for evaluation frameworks; implementing the metric is a common practical screen for eval work; medium confidence.

</details>

<a id="q4"></a>

#### Q4. A coding assistant wrote this kernel verifier and reward function. Review it before RL uses it

Round: Onsite coding. AI policy: Not allowed. Type: Review. Difficulty: Hard. Topic: Verifier correctness, timing, reward hacking, process safety.

<details>
<summary>Show answer</summary>

Task: a teammate generated this with an assistant and wants to plug it into a kernel RL run today. `run_fn(src, x)` compiles and runs the candidate on the GPU. Find the defects, rank them, and write the fix.

```python
import subprocess, time
import numpy as np

def compile_kernel(src_path, name):
    subprocess.run(f"nvcc -O3 -o /tmp/{name} {src_path}", shell=True)

def check_kernel(src, ref_fn, make_inputs, run_fn):
    x = make_inputs(1024)
    expected = ref_fn(x)
    out = run_fn(src, x)
    np.testing.assert_close(out, expected, atol=1e-1)
    return True

def time_kernel(src, x, run_fn, iters=10):
    start = time.time()
    for _ in range(iters):
        run_fn(src, x)
    return (time.time() - start) / iters

def reward(src, ref_fn, make_inputs, run_fn, ref_time):
    try:
        ok = check_kernel(src, ref_fn, make_inputs, run_fn)
    except Exception:
        ok = False
    speedup = ref_time / time_kernel(src, make_inputs(1024), run_fn)
    return speedup if ok else 0.5 * speedup
```

What they grade: finding the defects that corrupt the reward first, a fix you can test without a GPU, and how you would catch this class of bug next time.

Approach: rank by damage to training. (1) `np.testing.assert_close` does not exist in NumPy (it is `assert_allclose`; `assert_close` is PyTorch), so every call raises AttributeError, the broad `except Exception` swallows it, and every kernel is "incorrect". (2) Incorrect kernels still get half the speedup, so a fast wrong kernel beats a slow correct one: reward hacking by design. (3) One shape, 1024, a power of two, never tests tail handling; the same inputs every call let a kernel that caches its last output pass. (4) `atol=1e-1` with no `rtol` passes badly wrong results for small values. (5) Timing has no warmup and no device sync, so it measures launch overhead, and `time.time` is too coarse; use events or `perf_counter` after a sync, take the median of many runs. (6) `shell=True` with an unvalidated name is command injection, the return code is ignored, and `/tmp/{name}` collides across workers.

Code:

```python
import statistics, time
import numpy as np

def check_kernel(src, ref_fn, make_inputs, run_fn, sizes=(1, 31, 1000, 4097), seeds=(0, 1)):
    for n in sizes:
        for seed in seeds:
            x = make_inputs(n, seed)
            exp = ref_fn(x)
            out = run_fn(src, x)
            if out.shape != exp.shape or not np.all(np.isfinite(out)):
                return False
            if not np.allclose(out, exp, rtol=1e-3, atol=1e-5):
                return False
    return True

def time_kernel(src, x, run_fn, sync=lambda: None, warmup=3, iters=20):
    for _ in range(warmup):
        run_fn(src, x)
    sync()
    times = []
    for _ in range(iters):
        t0 = time.perf_counter()
        run_fn(src, x)
        sync()
        times.append(time.perf_counter() - t0)
    return statistics.median(times)

def reward(src, ref_fn, make_inputs, run_fn, ref_time, sync=lambda: None):
    if not check_kernel(src, ref_fn, make_inputs, run_fn):
        return 0.0
    t = time_kernel(src, make_inputs(4097, 7), run_fn, sync)
    return 1.0 + max(0.0, min(ref_time / t, 10.0) - 1.0)

def make_inputs(n, seed):
    return np.random.default_rng(seed).standard_normal(n).astype(np.float32)

ref = lambda x: np.cumsum(x, dtype=np.float64).astype(np.float32)
good = lambda src, x: np.cumsum(x, dtype=np.float64).astype(np.float32)
wrong = lambda src, x: np.cumsum(x[:1024], dtype=np.float64).astype(np.float32)
nan = lambda src, x: np.full_like(x, np.nan)
cache = {}
def stale(src, x):
    cache.setdefault("out", ref(x))
    return cache["out"]

assert check_kernel("g", ref, make_inputs, good)
assert not check_kernel("w", ref, make_inputs, wrong)
assert not check_kernel("n", ref, make_inputs, nan)
assert not check_kernel("s", ref, make_inputs, stale)
assert reward("w", ref, make_inputs, wrong, ref_time=1.0) == 0.0
assert 1.0 <= reward("g", ref, make_inputs, good, ref_time=1e-9) <= 10.0
assert not hasattr(np.testing, "assert_close")
```

The fixed compile step is `subprocess.run(["nvcc", "-O3", "-o", out_path, src_path], check=True, timeout=120)` with `out_path` in a per-worker temp dir, run inside the sandbox from Q14.

Verify: the tests show a wrong-tail kernel, a NaN kernel and a stale-cache kernel all get zero reward, and the speedup is capped at 10 so a timing glitch cannot dominate a batch.

Pitfalls:
- Fixing the API name but keeping the broad `except Exception`, which hides the next bug the same way.
- Trusting the speedup when the kernel returned early on a shape it did not handle.

Follow-ups: a test that fails the old code (run it against a known-wrong kernel and assert reward 0); why the cap is 10 and not unbounded; how to add a reference run on fresh buffers (see Q9).

Why this is likely: a 2026 Blind thread lists an NVIDIA "bug bash" round, NVIDIA's kernel generation blog puts a verifier at the center of its agent loop, and its RL blog stresses verifier rewards; and code review and bug-bash style rounds sit alongside algorithm rounds in 2026 practical loops; medium confidence.

</details>

### CUDA and GPU performance (no AI, as at NVIDIA)

<a id="q5"></a>

#### Q5. This CUDA block reduction gives a wrong sum that changes from run to run. Find and fix the bugs

Round: CUDA and GPU performance. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Shared memory reduction, barriers, bounds, atomics.

<details>
<summary>Show answer</summary>

Task: for n = 1,000,003 floats the result is wrong and differs between runs. The launch is `block_sum<<<(n + 255) / 256, 256, 256 * sizeof(float)>>>(d_x, d_out, n)` after `cudaMemset(d_out, 0, sizeof(float))`. Find every bug, fix it, and say what you would change for speed.

```cuda
__global__ void block_sum(const float* x, float* out, int n) {
    extern __shared__ float s[];
    int tid = threadIdx.x;
    int i = blockIdx.x * blockDim.x + tid;
    s[tid] = x[i];
    for (int stride = 1; stride < blockDim.x; stride *= 2) {
        if (tid % (2 * stride) == 0) {
            s[tid] += s[tid + stride];
        }
    }
    if (tid == 0) {
        *out += s[0];
    }
}
```

What they grade: finding all three correctness bugs, knowing which one causes run-to-run changes, and a faster correct kernel.

Approach: (1) No bounds check: n is not a multiple of 256, so the last block reads past the end of `x`, which is undefined and adds garbage. Load `i < n ? x[i] : 0.0f`. (2) No `__syncthreads()` after the load or between levels: warps run independently, so a thread can read `s[tid + stride]` before another warp wrote it. That is a race and varies run to run. (3) `*out += s[0]` from 3,907 blocks is a read-modify-write race across blocks; most updates are lost. Use `atomicAdd`, or write one partial per block and reduce those in a second pass. For speed: the `tid % (2 * stride)` pattern leaves most threads in each warp idle and uses a slow modulo; sequential addressing (`tid < stride`, stride halving) keeps active threads packed into whole warps. A grid-stride loop that sums several elements per thread before the shared memory step cuts the block count, and the last 32 values can use `__shfl_down_sync` without shared memory. In production, `cub::BlockReduce` or `cub::DeviceReduce::Sum` is the baseline to beat.

Code:

```cuda
__global__ void block_sum(const float* __restrict__ x, float* out, int n) {
    extern __shared__ float s[];
    int tid = threadIdx.x;
    float v = 0.0f;
    for (int i = blockIdx.x * blockDim.x + tid; i < n; i += blockDim.x * gridDim.x)
        v += x[i];
    s[tid] = v;
    __syncthreads();
    for (int stride = blockDim.x / 2; stride > 0; stride >>= 1) {  // blockDim.x must be a power of two
        if (tid < stride) s[tid] += s[tid + stride];
        __syncthreads();
    }
    if (tid == 0) atomicAdd(out, s[0]);
}
```

A NumPy model of the fixed kernel and of the two memory bugs, checked against a float64 sum. If every block reads `*out` as 0 before any block writes, only the last writer's partial survives; the assert shows that result is off by over 99%.

```python
import numpy as np

def fixed(x, block=256, grid=120):
    total = np.float32(0)
    for b in range(grid):
        v = np.zeros(block, dtype=np.float32)
        for t in range(block):
            v[t] = x[b * block + t::block * grid].sum(dtype=np.float32)
        stride = block // 2
        while stride:
            v[:stride] += v[stride:2 * stride]
            stride //= 2
        total += v[0]
    return total

def partials(x, block, garbage):
    pad = (-len(x)) % block
    xp = np.concatenate([x, garbage[:pad]])
    return xp.reshape(-1, block).sum(axis=1, dtype=np.float32)

rng = np.random.default_rng(0)
n = 1_000_003
x = rng.random(n, dtype=np.float32)
truth = x.sum(dtype=np.float64)
assert abs(fixed(x) - truth) / truth < 1e-5
assert abs(fixed(x[:1000]) - x[:1000].sum(dtype=np.float64)) < 1e-3
p_oob = partials(x, 256, np.full(256, 1e6, dtype=np.float32))
assert abs(p_oob.sum(dtype=np.float64) - truth) > 1e6
p = partials(x, 256, np.zeros(256, dtype=np.float32))
last_writer_wins = p[-1]
assert abs(last_writer_wins - truth) / truth > 0.99
assert len(p) == 3907
```

Verify: test n values of 1, 255, 256, 257 and 1,000,003 against a float64 host sum with a relative tolerance, run under `compute-sanitizer --tool racecheck` and `memcheck`, and run the same input 100 times. With float `atomicAdd` the order of block adds varies, so expect tiny last-bit differences; if the reward or test needs bitwise repeatability, use the two-pass partials version.

Pitfalls:
- Putting `__syncthreads()` inside the `if (tid < stride)` branch, which deadlocks or is undefined because not all threads reach it.
- Dropping barriers for the last warp without `__syncwarp` or shuffles; since Volta, warp threads can diverge.

Why this is likely: CUDA prep guides for NVIDIA in 2026 list live reductions among the most common kernel tasks, a 2026 Blind invite lists a "bug bash" round, and the listing requires CUDA or GPU optimization; medium confidence.

</details>

<a id="q6"></a>

#### Q6. Write a fused row softmax that reads each row once, with one warp per row

Round: CUDA and GPU performance. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Warp shuffles, numerical stability, online softmax.

<details>
<summary>Show answer</summary>

Task: `y = softmax(x)` along rows of a `rows x cols` float matrix, with cols up to 1,024, attention-style masks (`-inf` entries) and rows that may be fully masked. Write the kernel, then a host model that proves the math. Say what changes when cols is 32,768.

What they grade: subtracting the row max for stability, coalesced loads, shuffle reductions without shared memory, the fully masked row, and knowing online softmax for long rows.

Approach: one warp owns one row. Lane `l` loads columns `l, l + 32, l + 64, ...` into registers, so each load instruction is coalesced across the warp and the row is read from HBM once. Reduce the max with `__shfl_xor_sync` (5 steps), compute `exp(v - m)`, reduce the sum the same way, then write `v / sum`. One read and one write per element is the minimum traffic, so the kernel is memory bound and should run near HBM bandwidth. A fully masked row has `m = -inf`; `-inf - (-inf)` is NaN, so special-case it and write zeros. For cols too large for registers, use the online softmax recurrence: keep a running max `m` and sum `d`, and when a new max arrives rescale `d` by `exp(m_old - m_new)`. That computes both in one pass, so the kernel reads the row twice instead of three times.

Code:

```cuda
template <int PER_LANE>  // PER_LANE * 32 >= cols
__global__ void softmax_rows(const float* __restrict__ x, float* __restrict__ y, int rows, int cols) {
    int row = (blockIdx.x * blockDim.x + threadIdx.x) / 32;
    int lane = threadIdx.x % 32;
    if (row >= rows) return;  // whole warp exits together
    const float* xr = x + (size_t)row * cols;
    float v[PER_LANE];
    float m = -INFINITY;
#pragma unroll
    for (int k = 0; k < PER_LANE; ++k) {
        int c = lane + 32 * k;
        v[k] = c < cols ? xr[c] : -INFINITY;
        m = fmaxf(m, v[k]);
    }
    for (int o = 16; o > 0; o >>= 1) m = fmaxf(m, __shfl_xor_sync(0xffffffffu, m, o));
    float d = 0.0f;
#pragma unroll
    for (int k = 0; k < PER_LANE; ++k) {
        v[k] = (m == -INFINITY) ? 0.0f : __expf(v[k] - m);
        d += v[k];
    }
    for (int o = 16; o > 0; o >>= 1) d += __shfl_xor_sync(0xffffffffu, d, o);
    float inv = d > 0.0f ? 1.0f / d : 0.0f;
    float* yr = y + (size_t)row * cols;
#pragma unroll
    for (int k = 0; k < PER_LANE; ++k) {
        int c = lane + 32 * k;
        if (c < cols) yr[c] = v[k] * inv;
    }
}
```

```python
import numpy as np

def warp_softmax(row):
    cols = len(row)
    per_lane = -(-cols // 32)
    v = np.full((per_lane, 32), -np.inf)
    v.flat[:cols] = row
    lane_max = v.max(axis=0)
    m = lane_max.max()
    if m == -np.inf:
        return np.zeros(cols)
    e = np.exp(v - m)
    d = e.sum(axis=0).sum()
    return e.flat[:cols] / d

def online_stats(row):
    m, d = -np.inf, 0.0
    for x in row:
        m_new = max(m, x)
        if m_new == -np.inf:
            continue
        d = d * np.exp(m - m_new) + np.exp(x - m_new)
        m = m_new
    return m, d

def ref(row):
    e = np.exp(row - row.max())
    return e / e.sum()

rng = np.random.default_rng(1)
for cols in (1, 31, 32, 33, 1000, 1024):
    r = rng.standard_normal(cols) * 5
    assert np.allclose(warp_softmax(r), ref(r))
    m, d = online_stats(r)
    assert np.isclose(m, r.max()) and np.isclose(d, np.exp(r - r.max()).sum())
big = np.array([1000.0, 1001.0, 999.0])
assert np.allclose(warp_softmax(big), ref(big)) and np.isfinite(warp_softmax(big)).all()
masked = np.array([2.0, -np.inf, 1.0, -np.inf])
out = warp_softmax(masked)
assert out[1] == 0 and out[3] == 0 and np.isclose(out.sum(), 1.0)
assert (warp_softmax(np.full(40, -np.inf)) == 0).all()
assert online_stats(np.array([-np.inf, 3.0])) == (3.0, 1.0)
```

Verify: column counts of 1, 31, 32, 33 and 1,024 cover the padded last lane; inputs near 1,000 overflow a naive `exp(x)` but pass here; masked and fully masked rows behave; the online recurrence matches the direct max and sum.

Pitfalls:
- Early `return` per thread inside a warp before a full-mask shuffle, which is undefined; exit per warp only.
- `__expf` is a fast approximation; check its error against your tolerance, or use `expf`.

Follow-ups: fusing the scale and mask of attention into the same kernel; why FlashAttention is this recurrence applied to tiles of Q times K transpose; what Nsight Compute metric proves the kernel is at the memory roofline (DRAM throughput as a percent of peak).

Why this is likely: 2026 NVIDIA prep guides list fused softmax with reductions and tiled kernels as common live CUDA tasks, and kernels like this are what the team's agents are asked to write; medium confidence.

</details>

<a id="q7"></a>

#### Q7. Use a roofline to decide whether three kernels are memory or compute bound, then fix the slow one

Round: CUDA and GPU performance. AI policy: Not allowed. Type: Optimize. Difficulty: Medium. Topic: Arithmetic intensity, roofline, fusion, Nsight Compute.

<details>
<summary>Show answer</summary>

Task: on an H100 SXM (take 989 TFLOPS dense BF16 and 3.35 TB/s HBM3 from the spec sheet), an agent produced these kernels. Classify each, compute the best possible time, and fix the one furthest from its bound.

```text
kernel                    shape                     dtype  measured
gelu_elementwise          4096 x 4096               bf16   61.0 us
gemm                      4096 x 4096 x 4096        bf16   180.0 us
gemv_decode (batch 1)     8192 x 8192 weights       bf16   44.0 us

Nsight Compute, gelu_elementwise:
  DRAM throughput           33% of peak
  SM throughput              4% of peak
  achieved occupancy        19%
  global load width          2 bytes per thread per instruction
  grid                       2 blocks per SM
```

What they grade: arithmetic intensity (FLOPs per byte), the ridge point, a floor time for each kernel, and a fix aimed at the real limiter.

Approach: the ridge point is 989e12 / 3.35e12, about 295 FLOP per byte. A kernel below it is memory bound, and its floor is bytes / bandwidth. GELU reads and writes 2 bytes per element and does about 20 FLOPs, so it is around 5 FLOP per byte: memory bound, floor 67.1 MB / 3.35 TB/s = 20 us. It runs at 61 us, so it is 3 times off. The GEMM moves at least 100.7 MB for 137 GFLOPs, about 1,365 FLOP per byte: compute bound, floor 139 us; 180 us is 77% of peak, which is fine. The batch 1 GEMV is about 1 FLOP per byte, floor 40 us; 44 us is already 91%. So fix GELU. The profile says both DRAM and SM are low, with 2-byte loads and low occupancy: there are too few bytes in flight to hide memory latency. Use 16-byte vector loads (8 bf16 values per thread per load), a grid-stride loop with enough blocks to fill every SM. Better still, fuse GELU into the GEMM epilogue so the 67 MB intermediate never goes to HBM at all.

Code:

```python
PEAK_FLOPS = 989e12
PEAK_BW = 3.35e12
RIDGE = PEAK_FLOPS / PEAK_BW

def roofline(flops, nbytes):
    ai = flops / nbytes
    floor_s = max(flops / PEAK_FLOPS, nbytes / PEAK_BW)
    return ai, floor_s, "memory" if ai < RIDGE else "compute"

n = 4096
cases = {
    "gelu_elementwise": (20 * n * n, 2 * 2 * n * n, 61.0e-6),
    "gemm": (2 * n ** 3, 3 * 2 * n * n, 180.0e-6),
    "gemv_decode": (2 * 8192 * 8192, 2 * 8192 * 8192 + 2 * 2 * 8192, 44.0e-6),
}
res = {k: roofline(f, b) + (t,) for k, (f, b, t) in cases.items()}
assert 294 < RIDGE < 296
ai, floor_s, bound, t = res["gelu_elementwise"]
assert bound == "memory" and round(ai) == 5 and round(floor_s * 1e6, 1) == 20.0
assert round(t / floor_s, 1) == 3.0
ai, floor_s, bound, t = res["gemm"]
assert bound == "compute" and round(ai) == 1365 and round(floor_s * 1e6) == 139
assert round(floor_s / t, 2) == 0.77
ai, floor_s, bound, t = res["gemv_decode"]
assert bound == "memory" and round(ai, 2) == 1.0 and round(floor_s * 1e6) == 40
gemv_ai = lambda b: 2 * 8192 * 8192 * b / (2 * 8192 * 8192 + 2 * 2 * 8192 * b)
assert gemv_ai(1) < 1.01 and 270 < gemv_ai(295) < 280 and gemv_ai(300) < RIDGE
worst = max(res, key=lambda k: res[k][3] / res[k][1])
assert worst == "gelu_elementwise"
```

Verify: Nsight Compute shows the standalone GELU near peak DRAM throughput, or no GELU kernel after fusion; outputs match the unfused version within BF16 tolerance.

Pitfalls:
- Counting the GEMM's bytes as reads of every tile, which makes it look memory bound; the floor uses compulsory traffic.
- Optimizing the GEMM first because it is the biggest number, when it is already at 77% of peak.

Follow-ups: the GEMV's intensity grows by about 1 FLOP per byte per sequence in the batch, so decode stays memory bound until the batch nears the ridge; an agent can use this floor to reject impossible speedups (Q9).

Why this is likely: roofline questions appear in 2026 CUDA interview guides, a 2026 prep guide describes NVIDIA hiring managers going deep into kernels you wrote, and the listing asks for co-design down to GPUs; medium confidence.

</details>

### ML and agent systems deep dive (no AI, as at NVIDIA)

<a id="q8"></a>

#### Q8. Implement causal multi-head self-attention from scratch and prove the mask works

Round: ML and agent systems deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Transformer attention, shapes, masking, cost.

<details>
<summary>Show answer</summary>

Task: given `x` of shape `(B, T, D)` and weights `Wq, Wk, Wv, Wo` of shape `(D, D)`, write causal multi-head self-attention with `h` heads in NumPy. Then test it, and state its compute and memory cost.

What they grade: correct reshapes and transposes, the `1 / sqrt(d_head)` scale, a stable softmax, a mask applied before the softmax, and tests that would catch a wrong transpose.

Approach: project once, then split D into `h` heads of `d_head = D / h` with a reshape to `(B, T, h, d_head)` and a transpose to `(B, h, T, d_head)`. Scores are `q @ k^T / sqrt(d_head)`, shape `(B, h, T, T)`. Set positions above the diagonal to `-inf` so token t only sees tokens up to t, softmax over the last axis, multiply by `v`, merge heads back and apply `Wo`. Test it three ways: against a slow loop over batch, head and position written straight from the formula; a causality test that changes future tokens and checks earlier outputs do not move; and attention rows that sum to 1. Cost: about `4 B T D^2` multiply-adds for the projections plus `2 B T^2 D` for scores and the weighted sum, and the score matrix needs `B h T^2` memory, which is why long contexts use FlashAttention-style tiling that never stores it.

Code:

```python
import numpy as np

def softmax(z):
    z = z - z.max(axis=-1, keepdims=True)
    e = np.exp(z)
    return e / e.sum(axis=-1, keepdims=True)

def mha(x, Wq, Wk, Wv, Wo, h, causal=True):
    B, T, D = x.shape
    dh = D // h
    split = lambda t: t.reshape(B, T, h, dh).transpose(0, 2, 1, 3)
    q, k, v = split(x @ Wq), split(x @ Wk), split(x @ Wv)
    s = q @ k.transpose(0, 1, 3, 2) / np.sqrt(dh)
    if causal:
        s = np.where(np.triu(np.ones((T, T), bool), 1), -np.inf, s)
    a = softmax(s)
    o = (a @ v).transpose(0, 2, 1, 3).reshape(B, T, D)
    return o @ Wo, a

def mha_loop(x, Wq, Wk, Wv, Wo, h):
    B, T, D = x.shape
    dh = D // h
    out = np.zeros((B, T, D))
    for b in range(B):
        q, k, v = x[b] @ Wq, x[b] @ Wk, x[b] @ Wv
        for hd in range(h):
            sl = slice(hd * dh, (hd + 1) * dh)
            for t in range(T):
                sc = np.array([q[t, sl] @ k[j, sl] / np.sqrt(dh) for j in range(t + 1)])
                w = np.exp(sc - sc.max())
                w /= w.sum()
                out[b, t, sl] = sum(w[j] * v[j, sl] for j in range(t + 1))
    return out @ Wo

rng = np.random.default_rng(0)
B, T, D, h = 2, 7, 16, 4
x = rng.standard_normal((B, T, D))
W = [rng.standard_normal((D, D)) / np.sqrt(D) for _ in range(4)]
y, a = mha(x, *W, h)
assert y.shape == (B, T, D)
assert np.allclose(y, mha_loop(x, *W, h))
assert np.allclose(a.sum(-1), 1) and np.all(np.triu(a[0, 0], 1) == 0)
x2 = x.copy()
x2[:, 4:] = rng.standard_normal((B, T - 4, D))
y2, _ = mha(x2, *W, h)
assert np.allclose(y2[:, :4], y[:, :4]) and not np.allclose(y2[:, 4:], y[:, 4:])
```

Verify: the loop reference catches a wrong head split (reshaping to `(B, h, T, dh)` directly mixes tokens and heads and still runs), the perturbation test catches a mask on the wrong side of the diagonal, and the row-sum test catches a softmax over the wrong axis.

Pitfalls:
- Scaling by `sqrt(D)` instead of `sqrt(d_head)`.
- Adding a large negative number after the softmax instead of before it.

Follow-ups: grouped-query attention (fewer K and V heads, which shrinks the KV cache, Q15); the KV cache for decoding one token at a time; why attention is memory bound at decode time (Q7).

Why this is likely: a 2025 Glassdoor report for an NVIDIA deep learning engineer says the round was to implement multi-headed self attention, and a NeMo LLM engineer report on Blind lists attention among the onsite topics; medium confidence.

</details>

<a id="q9"></a>

#### Q9. Reward climbs during kernel RL but held-out correctness does not. Read the trace and find the cause

Round: ML and agent systems deep dive. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Reward hacking, verifier buffers, physical sanity checks.

<details>
<summary>Show answer</summary>

Task: you own the RL run below. Training reward and train pass rate climb, the held-out pass rate (a separate harness, run nightly) is flat, and some speedups are huge. Find the cause from the trace, prove it, and fix the environment.

```text
step  mean_reward  train_pass  heldout_pass  median_speedup  p99_speedup  mean_tokens
   0      1.12        0.41         0.39           1.03            2.1        2310
 100      1.58        0.55         0.47           1.21            4.8        2050
 200      2.31        0.78         0.48           1.35           61.0        1420
 300      2.74        0.91         0.46           1.31          290.0         880

sample rollout, step 300, problem relu_backward, n = 4194304, fp32, H100 SXM
  compile      ok (0.9 s)
  correctness  PASS  max_abs_err 0.0
  timing       candidate 0.004 ms  reference 0.091 ms  speedup 22.8
  kernel:
    __global__ void relu_backward(const float* g, const float* x, float* out, int n) {
        int i = blockIdx.x * blockDim.x + threadIdx.x;
        if (i >= n) return;
        if (i < 0) out[i] = x[i] > 0.f ? g[i] : 0.f;
    }
```

What they grade: reading the evidence in order, proving the cause with a test, and fixing the environment.

Approach: three signals line up. Train and held-out pass rates split after step 100, so the train verifier accepts things the held-out one rejects. The p99 speedup goes to 290 while the median stays near 1.3, so a few rollouts get huge rewards. Tokens per rollout fall, so the policy found something short. The sample kernel never writes (`i < 0` is never true for this launch), yet it passes with an error of exactly 0.0. A kernel that writes nothing can only match if `out` already holds the answer: the train verifier runs the reference into the same buffer, or compares the buffer with itself. And the timing is physically impossible. ReLU backward moves 3 x 4 bytes x 4,194,304 = 50.3 MB, which takes at least 15 us at 3.35 TB/s, but it reports 4 us. Fix the environment: separate output buffers, fill the candidate's output with NaN before every run, reject unwritten or non-finite outputs, reject any time below the memory floor from Q7, and cap the speedup. Then rescore the last checkpoints with the fixed verifier and roll back to the last good one.

Code:

```python
import numpy as np

def ref(g, x, out):
    out[:] = np.where(x > 0, g, 0)

def noop(g, x, out):
    pass

def good(g, x, out):
    out[:] = g * (x > 0)

def verify_buggy(cand, g, x):
    out = np.zeros_like(x)
    ref(g, x, out)
    expected = out
    cand(g, x, out)
    return float(np.max(np.abs(out - expected))) == 0.0

def verify_fixed(cand, g, x, cand_s, bw=3.35e12):
    expected = np.empty_like(x)
    ref(g, x, expected)
    out = np.full_like(x, np.nan)
    cand(g, x, out)
    if not np.all(np.isfinite(out)):
        return False, "unwritten output"
    if not np.allclose(out, expected, rtol=1e-6, atol=0):
        return False, "wrong values"
    floor_s = 3 * x.nbytes / bw
    if cand_s < floor_s:
        return False, f"{cand_s * 1e6:.1f} us is below the {floor_s * 1e6:.1f} us memory floor"
    return True, "ok"

rng = np.random.default_rng(0)
n = 4_194_304
g = rng.standard_normal(n).astype(np.float32)
x = rng.standard_normal(n).astype(np.float32)
assert verify_buggy(noop, g, x)
ok, why = verify_fixed(noop, g, x, cand_s=4e-6)
assert not ok and why == "unwritten output"
ok, why = verify_fixed(good, g, x, cand_s=4e-6)
assert not ok and why.startswith("4.0 us is below the 15.0 us")
assert verify_fixed(good, g, x, cand_s=18e-6) == (True, "ok")
```

Verify: the buggy verifier passes the no-op kernel, the fixed one rejects it, the floor check rejects an impossible time, and a correct kernel at 18 us passes.

Pitfalls:
- Fixing it by penalizing short outputs in the reward, which treats the symptom and invites the next hack.
- Trusting the train pass rate as the success metric for the run; the held-out harness is what told you.

Follow-ups: other hacks to expect (calling the PyTorch reference, detecting the test shape, work on a side stream that timing misses), and reading the top 1% of rewards by hand every day.

Why this is likely: NVIDIA's RL blog for agents centers on verifiable rewards and its kernel blog relies on a verifier in a closed loop, and the listing names verification tools and reward mechanisms; finding reward hacking in a trace is the kind of practical debugging NVIDIA's 2026 rounds favor; medium confidence.

</details>

<a id="q10"></a>

#### Q10. Design a tiered reward for kernel RL and compute GRPO advantages for a group of rollouts

Round: ML and agent systems deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: RLVR, reward shaping, GRPO.

<details>
<summary>Show answer</summary>

Task: the policy writes a CUDA kernel for a PyTorch operator. Define the reward from compile, correctness and timing results, then compute GRPO advantages for a group of 8 rollouts of one prompt. Say what goes wrong when all 8 get the same reward.

What they grade: correctness gating speed, thresholds above timing noise, the group-relative advantage, and knowing where the gradient signal disappears.

Approach: correctness gates every other reward. Failing to compile or failing the verifier gives -1. A correct kernel gets 1. Correct and more than 5% faster than eager PyTorch gets 2, and more than 5% faster than both eager and `torch.compile` gets 3. The 5% margin has to exceed the noise of your timing (median of repeated runs); otherwise the policy is rewarded for luck. CUDA Agent (arXiv 2602.24286) uses tiers of this kind; the exact values here are my choice. Tiers beat a raw speedup ratio because a ratio is unbounded and one timing glitch dominates the batch (Q9). GRPO samples a group of answers per prompt and uses `(r - mean) / (std + eps)` within the group as each answer's advantage, so no value network is needed. NVIDIA's RL blog recommends starting with a simple binary verifier reward to prove the loop, then adding shaping. If all 8 rollouts score the same, every advantage is 0 and that prompt teaches nothing. Track the share of such groups: if most prompts are all-fail or all-solved, fix the prompt mix with easier or harder problems.

Code:

```python
import numpy as np

def reward(compiled, correct, t, t_eager, t_compile, margin=0.05):
    if not (compiled and correct):
        return -1.0
    if t < t_eager * (1 - margin):
        return 3.0 if t < t_compile * (1 - margin) else 2.0
    return 1.0

def grpo_advantages(rewards, eps=1e-6):
    r = np.asarray(rewards, dtype=float)
    return (r - r.mean()) / (r.std() + eps)

def dead_group_share(groups):
    return float(np.mean([np.ptp(g) == 0 for g in groups]))

assert reward(False, False, 1, 1, 1) == -1.0
assert reward(True, False, 0.1, 1, 1) == -1.0
assert reward(True, True, 0.97, 1.0, 0.9) == 1.0
assert reward(True, True, 0.90, 1.0, 0.8) == 2.0
assert reward(True, True, 0.70, 1.0, 0.8) == 3.0
assert reward(True, True, 0.78, 1.0, 0.8) == 2.0
group = [reward(*o) for o in [
    (True, True, 0.70, 1.0, 0.8), (True, True, 0.90, 1.0, 0.8),
    (True, True, 0.99, 1.0, 0.8), (True, False, 0.10, 1.0, 0.8),
    (False, False, 0, 1.0, 0.8), (True, True, 1.20, 1.0, 0.8),
    (True, True, 0.85, 1.0, 0.8), (True, False, 0.50, 1.0, 0.8)]]
assert group == [3.0, 2.0, 1.0, -1.0, -1.0, 1.0, 2.0, -1.0]
adv = grpo_advantages(group)
assert abs(adv.sum()) < 1e-9 and adv.argmax() == 0 and adv[3] == adv.min()
assert np.allclose(grpo_advantages([1.0] * 8), 0)
assert np.allclose(grpo_advantages(np.array(group) * 10), adv, atol=1e-5)
assert dead_group_share([group, [-1] * 8, [1] * 8, [1, -1] * 4]) == 0.5
```

Verify: the tier tests cover every branch, advantages sum to 0 and rank the best kernel first, identical groups give zero advantage, and scaling every reward leaves advantages unchanged.

Pitfalls:
- Giving incorrect kernels partial credit for speed (the defect in Q4).
- Measuring eager and `torch.compile` baselines once on a different GPU than the candidate.

Follow-ups: Dr. GRPO-style variants drop the division by the standard deviation because it overweights prompts with a tiny reward spread, the ones that are nearly all solved or nearly all failed; when a learned reward model would help (style and readability) and why it is easier to hack than a verifier; KL to the reference policy.

Why this is likely: NVIDIA's 2026 RL blog for agents covers RLVR with GRPO and binary verifier rewards, and the listing names RL environments and reward mechanisms; designing the reward is the core of the work; medium confidence.

</details>

<a id="q11"></a>

#### Q11. Design a benchmark for CUDA code generation agents that still means something next year

Round: ML and agent systems deep dive. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Eval design, contamination, statistical power, performance scoring.

<details>
<summary>Show answer</summary>

Task: leadership wants one number to track the team's CUDA agent against frontier models every quarter. Design the benchmark: problems, grading, metrics, statistics and how it stays uncontaminated.

What they grade: separate correctness and performance scores, statistics that say when a gap is real, contamination controls, and a plan for saturation.

Approach: problems are self-contained tasks (a kernel, or a CUDA or CCCL API use) with a prompt, a hidden test harness with edge shapes and fresh random inputs, and a reference solution. Grade correctness by compiling and running the hidden tests in a sandbox, with tolerances set per dtype. Score performance separately, only on correct solutions, as speedup over a stated baseline on pinned hardware, in the style of KernelBench's fast_p (the share of problems that are correct and faster than the baseline by a factor p). Release versions the way ComputeEval does (2025.2 has 232 problems): keep a private held-out split, write new problems after the cutoff dates of the models you test, put canary strings in the data, and scan public code for leaked solutions. Tag difficulty and retire saturated problems.

Numbers: in ComputeEval 2025.2, pass@1 was 0.5819 for GPT-5 (medium), 0.5517 for Claude Sonnet 4.0 and 0.4397 for DeepSeek-R1. With 232 problems the standard error of a score near 0.58 is about 3.2 points, so a 95% interval is about plus or minus 6.3 points, and the 3 point gap between the first two is not resolved by separate intervals. Use a paired test on the same problems (McNemar or a paired bootstrap). To detect a 3 point gap with 80% power unpaired you would need about 4,200 problems; paired, with 20% of problems disagreeing, about 1,700.

```python
import math

def se(p, n):
    return math.sqrt(p * (1 - p) / n)

def n_unpaired(p, d, za=1.96, zb=0.8416):
    return (za + zb) ** 2 * 2 * p * (1 - p) / d ** 2

def n_paired(disagree, d, za=1.96, zb=0.8416):
    return (za + zb) ** 2 * disagree / d ** 2

assert round(se(0.5819, 232) * 100, 1) == 3.2
assert round(1.96 * se(0.5819, 232) * 100, 1) == 6.3
assert round((0.5819 - 0.5517) * 100, 1) == 3.0
assert 4200 < n_unpaired(0.58, 0.03) < 4300
assert 1700 < n_paired(0.20, 0.03) < 1800
```

Alternatives: LLM-as-judge grading is cheap but cannot tell whether a kernel is correct, so it fails for this domain. A public, fixed benchmark is easy to share but gets contaminated within a release cycle. Pure speedup ranking without a correctness gate rewards hacks (Q9). Versioned releases with a private split win, at the cost of maintaining harnesses.

Risks: timing noise across GPUs and drivers (pin hardware, report medians and the driver version); tests too weak to catch wrong kernels (mutation testing of the harness).

Why this is likely: NVIDIA publishes ComputeEval with pass@1 results for CUDA code and its kernel generation blog measures agents on KernelBench, and the listing asks for evaluation frameworks; medium confidence.

</details>

<a id="q12"></a>

#### Q12. Spend a fixed inference budget across 100 kernel problems: best-of-N or sequential refinement

Round: ML and agent systems deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Inference-time compute, verifier-guided search, budget allocation.

<details>
<summary>Show answer</summary>

Task: you have 800 generation attempts for 100 problems and a verifier that says pass or fail. Compare a fixed best-of-8 per problem, a shared pool that stops each problem once it passes, and sequential refinement where each retry sees the verifier's error. Estimate the solve counts.

What they grade: using the verifier as a stopping rule, modelling difficulty separately for each problem, and naming what refinement assumes.

Approach: model each problem with a per-attempt success probability p. The illustrative mix is 60 easy problems (p from 0.3 to 0.9), 30 hard (0.01 to 0.1) and 10 the model cannot solve. Best-of-8 solves a problem with chance 1 - (1 - p)^8, but it spends all 8 attempts on easy problems that pass on the first try. A pooled budget gives one attempt to each unsolved problem per round until the budget is gone, so leftover attempts flow to hard problems. Refinement feeds the compiler or test error back into the next attempt; the model below assumes each failed attempt raises p by 20% of its starting value, which is an assumption to check on your own data. NVIDIA's kernel generation work used this kind of closed loop with a verifier and reported correct kernels for 100% of KernelBench Level-1 and 96% of Level-2 problems, with more inference time per problem raising the solve rate.

Code:

```python
import numpy as np

rng = np.random.default_rng(0)
p = np.concatenate([rng.uniform(0.3, 0.9, 60), rng.uniform(0.01, 0.1, 30), np.zeros(10)])
BUDGET = 800

def best_of_n(p, n):
    return float(np.sum(1 - (1 - p) ** n))

def pooled(p, budget, gain=0.0, trials=2000, seed=1):
    r = np.random.default_rng(seed)
    total = 0
    for _ in range(trials):
        solved = np.zeros(len(p), bool)
        tries = np.zeros(len(p))
        left = budget
        while left > 0 and not solved.all():
            idx = np.flatnonzero(~solved)[:left]
            p_now = np.minimum(1.0, p[idx] * (1 + gain * tries[idx]))
            solved[idx] |= r.random(len(idx)) < p_now
            tries[idx] += 1
            left -= len(idx)
        total += solved.sum()
    return total / trials

pass1 = best_of_n(p, 1)
uniform = best_of_n(p, BUDGET // len(p))
adaptive = pooled(p, BUDGET)
refine = pooled(p, BUDGET, gain=0.2)
assert round(pass1) == 38 and round(uniform) == 71
assert 82 <= adaptive <= 84 and 89 <= refine <= 91
assert uniform < adaptive < refine <= 90
```

Verify: with these assumptions one attempt each solves about 38, best-of-8 about 71, the pooled budget about 83 and refinement about 90. The 10 impossible problems cap every method at 90, and the final check asserts that.

Pitfalls:
- Comparing methods at different total budgets; count tokens or GPU seconds, not attempts, when refinement prompts are longer.
- Letting impossible problems eat the pool; stop a problem after a cap of failures or when its errors stop changing.

Follow-ups: tree search that branches from the best partial kernel; using a cheap compile check before an expensive GPU run; how the verifier's false pass rate caps the real gain (Q9).

Why this is likely: NVIDIA's kernel generation blog is about inference-time scaling with a verifier in the loop, and the listing names coding agents and co-design; choosing how to spend test-time compute is a core research decision for this team; medium confidence.

</details>

### System design (no AI, as at NVIDIA)

<a id="q13"></a>

#### Q13. Design the closed-loop agent that generates, verifies, profiles and refines CUDA kernels

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Agent loop, verification services, budget control, provenance.

<details>
<summary>Show answer</summary>

Task: given a PyTorch operator and target shapes, the system should return the fastest correct CUDA kernel it can find for a given GPU, for 1,000 operators per night. Design the components, the data flow, the budget and how you know a result is trustworthy.

What they grade: the verifier at the center, separate CPU and GPU pools, budget control, trustworthy timing and reproducible records.

Approach: a controller holds one task per operator and runs rounds. The generator (an LLM served with a shared prefix, Q15) gets the operator, the shapes, the target architecture and the history of failed attempts with their errors. The compile service runs `nvcc` in a sandbox on CPU nodes and caches binaries by source hash and architecture (Q1). The verifier runs the candidate on fresh, NaN-filled buffers with edge shapes and random inputs against the PyTorch reference (Q4, Q9). The profiler times correct kernels only, on GPUs locked for timing (median of repeated runs), and attaches Nsight Compute DRAM and SM throughput so the next prompt can say "memory bound at 40% of peak". The controller stops a task when it passes and stops improving, or hits its budget, and moves the rest of the budget to unsolved tasks (Q12). Every accepted kernel is stored with its prompt, model version, compiler flags, GPU, driver and timings.

Numbers: with assumptions of 20 attempts per operator, 4,000 output tokens per attempt, 10 CPU seconds to compile and 5 GPU seconds to verify and time, one night is 20,000 attempts, 80 million output tokens, about 56 CPU hours of compiling and about 28 GPU hours of verification. Cache compiles and fail fast on syntax errors before the GPU queue.

```python
attempts = 1000 * 20
assert attempts * 4000 == 80_000_000
assert round(attempts * 10 / 3600) == 56
assert round(attempts * 5 / 3600, 1) == 27.8
```

Alternatives: best-of-N with no feedback parallelizes perfectly but wastes attempts (Q12: 71 against 83 to 90 solved). Evolutionary search over a population of kernels finds more speed on hard operators at a higher cost per task. Hand-tuned templates with autotuning win on common operators, so start from the best known kernel when one exists.

Risks: verifier gaps the generator learns to exploit, timing noise, and results that only hold on one GPU or driver; rerun winners on a second machine.

Why this is likely: NVIDIA's kernel generation blog describes the core of this loop (DeepSeek-R1 generating kernels and a verifier on H100 that writes the next prompt), and this design adds profiling and budget control on top; the listing describes multi-agent workflows that generate and tune CUDA code; high confidence in the theme, the numbers are assumptions.

</details>

<a id="q14"></a>

#### Q14. Design the RL environment and sandbox that run a million coding agent rollouts safely

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: RL environments, isolation, throughput, async rollouts.

<details>
<summary>Show answer</summary>

Task: training uses GRPO with 256 prompts per step and 8 rollouts per prompt. Each rollout writes code, compiles it and runs it on a GPU. Design the environment service: isolation, throughput, failure handling and how the trainer gets rewards.

What they grade: treating generated code as hostile, pool sizing, a busy trainer and reproducible rewards.

Approach: split three pools. Trainer GPUs run the policy update and generation. A CPU sandbox pool compiles and runs tests. A separate verifier GPU pool runs and times kernels, so timing never shares a GPU with training. Each rollout runs in a fresh container with no network, a read-only image, a small scratch directory, CPU, memory and wall-clock limits, and a user-space kernel sandbox such as gVisor or a microVM for the CPU side. On the GPU side, give each job a whole GPU, kill on timeout, and reset the device if a kernel hangs. The environment API is small and versioned: `reset(task) -> observation`, `step(action) -> observation, reward, done, info`, with every verifier version pinned in `info` so a reward can be recomputed later. Generation and verification run asynchronously with a bounded queue, and each sample records the policy version that produced it, so you can cap staleness to one or two steps. NVIDIA's NeMo Gym takes this environment-as-a-service view, and NVIDIA reports that Nemotron 3 Super's RL used 21 NeMo Gym verifiers, 37 datasets and about 1.2 million rollouts.

Numbers: one step is 2,048 rollouts. With assumed costs of 10 CPU seconds to compile and 2 GPU seconds to verify, a step needs 20,480 CPU seconds and 4,096 GPU seconds. With 512 cores and 64 verifier GPUs, that is 40 and 64 seconds, which is shorter than a typical generation phase, so verification can hide behind generation if it runs async.

```python
rollouts = 256 * 8
assert rollouts == 2048
assert rollouts * 10 / 512 == 40.0
assert rollouts * 2 / 64 == 64.0
```

Alternatives: verifying on trainer GPUs needs no extra hardware but stalls training and adds timing noise; CPU or emulator runs are safe but cannot measure speed. A verifier service with its own GPU pool wins.

Risks: sandbox escapes (no secrets in the image, no network), reward drift when a verifier changes mid-run (version pinning), and stragglers that hold up a step (timeouts and dropping late samples).

Why this is likely: NVIDIA's 2026 RL blog describes NeMo RL and NeMo Gym environments with verifiers, and the listing names RL environments, verification tools and data pipelines; medium confidence in the exact prompt.

</details>

<a id="q15"></a>

#### Q15. Size the KV cache and pick a serving engine for eight agents sharing a long prompt on a 70B model

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: KV cache math, prefix caching, vLLM, SGLang, TensorRT-LLM.

<details>
<summary>Show answer</summary>

Task: eight agents use a Llama 3.1 70B class model (80 layers, 8 KV heads, head size 128, BF16). They share a 16,000-token system prompt with tool and hardware docs, and each adds 4,000 tokens of its own. Size the KV cache with and without prefix sharing, choose GPUs and an engine.

What they grade: the per-token KV formula, the effect of sharing, weights plus cache fitting on real GPUs, and a reasoned engine choice.

Approach: KV bytes per token = 2 (K and V) x layers x KV heads x head size x bytes = 2 x 80 x 8 x 128 x 2 = 327,680 bytes, about 0.33 MB. Without sharing, 8 x 20,000 tokens = 160,000 tokens = 52.4 GB. With prefix caching the shared 16,000 tokens are stored once: 48,000 tokens = 15.7 GB. The weights are about 140 GB in BF16, so one 80 GB H100 is not enough; tensor parallel over 4 GPUs puts about 35 GB of weights and 2 of the 8 KV heads on each GPU, which leaves plenty of room for the cache and for batching more agents. An FP8 KV cache halves the cache again.

Numbers:

```python
per_token = 2 * 80 * 8 * 128 * 2
assert per_token == 327_680
no_share = 8 * (16_000 + 4_000) * per_token
shared = (16_000 + 8 * 4_000) * per_token
assert round(no_share / 1e9, 1) == 52.4 and round(shared / 1e9, 1) == 15.7
assert round(shared / 4 / 1e9, 1) == 3.9
assert 140e9 / 4 == 35e9
```

Alternatives: vLLM's PagedAttention stores the cache in blocks and its automatic prefix caching reuses shared prefixes; it is the fastest way to prototype. SGLang's RadixAttention keeps prefixes in a radix tree and was built for programs that make many related calls, which fits agent workflows with branching. TensorRT-LLM gives NVIDIA-tuned kernels, in-flight batching and KV cache reuse; current releases serve Hugging Face checkpoints directly with `trtllm-serve`, and it pays off most for a fixed production model where you tune its configuration and latency matters. For the research loop, choose SGLang or vLLM; move the stable production agent to TensorRT-LLM.

Risks: prefix caching only helps when the shared text is byte-identical at the start, so put dynamic content (timestamps, agent names) after the shared block; long private contexts grow the cache linearly; eviction under load turns cache hits into full prefills.

Why this is likely: the listing prefers experience with TensorRT-LLM, SGLang and vLLM and describes multi-agent workflows, and KV cache sizing is a common serving question; medium confidence.

</details>

<a id="q16"></a>

#### Q16. Design a multi-agent workflow for CUDA work, and decide when one agent is better

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Agent roles, handoffs, tools, failure compounding.

<details>
<summary>Show answer</summary>

Task: a team wants a "planner, coder, tester, profiler, reviewer" set of agents to port a PyTorch model's hot path to custom kernels. Design it, and say when a single agent with tools would be better.

What they grade: roles that match real boundaries, typed handoffs, deterministic tools where possible, and numbers on cost and error compounding.

Approach: make only the uncertain steps agents. A planner reads the profile of the model and proposes which operators to fuse and in what order. A coder agent writes one kernel at a time. Testing and timing run as deterministic tools: the verifier and profiler from Q13 return structured JSON, which removes a whole class of model mistakes. A reviewer agent reads the diff for unsafe patterns and hardware assumptions. Handoffs are typed artifacts (a plan file, kernel source, a test report, profile metrics), stored with versions, so any agent can restart from the last good state and a human can inspect each step. A single agent with the same tools is better when the task fits in one context and the steps are tightly coupled, like fixing one kernel, because every handoff loses context and adds tokens.

Numbers: if each of 5 handoffs keeps the needed information with probability 0.95, the chain succeeds 0.95^5, about 77% of the time, before any coding errors. A five-role workflow also re-reads shared context, so expect several times the tokens of one agent. Spend that only where parallelism pays, such as several coder agents working on independent operators at once.

Alternatives: one agent with tools (cheapest, best for single kernels), a fixed pipeline with no LLM planning (most predictable, but cannot choose fusions), and planner plus parallel workers (wins when the plan splits into independent pieces).

Risks: agents agreeing with each other without checking (let the verifier tool decide pass or fail); loops between coder and reviewer (cap rounds); costs that grow quietly (per-task token budgets).

Why this is likely: the listing names agent architectures and multi-agent workflows for CUDA development as the core of the role; medium confidence in the exact prompt.

</details>

### Hiring manager and behavioral (no AI, as at NVIDIA)

<a id="q17"></a>

#### Q17. Explain how you would train a 70B model across 64 GPUs, with the memory and communication math

Round: Hiring manager screen. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Data, tensor and pipeline parallelism, ZeRO, all-reduce cost.

<details>
<summary>Show answer</summary>

Task: you have 8 nodes of 8 H100 80 GB GPUs, NVLink inside a node and InfiniBand between nodes. Explain how you would fine-tune a 70B model with Adam, why it does not fit with plain data parallelism, and what the communication costs.

What they grade: the memory math, the right parallelism for each link, the all-reduce cost, and the pipeline bubble.

Approach: mixed-precision Adam keeps about 16 bytes per parameter (BF16 weights and gradients, plus FP32 master weights and two Adam moments), so 70B parameters need about 1.12 TB before activations. Plain data parallelism copies all of it to every GPU, which cannot fit in 80 GB. Shard the states with ZeRO stage 3 or FSDP across 64 GPUs: about 17.5 GB each, which leaves room for activations, with activation checkpointing for long sequences. Put tensor parallelism inside a node, where NVLink is fast, and shard or replicate across nodes over InfiniBand. If memory is still tight, add pipeline parallelism, whose idle bubble is (p - 1) / (m + p - 1) for p stages and m microbatches: 4 stages with 16 microbatches idle about 16% of the time. For comparison, plain data parallelism within one node uses a ring all-reduce, which sends 2 (N - 1) / N times the data per GPU; for 140 GB of BF16 gradients across 8 GPUs that is 245 GB, about 0.61 s at an assumed 400 GB/s per GPU. ZeRO-3 across 64 GPUs instead does two parameter all-gathers (forward and backward) plus one gradient reduce-scatter, so each GPU sends about 3 (N - 1) / N x 140 GB, about 413 GB per step or 1.5 times the all-reduce, which is why that communication is overlapped with compute.

Code:

```python
params = 70e9
assert round(params * 16 / 1e12, 2) == 1.12
assert params * 16 / 64 / 1e9 == 17.5
bubble = lambda p, m: (p - 1) / (m + p - 1)
assert round(bubble(4, 16), 3) == 0.158 and bubble(1, 16) == 0
ring_bytes = lambda size, n: 2 * (n - 1) / n * size
grads = params * 2
assert round(ring_bytes(grads, 8) / 1e9, 1) == 245.0
assert round(ring_bytes(grads, 8) / 400e9, 2) == 0.61
assert ring_bytes(grads, 64) / ring_bytes(grads, 8) < 1.15
zero3_bytes = lambda size, n: 3 * (n - 1) / n * size
assert round(zero3_bytes(grads, 64) / 1e9) == 413
assert zero3_bytes(grads, 64) / ring_bytes(grads, 64) == 1.5
```

Verify: the numbers check that sharding brings optimizer state under 80 GB, that the ring cost barely grows from 8 to 64 GPUs (bandwidth sets the cost), and that more microbatches shrink the bubble.

Pitfalls:
- Forgetting the FP32 master weights and counting only 2 bytes per parameter.
- Putting tensor parallelism across nodes, where its frequent per-layer communication is slow.

Follow-ups: LoRA as the alternative when full fine-tuning is not needed (optimizer state only for the adapters); how RL changes the picture (generation and training share or swap GPUs); how you would spot a communication bottleneck in an Nsight Systems timeline.

Why this is likely: a 2026 Glassdoor report for an NVIDIA deep learning engineer says the first round covered distributed training plus past projects, and the role trains agents with RL at scale; medium confidence.

</details>

<a id="q18"></a>

#### Q18. Walk us through a result that looked better than it was, and how you check AI-written code

Round: Research talk or project presentation. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Research judgment, verification habits, AI tool use.

<details>
<summary>Show answer</summary>

Task: in the talk or the follow-up questions, the panel asks for one project where an early result was wrong or inflated, how you found out, and what you changed. They also ask how you use coding assistants day to day and how you check their output.

What they grade: a specific story with numbers, how you found the problem (not luck), what you changed in the process, and honest, concrete AI habits.

Approach: pick a real story in four parts. Context: the system and the metric, for example an agent whose pass rate jumped 15 points after a change. Signal: what did not fit, such as a held-out set that did not move, a speedup faster than the memory floor allows, or results that vanished on a second GPU. Proof: the experiment that settled it, such as rerunning with a fixed seed, a known-bad input, or an ablation. Change: the guard you added so it cannot happen again, such as a held-out harness, a canary test or a physical bound on timing. For AI tools, describe what you hand off (boilerplate, test scaffolding, first drafts of kernels), what you always check yourself (numerics, tolerances, synchronization, memory bounds, any API you have not used before), and one real case where the assistant was wrong and how you caught it, like a nonexistent function name or a missing `__syncthreads()`.

Pitfalls:
- A story where the fix was "we looked harder"; name the test or tool.
- Claiming you never use AI tools, or that you trust them; neither is credible for this team.
- Taking credit for team work; say what you owned.

Follow-ups: what you would measure next; what you would do with twice the compute; how you would explain the result to the hardware team.

Why this is likely: NVIDIA research loops center on a talk with questions on choices and next steps (Glassdoor senior research scientist report and a 2026 prep guide), and NVIDIA bans AI in interviews while the team builds AI coding agents, so how you check AI output is a natural question; medium confidence.

</details>

## Prep checklist

- [ ] Write a block reduction, a warp-per-row softmax and a tiled matmul in CUDA from memory, with bounds checks and barriers in the right places.
- [ ] Compute arithmetic intensity and the roofline floor for elementwise, GEMM and GEMV kernels on H100 without notes.
- [ ] Read Nsight Compute output and name the limiter: DRAM throughput, SM throughput, occupancy and load width.
- [ ] Implement causal multi-head attention in NumPy with a loop reference test, and explain its cost and the KV cache.
- [ ] Implement the unbiased pass@k estimator and compute a confidence interval and a paired test for two models.
- [ ] Design a kernel verifier that survives no-op, stale-cache, wrong-tail and impossible-timing kernels.
- [ ] Explain RLVR and GRPO advantages, tiered rewards and what to do when whole groups get the same reward.
- [ ] Sketch an RL environment service with sandboxing, async rollouts and pool sizing from per-rollout costs.
- [ ] Compute KV cache sizes with and without prefix sharing and compare vLLM, SGLang and TensorRT-LLM for agents.
- [ ] Solve LRU Cache, Merge k Sorted Lists and Top K Frequent Elements by hand under time.
- [ ] Prepare a 45 minute project talk with ablations, one result that turned out wrong, and what you would do next.
- [ ] Plan to do every round without AI tools, and expect at least one in-person round.

## Sources

- NVIDIA, How We Hire (official process, AI tool rule, in-person onsite): https://www.nvidia.com/en-us/about-nvidia/careers/how-we-hire/
- NVIDIA technical blog, Automating GPU kernel generation with DeepSeek-R1 and inference-time scaling: https://developer.nvidia.com/blog/automating-gpu-kernel-generation-with-deepseek-r1-and-inference-time-scaling
- NVIDIA technical blog, Benchmarking LLMs on AI-generated CUDA code with ComputeEval 2025.2: https://developer.nvidia.com/blog/benchmarking-llms-on-ai-generated-cuda-code-with-computeeval-2025-2
- NVIDIA ComputeEval repository: https://github.com/NVIDIA/compute-eval
- NVIDIA technical blog, AI agent reinforcement learning: https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-reinforcement-learning
- CUDA Agent paper (third party, topic source for tiered rewards): https://arxiv.org/abs/2602.24286
- Glassdoor, NVIDIA deep learning engineer interviews: https://www.glassdoor.com/Interview/NVIDIA-Deep-Learning-Engineer-Interview-Questions-EI_IE7633.0,6_KO7,29.htm
- Glassdoor, NVIDIA senior research scientist interviews: https://www.glassdoor.co.in/Interview/NVIDIA-Senior-Research-Scientist-Interview-Questions-EI_IE7633.0,6_KO7,32.htm
- Glassdoor, NVIDIA research interviews: https://www.glassdoor.com/Interview/NVIDIA-Research-Interview-Questions-EI_IE7633.0,6_KO7,15.htm
- Blind, NVIDIA interview format thread: https://www.teamblind.com/post/nvidia-interview-format-y4cpgjpa
- Blind, NVIDIA onsite for ML and LLM engineer: https://www.teamblind.com/post/nvidia-onsite-for-ml-llm-engineer-c7w0b8qd
- interviewing.io, NVIDIA interview questions: https://interviewing.io/nvidia-interview-questions
- cleverprep, NVIDIA research scientist guide: https://www.cleverprep.com/companies/nvidia/research-scientist
- techscreen, NVIDIA technical interview process: https://techscreen.app/articles/nvidia-technical-interview-process-2026
- calibrd, NVIDIA interview guide: https://www.calibrd.com/interview-prep/nvidia-interview
- datainterview, CUDA interview questions: https://www.datainterview.com/blog/cuda-interview-questions
- Crowd-reported LeetCode company tags, NVIDIA (low confidence, practice links only): https://github.com/liquidslr/leetcode-company-wise-problems
