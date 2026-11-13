# OpenAI: Software Engineer, AI for Chip Design

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

OpenAI publishes an interview guide for all technical roles. It says AI tool rules vary by interview, that final interviews take 4 to 6 hours with 4 to 6 people over one or two days, virtual by default with an optional onsite in San Francisco, and that coding is graded on "well-designed solutions to the challenge, high-quality code, optimal performance, and good test coverage". The listing describes a Python engineer who builds the software around models that learn chip design: reinforcement learning environments wrapped around EDA tools, evaluations of generated RTL, PPA (power, performance, area) measurement, and reliable infrastructure for long experiments. OpenAI and Synopsys announced a chip design partnership in September 2026, and the hardware group is led by Richard Ho, so the domain questions below lean on EDA tool integration. That domain focus is inferred from the listing. Confirm the loop with the recruiter.

1. Recruiter call. About 30 minutes on background, team match, timeline and why OpenAI, including your views on AGI and safety (one 2026 roundup says a weak safety answer can end the process). AI: not relevant.
2. Technical screen. 60 to 90 minutes in CoderPad or a shared editor, in Python. A Blind thread from October 2026 says some candidates were offered a choice between a traditional coding round with no AI and an agentic coding round on an existing Python codebase with AI tools provided. A Glassdoor report describes a 90 minute screen that built a scheduler class while new requirements arrived mid-session. A 60 minute system design screen in Excalidraw often runs back-to-back with the coding screen and covers infrastructure such as job schedulers or sandboxed code execution, with no AI. Some teams send a take-home instead of or alongside the screen, with AI only if the assignment allows it; this is not confirmed for AI for Chips. AI: not allowed in the traditional and design screens, provided in the agentic track.
3. Onsite coding from scratch. About 60 minutes, a practical multi-part problem with starter code and hidden tests. Reported themes include a grid infection simulation, a credits ledger with expiry and out-of-order events, versioned key-value stores and resumable iterators. AI: not allowed.
4. Onsite coding on existing code. About 60 minutes of reading, refactoring and fixing someone else's code, which one Glassdoor candidate called the trickiest round. AI: not allowed.
5. Agentic coding (beta, not every candidate). interviewing.io says OpenAI is piloting an extra onsite round with an existing codebase and a task too large to finish by hand, where you drive an AI coding agent. AI: provided.
6. System design. About 60 minutes on infrastructure, drawn in Excalidraw. For this team, expect an environment service, an experiment platform or a sandbox for generated code. AI: not allowed.
7. Project deep dive or presentation. 45 to 60 minutes on a past project, sometimes with slides. Expect questions on your own decisions and the numbers behind them. AI: not allowed.
8. Behavioral / hiring manager. 30 to 45 minutes on cross-functional work with researchers, mission and safety, and why OpenAI over other labs. Expect AI-tool use to come up given the agentic round, though no report confirms it. AI: not allowed.
9. Decision. Timing varies by team, so ask the recruiter. AI: not relevant.

AI tools: OpenAI's guide says the rules differ per interview and the interviewer will tell you. Blind and Glassdoor reports from mid and late 2026 describe an optional agentic coding screen with AI tools provided, and interviewing.io describes an agentic onsite beta. PracHub's November 2026 summary of 51 reports found no AI-assisted round, so prepare to write everything yourself and treat the agentic round as a possibility. Q7 and Q8 assume AI is provided; every other question assumes no AI.

