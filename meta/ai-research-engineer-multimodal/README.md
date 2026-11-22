# Meta: AI Research Engineer (multimodal foundation models)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Meta says schedules and prep guides are personalized per role and that the recruiter is the source of truth. Research and ML loops use a different round mix from software engineering loops. This estimate is built from Meta's hiring page, 2026 Research Engineer candidate reports, an aggregated 2026 report on Meta loops, and a prep vendor guide. The role trains multimodal LLMs (understanding, generation, grounding), owns pre, mid and post-training data curation and pipelines, and asks for formal tech-lead experience, so expect a senior level loop.

1. Recruiter screen (20 to 30 minutes, phone or video). Background, level, and interest in multimodal foundation model work. Some roles send a short profile form first. AI: Not allowed.
2. Hiring manager screen and project deep dive (about 60 minutes, video, no coding). Expect very detailed questions about your current work. It doubles as team match. AI: Not allowed.
3. Technical coding screen (45 minutes, CoderPad, may be waived for referrals). Usually two LeetCode-style problems. Some Meta screens now open with an AI system you built, then AI-assisted coding. AI: Not allowed in the classic version.
4. Classic coding (45 minutes, CoderPad, no code execution). Two problems. A 2026 Research Engineer report names a grid island problem and a tree path-sum problem. AI: Not allowed.
5. AI coding (45 to 60 minutes, CoderPad). The evidence supports two readings. One is Meta's AI-enabled round: a multi-file project with a built-in assistant where you fix bugs, extend and optimize. The other is ML-domain coding: one 2026 Research Engineer got a sparse dot product and a batch norm forward pass. Prepare for both. AI: Provided (CoderPad assistant) in the AI-enabled version; Unknown in the ML coding version.
6. AI and ML design, usually two rounds (45 minutes each, CoderPad with Mermaid). For this team expect multimodal data pipeline and model design, plus an ML product system such as LLMs in feed ranking. The assistant is nominally on, but candidates say it often goes unused. AI: Provided (CoderPad assistant).
7. Behavioral (45 minutes). Conflict, ambiguity, results, tech-lead scope with cross-functional impact, and an AI-oriented behavioral question that shows up in most 2026 Meta loops. AI: Not allowed.
8. Research or project retrospective (45 to 60 minutes, possible). Present and defend past work in depth. Candidates report it for some research positions. AI: Not allowed.
9. Hiring committee, team match and offer. Candidates report about 2 to 3 months overall. AI: Not relevant.

