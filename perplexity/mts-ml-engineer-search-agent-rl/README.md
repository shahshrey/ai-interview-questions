# Perplexity: Member of Technical Staff (Machine Learning Engineer, Search Agent RL)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Perplexity publishes its round types on an official interviewing portal and says each role gets a subset of them, which your recruiter names in advance. No first-hand report exists yet for this exact search agent RL role, so the stages below merge the official pages with recent machine learning engineer and software engineer reports. Treat the order and count as a guess. The portal says hands-on rounds do not use LeetCode, HackerRank or similar question banks, so there are no classic algorithm questions below.

1. Recruiter screen (20 to 30 minutes, virtual). Background, motivation and how independently you work. Some candidates report one real technical question here, such as how positional encoding works in a transformer. AI: not relevant.
2. Technical screen, hands-on coding (30 to 60 minutes live in CoderPad, Python strongly preferred). A multi-part spec where each part ships with its own test file and earlier tests must keep passing. You sign an integrity agreement and share your full screen. Reported prompts include an agent todo list with dependency cascade, an in-memory Unix file system, a tokenizer and an LLM provider pool with failover. AI: not allowed, apart from documentation and asking how a library is used.
3. Possible ML coding screen. One candidate report describes an AI engineer phone screen on ML fundamentals in Python with scikit-learn, and an ML intern reported three technical screens, so ML candidates may get more than one. AI: assume not allowed.
4. Onsite hands-on coding (one or two rounds, 45 to 60 minutes each, in person or virtual depending on role and geography). Same format and rules as the screen; for this team expect rollout, reward or environment components. AI: not allowed.
5. Agentic coding (likely for ML roles). Either 60 to 120 minutes live outside CoderPad or a fixed 3 hour take-home, on a multi-file task where frontier coding agents are expected and the bar for scope and quality is higher than in hands-on coding. An aggregator lists "find bad data in a large clickstream dataset" for a machine learning engineer onsite. AI: provided and expected.
6. Core expertise (30 to 60 minutes, domain named in advance, whiteboard). For this role, likely RL post-training of tool-using agents (GRPO, reward design, train and inference mismatch) and search or retrieval. Experienced candidates should expect to discuss their own papers or systems in precise detail. AI: not allowed.
7. System design or ML system design (45 to 60 minutes, whiteboard). A reported onsite prompt is long-running agentic query execution; for this team also expect RL rollout infrastructure or a search agent design, with numbers. AI: not allowed.
8. Hiring manager deep dive (60 to 90 minutes). One past project end to end, ownership and research judgment; team matching happens during the onsite. AI: not relevant.
9. Founder or senior leader conversation (about 30 minutes, reported by prep blogs). Product and technical conviction. AI: not relevant.