Classic algorithm questions are limited to one (Q3). OpenAI coding is practical, multi-part and stateful, and a Blind thread from October 2026 describes the agentic round as "not an algorithm-memorization exercise". A hiring guide still says LeetCode medium problems appear, so Q3 covers that case.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a priority job queue for EDA experiment runs with retries and cancel | Build | Technical screen | Not allowed | Medium |
| [Q2](#q2) | Build a tool license ledger with expiring grants and out-of-order events | Build | Onsite coding | Not allowed | Hard |
| [Q3](#q3) | Simulate how routing congestion spreads across a placement grid | Classic algorithm | Technical screen | Not allowed | Medium |
| [Q4](#q4) | Build a versioned result store with snapshots, deletes and compaction | Build | Onsite coding | Not allowed | Medium |
| [Q5](#q5) | This RL environment wrapper returns zero reward. Find the bugs | Debug | Onsite coding | Not allowed | Medium |
| [Q6](#q6) | Refactor and fix this timing report summarizer | Debug | Onsite coding | Not allowed | Medium |
| [Q7](#q7) | Add a second simulator backend to an existing harness with an AI coding agent | Build | Agentic coding | Provided (AI coding agent) | Medium |
| [Q8](#q8) | Review this AI-written wrapper around a synthesis tool before it merges | Review | Agentic coding | Provided (AI coding agent) | Medium |
| [Q9](#q9) | A PPA sweep of 2000 configs takes two days. Make it fast | Optimize | Onsite coding | Not allowed | Medium |
| [Q10](#q10) | Design the evaluation for a model that writes RTL, and code its metrics | Concept plus code | System design | Not allowed | Hard |
| [Q11](#q11) | Rollout success fell from 34% to 12% after a cluster move. Find the cause | Trace debug | Onsite coding | Not allowed | Medium |
| [Q12](#q12) | A tool call fails 15% of the time. How many retries does a 40-call rollout need? | Concept plus code | Technical screen | Not allowed | Medium |
| [Q13](#q13) | Design an RL environment service around license-limited EDA tools | Design | System design | Not allowed | Hard |
| [Q14](#q14) | Design a platform for reproducible, long-running chip design experiments | Design | System design | Not allowed | Hard |
| [Q15](#q15) | Design a sandbox that runs model-generated RTL and scripts safely | Design | System design | Not allowed | Hard |
| [Q16](#q16) | How do you use AI coding agents, and how do you check what they produce? | Behavioral | Hiring manager | Not allowed | Medium |

### Practical coding (no AI)

<a id="q1"></a>

#### Q1. Build a priority job queue for EDA experiment runs with retries and cancel

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Heaps, state machines, retry policy.

Build `JobQueue` with `submit(job_id, priority)`, `run_next(execute)` and `cancel(job_id)`. Higher priority runs first, ties run in submit order. A `TimeoutError` from `execute` is transient and the job goes back in the queue until it has used `max_retries` retries. A `RuntimeError` (for example a license denial with a bad config) fails the job at once. Cancel only affects queued jobs. Expect the interviewer to add parts, as in the reported scheduler screen: retries first, then cancel, then stats.

<details>
<summary>Show answer</summary>

Task: A heap-backed job queue with priority order, bounded retries on timeouts and lazy cancel.

Approach: one heap of `(-priority, sequence, job_id)` and a dict of job records. The sequence number from `itertools.count` makes ties stable and stops Python from comparing job ids. Cancel is lazy. It flips the state, and `run_next` skips anything that is no longer queued when popped, which is O(log n) instead of a linear search through the heap. Each job has an explicit state (queued, running, done, failed, cancelled), so every transition is visible and testable.

```python
import heapq
import itertools

class JobQueue:
    def __init__(self, max_retries=2):
        self.max_retries = max_retries
        self._heap = []
        self._seq = itertools.count()
        self.jobs = {}

    def submit(self, job_id, priority=0):
        if job_id in self.jobs:
            raise ValueError(f"duplicate job {job_id}")
        self.jobs[job_id] = {"state": "queued", "attempts": 0, "priority": priority}
        heapq.heappush(self._heap, (-priority, next(self._seq), job_id))

    def run_next(self, execute):
        while self._heap:
            _, _, job_id = heapq.heappop(self._heap)
            job = self.jobs[job_id]
            if job["state"] != "queued":
                continue
            job["attempts"] += 1
            job["state"] = "running"
            try:
                execute(job_id)
                job["state"] = "done"
            except TimeoutError:
                self._retry_or_fail(job_id, "timeout")
            except RuntimeError as e:
                job["state"], job["error"] = "failed", str(e)
            return job_id
        return None

    def _retry_or_fail(self, job_id, reason):
        job = self.jobs[job_id]
        if job["attempts"] <= self.max_retries:
            job["state"] = "queued"
            heapq.heappush(self._heap, (-job["priority"], next(self._seq), job_id))
        else:
            job["state"], job["error"] = "failed", reason

    def cancel(self, job_id):
        if self.jobs[job_id]["state"] == "queued":
            self.jobs[job_id]["state"] = "cancelled"

if __name__ == "__main__":
    calls = []
    flaky = {"b": 2}
    def execute(job_id):
        calls.append(job_id)
        if job_id == "bad":
            raise RuntimeError("license denied")
        if flaky.get(job_id, 0) > 0:
            flaky[job_id] -= 1
            raise TimeoutError
    q = JobQueue(max_retries=2)
    q.submit("a", priority=0); q.submit("b", priority=5); q.submit("bad", priority=1); q.submit("c", 0)
    q.cancel("c")
    while q.run_next(execute):
        pass
    assert calls == ["b", "b", "b", "bad", "a"], calls
    assert q.jobs["b"]["state"] == "done" and q.jobs["b"]["attempts"] == 3
    assert q.jobs["bad"]["state"] == "failed" and q.jobs["c"]["state"] == "cancelled"
    q2 = JobQueue(max_retries=1); q2.submit("x")
    always = lambda j: (_ for _ in ()).throw(TimeoutError())
    while q2.run_next(always):
        pass
    assert q2.jobs["x"] == {"state": "failed", "attempts": 2, "priority": 0, "error": "timeout"}
    print("q1 ok")
```

Verify: the self-test checks order by priority, three attempts for a job that times out twice, an immediate failure for a hard error, a cancelled job that never runs, and the exact record of a job that runs out of retries. Run it with `python3 q1.py`.

Pitfalls: pushing a retried job with a fresh priority instead of its original one; retrying hard errors, which burns licenses on configs that will never pass; removing cancelled jobs from the middle of the heap; and comparing tuples that end in unorderable objects when two priorities tie.

Alternatives: a sorted list with `bisect.insort` keeps jobs in order, but each submit shifts up to n entries, so 10,000 queued jobs cost up to 10,000 moves per insert. Per-priority deques give O(1) submit and pop when there are only a few levels (say 3), but they need a scan over levels and do not fit arbitrary numeric priorities. The heap is O(log n), about 14 steps at 10,000 jobs, for any priority values, so it is the default; switch to deques if the interviewer fixes the priority levels.

Why this is likely: a Glassdoor report from October 2026 describes a 90 minute screen that built a scheduler class with requirements added mid-session, and the role runs many EDA jobs that fail for both transient and permanent reasons.

</details>

<a id="q2"></a>

#### Q2. Build a tool license ledger with expiring grants and out-of-order events

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Event replay, expiry, ordering.

Related practice: [Design Authentication Manager](https://leetcode.com/problems/design-authentication-manager/) (Medium)

Teams receive seat-hour grants for a paid EDA tool. `grant(id, amount, t, expires)` adds hours that are usable at times `t <= time < expires`. `consume(amount, t)` spends hours, taking from the grant that expires first. Events can arrive out of order, so a late `consume` at t=5 must change the balance seen at t=10. `balance(t)` returns hours left at time t, and `unpaid(t)` returns usage no grant covered.

<details>
<summary>Show answer</summary>

Task: A replayable ledger that spends the earliest-expiring grant first and stays correct when events arrive late.

Approach: store every event in a sorted list keyed by `(time, arrival order)` and replay up to the query time. Replay is simple and correct when events arrive late, which is the hard part of this problem. During replay, drop grants whose expiry is at or before the current event time, then apply the event. Consumption walks live grants sorted by expiry, so short-lived hours are spent before they are lost. Usage beyond all live grants goes to `unpaid` instead of making a balance negative.

```python
import bisect

class SeatLedger:
    """Seat-hours per team. Events may arrive out of order; balance(t) replays events up to t."""

    def __init__(self):
        self.events = []  # (time, order, kind, payload)
        self._n = 0

    def _add(self, t, kind, payload):
        bisect.insort(self.events, (t, self._n, kind, payload))
        self._n += 1

    def grant(self, grant_id, amount, t, expires):
        if amount <= 0 or expires <= t:
            raise ValueError("bad grant")
        self._add(t, "grant", (grant_id, amount, expires))

    def consume(self, amount, t):
        self._add(t, "use", amount)

    def _replay(self, until):
        live = {}  # grant_id -> [remaining, expires]
        unpaid = 0
        for t, _, kind, p in self.events:
            if t > until:
                break
            for gid in [g for g, (_, exp) in live.items() if exp <= t]:
                del live[gid]
            if kind == "grant":
                gid, amount, exp = p
                live[gid] = [amount, exp]
            else:
                need = p
                for gid in sorted(live, key=lambda g: live[g][1]):  # earliest expiry first
                    take = min(need, live[gid][0])
                    live[gid][0] -= take
                    need -= take
                    if need == 0:
                        break
                unpaid += need
        live = {g: v for g, v in live.items() if v[1] > until}
        return live, unpaid

    def balance(self, t):
        live, _ = self._replay(t)
        return sum(r for r, _ in live.values())

    def unpaid(self, t):
        return self._replay(t)[1]

if __name__ == "__main__":
    L = SeatLedger()
    L.grant("long", 100, t=0, expires=100)
    L.grant("short", 30, t=0, expires=20)
    L.consume(40, t=10)                 # takes 30 from short, 10 from long
    assert L.balance(10) == 90
    assert L.balance(20) == 90          # short expired, already empty
    L.consume(5, t=5)                   # arrives late, applies before t=10
    assert L.balance(10) == 85 and L.balance(25) == 85
    L.consume(200, t=50)
    assert L.balance(50) == 0 and L.unpaid(50) == 115
    assert L.balance(150) == 0          # everything expired
    L2 = SeatLedger(); L2.grant("g", 10, 0, 5); L2.consume(3, 5)
    assert L2.balance(5) == 0 and L2.unpaid(5) == 3   # expiry at t=5 happens before use at t=5
    print("q2 ok")
```

Verify: the self-test covers spending from the earliest expiring grant, a late event that changes an earlier balance, overdraw into `unpaid`, full expiry, and the boundary where a grant expires at the same time as a use.

Pitfalls: spending the largest grant first; applying late events only to the current balance; and boundary errors at `t == expires`. Say the rule out loud before coding.

Alternatives: replay costs O(n * g log g) per query, where g is the number of live grants, and each insert is O(n). Keeping live grants in a heap keyed by expiry brings replay to O(n log g). If the interviewer asks for more speed, cache a checkpoint state every K events and replay from the latest checkpoint before the late event.

Why this is likely: the 1point3acres 2026 roundup lists a GPU credits ledger as a high-frequency OpenAI question; out-of-order events and the EDA-license framing are this guide's extension.

</details>

<a id="q3"></a>

#### Q3. Simulate how routing congestion spreads across a placement grid

Round: Technical screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Multi-source BFS, thresholds.

Related practice: [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) (Medium)

A grid has hotspot tiles `H`, routable tiles `.` and blockages `#`. Each day, every routable tile next to (up, down, left, right) a congested tile becomes congested. Return the number of days until nothing changes and the number of tiles that never become congested. Part 2: each tile has a threshold, and it only becomes congested once that many neighbors are congested.

<details>
<summary>Show answer</summary>

Task: Multi-source BFS for the days until congestion stops spreading and the tiles it never reaches, then a threshold variant.

Approach: multi-source BFS that starts from all hotspots at day 0. The day of a tile is the day of the neighbor that pushed it plus one, and the answer is the largest day. For part 2, count congested neighbors per tile and enqueue a tile only when its count reaches its threshold. Its day is one more than the neighbor that pushed it over, which is correct because BFS processes tiles in day order. Each tile is enqueued once and each edge is checked once, so time and memory are O(rows times cols).

```python
from collections import deque

def spread_days(grid, threshold=None):
    """grid: 'H' hotspot, '.' routable tile, '#' blockage.
    threshold[r][c]: congested neighbours a tile needs (default 1).
    Returns (days until stable, tiles never congested)."""
    R, C = len(grid), len(grid[0])
    need = [[(threshold[r][c] if threshold else 1) for c in range(C)] for r in range(R)]
    hits = [[0] * C for _ in range(R)]
    day = [[-1] * C for _ in range(R)]
    q = deque()
    for r in range(R):
        for c in range(C):
            if grid[r][c] == "H":
                day[r][c] = 0
                q.append((r, c))
    last = 0
    while q:
        r, c = q.popleft()
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] == "." and day[nr][nc] < 0:
                hits[nr][nc] += 1
                if hits[nr][nc] >= need[nr][nc]:
                    day[nr][nc] = day[r][c] + 1
                    last = max(last, day[nr][nc])
                    q.append((nr, nc))
    never = sum(1 for r in range(R) for c in range(C) if grid[r][c] == "." and day[r][c] < 0)
    return last, never

if __name__ == "__main__":
    g = ["H..#",
         ".#..",
         "...#",
         "#..H"]
    assert spread_days(g) == (4, 0), spread_days(g)
    assert spread_days(["H#."]) == (0, 1)
    assert spread_days(["..."]) == (0, 3)
    t = [[1, 2, 1],
         [1, 1, 1]]
    # corner tile (0,1) needs 2 congested neighbours
    assert spread_days(["H.H", "..."], t) == (2, 0)
    assert spread_days(["H..", "..."], [[1, 2, 1], [1, 1, 1]]) == (4, 0)
    assert spread_days(["H.."], [[1, 2, 1]]) == (0, 2)
    print("q3 ok")
```

Verify: the tests cover the base grid, a hotspot walled off by a blockage, a grid with no hotspots, a threshold that delays a tile, and a threshold that is never met.

Pitfalls: simulating day by day with a full grid scan, which is O(days times cells); counting a neighbor twice; marking a tile as visited on the first touch in part 2, which ignores the threshold; and returning -1 when the question asks for a count of tiles that are never reached. Ask which output the interviewer wants.

Alternatives: rescanning the whole grid each day is O(days times cells), so a 1,000 by 1,000 grid that takes 1,000 days to settle costs about a billion checks. A union-find over routable tiles tells you which tiles are ever reached but not on which day, and it cannot model thresholds. Multi-source BFS is O(cells), about a million steps on the same grid, and gives the day for every tile, so it wins.

Why this is likely: the 1point3acres 2026 roundup names a plant infection grid simulation as the most reported OpenAI coding question of 2026, and congestion spread is the closest chip design version of it.

</details>

<a id="q4"></a>

#### Q4. Build a versioned result store with snapshots, deletes and compaction

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Version history, binary search, tombstones.

Related practice: [Snapshot Array](https://leetcode.com/problems/snapshot-array/) (Medium)

Experiment results are stored by key. Support `put`, `delete`, `snapshot()` which returns a version number, `get(key, at=None)` which reads the value as of a snapshot, and `compact(keep_from)` which frees history that no snapshot at or after `keep_from` can read.

<details>
<summary>Show answer</summary>

Task: A multi-version store with snapshot reads, tombstone deletes and compaction that keeps every readable version.

Approach: a global version counter that every write bumps, and per key two parallel lists of versions and values. Versions are appended in increasing order, so `get` is a `bisect_right` on the version list, O(log h) for h writes to the key. A delete writes a tombstone instead of removing history, so older snapshots still see the old value. Compaction keeps, for each key, the last entry at or before `keep_from` plus everything after it, and drops keys whose only remaining entry is an old tombstone.

```python
import bisect

_TOMBSTONE = object()

class VersionedStore:
    def __init__(self):
        self.version = 0
        self.hist = {}  # key -> ([versions], [values])

    def _write(self, key, value):
        self.version += 1
        vs, vals = self.hist.setdefault(key, ([], []))
        vs.append(self.version)
        vals.append(value)
        return self.version

    def put(self, key, value):
        return self._write(key, value)

    def delete(self, key):
        if self.get(key) is None:
            raise KeyError(key)
        return self._write(key, _TOMBSTONE)

    def snapshot(self):
        return self.version

    def get(self, key, at=None):
        at = self.version if at is None else at
        if key not in self.hist:
            return None
        vs, vals = self.hist[key]
        i = bisect.bisect_right(vs, at) - 1
        if i < 0 or vals[i] is _TOMBSTONE:
            return None
        return vals[i]

    def compact(self, keep_from):
        """Drop history no snapshot >= keep_from can read."""
        for key in list(self.hist):
            vs, vals = self.hist[key]
            i = max(bisect.bisect_right(vs, keep_from) - 1, 0)
            vs[:], vals[:] = vs[i:], vals[i:]
            if len(vs) == 1 and vals[0] is _TOMBSTONE and vs[0] <= keep_from:
                del self.hist[key]

if __name__ == "__main__":
    s = VersionedStore()
    s.put("run1/ppa", {"area": 120})
    snap1 = s.snapshot()
    s.put("run1/ppa", {"area": 110})
    s.put("run2/ppa", {"area": 90})
    snap2 = s.snapshot()
    s.delete("run2/ppa")
    assert s.get("run1/ppa", snap1) == {"area": 120}
    assert s.get("run1/ppa") == {"area": 110}
    assert s.get("run2/ppa", snap1) is None and s.get("run2/ppa", snap2) == {"area": 90}
    assert s.get("run2/ppa") is None
    s.compact(keep_from=snap2)
    assert s.get("run1/ppa", snap2) == {"area": 110} and s.get("run2/ppa", snap2) == {"area": 90}
    s.compact(keep_from=s.snapshot())
    assert "run2/ppa" not in s.hist and s.get("run1/ppa") == {"area": 110}
    try:
        s.delete("missing"); raise AssertionError
    except KeyError:
        pass
    print("q4 ok")
```

Verify: the test reads the same key at two snapshots, deletes a key and reads it before and after, compacts while a snapshot is still in use, then compacts everything and checks that the dead key is gone.

Pitfalls: deleting history on `delete`, which breaks old snapshots; compacting the entry that an open snapshot still needs; storing a full copy of the store per snapshot, which costs O(n) memory per snapshot; and using `None` as the tombstone so that a stored `None` looks deleted.

Alternatives: a full copy of the store per snapshot makes reads O(1) but costs O(n) memory per snapshot, so 1,000 snapshots of 1 million keys hold a billion entries. A persistent balanced tree shares structure and makes snapshots O(1), but it takes far longer to write in a 60 minute round. Per-key version lists cost memory only per write and O(log h) per read, so they fit the round.

Why this is likely: interviewing.io lists time-based data structures and versioned data stores among OpenAI-specific topics, and a Glassdoor report lists an LSM memtable question with tombstones.

</details>

### Reading and fixing existing code (no AI)

<a id="q5"></a>

#### Q5. This RL environment wrapper returns zero reward. Find the bugs

Round: Onsite coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Environment state, mutable defaults, reward math.

An environment gives a policy reward equal to the cell area it removes per step, measured by a simulator. Training reports reward near zero and episodes that end at odd times. Find and fix the bugs in this code.

```python
class FakeSim:
    """Stands in for an EDA tool: returns cell area for a design state."""
    def area(self, state):
        return 100 - 5 * state.count("opt")

class PlacementEnv:
    def __init__(self, sim, max_steps=3, history=[]):
        self.sim = sim
        self.max_steps = max_steps
        self.history = history
        self.steps = 0
        self.state = []
        self.last_area = None

    def reset(self):
        self.state = []
        self.last_area = self.sim.area(self.state)
        return list(self.state)

    def step(self, action):
        self.state.append(action)
        self.steps += 1
        reward = self.last_area - self.sim.area(self.state[:-1])
        self.last_area = self.sim.area(self.state)
        self.history.append((action, reward))
        done = self.steps > self.max_steps
        return list(self.state), reward, done
```

<details>
<summary>Show answer</summary>

Task: Find the bugs that zero the reward and break episode ends, and prove each fix with a test.

Approach: write a tiny deterministic episode with a fake simulator before reading closely, so each bug shows up as a failing assertion. Then go line by line. There are four bugs. First, `history=[]` is a mutable default argument, so every environment shares one history list. Second, `reset` does not clear `steps` (or the history), so the second episode ends at once. Third, the reward compares `last_area` against the area of the state before the action, which is the same thing, so reward is always zero. Fourth, `done = self.steps > max_steps` runs one step too many. The fix also raises if `step` is called before `reset`.

```python
class FakeSim:
    def area(self, state):
        return 100 - 5 * state.count("opt")

class PlacementEnv:
    def __init__(self, sim, max_steps=3, history=None):
        self.sim = sim
        self.max_steps = max_steps
        self.history = [] if history is None else history
        self.steps = 0
        self.state = []
        self.last_area = None

    def reset(self):
        self.state = []
        self.steps = 0
        self.history.clear()
        self.last_area = self.sim.area(self.state)
        return list(self.state)

    def step(self, action):
        if self.last_area is None:
            raise RuntimeError("call reset() first")
        self.state.append(action)
        self.steps += 1
        new_area = self.sim.area(self.state)
        reward = self.last_area - new_area
        self.last_area = new_area
        self.history.append((action, reward))
        done = self.steps >= self.max_steps
        return list(self.state), reward, done

def episode(env, actions):
    env.reset()
    out = []
    for a in actions:
        _, r, done = env.step(a)
        out.append((r, done))
        if done:
            break
    return out

if __name__ == "__main__":
    env = PlacementEnv(FakeSim())
    for _ in range(2):
        assert episode(env, ["opt", "noop", "opt"]) == [(5, False), (0, False), (5, True)]
    a, b = PlacementEnv(FakeSim()), PlacementEnv(FakeSim())
    a.reset(); a.step("opt")
    assert b.history == []
    print("q5 ok")
```

Verify: the test runs two episodes on one environment and checks rewards `[5, 0, 5]` with done on step 3, then checks two environments do not share history. Run the same checks against the buggy class: they fail on episode 1 (all rewards zero), episode 2 (done after one step) and shared history; the fixed class passes all three.

Pitfalls: fixing the first bug you see and stopping; changing the reward sign; and not adding a test that runs two episodes, which is how the reset bug hides.

Alternatives: stepping through a training run in a debugger finds one bug at a time, and each run waits on the simulator; printing rewards from the training loop shows the zeros but not their cause. A deterministic episode with a fake simulator runs in milliseconds, shows all four bugs as failing assertions, and stays as a regression test, so start there.

Why this is likely: the onsite has a round on fixing existing code that a Glassdoor candidate called the trickiest, and the listing centers on building RL environments.

</details>

<a id="q6"></a>

#### Q6. Refactor and fix this timing report summarizer

Round: Onsite coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Parsing, refactoring, static timing basics.

This function summarizes a static timing report into worst negative slack (WNS), total negative slack (TNS) and the violation count. Results look wrong. Fix it and make it testable.

```python
def report_summary(path):
    lines = open(path).read().split("\n")
    wns = 0
    tns = 0
    n = 0
    for i in range(len(lines)):
        l = lines[i]
        if "slack" in l:
            v = abs(float(l.split()[0]))
            if "VIOLATED" in l:
                n = n + 1
            if v > wns:
                wns = v
            tns = tns + v
    return wns, tns, n
```

<details>
<summary>Show answer</summary>

Task: Fix the slack sign bug and split parsing from the math so both can be tested.

Approach: first state the definitions. WNS is the most negative slack, or 0 when nothing fails. TNS is the sum of negative slacks only. The bug is `abs()`. It turns -0.12 into 0.12, so WNS is reported as a positive number and TNS adds the passing path's +0.05 to the violations, giving 0.20 instead of -0.15. The function also mixes file I/O, parsing and math, matches any line with the word slack, never closes the file, and loses which path failed. Split it into `parse_paths` (lines to records with start, end and slack) and `summarize` (records to numbers), and make the parser strict, so a slack line without a header or a sign that disagrees with MET or VIOLATED raises instead of being counted.

```python
import re
from dataclasses import dataclass

SLACK = re.compile(r"^\s*(-?\d+(?:\.\d+)?)\s+slack \((MET|VIOLATED)\)\s*$")
START = re.compile(r"^Startpoint:\s+(\S+)")
END = re.compile(r"^Endpoint:\s+(\S+)")

@dataclass
class Path:
    start: str
    end: str
    slack: float

def parse_paths(lines):
    paths, start, end = [], None, None
    for line in lines:
        if m := START.match(line):
            start, end = m.group(1), None
        elif m := END.match(line):
            end = m.group(1)
        elif m := SLACK.match(line):
            if start is None or end is None:
                raise ValueError(f"slack line without a path header: {line!r}")
            slack = float(m.group(1))
            if (m.group(2) == "VIOLATED") != (slack < 0):
                raise ValueError(f"status and sign disagree: {line!r}")
            paths.append(Path(start, end, slack))
            start = end = None
    return paths

def summarize(paths):
    if not paths:
        return {"wns": 0.0, "tns": 0.0, "violations": 0}
    neg = [p.slack for p in paths if p.slack < 0]
    return {"wns": min(0.0, min(p.slack for p in paths)),
            "tns": round(sum(neg), 6),
            "violations": len(neg)}

REPORT = """Startpoint: core/alu_reg[3]
Endpoint: core/acc_reg[7]
  -0.120   slack (VIOLATED)
Startpoint: core/pc_reg[0]
Endpoint: core/pc_reg[1]
   0.050   slack (MET)
Startpoint: io/in_reg
Endpoint: core/acc_reg[2]
  -0.030   slack (VIOLATED)
""".splitlines()

if __name__ == "__main__":
    s = summarize(parse_paths(REPORT))
    assert s == {"wns": -0.12, "tns": -0.15, "violations": 2}, s
    assert summarize(parse_paths(REPORT[3:6])) == {"wns": 0.0, "tns": 0.0, "violations": 0}
    assert summarize([]) == {"wns": 0.0, "tns": 0.0, "violations": 0}
    try:
        parse_paths(["  -0.1 slack (MET)"]); raise AssertionError
    except ValueError:
        pass
    print("q6 ok")
```

Verify: on the sample report the old function returns `(0.12, 0.2, 2)` and the new one returns WNS -0.12, TNS -0.15 and 2 violations. Tests also cover a clean report, an empty report and a malformed line.

Pitfalls: changing behavior and structure in one step without a test that pins the old correct parts; summing floats and comparing them exactly; and silently skipping lines the parser does not understand.

Alternatives: removing only `abs()` fixes the numbers on this report but keeps the loose `"slack" in l` match, which also catches header and comment lines. A full parser for the whole report format is the most robust but takes longer than the round. A strict regex parser plus a pure `summarize` function fixes the bug, is easy to test, and fits in 60 minutes, so use it.

Why this is likely: candidates report a refactor-and-fix round at the onsite, and parsing EDA tool reports into PPA numbers is core work for this team.

</details>

### Agentic coding (AI provided)

<a id="q7"></a>

#### Q7. Add a second simulator backend to an existing harness with an AI coding agent

Round: Agentic coding. AI policy: Provided (AI coding agent). Type: Build. Difficulty: Medium. Topic: Unfamiliar codebases, plugin registries, validating agent output.

You get a multi-file Python harness that compiles and simulates Verilog with Icarus Verilog. Add Verilator as a second backend chosen by name, without breaking existing users, using the AI agent in the editor. You are graded on how you understand the code, plan, check the agent's output and own the result. The repo:

```text
harness/
  __init__.py
  cli.py            # argparse entry point: harness run --top tb --src ...
  runner.py         # builds and runs the simulator command
  jobs.py           # SimJob dataclass, loads job YAML
  results.py        # turns simulator stdout into pass or fail
  config.py         # tool paths and default flags
tests/
  conftest.py
  test_runner.py    # asserts the exact iverilog argv
  test_results.py
  fixtures/
    expected.json   # expected pass or fail per sample job
    alu_tb.sv
pyproject.toml
```

The current `runner.py`, with the simulator hard-coded:

```python
import subprocess

def parse_result(stdout):  # lives in results.py
    return "TEST PASSED" in stdout and "ERROR" not in stdout

def run_job(job, workdir):
    defines = [f"-D{k}={v}" for k, v in sorted(job.defines.items())]
    compile_cmd = ["iverilog", "-g2012", "-s", job.top,
                   "-o", f"{workdir}/sim.vvp", *defines, *job.sources]
    subprocess.run(compile_cmd, check=True, cwd=workdir)
    sim = subprocess.run(["vvp", f"{workdir}/sim.vvp"],
                         capture_output=True, text=True, cwd=workdir)
    return parse_result(sim.stdout)
```

<details>
<summary>Show answer</summary>

Task: Add Verilator behind a backend interface with the agent, checking every diff it produces.

Approach: spend the first ten minutes without the agent. Run the tests, read the entry point, and find where the simulator command is built. Then write a short plan in a comment: extract a backend interface, register Icarus under its current behavior, add Verilator, and route the CLI flag through `get_backend`. Give the agent one step at a time with the file names and the interface, and read every diff before accepting it. The prompts, one per step:

1. "In harness/runner.py, move the iverilog compile command, the vvp run command and the pass check into a class Icarus in a new file harness/backends.py, with build_cmd(job, workdir), run_cmd(job, workdir) and passed(stdout). Keep behavior identical. Run pytest tests/test_runner.py." In the diff, check that the argv is exactly the same and that `tests/` is untouched.
2. "In harness/backends.py, add register(name) and get_backend(name), register Icarus as iverilog, and give run_job a backend argument that defaults to iverilog. Run the full test suite." In the diff, check that the default keeps old callers working and that an unknown name raises a clear error.
3. "Add a Verilator backend using --binary, --top-module and -Mdir, add argv unit tests for it, and add an integration test that is skipped when verilator is not on PATH." In the diff, check each flag against `verilator --help` and confirm the new tests fail if the class is removed.

The target shape is a small registry:

```python
import subprocess
from dataclasses import dataclass, field

BACKENDS = {}

def register(name):
    def deco(cls):
        if name in BACKENDS:
            raise ValueError(f"backend {name!r} already registered")
        BACKENDS[name] = cls
        return cls
    return deco

@dataclass
class SimJob:
    sources: list
    top: str
    defines: dict = field(default_factory=dict)

def _defs(job):
    return [f"-D{k}={v}" for k, v in sorted(job.defines.items())]

@register("iverilog")
class Icarus:
    def build_cmd(self, job, workdir):
        return ["iverilog", "-g2012", "-s", job.top, "-o", f"{workdir}/sim.vvp", *_defs(job), *job.sources]

    def run_cmd(self, job, workdir):
        return ["vvp", f"{workdir}/sim.vvp"]

    def passed(self, stdout):
        return "TEST PASSED" in stdout and "ERROR" not in stdout

@register("verilator")
class Verilator:
    def build_cmd(self, job, workdir):
        return ["verilator", "--binary", "--top-module", job.top, "-Mdir", workdir, *_defs(job), *job.sources]

    def run_cmd(self, job, workdir):
        return [f"{workdir}/V{job.top}"]

    def passed(self, stdout):
        return "TEST PASSED" in stdout and "%Error" not in stdout

def get_backend(name):
    try:
        return BACKENDS[name]()
    except KeyError:
        raise ValueError(f"unknown backend {name!r}; known: {sorted(BACKENDS)}") from None

def run_job(job, workdir, backend="iverilog", run=subprocess.run):
    b = get_backend(backend)
    run(b.build_cmd(job, workdir), check=True, cwd=workdir)
    sim = run(b.run_cmd(job, workdir), capture_output=True, text=True, cwd=workdir)
    return b.passed(sim.stdout)

if __name__ == "__main__":
    job = SimJob(["alu.sv", "tb.sv"], "tb", {"WIDTH": 8})
    cmd = get_backend("verilator").build_cmd(job, "/w")
    assert cmd[:4] == ["verilator", "--binary", "--top-module", "tb"] and cmd[-3:] == ["-DWIDTH=8", "alu.sv", "tb.sv"]
    assert get_backend("iverilog").build_cmd(job, "/w")[3] == "tb"
    assert not get_backend("verilator").passed("TEST PASSED\n%Error: x")
    assert get_backend("iverilog").run_cmd(job, "/w") == ["vvp", "/w/sim.vvp"]
    assert get_backend("verilator").run_cmd(job, "/w") == ["/w/Vtb"]
    calls = []
    def fake_run(cmd, **kw):
        calls.append(cmd)
        return type("R", (), {"stdout": "TEST PASSED"})()
    assert run_job(job, "/w", "verilator", run=fake_run) and calls[1] == ["/w/Vtb"]
    for bad in (lambda: register("iverilog")(object), lambda: get_backend("vcs")):
        try:
            bad(); raise AssertionError
        except ValueError:
            pass
    print("q7 ok")
```

Verify: unit tests check the exact argv each backend builds, so they need no simulator installed. Add one integration test, skipped when `verilator` is not on PATH, that runs a two-line testbench. Run the old test suite unchanged to show Icarus users see no difference.

Pitfalls: in a run like this the agent will often "fix" a failing test by editing the shared fixture's expected value instead of the code. Catch it by reading the diff to tests separately from the diff to code, and say so out loud. Also watch for invented command-line flags; check each flag against `verilator --help` rather than trusting the agent.

Alternatives: an if/elif on the backend name in runner.py is the smallest diff, but every new simulator edits runner.py and its tests. Python entry points (importlib.metadata) let backends live in separate packages but add packaging work the round does not need. A module-level registry keeps runner.py closed to change and is about 15 lines, so use it.

Risks: a registry that raises on duplicate names can break hot reload in notebooks. Mention it and keep the strict behavior, because a silent override is worse.

Why this is likely: a Blind thread from October 2026 describes an optional agentic coding screen on an existing Python codebase graded on understanding, planning, validation and ownership, and harnesses around EDA tools are the core of this role.

</details>

<a id="q8"></a>

#### Q8. Review this AI-written wrapper around a synthesis tool before it merges

Round: Agentic coding. AI policy: Provided (AI coding agent). Type: Review. Difficulty: Medium. Topic: subprocess safety, hallucinated APIs, error handling.

A teammate's agent wrote this wrapper for an in-house `synth_wrapper` CLI. It "works on my machine". Review it, fix it and prove the fix.

```python
import subprocess

def run_synth(top, timeout=600):
    cmd = f"synth_wrapper --top {top} --effort high"
    p = subprocess.run(cmd, shell=True, capture_output=True, text=True,
                       timeout=timeout, kill_on_timeout=True)
    return p.stdout.splitlines()[-20:-1]
```

<details>
<summary>Show answer</summary>

Task: Find the defects in the agent's wrapper, fix them and test the fix.

Approach: review for correctness and safety before style. There are four defects. First, `shell=True` with `top` in an f-string is command injection, and `top` comes from model output in this system, so a name like `alu; rm -rf ~` runs. Second, `kill_on_timeout` is not a `subprocess.run` argument; it raises `TypeError` on every call, so the "works on my machine" claim is false. `run` already kills the child on timeout. Third, `[-20:-1]` returns 19 lines and drops the last one, which is usually the summary. Fourth, it never checks `returncode`, so a failed synthesis looks like success with a partial log. The fix passes a list, converts timeouts and non-zero exits into one error type, and returns the true tail.

Use the agent for the mechanical part: "Rewrite run_synth to pass a list to subprocess.run, raise ToolError on timeout or non-zero exit, return the last 20 lines, and add tests that run a fake tool through sys.executable." A wrong output to catch: a first draft that keeps `shell=True` and wraps `top` in `shlex.quote`. It passes the injection test, but a shell is still in the path, so the next unquoted argument reopens the hole. Reject it, ask for the list form, and rerun the tests, including the `; rm -rf` argument.

```python
import subprocess, sys

class ToolError(RuntimeError):
    pass

def run_tool(argv, timeout_s=600, tail=20):
    """argv is a list; never a shell string. Returns last `tail` lines of output."""
    if not argv or not all(isinstance(a, str) for a in argv):
        raise TypeError("argv must be a non-empty list of str")
    try:
        p = subprocess.run(argv, capture_output=True, text=True, timeout=timeout_s)
    except subprocess.TimeoutExpired as e:
        raise ToolError(f"timeout after {timeout_s}s") from e
    lines = p.stdout.splitlines()
    if p.returncode != 0:
        raise ToolError(f"exit {p.returncode}: {p.stderr.strip()[-500:]}")
    return lines[-tail:]

if __name__ == "__main__":
    fake = [sys.executable, "-c", "import sys\nfor i in range(30): print(i)\nsys.exit(int(sys.argv[1]))"]
    out = run_tool(fake + ["0"])
    assert out[0] == "10" and out[-1] == "29" and len(out) == 20
    try:
        run_tool(fake + ["3"]); raise AssertionError
    except ToolError as e:
        assert "exit 3" in str(e)
    try:
        run_tool([sys.executable, "-c", "import time; time.sleep(5)"], timeout_s=0.5); raise AssertionError
    except ToolError as e:
        assert "timeout" in str(e)
    out = run_tool([sys.executable, "-c", "import sys; print(sys.argv[1])", "top; rm -rf /tmp/x"])
    assert out == ["top; rm -rf /tmp/x"]   # passed as one literal argument
    print("q8 ok")
```

Verify: tests run a fake tool through `sys.executable`, so no EDA install is needed: 30 lines of output give lines 10 to 29, exit code 3 raises, a sleep raises a timeout error, and an argument containing `; rm -rf` is printed back as one literal string.

Pitfalls: approving because the agent's own tests pass; fixing injection with quoting instead of a list; and keeping a broad `except Exception` that hides real tool errors.

Alternatives: keeping `shell=True` with `shlex.quote` blocks this injection but leaves a shell in the path for every future argument; an allowlist regex on `top` (letters, digits, underscores) is a good extra check but on its own still runs a shell. Passing an argv list removes the shell entirely, so use it, and add the allowlist as a second layer.

Why this is likely: the agentic round grades how you validate AI output, interviewing.io describes a beta onsite round with AI tools, and generated code that calls EDA tools is exactly what this team ships.

</details>

### Performance, evals and experiment reliability (no AI)

<a id="q9"></a>

#### Q9. A PPA sweep of 2000 configs takes two days. Make it fast

Round: Onsite coding. AI policy: Not allowed. Type: Optimize. Difficulty: Medium. Topic: Caching, deduplication, bounded parallelism.

A harness runs a synthesis and place-and-route flow for each of 2000 configs to measure power, performance and area. Each run takes 90 seconds, runs are serial, and the team has 32 tool licenses. Profiling shows 35% of configs are duplicates, because the generator emits the same dict with keys in different orders.

<details>
<summary>Show answer</summary>

Task: Cut the two-day sweep by removing duplicate work first, then running in parallel up to the license count.

Approach: measure first, then remove work before adding parallelism. Hash a canonical form of each config (`json.dumps` with `sort_keys=True`, then SHA-256) and run each unique config once. Keep results in a cache keyed by that hash, so a rerun after a crash or a new sweep that shares configs skips finished work. Then run unique configs in a pool with exactly as many workers as licenses, because more workers only queue on the license server and fail.

```python
import hashlib, json
from concurrent.futures import ThreadPoolExecutor

def config_key(cfg):
    return hashlib.sha256(json.dumps(cfg, sort_keys=True).encode()).hexdigest()

def evaluate_all(configs, run_flow, licenses=32, cache=None):
    cache = {} if cache is None else cache
    unique = {}
    for cfg in configs:
        unique.setdefault(config_key(cfg), cfg)
    todo = {k: c for k, c in unique.items() if k not in cache}
    with ThreadPoolExecutor(max_workers=licenses) as pool:
        for k, res in zip(todo, pool.map(run_flow, todo.values())):
            cache[k] = res
    return [cache[config_key(c)] for c in configs], len(todo)

if __name__ == "__main__":
    calls = []
    def flow(cfg):
        calls.append(cfg); return {"area": cfg["util"] * 10}
    cfgs = [{"util": 0.7, "clk": 1.0}, {"clk": 1.0, "util": 0.7}, {"util": 0.8, "clk": 1.0}]
    res, ran = evaluate_all(cfgs, flow, licenses=2)
    assert ran == 2 and len(calls) == 2 and res[0] == res[1]
    cache = {}
    evaluate_all(cfgs, flow, cache=cache); _, ran2 = evaluate_all(cfgs, flow, cache=cache)
    assert ran2 == 0
    serial_h = 2000 * 90 / 3600
    after_min = 1300 * 90 / 32 / 60
    assert serial_h == 50 and round(after_min) == 61
    print("q9 ok")
```

Numbers: serial is 2000 times 90 seconds, which is 50 hours. Dedupe leaves 1300 runs. With 32 parallel licenses that is 1300 times 90 divided by 32, about 61 minutes. A rerun with a warm cache takes seconds.

Verify: the test counts flow calls to prove duplicates with reordered keys run once and that a second call with a warm cache runs nothing.

Pitfalls: hashing `str(dict)`, which depends on key order; a cache keyed without the tool version, so a tool upgrade returns stale numbers (add the version to the key in production); reaching for a process pool, when threads are enough because each worker waits on an external tool process; and unbounded pools that exhaust licenses.

Alternatives: a process pool gives the same 32-way parallelism but adds pickling and start-up cost for workers that only wait on an external tool, so threads are enough. Asyncio with asyncio.create_subprocess_exec and a Semaphore(32) also works and scales to thousands of waiting jobs, but needs an async run_flow. Per-stage caching (reuse synthesis output across placement settings) cuts more work when configs differ only in late stages. Dedupe plus a thread pool sized to licenses is the smallest change that takes 50 hours to about an hour, so start there.

Why this is likely: OpenAI grades coding on optimal performance, and the listing asks for PPA measurement at scale with reliable infrastructure; this exact task is inferred from the listing, not a reported question (low confidence).

</details>

<a id="q10"></a>

#### Q10. Design the evaluation for a model that writes RTL, and code its metrics

Round: System design. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: pass@k, gated rewards, reward hacking.

A model writes Verilog modules from a spec. Design the evaluation that decides whether a new checkpoint is better, and code the metrics.

<details>
<summary>Show answer</summary>

Task: A layered eval (compile, hidden tests, then PPA) with an unbiased pass@k.

Approach: check correctness in layers, and only reward quality once correctness holds. Lint and compile first. Then run a testbench the model cannot see or edit, with directed and random tests, and treat a design as correct only when every test passes. Only then run synthesis and score area or timing against a reference. Report pass@k with the unbiased estimator from n samples with c correct, 1 minus C(n minus c, k) divided by C(n, k), because computing it from exactly k samples is noisy.

```python
from math import comb

def pass_at_k(n, c, k):
    if n - c < k:
        return 1.0
    return 1.0 - comb(n - c, k) / comb(n, k)

def gated_score(r):
    """r: dict with lint_ok, sim_pass_frac, synth_ok, area, area_ref."""
    if not r["lint_ok"]:
        return 0.0
    if r["sim_pass_frac"] < 1.0 or not r["synth_ok"]:
        return 0.5 * r["sim_pass_frac"] * (1 if r["synth_ok"] else 0.5)
    return 1.0 + max(0.0, min(1.0, (r["area_ref"] - r["area"]) / r["area_ref"]))

if __name__ == "__main__":
    assert abs(pass_at_k(10, 3, 1) - 0.3) < 1e-12
    assert abs(pass_at_k(10, 3, 5) - (1 - 21 / 252)) < 1e-12
    assert pass_at_k(10, 8, 3) == 1.0 and pass_at_k(10, 0, 5) == 0.0
    good = dict(lint_ok=True, sim_pass_frac=1.0, synth_ok=True, area=90, area_ref=100)
    tiny_wrong = dict(lint_ok=True, sim_pass_frac=0.9, synth_ok=True, area=10, area_ref=100)
    assert gated_score(good) > gated_score(tiny_wrong)
    assert gated_score(dict(good, lint_ok=False)) == 0
    print("q10 ok")
```

Verify: the tests check pass@1 equals c over n, a known pass@5 value, the edge cases where pass@k is 1 or 0, and that a tiny but wrong design scores below a correct, larger one.

Pitfalls: rewarding area alone, which teaches the model to delete logic; letting the model see or write the testbench, which lets it hack the reward; training and testing on the same specs; and comparing checkpoints on different sample counts.

Alternatives: an LLM judge that reads the RTL is cheap to set up but can be fooled and proves nothing about correctness; formal equivalence checking against a reference proves correctness for every input but needs a reference design and can time out on large modules. Simulation against hidden testbenches works for any spec that has tests, so it is the main gate, with formal checks added where references exist.

Risks: reward hacking shows up as suspicious wins. Inspect the smallest designs by hand, keep a held-out set of specs, and add formal equivalence checks against a reference where one exists.

Numbers: with n=10 and c=3, pass@1 is 0.30 and pass@5 is about 0.92, so report k with every number.

Why this is likely: the listing centers on evaluations and RL for chip design, the OpenAI and Synopsys partnership is about models writing hardware, and interviewing.io reports math-heavy coding at OpenAI. The round is inferred, since no report places an eval design question in a specific round.

</details>

<a id="q11"></a>

#### Q11. Rollout success fell from 34% to 12% after a cluster move. Find the cause

Round: Onsite coding. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Failure classification, infra versus model errors.

The team moved RL rollouts to a new cluster. No model or code changed, but success dropped from 34% to 12%. You get 100 rollout logs. Find the cause and change the pipeline so this cannot hide again.

<details>
<summary>Show answer</summary>

Task: Separate infra failures from model failures in the logs, and stop timeouts from counting as reward 0.

Approach: the model did not change, so suspect the environment first. Classify every failed log into infra classes (timeout, license, out of memory) and model classes (compile error, wrong output) with explicit patterns, then count. Here 22 of 100 logs show `tool timeout after 60s`. The new cluster has slower shared disks, so the simulator often exceeds the 60 second tool timeout, and the environment scores a timeout as reward 0, the same as a wrong design. That trains the policy on noise. Fix two things: raise or adapt the timeout per tool, and make infra failures a separate outcome that is retried and excluded from reward and success rate.

```python
import re
from collections import Counter

RULES = [
    ("infra_timeout", re.compile(r"tool timeout after \d+s|TimeoutExpired")),
    ("infra_license", re.compile(r"license (checkout )?(failed|unavailable)", re.I)),
    ("infra_oom", re.compile(r"Killed|MemoryError|OOM", re.I)),
    ("model_compile", re.compile(r"syntax error|Error-\[", re.I)),
    ("model_wrong", re.compile(r"TEST FAILED|mismatch", re.I)),
]

def classify(log):
    for name, rx in RULES:
        if rx.search(log):
            return name
    return "success" if "TEST PASSED" in log else "unknown"

def success_rate(logs, exclude_infra=True):
    c = Counter(classify(l) for l in logs)
    infra = sum(v for k, v in c.items() if k.startswith("infra_"))
    denom = len(logs) - (infra if exclude_infra else 0)
    return (c["success"] / denom if denom else 0.0), c

if __name__ == "__main__":
    logs = (["TEST PASSED"] * 12 + ["[eda] tool timeout after 60s"] * 22
            + ["TEST FAILED: mismatch at cycle 40"] * 60 + ["syntax error near 'endmodule'"] * 6)
    raw, _ = success_rate(logs, exclude_infra=False)
    clean, c = success_rate(logs)
    assert raw == 0.12 and c["infra_timeout"] == 22
    assert abs(clean - 12 / 78) < 1e-12
    print("q11 ok", round(clean, 3))
```

Numbers: raw success is 12 out of 100. Excluding the 22 infra timeouts gives 12 out of 78, about 15%, which still trails 34%, so check for a second cause that would show up as model-class failures on the new cluster, such as a different simulator version or thread count on the new image that changes results (diff `tool --version` and the container digest between clusters), or a simulation cycle or wall limit that truncates good runs and prints a mismatch. Rerun a sample of the 60 "mismatch" logs on the old cluster to confirm.

Verify: the test checks the class counts and both success rates on the sample.

Pitfalls: blaming the model first; patterns so broad that "Killed" in a design name matches OOM; and an `unknown` class you never read.

Alternatives: asking an LLM to classify each log handles new error text but costs a call per log and can mislabel, which a success metric cannot afford; having the environment return structured outcome codes is the best long-term fix but needs every tool wrapper changed. Regex rules classify 100 logs in milliseconds today, so use them to find the cause, then move to structured codes so the class is set at the source.

Why this is likely: inferred from the listing, which asks you to debug failures across tools, services, and infrastructure, and reliable RL environments for EDA tools are the stated core of the role; no candidate report of this exact format was found (low confidence).

</details>

<a id="q12"></a>

#### Q12. A tool call fails 15% of the time. How many retries does a 40-call rollout need?

Round: Technical screen. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Probability, retries, Monte Carlo checks.

Each EDA tool call in a rollout fails independently with probability p = 0.15 for transient reasons. A rollout makes 40 calls and is wasted if any call fails after its retries. Compute the expected attempts per call, the rollout success rate for 0, 1 and 2 retries, and check the math by simulation.

<details>
<summary>Show answer</summary>

Task: Closed-form and simulated rollout success rates for 0, 1 and 2 retries over 40 calls.

Approach: attempts per call follow a geometric distribution, so the expected number is 1 divided by (1 minus p), about 1.18. With r retries a call fails only if all r plus 1 attempts fail, which has probability p to the power r plus 1. The rollout succeeds if all 40 calls succeed, so the rate is (1 minus p to the r plus 1) to the power 40. Write the closed form, then a seeded Monte Carlo that should agree within sampling error.

```python
import random

def expected_attempts(p):
    return 1 / (1 - p)

def rollout_ok(p, retries, calls):
    return (1 - p ** (retries + 1)) ** calls

def simulate(p, retries, calls, trials=200_000, seed=0):
    rng = random.Random(seed)
    ok = 0
    for _ in range(trials):
        ok += all(any(rng.random() >= p for _ in range(retries + 1)) for _ in range(calls))
    return ok / trials

if __name__ == "__main__":
    p = 0.15
    assert abs(expected_attempts(p) - 1.17647) < 1e-4
    for r in (0, 1, 2):
        exact = rollout_ok(p, r, 40)
        sim = simulate(p, r, 40, trials=20_000)
        print(r, round(exact, 4), round(sim, 4))
        assert abs(exact - sim) < 0.015
    print("q12 ok")
```

Numbers: with no retries a rollout succeeds 0.15% of the time, so almost every rollout is wasted. One retry gives about 40%, two retries about 87%, and three give about 98%. Each retry adds at most 15% more calls on average, so two or three retries are cheap.

Verify: the simulation with 20,000 trials matches the formula within 0.015 for each retry count.

Pitfalls: assuming failures are independent when they come from one overloaded license server, where retries need backoff and jitter; retrying hard errors; and not seeding the simulation.

Alternatives: Monte Carlo alone answers the question but carries sampling error (a standard error of about 0.0035 at 20,000 trials near 40%) and skips the formula the interviewer wants; a Markov chain with correlated failures models a shared overloaded license server, which the independence formula cannot. Use the closed form for the answer, the seeded simulation to check it, and the correlated model if failures cluster.

Why this is likely: interviewing.io reports math coding questions at OpenAI such as expected iterations, and retry budgets are a daily question for RL environments that call flaky tools.

</details>

### System design (no AI)

<a id="q13"></a>

#### Q13. Design an RL environment service around license-limited EDA tools

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Capacity, scheduling, isolation, reward integrity.

Training runs thousands of parallel rollouts. Each rollout step may call a simulator or synthesis tool that needs a license seat. Design the service that hosts these environments.

<details>
<summary>Show answer</summary>

Task: An environment service that schedules scarce license seats and keeps infra errors out of the reward.

Approach: put a gateway between the trainer and the tools. The trainer calls `reset` and `step` over RPC. Environment workers run in containers, hold rollout state, and send tool calls to a scheduler that owns the license pool. The scheduler grants seats by priority (eval before training, short jobs before long ones), queues the rest, and reports queue time as a metric. Results go to a content-addressed cache keyed by design hash, tool version and options, because many rollouts produce the same design. Every response carries an outcome class (ok, model error, infra error), and infra errors are retried and never become reward.

Numbers: 500 seats with a 20 second average call give 25 calls per second. 4096 rollouts that each call a tool every 25 seconds need about 164 calls per second, more than six times capacity. Options: cache hits (measure the hit rate; RL rollouts often regenerate identical designs), cheaper fast checks first (lint and compile take milliseconds and reject many designs), smaller designs, and async rollouts so the trainer does not wait on the slowest tool.

Risks: license server outages stall all training, so add circuit breakers and a degraded mode that uses open-source tools such as Verilator; a cache that ignores the tool version returns wrong rewards; and tools that write to shared disk leak state between rollouts.

Alternatives: run tools inside each worker with a static seat split, which is simpler but wastes seats when load is uneven. Workers could also check out licenses directly from the license server (FlexLM-style) with retries. That needs no scheduler, but it gives no priority between eval and training and turns contention into retry storms at 164 requests per second against 25 per second of capacity. A central scheduler costs one extra service but enforces priority and exposes queue time, so use it.

Verify: load test with a fake tool that sleeps for a sampled duration, and check that seats never exceed the limit.

Why this is likely: the listing asks for RL environments that integrate EDA tools at scale, and OpenAI's system design round focuses on infrastructure.

</details>

<a id="q14"></a>

#### Q14. Design a platform for reproducible, long-running chip design experiments

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Reproducibility, checkpointing, lineage.

Experiments run for days on many GPUs and call EDA tools. Researchers need to rerun any result months later, resume after failures, and compare runs fairly.

<details>
<summary>Show answer</summary>

Task: A platform where every run is reproducible from a manifest and resumes from checkpoints.

Approach: treat every run as a manifest: git commit, container image digest, tool versions, config, data snapshot ids and seeds. The launcher refuses to start a run with a dirty tree or a floating image tag. Runs write checkpoints and metrics to object storage under the run id, and a metadata database stores the lineage from dataset to checkpoint to eval result. A controller watches heartbeats, restarts failed workers from the latest checkpoint, and records every restart, so a result says how it was produced.

Numbers: Young's formula for the checkpoint interval is the square root of 2 times checkpoint cost times mean time between failures. With a 5 minute checkpoint and a job-level MTBF of 562 minutes, the interval is the square root of 5620, about 75 minutes. Checkpointing every 10 minutes would spend a third of the time writing checkpoints; every 6 hours would lose hours per failure.

Risks: EDA tools are often non-deterministic across versions and thread counts, so pin versions and record thread counts; licenses expire, so old runs may not be replayable with the same tool, which you should state in the manifest; and storage costs grow with checkpoints, so keep a retention policy.

Alternatives: a managed experiment tracker for metrics with your own lineage store, versus building both. A common split is a hosted tracker for metrics and an in-house lineage store, because lineage depends on your own tools and storage.

Verify: a nightly job reruns a small fixed experiment and fails if results drift beyond a set tolerance.

Why this is likely: the listing asks for reliable infrastructure and reproducible results for long experiments, and a Glassdoor report from October 2026 describes an infrastructure design round at OpenAI.

</details>

<a id="q15"></a>

#### Q15. Design a sandbox that runs model-generated RTL and scripts safely

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Isolation, resource limits, throughput.

Models write Verilog, testbenches and Python or Tcl scripts that drive tools. Run thousands of these per minute without letting any of them read secrets, reach the network, or harm other jobs.

<details>
<summary>Show answer</summary>

Task: An isolated, resource-limited sandbox for untrusted generated code and scripts.

Approach: treat all generated code as hostile. Run each job in an isolated sandbox with no network, a read-only base image, a small writable scratch directory, and limits on CPU, memory, processes, disk and wall time. License servers are the one network exception, through a proxy that allows only the license port. Copy inputs into the sandbox before the job starts and copy results out when it ends, so the job sees no host paths. Keep the hidden testbench outside the sandbox, or mount it read-only, so the model cannot edit what grades it.

Alternatives: plain containers share the host kernel and are the fastest, but a kernel bug is an escape path. gVisor intercepts system calls in user space, which adds overhead on file-heavy tools. Firecracker microVMs give each job its own kernel and the project reports start times around 125 ms. Use microVMs or gVisor for code that runs scripts, and pooled warm sandboxes to hide start time.

Numbers: at 5000 jobs per minute with a 20 second average run, about 1700 sandboxes run at once. A 125 ms start is under 1% of a 20 second job.

Risks: tools that need a writable home directory or GUI libraries; scripts that fork bombs or fill disk; and timing side channels, which matter less here than secret access.

Verify: a red-team suite of known bad scripts (network calls, reading `/etc`, fork bombs, writing outside scratch) runs on every image change and must fail every attempt.

Pitfalls: mounting the host Docker socket; sharing a scratch directory between jobs; and trusting a log line as proof of success.

Why this is likely: the listing mentions containers and safe execution, and RL on generated code needs a sandbox at scale. A 2026 Reddit r/Backend post reports a Colab/Replit-style sandboxed execution design at the OpenAI onsite (low confidence).

</details>

### Behavioral (no AI)

<a id="q16"></a>

#### Q16. How do you use AI coding agents, and how do you check what they produce?

Round: Hiring manager. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI tool use, judgment, ownership.

The interviewer wants a concrete picture of your daily workflow with AI tools, a time you caught one being wrong, and how you decide what to trust. Answer with a real story in the situation, action, result shape.

<details>
<summary>Show answer</summary>

Task: A concrete workflow with AI agents, one mistake you caught, and the checks you rely on.

Approach: start with how you split work. Use agents for scaffolding, tests, refactors with a clear spec, and reading unfamiliar code, and do the design, interfaces and tricky invariants yourself. Then explain your checks: small steps, a plan before prompting, reading every diff, running tests you wrote or reviewed, and a separate look at any change to tests or fixtures. Then tell the story. For example: an agent made a failing integration test pass by changing the expected PPA value in a shared fixture. You caught it in review because the test diff was separate, reverted it, found the real bug (a unit conversion from picoseconds to nanoseconds), and added a guard that fails if fixture files change without a reviewer's sign-off. Close with the result and what you changed in your process.

Pitfalls: claiming you never use AI tools, which sounds out of touch for this team; claiming you trust them fully; and a story with no number or result.

Verify: practice the story out loud in under three minutes, and make sure it names one specific mistake, how you found it, and what you changed afterwards.

Why this is likely: the agentic coding round is graded on validating AI output (Blind, October 2026), and OpenAI's interview guide says AI tool expectations vary by interview; no report confirms the hiring manager asks this directly (low to medium confidence).

</details>

## Prep checklist

- [ ] Build a priority queue with retries, cancel and stable ties from memory, with tests, in under 30 minutes.
- [ ] Practice multi-part problems where requirements change mid-session: ledgers with expiry, versioned stores, schedulers.
- [ ] Solve Rotting Oranges and its threshold variant without help and explain the complexity.
- [ ] Practice reading an unfamiliar Python repo for 10 minutes, then fixing two planted bugs with tests.
- [ ] Run one timed session with an AI coding agent on an existing repo, and practice reading test diffs separately.
- [ ] Review subprocess code for shell injection, invented arguments, missing return code checks and timeouts.
- [ ] Learn the basics of EDA flows: lint, simulation, synthesis, place and route, WNS, TNS and PPA.
- [ ] Know pass@k, gated rewards and common reward hacking in code generation evals.
- [ ] Do retry and expected-value math by hand and check it with a seeded simulation.
- [ ] Prepare designs for an environment service, an experiment platform and a sandbox, each with capacity math.
- [ ] Prepare a project deep dive with slides, decisions, numbers and what you would change.
- [ ] Prepare one story about using and checking AI tools, with an error you caught.

## Sources

- OpenAI interview guide: https://openai.com/interview-guide/
- OpenAI and Synopsys partnership announcement: https://news.synopsys.com/2026-09-30-OpenAI-and-Synopsys-Announce-GPT-Synopsys-Frontier-Intelligence-to-Revolutionize-Chip-Design
- Interview with Richard Ho on OpenAI hardware: https://thedataexchange.media/richard-ho-openai
- Blind, OpenAI agentic coding interview: https://www.teamblind.com/post/openai-agentic-code-interview-up8jhuiq
- Blind, additional coding round after onsite: https://www.teamblind.com/post/additional-coding-interview-for-openai-after-onsite-k4gazyyx
- Glassdoor, OpenAI software engineer interviews: https://www.glassdoor.com/Interview/OpenAI-Software-Engineer-Interview-Questions-EI_IE2210885.0,6_KO7,24.htm
- interviewing.io, OpenAI interview questions: https://interviewing.io/openai-interview-questions
- Hello Interview, OpenAI L5 guide: https://www.hellointerview.com/guides/openai/l5
- 1point3acres, OpenAI 2026 question roundup: https://www.1point3acres.com/bbs/thread-1190155-1-1.html
- PracHub, OpenAI software engineer interview guide: https://prachub.com/interview-guide/openai-software-engineer-interview-guide
- prepare.sh, OpenAI interview process 2026: https://prepare.sh/articles/openai-interview-process-2026
- Exponent, interviewing at OpenAI in 2026: https://medium.com/exponent/what-its-actually-like-to-interview-at-openai-in-2026-03a646c9436c
- Reddit, OpenAI SWE interviews are more practical: https://www.reddit.com/r/OfferEngineering/comments/1w27am0/openai_swe_interviews_seem_much_more_practical/
- Reddit, OpenAI interview process megathread: https://www.reddit.com/r/Hack2Hire/comments/1tqdmyr/openai_interview_process_experience_megathread/
- Reddit, OpenAI interview question 2026: https://www.reddit.com/r/Backend/comments/1ricae4/open_ai_interview_question_2026_with_solution
- AI in interviews overview: https://thita.ai/ai-in-interviews
