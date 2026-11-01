# Capital One: Applied Researcher 5 (AI Foundations, LLM Core and Agentic AI)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Capital One's AI and ML recruiters describe the loop on the company careers site, and Applied Researcher candidate reports fill in the details. Every round is virtual and on video. The hiring team shapes the technical content, so ask your recruiter which AI Foundations team you will meet.

1. Recruiter screen (about 30 minutes). The recruiter covers your background, the role and logistics, and there is no coding.
2. CodeSignal assessment (70 minutes, 4 questions). You solve data structures and algorithms problems on your own, proctored with a webcam. Capital One calls it pass/fail and gives you a window of days to start it, but once you start you finish in one sitting. Applied Researcher candidates report mostly simulation problems that reward careful implementation.
3. Hiring manager pre-screen (30 minutes). The manager goes deep on your past research, asking what you owned, why you made each choice, and what the next step of the project would be.
4. Power Day, technical round (60 minutes). 30 minutes of live coding in CodeSignal (one candidate reported a medium binary tree problem), then 30 minutes of AI and ML depth tied to the team. For this team that means pre-training, training and inference optimization, fine-tuning and agentic AI. Expect to reason from first principles.
5. Power Day, system design (60 minutes). Design an end-to-end system built on modern LLM concepts, slanted toward the team's work (inference, safety and guardrails, agents, or training).
6. Power Day, behavioral (60 minutes). STAR format with follow-ups on why you did things and what you personally contributed. Engineering candidates also report a question on how you use AI day to day and where you do not trust it.
7. Power Day, case study (60 minutes). A business problem built on a public Capital One product, often ending in reading or debugging code against a business rule without running it. A customer chatbot and virtual card numbers are the most reported cases.
8. Possibly a research presentation. Some research candidates report presenting one project to two researchers, or discussing a pre-assigned LLM project. This depends on the team.

Capital One has added an agentic coding round with Claude Code to software engineering Power Days, where you fix a bug in a large codebase and then build a feature. Applied Researcher reports have yet to mention it, so ask your recruiter. Q4 below is a drill in that format in case it appears. All four Power Day rounds count equally.

## Questions and answers

### CodeSignal assessment (unassisted, 70 minutes)

#### Q1. The heights of houses on a street are in an array. One operation raises one house by 1. Return the minimum number of operations so the heights either increase by exactly 1 from left to right or decrease by exactly 1 from left to right.

Since you can only raise houses, the target array B must cover the input A everywhere: B[i] >= A[i]. An increasing staircase is fully described by its starting height c, so B[i] = c + i. The cost is the sum of B[i] minus A[i], which grows with c, so you want the smallest c that keeps every B[i] at or above A[i]. That is c = max(A[i] minus i). The decreasing case is the mirror image: B[i] = c minus i with c = max(A[i] plus i). Compute both totals and return the smaller one.

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

This runs in O(n) time and O(1) extra space. That matters because CodeSignal hides large test cases that time out quadratic solutions. Python integers do not overflow, but in Java or C++ use 64-bit sums, since n times the max height can pass 2 to the power 31.

Talk through the brute force first in one sentence, then state the observation (cost is monotonic in c) that makes the closed form correct. On this assessment the grader only sees passing tests, so write the brute force as a local checker the way the code above does, run it on random inputs, and then submit the fast version. Edge cases to name: a single house, an already valid staircase, and all equal heights.

Why this is likely: first-hand Applied Researcher candidate report of the Capital One CodeSignal assessment, plus Capital One's own description of the assessment as four data structures and algorithms questions.

#### Q2. Five block shapes arrive in a given order. Place each one on an m by n board at the first position where it fits, scanning rows top to bottom and columns left to right, without overlapping earlier blocks. Return the final board.

This is a simulation problem, and the work is writing correct, readable code quickly. Represent each shape as a list of (row, column) offsets from its top-left anchor. For each arriving block, scan anchors in row-major order, check that every offset lands inside the board on an empty cell, and stamp the letter into those cells. The problem guarantees a solution exists, so you can stop at the first fit.

