# Meta: AI Research Scientist, Robotics (Meta Superintelligence Labs)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Meta says research scientist loops vary by org, so confirm the round mix with your recruiter. No robotics-specific report for this lab was found; the loop below combines recent Meta research scientist and ML research scientist reports with the team's published research.

1. Recruiter screen. 30 minutes, phone or video. Background, research areas, team fit. AI: not applicable.
2. Online assessment (some roles). 60 to 70 minutes, CodeSignal style, four algorithm problems from easy to hard. AI: not allowed.
3. Technical or research phone screen. 45 minutes in CoderPad. Usually two Meta-tagged algorithm problems; some newer screens instead discuss a system you built and then run AI-assisted coding. Research variants add ML fundamentals and experiment design. AI: not allowed in the classic variant; provided (CoderPad assistant) in the newer one.
4. Onsite classic coding. 45 minutes, two LeetCode-style problems (medium, sometimes hard). One recent research scientist report names a BFS maze and lowest common ancestor. AI: not allowed.
5. Onsite AI-enabled coding. 60 minutes, a multi-file CoderPad project with tests: fix bugs, write tests, then extend. The built-in assistant has a model picker, and interviewers can ask you to work without it for part of the round. AI: provided (CoderPad assistant); outside AI tools not allowed.
6. ML or research system design. 45 minutes in CoderPad with Mermaid. Research scientists have reported product ML prompts; for a robotics team expect a data, model and evaluation design for an embodied system. AI: provided (CoderPad assistant with Mermaid).
7. Research deep dive or presentation. 45 to 60 minutes. You defend your papers, baselines and failed experiments, and say how the work would scale with Meta's resources. AI: not allowed.
8. ML coding and fundamentals. 45 minutes. Implement a model piece by hand (attention, a loss, a training step) in PyTorch or NumPy, and debug training problems. Reports say this lab leans toward hand-coded models and past research. AI: not allowed.
9. Behavioral. 45 minutes, the classic Meta behavioral plus an AI-oriented behavioral. AI: not allowed.
10. Hiring committee, then team match and offer. AI: not applicable.