All 2026 Research Engineer rounds were reported as virtual. Classic algorithm questions are limited to two below because the reports still show classic coding at the screen and in one onsite round. The AI coding round is covered from both sides: ML coding (Q3, Q4) and assistant-driven multi-file work (Q5 to Q8).

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | A segmentation mask is a 0/1 grid. Flip at most one 0 and return the largest... | Classic algorithm | Classic coding or screen | Not allowed | Hard |
| [Q2](#q2) | Training configs set sampling budgets with integer expressions. Implement calc(s)... | Classic algorithm | Classic coding or screen | Not allowed | Medium |
| [Q3](#q3) | Implement the BatchNorm2d forward pass in NumPy for (B, C, H, W) input, with... | Build | AI coding (ML coding version) | Unknown | Medium |
| [Q4](#q4) | Image and caption features are sparse vectors over 1 million dimensions. Build... | Build | AI coding (ML coding version) | Unknown | Medium |
| [Q5](#q5) | This three-file image-text pipeline drops valid pairs and shards differently per... | Debug | AI coding (AI-enabled version) | Provided (CoderPad assistant) | Medium |
| [Q6](#q6) | The assistant wrote a MetaCLIP-style balanced sampler. Review it like a pull... | Review | AI coding (AI-enabled version) | Provided (CoderPad assistant) | Hard |
| [Q7](#q7) | Extend the pipeline so every dataset build writes a versioned manifest and a diff... | Build | AI coding (AI-enabled version) | Provided (CoderPad assistant) | Medium |
| [Q8](#q8) | dedup_near() builds a full similarity matrix over 2 million image embeddings and... | Optimize | AI coding (AI-enabled version) | Provided (CoderPad assistant) | Hard |
| [Q9](#q9) | After a data-mixture change, image-text loss spikes and the run goes to NaN. Read... | Trace debug | AI and ML design | Provided (CoderPad assistant) | Hard |
| [Q10](#q10) | Design the evaluation for visual grounding: metric, contamination check, and LLM... | Concept plus code | AI and ML design | Provided (CoderPad assistant) | Hard |
| [Q11](#q11) | Design the data curation system for pre, mid and post-training of a multimodal... | Design | AI and ML design | Provided (CoderPad assistant) | Hard |
| [Q12](#q12) | One large pretraining run, five candidate data mixtures. Plan small runs and fit a... | Concept plus code | AI and ML design | Provided (CoderPad assistant) | Hard |
| [Q13](#q13) | Choose an architecture that both understands and generates images: adapter, early... | Design | AI and ML design | Provided (CoderPad assistant) | Hard |
| [Q14](#q14) | Estimate compute, storage and wall-clock time to curate and tokenize 10 billion... | Design | AI and ML design | Provided (CoderPad assistant) | Medium |
| [Q15](#q15) | Design how a multimodal LLM improves ranking in an image feed: semantic IDs, where... | Design | AI and ML design | Provided (CoderPad assistant) | Hard |
| [Q16](#q16) | Your curation change moved a 500-item visual QA benchmark from 61.0 to 62.4... | Concept plus code | Project deep dive or retrospective | Not allowed | Medium |
| [Q17](#q17) | Tell me about leading a cross-functional data or model effort where AI-written... | Behavioral | Behavioral | Not allowed | Medium |

### Classic coding (no AI, as at Meta)

These match the classic coding round and the coding screen. CoderPad runs without an assistant and, in the onsite round, without code execution, so trace your code by hand.

<a id="q1"></a>

#### Q1. (Classic coding or screen, no AI) A segmentation mask is a 0/1 grid where 1 marks object pixels. You may flip at most one 0 to 1. Return the size of the largest 4-connected object afterward.

Round: Classic coding or coding screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Hard. Topic: Grid connected components.
Related practice: [Making A Large Island](https://leetcode.com/problems/making-a-large-island/) (Hard), [Number of Islands](https://leetcode.com/problems/number-of-islands/) (Medium).

<details>
<summary>Show answer</summary>

Task: Given an n by m grid of 0s and 1s, return the largest connected region of 1s you can get by changing at most one 0. The onsite round has no code execution, so the dry run is by hand.

What they grade: a linear-time plan stated before coding, correct all-ones and all-zeros cases, no double counting when one region touches the flipped cell from two sides, and a clean hand trace.

Approach: The brute force flips each 0 and reruns a flood fill. That is O((nm)^2), far too slow on a 500 by 500 mask. Union-find also works, but labeling is shorter to write and easier to trace. Pass one flood fills every region with an explicit stack (Python recursion overflows on a 250,000 pixel object), writes a region id into a separate label grid, and stores the size per id. A separate grid keeps the caller's mask unchanged. Pass two visits each 0, collects the distinct region ids of its up to four neighbors in a set, and scores 1 plus the sum of their sizes. The answer is the best candidate or the largest existing region, which covers a grid with no 0s. Both passes are O(nm) time and O(nm) memory.

```python
import random

DIRS = ((1, 0), (-1, 0), (0, 1), (0, -1))

def largest_after_flip(grid):
    n, m = len(grid), len(grid[0])
    label = [[0] * m for _ in range(n)]
    size, nxt = {}, 1
    for i in range(n):
        for j in range(m):
            if grid[i][j] == 1 and label[i][j] == 0:
                label[i][j], stack, s = nxt, [(i, j)], 0
                while stack:
                    x, y = stack.pop()
                    s += 1
                    for dx, dy in DIRS:
                        a, b = x + dx, y + dy
                        if 0 <= a < n and 0 <= b < m and grid[a][b] == 1 and label[a][b] == 0:
                            label[a][b] = nxt
                            stack.append((a, b))
                size[nxt], nxt = s, nxt + 1
    best = max(size.values(), default=0)
    for i in range(n):
        for j in range(m):
            if grid[i][j] == 0:
                ids = {label[i + dx][j + dy] for dx, dy in DIRS
                       if 0 <= i + dx < n and 0 <= j + dy < m and label[i + dx][j + dy]}
                best = max(best, 1 + sum(size[k] for k in ids))
    return best

def brute(grid):
    def biggest(g):
        n, m, seen, best = len(g), len(g[0]), set(), 0
        for i in range(n):
            for j in range(m):
                if g[i][j] and (i, j) not in seen:
                    stack, s = [(i, j)], 0
                    seen.add((i, j))
                    while stack:
                        x, y = stack.pop(); s += 1
                        for dx, dy in DIRS:
                            a, b = x + dx, y + dy
                            if 0 <= a < n and 0 <= b < m and g[a][b] and (a, b) not in seen:
                                seen.add((a, b)); stack.append((a, b))
                    best = max(best, s)
        return best
    best = biggest(grid)
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == 0:
                g = [row[:] for row in grid]; g[i][j] = 1
                best = max(best, biggest(g))
    return best

assert largest_after_flip([[1, 0], [0, 1]]) == 3
assert largest_after_flip([[1, 1], [1, 0]]) == 4
assert largest_after_flip([[1, 1], [1, 1]]) == 4
assert largest_after_flip([[0, 0], [0, 0]]) == 1
assert largest_after_flip([[1, 0, 1]]) == 3
rng = random.Random(0)
for _ in range(300):
    n, m = rng.randint(1, 6), rng.randint(1, 6)
    g = [[int(rng.random() < 0.5) for _ in range(m)] for _ in range(n)]
    assert largest_after_flip(g) == brute(g), g
print("ok")
```

Verify: trace [[1, 0], [0, 1]] by hand (two regions of size 1, either 0 joins both, answer 3). Run the all-ones and all-zeros grids and a single row, then compare against the brute force on 300 random small grids.

Pitfalls: adding a region twice when it touches the 0 from two sides (the set fixes it), returning 0 for an all-zero grid, and recursion depth on large objects.

Follow-ups: 8-connectivity (eight offsets, same plan); flipping k cells (no longer local, discuss why it gets much harder); masks arriving per video frame, where you would reuse labels between frames.

Why this is likely: a 2026 Meta Research Engineer onsite report names this exact problem in the classic coding round, and recent Meta-tagged lists include Number of Islands. Medium confidence.

</details>

<a id="q2"></a>

#### Q2. (Classic coding or screen, no AI) Training configs set sampling budgets with integer expressions such as "3*(200-40)/4". Implement calc(s) with + - * /, parentheses, spaces and unary minus, where division truncates toward zero.

Round: Classic coding or coding screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Stack parsing and recursion.
Related practice: [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) (Medium), [Basic Calculator](https://leetcode.com/problems/basic-calculator/) (Hard).

<details>
<summary>Show answer</summary>

Task: Evaluate a string expression over non-negative integers with the four operators, parentheses, spaces, and a leading or post-parenthesis unary minus. Division truncates toward zero, as in C, so -7/2 is -3.

What they grade: operator precedence without a full parser generator, correct handling of nested parentheses and unary minus, truncation toward zero (Python's // floors, which is wrong here), and a clear trace.

Approach: There are three options. Python eval is unsafe on config text, and the interviewer wants to see the parsing. Shunting-yard to postfix is correct but needs two passes and two stacks. I use one pass per parenthesis level: keep a stack of signed terms, the current number, and the last operator. When an operator, a closing parenthesis or the end arrives, apply the last operator: push for + and -, combine with the top of the stack for * and /. An opening parenthesis recurses and returns its value plus the index of its closing parenthesis. The level's value is the sum of the stack. Unary minus falls out for free because the first operator defaults to + with a number of 0. Time and space are O(len(s)).

```python
def calc(s):
    s = s.replace(" ", "")

    def parse(i):
        stack, num, op = [], 0, "+"
        while i < len(s):
            c = s[i]
            if c.isdigit():
                num = num * 10 + int(c)
            elif c == "(":
                num, i = parse(i + 1)
            if c in "+-*/)" or i == len(s) - 1:
                if op == "+":
                    stack.append(num)
                elif op == "-":
                    stack.append(-num)
                elif op == "*":
                    stack.append(stack.pop() * num)
                else:
                    stack.append(int(stack.pop() / num))
                if c == ")":
                    return sum(stack), i
                op, num = c, 0
            i += 1
        return sum(stack), i

    return parse(0)[0]

assert calc("3*(200-40)/4") == 120
assert calc(" 14-3/2 ") == 13
assert calc("2-(5-6)") == 3
assert calc("-(2+3)*2") == -10
assert calc("-7/2") == -3
assert calc("((7))") == 7
assert calc("1+2*3-4/2") == 5
assert calc("10/(1-3)") == -5
print("ok")
```

Verify: trace "2-(5-6)": the inner level returns -1, the outer applies - to it, and 2 - (-1) gives 3. Test -7/2 for truncation, nested parentheses, a trailing parenthesis, and spaces.

Pitfalls: using // (floors to -4), forgetting to flush the last number at the end of the string, and pushing the inner value twice when the string ends right after a closing parenthesis.

Follow-ups: floats and variables such as "0.4*web_tokens" (switch to a tokenizer and a recursive descent grammar); error messages with positions; a related tree problem from the same round, summing the root-to-leaf numbers of a binary tree.

Why this is likely: a 2026 vendor guide for Meta Research Engineers lists an expression calculator, and a 2026 aggregated report says Basic Calculator still appears on Meta phone screens. Medium confidence.

</details>

### ML coding (the ML version of the AI coding round)

One 2026 Research Engineer report says the "AI coding" round was ML-domain coding: a sparse dot product and a batch norm forward pass. Whether the assistant is on in that version is not reported, so the policy is marked Unknown. Practice them without help.

<a id="q3"></a>

#### Q3. Implement the BatchNorm2d forward pass in NumPy for input shaped (B, C, H, W): batch statistics per channel in training, running statistics in eval, and a learnable scale and shift.

Round: AI coding (ML coding version). AI policy: Unknown. Type: Build. Difficulty: Medium. Topic: Normalization layers, train versus eval behavior.

<details>
<summary>Show answer</summary>

Task: The spec of the existing module: a class BatchNorm2d(num_channels, eps=1e-5, momentum=0.1) with attributes gamma, beta, running_mean, running_var and a training flag, and an empty __call__(x). Fill in __call__ so it matches the PyTorch convention: normalize with the biased batch variance, but update running_var with the unbiased one.

What they grade: the right reduction axes (0, 2, 3), correct broadcasting back to (B, C, H, W), the momentum convention (new = (1 - m) * old + m * batch), the biased versus unbiased variance detail, and tests that prove it.

Approach: In training, compute mean and variance over batch, height and width for each channel. The count per channel is B times H times W, not B, which is the most common slip. Normalize with sqrt(var + eps), then apply gamma and beta, reshaped to (1, C, 1, 1). Update the running buffers with the unbiased variance, guarding a count of 1. In eval, use the running buffers and leave them unchanged. Alternatives: a per-channel Python loop is clearer but about C times slower; einsum adds nothing here; a fused one-pass mean and mean-of-squares is faster but loses precision when the mean is large, so I use two passes and say so.

```python
import numpy as np

class BatchNorm2d:
    def __init__(self, c, eps=1e-5, momentum=0.1):
        self.gamma, self.beta = np.ones(c), np.zeros(c)
        self.running_mean, self.running_var = np.zeros(c), np.ones(c)
        self.eps, self.momentum, self.training = eps, momentum, True

    def __call__(self, x):
        if self.training:
            mean = x.mean(axis=(0, 2, 3))
            var = x.var(axis=(0, 2, 3))
            n = x.shape[0] * x.shape[2] * x.shape[3]
            unbiased = var * n / max(n - 1, 1)
            m = self.momentum
            self.running_mean = (1 - m) * self.running_mean + m * mean
            self.running_var = (1 - m) * self.running_var + m * unbiased
        else:
            mean, var = self.running_mean, self.running_var
        shape = (1, -1, 1, 1)
        xh = (x - mean.reshape(shape)) / np.sqrt(var.reshape(shape) + self.eps)
        return self.gamma.reshape(shape) * xh + self.beta.reshape(shape)

rng = np.random.default_rng(0)
x = rng.normal(3.0, 2.0, size=(8, 4, 5, 5))
bn = BatchNorm2d(4)
y = bn(x)
assert np.allclose(y.mean(axis=(0, 2, 3)), 0, atol=1e-7)
assert np.allclose(y.std(axis=(0, 2, 3)), 1, atol=1e-3)
c1 = x[:, 1].ravel()
assert np.allclose(y[:, 1].ravel(), (c1 - c1.mean()) / np.sqrt(c1.var() + 1e-5))
for _ in range(200):
    bn(rng.normal(3.0, 2.0, size=(8, 4, 5, 5)))
assert np.allclose(bn.running_mean, 3.0, atol=0.15)
assert np.allclose(bn.running_var, 4.0, atol=0.4)
bn.training = False
before = bn.running_mean.copy()
z = bn(x)
assert np.array_equal(before, bn.running_mean)
assert z.shape == x.shape
one = BatchNorm2d(2)
assert np.isfinite(one(np.ones((1, 2, 1, 1)))).all()
print("ok")
```

Verify: per-channel output mean 0 and std 1 in training; one channel checked against a hand formula; running stats converge to the true mean 3 and variance 4; eval leaves the buffers alone; a batch with one value per channel stays finite.

Pitfalls: reducing over axis 0 only, updating running stats in eval, using the unbiased variance to normalize, and forgetting that eps goes inside the square root.

Follow-ups: the backward pass; why batch norm breaks with micro-batches of 1 per GPU and when you would switch to GroupNorm or LayerNorm; SyncBatchNorm across data-parallel workers.

Why this is likely: a 2026 Meta Research Engineer onsite report says the AI coding round included a batch norm forward pass on BCHW input from a public ML coding problem set. Medium confidence.

</details>

<a id="q4"></a>

#### Q4. Image and caption features are sparse vectors over 1 million dimensions with about 1 percent non-zeros. Build a SparseVector with dot(), then extend it to return the top k stored vectors for a query across 1 million stored vectors.

Round: AI coding (ML coding version). AI policy: Unknown. Type: Build. Difficulty: Medium. Topic: Sparse vectors, inverted index, retrieval.

<details>
<summary>Show answer</summary>

Task: Part one: SparseVector(dense or pairs) and dot(other). Part two: an index over many stored vectors with top_k(query, k) that returns ids and scores, highest first, ties broken by lower id.

What they grade: a representation that stores only the non-zeros, dot cost bounded by the smaller vector, the jump from pairwise dots to an inverted index for part two, and memory numbers.

Approach: Store a dict from index to value. Dot iterates the smaller dict and looks up the larger one, so cost is O(min(nnz_a, nnz_b)). The alternative, two sorted arrays with two pointers, costs O(nnz_a + nnz_b) but is cache friendly and is the better choice when both sides have similar size or when you vectorize; I mention both and pick the dict for clarity. For part two, scoring every stored vector costs 1 million times 10,000 lookups, about 10^10 operations per query. An inverted index maps each dimension to a postings list of (doc id, value). A query only touches the postings of its own non-zero dimensions and accumulates scores, then a heap picks the top k. Memory: 1 million docs times 10,000 non-zeros is 10^10 postings, about 80 GB at 8 bytes each, so in production I would shard by doc id, prune low-weight postings, or move to dense embeddings with an approximate nearest neighbor index.

```python
import heapq
from collections import defaultdict
import numpy as np

class SparseVector:
    def __init__(self, dense=None, pairs=None):
        if pairs is None:
            pairs = [(i, v) for i, v in enumerate(dense) if v != 0]
        self.d = {i: v for i, v in pairs if v != 0}

    def dot(self, other):
        a, b = (self.d, other.d) if len(self.d) <= len(other.d) else (other.d, self.d)
        return sum(v * b[i] for i, v in a.items() if i in b)

class SparseIndex:
    def __init__(self):
        self.post = defaultdict(list)

    def add(self, doc_id, vec):
        for i, v in vec.d.items():
            self.post[i].append((doc_id, v))

    def top_k(self, q, k):
        acc = defaultdict(float)
        for i, qv in q.d.items():
            for doc, v in self.post.get(i, ()):
                acc[doc] += qv * v
        return heapq.nsmallest(k, ((-s, d) for d, s in acc.items()))

rng = np.random.default_rng(0)
dim, n = 2000, 300
dense = rng.normal(size=(n, dim)) * (rng.random((n, dim)) < 0.01)
vecs = [SparseVector(row) for row in dense]
assert abs(vecs[0].dot(vecs[1]) - dense[0] @ dense[1]) < 1e-9
assert SparseVector([0, 0]).dot(SparseVector([1, 2])) == 0
idx = SparseIndex()
for d, v in enumerate(vecs):
    idx.add(d, v)
qd = dense[7] + rng.normal(size=dim) * (rng.random(dim) < 0.005)
got = [(d, -s) for s, d in idx.top_k(SparseVector(qd), 5)]
scores = dense @ qd
want = sorted(range(n), key=lambda d: (-scores[d], d))[:5]
assert [d for d, _ in got] == want, (got, want)
assert got[0][0] == 7
print("ok")
```

Verify: dot matches a dense NumPy product, an empty vector gives 0, and top_k matches a brute-force dense ranking including the order.

Pitfalls: building the dense vector inside dot, scanning all stored vectors in part two, and float ties that make the order unstable.

Follow-ups: cosine instead of dot (store norms); a streaming top k with a bounded heap; when to drop sparse features for dense CLIP-style embeddings.

Why this is likely: the same 2026 Research Engineer report names the sparse vector dot product in the AI coding round, and a 2026 vendor guide lists sparse vectors among Meta Research Engineer coding examples. Medium confidence.

</details>

### AI-enabled coding (AI assistant allowed, as in Meta's AI-enabled round)

Meta's hiring page says many interviews include a built-in assistant in CoderPad, that candidates are expected to use it, and that outside AI tools are not allowed. The task is a multi-file project: read it, fix it, extend it, and verify every change the assistant makes.

<a id="q5"></a>

#### Q5. (AI assistant allowed) This three-file image-text pipeline (dedup.py, filters.py, shard.py) drops valid pairs and assigns different shards on different workers. Three tests fail. Find the root causes, fix them, and explain each assistant change you accept.

Round: AI coding (AI-enabled version). AI policy: Provided (CoderPad assistant). Type: Debug. Difficulty: Medium. Topic: Data pipelines for multimodal pretraining.

<details>
<summary>Show answer</summary>

Task: The spec says dedup removes exact duplicates (same image hash and same normalized caption), filters keep captions of at least 3 words and images at least 64 pixels per side, and shard_of must give the same shard for the same image on every worker and every run.

```text
pipeline/
  dedup.py
  filters.py
  shard.py
  run.py
tests/
  test_pipeline.py

$ pytest -q tests/test_pipeline.py
F.F.F
FAILED test_dedup_keeps_distinct_images_with_same_caption: assert 1 == 2
FAILED test_filter_keeps_three_word_caption: assert False
FAILED test_shard_is_stable_across_processes: worker A shard 3, worker B shard 0
3 failed, 2 passed
```

```python
# pipeline/dedup.py
def dedup(pairs):
    seen, out = set(), []
    for p in pairs:
        key = p["caption"].strip().lower()
        if key in seen:
            continue
        seen.add(key)
        out.append(p)
    return out

# pipeline/filters.py
MIN_WORDS = 3
def keep(p):
    return len(p["caption"].split()) > MIN_WORDS and p["width"] >= 64 and p["height"] >= 64

# pipeline/shard.py
def shard_of(p, n_shards):
    return hash(p["image_sha1"]) % n_shards
```

What they grade: reading the tests and spec before prompting, one root cause per failure, a fix that matches the spec, and catching a wrong assistant suggestion.

Approach: I run the tests first, then prompt: "Explain each failing test against the spec; do not edit yet." Root causes: dedup keys on caption only, so two different dog photos captioned "a photo of a dog" collapse (the key must include the image hash); the filter uses > 3, an off-by-one against "at least 3"; shard_of uses Python's built-in hash, which is salted per process for strings, so workers disagree. Then I ask for the three fixes as one small diff each. The assistant proposed setting PYTHONHASHSEED inside run.py. I rejected it because the variable only works if set before the interpreter starts, and it silently breaks the next time someone launches a worker differently. A stable digest (hashlib) is the right fix. Alternatives for dedup: caption-only (wrong, loses data) versus perceptual hashes for near duplicates (Q8; out of scope for exact dedup).

```python
import hashlib, os, subprocess, sys

def dedup(pairs):
    seen, out = set(), []
    for p in pairs:
        key = (p["image_sha1"], " ".join(p["caption"].lower().split()))
        if key not in seen:
            seen.add(key)
            out.append(p)
    return out

MIN_WORDS = 3
def keep(p):
    return len(p["caption"].split()) >= MIN_WORDS and p["width"] >= 64 and p["height"] >= 64

def shard_of(p, n_shards):
    h = hashlib.sha1(p["image_sha1"].encode()).digest()
    return int.from_bytes(h[:8], "big") % n_shards

a = {"image_sha1": "aa", "caption": "a photo of a dog", "width": 64, "height": 64}
b = {"image_sha1": "bb", "caption": "A photo of  a dog", "width": 80, "height": 80}
assert len(dedup([a, b, dict(a)])) == 2
assert keep({"caption": "red bike outside", "width": 64, "height": 64})
assert not keep({"caption": "red bike", "width": 64, "height": 64})
assert not keep({"caption": "red bike outside", "width": 63, "height": 64})
code = ("import hashlib;h=hashlib.sha1(b'aa').digest();"
        "print(int.from_bytes(h[:8],'big')%16, hash('aa')%16)")
outs = []
for seed in ("1", "2", "3"):
    env = dict(os.environ, PYTHONHASHSEED=seed)
    outs.append(subprocess.run([sys.executable, "-c", code], env=env,
                               capture_output=True, text=True).stdout.split())
assert len({o[0] for o in outs}) == 1
assert len({o[1] for o in outs}) > 1
assert int(outs[0][0]) == shard_of(a, 16)
print("ok")
```

Verify: the three failing tests pass, the two passing ones still pass, and a subprocess check under three hash seeds shows the new shard id is stable (the old built-in hash varies).

Pitfalls: accepting the PYTHONHASHSEED patch, normalizing captions differently in dedup and in later caption matching, and changing the shard function without a migration plan for already-written shards.

Follow-ups: dedup across 10,000 workers (a sharded key-value store or a sort-based join keyed on the hash); near-duplicate images; logging what each filter dropped so data owners can audit it.

Why this is likely: Meta's hiring page describes AI-enabled rounds as reading, debugging and building on existing code, a 2026 Research Engineer report lists an AI coding round, and the job asks for data pipelines and versioning. Medium confidence on this team using the AI-enabled version.

</details>

<a id="q6"></a>

#### Q6. (AI assistant allowed) The assistant wrote this MetaCLIP-style balanced sampler: match captions to metadata entries, count matches per entry, then cap head entries at t. Review it like a pull request, decide whether to merge, and write the tests that would have caught the defects.

Round: AI coding (AI-enabled version). AI policy: Provided (CoderPad assistant). Type: Review. Difficulty: Hard. Topic: Data curation and balancing.

<details>
<summary>Show answer</summary>

```python
import numpy as np

def load_metadata(path):
    with open(path) as f:
        return [eval(line) for line in f]

def match(texts, entries):
    return [[e for e in entries if e in t.lower()] for t in texts]

def balanced_sample(texts, entries, t=20000, seed=0):
    rng = np.random.default_rng(seed)
    matches = match(texts, entries)
    counts = {}
    for i in range(1, len(texts)):
        for e in matches[i]:
            counts[e] = counts.get(e, 0) + 1
    keep = []
    for i, ms in enumerate(matches):
        p = max(min(1.0, t / counts[e]) for e in ms)
        if rng.bernoulli(p):
            keep.append(i)
    return keep
```

Task: Review against the curation spec: substring match captions to metadata entries, drop texts with no match, keep every text of a tail entry (count at most t), and sample head entries so each keeps about t texts in expectation.

What they grade: finding the planted defects with a severity for each, a clear merge decision, and tests that fail on this code.

Approach: I read it top to bottom before asking the assistant anything, then ask it to list risky lines and compare its list with mine. Defects, worst first. Security: load_metadata calls eval on each line of a file, which runs arbitrary code from a data file; read plain strings instead. Hallucinated API: NumPy's Generator has no bernoulli method, so the code crashes on first use; use rng.random() < p. Off-by-one: the count loop starts at 1, so text 0 is never counted, which skews counts and raises KeyError when an entry only appears in text 0. Missing edge case: an unmatched text calls max on an empty sequence and crashes; the spec says drop it. Semantics: taking the max probability over entries differs from the published recipe, where a text is kept if any of its matched entries selects it; with several head entries the max under-samples. Performance: the match loop is texts times entries substring checks; with 500,000 entries that is too slow, so production needs an Aho-Corasick automaton. Decision: request changes before merging. When I asked the assistant to fix the crash, it replaced bernoulli with rng.binomial(1, p, size=len(ms)).any(), which silently changes the semantics a second time, so I wrote the fix myself.

```python
import numpy as np

def load_metadata_lines(lines):
    return [ln.strip().lower() for ln in lines if ln.strip()]

def balanced_sample(texts, entries, t, seed=0):
    rng = np.random.default_rng(seed)
    matches = [[e for e in entries if e in s.lower()] for s in texts]
    counts = {}
    for ms in matches:
        for e in ms:
            counts[e] = counts.get(e, 0) + 1
    keep = []
    for i, ms in enumerate(matches):
        if any(rng.random() < min(1.0, t / counts[e]) for e in ms):
            keep.append(i)
    return keep

entries = load_metadata_lines(["cat\n", "Dog\n", "axolotl\n", "\n"])
texts = (["a cat on a sofa"] * 4000 + ["a dog in snow"] * 300
         + ["an axolotl"] * 5 + ["nothing here"] * 50)
kept = balanced_sample(texts, entries, t=200, seed=1)
got = {k: sum(1 for i in kept if k in texts[i]) for k in ("cat", "dog", "axolotl", "nothing")}
assert abs(got["cat"] - 200) < 60, got
assert abs(got["dog"] - 200) < 40, got
assert got["axolotl"] == 5 and got["nothing"] == 0, got
assert balanced_sample(["axolotl only"], entries, t=200) == [0]
print("ok", got)
```

Verify: tail entries keep everything, head entries land near t, unmatched texts are dropped, the first text counts, and metadata loads without executing anything.

Pitfalls: merging because the demo ran on a tiny file, fixing the crash but not the semantics, and trusting a second assistant patch without a test.

Follow-ups: matching speed at 500,000 entries; multilingual metadata; how t changes the head-to-tail ratio and downstream zero-shot accuracy.

Why this is likely: Meta's hiring page says candidates should verify assistant output, and Meta's MetaCLIP 2 paper describes metadata-balanced curation for worldwide image-text data. Medium confidence on the format, low on the exact topic.

</details>

<a id="q7"></a>

#### Q7. (AI assistant allowed) Extend the pipeline so every dataset build writes a versioned manifest: a content hash per shard, a version id that changes only when the data or the build config changes, and a diff between two versions.

Round: AI coding (AI-enabled version). AI policy: Provided (CoderPad assistant). Type: Build. Difficulty: Medium. Topic: Dataset versioning and reproducibility.

<details>
<summary>Show answer</summary>

Task: The spec of the existing code: run.py calls build(config) and gets back a dict from shard name to shard bytes, then writes the shards. Add build_manifest(shards, config) that returns a JSON-ready dict, and diff(old, new) that lists added, removed and changed shards plus config keys that changed. Training checkpoints will store the version id, so any checkpoint can be traced to its exact data.

What they grade: a deterministic id, hashing content rather than names or timestamps, a useful diff, and tests that pin down determinism.

Approach: Prompt plan: "Read run.py and the shard writer. Propose a manifest schema with a per-shard sha256 and a version id derived from the sorted shard hashes and the canonical config. Do not touch the writer." Then I check the diff line by line. Alternatives: a git-style store such as DVC or lakeFS gives this for free and is what I would propose for the team; a timestamped folder name is simple but not reproducible, since rebuilding the same data gives a new id. Hashing content wins because identical data gives an identical id. The assistant's first version hashed json.dumps(config) without sort_keys and included the build time in the id. Both make the id change on a no-op rebuild; the determinism test caught it, and I fixed both. Cost: sha256 runs at roughly 1 GB per second per core, so hashing 100 TB of shards is about 28 core-hours, small next to the curation itself.

```python
import hashlib, json

def sha(b):
    return hashlib.sha256(b).hexdigest()

def build_manifest(shards, config):
    files = {name: {"sha256": sha(data), "bytes": len(data)} for name, data in sorted(shards.items())}
    canon = json.dumps(config, sort_keys=True, separators=(",", ":"))
    body = canon + "\n" + "\n".join(f"{n}:{f['sha256']}" for n, f in files.items())
    return {"version": sha(body.encode())[:16], "config": config, "files": files}

def diff(old, new):
    a, b = old["files"], new["files"]
    keys = set(old["config"]) | set(new["config"])
    return {
        "added": sorted(set(b) - set(a)),
        "removed": sorted(set(a) - set(b)),
        "changed": sorted(n for n in set(a) & set(b) if a[n]["sha256"] != b[n]["sha256"]),
        "config": sorted(k for k in keys if old["config"].get(k) != new["config"].get(k)),
    }

shards = {"s0": b"img-text-0", "s1": b"img-text-1"}
cfg = {"min_words": 3, "dedup": "exact", "t": 20000}
m1 = build_manifest(shards, cfg)
m2 = build_manifest(dict(reversed(list(shards.items()))), dict(reversed(list(cfg.items()))))
assert m1["version"] == m2["version"]
m3 = build_manifest(shards, {**cfg, "t": 10000})
assert m3["version"] != m1["version"]
m4 = build_manifest({"s0": b"img-text-0", "s1": b"CHANGED", "s2": b"new"}, cfg)
assert diff(m1, m4) == {"added": ["s2"], "removed": [], "changed": ["s1"], "config": []}
assert diff(m1, m3)["config"] == ["t"]
assert json.loads(json.dumps(m1)) == m1
print("ok", m1["version"])
```

Verify: same data and config in any order give the same id; a config change or a changed shard gives a new id; the diff names exactly what moved; the manifest round-trips through JSON.

Pitfalls: hashing file paths or timestamps, float formatting differences in the config, and storing the manifest somewhere the training job cannot read.

Follow-ups: lineage across stages (pre, mid and post-training sets that reference parent versions); deleting data for legal requests and what that does to old ids; hashing 100 TB without reading it twice.

Why this is likely: the job asks for managing data pipelines and versioning, and Meta's hiring page describes AI-enabled rounds where you build on an existing codebase. Medium confidence.

</details>

<a id="q8"></a>

#### Q8. (AI assistant allowed) dedup_near() builds a full similarity matrix over 2 million image embeddings and runs out of memory. Optimize it to find pairs above cosine 0.95 with at least 98 percent recall, and show the numbers.

Round: AI coding (AI-enabled version). AI policy: Provided (CoderPad assistant). Type: Optimize. Difficulty: Hard. Topic: Near-duplicate detection, locality-sensitive hashing.

<details>
<summary>Show answer</summary>

Task: The spec of the existing code: dedup_near(E, thr) takes an n by d float32 matrix of unit-norm embeddings (d = 512), computes S = E @ E.T, and returns every pair i < j with S[i, j] > thr. It works at 50,000 images and dies at 2 million.

What they grade: the memory and compute numbers, at least two alternatives with a reason for the pick, a recall check against an exact oracle, and how the answer changes at 10 billion.

Approach: Start with the numbers. The full matrix at 2 million is 4 times 10^12 entries, 16 TB in float32, so it can never be materialized. The compute is about 2 times 10^15 multiply-adds (4 times 10^15 FLOPs), roughly 10 seconds on one H100 with TF32 or bf16 tensor cores, or about a minute in strict FP32, so at 2 million the cheapest fix is tiling: multiply 8,192 by 8,192 tiles, threshold each tile, and keep only the hits. That is exact and uses about 268 MB per float32 tile. At 10 billion the pair count grows 25 million times, so exact search is out and you need candidate generation. Options: an approximate nearest neighbor index (IVF or HNSW, as in FAISS), or random-hyperplane LSH with banding, shown here because its recall is easy to reason about. A pair at cosine 0.95 has an angle of 0.318 rad, so it agrees on one random bit with probability 1 - 0.318 / pi, about 0.90. With 12 bits per band it collides in a band with probability 0.28, and with 16 bands recall is 1 - 0.72^16, about 0.995. Random pairs collide per band with probability 2^-12, so candidates shrink by about 250 times. The assistant first set 32 bits and 4 bands, which gives about 12 percent recall at 0.95; the recall test caught it.

```python
import numpy as np

def exact_tiled(E, thr, block=512):
    out = set()
    for s in range(0, len(E), block):
        S = E[s:s + block] @ E.T
        for i, j in zip(*np.where(S > thr)):
            if s + i < j:
                out.add((int(s + i), int(j)))
    return out

def lsh_pairs(E, thr, bits=12, bands=16, seed=0):
    rng = np.random.default_rng(seed)
    planes = rng.standard_normal((bands, bits, E.shape[1]))
    weights = 1 << np.arange(bits)
    cand = set()
    for b in range(bands):
        keys = (E @ planes[b].T > 0).astype(np.int64) @ weights
        order = np.argsort(keys, kind="stable")
        ks = keys[order]
        cuts = np.flatnonzero(np.diff(ks)) + 1
        for grp in np.split(order, cuts):
            g = sorted(int(x) for x in grp)
            for a in range(len(g)):
                for c in range(a + 1, len(g)):
                    cand.add((g[a], g[c]))
    hits = {(a, c) for a, c in cand if float(E[a] @ E[c]) > thr}
    return hits, len(cand)

rng = np.random.default_rng(0)
base = rng.standard_normal((1000, 64))
dups = base + 0.02 * rng.standard_normal(base.shape)
E = np.vstack([base, dups])
E /= np.linalg.norm(E, axis=1, keepdims=True)
truth = exact_tiled(E, 0.95)
hits, n_cand = lsh_pairs(E, 0.95)
recall = len(hits & truth) / len(truth)
all_pairs = len(E) * (len(E) - 1) // 2
assert len(truth) >= 1000 and hits <= truth
assert recall >= 0.98, recall
assert n_cand < 0.02 * all_pairs, n_cand
print("ok", len(truth), round(recall, 4), n_cand, all_pairs)
```

Verify: the tiled exact version is the oracle; LSH hits are a subset of it with recall at least 0.98 while scoring under 2 percent of all pairs.

Pitfalls: trusting the assistant's band settings untested, and huge buckets from near-constant embeddings (blank images), which need a cap.

Follow-ups: clustering pairs (transitive chains); dedup of new data against an existing 10 billion item set.

Why this is likely: Meta's hiring page lists optimizing existing code in AI-enabled rounds, and Meta's curation work removes duplicates at the scale of billions of image-text pairs. Medium confidence on the format, low on the exact task.

</details>

### Multimodal training, data and evals (AI and ML design rounds)

Meta runs design rounds in CoderPad with Mermaid and the same assistant, though candidates say it is often never used. For this team expect the design prompts to be about multimodal data, mixtures, architectures and evals, since the job is about training multimodal LLMs and owning their data.

<a id="q9"></a>

#### Q9. After a data-mixture change at step 40,000, the image-text loss spikes and the run goes to NaN by step 41,800. Read the log, name the likely root causes, the next three checks, and the fix.

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Trace debug. Difficulty: Hard. Topic: Training stability under data-mixture changes.

<details>
<summary>Show answer</summary>

```text
step   loss_text  loss_img_text  grad_norm  mix      max_img_tokens  frac_interleaved
39000  2.11       2.48           0.92       v7       1024            0.10
39500  2.10       2.47           0.95       v7       1024            0.10
40000  2.10       2.46           0.94       v8       4096            0.25
40200  2.10       2.71           1.80       v8       4096            0.25
40400  2.11       2.95           3.40       v8       4096            0.25
40600  2.12       2.62           1.10       v8       4096            0.25
40800  2.13       3.40           7.90       v8       4096            0.25
41000  2.16       3.90           12.5       v8       4096            0.25
41400  2.40       5.10           41.0       v8       4096            0.25
41800  nan        nan            inf        v8       4096            0.25
note: v8 adds source web_interleaved_v2 (long documents, up to 16 images each)
note: per-source loss at 40800: web_interleaved_v2=4.9, captions=2.5, ocr=2.3
warn: trainer v3 ignored unused config key "grad_clip" (expects "clip_grad_norm")
```

Task: Explain the divergence from the log, list what you would check next and in what order, and propose a restart plan.

What they grade: reading the evidence before guessing, separating the trigger from the amplifier, cheap checks before expensive ones, and a fix you can verify.

Approach: The trigger is the v8 mixture: interleaved data jumps from 10 to 25 percent in one step, image token budget per sequence goes from 1,024 to 4,096, and the new source has twice the loss of the others. The amplifier is that gradient clipping is off: the warning shows the config key was renamed, so grad norm climbs from 0.9 to 41 with nothing capping it. Text loss barely moves until late, which points at the image path, not the optimizer state as a whole. Checks, cheapest first: (1) confirm clipping is inactive by reading the trainer's resolved config; (2) replay the step 40,800 batches from the step 40,000 checkpoint and log loss per sample, to see whether a few documents dominate (for example, image placeholder counts that do not match the number of images, which misaligns embeddings); (3) log max attention logits per layer, since longer image runs can blow up logits without QK normalization. Fix: restart from step 40,000 with clipping restored, ramp the new source from 10 to 25 percent over about 2,000 steps, filter documents with placeholder mismatches or more than 8 images, and add a short LR warmup on the switch. Alternatives: auto-skip spiky batches (cheap, hides data bugs) or a lower learning rate for the whole run (safe, slower). I pick the targeted fix plus a spike detector.

```python
import numpy as np

steps = np.array([39000, 39500, 40000, 40200, 40400, 40600, 40800, 41000, 41400])
loss = np.array([2.48, 2.47, 2.46, 2.71, 2.95, 2.62, 3.40, 3.90, 5.10])
gnorm = np.array([0.92, 0.95, 0.94, 1.80, 3.40, 1.10, 7.90, 12.5, 41.0])

def first_spike(x, window=3, k=3.0):
    for i in range(window, len(x)):
        ref = x[i - window:i]
        mad = np.median(np.abs(ref - np.median(ref))) + 1e-3
        if (x[i] - np.median(ref)) / mad > k:
            return i
    return None

i = first_spike(gnorm)
assert steps[i] == 40200
share = {"web_interleaved_v2": 0.25, "captions": 0.45, "ocr": 0.30}
src_loss = {"web_interleaved_v2": 4.9, "captions": 2.5, "ocr": 2.3}
contrib = {s: share[s] * src_loss[s] for s in share}
worst = max(contrib, key=lambda s: contrib[s] / share[s])
assert worst == "web_interleaved_v2"
print("first spike at", steps[i], "worst source", worst)
```

Verify: after the restart, grad norm stays near its old level, the per-source loss of the new source falls, and a replay of the bad batches no longer spikes.

Pitfalls: restarting from a checkpoint after the spike began, and fixing clipping without finding the bad documents.

Follow-ups: catching the renamed key before launch; what a data-mixture canary run looks like.

Why this is likely: the job centers on training multimodal LLMs and running their data pipelines, a 2026 Meta Research Engineer report lists two AI design rounds, and a training-run debug is a natural prompt for a team that owns the data mixture. Low to medium confidence on the exact format.

</details>

<a id="q10"></a>

#### Q10. Design the evaluation for visual grounding in the new model: it returns a box for a referring phrase and a free-form grounded answer. Cover the metric, a contamination check against pretraining data, and how you calibrate an LLM judge.

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Concept plus code. Difficulty: Hard. Topic: Multimodal evaluation design.

<details>
<summary>Show answer</summary>

Task: Build an eval that tells the team whether a checkpoint grounds phrases better, that cannot be gamed by test images leaking into training, and whose free-text scoring is trustworthy.

What they grade: a metric that matches the product need, held-out data you can defend, judge calibration with numbers, and slices that find failures.

Approach: The eval has two parts. For box grounding, accuracy at IoU 0.5 is the standard headline, but I also report mean IoU and accuracy at 0.75 because 0.5 hides sloppy boxes. Slice by object size, number of same-class distractors, and phrase type (attribute, spatial relation, OCR text). Use about 3,000 items; with accuracy near 0.7, the 95 percent interval is plus or minus 1.6 points, enough to see a 3 point change. For contamination, hash every eval image with a perceptual hash and an embedding, and check them against the pretraining dedup index (Q8); also check 13-gram overlap of referring phrases with training captions. Report scores on the clean subset. For free-form answers, use an LLM judge with a rubric (correct object, correct location words, no invented objects). Calibrate it on 300 items labeled by two humans: report human-human agreement and judge-human agreement as Cohen's kappa, and only trust the judge if its kappa is close to the human ceiling. Alternatives: exact string match (too strict for free text), human rating only (too slow for every checkpoint), judge plus a monthly human audit (chosen). Public benchmarks are useful for comparison, but a held-out internal set is the one that guides decisions.

```python
import numpy as np

def iou(a, b):
    x1, y1 = max(a[0], b[0]), max(a[1], b[1])
    x2, y2 = min(a[2], b[2]), min(a[3], b[3])
    inter = max(0, x2 - x1) * max(0, y2 - y1)
    union = (a[2] - a[0]) * (a[3] - a[1]) + (b[2] - b[0]) * (b[3] - b[1]) - inter
    return inter / union if union else 0.0

def acc_at(preds, gts, thr):
    return float(np.mean([iou(p, g) >= thr for p, g in zip(preds, gts)]))

def kappa(a, b):
    a, b = np.asarray(a), np.asarray(b)
    po = np.mean(a == b)
    pe = sum(np.mean(a == c) * np.mean(b == c) for c in np.union1d(a, b))
    return (po - pe) / (1 - pe)

def ngrams(s, n=13):
    w = s.lower().split()
    return {tuple(w[i:i + n]) for i in range(len(w) - n + 1)}

assert iou((0, 0, 2, 2), (1, 1, 3, 3)) == 1 / 7
assert iou((0, 0, 1, 1), (2, 2, 3, 3)) == 0
preds = [(0, 0, 10, 10), (0, 0, 10, 10), (5, 5, 15, 15)]
gts = [(0, 0, 10, 10), (1, 1, 11, 11), (0, 0, 10, 10)]
assert acc_at(preds, gts, 0.5) == 2 / 3 and acc_at(preds, gts, 0.75) == 1 / 3
human = [1, 1, 0, 1, 0, 0, 1, 1, 0, 1]
judge = [1, 1, 0, 1, 1, 0, 1, 1, 0, 1]
assert round(kappa(human, judge), 3) == 0.783
cap = "the small red mug to the left of the laptop on the wooden desk near the window"
assert ngrams(cap) & ngrams("photo: " + cap)
print("ok")
```

Verify: IoU on overlapping and disjoint boxes, accuracy at two thresholds, kappa on a toy label set, and that the overlap check flags a caption copied into a training text.

Pitfalls: reporting only IoU 0.5, using a judge never compared with humans, and checking contamination by file name instead of content.

Follow-ups: grounding in video (temporal IoU); judge drift when the judge model is upgraded; what to do when the clean subset and the full set disagree.

Why this is likely: the job lists grounding as a core capability, Meta's PerceptionLM work publishes visual understanding benchmarks, and a 2026 Research Engineer report lists two AI design rounds, where a team that owns grounding is likely to ask how you would measure it. Medium confidence.

</details>

<a id="q11"></a>

#### Q11. Design the data curation system for pre-training, mid-training and post-training of a multimodal LLM, with dataset versioning so any checkpoint can be traced to the exact data it saw.

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Design. Difficulty: Hard. Topic: Multimodal data curation at scale.

<details>
<summary>Show answer</summary>

Task: Sources include web image-text pairs, interleaved documents, video with transcripts, OCR-heavy documents, licensed data and human-written post-training data. Design the stages, the storage, the versioning and the feedback loop to model quality.

What they grade: clear stage boundaries, numbers, how quality signals are chosen and validated, and lineage you can audit.

Approach: Use three tiers that share one framework. Pre-training: ingest, decode and validate, safety and legal filters, exact dedup by hash, near dedup by embedding (Q8), language ID, image-text alignment scoring, metadata balancing to flatten the head (Q6), then shard and tokenize. Mid-training: smaller, higher quality sets (long interleaved documents, charts, OCR, grounding data with boxes, video) selected by classifiers trained on curated seeds, plus decontamination against every eval set. Post-training: human and synthetic instruction data, preference pairs, and grounding tasks, with each synthetic generator versioned like a dataset. Every stage writes a manifest (Q7) with parent version ids, so a checkpoint stores one id and the full lineage can be walked back to raw sources. Storage is columnar tables for metadata and scores, plus object storage for media, so filters can be rerun from scores without decoding images again.

Numbers: 10 billion raw pairs, about 1 PB of images. Dedup and filters typically keep a fraction of that, say 2 to 3 billion pairs. Scores per item (about 20 floats) are about 1 TB in total, cheap enough to keep so thresholds can change later without recomputation.

Alternatives: one monolithic filter script (fast to start, no lineage, painful to audit) versus a stage graph with manifests (chosen). Filtering with a large VLM judge on every item is the most accurate but costs about 100 times a CLIP-score pass, so use the VLM only to label seed sets that train cheap classifiers.

Risks: filters that remove whole dialects or regions (audit kept and dropped rates per language), eval leakage, licensing and deletion requests that must propagate to every child version, and quality classifiers that reward a style rather than real quality, checked by small training ablations.

Why this is likely: the job lists pre, mid and post-training data curation, pipelines and versioning, Meta says it rebuilt data curation for Muse Spark, and a 2026 Research Engineer report lists two AI design rounds. Medium confidence.

</details>

<a id="q12"></a>

#### Q12. You have the compute for one large multimodal pretraining run and five candidate data mixtures. Plan the small-scale runs, fit a scaling law per mixture, and choose. Show the fit.

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Concept plus code. Difficulty: Hard. Topic: Scaling laws and data mixtures.

<details>
<summary>Show answer</summary>

Task: Pick the mixture that gives the lowest loss (and best downstream evals) at the target compute, using small runs that cost a few percent of the big one.

What they grade: why small-scale winners can lose at scale, a fit with an irreducible loss term, a budget for the ablation, and a decision rule that accounts for noise.

Approach: Train each mixture at 6 compute points spread over about three orders of magnitude (10^18 to 10^21 FLOPs), with the model and token counts following one fixed compute-optimal recipe so only the data changes. Fit L(C) = E + A * C^(-alpha) per mixture, where E is the irreducible loss. Fitting is a grid over E plus a linear fit of log(L - E) on log C. Extrapolate each curve to the target compute and pick the lowest, but only trust a gap larger than the spread you get from refitting with one point left out. Mixtures with a better exponent can lose at small scale and win at large scale; a scaling fit catches that crossover, and a single small ablation misses it. Measure loss on a fixed, held-out validation mix rather than each mixture's own data, and check two downstream evals at the largest small run. Budget: if the target run is 10^24 FLOPs and the largest ablation point is 10^21, five mixtures times six points costs under 1 percent of the target. Alternatives: pick by one small run (cheap, wrong when curves cross); a learned mixture method that reweights domains online (powerful, harder to debug); fits per mixture (chosen) as the baseline you trust.

```python
import numpy as np

def fit(C, L):
    best = None
    x = np.log(C)
    for E in np.linspace(0.5, L.min() - 1e-3, 2000):
        y = np.log(L - E)
        k, b = np.polyfit(x, y, 1)
        r = float(((y - (k * x + b)) ** 2).sum())
        if best is None or r < best[0]:
            best = (r, E, np.exp(b), -k)
    return best[1:]

def predict(p, C):
    E, A, a = p
    return E + A * C ** (-a)

rng = np.random.default_rng(0)
C = np.logspace(18, 21, 6)
truth = {"mix_A": (1.90, 30.0, 0.080), "mix_B": (1.70, 45.0, 0.085)}
target = 1e24
fits, preds = {}, {}
for name, p in truth.items():
    L = predict(p, C) * (1 + 0.002 * rng.standard_normal(len(C)))
    fits[name] = fit(C, L)
    preds[name] = predict(fits[name], target)
    assert abs(preds[name] - predict(p, target)) / predict(p, target) < 0.03
assert predict(truth["mix_A"], C[0]) < predict(truth["mix_B"], C[0])
assert min(preds, key=preds.get) == "mix_B"
print({k: round(v, 3) for k, v in preds.items()})
```

Verify: on synthetic curves with known parameters the fit predicts the target loss within 3 percent, and it picks mix_B even though mix_A wins at the smallest scale.

Pitfalls: validating each mixture on its own data, changing model shape between points, and reading a 0.5 percent gap as real.

Follow-ups: different exponents for image and text tokens; repeating scarce high-quality data for several epochs; whether lower loss predicts better grounding evals.

Why this is likely: Meta says it fit scaling laws on small models while rebuilding pretraining for Muse Spark, its Transfusion paper studies scaling across modalities, and the job owns pre-training data. Medium confidence.

</details>

<a id="q13"></a>

#### Q13. Choose an architecture for a model that must both understand and generate images: an adapter on a frozen vision encoder, early fusion with discrete image tokens, or one transformer trained with next-token and diffusion losses. Defend the choice with compute numbers.

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Design. Difficulty: Hard. Topic: Multimodal model architecture.

<details>
<summary>Show answer</summary>

Task: The product needs image understanding with grounding, image generation and editing, and interleaved outputs. Choose the backbone and say what you would ablate before committing.

What they grade: clear tradeoffs per option, numbers on tokens and compute, how the data pipeline changes with the choice, and a plan to de-risk it.

Approach: Adapter models (a pretrained vision encoder plus a projector into the LLM) are the cheapest route to strong understanding, since the encoder arrives pretrained and only the projector and LLM need multimodal training. They cannot generate images without bolting on a separate decoder, so they fail the generation requirement. Early fusion with discrete tokens (a VQ tokenizer, every image a sequence of codes, one next-token loss) gives one model for both directions, but quantization loses detail, and the published Transfusion comparison found that discrete image tokens scale worse than continuous ones. One transformer with next-token loss on text and a diffusion loss on continuous image patches keeps detail and shares one backbone. Mixture-of-Transformers adds modality-specific weights (feed-forward, attention projections, norms) with shared global attention; its paper reports matching a dense baseline with roughly half the FLOPs in an early-fusion setting. I would pick a Transfusion-style objective with modality-specific weights and validate it by ablation first.

Numbers: a 1024 by 1024 image at 16 pixels per patch is 4,096 tokens; a VAE that downsamples 8 times followed by 2 by 2 patches is also 4,096, while 8 by 8 latent patches give 256. At 7 billion parameters, training compute is about 6 times parameters times tokens, so each image costs about 1.1 times 10^13 FLOPs at 256 tokens and 16 times more at 4,096. Image token count drives most of the cost, so I would ablate 256 versus 1,024 tokens per image first.

Alternatives: adapter (cheapest, no generation), discrete early fusion (simplest objective, lossy), continuous fusion with modality-specific weights (chosen, more complex training).

Risks: balancing two losses (the diffusion loss weight needs a sweep), slower training throughput, and the data pipeline must now store image latents or patches at a fixed resolution, which changes curation (Q11).

Why this is likely: the job lists understanding, generation and grounding, Meta published Transfusion and Mixture-of-Transformers as multimodal training recipes, and a 2026 Research Engineer report lists two AI design rounds, whose team-specific framing is low confidence. Low to medium confidence.

</details>

<a id="q14"></a>

#### Q14. Estimate the compute, storage and wall-clock time to curate and tokenize 10 billion image-text pairs: decode, filter, embed, dedup, and write training shards. Where is the bottleneck?

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Design. Difficulty: Medium. Topic: Capacity planning for data pipelines.

<details>
<summary>Show answer</summary>

Task: Give an order-of-magnitude plan with stated assumptions that a hiring manager can check on a whiteboard, then say what you would measure first.

What they grade: explicit assumptions, correct arithmetic, finding the bottleneck, and a plan that survives a 2 times error in any one assumption.

Approach: Assumptions (to confirm with a 1 million pair benchmark): average JPEG 100 KB; CPU decode and resize at 200 images per second per core; a CLIP-style ViT embedding at 2,000 images per second per GPU; 768-dimensional embeddings in fp16; image tokenization at 256 tokens per image stored as 2-byte ids; 40 percent of pairs survive filters and dedup.

Numbers: raw images are 1 PB. Decode is 5 times 10^7 core-seconds, about 14,000 core-hours, so 7 hours on 2,000 cores. Embedding is 5 times 10^6 GPU-seconds, about 1,400 GPU-hours, so under 6 hours on 256 GPUs. Embeddings take 15 TB, so dedup cannot hold them all in one GPU's memory and must be sharded. Tokenized shards for 4 billion kept pairs are about 2 TB of image tokens plus captions. Reading 1 PB at 100 GB per second of aggregate storage throughput is about 3 hours. The bottleneck is CPU decode and data movement, not the GPUs, so decode once, write resized images or latents, and run every later filter on the stored embeddings and scores.

```python
pairs, kept_frac = 10e9, 0.4
raw_tb = pairs * 100e3 / 1e12
decode_core_h = pairs / 200 / 3600
embed_gpu_h = pairs / 2000 / 3600
emb_tb = pairs * 768 * 2 / 1e12
tok_tb = pairs * kept_frac * 256 * 2 / 1e12
read_h = raw_tb * 1e12 / 100e9 / 3600
assert raw_tb == 1000 and round(emb_tb, 2) == 15.36 and round(tok_tb, 2) == 2.05
assert 13000 < decode_core_h < 14000 and 1300 < embed_gpu_h < 1500
print(f"decode {decode_core_h / 2000:.1f} h on 2,000 cores, embed {embed_gpu_h / 256:.1f} h on 256 GPUs, read {read_h:.1f} h")
```

Alternatives: decode on GPU (fast, but the GPUs are needed for training), decode on CPU once and cache (chosen), or embed at lower resolution (cheaper, worse dedup and alignment scores).

Risks: a long tail of corrupt or huge images that stalls workers (add per-item timeouts), small-file overhead in object storage (pack into large shards), and recomputation when a filter changes, avoided by keeping scores.

Why this is likely: the job asks to scale data pipelines, and MetaCLIP 2 describes curation at tens of billions of seen pairs. Medium confidence.

</details>

<a id="q15"></a>

#### Q15. Design how a multimodal LLM improves ranking in an image feed: semantic IDs for items, where the LLM sits (offline features, retrieval, or ranking), and how you stay inside the serving latency budget.

Round: AI and ML design. AI policy: Provided (CoderPad assistant). Type: Design. Difficulty: Hard. Topic: LLMs in recommendation systems.

<details>
<summary>Show answer</summary>

Task: The feed ranks about 1,000 candidates per request in under 100 ms. Leadership wants the new multimodal model to make ranking better. Propose where it goes, how items are represented, and how you prove the gain.

What they grade: respecting latency and cost, semantic IDs explained by how they are built, staged rollout with offline and online metrics, and cold start.

Approach: Running a multi-billion parameter model on 1,000 candidates per request is out of budget: even at 5 ms per candidate on one GPU it is 5 seconds. So the model works mostly offline. Stage one: embed every item once with the multimodal model and feed the embeddings to the existing ranker as features; this helps cold-start items most, since they have no engagement history. Stage two: semantic IDs. Quantize each item embedding with residual quantization, for example 3 levels of 256 codes, which gives about 16.7 million distinct ids where similar items share prefixes. The ranker then learns embeddings per prefix, so a new item inherits signal from similar older items. Stage three: generative retrieval, where a smaller sequence model predicts the next semantic IDs from a user's history and adds candidates to retrieval. An LLM reranker on the top 20 is possible only if a distilled model fits in about 10 ms.

Numbers: 1 billion items times one embedding pass is a one-time batch job (about 140 GPU-hours at 2,000 items per second per GPU); fresh items embed in near real time. Semantic ID tables at 3 by 256 codes are tiny compared with per-item id tables.

Alternatives: per-item learned ids (strong for popular items, useless for new ones), raw embeddings as features (simple, no sharing of learned signal), semantic IDs (chosen for sharing plus cold start).

Risks: code collisions that merge unrelated items (monitor items per code), id churn when the encoder is retrained (version the codebook), and offline gains that do not move engagement, so run an A/B test with guardrails on time spent and creator diversity.

Why this is likely: a 2026 Meta Research Engineer report names LLMs in the ranking stage with semantic IDs and an image-feed timing problem as its design prompts. Medium confidence.

</details>

### Project deep dive and behavioral (no AI, as at Meta)

<a id="q16"></a>

#### Q16. Your curation change moved a 500-item visual QA benchmark from 61.0 to 62.4 percent. The hiring manager asks whether that is real. Show how you decide and what you would run before claiming the gain.

Round: Hiring manager deep dive or project retrospective. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Ablation rigor and statistical testing.

<details>
<summary>Show answer</summary>

Task: Decide whether a 1.4 point gain is signal or noise, using the per-item results of both checkpoints, and say what extra evidence would make you confident.

What they grade: a paired test (same items, two models), seed variance, honesty about what you cannot conclude, and a better experiment design.

Approach: 1.4 points on 500 items is 7 items. An unpaired interval on a single accuracy near 0.6 is about plus or minus 4.3 points, so the gain is inside the noise before any pairing. Paired analysis is sharper because most items are answered the same way by both models: only the discordant items matter. Run McNemar's exact test on the discordant counts and a paired bootstrap for the interval. With 25 items only the old model gets right and 32 only the new one gets right, the two-sided p-value is about 0.43, so I would not claim the gain. Before claiming it: run 3 training seeds per arm (seed variance on small benchmarks is often a point or more), add two more evals that the change should help, and check contamination, since a curation change can pull benchmark-like images into training. Alternatives: an unpaired test (too weak), a paired test on one seed (better, still ignores training noise), seeds plus paired tests across several evals (chosen).

```python
import math
import numpy as np

both, old_only, new_only, neither = 280, 25, 32, 163
old = np.array([1] * both + [1] * old_only + [0] * new_only + [0] * neither)
new = np.array([1] * both + [0] * old_only + [1] * new_only + [0] * neither)
assert old.mean() == 0.61 and new.mean() == 0.624

def mcnemar_exact(b, c):
    n, k = b + c, min(b, c)
    tail = sum(math.comb(n, i) for i in range(k + 1)) / 2 ** n
    return min(1.0, 2 * tail)

p = mcnemar_exact(old_only, new_only)
rng = np.random.default_rng(0)
idx = rng.integers(0, len(old), size=(5000, len(old)))
diffs = new[idx].mean(axis=1) - old[idx].mean(axis=1)
lo, hi = np.percentile(diffs, [2.5, 97.5])
assert 0.3 < p < 0.6 and lo < 0 < hi
print(f"p={p:.2f}, 95% CI for the gain: [{lo:.3f}, {hi:.3f}]")
```

Verify: the counts reproduce 61.0 and 62.4 percent; the exact test gives p near 0.43; the bootstrap interval contains 0.

Pitfalls: claiming a gain from one seed, choosing the best checkpoint by the same benchmark you report, and ignoring contamination.

Follow-ups: how many items you need to detect a 1 point gain (several thousand, depending on discordance); how you present a null result to a skeptical lead.

Why this is likely: 2026 Meta Research Engineer reports describe deep questions on current work and a technical deep dive on previous projects, and research loops include a project retrospective. Medium confidence.

</details>

<a id="q17"></a>

#### Q17. Tell me about a time you led a cross-functional data or model effort where AI-written code or AI output was wrong. How did you find it, and what did you change in the team's process?

Round: Behavioral. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Technical leadership, AI use and verification.

<details>
<summary>Show answer</summary>

Task: Tell one story in about 3 minutes that shows tech-lead scope (several teams, a decision you owned), concrete AI use, a real error, and a lasting change.

What they grade: scope that matches the level (the job asks for formal tech lead experience with cross-functional impact), specific numbers, ownership of the mistake, and judgment about where AI helps and where it needs checks.

Approach: Use situation, task, action, result, then a short reflection. A strong skeleton: you led a data-quality effort across a research team, a data platform team and a policy team to cut noisy image-text pairs before a pretraining run. You used an AI assistant to write the filter jobs and unit tests, which cut the build from about three weeks to one. A spot check of 200 dropped samples showed the generated language filter dropped most captions in two scripts because the assistant used a language-ID call with a default that treated low-confidence text as unknown and removed it. You caught it because your rule was to sample dropped and kept items per language before any filter shipped. You fixed the threshold, re-ran on the affected shards, and the dropped rate for those scripts fell from roughly 60 percent to 8 percent. The lasting change was that every filter now ships with per-slice drop-rate reports and a 200-item human audit, and AI-written data code needs a reviewer who did not prompt it.

Verify: be ready for "what would have happened if you had not caught it", "who disagreed and how did you resolve it", and "what do you still not delegate to AI".

Pitfalls: a story with no AI detail, blaming the tool instead of the process, a team-only result with no clear personal decision, and numbers you cannot defend under follow-up.

Follow-ups: how you set AI usage norms for the team; a time you were wrong about a technical call; how you handled a partner team that did not want to adopt your pipeline.

Why this is likely: a 2026 aggregated report sees an AI-oriented behavioral question in most Meta loops, Meta's hiring page says candidates are expected to use AI in many interviews, and the job requires leading initiatives with cross-functional impact. Medium confidence.

</details>

## Prep checklist

- [ ] Solve grid flood fill, island merging and expression parsing in 20 minutes each, out loud, with no AI and no code execution.
- [ ] Write BatchNorm2d forward and backward, a sparse dot product, and causal attention in NumPy from memory, with tests.
- [ ] Practise one multi-file CoderPad-style task: run the tests first, prompt the assistant, then review every line of its diff.
- [ ] Prepare one story where you caught a wrong AI output, with the check that caught it and the process change.
- [ ] Rehearse a multimodal data curation design with numbers for storage, decode throughput, GPU-hours and kept fraction.
- [ ] Read Meta's MetaCLIP 2, Transfusion and Mixture-of-Transformers papers and form one opinion you can defend on each.
- [ ] Be able to fit a scaling law and explain why small-scale mixture winners can lose at scale.
- [ ] Know how to design a grounding eval: IoU thresholds, contamination checks, and judge calibration with kappa.
- [ ] Prepare a 5-minute and a 30-minute deep dive on a pipeline or model you led, including one failed experiment.
- [ ] Practise a paired significance test and a bootstrap interval on per-item benchmark results.
- [ ] Ask the recruiter whether the AI coding round is assistant-enabled or ML coding, and whether there is a retrospective.

## Sources

- Meta careers, hiring process and FAQ on AI-enabled interviews: https://www.metacareers.com/hiring-process
- Meta, post on AI-enabled interviewing (company official): https://www.linkedin.com/posts/meta_metacareers-interviewing-softwareengineering-activity-7386046074240737280-wnwA
- Glassdoor, Meta Research Engineer interview reports: https://www.glassdoor.com/Interview/Meta-Research-Engineer-Interview-Questions-EI_IE40772.0,4_KO5,22.htm
- Reddit, Meta Research Engineer interview with 6 rounds: https://www.reddit.com/r/OfferEngineering/comments/1sx5hja/meta_research_engineer_interview_6_rounds_ml/
- 1point3acres, Meta interview trend report: https://www.1point3acres.com/interview/guides/meta-interview-trend-2026q4
- Blind, AI coding interviews at Meta AI (low confidence): https://www.teamblind.com/post/ai-coding-interviews-at-meta-ai-6z001dax
- Blind, Meta AI research EM interview thread (low confidence): https://www.teamblind.com/post/what-the-f-meta-ai-research-em-interview-ai-design-ai-research-design-f1ttrnh8
- IGotAnOffer, Meta Research Engineer interview guide (prep vendor, low confidence): https://igotanoffer.com/en/advice/meta-research-engineer-interview-guide
- Community prep repo, Meta AI interview questions (low confidence): https://github.com/ombharatiya/AI-Engineer-Interview-Questions/blob/main/14-company-interview-questions/meta-ai.md
- Deep-ML, batch norm problem named in a Research Engineer report: https://www.deep-ml.com/problems/115
- Meta AI, Introducing Muse Spark: https://ai.meta.com/blog/introducing-muse-spark-msl
- Muse Spark Open model card (training data and curation): https://huggingface.co/meta-models/Muse-Spark-Open-816B-A42B
- MetaCLIP 2 paper (worldwide metadata-balanced curation): https://arxiv.org/abs/2507.22062
- Transfusion paper: https://ai.meta.com/research/publications/transfusion-predict-the-next-token-and-diffuse-images-with-one-multi-modal-model
- PerceptionLM paper: https://ai.meta.com/research/publications/perceptionlm-open-access-data-and-models-for-detailed-visual-understanding
- Company-wise LeetCode lists (crowd-reported tags): https://github.com/liquidslr/leetcode-company-wise-problems
