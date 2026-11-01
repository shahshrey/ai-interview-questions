# Capital One: Applied Researcher 5 (AI Foundations, LLM Core and Agentic AI)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Capital One's AI and ML recruiters describe the loop on the company careers site, and Applied Researcher candidate reports fill in the details. Every round is virtual and on video. The hiring team shapes the technical content, so ask your recruiter which AI Foundations team you will meet.

1. Recruiter screen (about 30 minutes). The recruiter covers your background, the role and logistics, and there is no coding. AI: not relevant.
2. CodeSignal assessment (70 minutes, 4 questions). You solve data structures and algorithms problems on your own, proctored with a webcam. Capital One calls it pass/fail and gives you a window of days to start it, but once you start you finish in one sitting. Applied Researcher candidates report mostly simulation problems that reward careful implementation. AI: not allowed (proctored; no assistant reported).
3. Hiring manager pre-screen (30 minutes). The manager goes deep on your past research, asking what you owned, why you made each choice, and what the next step of the project would be. AI: not relevant.
4. Power Day, technical round (60 minutes). 30 minutes of live coding in CodeSignal (one candidate reported a medium binary tree problem), then 30 minutes of AI and ML depth tied to the team. For this team that means pre-training, training and inference optimization, fine-tuning and agentic AI. Expect to reason from first principles. AI: not allowed (no assistant reported for the AI and ML track).
5. Power Day, system design (60 minutes). Design an end-to-end system built on modern LLM concepts, slanted toward the team's work (inference, safety and guardrails, agents, or training). AI: not allowed (none reported).
6. Power Day, behavioral (60 minutes). STAR format with follow-ups on why you did things and what you personally contributed. Engineering candidates also report a question on how you use AI day to day and where you do not trust it. AI: not relevant.
7. Power Day, case study (60 minutes). A business problem built on a public Capital One product, often ending in reading or debugging code against a business rule without running it. Engineering-track reports name a customer chatbot and virtual card numbers most often. AI: unknown for this track; engineering Power Days now pair it with an agentic round where Claude Code is provided.
8. Possibly a research presentation. Some research candidates report presenting one project to two researchers, or discussing a pre-assigned LLM project. This depends on the team. AI: not relevant.