Every report found ran the loop virtually. The questions below follow that mix: two classic problems, three AI-enabled coding tasks, three ML coding and fundamentals tasks, four design and evaluation questions, three research deep dive questions and one behavioral question.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Shortest path for a robot on an occupancy grid, then forbid diagonal corner cutting | Classic algorithm | Onsite classic coding | Not allowed | Medium |
| [Q2](#q2) | Count the joints between two links of a robot's kinematic tree, for many queries | Classic algorithm | Phone screen or onsite classic coding | Not allowed | Medium |
| [Q3](#q3) | Fix the action-chunk replay buffer that mixes episodes, then pad chunks at episode ends | Debug | AI-enabled coding | Provided (CoderPad assistant) | Medium |
| [Q4](#q4) | Review this AI-generated GAE and PPO loss before it trains a locomotion policy | Review | AI-enabled coding | Provided (CoderPad assistant) | Medium |
| [Q5](#q5) | Extend the grid planner so the robot can pick up keycards and open doors, and return the path | Build | AI-enabled coding | Provided (CoderPad assistant) | Hard |
| [Q6](#q6) | This bf16 VLA run diverged at step 3461. Read the log, name the cause, add a guard | Trace debug | ML coding and fundamentals | Not allowed | Hard |
| [Q7](#q7) | Implement causal multi-head attention for an action-token policy in NumPy | Build | ML coding and fundamentals | Not allowed | Medium |
| [Q8](#q8) | Implement the training step of a diffusion policy and check the noise schedule | Concept plus code | ML coding and fundamentals | Not allowed | Medium |
| [Q9](#q9) | Design the data flywheel and training stack for a generalist manipulation policy | Design | ML or research system design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q10](#q10) | Policy A won 14 of 20 real trials, policy B won 10 of 20. Design the eval that settles it | Concept plus code | ML or research system design | Provided (CoderPad assistant with Mermaid) | Medium |
| [Q11](#q11) | A world-model planner reaches goals but fails pick-and-place. Read the planner log and fix it | Trace debug | ML or research system design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q12](#q12) | Build an LLM planner eval for multi-agent home tasks and sort its failures | Concept plus code | ML or research system design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q13](#q13) | Your in-hand rotation policy drops 18% of objects. Add touch to fix slip | Design | Research deep dive | Not allowed | Hard |
| [Q14](#q14) | Defend the sim-to-real claim in your talk against a skeptical interviewer | Design | Research deep dive or presentation | Not allowed | Hard |
| [Q15](#q15) | Plan a research program for long-horizon mobile manipulation with milestones | Design | Research deep dive | Not allowed | Hard |
| [Q16](#q16) | How do you use AI tools in your research, and where do you refuse to trust them? | Behavioral | Behavioral | Not allowed | Medium |

### Classic coding (no AI, as at Meta)

<a id="q1"></a>

#### Q1. Shortest path for a robot on an occupancy grid, then forbid diagonal corner cutting

Round: Onsite classic coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: BFS on grids.
Related practice: [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) (Medium), [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) (Medium).

A robot lives on an occupancy grid (0 free, 1 occupied). It moves to any of its 8 neighbours at cost 1. Return the fewest moves from `start` to `goal`, or -1. Follow-up: a diagonal move is illegal if either of the two cells it squeezes between is occupied.

<details>
<summary>Show answer</summary>

Task: unit-cost shortest path on a grid, then add a motion constraint without breaking optimality.

What they grade: picking BFS over Dijkstra and saying why, clean neighbour handling, edge cases, the follow-up as a local rule.

Approach: Every move costs 1, so breadth-first search returns the optimal count the first time it pops the goal. Dijkstra gives the same answer with a heap, O(V log V) instead of O(V), so it only wins once diagonal moves cost sqrt(2). A* with the Chebyshev distance heuristic expands fewer cells on open maps, but it adds code you must get right on a whiteboard. On a 1000 by 1000 grid BFS touches at most a million cells, which is fine.

Mark cells visited when you push them, not when you pop them; otherwise a cell can enter the queue up to 8 times. The corner-cut rule only changes which neighbours are legal: for a diagonal step (dr, dc), both (r, c+dc) and (r+dr, c) must be free. It stays a local check, so BFS remains correct.

Code:

```python
from collections import deque

def shortest_path(grid, start, goal, no_corner_cut=False):
    R, C = len(grid), len(grid[0])
    if grid[start[0]][start[1]] or grid[goal[0]][goal[1]]:
        return -1
    dist = {start: 0}
    q = deque([start])
    while q:
        r, c = q.popleft()
        if (r, c) == goal:
            return dist[(r, c)]
        for dr in (-1, 0, 1):
            for dc in (-1, 0, 1):
                if dr == 0 and dc == 0:
                    continue
                nr, nc = r + dr, c + dc
                if not (0 <= nr < R and 0 <= nc < C) or grid[nr][nc]:
                    continue
                if no_corner_cut and dr and dc and (grid[r][nc] or grid[nr][c]):
                    continue
                if (nr, nc) not in dist:
                    dist[(nr, nc)] = dist[(r, c)] + 1
                    q.append((nr, nc))
    return -1

g = [[0, 0, 0],
     [1, 1, 0],
     [1, 1, 0]]
assert shortest_path(g, (0, 0), (2, 2)) == 3
assert shortest_path(g, (0, 0), (2, 2), no_corner_cut=True) == 4
assert shortest_path(g, (0, 0), (0, 0)) == 0
assert shortest_path([[0, 1], [1, 0]], (0, 0), (1, 1), no_corner_cut=True) == -1
assert shortest_path([[1, 0], [0, 0]], (0, 0), (1, 1)) == -1
```

Verify: start equals goal, blocked start or goal, unreachable goal, the corner-cut case that turns 3 moves into 4, and a 1 by 1 grid.

Pitfalls: LeetCode 1091 counts cells on the path, not moves, so its answer is one higher; marking visited on pop; forgetting that the corner rule needs both side cells.

Follow-ups: diagonal cost sqrt(2) (switch to Dijkstra or A*); a robot with a 2-cell radius (inflate obstacles first); many goals (multi-source BFS from the goals).

Why this is likely: a recent Glassdoor Meta research scientist report lists a BFS maze problem in a no-AI onsite round, and grid BFS problems sit in the recent Meta-tagged LeetCode lists (medium confidence).

</details>

<a id="q2"></a>

#### Q2. Count the joints between two links of a robot's kinematic tree, for many queries

Round: Phone screen or onsite classic coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Lowest common ancestor.
Related practice: [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) (Medium), [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) (Easy).

A robot description gives `parent[i]` for each link (the root has -1). Answer many queries "how many joints lie between link a and link b?" A humanoid with two arms, two hands and a head is a typical input.

<details>
<summary>Show answer</summary>

Task: path length in a rooted tree, which is depth(a) + depth(b) - 2 * depth(lca(a, b)).

What they grade: reducing the question to LCA, picking a method by query count, iterative code that will not hit recursion limits.

Approach: For one query, climb from the deeper node until both depths match, then climb both together. That is O(h) per query and needs no preprocessing. For Q queries on n links, binary lifting stores the 2^k-th ancestor of every node: O(n log n) build and O(log n) per query. A third option is an Euler tour plus a sparse table for O(1) queries, which pays off only for millions of queries. A robot has tens of links, so the naive climb is fine there; I would still write binary lifting because the interviewer will grow n to 10^5 next.

Depths are computed without recursion: walk up until a node with a known depth, then fill the path back down. The lifting table maps the root to itself so jumps never fall off the tree.

Code:

```python
import random

def preprocess(parent):
    n = len(parent)
    root = parent.index(-1)
    depth = [-1] * n
    depth[root] = 0
    for i in range(n):
        path, j = [], i
        while depth[j] < 0:
            path.append(j)
            j = parent[j]
        for k in reversed(path):
            depth[k] = depth[parent[k]] + 1
    LOG = max(1, (n - 1).bit_length())
    up = [[p if p >= 0 else root for p in parent]]
    for _ in range(1, LOG):
        prev = up[-1]
        up.append([prev[prev[v]] for v in range(n)])
    return depth, up

def lca(a, b, depth, up):
    if depth[a] < depth[b]:
        a, b = b, a
    diff, k = depth[a] - depth[b], 0
    while diff:
        if diff & 1:
            a = up[k][a]
        diff >>= 1
        k += 1
    if a == b:
        return a
    for k in reversed(range(len(up))):
        if up[k][a] != up[k][b]:
            a, b = up[k][a], up[k][b]
    return up[0][a]

def joints_between(a, b, depth, up):
    return depth[a] + depth[b] - 2 * depth[lca(a, b, depth, up)]

# 0 base, 1 torso, 2 l_shoulder, 3 l_hand, 4 r_shoulder, 5 r_hand, 6 head_pan, 7 camera
parent = [-1, 0, 1, 2, 1, 4, 0, 6]
d, up = preprocess(parent)
assert joints_between(3, 5, d, up) == 4
assert joints_between(3, 7, d, up) == 5
assert joints_between(3, 3, d, up) == 0
assert joints_between(0, 0, *preprocess([-1])) == 0

def naive(a, b, parent):
    anc = {}
    x, k = a, 0
    while x != -1:
        anc[x] = k; x = parent[x]; k += 1
    x, k = b, 0
    while x not in anc:
        x = parent[x]; k += 1
    return k + anc[x]

random.seed(0)
p = [-1] + [random.randrange(i) for i in range(1, 300)]
d, up = preprocess(p)
for _ in range(500):
    a, b = random.randrange(300), random.randrange(300)
    assert joints_between(a, b, d, up) == naive(a, b, p)
```

Verify: a with itself, a node and its ancestor, a single-node tree, and 500 random queries against a brute-force walk.

Pitfalls: recursive DFS on a 10^5-deep chain overflows Python's stack; an off-by-one in LOG; forgetting that the root must map to itself in the table.

Follow-ups: joint limits as edge weights (store weighted depth); links added online (lifting still works for new leaves).

Why this is likely: a recent Glassdoor Meta research scientist report names lowest common ancestor in a no-AI onsite round, and a recent ML research scientist report names a diameter-of-binary-tree variant (medium confidence).

</details>

### AI-enabled coding (AI assistant allowed, as in Meta's AI-enabled round)

<a id="q3"></a>

#### Q3. Fix the action-chunk replay buffer that mixes episodes, then pad chunks at episode ends

Round: AI-enabled coding. AI policy: Provided (CoderPad assistant). Type: Debug. Difficulty: Medium. Topic: Imitation learning data pipelines.

The repo trains an action-chunking policy (predict the next H actions from one observation). Two tests fail.

```text
robo_data/
  episodes.py     # load_episodes() -> list of {"obs": (T, D), "act": (T, A)}
  buffer.py       # ChunkBuffer(episodes, horizon)
  loader.py       # batches samples, normalizes actions with stats.json
  stats.py        # compute_stats(episodes)
  test_buffer.py  # 7 tests, 2 failing

# buffer.py (current)
class ChunkBuffer:
    def __init__(self, episodes, horizon):
        self.obs = np.concatenate([e["obs"] for e in episodes])
        self.act = np.concatenate([e["act"] for e in episodes])
        self.H = horizon
        self.valid = np.arange(len(self.obs) - horizon)

    def sample(self, idx):
        t = self.valid[idx]
        return self.obs[t], self.act[t:t + self.H]

FAILED test_chunk_stays_in_episode: chunk at t=3 contains actions from episode 1
FAILED test_last_start_included: start 5 missing from valid
```

Phase 2: keep every start, pad short chunks at episode ends, and return a mask.

<details>
<summary>Show answer</summary>

Task: stop chunks from crossing episode boundaries, fix the dropped last start, then pad and mask instead of dropping.

What they grade: reading the failure before prompting, checking the assistant's diff, boundary tests, a reasoned choice between drop and pad.

Approach: `valid` comes from the concatenated array, so a start near the end of episode 0 reads actions from episode 1. That teaches the policy to teleport between scenes. Separately, `arange(len - H)` stops one short: the last full chunk starts at `len - H`. The fix tracks each episode's end.

With the assistant, I prompt: "Explain buffer.py and list every caller of ChunkBuffer.valid. Then propose a minimal fix that computes per-episode valid starts. Do not touch loader.py." I check the caller list against the file tree. Its first diff used `np.cumsum(lengths)` as start offsets, which is off by one episode (cumsum gives ends). The `test_chunk_stays_in_episode` test caught it, and I asked for `ends - lengths`.

For phase 2 there are two options. Dropping starts with t + H past the episode end is simple, but with 200-step episodes and H = 16 it drops 15 of 200 starts (7.5%), all at the end where the release and retreat happen, so the policy rarely learns to let go. Padding with the last action plus a mask keeps those steps and excludes padded targets from the loss. I choose padding.

Code:

```python
import numpy as np

class ChunkBuffer:
    def __init__(self, episodes, horizon, pad=True):
        self.obs = np.concatenate([e["obs"] for e in episodes])
        self.act = np.concatenate([e["act"] for e in episodes])
        self.H = horizon
        lengths = np.array([len(e["act"]) for e in episodes])
        ends = np.cumsum(lengths)
        starts = ends - lengths
        self.ep_end = np.repeat(ends, lengths)
        if pad:
            self.valid = np.arange(len(self.act))
        else:
            self.valid = np.concatenate(
                [np.arange(s, e - horizon + 1) for s, e in zip(starts, ends)]).astype(int)

    def __len__(self):
        return len(self.valid)

    def sample(self, idx):
        t = self.valid[idx]
        end = min(t + self.H, self.ep_end[t])
        chunk = self.act[t:end]
        n_pad = self.H - len(chunk)
        mask = np.r_[np.zeros(len(chunk), bool), np.ones(n_pad, bool)]
        if n_pad:
            chunk = np.concatenate([chunk, np.repeat(chunk[-1:], n_pad, axis=0)])
        return self.obs[t], chunk, mask

ep0 = {"obs": np.zeros((5, 2)), "act": np.arange(5.0).reshape(5, 1)}
ep1 = {"obs": np.ones((3, 2)), "act": np.array([[100.0], [101.0], [102.0]])}
b = ChunkBuffer([ep0, ep1], horizon=3, pad=False)
assert list(b.valid) == [0, 1, 2, 5]
for i in range(len(b)):
    _, chunk, mask = b.sample(i)
    assert not mask.any() and (chunk.max() < 100 or chunk.min() >= 100)
p = ChunkBuffer([ep0, ep1], horizon=3)
_, chunk, mask = p.sample(4)
assert chunk.ravel().tolist() == [4.0, 4.0, 4.0] and mask.tolist() == [False, True, True]
_, chunk, mask = p.sample(7)
assert chunk.ravel().tolist() == [102.0] * 3 and mask.sum() == 2
```

Verify: chunks never mix episodes, the last full start is present, padded chunks repeat the final action with the right mask, and an episode shorter than H still yields padded samples.

Pitfalls: accepting the cumsum diff because the shapes looked right; computing action statistics on padded rows in stats.py; applying the loss to masked steps.

Follow-ups: weight the loss so each episode counts equally; sample uniformly over episodes instead of over steps.

Why this is likely: Meta's careers FAQ and recent research scientist reports describe a multi-file CoderPad round with failing tests to fix and features to add; the robot data theme follows the listing's learning-from-demonstration focus (medium confidence).

</details>

<a id="q4"></a>

#### Q4. Review this AI-generated GAE and PPO loss before it trains a locomotion policy

Round: AI-enabled coding. AI policy: Provided (CoderPad assistant). Type: Review. Difficulty: Medium. Topic: Policy gradient correctness.

The assistant wrote this for a PPO run with 2048-step rollouts and time-limit truncation at 1000 steps. Review it like a pull request.

```python
import torch

def compute_gae(rewards, values, dones, gamma=0.99, lam=0.95):
    # rewards, values, dones: tensors of shape (T,)
    T = len(rewards)
    adv = torch.zeros(T)
    last = 0
    for t in reversed(range(T)):
        next_value = values[t + 1] if t + 1 < T else 0
        delta = rewards[t] + gamma * next_value - values[t]
        last = delta + gamma * lam * last
        adv[t] = last
    returns = adv + values
    adv = torch.normalize(adv)
    return adv, returns

def ppo_loss(new_logp, old_logp, adv, clip=0.2):
    ratio = torch.exp(new_logp - old_logp)
    unclipped = ratio * adv
    clipped = torch.clamp(ratio, 1 - clip, 1 + clip) * adv
    return -torch.max(unclipped, clipped).mean()
```

<details>
<summary>Show answer</summary>

Task: find the defects, rank them by damage, and ship a tested fix.

What they grade: catching the planted bugs, knowing which are silent, tests that would have caught them.

Approach: I found four problems, in order of damage, and one line that is correct but fragile.

1. `dones` is never used. The delta bootstraps across episode boundaries and the recursion carries advantage from the next episode into this one. Training still runs, which makes this bug dangerous.
2. The last step bootstraps with 0. A rollout cut at step 2048 is not terminal, so it must use V(s_T) from the critic. Time-limit truncation at 1000 steps is the same case: bootstrap, do not zero. So a truncation must never zero the bootstrap. With this compute_gae, at a mid-rollout time-limit truncation add gamma * V(final_obs) to that step's reward and set dones = 1 there. The done then zeroes only the next episode's value, which the folded-in bootstrap already replaced, and it resets the GAE recursion, because the next step starts a new episode. The cleaner alternative is separate `terminated` and `truncated` arrays, bootstrapping from V(final_obs) on truncation and resetting the recursion on either.
3. `torch.max` turns PPO's pessimistic bound into an optimistic one. With a positive advantage and ratio 1.5 the loss uses 1.5 instead of 1.2, so the clip does nothing.
4. `torch.normalize` does not exist. The nearest real API, `torch.nn.functional.normalize`, does L2 normalization, not mean and standard deviation scaling, so "fixing the import" would also be wrong.
5. `returns` is computed before normalization, which is correct; I would add a comment so nobody moves it.

To verify, I asked the assistant for a brute-force GAE that sums discounted deltas inside each episode. Its first version also ignored dones. I fixed it by hand, then compared it to the recursion on a rollout with a terminal in the middle.

Code: NumPy version of the fix, with the tests.

```python
import numpy as np

def compute_gae(rewards, values, dones, last_value, gamma=0.99, lam=0.95):
    T = len(rewards)
    adv, gae = np.zeros(T), 0.0
    for t in reversed(range(T)):
        next_v = last_value if t == T - 1 else values[t + 1]
        nonterm = 1.0 - dones[t]
        delta = rewards[t] + gamma * next_v * nonterm - values[t]
        gae = delta + gamma * lam * nonterm * gae
        adv[t] = gae
    return adv, adv + values

def standardize(x, eps=1e-8):
    return (x - x.mean()) / (x.std() + eps)

def ppo_clip_loss(new_logp, old_logp, adv, clip=0.2):
    ratio = np.exp(new_logp - old_logp)
    return -np.minimum(ratio * adv, np.clip(ratio, 1 - clip, 1 + clip) * adv).mean()

def gae_reference(r, v, d, last_value, gamma, lam):
    vext, out = list(v) + [last_value], []
    for t in range(len(r)):
        a = 0.0
        for k, j in enumerate(range(t, len(r))):
            a += (gamma * lam) ** k * (r[j] + gamma * vext[j + 1] * (1 - d[j]) - v[j])
            if d[j]:
                break
        out.append(a)
    return np.array(out)

r = np.array([1.0, 0.0, 2.0, 1.0]); v = np.array([0.5, 0.4, 0.3, 0.2]); d = np.array([0, 1, 0, 0.0])
adv, ret = compute_gae(r, v, d, last_value=0.7, gamma=0.9, lam=0.8)
assert np.allclose(adv, gae_reference(r, v, d, 0.7, 0.9, 0.8))
assert np.isclose(adv[1], -0.4)
assert np.isclose(ppo_clip_loss(np.log([1.5]), np.log([1.0]), np.array([2.0])), -2.4)
assert np.isclose(ppo_clip_loss(np.log([1.5]), np.log([1.0]), np.array([-2.0])), 3.0)
assert abs(standardize(adv).mean()) < 1e-9
# time-limit truncation at step 1 with V(final_obs) = 0.6: fold the bootstrap into the reward, reset there
r_tr = r.copy(); r_tr[1] += 0.9 * 0.6
adv_tr, _ = compute_gae(r_tr, v, np.array([0, 1, 0, 0.0]), last_value=0.7, gamma=0.9, lam=0.8)
assert np.isclose(adv_tr[1], 0.14) and np.isclose(adv_tr[0], 0.86 + 0.72 * 0.14)
```

Verify: terminal mid-rollout, truncation mid-rollout, truncated final step, clipping for both signs of advantage, normalized advantage with mean zero.

Pitfalls: zeroing the bootstrap at a truncation (treating it as a true termination); normalizing per minibatch of 8 samples (noisy); computing GAE from value tensors that still require grad (compute advantages and returns under torch.no_grad() or detach the values), so policy-loss gradients flow through the advantage estimate.

Follow-ups: what the value-loss clip does; why the approximate KL is worth logging.

Why this is likely: Meta's AI-enabled coding round includes reviewing and fixing code with the provided assistant, and the listing names reinforcement learning among its required research areas. The PPO and GAE topic comes from the ML coding source row (the job listing and a Glassdoor research scientist intern report of ML and PyTorch coding), framed here as an AI-code review because the AI-enabled round includes reviewing and fixing code (medium confidence).

</details>

<a id="q5"></a>

#### Q5. Extend the grid planner so the robot can pick up keycards and open doors, and return the path

Round: AI-enabled coding. AI policy: Provided (CoderPad assistant). Type: Build. Difficulty: Hard. Topic: State-space search.

`planner.py` has a 4-connected BFS `plan(grid, start, goal)` that returns a move count. The grid uses `.` free, `#` wall, `a` to `f` keycards and `A` to `F` doors that need the matching card.

```text
nav/
  grid.py          # parse(str) -> list[str], neighbours(r, c)
  planner.py       # plan(grid, start, goal) -> int  (plain BFS)
  test_planner.py  # 4 pass; test_keys_and_doors, test_backtrack_for_key fail
  bench.py         # 1000 x 1000 grid with 6 keys, must finish in 10 s

test_backtrack_for_key:
  grid  = ["a..A."]
  start = (0, 1), goal = (0, 4)
  expected path length 6 cells: (0,1) (0,0) (0,1) (0,2) (0,3) (0,4)
```

Phase 1: support keys and doors. Phase 2: return the path. Phase 3: pass bench.py.

<details>
<summary>Show answer</summary>

Task: search over (row, col, keys held), return the cell path, then fit the large benchmark.

What they grade: the right state definition, path reconstruction, checking the assistant's output against the backtracking test, memory math for phase 3.

Approach: The robot can stand on the same cell holding different keys, so the state is (r, c, mask) with a 6-bit mask, at most 64 key sets per cell. BFS on that state space is still optimal because every move costs 1.

Prompt to the assistant: "Change plan() to BFS over (r, c, keymask), store parents, return the list of cells. Keep grid.py unchanged." Its first diff kept `visited` keyed on (r, c). That fails `test_backtrack_for_key`, because the robot must revisit (0, 1) after picking up key a. The test caught it, and I changed the key to the full state.

Phase 3 numbers: 10^6 cells times 64 masks is 64 million states. A Python dict with tuple keys costs roughly 100 bytes per entry, so about 6 GB, which is too much. Two alternatives: a flat NumPy `uint8` visited array of 64 MB indexed by (r * C + c) * 64 + mask, with parents stored as one int32 predecessor array (256 MB); or A* with Manhattan distance, which helps on open maps but not in mazes. I would use the flat arrays and only keep A* if bench.py has open rooms. Memory is only half of it: a Python loop over up to 64 million states will not finish in 10 s. Only reachable (cell, mask) states matter, and with 6 keys most masks are unreachable in practice, so I would measure the reachable count first. If it is still large, I would vectorise the BFS one frontier at a time in NumPy over the flat arrays, or compile the loop with Numba. Then I would time it on bench.py before claiming phase 3.

Code:

```python
from collections import deque

def plan(grid, start, goal):
    R, C = len(grid), len(grid[0])
    s0 = (start[0], start[1], 0)
    parent = {s0: None}
    q = deque([s0])
    while q:
        r, c, keys = q.popleft()
        if (r, c) == goal:
            path, s = [], (r, c, keys)
            while s is not None:
                path.append(s[:2])
                s = parent[s]
            return path[::-1]
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if not (0 <= nr < R and 0 <= nc < C) or grid[nr][nc] == "#":
                continue
            cell = grid[nr][nc]
            if cell.isupper() and not (keys >> (ord(cell) - ord("A"))) & 1:
                continue
            nk = keys | (1 << (ord(cell) - ord("a"))) if cell.islower() else keys
            if (nr, nc, nk) not in parent:
                parent[(nr, nc, nk)] = (r, c, keys)
                q.append((nr, nc, nk))
    return None

assert plan(["a..A."], (0, 1), (0, 4)) == [(0, 1), (0, 0), (0, 1), (0, 2), (0, 3), (0, 4)]
g = ["..a",
     "##.",
     ".A."]
assert len(plan(g, (0, 0), (2, 0))) == 7
assert plan(["..#", "##.", ".A."], (0, 0), (2, 0)) is None
assert plan(["..."], (0, 0), (0, 0)) == [(0, 0)]
```

Verify: the backtracking case, a door with no reachable key (None), start equals goal, and the benchmark under 10 s.

Pitfalls: visited keyed on cells only; checking the door bit with the wrong letter offset; reconstructing a path that skips the key pickup.

Follow-ups: keys that are consumed when used (the mask changes, the state space does not); weighted terrain (Dijkstra on the same states).

Why this is likely: 1point3acres trend reports name a maze solver task in Meta's AI coding round, and Hello Interview describes the same phased format (medium confidence).

</details>

### ML coding and fundamentals (no AI, as at Meta)

<a id="q6"></a>

#### Q6. This bf16 VLA run diverged at step 3461. Read the log, name the cause, add a guard

Round: ML coding and fundamentals. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Training stability.

```text
step  lr      loss   grad_norm  act_l1  max_attn_logit
1000  3.0e-4  1.92   0.84       0.071   38
2000  3.0e-4  1.71   0.91       0.064   61
3000  3.0e-4  1.66   1.10       0.062   140
3400  3.0e-4  1.64   7.90       0.061   410
3460  3.0e-4  5.83   112.4      0.240   2900
3461  3.0e-4  nan    nan        nan     inf
config: 1.3B VLA, bf16 weights and AdamW states, no grad clipping,
        attention softmax computed in bf16, no QK-norm, z-loss off,
        loss = CE(action tokens) + 0.1 * L1(continuous head)
```

<details>
<summary>Show answer</summary>

Task: explain the divergence from the log, fix the root cause, add a guard so the next run survives a spike.

What they grade: reading the leading indicator, separating cause from symptom, fixes ranked by cost.

Approach: Loss is flat and healthy until step 3400, so data and LR are not the first suspects. The leading signal is `max_attn_logit`: 38, 61, 140, 410, then 2900. Attention logits are growing without bound, a known instability in large transformers. Grad norm only jumps after the logits explode. Once a logit is that large, a bf16 softmax saturates, and with only 8 bits of precision (7 stored mantissa bits) the scores lose precision. Gradients spike, and without clipping one update pushes the weights into inf.

Fixes by cost. First, compute softmax and the losses in fp32 and keep fp32 master weights and optimizer states; bf16 Adam second moments lose small updates. Second, add QK-norm (or tanh soft-capping of logits) so logit growth is bounded by design. Third, clip gradients at 1.0 and add a small z-loss (1e-4) on output logits. Lowering the LR also delays the blow-up but costs speed and does not remove the cause. QK-norm plus fp32 softmax fixes it at a small compute cost.

The guard tracks an EMA of grad norm and skips any update more than 6 standard deviations out. On a non-finite value it rewinds to the last checkpoint and lowers the LR.

Code:

```python
import math
import numpy as np

def stable_softmax(z):
    z = np.asarray(z, dtype=np.float32)
    z = z - z.max(axis=-1, keepdims=True)
    e = np.exp(z)
    return e / e.sum(axis=-1, keepdims=True)

class SpikeGuard:
    def __init__(self, beta=0.98, z_max=6.0, warmup=20):
        self.beta, self.z_max, self.warmup = beta, z_max, warmup
        self.mean, self.var, self.n = 0.0, 0.0, 0

    def check(self, grad_norm):
        if not math.isfinite(grad_norm):
            return "rewind"
        if self.n == 0:
            self.mean = grad_norm
        elif self.n >= self.warmup:
            z = (grad_norm - self.mean) / math.sqrt(self.var + 1e-12)
            if z > self.z_max:
                return "skip"
        d = grad_norm - self.mean
        self.mean += (1 - self.beta) * d
        self.var = self.beta * (self.var + (1 - self.beta) * d * d)
        self.n += 1
        return "ok"

assert np.allclose(stable_softmax([1000.0, 1001.0]), [0.268941, 0.731059], atol=1e-5)
g = SpikeGuard()
rng = np.random.default_rng(0)
assert all(g.check(float(x)) == "ok" for x in rng.normal(0.9, 0.05, 200))
assert g.check(7.9) == "skip"
assert g.check(float("nan")) == "rewind"
assert g.check(0.92) == "ok"
```

Verify: the guard passes 200 normal steps, skips the 7.9 spike, rewinds on NaN, and keeps skipped steps out of the running statistics.

Pitfalls: blaming the LR because the loss moved last; turning on loss scaling (bf16 has fp32's range, so it does not need it); a guard that silently skips forever.

Follow-ups: what to log per layer to find which block's logits grow; whether the 0.1 L1 weight changes stability.

Why this is likely: 1point3acres reports that this lab's interviews lean on hand-coded models, and a recent Blind thread about the lab's loop mentions numerical stability (medium to low confidence).

</details>

<a id="q7"></a>

#### Q7. Implement causal multi-head attention for an action-token policy in NumPy

Round: ML coding and fundamentals. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Transformers from scratch.

Spec: `mha(x, Wq, Wk, Wv, Wo, n_heads, causal=True, key_pad=None)` with x of shape (B, T, D), square weight matrices, and `key_pad` of shape (B, T) where True marks padding. The policy predicts discretized action tokens one step at a time.

<details>
<summary>Show answer</summary>

Task: batched, masked multi-head attention with no loops over heads, plus tests that prove the masks work.

What they grade: correct shapes, scaling by sqrt(head_dim), numerically stable softmax, masks applied before softmax, and a test of causality.

Approach: Project once, reshape to (B, h, T, hd), compute scores with one batched matmul, mask, softmax, then merge heads and apply Wo. Masked scores use a large negative number. Using -inf is also common, but a fully padded query row then becomes NaN (0 divided by 0). With -1e9 that row becomes uniform, and padded positions are excluded from the loss anyway.

Alternatives: in PyTorch I would call `F.scaled_dot_product_attention(q, k, v, is_causal=True)` when there is no padding, or build one boolean `attn_mask` that combines the causal and padding masks and leave `is_causal=False` (setting both raises an error). Without a custom mask it can use a fused kernel that never stores the T by T matrix. The manual version is what this round asks for and is easier to debug. Memory decides the choice in production: with B = 64, h = 16, T = 512, the fp32 score tensor alone is 64 x 16 x 512 x 512 x 4 bytes, about 1.07 GB, which a fused kernel avoids.

Action-chunking policies such as ACT decode a whole chunk in parallel without a causal mask; the causal version fits autoregressive action tokens.

Code:

```python
import numpy as np

def mha(x, Wq, Wk, Wv, Wo, n_heads, causal=True, key_pad=None):
    B, T, D = x.shape
    hd = D // n_heads
    def split(W):
        return (x @ W).reshape(B, T, n_heads, hd).transpose(0, 2, 1, 3)
    q, k, v = split(Wq), split(Wk), split(Wv)
    scores = q @ k.transpose(0, 1, 3, 2) / np.sqrt(hd)
    mask = np.zeros((B, 1, T, T), bool)
    if causal:
        mask |= np.triu(np.ones((T, T), bool), k=1)
    if key_pad is not None:
        mask |= key_pad[:, None, None, :]
    scores = np.where(mask, -1e9, scores)
    scores = scores - scores.max(-1, keepdims=True)
    w = np.exp(scores)
    w /= w.sum(-1, keepdims=True)
    return (w @ v).transpose(0, 2, 1, 3).reshape(B, T, D) @ Wo

def naive(x, Wq, Wk, Wv, Wo, h):
    B, T, D = x.shape
    hd, out = D // h, np.zeros((B, T, D))
    for b in range(B):
        q, k, v = x[b] @ Wq, x[b] @ Wk, x[b] @ Wv
        heads = []
        for i in range(h):
            s = slice(i * hd, (i + 1) * hd)
            o = np.zeros((T, hd))
            for t in range(T):
                sc = q[t, s] @ k[: t + 1, s].T / np.sqrt(hd)
                w = np.exp(sc - sc.max()); w /= w.sum()
                o[t] = w @ v[: t + 1, s]
            heads.append(o)
        out[b] = np.concatenate(heads, axis=1) @ Wo
    return out

rng = np.random.default_rng(0)
B, T, D, h = 2, 5, 8, 2
x = rng.standard_normal((B, T, D))
W = [rng.standard_normal((D, D)) * 0.3 for _ in range(4)]
y = mha(x, *W, n_heads=h)
assert np.allclose(y, naive(x, *W, h))
x2 = x.copy(); x2[:, -1] += 10.0
assert np.allclose(mha(x2, *W, n_heads=h)[:, :-1], y[:, :-1])
pad = np.zeros((B, T), bool); pad[0, 3:] = True
x3 = x.copy(); x3[0, 3:] = 99.0
yp = mha(x3, *W, n_heads=h, causal=False, key_pad=pad)
yq = mha(x, *W, n_heads=h, causal=False, key_pad=pad)
assert np.allclose(yp[0, :3], yq[0, :3])
```

Verify: matches a loop-based reference, changing the last token leaves earlier outputs unchanged, padded keys have no effect on unpadded queries.

Pitfalls: scaling by sqrt(D) instead of sqrt(head_dim); masking after softmax; reshaping (B, T, D) straight to (B, h, T, hd) without the transpose, which scrambles heads.

Follow-ups: add a KV cache for decoding; add rotary position embeddings.

Why this is likely: 1point3acres says this lab's interviews lean toward hand-coded models, and a recent Blind thread on the lab's loop mentions implementing model pieces from scratch (medium confidence).

</details>

<a id="q8"></a>

#### Q8. Implement the training step of a diffusion policy and check the noise schedule

Round: ML coding and fundamentals. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Diffusion policies for imitation learning.

You train a policy that denoises an action chunk conditioned on observations, with 100 diffusion steps, given `denoiser(a_t, t, obs)`. The existing code has this schedule:

```python
import numpy as np

# copied from the DDPM paper
T = 100
betas = np.linspace(1e-4, 0.02, T)
alpha_bar = np.cumprod(1 - betas)
```

Write q_sample and the epsilon-prediction loss, then say whether this schedule is right for T = 100.

<details>
<summary>Show answer</summary>

Task: DDPM forward process and loss, plus a sanity check that the last step is close to pure noise.

What they grade: the closed-form q(a_t | a_0), sampling t per example, a loss you can test with an oracle, and spotting the schedule bug.

Approach: With alpha_bar_t the cumulative product of (1 - beta_t), the noised action is sqrt(alpha_bar_t) a_0 + sqrt(1 - alpha_bar_t) noise. Sample one t per example, noise the chunk, predict the noise and take the MSE. Two tests make it checkable: an oracle denoiser that knows a_0 should give near-zero loss, and a zero denoiser should give a loss near 1 because the noise has unit variance.

The schedule has a bug. The DDPM linear schedule (1e-4 to 0.02) was designed for 1000 steps. Keep those endpoints with 100 steps and alpha_bar at the last step is about 0.36, so the model never trains on pure noise but sampling starts from it. The policy then produces jittery or out-of-range actions. Alternatives: rescale the linear betas to 100 steps, or use the cosine schedule, which drives alpha_bar near zero at the end for any step count. I use the cosine schedule because it also spends more steps at low noise, where fine action detail lives.

For the prediction target, epsilon prediction is the standard; a_0 or velocity prediction can be more stable with very few steps. I would keep epsilon and check both on held-out action MSE.

Code:

```python
import numpy as np

def linear_alpha_bar(T=100, b0=1e-4, b1=2e-2):
    return np.cumprod(1.0 - np.linspace(b0, b1, T))

def cosine_alpha_bar(T=100, s=0.008):
    f = np.cos((np.arange(T + 1) / T + s) / (1 + s) * np.pi / 2) ** 2
    betas = np.clip(1 - f[1:] / f[:-1], 0, 0.999)
    return np.cumprod(1 - betas)

def q_sample(a0, t, noise, alpha_bar):
    ab = alpha_bar[t].reshape(-1, *([1] * (a0.ndim - 1)))
    return np.sqrt(ab) * a0 + np.sqrt(1 - ab) * noise

def diffusion_loss(denoiser, obs, a0, alpha_bar, rng):
    t = rng.integers(0, len(alpha_bar), size=a0.shape[0])
    noise = rng.standard_normal(a0.shape)
    eps_hat = denoiser(q_sample(a0, t, noise, alpha_bar), t, obs)
    return np.mean((eps_hat - noise) ** 2)

rng = np.random.default_rng(0)
a0 = rng.uniform(-1, 1, size=(256, 16, 7))
lin, cos = linear_alpha_bar(), cosine_alpha_bar()
assert 0.3 < lin[-1] < 0.4
assert cos[-1] < 2e-3

def oracle(a_t, t, obs):
    ab = cos[t].reshape(-1, 1, 1)
    return (a_t - np.sqrt(ab) * obs) / np.sqrt(1 - ab)

assert diffusion_loss(oracle, a0, a0, cos, rng) < 1e-8
zero_loss = diffusion_loss(lambda a, t, o: np.zeros_like(a), None, a0, cos, rng)
assert 0.95 < zero_loss < 1.05
```

Verify: oracle loss near 0, zero-predictor loss near 1, alpha_bar at the last step below 0.002 for cosine and 0.36 for the misused linear schedule.

Pitfalls: one shared t for the whole batch; actions not normalized to [-1, 1] first; evaluating with a different number of steps than training without a DDIM-style sampler.

Follow-ups: sample with 10 DDIM steps for 10 Hz control; compare with flow matching.

Why this is likely: the listing names learning from demonstration and imitation learning, and 1point3acres reports hand-coded model rounds for this lab (medium confidence).

</details>

### ML and research system design (AI assistant allowed, as in Meta's design rounds)

<a id="q9"></a>

#### Q9. Design the data flywheel and training stack for a generalist manipulation policy

Round: ML or research system design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Design. Difficulty: Hard. Topic: Robot learning at scale.

Goal: one policy that follows language instructions for tabletop and kitchen manipulation on two arm types. Inputs: 20 teleoperation stations, a large pool of egocentric human video, and a GPU budget of 64 GPUs for training. Design data collection, the model, training and evaluation, with numbers.

<details>
<summary>Show answer</summary>

Task: an end-to-end plan that turns robot hours and human video into a policy, and tells you weekly whether it got better.

What they grade: numbers that add up, a model choice with alternatives, evaluation you can trust, a loop that targets failures.

Approach: Draw it in Mermaid as four boxes: collect, curate, train, evaluate, with an arrow from evaluation back to collection. The assistant drafts the diagram; I write the numbers myself and check any it adds. In my practice run it claimed the egocentric video "provides action labels", which is wrong. Human video has no robot action labels, so it mainly pretrains representations or a world model; hand-pose trajectories extracted from it can also act as approximate actions after retargeting (as in EgoMimic), but they are not robot commands.

Numbers: 20 stations at 5 productive hours a day is 100 robot hours a day, so 1000 hours in 10 working days. Three cameras of 224 by 224 JPEG at about 15 KB per frame and 15 Hz is roughly 2.4 GB per hour, so 1000 hours is about 2.4 TB. That gives 54 million timesteps. A 1B-parameter model seeing about 816 tokens per sample (three images at 256 patches, 32 text tokens, a 16-step action chunk) costs about 6 x 1e9 x 816, or 4.9e12 FLOPs per sample, so 2.6e20 FLOPs per epoch. At 40% utilization of an H100's roughly 989 dense bf16 TFLOPS that is about 186 GPU-hours, or about 3 hours per epoch on 64 GPUs.

Alternatives: (a) a VLM with discrete action tokens, simple but coarse for dexterous control; (b) a pretrained VLM backbone with a diffusion or flow action head, smooth continuous actions; (c) world-model pretraining on human video (V-JEPA 2 style) and then action-conditioned training. I pick (b) for the policy and use (c) as a representation source, because (c) alone plans slowly.

Evaluation: a fixed suite of 50 tasks with seeded layouts on held-out stations, run nightly, plus a sim proxy for regressions. Failures route back to collection as targeted teleop requests.

Risks: operator style drift between stations; embodiment mismatch between arms (add an embodiment token); eval suites that leak into training.

Why this is likely: research scientist reports describe an ML system design round in CoderPad with the assistant available, and the listing asks for data-driven paradigms across images, video, text, audio and touch (medium confidence for the format, low for the robotics framing).

</details>

<a id="q10"></a>

#### Q10. Policy A won 14 of 20 real trials, policy B won 10 of 20. Design the eval that settles it

Round: ML or research system design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Concept plus code. Difficulty: Medium. Topic: Real-robot evaluation statistics.

Your teammate wants to write "A beats B by 20 points" in a paper. The trials ran on different days with different object layouts. Decide what you can claim, and design the evaluation that would settle it.

<details>
<summary>Show answer</summary>

Task: quantify the uncertainty in the current numbers, then design an evaluation with enough power and fewer confounds.

What they grade: correct small-sample statistics, controlling confounds, a sample size computed rather than guessed, honest wording.

Approach: 70% versus 50% on 20 trials each is not significant. Wilson 95% intervals are 0.48 to 0.85 for A and 0.30 to 0.70 for B, and they overlap heavily. Fisher's exact test gives a two-sided p of about 0.33. I asked the assistant for a p-value and it ran a chi-square test without continuity correction (p about 0.20). At n = 20 per arm the uncorrected chi-square is slightly anti-conservative, so I computed Fisher's exact test myself, below; neither test is close to significance.

Confounds matter more than counts here. Different days mean lighting, wear and operator changes. The redesign pairs trials. Fix a set of seeded initial layouts, run A and B on each layout back to back in a random order, and keep the operator blind to which policy runs. Then analyse discordant pairs with McNemar's test, which is more powerful than comparing two independent proportions.

Numbers: to detect 70% versus 50% with 80% power at alpha 0.05 with independent samples you need about 93 trials per policy. At about 2 minutes per trial with resets, that is about 6 hours of robot time. A gap of 80% versus 50% needs about 39 per policy. Pairing reduces these further when outcomes are correlated across policies.

Code:

```python
import math

def wilson(k, n, z=1.96):
    p = k / n
    d = 1 + z * z / n
    c = (p + z * z / (2 * n)) / d
    h = z * math.sqrt(p * (1 - p) / n + z * z / (4 * n * n)) / d
    return c - h, c + h

def fisher_two_sided(a, n1, b, n2):
    K, N = a + b, n1 + n2
    pmf = lambda x: math.comb(n1, x) * math.comb(n2, K - x) / math.comb(N, K)
    p0 = pmf(a)
    return sum(pmf(x) for x in range(max(0, K - n2), min(K, n1) + 1) if pmf(x) <= p0 * (1 + 1e-9))

def n_per_arm(p1, p2, za=1.96, zb=0.8416):
    pb = (p1 + p2) / 2
    num = za * math.sqrt(2 * pb * (1 - pb)) + zb * math.sqrt(p1 * (1 - p1) + p2 * (1 - p2))
    return math.ceil(num ** 2 / (p1 - p2) ** 2)

lo, hi = wilson(14, 20)
assert round(lo, 2) == 0.48 and round(hi, 2) == 0.85
assert abs(fisher_two_sided(14, 20, 10, 20) - 0.333) < 0.001
assert n_per_arm(0.7, 0.5) == 93 and n_per_arm(0.8, 0.5) == 39
```

Verify: Wilson bounds against a textbook table, Fisher's p symmetric when you swap A and B, sample sizes against a standard power calculator.

Pitfalls: reporting the point estimate gap without intervals; dropping "failed due to hardware" trials for one policy only; tuning on the evaluation layouts.

Follow-ups: partial credit for multi-stage tasks; how many layouts versus repeats per layout.

Why this is likely: the listing asks you to "define, build and benchmark new capabilities", and Meta's design rounds run in CoderPad with the assistant available (medium confidence).

</details>

<a id="q11"></a>

#### Q11. A world-model planner reaches goals but fails pick-and-place. Read the planner log and fix it

Round: ML or research system design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Trace debug. Difficulty: Hard. Topic: Planning with latent world models.

The team plans with an action-conditioned video world model in the style of V-JEPA 2-AC: sample action sequences, roll them out in latent space, score them by distance to a goal image's embedding, execute the first action, replan.

```text
planner: CEM over 7-DoF end-effector deltas, horizon 3, 300 samples, 10 iters, top 30
energy:  L1(predicted embedding after horizon, goal embedding)
task              trials  success
reach target        20      18
grasp cup           20      11
pick cup to bowl    20       3      (goal = one image of cup in bowl)

episode 07 (pick cup to bowl), energy per replan step:
0.41 0.40 0.40 0.39 0.40 0.39 0.40 0.40 ...  timeout, gripper hovering above cup

energy to final goal along a successful teleop demo of the same task:
0.52 0.55 0.58 0.49 0.31 0.18 0.05
```

<details>
<summary>Show answer</summary>

Task: explain why the planner stalls on the long task, propose fixes in order, and give a test that predicts the stall.

What they grade: reading the energy trace, separating model error from planner failure, fixes with costs.

Approach: Reaching works, so the model and the energy are fine locally. The teleop demo shows the cause: energy to the final goal rises from 0.52 to 0.58 before it falls. Grasping requires moving into states that look less like "cup in bowl" (gripper closed around the cup, cup lifted). A greedy planner with a 3-step horizon never accepts that rise, so it settles at a local minimum and hovers. Episode 07's flat trace near 0.40 is that stall.

Fixes in order of cost. One: visual subgoals. Give the planner an intermediate goal image (cup grasped), then the final one. Meta's V-JEPA 2 post reports 65% to 80% pick-and-place success on new objects with visual subgoals, after action-conditioned training on only 62 hours of robot data. Two: a longer horizon. Rollout cost grows linearly with horizon and model error compounds, so going from 3 to 10 steps triples compute per replan and may still not cross the hump. Three: learn a goal-conditioned value function to replace raw distance, which needs extra training data. I start with subgoals and measure which tasks still fail.

With the assistant I would ask for a script that flags demos whose energy is not monotone and proposes subgoal frames. I check its choices against the demo video.

Code:

```python
import numpy as np

def subgoal_indices(profile, tol=0.0):
    idx = [i for i in range(1, len(profile) - 1)
           if profile[i] >= profile[i - 1] + tol and profile[i] > profile[i + 1]]
    return idx + [len(profile) - 1]

def energy_profile(embs, goal):
    return [float(np.abs(e - goal).mean()) for e in embs]

demo = [0.52, 0.55, 0.58, 0.49, 0.31, 0.18, 0.05]
assert subgoal_indices(demo) == [2, 6]
assert subgoal_indices([0.5, 0.4, 0.2, 0.05]) == [3]
rng = np.random.default_rng(0)
goal = rng.standard_normal(64)
embs = [goal + rng.standard_normal(64) * s for s in (1.0, 1.2, 0.6, 0.1)]
prof = energy_profile(embs, goal)
assert subgoal_indices(prof) == [1, 3]
```

Verify: the flagger finds the hump in the demo and returns only the final goal for monotone demos; then success on pick-and-place with subgoals, on the same 20 seeded layouts.

Pitfalls: blaming the world model without checking a monotone task first; adding horizon without measuring rollout error growth; using subgoals from the evaluation episodes.

Follow-ups: generating subgoal images automatically; how gripper actions should be sampled in CEM.

Why this is likely: V-JEPA 2 is the team's published world-model work, and its blog describes model-predictive planning to goal images and the subgoal approach; the design round format comes from research scientist reports (medium confidence).

</details>

<a id="q12"></a>

#### Q12. Build an LLM planner eval for multi-agent home tasks and sort its failures

Round: ML or research system design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Concept plus code. Difficulty: Hard. Topic: Embodied agent evaluation.

An LLM plans for a robot that shares a simulated home with a human partner, on PARTNR-style tasks such as "put the plates in the sink, then wipe the table". Success is 30% versus 93% for human pairs. Design the evaluation that tells the team why it fails, and write the failure classifier over episode logs.

<details>
<summary>Show answer</summary>

Task: an evaluation that splits failures into causes you can fix, with code that labels episodes from logs.

What they grade: grounded failure categories, automatic labels with a human check, metrics beyond success rate.

Approach: Meta's PARTNR benchmark has 100,000 natural language tasks across 60 houses and over 5,800 objects. Its authors report that people solve 93% of tasks and LLM planners 30%, with failures in coordination, task tracking and recovery from errors. I make those three the top-level labels, plus perception and skill failures so planner and skill errors stay separate.

Each episode log records actions, who did them, skill results and the task's evaluation predicates. Rules: coordination means both agents attempt the same subtask or one waits for more than N steps while work remains; tracking means the planner declares done with predicates unmet; recovery means the same failed action repeats three or more times; skill failure means a pick or place returned an error the planner then handled. Labels are not exclusive.

Metrics beyond success: percent of predicates satisfied, steps relative to the human pair, and redundant actions per episode. Alternatives for labelling: an LLM judge reading logs is flexible but drifts and costs money; rules are cheap and auditable but miss new failure types. I use rules first, then have the assistant propose labels for 50 unlabelled failures and compare them with my own reading. In a practice run it labelled a skill failure as "recovery"; the log showed the planner retried correctly and the grasp itself failed.

Code:

```python
def classify(ep, wait_limit=5):
    labels, acts = set(), ep["actions"]
    for i in range(1, len(acts)):
        a, b = acts[i - 1], acts[i]
        if a["agent"] != b["agent"] and a["target"] == b["target"] and a["verb"] == b["verb"]:
            labels.add("coordination")
    if ep.get("robot_idle_steps", 0) > wait_limit and ep["remaining_subtasks"] > 0:
        labels.add("coordination")
    if ep["declared_done"] and not all(ep["predicates"].values()):
        labels.add("task_tracking")
    run = 1
    for i in range(1, len(acts)):
        same = acts[i] == acts[i - 1] and not acts[i]["ok"]
        run = run + 1 if same else 1
        if run >= 3:
            labels.add("error_recovery")
    if any(not x["ok"] and x["verb"] in ("pick", "place") for x in acts):
        labels.add("skill_failure")
    return labels

A = lambda agent, verb, target, ok=True: {"agent": agent, "verb": verb, "target": target, "ok": ok}
ep = {"actions": [A("robot", "pick", "plate1"), A("human", "pick", "plate1"),
                  A("robot", "place", "sink", ok=False), A("robot", "place", "sink", ok=False),
                  A("robot", "place", "sink", ok=False)],
      "declared_done": True, "predicates": {"plate1_in_sink": False}, "remaining_subtasks": 1}
assert classify(ep) == {"coordination", "task_tracking", "error_recovery", "skill_failure"}
ok_ep = {"actions": [A("robot", "pick", "plate1"), A("robot", "place", "sink")],
         "declared_done": True, "predicates": {"plate1_in_sink": True}, "remaining_subtasks": 0}
assert classify(ok_ep) == set()
```

Verify: hand-label 100 failed episodes and require at least 85% agreement per label before trusting the dashboard.

Pitfalls: one label per episode; reporting success only; letting the planner see the predicates it is graded on.

Follow-ups: evaluating with real human partners in the loop; how to weight partial completion.

Why this is likely: PARTNR is Meta FAIR's human-robot collaboration benchmark, and the listing names human-robot interaction, LLMs and digital agents (medium confidence for the topic, medium for the design round format).

</details>

### Research deep dive (no AI, as at Meta)

<a id="q13"></a>

#### Q13. Your in-hand rotation policy drops 18% of objects. Add touch to fix slip

Round: Research deep dive. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Tactile sensing for dexterous manipulation.

A vision-only policy rotates objects in a multi-fingered hand. 18% of episodes end with a drop, mostly on glossy or light objects, and the drops start when a finger loses contact. You can add fingertip tactile sensors and a pretrained touch encoder. Propose the design and the experiment.

<details>
<summary>Show answer</summary>

Task: decide where touch enters the system, how fast it must act, and how to prove it helped.

What they grade: a reason touch fixes this failure, a latency budget, alternatives, an ablation that isolates touch.

Approach: Vision sees slip late. The object must move a few millimetres before the camera notices, and fingers occlude it. Touch sees shear and contact area change before gross motion. Meta's Digit 360 fingertip senses forces as small as 1 millinewton, and Sparsh is a family of self-supervised touch encoders pretrained on over 460,000 tactile images, so I can start from a frozen encoder instead of labelling slip myself.

Design: two loops. The policy runs at about 10 Hz on vision, proprioception and Sparsh embeddings. A reflex loop runs at the tactile frame rate with a small slip head on the frozen encoder; on predicted slip it increases grip force on that finger for 100 to 200 ms. The reflex reacts many times faster than the 10 Hz policy could.

Alternatives: (a) raw tactile images into the policy end to end, which needs far more demonstrations; (b) a wrist force-torque sensor, which is cheap but cannot localise slip to a finger; (c) a hand-engineered shear threshold, which is fast to try but tuned per object. I pick the frozen encoder plus slip head, and keep (c) as a baseline.

Numbers: to show a drop from 18% to 6%, about 120 episodes per arm give 80% power at alpha 0.05, roughly 4 hours each at 2 minutes per episode. Arms: vision only, vision plus touch features, vision plus reflex, both.

Risks: sensor gel wear changes the signal over weeks (re-check a calibration object daily); the slip head overfits to training objects (hold out glossy objects); reflex grip force crushes soft objects.

Pitfalls: claiming touch helped when the reflex alone did; testing only on training objects.

Why this is likely: Sparsh and Digit 360 are Meta FAIR's published touch work, the listing names tactile signals among its modalities and dexterous manipulation among its research areas, and research deep dive rounds probe exactly this kind of design (medium confidence).

</details>

<a id="q14"></a>

#### Q14. Defend the sim-to-real claim in your talk against a skeptical interviewer

Round: Research deep dive or presentation. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Sim-to-real transfer.

Your slide says: "Trained only in simulation with domain randomization, the policy reaches 82% success on the real robot." The interviewer asks: "You also hand-tuned 40 real-world reset positions and a low-pass filter on the real robot. How do you know randomization did the work? What would you change with Meta's compute?"

<details>
<summary>Show answer</summary>

Task: separate the contributions, propose the experiment that would settle it, and scale the idea.

What they grade: honesty about confounds, a clean ablation, real-world trial budgets, a scaling plan that is more than "more GPUs".

Approach: Start by conceding the confound. The filter and the reset choice were tuned on the real robot, so 82% is not a pure sim-only number. Then give the ablation. Four conditions on the same 30 seeded real layouts: no randomization without the filter, no randomization with the filter, randomization without the filter, randomization with the filter. That is 120 trials per seed of the training run, about 4 hours at 2 minutes each. Three training seeds per condition, because sim-to-real results vary a lot by seed. Report each with an interval. If randomization alone moves success from, say, 40% to 75% and the filter adds 7 points, the claim holds with a footnote; if the filter does most of the work, the slide changes.

Alternatives to explain the gap: system identification of the real robot's dynamics (lower variance, but needs a model of what is wrong); real-world fine-tuning with a few demonstrations; randomization. Most teams combine them, randomizing the parameters they cannot measure and identifying the ones they can.

Scaling with Meta's compute: randomize over many embodiments and scenes, distil into one policy, and use a world model trained on real video as a learned simulator to check what the physics simulator gets wrong. Measure transfer as the gap between sim and real success on matched tasks, tracked per release.

Risks: overclaiming in the paper; the ablation changes the seeds, so differences are noise; real hardware drift between conditions (interleave them).

Pitfalls: getting defensive; answering with more results instead of the missing control.

Why this is likely: a recent Glassdoor Meta research scientist report describes a research presentation followed by technical discussions, and the listing names sim-to-real transfer and benchmarking new capabilities (medium confidence).

</details>

<a id="q15"></a>

#### Q15. Plan a research program for long-horizon mobile manipulation with milestones

Round: Research deep dive. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Research planning.

"In two years we want a mobile robot that can tidy an unseen apartment from one instruction. You lead it. What do you build first, what are the milestones, and how will we know at month 6 whether it is working?"

<details>
<summary>Show answer</summary>

Task: break an open-ended goal into measurable milestones, pick an architecture with alternatives, and name what would make you change course.

What they grade: decomposition, falsifiable milestones, honest risk, use of existing team assets.

Approach: "Tidy an apartment" decomposes into finding objects, deciding where they go, navigating, and picking and placing many object types, over tens of steps with recovery. I start with the evaluation, because the listing asks to define and benchmark new capabilities. The suite is 20 tidy tasks in simulated homes (Habitat-style scenes) and 5 real rooms, each scored by the fraction of objects in valid places.

Architecture alternatives: (a) one end-to-end policy from pixels to actions, elegant but data-hungry for 30-step horizons; (b) an LLM or VLM planner calling learned skills (navigate, pick, place, open), debuggable and quick to start; (c) a world-model planner with subgoals. I start with (b), because PARTNR-style results show where planners fail, and swap individual skills for stronger learned ones over time. (c) is a 12-month research bet.

Milestones. Month 3: skill library with 80% pick success on 50 household objects in sim, 60% on real. Month 6: 10-step tidy tasks at 50% fraction-correct in unseen sim homes, with logs labelled by failure type. Month 12: 30-step tasks in 3 unseen real rooms at 40%. Month 24: one instruction, unseen apartment, 70%. If month-6 failures are dominated by skills, invest in data; if by planning, invest in the planner and memory.

Numbers: a real evaluation of 5 rooms by 4 tasks by 5 trials is 100 runs of about 10 minutes, roughly 17 robot-hours per checkpoint, so real evaluation happens monthly, with sim evaluation nightly.

Risks: sim homes do not predict real failures (track the sim-real gap per release); skills plateau below the needed reliability (at 95% per step, a 30-step task succeeds only about 21% of the time, so per-step reliability is the main lever).

Why this is likely: a recent Glassdoor Meta research scientist report describes being asked how they would approach an open-ended research problem, and the listing asks for long-term research goals with intermediate milestones (medium confidence).

</details>

### Behavioral (no AI, as at Meta)

<a id="q16"></a>

#### Q16. How do you use AI tools in your research, and where do you refuse to trust them?

Round: Behavioral. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI-assisted research practice.

The interviewer wants one concrete story: a time an AI tool sped up your research, a time it was wrong, and the process you now follow.

<details>
<summary>Show answer</summary>

Task: one specific story with numbers, plus a repeatable rule you follow.

What they grade: concrete use, verification habits, judgment about where AI fails in research, ownership of results.

Approach: Use one situation, your task, the actions and the result, and keep it to about three minutes.

Example shape: "Last spring I needed a new evaluation harness for 12 manipulation tasks across two simulators. I used a coding assistant to write the adapters and plotting code, which took about two days instead of a week. I wrote the success predicates by hand, because they define the result, and I asked the assistant to write tests against 10 recorded episodes whose outcomes I knew. Two of its adapters failed those tests. One read the gripper state from the previous frame, an off-by-one that inflated grasp success by about 6 points. I would not have caught that by reading the diff."

Then the rule I follow. AI drafts boilerplate, tests, plots and literature search leads; I write or re-derive anything that decides a result (metrics, loss functions, data splits); every claim it makes about a paper gets checked against the paper itself, because I have seen it invent ablations that do not exist. Finally, the team now uses a shared checklist for AI-written evaluation code and keeps recorded episodes with known outcomes as tests.

Verify: the story has a number for the speed-up, a number for the error, and the change you made afterwards.

Pitfalls: claiming you never use AI (Meta expects you to); a story where the AI is never wrong; vague "I always double-check".

Follow-ups: how you would use the CoderPad assistant in the AI-enabled round; where AI helped in writing a paper and where you stopped it.

Why this is likely: 1point3acres reports that Meta's behavioral loop now includes an AI-oriented behavioral round, and Meta's careers FAQ says candidates are expected to use the provided assistant (medium confidence).

</details>

## Prep checklist

- [ ] Solve grid BFS and lowest common ancestor problems in 20 minutes each, out loud, with no AI.
- [ ] Practice one multi-file CoderPad-style task: run the tests first, then prompt, then review every diff line.
- [ ] Hand-write causal multi-head attention, a PPO loss with GAE and a diffusion training step in NumPy or PyTorch.
- [ ] Prepare a 3-minute and a 20-minute version of your strongest robot-learning paper, including one failed experiment.
- [ ] For each paper, write down the ablation a skeptical reviewer would ask for and whether you ran it.
- [ ] Read the V-JEPA 2, PARTNR and Sparsh papers and form one opinion on each that you can defend.
- [ ] Rehearse one data, model and evaluation design for an embodied system with numbers for robot hours, storage and GPU hours.
- [ ] Know Wilson intervals, Fisher's exact test and a sample size calculation for real-robot evaluations.
- [ ] Prepare one AI-use story with a measured speed-up and a caught error.
- [ ] Ask the recruiter which rounds are AI-enabled and whether there is a research presentation.

## Sources

- Meta careers, hiring process and FAQ on AI-enabled interviews: https://www.metacareers.com/hiring-process
- Meta careers, onsite preparation: https://www.metacareers.com/swe-prep-onsite
- Glassdoor, Meta research scientist interview reports: https://www.glassdoor.com.au/Interview/Meta-Research-Scientist-Interview-Questions-EI_IE40772.0,4_KO5,23.htm
- Reddit, Meta ML research scientist interview experience: https://www.reddit.com/r/leetcode/comments/1r37w7q/meta_ml_research_scientist_interview_experience
- 1point3acres, Meta interview trend reports: https://www.1point3acres.com/interview/guides/meta-interview-trend-2026q4 and https://www.1point3acres.com/interview/guides/meta-interview-trend-2026q2
- Blind, ML and AI interview threads: https://www.teamblind.com/post/been-grinding-mlai-interviews-for-the-past-6-months-oratewjm and https://www.teamblind.com/post/ai-coding-interviews-at-meta-ai-6z001dax
- First-hand blogs: https://jasonppy.github.io/story/phd-in-AI-my-experience and https://www.ahl27.com/posts/2025/02/meta-interview/
- Hello Interview, Meta AI-enabled coding: https://www.hellointerview.com/blog/meta-ai-enabled-coding
- Interview Query, Meta research scientist guide (low confidence): https://www.interviewquery.com/interview-guides/facebook-research-scientist
- Meta AI, V-JEPA 2 world model: https://ai.meta.com/blog/v-jepa-2-world-model-benchmarks
- Meta AI, Sparsh, Digit 360 and PARTNR: https://ai.meta.com/blog/fair-robotics-open-source
- PARTNR benchmark: https://aihabitat.org/partnr
- Company-wise LeetCode lists (crowd-reported tags): https://github.com/liquidslr/leetcode-company-wise-problems