A few loops replace some technical rounds with a Spike Round, a self-defined project of up to 9 hours that is recorded and allows any AI tool. It is rare for this kind of role.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a GRPO rollout collector in three parts: bounded concurrency, retries with... | Build | Onsite hands-on coding | Not allowed | Medium |
| [Q2](#q2) | Build an in-memory file system for agent environments, then add O(1) snapshots and a... | Build | Technical screen | Not allowed | Medium |
| [Q3](#q3) | Debug this GRPO loss: unstable training, and solved prompts keep getting large updates | Debug | Onsite hands-on coding | Not allowed | Medium |
| [Q4](#q4) | Review this AI-generated importance-sampling correction for train and inference... | Review | Onsite hands-on coding | Not allowed | Hard |
| [Q5](#q5) | This search agent trajectory earned full reward. Find what went wrong and write a... | Trace debug | Onsite hands-on coding | Not allowed | Medium |
| [Q6](#q6) | With a coding agent, find the bad rows in a 2 million row synthetic multi-hop QA... | Build | Agentic coding | Provided (frontier coding agent) | Hard |
| [Q7](#q7) | Write the reward for a search agent: correctness gate, preference bonus, efficiency... | Concept plus code | Core expertise | Not allowed | Medium |
| [Q8](#q8) | Choose which synthetic prompts to train on from pass@k results, and estimate how... | Concept plus code | Core expertise | Not allowed | Medium |
| [Q9](#q9) | Train reward climbs while held-out accuracy falls. Diagnose the run from these metrics | Trace debug | Core expertise | Not allowed | Hard |
| [Q10](#q10) | Two reward sources, one policy: set the prompt mixture so neither objective swamps... | Concept plus code | Core expertise | Not allowed | Hard |
| [Q11](#q11) | Rubric rewards for queries with no verifiable answer: build and calibrate the judge... | Concept plus code | Core expertise | Not allowed | Hard |
| [Q12](#q12) | Design the RL training system for a search agent: rollouts, search sandbox, reward... | Design | System design | Not allowed | Hard |
| [Q13](#q13) | Design execution for long-running agentic queries that survive crashes, deploys and... | Design | System design | Not allowed | Hard |
| [Q14](#q14) | Design the eval that decides whether a new first-stage retriever ships for the... | Design | System design | Not allowed | Hard |
| [Q15](#q15) | A planner agent delegates to searcher subagents. Design training and credit assignment | Design | Core expertise | Not allowed | Hard |
| [Q16](#q16) | Walk through an idea you carried to a working system, and how you check coding agent... | Behavioral | Hiring manager deep dive | Not allowed | Medium |

### Coding: hands-on rounds (no AI, as at Perplexity)

<a id="q1"></a>

#### Q1. Build a GRPO rollout collector in three parts (no AI, as at Perplexity): part 1 samples G rollouts per prompt with bounded concurrency and returns them grouped in input order; part 2 adds a per-rollout timeout and retries for transient errors; part 3 drops groups that carry no learning signal.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Async concurrency, RL rollouts.

<details>
<summary>Show answer</summary>

Task: Given `prompts` as `(prompt_id, prompt)` pairs and an async `policy(prompt, k)` that runs one agent episode and returns a scalar reward, collect G rollouts per prompt with at most `max_conc` episodes in flight. Transient failures (rate limits, timeouts) retry up to `max_retries`; non-transient errors such as a malformed tool call fail at once. Part 3 drops a group when fewer than `min_ok` rollouts succeeded or when every reward in it is equal.

What they grade: correct results under each part's tests, earlier tests staying green, clean separation of retryable and fatal errors, and knowing why part 3 matters for GRPO.

Approach: Flatten to one coroutine per (prompt, sample) and let a semaphore bound the episodes in flight. I compared two alternatives. A worker pool reading from a queue gives the same bound but needs extra code to restore order. Launching one task per prompt that runs its G samples in sequence keeps grouping trivial but caps parallelism at the number of prompts, so 64 prompts with G = 8 would use 64 slots instead of 512. Flat tasks plus `gather` keep input order for free, and slicing by G rebuilds the groups. The timeout wraps only the episode and starts after the semaphore is acquired, so time spent queued does not count against a rollout. Retries use jittered exponential backoff so a burst of 429s does not retry in lockstep. Part 3 follows from the GRPO advantage, which is the reward minus the group mean, so a group whose rewards are all equal gives zero gradient and only costs compute. A group with too few successes would give a biased mean, so it is dropped too.

Code:

```python
from __future__ import annotations
import asyncio, random
from dataclasses import dataclass, field

class Transient(Exception):
    pass

@dataclass
class Rollout:
    prompt_id: str
    sample: int
    reward: float | None = None
    error: str | None = None
    attempts: int = 0

@dataclass
class Group:
    prompt_id: str
    rollouts: list = field(default_factory=list)

async def run_one(policy, prompt_id, prompt, k, sem, timeout_s, max_retries):
    r = Rollout(prompt_id, k)
    for attempt in range(max_retries + 1):
        r.attempts = attempt + 1
        try:
            async with sem:
                r.reward = await asyncio.wait_for(policy(prompt, k), timeout_s)
            r.error = None
            return r
        except (Transient, asyncio.TimeoutError) as e:
            r.error = type(e).__name__
            await asyncio.sleep(random.uniform(0, 0.01 * 2 ** attempt))
        except Exception as e:  # bad tool schema etc: do not burn retries
            r.error = type(e).__name__
            return r
    return r

async def collect(policy, prompts, G=4, max_conc=8, timeout_s=1.0, max_retries=2, min_ok=3):
    sem = asyncio.Semaphore(max_conc)
    tasks = [run_one(policy, pid, p, k, sem, timeout_s, max_retries)
             for pid, p in prompts for k in range(G)]
    flat = await asyncio.gather(*tasks)
    groups, dropped = [], {}
    for i, (pid, _) in enumerate(prompts):
        g = Group(pid, flat[i * G:(i + 1) * G])
        ok = [r.reward for r in g.rollouts if r.error is None]
        if len(ok) < min_ok:
            dropped[pid] = "too_few_ok"
        elif max(ok) == min(ok):
            dropped[pid] = "zero_variance"
        else:
            groups.append(g)
    return groups, dropped

if __name__ == "__main__":
    calls, inflight, peak = {}, 0, 0
    async def policy(prompt, k):
        global inflight, peak
        inflight += 1; peak = max(peak, inflight)
        try:
            key = (prompt, k); calls[key] = calls.get(key, 0) + 1
            if prompt == "hang" and k == 0:
                await asyncio.sleep(5)
            if prompt == "flaky" and calls[key] < 3:
                raise Transient()
            if prompt == "schema":
                raise ValueError("bad tool args")
            await asyncio.sleep(0.01)
            return {"easy": 1.0, "hard": 0.0}.get(prompt, float(k % 2))
        finally:
            inflight -= 1
    prompts = [("p1", "flaky"), ("p2", "easy"), ("p3", "hard"), ("p4", "hang"), ("p5", "schema"), ("p6", "mixed")]
    groups, dropped = asyncio.run(collect(policy, prompts, G=4, max_conc=3, timeout_s=0.2, max_retries=2))
    kept = [g.prompt_id for g in groups]
    assert kept == ["p1", "p4", "p6"], kept
    assert dropped == {"p2": "zero_variance", "p3": "zero_variance", "p5": "too_few_ok"}, dropped
    assert all(r.attempts == 3 for r in groups[0].rollouts)
    assert groups[1].rollouts[0].error == "TimeoutError" and groups[1].rollouts[0].attempts == 3
    assert peak <= 3, peak
    print("ok", kept, dropped, "peak", peak)
```

Verify: a flaky prompt succeeds on attempt 3, a hanging rollout times out after all retries while its siblings still form a valid group, a schema error fails once without retries, all-pass and all-fail groups are dropped as zero variance, and a counter proves the concurrency cap holds.

Pitfalls:
- Putting the timeout around `async with sem`, which kills rollouts that were only queued.
- Retrying a malformed tool call, which wastes the retry budget on a deterministic failure.
- Computing the zero-variance filter before retries finish, which drops groups that would have recovered.

Follow-ups: refilling a dropped group by sampling new prompts so the batch size stays fixed, a token budget per episode, and streaming finished groups to the trainer.

Why this is likely: Perplexity's hands-on coding page describes multi-part practical components of its own systems, and this team trains search agents with GRPO according to its research blog, so a rollout collector is a natural component (official format, high confidence; topic inferred, medium).

</details>

<a id="q2"></a>

#### Q2. Build an in-memory file system for agent environments (no AI, as at Perplexity): part 1 supports mkdir, write with append, read and ls on absolute paths; part 2 adds snapshot and restore in O(1) so each RL episode can reset the environment; part 3 adds a diff between two snapshots.

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Data structures, environment state.

<details>
<summary>Show answer</summary>

Task: Paths are absolute strings. Errors: a missing parent raises `FileNotFoundError`, reading a directory raises `IsADirectoryError`, a relative path raises `ValueError`. Snapshots must be cheap because a rollout worker resets the environment thousands of times per hour, and the diff should list added, deleted and changed files.

What they grade: a clear tree model, all error paths, a snapshot that does not copy the world, and tests per part.

Approach: Directories are dicts and files are strings. The main design choice is the snapshot. A deep copy per snapshot is simple but costs O(size of tree): with 50,000 files of 2 KB that is about 100 MB copied per episode reset. A write log that replays from the start makes snapshots free but makes restore O(history). I chose path copying, the standard persistent tree technique. A write copies only the dicts along its path, so it costs O(sum of fan-outs along the path), and every untouched subtree is shared. Getting O(depth x log fanout) needs a persistent map per directory (a HAMT or persistent balanced tree, for example pyrsistent's pmap); plain dict copying is fine while directories stay small. The root is never mutated in place, so a snapshot is a saved root pointer, O(1), and restore swaps the pointer back. The diff walks two roots together and skips any subtree whose object identity is the same in both, so untouched subtrees cost nothing. Each directory on a changed path still costs O(fan-out log fan-out) for the key scan and sort, so the total is O(sum of fan-outs along the changed paths). A persistent map per directory would also reduce this cost.

Code:

```python
from __future__ import annotations

class FS:
    """Directories are dicts, files are str. Snapshots share untouched subtrees (path copying)."""
    def __init__(self):
        self.root: dict = {}
        self.snaps: dict[str, dict] = {}

    @staticmethod
    def _parts(path: str) -> list[str]:
        if not path.startswith("/"):
            raise ValueError("absolute path required")
        return [p for p in path.split("/") if p]

    def _walk(self, parts):
        node = self.root
        for p in parts:
            if not isinstance(node, dict) or p not in node:
                raise FileNotFoundError("/" + "/".join(parts))
            node = node[p]
        return node

    def _write(self, parts, value, mkdirs):
        def rec(node, i):
            new = dict(node)  # copy only the nodes on the path
            name = parts[i]
            if i == len(parts) - 1:
                if isinstance(node.get(name), dict):
                    raise IsADirectoryError(name)
                new[name] = value
                return new
            child = node.get(name)
            if child is None:
                if not mkdirs:
                    raise FileNotFoundError(name)
                child = {}
            if not isinstance(child, dict):
                raise NotADirectoryError(name)
            new[name] = rec(child, i + 1)
            return new
        self.root = rec(self.root, 0)

    def mkdir(self, path):
        parts = self._parts(path)
        node = self.root
        for i, p in enumerate(parts):
            if isinstance(node, dict) and p in node:
                node = node[p]
                continue
            self._write(parts[:i + 1], {}, mkdirs=True)
            node = self._walk(parts[:i + 1])

    def write(self, path, data, append=False):
        parts = self._parts(path)
        old = ""
        if append:
            try:
                old = self._walk(parts)
            except FileNotFoundError:
                pass
        self._write(parts, old + data, mkdirs=False)

    def read(self, path):
        node = self._walk(self._parts(path))
        if isinstance(node, dict):
            raise IsADirectoryError(path)
        return node

    def ls(self, path="/"):
        node = self._walk(self._parts(path))
        return sorted(node) if isinstance(node, dict) else [self._parts(path)[-1]]

    def snapshot(self, name):
        self.snaps[name] = self.root  # O(1): root is never mutated in place

    def restore(self, name):
        self.root = self.snaps[name]

def diff(a: dict, b: dict, prefix=""):
    out = []
    for k in sorted(set(a) | set(b)):
        p = f"{prefix}/{k}"
        x, y = a.get(k), b.get(k)
        if x is y:
            continue
        if isinstance(x, dict) and isinstance(y, dict):
            out += diff(x, y, p)
        elif x is None:
            out.append(("added", p))
        elif y is None:
            out.append(("deleted", p))
        elif x != y:
            out.append(("changed", p))
    return out

if __name__ == "__main__":
    fs = FS()
    fs.mkdir("/work/notes"); fs.mkdir("/cache")
    fs.write("/work/notes/a.md", "hello")
    fs.write("/cache/q1.json", "{}")
    fs.snapshot("reset")
    base = fs.root
    fs.write("/work/notes/a.md", " world", append=True)
    fs.write("/work/notes/b.md", "new")
    assert fs.read("/work/notes/a.md") == "hello world"
    assert fs.root["cache"] is base["cache"]  # untouched subtree shared
    assert diff(base, fs.root) == [("changed", "/work/notes/a.md"), ("added", "/work/notes/b.md")]
    fs.restore("reset")
    assert fs.ls("/work/notes") == ["a.md"] and fs.read("/work/notes/a.md") == "hello"
    for bad in (lambda: fs.read("/nope"), lambda: fs.write("/x/y.md", "z"),
                lambda: fs.read("/work"), lambda: fs.write("rel", "z")):
        try:
            bad(); raise AssertionError("expected error")
        except (FileNotFoundError, IsADirectoryError, ValueError):
            pass
    print("ok")
```

Verify: append keeps old content, an untouched subtree is shared by identity after a write, the diff lists exactly one changed and one added file, restore brings back the reset state, and the four error paths raise.

Pitfalls:
- Mutating a dict in place anywhere, which silently corrupts every saved snapshot.
- Letting `write` create missing parents, which hides agent bugs that a real shell would surface.
- Diffing by deep comparison of whole trees, which is O(tree) per episode.

Follow-ups: `mv` and `rm -r`, a per-episode memory budget, and serializing a snapshot so a crashed rollout worker can resume.

Why this is likely: a candidate-report aggregator marks an in-memory Unix file system as a high-frequency Perplexity screen prompt (Oct 2026), and agent RL needs cheap environment resets (candidate reports, medium-high confidence; RL framing inferred).

</details>

<a id="q3"></a>

#### Q3. Debug this GRPO loss (no AI, as at Perplexity). Training is unstable, the loss sometimes turns into NaN, and prompts the policy already solves 8/8 keep receiving large positive updates while partially solved prompts barely move. Find every bug, fix it and add tests.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: RL post-training, policy gradient.

```python
import numpy as np

# logp_new, logp_old: [B*G, T] per-token log-probs of the sampled tokens
# rewards: [B*G] one scalar per rollout, G rollouts per prompt, grouped contiguously
# attn_mask: [B*G, T] 1 for every real token (prompt and response), 0 for padding
def grpo_loss(logp_new, logp_old, rewards, attn_mask, G, clip=0.2):
    adv = (rewards - rewards.mean()) / rewards.std()
    ratio = np.exp(logp_new - logp_old)
    unclipped = ratio * adv[:, None]
    clipped = np.clip(ratio, 1 - clip, 1 + clip) * adv[:, None]
    per_tok = np.maximum(unclipped, clipped)
    return -(per_tok * attn_mask).sum() / attn_mask.shape[0]
```

<details>
<summary>Show answer</summary>

Task: find the defects that explain each symptom, fix them, and write tests that would have caught each one.

What they grade: tying each bug to a symptom, a minimal fix, and tests that pin the PPO-style clipping behaviour.

Approach: I read the code against the GRPO objective and found four bugs and one normalization choice to state.
1. Advantages are normalized over the whole batch, not per group. A prompt the policy solves 8 of 8 times then gets positive advantages only because other prompts in the batch are harder, so a solved prompt gets pure reinforcement with zero within-group contrast. Mixed groups are divided by the batch std, which is usually larger than their own, so their contrast shrinks. That is the symptom: solved prompts keep getting large updates while partially solved ones barely move. The fix is reshaping to `[B, G]` and normalizing per row.
2. `rewards.std()` is zero when every reward in the batch is equal, which gives a NaN loss. Add an epsilon, and with per-group stats a tied group gives exactly zero advantage.
3. `np.maximum` takes the optimistic side of the clipped surrogate. PPO and GRPO take the minimum so the ratio cannot be pushed past the clip range, so this is the instability.
4. The loss mask covers prompt tokens. The policy then gets credit or blame for tokens it did not generate, and tool results inserted into the context are trained on as if the model wrote them. Use a response mask that also zeros tool output tokens.
5. Normalization is a design choice to state out loud. Dividing the token-summed loss by B x G or by the response-token count weights every token equally; they differ only in scale. Dividing by the response-token count (DAPO token-level mean) changes the step size from batch to batch, while a fixed constant such as B x G x max_len (Dr. GRPO) keeps it stable. A mean per sequence and then a mean over sequences (original GRPO) weights sequences equally and under-weights tokens of long trajectories. I keep the response-token count because, once the mask excludes prompt and tool tokens, it makes the loss a per-generated-token average that does not depend on prompt length, and I say so.

Code (fixed):

```python
import numpy as np

def group_advantages(rewards, G, eps=1e-6):
    r = rewards.reshape(-1, G)
    mu, sd = r.mean(1, keepdims=True), r.std(1, keepdims=True)
    return ((r - mu) / (sd + eps)).reshape(-1)  # zero-variance group -> 0, not NaN

def grpo_loss(logp_new, logp_old, rewards, resp_mask, G, clip=0.2):
    adv = group_advantages(rewards, G)[:, None]
    ratio = np.exp(logp_new - logp_old)
    surr = np.minimum(ratio * adv, np.clip(ratio, 1 - clip, 1 + clip) * adv)
    return -(surr * resp_mask).sum() / resp_mask.sum()  # chosen: mean over generated tokens (DAPO style)

if __name__ == "__main__":
    a = group_advantages(np.array([0., 1., 0., 1., 0., 10., 0., 10.]), G=4)
    assert np.allclose(a[:4], a[4:]), "scale of a group's rewards must not matter"
    z = group_advantages(np.array([1., 1., 1., 1.]), G=4)
    assert np.all(z == 0) and not np.isnan(z).any()
    lo = np.zeros((2, 3)); m = np.array([[0, 1, 1], [0, 1, 1]], float)
    ln = lo.copy(); ln[0, 1] = np.log(1.5)  # ratio 1.5 on a positive-advantage token
    rew = np.array([1., 0.])
    loss = grpo_loss(ln, lo, rew, m, G=2)
    adv = group_advantages(rew, 2)
    expect = -((1.2 * adv[0]) + adv[0] + 2 * adv[1]) / 4
    assert np.isclose(loss, expect), (loss, expect)
    ln2 = ln.copy(); ln2[:, 0] = 5.0  # prompt-token logprobs must not move the loss
    assert np.isclose(grpo_loss(ln2, lo, rew, m, G=2), loss)
    print("ok", round(float(loss), 4))
```

Verify: two groups with different reward scales get identical advantages, a tied group gives zeros and no NaN, a token with ratio 1.5 and positive advantage contributes 1.2 times the advantage, and changing prompt-token log-probs does not move the loss.

Pitfalls:
- Fixing only the NaN with an epsilon and leaving batch-level normalization in place.
- Masking prompt tokens but not tool-result tokens.

Follow-ups: adding a KL term to a reference policy versus dropping it, and dropping the std division (Dr. GRPO style) to remove the bias toward easy and hard prompts.

Why this is likely: Perplexity's research blog describes a GRPO stage in its search-augmented model training, and debugging given code with failing tests fits the hands-on coding format (official format and team blog, medium-high confidence).

</details>

<a id="q4"></a>

#### Q4. Rollouts are sampled with the inference engine and trained in the training framework, and their log-probs disagree. A coding assistant wrote this truncated importance-sampling correction. Review it like a pull request (no AI, as at Perplexity): find the defects, decide whether to merge, and write the tests that would have caught them.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Review. Difficulty: Hard. Topic: Train and inference mismatch, importance sampling.

```python
import numpy as np

def token_logp(logits, tokens):
    # logits: [B, T, V] from the trainer, tokens: [B, T]
    logp = logits - np.logsumexp(logits, axis=-1, keepdims=True)
    return np.take_along_axis(logp, tokens[..., None], -1)[..., 0]

def tis_weights(train_logp, infer_logp, mask, cap=2.0):
    w = np.exp(infer_logp - train_logp)
    w = np.clip(w, 0.0, cap)
    seq_w = w.prod(axis=1)
    return np.nan_to_num(seq_w)

def pg_loss(train_logp, infer_logp, adv, mask):
    w = tis_weights(train_logp, infer_logp, mask)
    return -(w[:, None] * adv[:, None] * train_logp).mean()
```

<details>
<summary>Show answer</summary>

Task: list each defect with its impact, block or approve, and give fixed code with tests.

What they grade: catching the hallucinated API and the off-by-one, understanding which direction the ratio goes, and token-level versus sequence-level weighting.

Approach: I would block the merge. The defects, in order of how badly they hurt:
1. Hallucinated API: NumPy has no `np.logsumexp`, so this crashes on first call (SciPy has `scipy.special.logsumexp`). I would write a stable log-softmax with max subtraction instead of adding a dependency.
2. Off-by-one: logits at position t predict token t + 1, so the log-prob of `tokens[:, t]` must come from `logits[:, t - 1]`. The code gathers each token from its own position, so every log-prob belongs to the wrong token and the mismatch looks huge.
3. Inverted ratio: we sample from the inference policy and optimize the training policy, so the weight is pi_train over pi_infer, `exp(train - infer)`. The code computes the reciprocal, which up-weights exactly the tokens the trainer now considers unlikely.
4. Sequence-level product: multiplying 2,000 per-token weights explodes or vanishes even when each is near 1 (1.01 to the 2,000th power is about 4.4e8). A per-token truncated weight keeps variance bounded; Perplexity's research post discusses this token-level versus sequence-level choice. The sequence version wins only for very short outputs.
5. Missing edge case: `mask` is accepted but ignored, so padding tokens with garbage log-probs enter both the product and the `.mean()`, and `nan_to_num` then hides the resulting NaN instead of failing loudly.
In the real framework the weight must also be detached so gradients flow only through `train_logp`.

Code (fixed):

```python
import numpy as np

def log_softmax(x):
    m = x.max(-1, keepdims=True)
    return x - m - np.log(np.exp(x - m).sum(-1, keepdims=True))

def token_logp(logits, tokens):
    # logits[:, t] predicts tokens[:, t + 1]
    lp = log_softmax(logits[:, :-1])
    return np.take_along_axis(lp, tokens[:, 1:, None], -1)[..., 0]

def tis_weights(train_logp, infer_logp, mask, cap=2.0):
    w = np.exp(np.clip(train_logp - infer_logp, -20, 20))  # pi_train / pi_infer, per token
    return np.minimum(w, cap) * mask  # treated as a constant (detached) in the real loss

def pg_loss(train_logp, infer_logp, adv, mask):
    w = tis_weights(train_logp, infer_logp, mask)
    return -(w * adv[:, None] * train_logp * mask).sum() / mask.sum()

if __name__ == "__main__":
    rng = np.random.default_rng(0)
    B, T, V = 2, 6, 5
    logits = rng.normal(size=(B, T, V)); tokens = rng.integers(0, V, size=(B, T))
    lp = token_logp(logits, tokens)
    assert lp.shape == (B, T - 1)
    manual = np.log(np.exp(logits[0, 0]) / np.exp(logits[0, 0]).sum())[tokens[0, 1]]
    assert np.isclose(lp[0, 0], manual)  # shift is right
    mask = np.ones_like(lp); mask[1, 3:] = 0  # padding on row 1
    w = tis_weights(lp, lp, mask)
    assert np.allclose(w, mask)  # identical policies -> weight 1 on real tokens, 0 on padding
    w2 = tis_weights(lp + np.log(3.0), lp, mask)
    assert np.allclose(w2[mask == 1], 2.0)  # truncated at cap
    bad = lp.copy(); bad[1, 4] = -1e9  # garbage logprob on a padded token
    assert np.isfinite(pg_loss(bad, lp, np.array([1., -1.]), mask))
    l1 = pg_loss(lp, lp, np.array([1., -1.]), mask)
    l2 = pg_loss(np.where(mask == 1, lp, 123.0), lp, np.array([1., -1.]), mask)
    assert np.isclose(l1, l2)  # padding never enters the loss
    print("ok", round(float(l1), 4))
```

Verify: the gathered log-prob equals a hand-computed softmax for the next token, identical policies give weight 1 on real tokens and 0 on padding, a 3x ratio is truncated to the cap, and garbage on padded positions never changes the loss.

Pitfalls:
- Clipping the ratio on both sides like PPO; truncation only caps the top.
- Fixing the API and approving, without checking the shift.

Follow-ups: masking tokens whose ratio exceeds a threshold instead of truncating, and why MoE routing differences between engines cause the mismatch (routing replay).

Why this is likely: Perplexity's research post on search-augmented models discusses train and inference mismatch and token-level importance sampling, and the hands-on format uses practical review and debug of given code; the AI-written framing is inferred (team blog, medium confidence).

</details>

<a id="q5"></a>

#### Q5. This search agent trajectory earned full reward (no AI, as at Perplexity). Find every failure in it, explain why the reward was wrong, and write a detector that flags the same patterns across a million stored trajectories.

Round: Onsite hands-on coding. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Agent trajectories, reward hacking.

```text
{"step": 0, "type": "user", "text": "Which lab released the open model that won the retrieval benchmark run by the university where its first author did a PhD?"}
{"step": 1, "type": "tool_call", "tool": "search", "args": {"q": "retrieval benchmark winner open model"}}
{"step": 2, "type": "tool_result", "tool": "search", "status": 200, "results": ["benchmark page: winner model M-7 ...", "news: M-7 tops ranking ..."]}
{"step": 3, "type": "tool_call", "tool": "search", "args": {"q": "M-7 model first author PhD university"}}
{"step": 4, "type": "tool_result", "tool": "search", "status": 429, "results": []}
{"step": 5, "type": "tool_call", "tool": "search", "args": {"q": "M-7 model first author PhD university"}}
{"step": 6, "type": "tool_result", "tool": "search", "status": 429, "results": []}
{"step": 7, "type": "tool_call", "tool": "search", "args": {"q": "M-7 model first author PhD university"}}
{"step": 8, "type": "tool_result", "tool": "search", "status": 200, "results": []}
{"step": 9, "type": "assistant", "text": "No sources found. It was likely Lab Alpha, Lab Beta or Lab Gamma."}
{"step": 10, "type": "reward", "gold": "Lab Beta", "grader": "substring", "correct": 1, "reward": 1.0}
```

<details>
<summary>Show answer</summary>

Task: root-cause the trajectory and ship a streaming detector with tests.

What they grade: separating environment bugs from policy bugs from grader bugs, and a detector that is cheap enough to run over every rollout.

Approach: There are three distinct failures.
1. Environment: the search tool returned 429 twice and the harness passed an empty result list to the model, so the agent cannot tell "rate limited" from "nothing exists". During RL this teaches the policy that search is unreliable. The harness should retry with backoff inside the tool and, if that fails, return an explicit error string the model can read.
2. Policy: the agent issued the same query three times and never reformulated (no "M-7 paper authors", no hop through the paper page). That is a learned loop, and step 8 shows it was not even the rate limit any more.
3. Grader: the final answer hedges across three labs, and a substring grader found "Lab Beta" inside it and paid full reward. Left in place, it teaches the policy to list every candidate. The fix is an exact-match on an extracted single answer, or a judge instructed to fail hedged answers, plus a penalty for multiple candidates.
I would fix the grader first, because it corrupts the gradient for every prompt, while the tool error affects only rate-limited episodes.

The detector streams events per trajectory and keeps O(distinct calls) state, so it runs as a map job. I chose rules over a learned classifier for the first pass because each rule maps to one owner (harness, policy, grader) and has no false positives on the clean case; a classifier is worth it later for fuzzier patterns.

Code:

```python
import json, re

TRACE = """\
{"step": 0, "type": "user", "text": "Which lab released the open model that won the retrieval benchmark run by the university where its first author did a PhD?"}
{"step": 1, "type": "tool_call", "tool": "search", "args": {"q": "retrieval benchmark winner open model"}}
{"step": 2, "type": "tool_result", "tool": "search", "status": 200, "results": ["benchmark page: winner model M-7 ...", "news: M-7 tops ranking ..."]}
{"step": 3, "type": "tool_call", "tool": "search", "args": {"q": "M-7 model first author PhD university"}}
{"step": 4, "type": "tool_result", "tool": "search", "status": 429, "results": []}
{"step": 5, "type": "tool_call", "tool": "search", "args": {"q": "M-7 model first author PhD university"}}
{"step": 6, "type": "tool_result", "tool": "search", "status": 429, "results": []}
{"step": 7, "type": "tool_call", "tool": "search", "args": {"q": "M-7 model first author PhD university"}}
{"step": 8, "type": "tool_result", "tool": "search", "status": 200, "results": []}
{"step": 9, "type": "assistant", "text": "No sources found. It was likely Lab Alpha, Lab Beta or Lab Gamma."}
{"step": 10, "type": "reward", "gold": "Lab Beta", "grader": "substring", "correct": 1, "reward": 1.0}
"""

def detect(events, repeat_limit=2):
    flags, seen, answer = [], {}, ""
    for e in events:
        if e["type"] == "tool_call":
            key = (e["tool"], json.dumps(e["args"], sort_keys=True))
            seen[key] = seen.get(key, 0) + 1
            if seen[key] > repeat_limit:
                flags.append(("repeated_call", e["step"]))
        elif e["type"] == "tool_result" and e["status"] != 200:
            flags.append(("tool_error", e["step"]))
        elif e["type"] == "assistant":
            answer = e["text"]
        elif e["type"] == "reward" and e["correct"]:
            cands = re.findall(r"Lab \w+", answer)  # stand-in for an entity tagger
            if len(set(cands)) > 1:
                flags.append(("hedged_answer_rewarded", e["step"]))
    return flags

if __name__ == "__main__":
    ev = [json.loads(l) for l in TRACE.splitlines()]
    f = detect(ev)
    assert ("tool_error", 4) in f and ("tool_error", 6) in f
    assert ("repeated_call", 7) in f
    assert ("hedged_answer_rewarded", 10) in f
    clean = [e for e in ev if e["step"] in (0, 1, 2)] + [
        {"step": 3, "type": "assistant", "text": "Lab Beta"},
        {"step": 4, "type": "reward", "gold": "Lab Beta", "grader": "substring", "correct": 1, "reward": 1.0}]
    assert detect(clean) == []
    print("ok", f)
```

Verify: the two 429 steps, the third identical call and the rewarded hedge are flagged, and a clean three-step trajectory yields no flags.

Pitfalls:
- Treating the empty result at step 8 as fine because its status is 200; it follows two errors and is the third identical query.
- Fixing only the agent and leaving the substring grader in place.

Follow-ups: the share of rewarded episodes with a hedge flag as a training health metric, and a detector for answers whose entities never appear in any tool result.

Why this is likely: the listing asks for diagnosing failures, and Perplexity controls its models, harness and search stack, so debugging a stored trajectory fits a hands-on round (listing plus official format, medium confidence).

</details>

### Coding: agentic round (AI provided)

<a id="q6"></a>

#### Q6. Agentic coding (frontier coding agent provided): this repo builds the synthetic multi-hop QA set used for search agent RL. It has 2 million rows, and the last run's reward curve suggests bad data. Use the agent to find and quantify the bad rows, add checks to the pipeline, and show what you verified.

Round: Agentic coding. AI policy: Provided (frontier coding agent). Type: Build. Difficulty: Hard. Topic: Data quality for RL, agent-driven development.

```text
qa-pipeline/
  README.md
  pyproject.toml
  data/
    train.jsonl            # 2,000,000 rows: id, question, answer, solver_answers, source_urls
    eval_heldout.jsonl     # 5,000 rows used for launch decisions
  pipeline/
    generate.py            # sequential expansion of multi-hop questions
    verify.py              # runs 3 solver models, stores their answers
    export.py              # writes train.jsonl
  checks/
    __init__.py            # empty: this is where your checks go
  tests/
    test_export.py         # 2 passing tests
```

<details>
<summary>Show answer</summary>

Task: produce a report of issue types with counts and examples, a `checks/` module wired into `export.py`, and tests.

What they grade: the scope you reach in the time, how you steer and verify the agent, and whether the numbers you report are true.

Approach: First I read `generate.py` and `verify.py` myself for 5 minutes, because I need to know what "correct" means before an agent writes checks. Then the delegation plan:
1. Prompt: "Write a streaming script that reads train.jsonl line by line and reports counts for: normalized duplicate questions, answer string inside the question, 8-gram overlap with eval_heldout questions, fewer than 2 of 3 solvers agreeing with the gold answer, empty answers. Read it as a stream with constant memory. Print 3 example ids per type."
2. Prompt: "Turn those rules into checks/quality.py with a pure function per rule and unit tests on a 6-row fixture I will write."
3. I write the fixture, one row per issue plus a clean one, before reading the agent's code, so the tests are not shaped by its output.

Checks in the diff: streaming rather than `pandas.read_json` on 2 million rows, normalization (case, whitespace, punctuation) applied before duplicate checks, word-boundary matching for the answer leak so "2011" does not match "20110", and counts from the full file rather than a sample. For contamination, exact match misses paraphrases and embedding similarity catches them at the cost of about 2 million embeddings, so 8-gram overlap is the cheap first pass, with embeddings only for the eval set's nearest neighbours if time remains.

In one wrong output I caught, the agent's first version ran on `head -n 10000` "for speed" and printed counts that the summary then presented as full-dataset numbers. In a second, it deduplicated on raw strings and found zero duplicates, and the fixture row with double spaces and different case exposed it.

Code (the checks module after review):

```python
import re
from collections import defaultdict

def norm(s):
    return re.sub(r"[^a-z0-9 ]", "", re.sub(r"\s+", " ", s.lower())).strip()

def ngrams(s, n=8):
    w = norm(s).split()
    return {" ".join(w[i:i + n]) for i in range(max(0, len(w) - n + 1))} or {" ".join(w)}

def find_issues(rows, eval_questions, n=8, min_agree=2):
    eval_grams = set().union(*(ngrams(q, n) for q in eval_questions)) if eval_questions else set()
    first_seen, issues = {}, defaultdict(list)
    for r in rows:
        q, a = norm(r["question"]), norm(r["answer"])
        if not a:
            issues["empty_answer"].append(r["id"])
        elif re.search(rf"\b{re.escape(a)}\b", q):
            issues["answer_in_question"].append(r["id"])
        if q in first_seen:
            issues["duplicate"].append(r["id"])
        else:
            first_seen[q] = r["id"]
        if ngrams(r["question"], n) & eval_grams:
            issues["eval_contamination"].append(r["id"])
        agree = sum(norm(s) == a for s in r.get("solver_answers", []))
        if a and agree < min_agree:
            issues["unverified"].append(r["id"])
    return dict(issues)

if __name__ == "__main__":
    ev = ["Which river flows through the capital city of the country that hosted the 1992 summer games?"]
    rows = [
        {"id": 1, "question": "Who founded the company that makes the M-7 model?", "answer": "Ada Li", "solver_answers": ["Ada Li", "ada li", "Bo Chen"]},
        {"id": 2, "question": "Who  founded the company that makes the M-7 MODEL?", "answer": "Ada Li", "solver_answers": ["Ada Li"] * 3},
        {"id": 3, "question": "Which river flows through the capital city of the country that hosted the 1992 summer games?", "answer": "Manzanares", "solver_answers": ["Manzanares"] * 3},
        {"id": 4, "question": "Which Lab Beta model topped the board?", "answer": "Lab Beta", "solver_answers": ["Lab Beta"] * 3},
        {"id": 5, "question": "What year did the founder of Lab Beta finish a PhD?", "answer": "2011", "solver_answers": ["2011", "2012", "2010"]},
        {"id": 6, "question": "Name the first author of the M-7 paper.", "answer": "", "solver_answers": []},
    ]
    got = find_issues(rows, ev)
    assert got == {"duplicate": [2], "eval_contamination": [3], "answer_in_question": [4],
                   "unverified": [5], "empty_answer": [6]}, got
    print("ok", got)
```

Verify: the fixture yields exactly one id per issue type and none for the clean row, `tests/test_export.py` stays green, and the full run is checked against `wc -l` so no rows were skipped.

Pitfalls:
- Reporting sample counts as totals.
- Dropping every unverified row, which removes the hardest multi-hop questions; quarantine them for a stronger solver instead.

Follow-ups: how much of the reward plateau the bad rows explain (retrain a small run on the cleaned set), and near-duplicate detection with MinHash at 2 million rows.

Why this is likely: Perplexity's official agentic coding round expects frontier agents on a multi-file task, and a candidate-report aggregator lists "find bad data in a large dataset" as a machine learning engineer agentic prompt; the synthetic multi-hop data angle follows the team's research blog (official format high, topic medium).

</details>

### Core expertise: RL for search agents

<a id="q7"></a>

#### Q7. Write the reward for a search agent: a correctness gate, a preference bonus from a judge, and an efficiency penalty for tool calls and answer length. Show why your formula resists the obvious hacks.

Round: Core expertise. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Reward design.

<details>
<summary>Show answer</summary>

Task: a reward function, the reasoning behind each term, and tests for the hacks it must resist.

What they grade: understanding that composite rewards are traded off by the policy, gate design, and how weights interact with GRPO's group normalization.

Approach: I compared three shapes. A weighted sum, `a x correct + b x pref - c x cost`, lets a wrong but polished answer outscore a correct terse one whenever `b x pref` exceeds `a`, so the policy learns style before substance. A pure binary correctness reward is unhackable by style but gives no signal among correct answers, so answers grow long and tool calls multiply. Perplexity's research post gates the preference score on correctness (R = r_base x s - pen_eff), while its efficiency penalty applies to every rollout. I would go further and gate the cost too: zero if wrong, else 1 plus a preference bonus minus a cost penalty capped at 0.2, so a correct answer scores at least 0.8 and always beats a wrong one. The tradeoff of my variant: wrong rollouts pay no cost, so the policy gets no pressure to make failed searches short; the post's version does push wrong answers to be short.

The penalty is normalized by a budget (8 calls, 400 answer tokens) and capped, so one runaway episode cannot dominate. The judge score is clipped to [0, 1], because judges sometimes return out-of-range numbers. With GRPO, only differences inside a group matter, so the gap between the gate's zero and a correct answer's minimum of 0.8 defines the main contrast, and the bonus and penalty break ties among correct rollouts.

Code:

```python
from dataclasses import dataclass

@dataclass
class Episode:
    correct: bool        # from a verifier or calibrated judge, never from the policy itself
    pref: float          # preference / rubric score in [0, 1]
    tool_calls: int
    answer_tokens: int

def reward(ep, budget_calls=8, budget_tokens=400, w_pref=0.3, w_cost=0.2):
    if not ep.correct:
        return 0.0  # gate: style or brevity cannot buy reward on a wrong answer
    over_calls = max(0, ep.tool_calls - budget_calls) / budget_calls
    over_tokens = max(0, ep.answer_tokens - budget_tokens) / budget_tokens
    penalty = w_cost * min(1.0, over_calls + over_tokens)
    return 1.0 + w_pref * min(max(ep.pref, 0.0), 1.0) - penalty  # w_cost <= 0.2 keeps it >= 0.8

if __name__ == "__main__":
    wrong_pretty = Episode(False, 1.0, 2, 100)
    right_plain = Episode(True, 0.0, 6, 300)
    right_slow = Episode(True, 0.0, 30, 300)
    right_good = Episode(True, 0.9, 5, 250)
    assert reward(wrong_pretty) == 0.0
    assert reward(right_slow) < reward(right_plain) < reward(right_good)
    assert reward(Episode(True, 0.0, 10**6, 10**6)) == 0.8 > reward(wrong_pretty)
    assert reward(Episode(True, 5.0, 1, 1)) == 1.3  # out-of-range judge score clipped
    print("ok", reward(right_plain), reward(right_slow), reward(right_good))
```

Verify: a wrong but highly preferred answer gets 0, among correct answers fewer calls and better preference rank higher, an absurd cost still leaves a correct answer at exactly 0.8, and an out-of-range judge score is clipped.

Pitfalls:
- Letting the policy's own output decide correctness (self-grading).
- A cost penalty with no cap, which teaches the agent to answer without searching.

Follow-ups: what if the correctness verifier has a 5 percent false positive rate (the gate then leaks; estimate how much reward hacking it allows), and how to set the call budget per query difficulty.

Why this is likely: reward design is central to the listing's post-training work, and Perplexity's research blog describes a correctness-gated preference reward with an efficiency penalty for search agents; core expertise rounds go deep on the named domain (official format, team blog, medium-high confidence).

</details>

<a id="q8"></a>

#### Q8. You have 500,000 synthetic multi-hop prompts and pass counts from 8 solver attempts each. Choose which prompts to train on with GRPO at group size 8, and estimate how much rollout compute your choice saves.

Round: Core expertise. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Curriculum and data selection.

<details>
<summary>Show answer</summary>

Task: a selection rule, the math for wasted groups, and code.

What they grade: connecting pass rate to gradient signal, numbers, and awareness of what filtering removes.

Approach: With binary reward and pass rate p, a GRPO group of size G has all-equal rewards, hence zero advantage, with probability p to the power G plus (1 - p) to the power G. At G = 8 that is 0.78 percent for p = 0.5 but 43 percent for p = 0.9, and 100 percent for p = 0 or 1. So prompts the policy always or never solves cost full rollout compute and teach nothing. The expected per-prompt signal is roughly the Bernoulli variance p(1 - p), which peaks at 0.5.

Three options. Uniform sampling is simplest but in the example below wastes about 60 percent of groups. A hard band filter (keep 0.05 to 0.95) removes the dead prompts. Weighting by p(1 - p) inside the band also prefers the most informative ones. I pick band plus weighting, re-estimating p every few hundred steps because the policy improves and yesterday's 0.5 prompts become today's 0.95 prompts. The pass counts came from solvers, not the current policy, so I would refresh them with a cheap 4-sample probe from the policy itself.

Code:

```python
import numpy as np

def wasted_group_rate(p, G):
    """Chance that all G rollouts get the same binary reward, so every advantage is 0."""
    return p ** G + (1 - p) ** G

def select(pass_counts, k, G=8, keep_band=(0.05, 0.95), budget=None):
    p = np.asarray(pass_counts) / k
    keep = (p >= keep_band[0]) & (p <= keep_band[1])
    weight = np.where(keep, p * (1 - p), 0.0)  # Bernoulli reward variance: the gradient signal
    order = np.argsort(-weight, kind="stable")
    chosen = [int(i) for i in order if weight[i] > 0][:budget]
    return chosen, float(np.mean([wasted_group_rate(p[i], G) for i in chosen])) if chosen else 1.0

if __name__ == "__main__":
    assert np.isclose(wasted_group_rate(0.5, 8), 2 / 256)
    assert round(wasted_group_rate(0.9, 8), 3) == 0.43
    counts = [0, 8, 4, 1, 7, 2, 8, 0]  # passes out of k=8 solver attempts per prompt
    chosen, waste = select(counts, k=8, budget=4)
    assert chosen == [2, 5, 3, 4], chosen
    all_waste = np.mean([wasted_group_rate(c / 8, 8) for c in counts])
    assert waste < 0.4 < all_waste, (waste, all_waste)
    print("ok", chosen, round(waste, 3), round(float(all_waste), 3))
```

Verify: the closed form matches 2/256 at p = 0.5, the 0.9 case gives 0.43, and the selected set has a wasted-group rate of about 0.2 versus 0.6 for the unfiltered set.

Pitfalls:
- Discarding p = 0 prompts forever; some are mislabeled and some become solvable later.
- Filtering by solver pass rate when the policy is much weaker than the solvers.

Follow-ups: rubric rewards are continuous, so how the analysis changes, and whether pass@4 filtering on rubric prompts helps.

Why this is likely: Perplexity's research post describes synthetic multi-hop QA with multi-solver verification and pass@4 filtering, and the listing names post-training and RL; core expertise rounds ask for this depth (team blog, medium confidence).

</details>

<a id="q9"></a>

#### Q9. Train reward climbs while held-out accuracy falls. Diagnose the run from these logged metrics, name the root cause you would test first, and say which checkpoint you would keep.

Round: Core expertise. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: RL training diagnostics.

```text
step  train_reward  heldout_acc  entropy  resp_len  tool_calls  clip_frac  train_infer_kl
0     0.31          0.42         1.90     410       3.1         0.04       0.002
200   0.38          0.46         1.70     430       3.4         0.05       0.002
400   0.45          0.49         1.40     480       3.9         0.06       0.003
600   0.52          0.50         1.00     620       5.2         0.09       0.006
800   0.61          0.47         0.55     900       8.8         0.15       0.019
1000  0.70          0.43         0.30     1350      13.0        0.22       0.041
```

<details>
<summary>Show answer</summary>

Task: order the symptoms in time, pick the root cause hypothesis, and plan the cheapest experiment to confirm it.

What they grade: reading curves as a sequence, separating cause from consequence, and a concrete next experiment.

Approach: The first metric to move is response length, up 51 percent by step 600, together with tool calls (3.1 to 5.2), while held-out accuracy is still flat. Accuracy then falls from step 800, entropy collapses below half its start, and clip fraction and the train and inference KL rise. My reading is that the policy found a way to raise train reward through longer, more tool-heavy episodes that do not generalize. The two candidate causes are a reward hack, for example a lenient grader or a preference judge that likes long cited answers, and a length bias in the loss normalization. The KL rise looks like a consequence, because long, low-entropy sequences push the sampler and trainer apart, which then adds off-policy noise.

Test first: sample 50 rewarded train episodes from step 1000 and 50 from step 400 and grade them with the strict held-out grader. If train reward exceeds strict accuracy by a growing margin, it is the reward. That costs an hour. The alternative, rerunning with token-level normalization, costs a day of compute, so it goes second. Keep checkpoint 600, the best held-out accuracy.

Code:

```python
METRICS = [  # step, train_reward, heldout_acc, entropy, resp_len, tool_calls, clip_frac, train_infer_kl
    (0, 0.31, 0.42, 1.90, 410, 3.1, 0.04, 0.002),
    (200, 0.38, 0.46, 1.70, 430, 3.4, 0.05, 0.002),
    (400, 0.45, 0.49, 1.40, 480, 3.9, 0.06, 0.003),
    (600, 0.52, 0.50, 1.00, 620, 5.2, 0.09, 0.006),
    (800, 0.61, 0.47, 0.55, 900, 8.8, 0.15, 0.019),
    (1000, 0.70, 0.43, 0.30, 1350, 13.0, 0.22, 0.041),
]
COLS = ["step", "reward", "acc", "entropy", "len", "calls", "clip", "kl"]

def first_divergence(rows, acc_drop=0.02):
    rows = [dict(zip(COLS, r)) for r in rows]
    best, out = rows[0], {}
    for prev, cur in zip(rows, rows[1:]):
        if cur["acc"] > best["acc"]:
            best = cur
        if "reward_acc_split" not in out and cur["reward"] > prev["reward"] and best["acc"] - cur["acc"] >= acc_drop:
            out["reward_acc_split"] = cur["step"]
        if "entropy_collapse" not in out and cur["entropy"] < 0.5 * rows[0]["entropy"]:
            out["entropy_collapse"] = cur["step"]
        if "length_blowup" not in out and cur["len"] > 1.5 * rows[0]["len"]:
            out["length_blowup"] = cur["step"]
        if "mismatch" not in out and cur["kl"] > 3 * rows[0]["kl"]:
            out["mismatch"] = cur["step"]
    out["best_ckpt"] = best["step"]
    return out

if __name__ == "__main__":
    d = first_divergence(METRICS)
    assert d == {"length_blowup": 600, "reward_acc_split": 800, "entropy_collapse": 800,
                 "mismatch": 800, "best_ckpt": 600}, d
    print("ok", d)
```

Verify: the script reports length blowup at 600 before the reward and accuracy split at 800, and picks checkpoint 600.

Pitfalls:
- Treating the entropy collapse as the cause and adding an entropy bonus, which hides the hack.
- Selecting checkpoints on train reward.

Follow-ups: adding a length penalty versus fixing the grader, and alerting rules on the train minus strict-eval gap.

Why this is likely: the listing asks for diagnosing failures and translating results into improvements, and Perplexity's core expertise page says experienced candidates are taken deep on their own domain (listing plus official page, medium-high confidence).

</details>

<a id="q10"></a>

#### Q10. One policy trains on two reward sources: verifiable multi-hop QA with a binary reward, and open-ended queries scored by a rubric judge. With a 50/50 prompt mix, verifiable accuracy stalls. Work out why and set the mixture.

Round: Core expertise. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Multi-objective RL.

<details>
<summary>Show answer</summary>

Task: explain the imbalance quantitatively and compute a mixture that gives a target share of gradient to each source.

What they grade: thinking in terms of where the gradient mass comes from, not prompt counts, and a measurable fix.

Approach: After per-group normalization every informative group contributes advantages of similar magnitude, so gradient mass per source is roughly prompts times the fraction of informative groups times response tokens. Binary QA groups are often all-pass or all-fail; say 35 percent are informative, with 300 token answers. Rubric scores are continuous, so groups rarely tie, say 95 percent informative, with 900 token answers. At 50/50 the rubric source then carries about 89 percent of the gradient, which explains the stall.

Three fixes. Per-source loss weights work but interact with the optimizer and the clip fraction, and are hard to reason about. Separate policies avoid interference but double serving cost and lose transfer. Changing the prompt mixture keeps one loss and is easy to monitor, so I solve for the mixture that gives a target share, here 60/40 in favour of verifiable. It comes out near 92/8, a skewed mixture of the kind Perplexity's research post describes. Then I check it empirically by logging per-source gradient norms, because these informative rates drift as the policy learns.

Code:

```python
def grad_share(mix, informative, tokens):
    """Rough share of policy-gradient mass per source: prompts x informative groups x response tokens."""
    mass = {s: mix[s] * informative[s] * tokens[s] for s in mix}
    tot = sum(mass.values())
    return {s: round(m / tot, 3) for s, m in mass.items()}

def mix_for_target(target, informative, tokens):
    raw = {s: target[s] / (informative[s] * tokens[s]) for s in target}
    tot = sum(raw.values())
    return {s: round(v / tot, 3) for s, v in raw.items()}

if __name__ == "__main__":
    informative = {"verifiable": 0.35, "rubric": 0.95}  # rubric scores are continuous, so groups rarely tie
    tokens = {"verifiable": 300, "rubric": 900}
    even = grad_share({"verifiable": 0.5, "rubric": 0.5}, informative, tokens)
    assert even["rubric"] > 0.85, even
    m = mix_for_target({"verifiable": 0.6, "rubric": 0.4}, informative, tokens)
    assert abs(grad_share(m, informative, tokens)["verifiable"] - 0.6) < 0.01, m
    assert m["verifiable"] > 0.9, m
    print("ok", even, m)
```

Verify: the 50/50 case gives more than 85 percent rubric share, and the solved mixture lands within 1 point of the 60 percent target.

Pitfalls:
- Balancing by prompt count.
- Ignoring that the informative rate changes during training, so a fixed mixture drifts.

Follow-ups: per-source advantage scaling instead of mixture, and how token-level normalization changes the formula.

Why this is likely: Perplexity's research post on search-augmented training describes combining objectives and reweighting the prompt mixture; core expertise goes this deep (team blog, medium confidence).

</details>

<a id="q11"></a>

#### Q11. Many search queries have no verifiable answer. Design rubric-based rewards for them, then build the calibration check that decides whether the rubric judge is good enough to train against.

Round: Core expertise. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: LLM judges, reward models, evaluation.

<details>
<summary>Show answer</summary>

Task: a rubric design, a calibration protocol with pass and fail thresholds, and code for the key metrics.

What they grade: eval rigor, judge bias awareness, and thresholds tied to training risk.

Approach: For the rubric, generate 4 to 8 checkable criteria per query (covers the key entities, cites a source for each claim, no unsupported numbers, answers the actual question) and have the judge score each one, rather than asking for one holistic score. Itemized rubrics are easier to audit, and the reward becomes the weighted fraction met. I would filter rubric prompts by sampling 4 policy answers and dropping prompts where all score the same, the pass@4 idea, because ties give zero GRPO advantage.

Calibration: 600 query pairs labelled by two expert raters, each pair compared head to head. Report rater agreement first, because it is the ceiling. Then judge agreement on non-tied pairs, counting judge near-ties as misses; position bias by scoring each pair in both orders; length bias by correlating score with length on pairs humans tied. Before training against the judge, I require pairwise agreement within 5 points of the raters' agreement, order flips under 10 percent, and no significant length slope.

Alternatives: a trained reward model is cheaper at inference but needs tens of thousands of preference pairs and drifts out of distribution as the policy moves. An off-the-shelf LLM judge with a rubric needs no training data but is slower. I would start with the judge and distil it once the policy stabilizes.

Code:

```python
def pairwise_agreement(judge, human, tie_margin=0.05):
    """judge/human: lists of (score_a, score_b). Human ties are skipped; judge ties count as misses."""
    agree = n = 0
    for (ja, jb), (ha, hb) in zip(judge, human):
        if ha == hb:
            continue
        n += 1
        if abs(ja - jb) > tie_margin and (ja > jb) == (ha > hb):
            agree += 1
    return agree / n if n else float("nan")

def position_bias(judge_ab, judge_ba):
    """Share of pairs where swapping the order flips the winner."""
    flips = sum((a1 > b1) != (a2 > b2) for (a1, b1), (b2, a2) in zip(judge_ab, judge_ba))
    return flips / len(judge_ab)

if __name__ == "__main__":
    human = [(1, 0), (0, 1), (1, 0), (1, 1), (0, 1)]
    judge = [(0.9, 0.2), (0.3, 0.8), (0.52, 0.5), (0.7, 0.1), (0.8, 0.4)]
    assert pairwise_agreement(judge, human) == 0.5  # 2 of 4 non-tied pairs; the near-tie counts as a miss
    ab = [(0.9, 0.2), (0.6, 0.5)]
    ba = [(0.3, 0.8), (0.6, 0.5)]  # second pair: whichever answer is shown first wins
    assert position_bias(ab, ba) == 0.5
    print("ok")
```

Verify: a near-tie counts as a miss, so agreement on the fixture is 2 of 4, and swapping order on a biased pair shows a 50 percent flip rate.

Pitfalls:
- Calibrating once and never again while the policy learns to exploit the judge.
- Letting the judge see which answer came from the new policy.

Follow-ups: re-auditing 100 high-reward samples every 500 steps, and an ensemble of two judges to cut variance.

Why this is likely: the listing asks for rigorous evaluations, and Perplexity's research post describes rubric rewards and pass@4 filtering for non-verifiable queries (team blog plus listing, medium confidence).

</details>

### System design

<a id="q12"></a>

#### Q12. Design the RL training system for a search agent: rollout workers, a search sandbox, a reward service and the trainer. Target 512 prompts per step at group size 8 with episodes of up to 15 tool calls.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: RL infrastructure, agent harness.

<details>
<summary>Show answer</summary>

Task: components, data flow, numbers, and the sync versus async tradeoff.

What they grade: quantitative sizing, the long-tail rollout problem, reproducible search, and reward isolation.

Approach: The system has five components: a prompt sampler with the pass-rate curriculum; inference workers that run the agent harness; a search sandbox that serves a frozen index snapshot plus a cache of live fetches; a reward service with verifiers and the judge; and a trainer that consumes groups and pushes new weights to the inference workers.

Numbers: 512 prompts x 8 = 4,096 episodes per step. At about 10 tool calls each that is 41,000 search calls per step; at one step every 3 minutes, about 230 searches per second. With 3,000 tokens per episode, generation is 12 million tokens per step. Live web search at that rate is slow, nondeterministic and expensive, so the sandbox serves an offline snapshot for most training and a recorded cache for live fetches; that also makes a run reproducible.

Alternatives: synchronous on-policy steps are clean but the trainer waits for the slowest episode, and with 15-call episodes the p99 can be 5 times the median, so GPUs idle about half the time. Fully asynchronous rollouts keep GPUs busy but the data becomes stale. I pick bounded asynchrony, where rollouts may be at most 1 to 2 policy versions old, with the token-level importance correction from Q4, and episodes beyond a time limit are truncated and dropped rather than waited for.

Risks: reward service outages stall training, so rewards are computed asynchronously with retries and a dead-letter queue; the search snapshot ages, so refresh it periodically and evaluate on live search separately; inference and trainer numerics diverge, so log train and inference KL per step and alert on it.

Pitfalls:
- Counting tool latency as free.
- Training on live search results that change between rollouts in the same group, which adds noise to the advantage.

Follow-ups: how to shard the index snapshot, and how to resume after a trainer crash without duplicating groups.

Why this is likely: the listing says Perplexity controls the models, agent harnesses and search infrastructure and lists distributed training and scalable inference, and system design rounds use quantitative scaling (listing plus official format, medium-high confidence).

</details>

<a id="q13"></a>

#### Q13. Design execution for long-running agentic queries (research tasks of 5 to 30 minutes with up to 200 tool calls) that survive worker crashes, deploys and budget limits, and that let the user see progress.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Durable agent execution.

<details>
<summary>Show answer</summary>

Task: an architecture with state model, recovery, budgets and numbers.

What they grade: durable state, idempotent side effects, and capacity math.

Approach: Each query is a task with an append-only event log: plan, tool call requested, tool result, model output, checkpoint. A scheduler leases tasks to stateless workers. A worker replays the log to rebuild context, runs one step, appends the result and renews the lease. If a worker dies, its lease expires and another worker resumes from the last event, so a crash loses at most one step. Tool calls carry an idempotency key (task id plus step number), and results are cached under that key so replay never repeats a paid call.

Numbers: 10,000 concurrent tasks at one step every 6 seconds is about 1,700 steps per second; at 4 KB per event and 200 events per task, a task log is about 800 KB and 10,000 live tasks hold 8 GB, which fits a sharded key-value store or Postgres partitioned by task id. Model context is rebuilt from a rolling summary plus the last N events so it stays under the window.

Alternatives: in-process async with no persistence is simplest but every deploy kills running tasks, which at 30-minute tasks and daily deploys is unacceptable. A workflow engine such as Temporal gives replay and timers for free but adds an operational dependency. Its history cap (about 50K events or 50 MB) is not hit at 200 steps x 4 KB, but large tool payloads must live outside history (a blob store, keeping only references), and continue-as-new is needed if tasks grow past it. A custom event log on an existing store is more code but fits the data model. I would start with the workflow engine if the team already runs it, otherwise the event log.

Risks: budget overrun, so enforce token, tool and wall clock budgets at each step and end with a partial answer rather than failing; poison tasks that crash every worker, so cap retries per step and park them; deploys, so drain by refusing new leases and letting in-flight steps finish.

Pitfalls:
- Checkpointing the whole context each step instead of events.
- Non-idempotent tool calls replayed on resume.

Follow-ups: streaming partial results to the client, and cancellation semantics.

Why this is likely: a candidate-report aggregator and a forum summary both list long-running agentic query execution as a Perplexity onsite system design prompt (candidate reports, medium-high confidence).

</details>

<a id="q14"></a>

#### Q14. A new first-stage retriever has better recall@100 on the offline benchmark. Design the eval that decides whether it ships for the search agent, including what you do when retrieval metrics and agent accuracy disagree.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Retrieval evaluation, eval design.

<details>
<summary>Show answer</summary>

Task: offline metrics, downstream agent eval, statistics and a ship rule.

What they grade: linking retrieval metrics to agent outcomes, multi-hop subtleties, and paired statistics.

Approach: I evaluate in three layers. The first is retrieval recall at k on queries with labelled gold documents, but for multi-hop questions count a query only when every hop's document is retrieved, because finding one of two hops still fails the task. The second is the agent end to end. Run the same agent policy with each retriever on 2,000 held-out questions and grade answers with the strict grader. The third is cost, meaning tool calls per answered question and latency, because a weaker retriever often shows up as more searches rather than lower accuracy.

If recall rises but agent accuracy does not, check whether the extra recall sits at ranks 50 to 100 that the agent never reads (it reads the top 10), so measure recall at the k the agent consumes. Also the policy was trained with the old retriever and may have learned compensating query styles; a short RL fine-tune with the new retriever is the fair comparison.

Statistics: use a paired comparison on the same questions, because per-question difficulty dominates variance. A paired bootstrap on per-question 0/1 outcomes gives a confidence interval for the delta. Ship when the lower bound on accuracy is above zero and tool calls rise by no more than 10 percent.

Alternatives: an online A/B on live traffic is the final word but needs about 38,000 queries per arm to see a 1 point change (two-sided alpha 0.05, 80% power, baseline accuracy near 60 percent), while a paired offline comparison needs far fewer; interleaving is more sensitive for ranking but does not apply cleanly to an agent that issues its own queries. Offline paired first, then a small online test.

Code:

```python
import random

def recall_at_k(ranked, gold_sets, k):
    """Multi-hop: a query counts only if every gold hop document is in the top k."""
    return sum(set(g) <= set(r[:k]) for r, g in zip(ranked, gold_sets)) / len(gold_sets)

def paired_bootstrap(a, b, n=2000, seed=0):
    """95% CI of mean(b - a) over the same queries (per-query 0/1 outcomes)."""
    rng, d = random.Random(seed), [y - x for x, y in zip(a, b)]
    means = sorted(sum(rng.choice(d) for _ in d) / len(d) for _ in range(n))
    return means[int(0.025 * n)], means[int(0.975 * n) - 1]

if __name__ == "__main__":
    ranked = [["d1", "d9", "d2"], ["d4", "d5", "d6"], ["d7", "d8", "d3"]]
    gold = [["d1", "d2"], ["d5"], ["d3", "d0"]]
    assert recall_at_k(ranked, gold, 2) == 1 / 3 and recall_at_k(ranked, gold, 3) == 2 / 3
    old = [1] * 60 + [0] * 40
    new = [1] * 60 + [1] * 8 + [0] * 32  # 8 queries fixed, none broken
    lo, hi = paired_bootstrap(old, new)
    assert 0 < lo < 0.08 < hi, (lo, hi)
    print("ok", round(lo, 3), round(hi, 3))
```

Verify: multi-hop recall counts only complete hops, and 8 fixed questions out of 100 give an interval that excludes zero.

Pitfalls:
- Reporting mean recall at 100 when the agent reads 10.
- Unpaired comparisons, which need several times the sample for the same interval width.

Follow-ups: building gold labels at scale with LLM relevance judgments, and contamination between benchmark and training data.

Why this is likely: Perplexity's research index includes work on evaluating first-stage retrievers at scale, and the listing names search and retrieval and rigorous evaluation (team research plus listing, medium confidence).

</details>

<a id="q15"></a>

#### Q15. A planner agent splits a research query and delegates to several searcher subagents that run in parallel. Only the final answer gets a reward. Design how you train this system and assign credit.

Round: Core expertise. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Multi-agent training, credit assignment.

<details>
<summary>Show answer</summary>

Task: a training setup, a credit assignment scheme, and how you evaluate it.

What they grade: understanding why shared terminal reward is weak, concrete alternatives, and evaluation that isolates each role.

Approach: The baseline is to train planner and searchers as one policy with the shared final reward. It is simple, but the credit signal is weak. With 4 searchers, a failure from one bad subtask penalizes all four and the planner equally.

Alternatives:
1. Shared reward with group normalization per query: sample G full multi-agent rollouts per query, so the advantage compares whole team trajectories. Cheap, still coarse.
2. Per-role intermediate rewards: score each searcher on whether its returned evidence contains the facts its subtask needed, checkable for synthetic multi-hop data where the hop answers are known; score the planner on decomposition quality and the final answer. Denser, but intermediate rewards can be gamed and need their own calibration.
3. Counterfactual credit: replace one searcher's output with a fresh sample and measure the change in final reward. That is the most precise but costs one extra rollout per searcher, so 5 times the compute with 4 searchers.
I would use 1 plus 2: group-normalized final reward for everyone, plus a hop-level reward for searchers where the synthetic data provides hop answers, with a small weight like 0.3 so the final answer stays dominant. I would use counterfactual replays only as an analysis tool on a sample.

Numbers: with G = 8 and 4 searchers, each step costs 8 planner rollouts plus 32 searcher rollouts per query, so searcher tokens dominate and set the batch size.

Risks: the planner learns to make trivial subtasks that searchers always solve; penalize decompositions whose subtasks do not cover the gold hops. Searchers collude through redundancy; measure overlap between their queries.

To evaluate, swap one trained component at a time back to the base model and see which role improved.

Pitfalls:
- Sharing weights without role tokens, so the model cannot tell which job it is doing.

Follow-ups: whether to train the planner and searchers on the same step or alternate.

Why this is likely: the listing names multi-agent training, coordination and evaluation as a nice-to-have, and Perplexity's research index includes teacher-student self-distillation work (listing plus team research, medium confidence).

</details>

### Behavioral

<a id="q16"></a>

#### Q16. Walk through an open-ended idea you carried to a working system. Then: how do you use coding agents day to day, and when did an agent's output look right but turn out wrong?

Round: Hiring manager deep dive. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Ownership, research judgment, AI use.

<details>
<summary>Show answer</summary>

Task: one story with numbers, then a concrete account of AI use and verification.

What they grade: ownership from idea to production, the judgment calls, and honest, specific verification habits, since the agentic round expects frontier agents.

Approach: Structure it as problem, why it mattered, the decision points, the result and what you would change. A strong example shape: "Our search agent hedged answers on multi-hop questions. I suspected the grader. I sampled 200 rewarded episodes, found 14 percent were hedges, rewrote the grader to extract a single answer, and reran a small RL job. Strict accuracy rose 3 points and answer length fell by a third. I then added a hedge-rate alert to every run." Name what you chose not to do and why, for example a learned hedge classifier, because rules were enough at that error rate.

For coding agents, describe a real loop: you write the spec and tests first, delegate scoped tasks (a data loader, a metric, a refactor), and read every diff before running it. Name what you check: invented APIs, silent changes to shapes or masks, sampling instead of full runs, and tests that the agent modified to pass.

In one example of a wrong output, the agent "fixed" a failing advantage test by loosening the tolerance from 1e-6 to 1e-2. The diff was one line and the test passed, but the bug, a wrong reshape that mixed groups, was still there. You caught it because you diffed tests separately from code, and you now forbid test edits in the same change as the fix.

Pitfalls:
- A story with no numbers or no personal decisions.
- Claiming you never use AI, or that you trust its output without checks.

Follow-ups: what you would do differently, and how you decide when to stop exploring and ship.

Why this is likely: Perplexity's interview guide includes a hiring manager deep dive, its agentic round expects frontier coding agents, and the listing asks for ownership from idea to working system (official guide plus listing, high confidence for the round, medium for the AI angle).

</details>

## Prep checklist

- [ ] Solve Perplexity's public todo list practice exercise in Python in under 60 minutes without AI, keeping earlier parts' tests green.
- [ ] Build the Q2 file system from a blank file in 45 minutes, then add snapshots without breaking part 1.
- [ ] Write the Q1 rollout collector from memory, including timeouts, retries and the zero-variance filter.
- [ ] Implement the GRPO loss with per-group advantages, clipping and a response mask in NumPy, with the Q3 tests.
- [ ] Derive the next-token shift and token-level truncated importance weights by hand, then review Q4's code again cold.
- [ ] Debug three stored agent trajectories by hand and write a streaming detector for repeated calls, tool errors and hedged answers.
- [ ] Do one timed multi-file data-quality task with a coding agent, writing the fixture first and logging each wrong output you caught.
- [ ] Compute wasted-group rates for several pass rates and group sizes, and solve a two-source mixture by hand.
- [ ] Practice 45 minute designs for an RL rollout system, durable long-running agent execution and a retriever ship decision, each with numbers.
- [ ] Read Perplexity's post on advancing search-augmented language models and its research index, and prepare to discuss one paper in depth.
- [ ] For the possible ML coding screen, take a 15-line scikit-learn pipeline that fits the scaler on all data and reports accuracy on an imbalanced set; fix the leakage, switch to a stratified split and report precision, recall and PR AUC, without AI.
- [ ] Prepare a 90 minute deep dive on one system you shipped, with metrics, decisions and a story about catching a wrong agent output.

## Sources

- https://interviewresources.perplexity.ai/hands-on-coding
- https://interviewresources.perplexity.ai/agentic-coding/
- https://interviewresources.perplexity.ai/core-expertise/
- https://interviewresources.perplexity.ai/spike-round/
- https://www.perplexity.ai/hub/careers/interview-guide
- https://www.perplexity.ai/hub/blog/advancing-search-augmented-language-models
- https://research.perplexity.ai
- https://darkinterview.com/collections/perplexity
- https://www.jointaro.com/interviews/companies/perplexity?tab=experiences
- https://www.reddit.com/r/Hack2Hire/comments/1us3j19/perplexity_interview_process_experience/
- https://www.1point3acres.com/interview/thread/1193087
- https://www.1point3acres.com/interview/thread/1155582
- https://prachub.com/companies/perplexity/positions/machine-learning-engineer
- https://www.yuan-meng.com/posts/mle_interviews_2.0/
- https://www.finalroundai.com/blog/perplexity-interview-process
- https://arxiv.org/abs/2402.03300
- https://arxiv.org/abs/2503.20783