```python
SHAPES = {
    "A": [(0, 0)],                          # single cell
    "B": [(0, 0), (0, 1)],                  # horizontal pair
    "C": [(0, 0), (1, 0)],                  # vertical pair
    "D": [(0, 0), (0, 1), (1, 0), (1, 1)],  # 2 by 2 square
    "E": [(0, 0), (1, 0), (1, 1)],          # L shape
}


def place_blocks(m, n, order):
    board = [["."] * n for _ in range(m)]

    def fits(r, c, cells):
        return all(0 <= r + dr < m and 0 <= c + dc < n and board[r + dr][c + dc] == "."
                   for dr, dc in cells)

    for letter in order:
        cells = SHAPES[letter]
        placed = False
        for r in range(m):
            for c in range(n):
                if fits(r, c, cells):
                    for dr, dc in cells:
                        board[r + dr][c + dc] = letter
                    placed = True
                    break
            if placed:
                break
        if not placed:
            raise ValueError("no room for " + letter)
    return ["".join(row) for row in board]


out = place_blocks(4, 4, "DBEAC")
assert out == ["DDBB", "DDEA", "C.EE", "C..."], out
print("\n".join(out))
```

The cost is O(k times m times n times s) for k blocks with s cells each, which is fine for interview sizes. If a hidden test is large, keep a pointer to the first row that still has an empty cell and start each scan there, or store each row as a bitmask so a fit check becomes a few AND operations.

The IDE may block pasting, so keep the shape table compact and write one helper (fits) rather than repeating bounds checks per shape. State your anchor convention out loud, because "smallest row, then smallest column" refers to the anchor, and a shape whose top-left cell is empty (an upside-down L) needs a different anchor rule.

Why this is likely: first-hand Applied Researcher candidate report of a Tetris-style block placement question on the Capital One CodeSignal assessment, and the same candidate noted the assessment favors simulation over heavy algorithms.

### Live coding (Power Day technical round, first 30 minutes)

#### Q3. Given a binary tree with possibly negative values, return the maximum sum of any path, where a path is any sequence of connected nodes that does not reuse a node.

At every node, two numbers matter. The first is the best downward path that starts at this node and continues into at most one child: node value plus the larger of the two child gains, where a negative gain is replaced by zero because you can stop. The second is the best path that bends at this node: node value plus both non-negative child gains. The answer is the maximum bend value over all nodes, and the parent only receives the downward value, because a path cannot fork twice.

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

Time is O(n) and memory is O(n) for the gain map and stack. Mention the two traps before the interviewer finds them: initializing the answer to zero breaks the all-negative case, and passing the bend value up to the parent creates a forked path. Then walk the small example by hand. The interviewer will drop hints and watch how you take feedback, which Capital One's recruiters say counts as much as finishing.

Why this is likely: an Applied Researcher candidate report describes a LeetCode-medium binary tree problem in the Power Day coding round, and Capital One's AI and ML recruiters describe 30 minutes of live CodeSignal coding in that round.

### AI-assisted coding drill (in case your Power Day uses the agentic format)

#### Q4. You get a small fine-tuning repo with an AI assistant. Tests pass, but a fine-tuned model never learns to stop generating. Find and fix the bug, then add response-only loss masking and truncation, and explain how you checked the assistant's changes.

Start by reading the code yourself before you prompt. Read the data path end to end. The collator builds input_ids, attention_mask and labels, and the loss shifts labels by one. Ask the assistant to summarize each file and list every place that compares against the pad id. Here the original collator built the mask with `ids != pad_id` and padded labels with `pad_id`. In this setup the pad id equals the end-of-sequence id, a common Llama-style choice, so every real EOS token was masked out of attention and loss. The model never saw a target that says "stop", which explains the symptom.

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