Capital One has added an agentic coding round with Claude Code to software engineering Power Days, where you fix a bug in a large codebase and then build a feature. Applied Researcher reports have yet to mention it, so ask your recruiter. Q3 below is a drill in that format in case it appears. All four Power Day rounds count equally.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | The heights of houses on a street are in an array. One operation raises one house by... | Classic algorithm | CodeSignal assessment | Not allowed | Medium |
| [Q2](#q2) | Given a binary tree with possibly negative values, return the maximum sum of any... | Classic algorithm | Power Day technical round, live coding | Not allowed | Hard |
| [Q3](#q3) | AI assistant allowed, as in Capital One's agentic round. You get a small fine-tuning... | Debug | Agentic drill (reported for engineering Power Days) | Provided (Claude Code in CodeSignal) | Medium |
| [Q4](#q4) | An AI assistant wrote this sampling service for the team's model server. Review it... | Review | Power Day technical round, code review | Unknown | Medium |
| [Q5](#q5) | You have to pre-train a 10B parameter model on 500B tokens. Estimate the compute... | Concept plus code | Power Day technical round, AI and ML depth | Not allowed | Medium |
| [Q6](#q6) | How much GPU memory does training that 10B model need, and how would you shard it?... | Concept plus code | Power Day technical round, AI and ML depth | Not allowed | Hard |
| [Q7](#q7) | Partway through pre-training the 10B model, a teammate pastes this log and asks... | Trace debug | Power Day technical round, AI and ML depth | Not allowed | Hard |
| [Q8](#q8) | When would you fine-tune all weights of a 10B model versus use LoRA? Implement a... | Build | Power Day technical round, AI and ML depth | Not allowed | Medium |
| [Q9](#q9) | Your team wants a 10B student to reason better on policy questions using teacher... | Build | Power Day technical round, AI and ML depth | Not allowed | Hard |
| [Q10](#q10) | Implement the DPO loss from sequence log-probabilities, and explain when you would... | Build | Power Day technical round, AI and ML depth | Not allowed | Medium |
| [Q11](#q11) | You need to serve the 10B model to many concurrent users. Size the KV cache, compare... | Optimize | Power Day technical round, AI and ML depth | Not allowed | Hard |
| [Q12](#q12) | The team wants a router that sends easy customer policy questions to the 10B model... | Design | Power Day system design | Not allowed | Hard |
| [Q13](#q13) | Design an LLM assistant that answers customer questions about their accounts and can... | Design | Power Day system design | Not allowed | Hard |
| [Q14](#q14) | How would you evaluate and red-team a multi-turn, tool-using agent before it touches... | Design | Power Day system design or AI and ML depth | Not allowed | Hard |
| [Q15](#q15) | Tell me about a research project you owned end to end. How did you use AI tools... | Behavioral | Power Day behavioral | Not allowed | Medium |

### CodeSignal assessment (70 minutes, no AI)

<a id="q1"></a>

#### Q1. The heights of houses on a street are in an array. One operation raises one house by 1. Return the minimum number of operations so the heights either increase by exactly 1 from left to right or decrease by exactly 1 from left to right (online assessment, no AI).

Round: CodeSignal assessment. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Greedy, arrays.
Related practice: [Minimum Number of Increments on Subarrays to Form a Target Array](https://leetcode.com/problems/minimum-number-of-increments-on-subarrays-to-form-a-target-array/) (Hard), [Alternating Groups II](https://leetcode.com/problems/alternating-groups-ii/) (Medium).

<details>
<summary>Show answer</summary>

Task: Return the fewest +1 raises that turn the array into a staircase that rises or falls by exactly 1 per house, in linear time.

Approach: Since you can only raise houses, the target array B must cover the input A everywhere: B[i] >= A[i]. An increasing staircase is fully described by its starting height c, so B[i] = c + i. The cost is the sum of B[i] minus A[i], which grows with c, so you want the smallest c that keeps every B[i] at or above A[i]. That is c = max(A[i] minus i). The decreasing case is the mirror image: B[i] = c minus i with c = max(A[i] plus i). Compute both totals and return the smaller one.

Picture the input as a bar chart and slide a staircase down until it touches the tallest bar relative to its own step. The area between the staircase and the bars is the number of operations.

```python
def min_ops_staircase(nums):
    if not nums:
        return 0
    c_up = max(a - i for i, a in enumerate(nums))
    up = sum(c_up + i - a for i, a in enumerate(nums))
    c_down = max(a + i for i, a in enumerate(nums))
    down = sum(c_down - i - a for i, a in enumerate(nums))
    return min(up, down)

def brute(nums):
    n, best = len(nums), None
    for c in range(-n, max(nums) + 2 * n + 1):
        for sign in (1, -1):
            b = [c + sign * i for i in range(n)]
            if all(x >= a for x, a in zip(b, nums)):
                cost = sum(x - a for x, a in zip(b, nums))
                best = cost if best is None else min(best, cost)
    return best

import random
random.seed(1)
assert min_ops_staircase([1, 4, 3, 2]) == 4   # decreasing 5,4,3,2
assert min_ops_staircase([5]) == 0
for _ in range(500):
    arr = [random.randint(0, 9) for _ in range(random.randint(1, 7))]
    assert min_ops_staircase(arr) == brute(arr), arr
print("ok")
```

Pitfalls: CodeSignal hides large test cases that time out a quadratic scan over start heights, and the closed form runs in O(n) time and O(1) extra space. Python integers do not overflow, but in Java or C++ use 64-bit sums, since n times the max height can pass 2 to the power 31.

Verify: talk through the brute force first in one sentence, then state the observation (cost is monotonic in c) that makes the closed form correct. On this assessment the grader only sees passing tests, so write the brute force as a local checker the way the code above does, run it on random inputs, and then submit the fast version. Edge cases to name: a single house, an already valid staircase, and all equal heights.

Why this is likely: first-hand Applied Researcher candidate report of the Capital One CodeSignal assessment, plus Capital One's own description of the assessment as four data structures and algorithms questions.

</details>

### Live coding (Power Day technical round, first 30 minutes, no AI)

<a id="q2"></a>

#### Q2. Given a binary tree with possibly negative values, return the maximum sum of any path, where a path is any sequence of connected nodes that does not reuse a node (Power Day live coding, no AI).

Round: Power Day technical round, live coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Hard. Topic: Binary tree, post-order traversal.
Related practice: [Binary Tree Paths](https://leetcode.com/problems/binary-tree-paths/) (Easy), [Find the Length of the Longest Common Prefix](https://leetcode.com/problems/find-the-length-of-the-longest-common-prefix/) (Medium).

<details>
<summary>Show answer</summary>

Task: Return the best path sum in a tree with negative values, iteratively, so a 5,000-deep tree does not hit the recursion limit.

Approach: At every node, two numbers matter. The first is the best downward path that starts at this node and continues into at most one child: node value plus the larger of the two child gains, where a negative gain is replaced by zero because you can stop. The second is the best path that bends at this node: node value plus both non-negative child gains. The answer is the maximum bend value over all nodes, and the parent only receives the downward value, because a path cannot fork twice.

Live rounds run in CodeSignal and the interviewer can paste a deep, skewed tree, so use an explicit stack for the post-order traversal instead of recursion, which would hit Python's recursion limit near 1,000 levels.

```python
class Node:
    def __init__(self, val, left=None, right=None):
        self.val, self.left, self.right = val, left, right

def max_path_sum(root):
    best = float("-inf")
    gain = {None: 0}
    stack = [(root, False)]
    while stack:
        node, seen = stack.pop()
        if node is None:
            continue
        if not seen:
            stack.append((node, True))
            stack.append((node.right, False))
            stack.append((node.left, False))
        else:
            left = max(gain[node.left], 0)
            right = max(gain[node.right], 0)
            best = max(best, node.val + left + right)
            gain[node] = node.val + max(left, right)
    return best

t = Node(-10, Node(9), Node(20, Node(15), Node(7)))
assert max_path_sum(t) == 42          # 15 + 20 + 7
assert max_path_sum(Node(-3)) == -3   # all negative: best single node
deep = Node(1)
cur = deep
for _ in range(5000):
    cur.left = Node(1)
    cur = cur.left
assert max_path_sum(deep) == 5001
print("ok")
```

Verify: time is O(n) and memory is O(n) for the gain map and stack. The asserts cover the worked example, an all-negative tree and a 5,000-deep chain.

Pitfalls: mention the two traps before the interviewer finds them: initializing the answer to zero breaks the all-negative case, and passing the bend value up to the parent creates a forked path. Then walk the small example by hand. The interviewer will drop hints and watch how you take feedback, which Capital One's recruiters say counts as much as finishing.

Why this is likely: an Applied Researcher candidate report describes a LeetCode-medium binary tree problem in the Power Day coding round, and Capital One's AI and ML recruiters describe 30 minutes of live CodeSignal coding in that round.

</details>

### Debug and review work

<a id="q3"></a>

#### Q3. AI assistant allowed, as in Capital One's agentic round. You get a small fine-tuning repo (collate.py, loss.py, test_collate.py). Tests pass, but a fine-tuned model never learns to stop generating. Find and fix the bug, then add response-only loss masking and truncation, and explain how you checked the assistant's changes.

Round: Agentic drill (reported for engineering Power Days). AI policy: Provided (Claude Code in CodeSignal). Type: Debug. Difficulty: Medium. Topic: Fine-tuning data pipeline, loss masking.

```python
# file: collate.py (as given, buggy), with the test_collate.py assert that passes on it
import numpy as np

def collate(examples, pad_id):
    seqs = [list(ex["ids"]) for ex in examples]
    batch, width = len(seqs), max(len(s) for s in seqs)
    input_ids = np.full((batch, width), pad_id, dtype=np.int64)
    labels = np.full((batch, width), pad_id, dtype=np.int64)
    for i, ids in enumerate(seqs):
        input_ids[i, :len(ids)] = ids
        labels[i, :len(ids)] = ids
    attention_mask = (input_ids != pad_id).astype(np.int64)
    return {"input_ids": input_ids, "attention_mask": attention_mask, "labels": labels}

# test_collate.py (as given): no EOS in the batch, so the bug stays invisible
PAD = 2
b = collate([{"ids": [5, 6, 7, 8]}, {"ids": [9, 10]}], pad_id=PAD)
assert b["input_ids"].tolist() == [[5, 6, 7, 8], [9, 10, PAD, PAD]]
assert b["attention_mask"].tolist() == [[1, 1, 1, 1], [1, 1, 0, 0]]
assert b["labels"].shape == b["input_ids"].shape
print("ok")
```

<details>
<summary>Show answer</summary>

Task: Find why the fine-tuned model never stops generating, fix it, add response-only loss masking and truncation, and show how you checked each assistant change.

Approach: Read the data path end to end yourself before you prompt the assistant. The collator builds input_ids, attention_mask and labels, and the loss shifts labels by one. Ask the assistant to summarize each file and list every place that compares against the pad id. Here the original collator built the mask with `ids != pad_id` and padded labels with `pad_id`. In this setup the pad id equals the end-of-sequence id, a common Llama-style choice, so every real EOS token was masked out of attention and loss. The model never saw a target that says "stop", which explains the symptom.

The fix is to build the mask from sequence lengths, never from token values, and to pad labels with an ignore index (-100). The new feature trains only on response tokens by setting labels before prompt_len to the ignore index, and truncation clips the sequence before the mask is built so the two can never disagree.

```python
# file: collate.py
import numpy as np

IGNORE = -100

def collate(examples, pad_id, max_len=None):
    seqs = []
    for ex in examples:
        ids = list(ex["ids"])[:max_len] if max_len else list(ex["ids"])
        seqs.append((ids, min(ex.get("prompt_len", 0), len(ids))))
    batch, width = len(seqs), max(len(s) for s, _ in seqs)
    input_ids = np.full((batch, width), pad_id, dtype=np.int64)
    attention_mask = np.zeros((batch, width), dtype=np.int64)
    labels = np.full((batch, width), IGNORE, dtype=np.int64)
    for i, (ids, p) in enumerate(seqs):
        n = len(ids)
        input_ids[i, :n] = ids
        attention_mask[i, :n] = 1          # from length, not token value
        labels[i, p:n] = ids[p:n]          # response tokens only
    return {"input_ids": input_ids, "attention_mask": attention_mask, "labels": labels}
```

```python
# file: loss.py
import numpy as np
from collate import IGNORE

def causal_lm_loss(logits, labels):
    logits, target = logits[:, :-1], labels[:, 1:]
    m = logits.max(-1, keepdims=True)
    logp = logits - m - np.log(np.exp(logits - m).sum(-1, keepdims=True))
    valid = target != IGNORE
    picked = np.take_along_axis(logp, np.where(valid, target, 0)[..., None], -1)[..., 0]
    return -(picked * valid).sum() / max(valid.sum(), 1)
```

```python
# file: test_collate.py
import numpy as np
from collate import collate, IGNORE as X
from loss import causal_lm_loss

EOS = PAD = 2
b = collate([{"ids": [5, 6, 7, 8, EOS], "prompt_len": 2},
             {"ids": [9, 10, EOS], "prompt_len": 1}], pad_id=PAD)
assert b["attention_mask"].tolist() == [[1, 1, 1, 1, 1], [1, 1, 1, 0, 0]]
assert b["labels"].tolist() == [[X, X, 7, 8, EOS], [X, 10, EOS, X, X]]
short = collate([{"ids": [5, 6, 7, 8, EOS], "prompt_len": 2}], pad_id=PAD, max_len=3)
assert short["labels"].tolist() == [[X, X, 7]]
logits = np.random.default_rng(0).normal(size=(2, 5, 12))
alone = collate([{"ids": [9, 10, EOS], "prompt_len": 1}], pad_id=PAD)
assert np.isclose(causal_lm_loss(logits[1:], b["labels"][1:]),
                  causal_lm_loss(logits[1:, :3], alone["labels"]))
print("ok")
```

Pitfalls: expect at least one wrong suggestion. Asked to fix the mask, the assistant first proposed `attention_mask = (ids != pad_id) | (ids == eos_id)`. It reads well in a diff, but with pad equal to EOS it marks every padding position as attended. The attention-mask assertion fails on it because the mask comes out all ones, so reject it and ask for a mask built from lengths.

Verify: run the checks in order: the new failing test, the full suite, then a 50-step training smoke run confirming the loss on EOS positions is no longer zero.

How to use the assistant: give it scoped tasks ("write a failing test where EOS equals PAD"), read every diff, and accept only changes to the files the task names. The mask assertion catches the attention bug, and the last assertion checks that padded labels use the ignore index. This numpy loss reads only the labels, so to catch mask bugs through the loss, run a real model forward pass. Say out loud what you accept and why, because reports say interviewers grade how you direct and verify the AI more than whether you finish.

Why this is likely: candidate reports describe a new Capital One Power Day round with Claude Code in a large codebase (fix a bug, then build a feature) for engineering roles; it has not been reported for Applied Researcher loops, so treat this as a precaution.

</details>

<a id="q4"></a>

#### Q4. An AI assistant wrote this sampling service for the team's model server. Review it before merge: list every defect, rank them by severity, and write the fixed version with tests that would have caught each one.

Round: Power Day technical round, code review. AI policy: Unknown. Type: Review. Difficulty: Medium. Topic: LLM sampling, serving security.

```python
# sampler.py, generated by an AI assistant (review only, do not merge)
import numpy as np

rng = np.random.default_rng()

def sample_next(logits, temperature=0.7, top_p=0.9):
    z = logits / temperature
    p = np.exp(z - z.max())
    p = p / p.sum()
    order = np.argsort(-p)
    csum = np.cumsum(p[order])
    keep = order[csum <= top_p]
    q = p[keep] / p[keep].sum()
    return int(keep[rng.categorical(q)])

def handle(request, model_logits):
    params = eval(request["params"])
    out = []
    for _ in range(params["max_tokens"]):
        out.append(sample_next(model_logits(request["prompt"], out), **params["sampling"]))
    return out
```

<details>
<summary>Show answer</summary>

Task: Review AI-written sampling code, rank every defect by severity, and ship a fixed version with one test per defect.

Approach: Read it like a colleague's pull request and rank the defects by how much damage each one could do.

1. Security, highest severity. `eval(request["params"])` runs any string a caller sends, so any client can execute code on the GPU host. Parse JSON and validate types and ranges.
2. Unbounded resource use. max_tokens comes straight from the request, so one call can hold a GPU slot for a billion steps. Cap it on the server, here at 1,024.
3. Off-by-one in top-p. `csum <= top_p` drops the token that crosses the threshold. When the model is confident, the kept set is empty. With logits [5, 1, 0, -1] at temperature 0.7 the top token alone has probability 0.996, so the division gives NaN and the request fails on the easiest inputs. Keep tokens through the first index where the cumulative sum reaches p.
4. Hallucinated API. NumPy's Generator has no categorical method, so the first call raises AttributeError, which also tells you nobody ran this code. Use `rng.choice(len(q), p=q)`.
5. Missing edge case. Temperature 0, which product teams set for consistency, divides by zero. Treat it as greedy argmax.

Also flag the shared module-level generator with no seed parameter, which makes tests irreproducible, and the missing stop on the end-of-sequence token.

```python
import json
import numpy as np

MAX_TOKENS = 1024

def sample(logits, temperature=1.0, top_p=1.0, rng=None):
    rng = rng or np.random.default_rng()
    if temperature == 0:
        return int(np.argmax(logits))                     # greedy, no division
    z = logits / temperature
    p = np.exp(z - z.max())
    p /= p.sum()
    order = np.argsort(-p)
    cutoff = int(np.searchsorted(np.cumsum(p[order]), top_p)) + 1
    keep = order[:cutoff]                                 # includes the token that crosses p
    q = p[keep] / p[keep].sum()
    return int(keep[rng.choice(len(keep), p=q)])

def parse_params(raw):
    params = json.loads(raw)                              # data, never code
    t = float(params.get("temperature", 0.7))
    top_p = float(params.get("top_p", 0.9))
    n = int(params.get("max_tokens", 256))
    if not (0 <= t <= 2 and 0 < top_p <= 1 and 1 <= n <= MAX_TOKENS):
        raise ValueError("sampling parameters out of range")
    return {"temperature": t, "top_p": top_p}, n

rng = np.random.default_rng(0)
peaked = np.array([5.0, 1.0, 0.0, -1.0])                 # top token has p > 0.9
assert all(sample(peaked, 0.7, 0.9, rng) == 0 for _ in range(200))
assert sample(peaked, temperature=0) == 0
flat = np.zeros(4)
assert len({sample(flat, 1.0, 0.5, rng) for _ in range(2000)}) == 2
for bad in ('{"max_tokens": 10**9}', '__import__("os").system("id")', '{"max_tokens": 100000}'):
    try:
        parse_params(bad)
        raise AssertionError(bad)
    except ValueError:
        pass
print(parse_params('{"temperature": 0, "max_tokens": 64}'))
print("ok")
```

For the fix, rejecting bad parameters with an error beats silently clamping them, because clamping hides client bugs. A schema library such as pydantic gives the same validation in fewer lines but adds a dependency to the serving path; either is fine if every rule has a test.

Verify: each assertion above maps to one defect, which is what the reviewer wants to see.

Why this is likely: a Capital One applied research candidate reported about twelve questions on inference parameters and how they change outputs, and Capital One's Power Day case round and new agentic round both ask candidates to read and fix code they did not write.

</details>

### Pre-training at scale

<a id="q5"></a>

#### Q5. You have to pre-train a 10B parameter model on 500B tokens. Estimate the compute, the GPU hours, and say whether that token count is compute-optimal and why a bank might choose it anyway.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Pre-training compute, scaling laws.

<details>
<summary>Show answer</summary>

Task: Estimate FLOPs and GPU hours for 10B parameters on 500B tokens, and judge the token count against compute-optimal scaling.

Approach: Use the standard estimate that training a dense transformer costs about 6 FLOPs per parameter per token: 2 for the forward pass and 4 for the backward pass. That gives 6 times 10B times 500B, or 3e22 FLOPs. An H100 delivers about 989 dense BF16 teraFLOPs at peak. Well-tuned large runs reach roughly 35 to 50 percent model FLOPs utilization (MFU), so plan around 40 percent.

```python
N, D = 10e9, 500e9
flops = 6 * N * D
peak, mfu = 989e12, 0.40
gpu_hours = flops / (peak * mfu) / 3600
print(f"compute {flops:.1e} FLOPs, {gpu_hours:,.0f} H100 hours")
for gpus in (256, 512, 1024):
    print(f"{gpus} GPUs: {gpu_hours / gpus:.0f} hours of wall clock before overheads")

# Chinchilla: compute-optimal tokens are about 20 per parameter.
n_opt = (flops / (6 * 20)) ** 0.5
print(f"tokens per param here: {D / N:.0f}; compute-optimal N for this budget: {n_opt / 1e9:.1f}B "
      f"on {20 * n_opt / 1e9:.0f}B tokens")
assert abs(flops - 3e22) < 1e18
```

Numbers: The output is about 21,000 H100 hours, roughly 41 hours on 512 GPUs. Add 10 to 30 percent for restarts, evaluation, checkpoint writes and lost nodes. Chinchilla's rule of thumb is about 20 tokens per parameter, which for this budget means about a 16B model on about 316B tokens. The plan here uses 50 tokens per parameter, so it is over-trained relative to compute-optimal.

Alternatives: That is usually the right call for a deployed model. Chinchilla minimizes training compute, but a bank pays for inference on every customer interaction for years. A smaller model trained longer costs less per token to serve, fits on fewer GPUs, and is easier to fine-tune and audit. Llama-style models push far past 20 tokens per parameter for this reason. State the limits too. Quality gains flatten as tokens per parameter rise, and high-quality domain text is finite, so repeating data past a few epochs gives diminishing returns.

Follow-ups: how MFU differs from hardware FLOPs utilization (activation recomputation adds FLOPs that MFU does not count), and how you would check this estimate with a small-scale loss curve before committing the cluster.

Why this is likely: the job listing prefers candidates who were on a team that trained a large language model from scratch at 10B plus parameters and 500B plus tokens, and Capital One's recruiters say pre-training teams ask pre-training questions.

</details>

<a id="q6"></a>

#### Q6. How much GPU memory does training that 10B model need, and how would you shard it? Cover optimizer state, activations, gradient checkpointing and the parallelism layout.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Training memory, FSDP, checkpointing.

<details>
<summary>Show answer</summary>

Task: Size model state and activation memory for a 10B training run and choose a sharding and checkpointing layout.

Approach: Split memory into model state and activations. With BF16 mixed precision and Adam, each parameter needs 2 bytes of BF16 weights, 2 bytes of gradients, 4 bytes of FP32 master weights, and 8 bytes for the two Adam moments: 16 bytes in total. A 10B model therefore carries about 160 GB of state before a single activation, which already exceeds one 80 GB GPU.

```python
L, h, V = 48, 4096, 128_000
params = 12 * L * h * h + V * h               # blocks plus embeddings
state_gb = params * 16 / 1e9
print(f"params {params / 1e9:.1f}B, model state {state_gb:.0f} GB")
for shards in (8, 64):
    print(f"FSDP/ZeRO-3 over {shards} GPUs: {state_gb / shards:.1f} GB each")

s, b = 4096, 1                                # sequence length, micro-batch
per_layer = 34 * s * b * h                    # bytes, flash attention (no s^2 term)
no_ckpt = per_layer * L / 1e9
full_ckpt = (2 * s * b * h * L + per_layer) / 1e9
print(f"activations without checkpointing {no_ckpt:.1f} GB, with full checkpointing {full_ckpt:.1f} GB")
assert 9.5e9 < params < 10.5e9
```

Numbers: Activations follow Korthikanti and colleagues: about 34 times s times b times h bytes per layer once flash attention removes the s-squared attention matrix. That comes to about 27 GB across 48 layers at sequence length 4096. Full activation checkpointing stores only each layer's input (2 times s times b times h bytes) and recomputes the rest during the backward pass, which drops this to about 2 GB at the cost of roughly one extra forward pass, about 33 percent more compute.

Alternatives: For the layout at 10B, FSDP (ZeRO-3) alone is usually enough. Sharding state across 64 GPUs leaves about 2.5 GB each, plus the temporarily gathered weights of one layer. Add tensor parallelism of 2 to 8 inside a node, over NVLink, only if the micro-batch activations or sequence length force it. Pipeline parallelism is rarely worth its bubble at this size. Prefer selective checkpointing (recompute only the cheap, memory-heavy pieces) over full checkpointing when memory allows, because it recovers most of the MFU.

Name the knobs you would tune in order: micro-batch size, checkpointing granularity, FSDP prefetch and overlap of communication with compute, and gradient accumulation to reach the global batch size the learning-rate schedule assumes.

Why this is likely: the job listing's optimization track names training parallelism and partitioning design, gradient checkpointing and optimizing training for a 10B plus model, and the AI and ML depth half of the technical round follows the team's focus.

</details>

<a id="q7"></a>

#### Q7. Partway through pre-training the 10B model, a teammate pastes this log and asks whether to restart. Diagnose it from the trace and say what you change.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Pre-training instability.

<details>
<summary>Show answer</summary>

Task: Decide from the training log whether this is a bad batch or an optimization instability, then pick the restart point and the fix.

```text
step    loss    grad_norm  max_attn_logit  lr
41000   2.112   0.31       38.2            3.0e-4
41100   2.109   0.33       41.0            3.0e-4
41200   2.107   0.35       44.9            3.0e-4
41300   2.106   0.41       49.6            3.0e-4
41400   2.104   0.52       55.3            3.0e-4
41500   2.108   0.88       62.0            3.0e-4
41600   2.131   1.00 (clipped)  70.4       3.0e-4
41700   2.402   1.00 (clipped)  83.1       3.0e-4
41800   3.917   1.00 (clipped)  97.5       3.0e-4
41900   7.880   1.00 (clipped)  nan        3.0e-4
```

Approach: Read the columns before the loss. Loss is flat and healthy through step 41500, but the max attention logit has climbed every 100 steps since 41000, from 38 to 62, and the gradient norm rose from 0.31 to 0.88 before it hit the clip at 1.0. The loss reacts only at 41700, and the logits overflow to NaN at 41900. A bad batch looks different: a one-step jump in loss with normal logits before it, and recovery when you skip that batch. This trace is optimization instability driven by attention logit growth. Gradient clipping from step 41600 limits each update and leaves the cause in place.

Known optimization causes in large transformers are attention logits that grow without bound (fixed with QK layer norm), output logits drifting large (fixed with an auxiliary z-loss that pulls the log partition function toward zero, as in PaLM), and Adam's second-moment estimate lagging for rare tokens, which is why large runs lower beta2 to about 0.95. Use warmup, gradient clipping at 1.0, and a learning rate that a small proxy model showed to be stable.

```python
import numpy as np

def logsumexp(x):
    m = x.max(-1, keepdims=True)
    return (m + np.log(np.exp(x - m).sum(-1, keepdims=True)))[..., 0]

def z_loss(logits, coef=1e-4):
    return coef * np.mean(logsumexp(logits) ** 2)

def spike_steps(losses, window=50, k=4.0):
    """Flag steps whose loss exceeds a rolling median by k robust standard deviations."""
    flagged = []
    for t in range(window, len(losses)):
        hist = np.asarray(losses[t - window:t])
        med = np.median(hist)
        mad = np.median(np.abs(hist - med)) * 1.4826 + 1e-8
        if losses[t] > med + k * mad:
            flagged.append(t)
    return flagged

rng = np.random.default_rng(0)
calm = rng.normal(0, 1, (4, 8))
drifting = calm + 40.0                        # same softmax, huge partition function
assert np.isclose(logsumexp(calm) - logsumexp(drifting), -40).all()
print("z-loss calm vs drifting:", round(z_loss(calm), 5), round(z_loss(drifting), 3))

loss = list(3.0 - 0.001 * np.arange(400) + rng.normal(0, 0.01, 400))
loss[300] += 0.8
assert spike_steps(loss) == [300]
print("spike at", spike_steps(loss))
```

Verify: The z-loss example shows why it helps. Adding 40 to every logit leaves the softmax unchanged, so cross entropy cannot see the drift, while the z-loss term grows sharply and pushes it back.

Pitfalls: choose the restart point from the logit column, because clipping kept the loss calm while the logits climbed. To recover, restart from step 41000, before the logits started climbing, and turn on QK layer norm. If the architecture is frozen, lower the learning rate by about 30 percent and set beta2 to 0.95 instead. Add an alert when the max attention logit rises for five straight logging intervals. Write down every intervention, because the final model's behavior has to be explainable later. In a regulated bank that record is part of model risk documentation.

Why this is likely: the job listing prefers candidates who pre-trained a 10B+ model from scratch, and Capital One's AI/ML recruiters say the depth half of the technical round follows the team's focus (pre-training here); no candidate report names this exact question, so treat it as an inference from the role.

</details>

### Fine-tuning, distillation and alignment

<a id="q8"></a>

#### Q8. When would you fine-tune all weights of a 10B model versus use LoRA? Implement a LoRA linear layer and count the trainable parameters.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Fine-tuning, LoRA.

<details>
<summary>Show answer</summary>

Task: Implement a LoRA linear layer that starts at the base model and merges exactly, count its trainable parameters, and choose between LoRA and full fine-tuning.

Approach: LoRA freezes the pretrained weight W and learns a low-rank update: the layer computes W x plus (alpha over r) times B A x, with A of shape r by d_in and B of shape d_out by r. B starts at zero, so training begins exactly at the base model. Only A and B get gradients and optimizer state. That cuts the 16 bytes per parameter of Adam state to almost nothing for the frozen weights, and you can keep one base model in memory and swap small adapters per task. After training you can merge B A into W, so inference has no extra latency.

```python
import numpy as np

class LoRALinear:
    def __init__(self, W, r=16, alpha=32, seed=0):
        rng = np.random.default_rng(seed)
        self.W = W                                        # frozen, shape (d_out, d_in)
        self.A = rng.normal(0, 1 / np.sqrt(W.shape[1]), (r, W.shape[1]))
        self.B = np.zeros((W.shape[0], r))
        self.scale = alpha / r

    def __call__(self, x):                                # x: (n, d_in)
        return x @ self.W.T + self.scale * (x @ self.A.T) @ self.B.T

    def merged(self):
        return self.W + self.scale * self.B @ self.A

rng = np.random.default_rng(1)
layer = LoRALinear(rng.normal(size=(64, 32)))
x = rng.normal(size=(5, 32))
assert np.allclose(layer(x), x @ layer.W.T)               # B = 0: identical to base
layer.B = rng.normal(size=layer.B.shape) * 0.01
assert np.allclose(layer(x), x @ layer.merged().T)        # merge is exact

L, h, kv, r = 48, 4096, 1024, 16                          # GQA: k and v project to 1024
shapes = [(h, h), (kv, h), (kv, h), (h, h)]               # q, k, v, o
per_layer = sum(r * (o + i) for o, i in shapes)
total = per_layer * L
print(f"LoRA r={r} on attention: {total / 1e6:.1f}M trainable, {100 * total / 10.2e9:.2f}% of 10.2B")
```

Numbers: At rank 16 on the four attention projections, that is about 20M trainable parameters, about 0.2 percent of the model.

Alternatives: Choose LoRA when you need many task-specific variants, compute is limited, or the target behavior is narrow (format, tone, a classification head expressed as text). Choose full fine-tuning, or LoRA on every linear layer at higher rank, when you are teaching new knowledge or a new domain at scale, such as continued pre-training on financial text. Low-rank updates underfit large distribution shifts, and studies of LoRA find it learns less and forgets less than full fine-tuning. In a bank, forgetting less is often a feature, but measure it on held-out general benchmarks either way.

Why this is likely: the job listing's fine-tuning track asks for parameter tuning, model adaptation and deploying a fine-tuned large language model.

</details>

<a id="q9"></a>

#### Q9. Your team wants a 10B student to reason better on policy questions using teacher critiques. Build one training example and its loss masking for critique-guided distillation, and argue for it against plain distillation and critique fine-tuning.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Distillation, reasoning.

<details>
<summary>Show answer</summary>

Task: Build one critique-guided distillation example with loss on the refined answer only, and argue for it against plain distillation and critique fine-tuning.

Alternatives: Plain supervised distillation trains a student to reproduce a teacher's correct answers. The student copies the surface form of the reasoning without learning why wrong reasoning is wrong, so it generalizes poorly on hard problems. Critique fine-tuning (CFT) instead trains the student to write critiques of flawed answers. That teaches error awareness, but it moves the student toward producing critiques. The paper reports output-format drift and a large drop in instruction following (21.3 percent on IFEval) compared with the base behavior.

Approach: Critique-guided distillation (CGD) separates using a critique from writing one. Each training example has four parts: the question, a flawed student response, a teacher critique of that response, and a refined correct response. The student sees the first three as context and is trained to produce only the refined response. Critiques act as a supervision signal during training and are absent at inference, where the student answers the question directly. The paper reports about 7 percent average gains over CFT and standard distillation on math reasoning, with up to 15 points on AMC23, while keeping instruction following intact.

```python
IGNORE = -100

def cgd_example(question, flawed, critique, refined, tok):
    parts = [("Question: " + question, False),
             ("Draft: " + flawed, False),
             ("Critique: " + critique, False),
             ("Refined: " + refined, True)]          # only this part is trained
    ids, labels = [], []
    for text, train in parts:
        t = tok(text)
        ids += t
        labels += t if train else [IGNORE] * len(t)
    return ids, labels

vocab = {}
tok = lambda s: [vocab.setdefault(w, len(vocab)) for w in s.split()]
ids, labels = cgd_example("2+3*4?", "20", "multiplication binds first", "3*4=12 then 2+12 = 14", tok)
trained = [i for i, l in zip(ids, labels) if l != IGNORE]
assert len(ids) == len(labels)
assert trained == tok("Refined: 3*4=12 then 2+12 = 14")
print(f"{len(trained)} of {len(ids)} tokens carry loss")
```

Verify: To evaluate it, report Pass@1 and Pass@k on held-out competition math (CGD reports bigger gains at low k, meaning each sample is better rather than just more varied), plus an instruction-following benchmark to show general ability did not degrade. Ablate the critique (refined answer without critique context) to show the critique, not just the extra refined data, causes the gain. For a bank, the same recipe works for policy reasoning, where a critique explains why a draft answer violates a disclosure rule.

Why this is likely: Capital One researchers presented critique-guided distillation at a main machine learning conference, and the AI Foundations LLM team that owns this research is the hiring team, so expect questions on the team's own papers.

</details>

<a id="q10"></a>

#### Q10. Implement the DPO loss from sequence log-probabilities, and explain when you would choose DPO over PPO-style RLHF for a customer-facing model.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Preference optimization, RLHF.

<details>
<summary>Show answer</summary>

Task: Implement a numerically stable DPO loss with the diagnostics you would log, and choose between DPO and online RL.

Approach: DPO starts from the KL-regularized RLHF objective, whose optimal policy is the reference policy reweighted by the exponential of the reward over beta. Inverting that gives an implicit reward of beta times log(pi over pi_ref). Plugging it into a Bradley-Terry preference model makes the partition function cancel. The result is a classification loss on preference pairs: the negative log sigmoid of beta times the difference between the chosen and rejected log ratios. No reward model or sampling loop is needed.

```python
import numpy as np

def dpo_loss(pc, pr, rc, rr, beta=0.1):
    """pc/pr: policy log p(chosen)/log p(rejected) summed over response tokens; rc/rr: reference."""
    margin = beta * ((pc - rc) - (pr - rr))
    loss = np.logaddexp(0.0, -margin)                 # -log sigmoid, stable
    return loss.mean(), {
        "reward_chosen": float((beta * (pc - rc)).mean()),
        "reward_rejected": float((beta * (pr - rr)).mean()),
        "pref_accuracy": float((margin > 0).mean()),
    }

rng = np.random.default_rng(0)
rc, rr = rng.normal(-50, 5, 64), rng.normal(-50, 5, 64)
loss0, _ = dpo_loss(rc, rr, rc, rr)
assert np.isclose(loss0, np.log(2))                   # policy == reference
loss1, m = dpo_loss(rc + 5, rr - 5, rc, rr)
assert loss1 < loss0 and m["pref_accuracy"] == 1.0
loss2, m2 = dpo_loss(rc - 20, rr - 30, rc, rr)        # both fall: still "wins"
print(round(loss1, 4), m2)
```

Pitfalls: The last case is the failure to watch for. The policy can lower the probability of both responses and still win if the rejected one falls faster. Chosen log-probabilities then go down during training (likelihood displacement), and probability mass leaks to responses nobody labeled. Log both implicit rewards next to the loss and accuracy. Fixes include adding an SFT term on chosen responses, raising beta, or using a length-normalized variant when chosen responses are systematically longer.

Alternatives: Choose DPO when you have a fixed offline preference set, limited infrastructure, and a target like tone, refusal style or format. Choose online RL (PPO or GRPO with a reward model or verifiable reward) when the behavior needs exploration, such as multi-step agent tasks or math with checkable answers, or when the preference data comes from an older policy and has gone stale. For a bank, the reward model or verifier is also an audit artifact you can test on its own.

Why this is likely: the job listing names RLHF expertise and instruction or dialogue fine-tuning, and Capital One's research site lists reinforcement learning from human feedback under its LLM and agentic AI focus area.

</details>

### Inference, routing and evaluation

<a id="q11"></a>

#### Q11. You need to serve the 10B model to many concurrent users. Size the KV cache, compare BF16, INT8 and INT4 weights, and show why per-channel quantization matters.

Round: Power Day technical round, AI and ML depth. AI policy: Not allowed. Type: Optimize. Difficulty: Hard. Topic: Inference, KV cache, quantization.

<details>
<summary>Show answer</summary>

Task: Size the KV cache and weights for serving the 10B model, and show with code why per-channel quantization beats per-tensor.

Approach: Weights scale with precision: 10.2B parameters take about 20 GB in BF16, 10 GB in INT8, and 5 GB in INT4. At long context and high concurrency the KV cache dominates. Each token stores a key and a value per layer, of width number-of-KV-heads times head dimension. With grouped-query attention (8 KV heads of 128 dimensions, versus 32 query heads), the cache is four times smaller than full multi-head attention.

```python
import numpy as np

L, kv_heads, head_dim, bytes_per = 48, 8, 128, 2
per_token = 2 * L * kv_heads * head_dim * bytes_per
print(f"KV per token {per_token / 1024:.0f} KiB; 4k context {per_token * 4096 / 2**20:.0f} MiB; "
      f"32 users x 4k {32 * per_token * 4096 / 2**30:.0f} GiB")
for name, b in (("bf16", 2), ("int8", 1), ("int4", 0.5)):
    print(f"weights {name}: {10.2e9 * b / 1e9:.1f} GB")

def quantize_int8(W, per_channel):
    amax = np.abs(W).max(axis=1, keepdims=True) if per_channel else np.abs(W).max()
    scale = amax / 127.0
    q = np.clip(np.round(W / scale), -127, 127)
    return q * scale

rng = np.random.default_rng(0)
W = rng.normal(0, 0.02, (256, 512))
W[7] *= 50                                            # one output channel with large weights
err = {pc: np.abs(quantize_int8(W, pc) - W)[np.arange(256) != 7].mean() for pc in (False, True)}
print(f"mean abs error on normal rows: per-tensor {err[False]:.2e}, per-channel {err[True]:.2e}")
assert err[True] < err[False] / 10
```

Numbers: That is 192 KiB per token, 768 MiB for one 4,096-token conversation, and 24 GiB for 32 of them. On an 80 GB GPU the cache, not the weights, sets the batch size. That is why paged attention (vLLM) and KV-cache quantization to INT8 or FP8 pay off.

The quantization example shows the outlier problem. One channel with large weights stretches a single per-tensor scale, so every other weight rounds coarsely. Per-channel scales fix weights. Activations have the same issue with outlier feature dimensions, which is why SmoothQuant moves scale from activations into weights and why AWQ and GPTQ protect salient channels when going to INT4. Weight-only INT4 helps decode, which is memory-bandwidth bound. Prefill is compute bound, so it gains more from FP8 matmuls.

Verify: Validate quantized models on the full task suite as well as perplexity, and check numbers and long-context retrieval most closely, because they degrade first. For a bank, a wrong digit in a balance is a material error.

Why this is likely: the job listing's optimization track names quantization, model compression and inference optimization, and Capital One's research team wrote up LLM serving efficiency as a leading theme from a recent systems conference.

</details>

<a id="q12"></a>

#### Q12. The team wants a router that sends easy customer policy questions to the 10B model and hard ones to a frontier model that costs 12 times more per query. Design the offline evaluation that decides whether the router ships and at what threshold.

Round: Power Day system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Evaluation design, model routing.

<details>
<summary>Show answer</summary>

Task: Define the ship gate, the data, the grader and the statistics that decide whether the router ships and at which threshold.

Approach: Fix the decision before the metric. The router ships if it cuts cost while quality on held-out traffic stays within 1 point of always using the large model, at 95 percent confidence, and no critical slice (fees and rates, disputes, account security) regresses at all.

For data, sample 2,000 recent, redacted questions stratified by intent, and deduplicate them against every fine-tuning and router training set by n-gram overlap, because leakage inflates the small model most. Have two annotators label a 200-item subset to settle the rubric.

Alternatives: For grading, exact match fails because good answers vary in wording, and human grading of every item at every threshold costs weeks. An LLM judge with a rubric (grounded in the cited policy, numbers correct, right refusal) is the scalable middle option, but only after you validate it. Require Cohen's kappa of at least 0.7 against the human labels, and test it for a bias toward longer answers or toward its own model family.

For statistics, both systems answer the same questions, so bootstrap the paired per-question difference. That needs far fewer items than comparing two independent accuracies, because most questions are right under both systems.

```python
import numpy as np

rng = np.random.default_rng(0)
n = 2000                                             # held-out, labeled policy questions
hard = rng.random(n)                                 # latent difficulty
small_ok = rng.random(n) > 0.6 * hard**3               # 10B model
large_ok = rng.random(n) > 0.25 * hard**3              # frontier model
router = hard + rng.normal(0, 0.15, n)               # router's difficulty score
COST_SMALL, COST_LARGE = 1.0, 12.0

def paired_ci(diff, B=2000):
    idx = rng.integers(0, len(diff), (B, len(diff)))
    return np.percentile(diff[idx].mean(1), [2.5, 97.5])

def kappa(a, b):
    po = (a == b).mean()
    pe = a.mean() * b.mean() + (1 - a.mean()) * (1 - b.mean())
    return (po - pe) / (1 - pe)

rows = []
for t in np.linspace(-0.4, 1.2, 33):
    to_small = router < t
    correct = np.where(to_small, small_ok, large_ok)
    lo, hi = paired_ci(correct.astype(float) - large_ok)
    cost = np.where(to_small, COST_SMALL, COST_LARGE).mean() / COST_LARGE
    rows.append((t, to_small.mean(), correct.mean(), lo, cost))

ship = min((r for r in rows if r[3] >= -0.01), key=lambda r: r[4])
print(f"large only: acc {large_ok.mean():.3f}, cost 1.00")
print(f"ship t={ship[0]:.2f}: {ship[1]:.0%} to 10B, acc {ship[2]:.3f}, "
      f"CI low {ship[3]:+.3f}, cost {ship[4]:.2f}")

human = rng.random(200) < 0.8                        # 200 double-labeled items
judge = np.where(rng.random(200) < 0.9, human, ~human)
print(f"judge vs human kappa {kappa(human, judge):.2f}")
assert ship[3] >= -0.01 and ship[4] < 0.9
```

Numbers: In this simulation, threshold 0.40 sends 40 percent of traffic to the 10B model, cuts cost by 37 percent, and keeps the lower bound of the quality change above minus 1 point. Threshold 0.45 already fails the gate. Report per-slice results next to the headline, rerun monthly on fresh traffic because the intent mix drifts, and send judge and human disagreements to review.

Why this is likely: Capital One researchers presented work on routing queries across models at a recent language conference, and Capital One's AI and ML recruiters describe a system design round on full end-to-end LLM infrastructure.

</details>

### LLM system design and agent safety

<a id="q13"></a>

#### Q13. Design an LLM assistant that answers customer questions about their accounts and can take simple actions, such as locking a card. Cover retrieval, tools, guardrails, latency and evaluation.

Round: Power Day system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: LLM system design, RAG, tools, guardrails.

<details>
<summary>Show answer</summary>

Task: Design an account assistant that answers from approved sources and takes confirmed actions, within a chat latency budget, with an evaluation plan.

Approach: Start with requirements, because the case and design rounds reward asking a second layer of questions. Which actions are allowed? What latency target applies (for example, first token under one second in chat)? What must never happen? Leaking another customer's data, inventing a fee or rate, and acting without confirmation are the obvious ones.

Alternatives: For the architecture, an orchestrator receives the authenticated session, so identity comes from the bank's auth system and never from the model. A small, fast router classifies the request (policy question, account lookup, action, out of scope or escalation). Policy questions go to retrieval-augmented generation over approved, versioned documents: hybrid BM25 plus dense retrieval, a reranker, and answers that must cite chunk IDs. Account data comes only from tools that call internal APIs with the customer's token, so the model never sees data the customer could not see. State-changing actions, such as locking a card, run through a typed tool schema, a deterministic policy check, and an explicit confirmation step shown to the user. The model proposes and the system executes. Two simpler designs are worth naming. One agent calling account APIs with broad credentials gives up per-customer access control, and policy text fine-tuned into the model skips retrieval latency but goes stale whenever a policy changes. The orchestrator design keeps both auditable.

Risks: Layer the guardrails. Screen inputs for prompt injection, including injection embedded in retrieved documents or transaction memos. Check that every number in the answer matches a tool result or a cited document, redact PII in logs, and hand off to a human agent on low confidence or repeated failure.

Numbers: For latency, retrieval and reranking take about 100 to 200 ms, tools 100 to 300 ms in parallel, and generation streams. Keep the router and guard models small, cache embeddings, and use prefix caching for the long system prompt. A 10B-class model is usually enough for the main answer when retrieval carries the facts, and it serves far cheaper than a frontier model.

Verify: For evaluation, build an offline test set of real (redacted) intents with gold answers and gold tool calls. Score groundedness, numeric exactness, correct refusals and over-refusals, and run adversarial suites. Online, run a shadow deployment, then a small percentage rollout with human review of sampled conversations, tracking containment, escalation rate, complaints and any incorrect action. Model risk management in a bank needs documented evaluation and monitoring, so plan the evaluation harness as a deliverable from the start.

Why this is likely: Capital One's AI and ML recruiters say the Power Day system design round asks for a full end-to-end system built on modern LLM concepts, and software engineering candidates and a prep-vendor summary name customer chatbots among Power Day case topics (low confidence for research roles).

</details>

<a id="q14"></a>

#### Q14. How would you evaluate and red-team a multi-turn, tool-using agent before it touches customer accounts? Include how you would find attacks automatically.

Round: Power Day system design or AI and ML depth. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Agent evaluation, automated red teaming.

<details>
<summary>Show answer</summary>

Task: Plan the evaluation and an automated red-teaming loop for a multi-turn, tool-using agent before it can act on accounts.

Approach: Evaluate whole multi-turn trajectories, because many agent failures only appear several turns in. A benign request becomes a harmful action after context builds up, or an injected instruction in a tool result takes effect three calls later. Define the threat model (malicious user, malicious content in retrieved or tool data, and honest user plus model error) and the harmful outcomes (unauthorized action, data disclosure, policy-violating advice). Then build an environment with mocked tools so the agent can act without touching real accounts.

Verify: Measure attack success rate per outcome type with confidence intervals, task success on benign tasks, and over-refusal, because an agent that refuses everything looks safe and is useless. Add trajectory-level risk signals. Estimate the agent's uncertainty at each step (for example, disagreement across sampled plans, or a calibrated probe), aggregate it over the trajectory, and use it to pause and ask for confirmation. Capital One researchers have published on uncertainty in multi-turn agents.

Alternatives: For automated red teaming, static jailbreak lists go stale and random combination wastes queries. Treat attack generation as a bandit: each arm is a composition of instructions (persona, obfuscation, goal framing), and the reward is whether the attack succeeded. That balances finding the strongest strategies with keeping coverage. Capital One's adaptive instruction composition work builds on the same idea with a neural contextual bandit.

```python
import numpy as np

def ucb_red_team(success_rates, budget=3000, c=1.0, seed=0):
    rng = np.random.default_rng(seed)
    k = len(success_rates)
    pulls, wins = np.zeros(k), np.zeros(k)
    for t in range(1, budget + 1):
        if t <= k:
            arm = t - 1
        else:
            arm = int(np.argmax(wins / pulls + c * np.sqrt(2 * np.log(t) / pulls)))
        pulls[arm] += 1
        wins[arm] += rng.random() < success_rates[arm]
    return pulls, wins

true_rates = [0.01, 0.02, 0.15, 0.03, 0.05]           # unknown to the red team
pulls, wins = ucb_red_team(true_rates)
print("pulls per strategy:", pulls.astype(int).tolist(), "attacks found:", int(wins.sum()))
assert pulls.argmax() == 2 and pulls.min() > 20
```

UCB spends most queries on the strongest strategy but still tries the others, so the report covers several failure modes. Feed successful attacks back as training data for refusal and tool-use policies, then rerun the whole suite to check for regressions and over-refusal.

Why this is likely: Capital One researchers published work on multi-turn agent uncertainty and on automated LLM red teaming with an adaptive bandit, and the job title names agentic AI.

</details>

### Behavioral

<a id="q15"></a>

#### Q15. Tell me about a research project you owned end to end. How did you use AI tools along the way, and how did you check their output?

Round: Power Day behavioral. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Research ownership, AI use.

<details>
<summary>Show answer</summary>

Task: Tell one owned research story in STAR form, built around your choices, your part, and how you used and checked AI tools.

Approach: Capital One's recruiters want to hear why you made each choice and which parts were yours, and they say candidates most often stumble on those two points. They also look for a second layer of questions, such as asking whether another approach was considered. Structure the answer as STAR, but spend most of the time on Action and on the choices behind it.

Situation and task (about 30 seconds). One sentence on the problem and why it mattered, with a baseline number. Example: "Our fine-tuned model hallucinated fee amounts in about 6 percent of answers on an internal test set, and the product team could not launch above 1 percent."

Action (about 2 minutes). Say what you decided and why. Mention the options you rejected: "I considered adding more SFT data but chose a grounding check plus preference tuning because the errors were confident ones." Describe what you built yourself and what you delegated. Then answer the AI part concretely. Name where AI tools sped you up (scaffolding evaluation scripts, drafting data filters, summarizing related papers) and how you checked each one. You wrote unit tests before accepting generated code. You spot-checked 100 model-labeled examples by hand and measured agreement. You verified every cited paper claim against the paper itself. Also give one place you chose to work without the tool, such as the evaluation metric definition, because a subtle error there would have invalidated every later result.

Pitfalls: say "I" for your own work, give numbers, name the options you rejected, and say exactly where AI helped and how you checked it.

Result (about 30 seconds). Give the metric change, say what shipped, and say what you would do differently. A real limitation you found and fixed makes the story more credible than a perfect outcome.

Follow-ups: prepare for "What would the next step of this project be?" Hiring managers for these roles report asking about next steps, so have a concrete plan with the experiment you would run first. Keep two backup stories ready, one about a failed experiment and one about disagreeing with a senior stakeholder, because Capital One weights behavioral rounds equally with technical ones.

Why this is likely: Capital One's AI and ML recruiters describe STAR behavioral interviews that probe ownership and the why behind decisions, hiring managers for these roles report asking about project next steps, and engineering candidates report a new question on how they use AI and where they do not trust it.

</details>

## Prep checklist

- [ ] Ask your recruiter which AI Foundations team you will meet and whether any round is AI-assisted.
- [ ] Do two timed sets of four CodeSignal-style problems in 70 minutes, including one long simulation problem, without an AI assistant.
- [ ] Practice one 30-minute live tree or graph problem out loud, including an iterative version for deep inputs.
- [ ] Rehearse the compute and memory math for a 10B model on 500B tokens until you can do it without notes.
- [ ] Be ready to explain pre-training instability fixes: QK norm, z-loss, beta2, warmup, clipping and checkpoint rewinds.
- [ ] Review LoRA versus full fine-tuning, DPO derivation and failure modes, and when online RL is worth its cost.
- [ ] Read the team's recent papers on critique-guided distillation, multi-turn agent uncertainty, LLM routing and automated red teaming.
- [ ] Prepare an end-to-end design for a customer-facing banking assistant, with guardrails, latency budget and evaluation plan.
- [ ] Run one AI-assisted drill: a bug fix and a feature in an unfamiliar repo, reviewing every change the assistant makes.
- [ ] Practice a case built on a public Capital One product: business benefits first, then trace and fix code against a rule without running it.
- [ ] Prepare three STAR stories with numbers, including how you use AI tools and check their output.
- [ ] Prepare a 10-minute talk on one project, in case your team runs a research presentation.

## Sources

- https://www.capitalonecareers.com/insider-tips-for-ai-and-machine-learning-interviews-tech-101-cdev
- https://www.capitalonecareers.com/what-to-expect-during-your-capital-one-interview-students-101
- https://www.glassdoor.com/Interview/Capital-One-Applied-Researcher-Interview-Questions-EI_IE3736.0,11_KO12,30.htm
- https://prachub.com/interview-experiences/capital-one-applied-researcher-interview-experience-codesignal-oa-with-a-tetris-style-block-placement-problem
- https://www.interviewquery.com/guides/capital-one-ai-research-scientist
- https://prachub.com/interview-experiences/capital-one-new-grad-software-engineer-interview-experience-power-day-with-behavioral-case-study-and-agentic-coding
- https://prachub.com/interview-experiences/capital-one-intern-software-engineer-interview-experience-superday-with-a-case-study-and-an-ai-assisted-coding-round
- https://www.calibrd.com/interview-prep/capital-one-interview
- https://capitalone.science/research
- https://capitalone.science/blog/llm-reasoning-and-agentic-safety-at-icml-2026
- https://capitalone.science/blog/capital-one-at-acl-2026
- https://capitalone.science/blog/highlights-from-mlsys-2026
- https://arxiv.org/abs/2505.11628
- https://arxiv.org/html/2604.21159v1
- https://arxiv.org/abs/2203.15556
- https://arxiv.org/abs/2205.05198
- https://arxiv.org/abs/1910.02054
- https://arxiv.org/abs/2204.02311
- https://arxiv.org/abs/2302.05442
- https://arxiv.org/abs/2106.09685
- https://arxiv.org/abs/2305.18290
- https://arxiv.org/abs/2211.10438
- https://arxiv.org/abs/2309.06180
- https://arxiv.org/abs/1904.09751