How to use the assistant: give it scoped tasks ("write a failing test where EOS equals PAD"), read every diff, and reject changes that touch unrelated files. The most important check is the last assertion, which proves padding does not change a sequence's loss. Say out loud what you accept and why, because reports say interviewers grade how you direct and verify the AI more than whether you finish.

Why this is likely: candidate reports describe a new Capital One Power Day round with Claude Code in a large codebase (fix a bug, then build a feature) for engineering roles; it has not been reported for Applied Researcher loops, so treat this as a precaution.

### Pre-training at scale

#### Q5. You have to pre-train a 10B parameter model on 500B tokens. Estimate the compute, the GPU hours, and say whether that token count is compute-optimal and why a bank might choose it anyway.

Use the standard estimate that training a dense transformer costs about 6 FLOPs per parameter per token: 2 for the forward pass and 4 for the backward pass. That gives 6 times 10B times 500B, or 3e22 FLOPs. An H100 delivers about 989 dense BF16 teraFLOPs at peak. Well-tuned large runs reach roughly 35 to 50 percent model FLOPs utilization (MFU), so plan around 40 percent.

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

The output is about 21,000 H100 hours, roughly 41 hours on 512 GPUs. Add 10 to 30 percent for restarts, evaluation, checkpoint writes and lost nodes. Chinchilla's rule of thumb is about 20 tokens per parameter, which for this budget means about a 16B model on about 316B tokens. The plan here uses 50 tokens per parameter, so it is over-trained relative to compute-optimal.

That is usually the right call for a deployed model. Chinchilla minimizes training compute, but a bank pays for inference on every customer interaction for years. A smaller model trained longer costs less per token to serve, fits on fewer GPUs, and is easier to fine-tune and audit. Llama-style models push far past 20 tokens per parameter for this reason. The limits to state: quality gains flatten as tokens per parameter rise, and high-quality domain text is finite, so repeating data past a few epochs gives diminishing returns.

Follow-ups to expect: how MFU differs from hardware FLOPs utilization (activation recomputation adds FLOPs that MFU does not count), and how you would check this estimate with a small-scale loss curve before committing the cluster.

Why this is likely: the job listing prefers candidates who were on a team that trained a large language model from scratch at 10B plus parameters and 500B plus tokens, and Capital One's recruiters say pre-training teams ask pre-training questions.

#### Q6. How much GPU memory does training that 10B model need, and how would you shard it? Cover optimizer state, activations, gradient checkpointing and the parallelism layout.

Split memory into model state and activations. With BF16 mixed precision and Adam, each parameter needs 2 bytes of BF16 weights, 2 bytes of gradients, 4 bytes of FP32 master weights, and 8 bytes for the two Adam moments: 16 bytes in total. A 10B model therefore carries about 160 GB of state before a single activation, which already exceeds one 80 GB GPU.

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

Activations follow Korthikanti and colleagues: about 34 times s times b times h bytes per layer once flash attention removes the s-squared attention matrix. That comes to about 27 GB across 48 layers at sequence length 4096. Full activation checkpointing stores only each layer's input (2 times s times b times h bytes) and recomputes the rest during the backward pass, which drops this to about 2 GB at the cost of roughly one extra forward pass, about 33 percent more compute.

For the layout at 10B, FSDP (ZeRO-3) alone is usually enough. Sharding state across 64 GPUs leaves about 2.5 GB each, plus the temporarily gathered weights of one layer. Add tensor parallelism of 2 to 8 inside a node, over NVLink, only if the micro-batch activations or sequence length force it. Pipeline parallelism is rarely worth its bubble at this size. Prefer selective checkpointing (recompute only the cheap, memory-heavy pieces) over full checkpointing when memory allows, because it recovers most of the MFU.

Name the knobs you would tune in order: micro-batch size, checkpointing granularity, FSDP prefetch and overlap of communication with compute, and gradient accumulation to reach the global batch size the learning-rate schedule assumes.

Why this is likely: the job listing's optimization track names training parallelism and partitioning design, gradient checkpointing and optimizing training for a 10B plus model, and the AI and ML depth half of the technical round follows the team's focus.

#### Q7. Partway through pre-training, the loss spikes and sometimes diverges. How do you diagnose it and what do you change?

First decide whether the spike is data or optimization. Log the loss per batch along with the global gradient norm, the update-to-weight ratio per layer, the maximum attention logit, and the output logit norm. If the spike lines up with one batch and the run recovers when you skip it, look at that batch: duplicated junk, very long repeated tokens, or a broken shard. If gradient norms and attention logits climb for hundreds of steps first, it is optimization instability.

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

The z-loss example shows why it helps. Adding 40 to every logit leaves the softmax unchanged, so cross entropy cannot see the drift, while the z-loss term grows sharply and pushes it back.

To recover, restart from the last good checkpoint a few hundred steps before the spike, skip the offending data range, and lower the learning rate or add the missing stabilizer. Write down every intervention, because the final model's behavior has to be explainable later. In a regulated bank that record is part of model risk documentation.

Why this is likely: the job listing asks for experience building large deep learning models and training optimization, and diagnosing pre-training instability is a standard depth question for teams that train from scratch.

### Fine-tuning, distillation and alignment

#### Q8. When would you fine-tune all weights of a 10B model versus use LoRA? Implement a LoRA linear layer and count the trainable parameters.

LoRA freezes the pretrained weight W and learns a low-rank update: the layer computes W x plus (alpha over r) times B A x, with A of shape r by d_in and B of shape d_out by r. B starts at zero, so training begins exactly at the base model. Only A and B get gradients and optimizer state. That cuts the 16 bytes per parameter of Adam state to almost nothing for the frozen weights, and you can keep one base model in memory and swap small adapters per task. After training you can merge B A into W, so inference has no extra latency.

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

At rank 16 on the four attention projections, that is about 20M trainable parameters, about 0.2 percent of the model.

Choose LoRA when you need many task-specific variants, compute is limited, or the target behavior is narrow (format, tone, a classification head expressed as text). Choose full fine-tuning, or LoRA on every linear layer at higher rank, when you are teaching new knowledge or a new domain at scale, such as continued pre-training on financial text. Low-rank updates underfit large distribution shifts, and studies of LoRA find it learns less and forgets less than full fine-tuning. In a bank, forgetting less is often a feature, but measure it on held-out general benchmarks either way.

Why this is likely: the job listing's fine-tuning track asks for parameter tuning, model adaptation and deploying a fine-tuned large language model.

#### Q9. Capital One researchers proposed critique-guided distillation. Explain the problem it solves, how its training data differs from plain distillation and critique fine-tuning, and write the loss masking for one example.

Plain supervised distillation trains a student to reproduce a teacher's correct answers. The student copies the surface form of the reasoning without learning why wrong reasoning is wrong, so it generalizes poorly on hard problems. Critique fine-tuning (CFT) instead trains the student to write critiques of flawed answers. That teaches error awareness, but it moves the student toward producing critiques. The paper reports output-format drift and a large drop in instruction following (21.3 percent on IFEval) compared with the base behavior.

Critique-guided distillation (CGD) separates using a critique from writing one. Each training example has four parts: the question, a flawed student response, a teacher critique of that response, and a refined correct response. The student sees the first three as context and is trained to produce only the refined response. Critiques act as a supervision signal during training and are absent at inference, where the student answers the question directly. The paper reports about 7 percent average gains over CFT and standard distillation on math reasoning, with up to 15 points on AMC23, while keeping instruction following intact.

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

To evaluate it, report Pass@1 and Pass@k on held-out competition math (CGD reports bigger gains at low k, meaning each sample is better rather than just more varied), plus an instruction-following benchmark to show general ability did not degrade. Ablate the critique (refined answer without critique context) to show the critique, not just the extra refined data, causes the gain. For a bank, the same recipe works for policy reasoning, where a critique explains why a draft answer violates a disclosure rule.

Why this is likely: Capital One researchers presented critique-guided distillation at a main machine learning conference, and the AI Foundations LLM team that owns this research is the hiring team, so expect questions on the team's own papers.

#### Q10. Implement the DPO loss from sequence log-probabilities, and explain when you would choose DPO over PPO-style RLHF for a customer-facing model.

DPO starts from the KL-regularized RLHF objective, whose optimal policy is the reference policy reweighted by the exponential of the reward over beta. Inverting that gives an implicit reward of beta times log(pi over pi_ref). Plugging it into a Bradley-Terry preference model makes the partition function cancel. The result is a classification loss on preference pairs: the negative log sigmoid of beta times the difference between the chosen and rejected log ratios. No reward model or sampling loop is needed.

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

The last case is the failure to watch for. The policy can lower the probability of both responses and still win if the rejected one falls faster. Chosen log-probabilities then go down during training (likelihood displacement), and probability mass leaks to responses nobody labeled. Log both implicit rewards next to the loss and accuracy. Fixes include adding an SFT term on chosen responses, raising beta, or using a length-normalized variant when chosen responses are systematically longer.

Choose DPO when you have a fixed offline preference set, limited infrastructure, and a target like tone, refusal style or format. Choose online RL (PPO or GRPO with a reward model or verifiable reward) when the behavior needs exploration, such as multi-step agent tasks or math with checkable answers, or when the preference data comes from an older policy and has gone stale. For a bank, the reward model or verifier is also an audit artifact you can test on its own.

Why this is likely: the job listing names RLHF expertise and instruction or dialogue fine-tuning, and Capital One's research site lists reinforcement learning from human feedback under its LLM and agentic AI focus area.

### Inference and serving

#### Q11. You need to serve the 10B model to many concurrent users. Size the KV cache, compare BF16, INT8 and INT4 weights, and show why per-channel quantization matters.

Weights scale with precision: 10.2B parameters take about 20 GB in BF16, 10 GB in INT8, and 5 GB in INT4. At long context and high concurrency the KV cache dominates. Each token stores a key and a value per layer, of width number-of-KV-heads times head dimension. With grouped-query attention (8 KV heads of 128 dimensions, versus 32 query heads), the cache is four times smaller than full multi-head attention.

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

That is 192 KiB per token, 768 MiB for one 4,096-token conversation, and 24 GiB for 32 of them. On an 80 GB GPU the cache, not the weights, sets the batch size. That is why paged attention (vLLM) and KV-cache quantization to INT8 or FP8 pay off.

The quantization example shows the outlier problem. One channel with large weights stretches a single per-tensor scale, so every other weight rounds coarsely. Per-channel scales fix weights. Activations have the same issue with outlier feature dimensions, which is why SmoothQuant moves scale from activations into weights and why AWQ and GPTQ protect salient channels when going to INT4. Weight-only INT4 helps decode, which is memory-bandwidth bound. Prefill is compute bound, so it gains more from FP8 matmuls.

Validate quantized models on the full task suite as well as perplexity, and check numbers and long-context retrieval most closely, because they degrade first. For a bank, a wrong digit in a balance is a material error.

Why this is likely: the job listing's optimization track names quantization, model compression and inference optimization, and Capital One's research team wrote up LLM serving efficiency as a leading theme from a recent systems conference.

#### Q12. Explain temperature, top-k and top-p sampling, implement them, and say how you would set them for a banking assistant versus for generating diverse synthetic training data.

Temperature divides the logits before softmax. Values below 1 sharpen the distribution toward the top token, values above 1 flatten it, and temperature 0 means greedy decoding. Top-k keeps the k most likely tokens and renormalizes. Top-p (nucleus sampling) keeps the smallest set of tokens whose cumulative probability reaches p, so the candidate set shrinks when the model is confident and grows when many tokens are plausible. Apply temperature first, then the truncation, then sample.

```python
import numpy as np


def sample(logits, temperature=1.0, top_k=None, top_p=None, rng=None):
    rng = rng or np.random.default_rng()
    if temperature == 0:
        return int(np.argmax(logits))
    z = logits / temperature
    p = np.exp(z - z.max())
    p /= p.sum()
    order = np.argsort(-p)
    keep = np.ones_like(p, dtype=bool)
    if top_k is not None:
        keep[order[top_k:]] = False
    if top_p is not None:
        csum = np.cumsum(p[order])
        cutoff = np.searchsorted(csum, top_p) + 1          # include the token that crosses p
        keep[order[cutoff:]] = False
    p = np.where(keep, p, 0.0)
    return int(rng.choice(len(p), p=p / p.sum()))


logits = np.array([3.0, 2.5, 1.0, 0.5, -1.0, -2.0])
rng = np.random.default_rng(0)
assert sample(logits, temperature=0) == 0
support = lambda **kw: sorted({sample(logits, rng=rng, **kw) for _ in range(3000)})
print("T=1.0", support(temperature=1.0))
print("T=0.5, top_p=0.9", support(temperature=0.5, top_p=0.9))
print("top_k=2", support(top_k=2))
assert support(top_k=2) == [0, 1]
```

With temperature 0.5 and top-p 0.9, only the top two tokens survive. At temperature 1 with no truncation, every token can appear.

For a customer-facing banking assistant, use greedy or low temperature (0 to 0.3) with top-p around 0.9, because consistency and factual accuracy matter more than variety. Remember that even temperature 0 is not perfectly deterministic on GPUs because batching changes floating-point reduction order. For synthetic data, self-consistency voting or red-team prompt generation, raise temperature to 0.8 to 1.2 with top-p near 0.95 to get diverse samples, then filter with a verifier. Also mention max tokens and stop sequences, which bound cost and keep tool-call formats parseable. Repetition and frequency penalties help open-ended text but can corrupt structured output such as JSON or account numbers.

Why this is likely: a Capital One applied research candidate reported a pre-assigned LLM project discussion with about twelve questions on inference parameters and how they change outputs.

### LLM system design and agent safety

#### Q13. Design an LLM assistant that answers customer questions about their accounts and can take simple actions, such as locking a card. Cover retrieval, tools, guardrails, latency and evaluation.

Start with requirements, because the case and design rounds reward asking a second layer of questions. Which actions are allowed? What latency target applies (for example, first token under one second in chat)? What must never happen? Leaking another customer's data, inventing a fee or rate, and acting without confirmation are the obvious ones.

For the architecture, an orchestrator receives the authenticated session, so identity comes from the bank's auth system and never from the model. A small, fast router classifies the request (policy question, account lookup, action, out of scope or escalation). Policy questions go to retrieval-augmented generation over approved, versioned documents: hybrid BM25 plus dense retrieval, a reranker, and answers that must cite chunk IDs. Account data comes only from tools that call internal APIs with the customer's token, so the model never sees data the customer could not see. State-changing actions, such as locking a card, run through a typed tool schema, a deterministic policy check, and an explicit confirmation step shown to the user. The model proposes and the system executes.

Layer the guardrails. Screen inputs for prompt injection, including injection embedded in retrieved documents or transaction memos. Check that every number in the answer matches a tool result or a cited document, redact PII in logs, and hand off to a human agent on low confidence or repeated failure.

For latency, retrieval and reranking take about 100 to 200 ms, tools 100 to 300 ms in parallel, and generation streams. Keep the router and guard models small, cache embeddings, and use prefix caching for the long system prompt. A 10B-class model is usually enough for the main answer when retrieval carries the facts, and it serves far cheaper than a frontier model.

For evaluation, build an offline test set of real (redacted) intents with gold answers and gold tool calls. Score groundedness, numeric exactness, correct refusals and over-refusals, and run adversarial suites. Online, run a shadow deployment, then a small percentage rollout with human review of sampled conversations, tracking containment, escalation rate, complaints and any incorrect action. Model risk management in a bank needs documented evaluation and monitoring, so plan the evaluation harness as a deliverable from the start.

Why this is likely: Capital One's AI and ML recruiters say the Power Day system design round asks for a full end-to-end system built on modern LLM concepts, and candidates report customer chatbot cases in the Power Day case round.

#### Q14. How would you evaluate and red-team a multi-turn, tool-using agent before it touches customer accounts? Include how you would find attacks automatically.

Evaluate whole multi-turn trajectories, because many agent failures only appear several turns in. A benign request becomes a harmful action after context builds up, or an injected instruction in a tool result takes effect three calls later. Define the threat model (malicious user, malicious content in retrieved or tool data, and honest user plus model error) and the harmful outcomes (unauthorized action, data disclosure, policy-violating advice). Then build an environment with mocked tools so the agent can act without touching real accounts.

Measure attack success rate per outcome type with confidence intervals, task success on benign tasks, and over-refusal, because an agent that refuses everything looks safe and is useless. Add trajectory-level risk signals. Estimate the agent's uncertainty at each step (for example, disagreement across sampled plans, or a calibrated probe), aggregate it over the trajectory, and use it to pause and ask for confirmation. Capital One researchers have published on uncertainty in multi-turn agents.

For automated red teaming, static jailbreak lists go stale and random combination wastes queries. Treat attack generation as a bandit: each arm is a composition of instructions (persona, obfuscation, goal framing), and the reward is whether the attack succeeded. That balances finding the strongest strategies with keeping coverage. Capital One's adaptive instruction composition work builds on the same idea with a neural contextual bandit.

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

### Behavioral

#### Q15. Tell me about a research project you owned end to end. How did you use AI tools along the way, and how did you check their output?

Capital One's recruiters say the most common stumble is describing what happened without saying why, and not making your own contribution clear. They also look for a second layer of questions, such as asking whether another approach was considered. Structure the answer as STAR, but spend most of the time on Action and on the choices behind it.

Situation and task (about 30 seconds). One sentence on the problem and why it mattered, with a baseline number. Example: "Our fine-tuned model hallucinated fee amounts in about 6 percent of answers on an internal test set, and the product team could not launch above 1 percent."

Action (about 2 minutes). Say what you decided and why. Mention the options you rejected: "I considered adding more SFT data but chose a grounding check plus preference tuning because the errors were confident ones." Describe what you built yourself and what you delegated. Then answer the AI part concretely. Name where AI tools sped you up (scaffolding evaluation scripts, drafting data filters, summarizing related papers) and how you checked each one. You wrote unit tests before accepting generated code. You spot-checked 100 model-labeled examples by hand and measured agreement. You verified every cited paper claim against the paper itself. Also give one place you chose to work without the tool, such as the evaluation metric definition, because a subtle error there would have invalidated every later result.

Result (about 30 seconds). The metric change, what shipped, and what you would do differently. A real limitation you found and fixed makes the story more credible than a perfect outcome.

Prepare for the follow-up: "What would the next step of this project be?" Hiring managers for these roles report asking about next steps, so have a concrete plan with the experiment you would run first. Keep two backup stories ready, one about a failed experiment and one about disagreeing with a senior stakeholder, because Capital One weights behavioral rounds equally with technical ones.

Why this is likely: Capital One's AI and ML recruiters describe STAR behavioral interviews that probe ownership and the why behind decisions, hiring managers for these roles report asking about project next steps, and engineering candidates report a new question on how they use AI and where they do not trust it.

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
- https://www.teamblind.com/post/capital-one-power-day-interview-ijgi4r10
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
