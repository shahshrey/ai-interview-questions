# Netflix: Machine Learning Engineer (L4/L5), Studio Media Algorithms

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Netflix publishes no fixed loop. Prep guides and candidate reports agree that each team designs its own rounds. The team builds ML, computer vision and graphics models plus artist-facing tools that analyze, create and transform media assets. Its engineers ship reusable frameworks, distributed GPU training and production deployment, and they work with VFX and content tools such as Nuke, Houdini, Maya and Unreal Engine. The loop below is built from 2026 reports on Netflix MLE, Research Engineer and SWE loops. Confirm the exact rounds with your recruiter.

1. Recruiter call. About 30 minutes, virtual. Role, level and team fit. AI: not relevant.
2. Hiring manager screen. About 30 minutes, virtual. A two-way conversation about your past work and the team. AI: not relevant.
3. Technical phone screen. 45 to 60 minutes in a shared editor, and the format depends on the team. One 2026 report had a timed cache with memory-pressure follow-ups. Another had 30 minutes of ML fundamentals, then 30 minutes reviewing a feature-pipeline pull request whose tests already pass. Some MLE candidates take an online assessment first. AI: unknown. Ask the recruiter.
4. Onsite coding. 45 to 60 minutes, virtual. LeetCode medium problems with a practical twist, or problems the team wrote itself (undo and redo, a timed cache, a time-based store). AI: unknown. Reports do not mention it, so practice without it.
5. AI-assisted coding round, in some loops. Length not reported; plan for about 60 minutes like other rounds. Virtual, with an interviewer watching. An AI assistant in the editor can rewrite your code inline. You are graded on how you prompt, review and test its output, and on when you push back. AI: provided (an in-editor assistant).
6. ML system design and domain depth. 45 to 60 minutes, virtual. Netflix gives system design the most weight. For this team, expect media pipelines: shot-level features and embeddings, GPU training, batch inference on new assets, and tools artists use. Interviewers ask what happens after the offline metric improves. AI: not relevant (a discussion), and no report mentions AI tools here.
7. Past-project deep dive. About 45 minutes, virtual. Walk through the biggest system you built, how it scaled and what you would change. For this role, an ML tool you deployed to artists or creators. AI: not relevant.
8. Director and culture rounds, usually two. 45 minutes each, virtual. One director is from the hiring org and one from a partner org. They cover giving and receiving feedback, cross-team conflict, judgment, and how you work with AI. AI: not relevant.

AI tools: in a July 2026 podcast, Netflix's Chief Product and Technology Officer said coding interviews now let candidates use AI tools, as press summaries reported. A July 2026 Blind thread describes an onsite round marked "AI assisted", where a Netflix-verified commenter said candidates are judged on when and how they push back on the AI. Netflix has published no written rule, and the August 2026 coding round reports do not mention AI. So questions Q4 to Q7 are labelled Provided (in-editor assistant), as in Netflix's AI-assisted round. Phone screen and classic coding questions are labelled Unknown. Domain-depth questions that include code (Q8 to Q11) are also labelled Unknown. Design, deep-dive and behavioral questions (Q12 to Q17) are discussion rounds with no coding, so the AI policy is labelled Unknown.

Classic algorithm questions are limited to two (Q1 and Q2), because 2026 reports for Netflix MLE and SWE loops still include a cache or undo and redo problem at the phone screen or onsite coding round.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a frame-thumbnail cache with a per-entry time limit and a hard memory cap | Classic algorithm | Phone screen | Unknown | Medium |
| [Q2](#q2) | Build undo and redo for a mask-painting tool, where one brush stroke undoes as one step | Classic algorithm | Onsite coding | Unknown | Medium |
| [Q3](#q3) | Review this per-shot feature pull request. Its tests pass. What breaks in production? | Review | Phone screen | Unknown | Medium |
| [Q4](#q4) | Add a memoized per-shot embedding feature to this media feature package | Build | AI-assisted coding | Provided (in-editor assistant) | Hard |
| [Q5](#q5) | The AI assistant wrote this frame sampler and proxy helper. Review it before it ships | Review | AI-assisted coding | Provided (in-editor assistant) | Medium |
| [Q6](#q6) | This shot boundary detector fails three tests on flashes, fades and cut positions. Fix it | Debug | AI-assisted coding | Provided (in-editor assistant) | Medium |
| [Q7](#q7) | Embedding inference keeps the GPU 23% busy. Read the profile and make it faster | Optimize | AI-assisted coding | Provided (in-editor assistant) | Hard |
| [Q8](#q8) | A 16-GPU training job hangs at epoch end and spikes in loss early. Read the logs, find... | Trace debug | Domain depth | Unknown | Hard |
| [Q9](#q9) | Fine-tune a CLIP-style text-to-shot model for in-video search. Write the loss and recall@k | Concept plus code | Domain depth | Unknown | Hard |
| [Q10](#q10) | Size attention memory for a transformer over every shot in a two-hour title | Concept plus code | Domain depth | Unknown | Medium |
| [Q11](#q11) | A generative relighting model (like the tools Netflix acquired with InterPositive) nears compositors. Design the evaluation | Concept plus code | Domain depth | Unknown | Hard |
| [Q12](#q12) | Your new match-cut ranker beats the old one offline. What happens before it ships? | Design | ML system design | Unknown | Medium |
| [Q13](#q13) | Design match-cut candidate retrieval over every shot in a season, then the catalog | Design | ML system design | Unknown | Hard |
| [Q14](#q14) | Design batch inference of shot embeddings for every new title, with GPU-hour math | Design | ML system design | Unknown | Hard |
| [Q15](#q15) | Design an ML matte tool that runs inside Nuke for compositors | Design | ML system design | Unknown | Hard |
| [Q16](#q16) | Walk through an ML tool you shipped to artists, then redesign it for 10x the users | Design | Past-project deep dive | Unknown | Medium |
| [Q17](#q17) | Tell me about a time an AI assistant gave you wrong code or analysis. How did you catch it? | Behavioral | Director and culture | Unknown | Medium |

### Phone screen and onsite coding (AI unknown, practice without it)

<a id="q1"></a>

#### Q1. Build a frame-thumbnail cache for a review tool: each entry has its own time limit, the cache holds at most N live entries, and `count()` returns only live ones

Round: Phone screen. AI policy: Unknown. Type: Classic algorithm. Difficulty: Medium. Topic: Hashing, heaps, LRU eviction.
Related practice: [Cache With Time Limit](https://leetcode.com/problems/cache-with-time-limit/) (Medium), [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) (Medium), [LRU Cache](https://leetcode.com/problems/lru-cache/) (Medium).

<details>
<summary>Show answer</summary>

Task: implement `set(key, value, ttl)`, which returns True if it overwrote a live key, `get(key)`, which returns -1 when the key is missing or expired, and `count()`. When the cache is over N items, evict the least recently used live entry. The interviewer adds memory pressure as a follow-up, so plan for it from the start.

What they grade: correct expiry at the boundary, amortized cost per call, a clean way to make time testable, and how you deal with stale heap entries.

Approach: keep an `OrderedDict` from key to (value, expires_at) for LRU order, and a min-heap of (expires_at, key) for expiry. Every call first pops heap entries whose time has passed. A popped entry deletes the key only if its expiry still matches the stored one, because an overwrite leaves an old heap entry behind. Eviction for size uses the LRU end of the dict. Inject the clock so tests control time instead of sleeping.

Stale heap entries grow with overwrites, so rebuild the heap when it is more than twice the live size. That keeps memory O(N) and the rebuild cost amortized O(1) per write.

Code:

```python
import heapq
from collections import OrderedDict


class TimedCache:
    def __init__(self, max_items, clock):
        self.max_items, self.clock = max_items, clock
        self.data = OrderedDict()  # key -> (value, expires_at)
        self.heap = []             # (expires_at, key), may hold stale entries

    def _purge(self):
        now = self.clock()
        while self.heap and self.heap[0][0] <= now:
            exp, key = heapq.heappop(self.heap)
            item = self.data.get(key)
            if item is not None and item[1] == exp:
                del self.data[key]

    def _compact(self):
        if len(self.heap) > 2 * len(self.data) + 16:
            self.heap = [(exp, k) for k, (_, exp) in self.data.items()]
            heapq.heapify(self.heap)

    def set(self, key, value, ttl):
        self._purge()
        existed = key in self.data
        exp = self.clock() + ttl
        self.data[key] = (value, exp)
        self.data.move_to_end(key)
        heapq.heappush(self.heap, (exp, key))
        while len(self.data) > self.max_items:
            self.data.popitem(last=False)
        self._compact()
        return existed

    def get(self, key):
        self._purge()
        item = self.data.get(key)
        if item is None:
            return -1
        self.data.move_to_end(key)
        return item[0]

    def count(self):
        self._purge()
        return len(self.data)


t = [0]
c = TimedCache(2, lambda: t[0])
assert c.set("a", 1, 10) is False
assert c.set("a", 2, 5) is True          # overwrite shortens the time limit
t[0] = 5
assert c.get("a") == -1 and c.count() == 0  # expiry is exclusive at exactly ttl
c.set("x", 1, 100); c.set("y", 2, 100)
assert c.get("x") == 1                   # x is now most recent
c.set("z", 3, 100)
assert c.get("y") == -1 and c.get("x") == 1 and c.count() == 2
for i in range(1000):
    c.set("x", i, 100)
assert len(c.heap) <= 2 * len(c.data) + 17
print("q1 ok")
```

Verify: an overwrite that shortens the time limit, expiry exactly at the boundary, LRU order after a `get`, and a 1,000-overwrite loop that proves the heap stays bounded.

Pitfalls:
- Using `time.time()` inside the class, which makes tests sleep and flake.
- Deleting a key when an old heap entry pops, which drops a fresh value.
- Evicting by expiry when the spec says LRU, or the reverse. Ask which one.

Follow-ups: thread safety (one lock, or shard by key hash). Capping by bytes instead of entries (track size per entry, and evict until under budget). A thumbnail cache shared across processes (Redis with per-key expiry, at the cost of a network hop).

Why this is likely: an August 2026 Netflix SWE phone screen report describes a timed cache with memory-pressure follow-ups, and Cache With Time Limit tops the crowd-reported Netflix six-month LeetCode list; medium confidence for this team.

</details>

<a id="q2"></a>

#### Q2. Build undo and redo for a mask-painting tool, where a whole brush stroke undoes as one step and history is capped at 100 steps

Round: Onsite coding. AI policy: Unknown. Type: Classic algorithm. Difficulty: Medium. Topic: Command pattern, stacks.
Related practice: [Design Browser History](https://leetcode.com/problems/design-browser-history/) (Medium).

<details>
<summary>Show answer</summary>

Task: a matte artist paints a mask. Each dab sets some pixels. Build `History` with `do(cmd)`, `undo()`, `redo()`, and `begin()` and `end()` to group a stroke. A new edit after an undo clears redo. Memory is bounded.

What they grade: the command abstraction, correct redo invalidation, grouping, the bound, and tests you write without being asked.

Approach: use the command pattern. Each command applies itself and remembers what it overwrote, so `revert` restores the exact old pixels. A `Group` applies its children in order and reverts them in reverse order. That order matters when two dabs in one stroke touch the same pixel. `History` keeps an undo deque with `maxlen` (the oldest step drops in O(1)) and a redo list. Every new top-level command clears redo, because the old future no longer applies.

The alternative is snapshots: copy the whole mask per step. That is simpler and safe, but a 4K mask is about 8 MB as one byte per pixel, so 100 steps cost about 800 MB. Commands store only changed pixels, so a stroke over 2,000 pixels costs kilobytes. Commands are the better fit for brush work, and snapshots suit edits that touch most of the image.

Code:

```python
from collections import deque


class SetPixels:
    def __init__(self, mask, changes):
        self.mask, self.changes, self.old = mask, dict(changes), None

    def apply(self):
        self.old = {p: self.mask.get(p, 0) for p in self.changes}
        self.mask.update(self.changes)

    def revert(self):
        for p, v in self.old.items():
            if v == 0:
                self.mask.pop(p, None)
            else:
                self.mask[p] = v


class Group:
    def __init__(self):
        self.cmds = []

    def apply(self):
        for c in self.cmds:
            c.apply()

    def revert(self):
        for c in reversed(self.cmds):
            c.revert()


class History:
    def __init__(self, limit=100):
        self.undo_stack, self.redo_stack, self.group = deque(maxlen=limit), [], None

    def do(self, cmd):
        cmd.apply()
        if self.group is not None:
            self.group.cmds.append(cmd)
        else:
            self.undo_stack.append(cmd)
            self.redo_stack.clear()

    def begin(self):
        self.group = Group()

    def end(self):
        g, self.group = self.group, None
        if g and g.cmds:
            self.undo_stack.append(g)
            self.redo_stack.clear()

    def undo(self):
        if not self.undo_stack:
            return False
        c = self.undo_stack.pop()
        c.revert()
        self.redo_stack.append(c)
        return True

    def redo(self):
        if not self.redo_stack:
            return False
        c = self.redo_stack.pop()
        c.apply()
        self.undo_stack.append(c)
        return True


mask, h = {}, History(limit=3)
h.begin()
h.do(SetPixels(mask, {(0, 0): 255}))
h.do(SetPixels(mask, {(0, 0): 128, (0, 1): 255}))  # overlaps the first dab
h.end()
assert mask == {(0, 0): 128, (0, 1): 255}
assert h.undo() and mask == {}          # whole stroke reverts in one step
assert h.redo() and mask == {(0, 0): 128, (0, 1): 255}
h.undo()
h.do(SetPixels(mask, {(5, 5): 9}))
assert h.redo() is False                 # new edit cleared redo
for i in range(5):
    h.do(SetPixels(mask, {(9, i): 1}))
assert len(h.undo_stack) == 3            # capped
assert History().undo() is False and History().redo() is False
print("q2 ok")
```

Verify: overlapping dabs inside one stroke, undo then redo round trip, redo cleared by a new edit, the cap, and undo on an empty history returning False.

Pitfalls:
- Reverting a group in forward order, which leaves the middle value of overlapping dabs.
- Recording old values when the command is built instead of when it is applied, which breaks redo after other edits.
- A plain list with `pop(0)` for the cap, which is O(n) per edit.

Follow-ups: merge tiny dabs into one command to save memory. Persist history across sessions. Collaborative editing, where undo means "undo my last change" while others keep editing.

Why this is likely: an August 2026 Netflix SWE onsite report describes an undo and redo coding problem, and this role builds artist-facing tools where undo is a core feature; medium confidence for this team.

</details>

<a id="q3"></a>

#### Q3. Review this pull request that adds a per-shot brightness feature. Its tests pass. What breaks in production, and what do you ask the author to change?

Round: Phone screen. AI policy: Unknown. Type: Review. Difficulty: Medium. Topic: Media feature pipelines, frame math, caching.

```diff
--- /dev/null
+++ b/media_features/shot_stats.py
+CACHE = {}
+
+def shot_brightness(asset, shots, fps=24, cache=CACHE):
+    """Mean luma per shot. shots: list of (start_s, end_s) in seconds."""
+    key = asset.title_id
+    if key in cache:
+        return cache[key]
+    frames = asset.decode_all()               # list of 2D luma arrays
+    out = []
+    for start_s, end_s in shots:
+        a, b = int(start_s * fps), int(end_s * fps)
+        seg = frames[a:b]
+        out.append(sum(f.mean() for f in seg) / len(seg) if seg else 0.0)
+    cache[key] = out
+    return out
+
+def test_shot_brightness():
+    asset = FakeAsset(title_id="t1", frames=[np.full((2, 2), i) for i in range(10)])
+    assert shot_brightness(asset, [(0, 0.2), (0.2, 0.4)], fps=10) == [0.5, 2.5]
```

<details>
<summary>Show answer</summary>

Task: find what the 10-frame test cannot catch, rank it by impact, and propose a fix with tests.

What they grade: finding the issues a real title exposes, ranking them by impact, and suggesting concrete tests.

Approach: think about what a real title looks like. It is two hours long, 23.976 frames per second, re-delivered after a conform, and processed by many workers. Then walk the diff with those inputs in mind.

1. The cache key is only `title_id`. A re-delivered asset or a new shot list returns stale features without any error. The key needs the asset version and a hash of the shots and the code version.
2. `fps=24` is hard-coded. Netflix takes the native frame rate (23.976, 24, 25, 29.97 and others), and 23.976, which is 24000/1001, is common. After one hour, index math at 24 lands 86 frames, about 3.6 seconds, late, so every shot reads the wrong pixels. Take fps from the asset as an exact fraction.
3. `int()` truncates float error. `int(0.29 * 100)` is 28, not 29, so shots lose a frame now and then. Round to the nearest frame instead.
4. `decode_all()` holds the whole title in memory. Two hours at 24 fps is 172,800 frames. As 1080p luma that is about 2 MB each, or roughly 350 GB. Stream the frames once.
5. An empty shot returns 0.0, a real-looking value that hides bugs downstream. Return None and count how often it happens.
6. A module-level cache dict is unbounded and shared across tests and threads. Inject a store.

Code:

```python
from fractions import Fraction


def frame_index(t_s, fps):
    return int(round(Fraction(t_s).limit_denominator(10**6) * fps))


def shot_brightness(asset, shots, store, code_version="v2"):
    key = (asset.title_id, asset.version, code_version, tuple(shots))
    if key in store:
        return store[key]
    bounds = [(frame_index(s, asset.fps), frame_index(e, asset.fps)) for s, e in shots]
    sums, counts = [0.0] * len(shots), [0] * len(shots)
    k = 0
    for i, mean_luma in asset.stream_frame_means():   # one decode pass, constant memory
        while k < len(bounds) and i >= bounds[k][1]:
            k += 1
        if k == len(bounds):
            break
        if i >= bounds[k][0]:
            sums[k] += mean_luma
            counts[k] += 1
    out = [s / c if c else None for s, c in zip(sums, counts)]
    store[key] = out
    return out


class FakeAsset:
    def __init__(self, n, fps, version=1):
        self.title_id, self.version, self.fps, self.n, self.decodes = "t1", version, fps, n, 0

    def stream_frame_means(self):
        self.decodes += 1
        return ((i, float(i)) for i in range(self.n))


a, store = FakeAsset(40, Fraction(100)), {}
assert shot_brightness(a, [(0, 0.29), (0.29, 0.29)], store) == [14.0, None]
assert frame_index(3600, Fraction(24000, 1001)) == 86314
shot_brightness(a, [(0, 0.29), (0.29, 0.29)], store)
assert a.decodes == 1                                  # cache hit
shot_brightness(FakeAsset(40, Fraction(100), version=2), [(0, 0.29)], store)
assert len(store) == 2                                 # new version, new entry
print("q3 ok")
```

Verify: a fixture at 24000/1001 fps that checks a timestamp an hour in, the 0.29 second boundary, an empty shot, and a re-delivered version that must miss the cache.

Pitfalls:
- Opening with naming and docstrings while the stale-cache bug goes unmentioned.
- Saying "use a better cache" without naming the missing key parts.

Follow-ups: what if shots overlap, as with handles around a cut? How do you backfill the features for the whole catalog after this fix?

Why this is likely: a Netflix Research Engineer phone screen report from about October 2026 describes reviewing a feature-pipeline pull request whose tests already pass, and Netflix's media ML blog describes features memoized per media asset; medium confidence.

</details>

### AI-assisted coding round (AI assistant allowed, as at Netflix)

<a id="q4"></a>

#### Q4. In this media feature package, add a `ShotEmbedding` feature that stores one embedding per shot and recomputes only the shots that changed when shot boundaries move or the encoder version changes

Round: AI-assisted coding. AI policy: Provided (in-editor assistant). Type: Build. Difficulty: Hard. Topic: Multi-file feature work, memoization keys.

```text
media_features/
  __init__.py
  base.py              # class Feature: name, version, compute(ctx); run() reads and writes the store
  registry.py          # FEATURES = {"luma_hist": LumaHist}; register(cls)
  store.py             # FeatureStore.get(key), put(key, value); S3-backed in prod, dict in tests
  features/
    luma_hist.py       # existing per-asset feature, uses Feature.run()
    shot_boundaries.py # returns [(start_frame, end_frame), ...], end exclusive
  models/
    clip_encoder.py    # encode(frames) -> list[float]; MODEL_VERSION = "clip-v3"
  tests/
    test_luma_hist.py
    test_store.py
```

<details>
<summary>Show answer</summary>

Task: add `features/shot_embedding.py`, register it, and add tests. Re-running on the same asset must not call the encoder again. Moving one boundary must re-encode only the shots it touched.

What they grade: reading the existing code before prompting, a small delegation plan, a memo key that is right, tests that count encoder calls, and catching the assistant's mistakes.

Approach: first read `base.py`, `store.py` and one existing feature yourself. That takes two minutes and lets you check the AI against the real interfaces. Then give the assistant one scoped prompt: "Add ShotEmbedding in features/shot_embedding.py following luma_hist.py. Key the store per shot by (asset_id, asset_version, start_frame, end_frame, MODEL_VERSION). Register it. Use only the existing dependencies. Write pytest tests with a fake encoder that counts calls." Ask for the diff.

What to check in the diff: the key contains every input that changes the output. Registration follows the existing pattern. State lives in the store passed to the feature. Tests assert call counts as well as shapes.

The assistant got this wrong twice. Its first draft decorated `compute` with `functools.lru_cache`. That caches in one process only, keyed by the context object's identity. It never hits across workers, and it leaks memory. A second draft keyed the store by `asset_id` alone, the same stale-data bug as Q3. I rejected both and kept a per-shot key.

Per-shot keys beat one key per asset. A one-frame boundary move re-encodes 2 shots instead of about 2,400.

Code:

```python
MODEL_VERSION = "clip-v3"


class FeatureStore(dict):
    pass


class FakeEncoder:
    def __init__(self):
        self.calls = 0

    def encode(self, frames):
        self.calls += 1
        return [sum(frames) / len(frames)]


class ShotEmbedding:
    name, version = "shot_embedding", 1

    def __init__(self, store, encoder):
        self.store, self.encoder = store, encoder

    def key(self, asset, shot):
        return (self.name, self.version, asset["id"], asset["version"], shot[0], shot[1], MODEL_VERSION)

    def run(self, asset, shots):
        out = []
        for shot in shots:
            k = self.key(asset, shot)
            if k not in self.store:
                frames = asset["frames"][shot[0]:shot[1]]
                self.store[k] = self.encoder.encode(frames) if frames else None
            out.append(self.store[k])
        return out


FEATURES = {}


def register(cls):
    FEATURES[cls.name] = cls
    return cls


register(ShotEmbedding)
asset = {"id": "t1", "version": 3, "frames": list(range(100))}
store, enc = FeatureStore(), FakeEncoder()
f = ShotEmbedding(store, enc)
f.run(asset, [(0, 40), (40, 70), (70, 100)])
assert enc.calls == 3
f.run(asset, [(0, 40), (40, 70), (70, 100)])
assert enc.calls == 3                       # memoized
f.run(asset, [(0, 40), (40, 72), (72, 100)])
assert enc.calls == 5                       # only the two moved shots
f.run(dict(asset, version=4), [(0, 40)])
assert enc.calls == 6                       # new asset version misses
assert "shot_embedding" in FEATURES
print("q4 ok")
```

Verify: run the existing test suite plus the new tests, and assert that the luma feature's call counts are unchanged.

Pitfalls:
- Accepting a large diff that also "cleans up" `base.py`.
- Tests that pass with memoization deleted.

Follow-ups: store vectors as float16 to halve storage. Garbage-collect entries for old model versions.

Why this is likely: a July 2026 Blind thread reports a Netflix onsite round marked "AI assisted", Netflix's CPTO said coding interviews now allow AI tools, and Netflix's media ML blog describes features memoized per media entity; medium confidence.

</details>

<a id="q5"></a>

#### Q5. The AI assistant wrote this frame sampler and proxy helper for a dataset job. Review it, list every defect, and fix it

Round: AI-assisted coding. AI policy: Provided (in-editor assistant). Type: Review. Difficulty: Medium. Topic: Reviewing AI output, video I/O, input safety.

```python
import subprocess
import cv2

def sample_frames(asset_path, shot, n=8):
    """Return n evenly spaced frames from shot=(start_frame, end_frame), end exclusive."""
    cap = cv2.VideoCapture(asset_path)
    step = (shot[1] - shot[0]) // n
    frames = []
    for i in range(n + 1):
        cap.seek(shot[0] + i * step)
        ok, frame = cap.read()
        frames.append(frame)
    return frames

def make_proxy(asset_id, out_dir):
    src = f"/mnt/media/{asset_id}.mov"
    subprocess.run(f"ffmpeg -i {src} -vf scale=640:-2 {out_dir}/{asset_id}.mp4", shell=True)
```

<details>
<summary>Show answer</summary>

Task: find the planted defects, explain the impact of each, and ship a version with tests that need neither OpenCV nor ffmpeg.

What they grade: catching the API the assistant made up, the off-by-one error, the edge cases and the security hole, plus how you steer the assistant to fix them.

Approach: read the code line by line before asking the AI anything. The defects, in order of severity:

1. `make_proxy` passes `asset_id` into a shell string with `shell=True`. An id such as `x; rm -rf /tmp` runs a command, and `../` reaches other paths. Validate the id against a strict pattern and pass an argument list with no shell.
2. `cap.seek` does not exist on `cv2.VideoCapture`. The assistant invented it. The real call is `cap.set(cv2.CAP_PROP_POS_FRAMES, idx)`.
3. `range(n + 1)` returns 9 frames, and the last index can equal `end_frame`, which is the first frame of the next shot.
4. When the shot is shorter than `n` frames, `step` is 0 and every frame is the same frame.
5. `ok` is ignored, so failed reads add `None` and crash the batch later. The capture is never released, and the ffmpeg return code is never checked.

I move the index math into a pure function so it is testable. Each sample sits at the center of one of n equal bins, which avoids both edges. The alternative, `linspace` including the ends, hits the boundary frames, which often carry blur from the cut or dissolve.

My prompt to the assistant: "Rewrite sample_frames using sample_indices below and cap.set(cv2.CAP_PROP_POS_FRAMES, ...). Skip frames where read fails and return the indices actually read. Release the capture in finally." Then I check its diff for any new invented calls.

Code:

```python
import re

ASSET_ID = re.compile(r"[A-Za-z0-9_]{1,64}")


def sample_indices(start, end, n):
    length = end - start
    if length <= 0:
        return []
    if length <= n:
        return list(range(start, end))
    return [start + (2 * k + 1) * length // (2 * n) for k in range(n)]


def proxy_cmd(asset_id, out_dir):
    if not ASSET_ID.fullmatch(asset_id):
        raise ValueError(f"bad asset id {asset_id!r}")
    return ["ffmpeg", "-nostdin", "-y", "-i", f"/mnt/media/{asset_id}.mov",
            "-vf", "scale=640:-2", f"{out_dir}/{asset_id}.mp4"]


idx = sample_indices(100, 180, 8)
assert len(idx) == 8 and len(set(idx)) == 8 and min(idx) > 100 and max(idx) < 180
assert sample_indices(0, 5, 8) == [0, 1, 2, 3, 4]
assert sample_indices(10, 10, 8) == []
for bad in ["x; rm -rf /tmp", "../etc/passwd", "a b", ""]:
    try:
        proxy_cmd(bad, "/out")
        raise AssertionError(bad)
    except ValueError:
        pass
assert proxy_cmd("title_123", "/out")[4] == "/mnt/media/title_123.mov"
print("q5 ok")
```

Verify: bins on a long shot, a shot shorter than n, an empty shot, and injection strings. In the real job, run `subprocess.run(cmd, check=True, timeout=600)`.

Pitfalls:
- Fixing `seek` and stopping there.
- Trusting the assistant's claim that it "tested" the code.

Follow-ups: frame-accurate seeking is slow with long-GOP codecs. Would you decode sequentially instead?

Why this is likely: a Netflix-verified commenter on a July 2026 Blind thread said candidates in the AI-assisted round are "judged on when and how you push back" on the AI; medium confidence.

</details>

<a id="q6"></a>

#### Q6. This shot boundary detector fails three tests. Read the failures, find the causes and fix it

Round: AI-assisted coding. AI policy: Provided (in-editor assistant). Type: Debug. Difficulty: Medium. Topic: Shot boundary detection, flashes and fades.

```text
def detect_cuts(hists, thresh=0.18):
    cuts = []
    for i in range(1, len(hists)):
        if hist_diff(hists[i - 1], hists[i]) > thresh:
            cuts.append(i - 1)
    return cuts

$ pytest tests/test_shot_boundaries.py -q
FFF
FAILED test_hard_cut         - assert [2] == [3]
FAILED test_flash_not_a_cut  - assert [4, 5] == []
FAILED test_fade_is_not_a_cut - assert [0, 1, 2, 3, 4, 5] == []
3 failed in 0.04s
```

<details>
<summary>Show answer</summary>

Task: frames arrive as normalized 4-bin luma histograms, and `hist_diff` is half the L1 distance, from 0 to 1. A cut must be reported as the index of the first frame of the new shot. A camera flash is not a cut. A fade or dissolve is a gradual transition, which a separate detector handles, so it must not produce hard cuts.

What they grade: reading the failures before changing code, one cause per failure, and fixes that do not overfit the three tests.

Approach: each failure points to a different cause.

1. `[2] == [3]`: the code appends `i - 1`, the last frame of the old shot. The contract says first frame of the new shot. That is an off-by-one, so append `i`.
2. `[4, 5]`: a flash makes two big jumps, into the white frame and out of it. The fix skips a jump when the picture returns to the frame before it within two frames. Also skip if the current frame matches the frame two back, which is the jump out of a flash.
3. Fades: each step changes the histogram by 0.2 to 0.27, above 0.18. A threshold that low turns every gradual change into a cut. A hard cut here changes it by 0.8. Raise the threshold to 0.5 and add a minimum shot length of 3 frames, so one bad frame cannot create two cuts.

Delegation: I gave the assistant the failing output and the contract and asked for the smallest diff. Its first answer only raised the threshold to 0.95. That passes the fade test but misses real cuts between similar-looking shots, so I rejected it. A fixed threshold also fails across dark and bright titles. An adaptive threshold, the diff divided by a rolling median, is a good follow-up.

Code:

```python
def hist_diff(a, b):
    return sum(abs(x - y) for x, y in zip(a, b)) / 2


def detect_cuts(hists, thresh=0.5, min_len=3, flash_window=2):
    cuts = []
    for i in range(1, len(hists)):
        if hist_diff(hists[i - 1], hists[i]) < thresh:
            continue
        ahead = range(i + 1, min(i + 1 + flash_window, len(hists)))
        if any(hist_diff(hists[i - 1], hists[j]) < thresh / 2 for j in ahead):
            continue                                  # flash: picture comes back
        behind = range(max(0, i - 1 - flash_window), i - 1)
        if any(hist_diff(hists[j], hists[i]) < thresh / 2 for j in behind):
            continue                                  # leaving a flash
        if cuts and i - cuts[-1] < min_len:
            continue
        cuts.append(i)
    return cuts


def lerp(a, b, t):
    return [x + (y - x) * t for x, y in zip(a, b)]


DARK, MID, BRIGHT, WHITE = [.7, .2, .1, 0], [.1, .6, .2, .1], [0, .1, .3, .6], [0, 0, 0, 1.0]
assert detect_cuts([DARK] * 3 + [BRIGHT] * 4) == [3]
assert detect_cuts([MID] * 5 + [WHITE] + [MID] * 4) == []
fade = [lerp(MID, DARK, k / 3) for k in range(4)] + [lerp(DARK, BRIGHT, k / 3) for k in range(1, 4)]
assert detect_cuts(fade) == []
assert detect_cuts([DARK] * 3 + [BRIGHT] * 3 + [MID] * 3) == [3, 6]
assert detect_cuts([DARK] * 3 + [MID] + [BRIGHT] * 4) == [3]   # min shot length
assert detect_cuts([]) == [] and detect_cuts([MID]) == []
print("q6 ok")
```

Verify: the three original tests, two real cuts in a row, a one-frame shot, and an empty list.

Pitfalls:
- Fixing only the threshold.
- Dropping the flash rule when a real cut lands next to a flash.

Follow-ups: how would you measure this on real titles? (Precision and recall against editor cut lists, with a tolerance of plus or minus one frame.)

Why this is likely: Netflix's MediaFM post says the model's input unit is a shot from a shot boundary detection algorithm, and a July 2026 Blind thread reports a Netflix AI-assisted round where candidates are judged on when and how they push back on the AI; the debug format is inferred, medium-low confidence.

</details>

<a id="q7"></a>

#### Q7. Shot-embedding inference on one GPU runs at 23% utilization. Read this profile and make the job faster without changing the model

Round: AI-assisted coding. AI policy: Provided (in-editor assistant). Type: Optimize. Difficulty: Hard. Topic: Inference input pipelines, decode, overlap.

```text
batch=64 frames, 1 dataloader worker, fp32, 4K ProRes source decoded then resized to 224
step  data_wait_ms  h2d_ms  forward_ms  gpu_util
 100         180       20          60      23%
 101         184       21          60      23%
 102         177       20          61      23%
 103         181       20          60      23%
 104         179       21          60      23%
top host ops: ProRes decode 4096x2160 (71%), resize (14%), to_tensor (9%)
nvidia-smi: memory 9 GB / 80 GB, PCIe RX 1.1 GB/s
```

<details>
<summary>Show answer</summary>

Task: find the bottleneck from the numbers, rank the fixes, and show the expected speedup before writing code.

What they grade: reading the profile rather than guessing, math that predicts the result, and rejecting suggestions that cannot help.

Approach: each batch takes 180 + 20 + 60 = 260 ms in sequence, and the GPU is busy only 60 of those, or 23%. Host decode dominates, because the job decodes full 4K ProRes to feed a 224 pixel model. The fixes, in order:

1. Decode from the existing low-resolution proxy, or let the decoder downscale. Going from 4096x2160 to 1024x540 cuts pixels 16x. Per-frame overhead does not shrink, so plan on decode cost dropping about 8x.
2. Run 4 to 8 dataloader workers and prefetch, so decode overlaps the forward pass.
3. Use pinned memory and non-blocking copies, so H2D overlaps compute.
4. Run fp16 or bf16 autocast. That roughly halves forward time on modern GPUs, but it only matters once the GPU is the bottleneck.

The assistant's first suggestion was to raise the batch size to 512. That cannot help an input-bound job and only adds memory, so I said no. Its second suggestion, workers plus pinned memory, was right. I asked for that diff alone and checked that it kept frame order and seeded nothing new.

Code: a model of the pipeline, to predict the gains before changing the job.

```python
def step_ms(decode, h2d, fwd, workers=1, overlap=False):
    if not overlap:
        return decode / workers + h2d + fwd
    return max(decode / workers, h2d, fwd)


base = step_ms(180, 20, 60)
assert base == 260 and round(60 / base, 2) == 0.23
proxy = 180 / 8                                   # decode from a 1024x540 proxy
plan = [("4 workers, overlapped", step_ms(180, 20, 60, workers=4, overlap=True)),
        ("plus proxy decode", step_ms(proxy, 20, 60, workers=4, overlap=True)),
        ("plus fp16 forward", step_ms(proxy, 20, 30, workers=4, overlap=True))]
for name, ms in plan:
    print(f"{name}: {ms:.0f} ms/batch, {base / ms:.1f}x, {64 * 1000 / ms:.0f} frames/s")
assert step_ms(180, 20, 60, workers=4, overlap=True) == 60
assert step_ms(proxy, 20, 30, workers=4, overlap=True) == 30
print("q7 ok")
```

The model predicts 4.3x from workers and overlap alone, because the GPU becomes the bottleneck. With proxy decode and fp16 it predicts 8.7x, or about 2,100 frames per second.

Verify: re-profile after each change. Compare embeddings with the old path on 200 shots: cosine similarity above 0.999 for fp16, and a check that proxy decode does not change results more than the team's tolerance allows. Confirm frame order with a checksum.

Pitfalls:
- Tuning the model when the profile shows the input is the bottleneck.
- Too many workers, which spill host RAM with 4K frames.

Follow-ups: when would you move decode to the GPU (NVDEC)? How does this change across 100 GPUs reading from shared storage?

Why this is likely: no Netflix report names this exact task; it is inferred from the job's model-deployment requirement and Netflix's Scaling Media ML post on GPU inference over media assets, set in the AI-assisted round format reported on Blind in July 2026; low to medium confidence.

</details>

### ML and computer vision depth

<a id="q8"></a>

#### Q8. A video model trains fine on one GPU. On 16 GPUs the loss spikes to NaN early, and after a fix the job hangs at the end of epoch 0. Read the logs, find both causes and fix them

Round: Domain depth. AI policy: Unknown. Type: Trace debug. Difficulty: Hard. Topic: Distributed data parallel training, learning rate scaling.

```text
config: 2 nodes x 8 GPUs, PyTorch DDP via Ray Train, per-GPU batch 32, global batch 512
config: lr = base_lr 0.1 x world_size 16 = 1.6 from step 0, no warmup
[rank 0] step    0 loss 6.91 lr 1.600
[rank 0] step   50 loss 9.84 lr 1.600 grad_norm 412.7
[rank 0] step  100 loss nan  lr 1.600 grad_norm inf
--- rerun with 500 step linear warmup ---
[rank 0] step  100 loss 6.12 lr 0.320 grad_norm 3.1
[rank 3]  epoch 0 dataset: skipped 41 unreadable clips, yielded 1209 batches
[rank 11] epoch 0 dataset: skipped 3 unreadable clips, yielded 1210 batches
[rank 0]  epoch 0 done at step 1209, entering validation
[rank 11] step 1209 waiting in allreduce (bucket 0)
[E ProcessGroupNCCL.cpp] [Rank 11] Watchdog caught collective operation timeout:
  WorkNCCL(OpType=ALLREDUCE, Timeout(ms)=1800000) ran for 1800042 milliseconds before timing out
```

<details>
<summary>Show answer</summary>

Task: explain the NaN and the hang from the logs, give the fix for each, and say how to prove the fixes work.

What they grade: tying each symptom to a line in the logs, knowing how DDP synchronizes, and fixes that hold for the whole catalog.

Approach: there are two separate faults.

The NaN: the learning rate was scaled linearly with 16 GPUs to 1.6 and applied from step 0. Gradient norm climbs from about 3 to 412, then to infinity, within 100 steps. Linear scaling needs warmup, and the rerun shows it: with warmup the gradient norm is 3.1 at step 100. Keep the warmup. Also clip the gradient norm, for example at 1.0, and log it. Alternatives: square-root scaling (lr 0.4) is safer without warmup but converges slower at large batch. LAMB or LARS help at much larger batches than 512. Warmup with linear scaling is the standard choice here.

The hang: each rank skips unreadable clips inside its own shard. So rank 3 yields 1,209 batches and rank 11 yields 1,210. DDP runs an allreduce on every backward pass. Rank 11 waits in allreduce for a step that no other rank will join, until the 30-minute watchdog fires. Three fixes:

1. Validate the manifest before sharding, so every rank gets the same count. This is the best fix, because it also stops bad clips from wasting decode time each epoch.
2. Agree on a step count. Allreduce the per-rank batch counts at epoch start and stop all ranks at the minimum.
3. Use DDP's `Join` context for uneven inputs. It works, but it hides the data problem.

I would ship fix 1 and add fix 2 as a guard, plus a lower NCCL timeout in dev so hangs fail in minutes.

Code:

```python
def batches(shard_size, skipped, per_gpu_batch, drop_last=False):
    n = shard_size - skipped
    return n // per_gpu_batch if drop_last else -(-n // per_gpu_batch)


def warmup_lr(step, peak, warmup_steps):
    return peak * min(1.0, (step + 1) / warmup_steps)


shard, bs = 38_715, 32
per_rank = {3: batches(shard, 41, bs), 11: batches(shard, 3, bs)}
assert per_rank == {3: 1209, 11: 1210}          # uneven: the hang
agreed = min(per_rank.values())                   # fix 2: every rank stops here
assert agreed == 1209
manifest_ok = shard - 41                          # fix 1: filter before sharding
assert len({batches(manifest_ok, 0, bs) for _ in range(16)}) == 1
assert abs(warmup_lr(99, 1.6, 500) - 0.32) < 1e-9
assert warmup_lr(10_000, 1.6, 500) == 1.6
print("q8 ok")
```

Verify: a 2-GPU test with one shard holding an extra bad clip must finish. A 16-GPU run must match the single-GPU loss curve within noise for the first 1,000 steps at the same effective batch.

Pitfalls:
- Raising the NCCL timeout, which delays the failure and leaves the cause in place.
- Blaming the network because the error message comes from NCCL.

Follow-ups: per-GPU batch norm statistics on small batches. How would you find a slow rank that drags every step?

Why this is likely: Netflix's media ML blog describes multi-GPU, multi-node training on Ray, and Netflix has published on large-scale distributed training of media models; the job asks for distributed model training with GPUs. Medium confidence on the exact prompt.

</details>

<a id="q9"></a>

#### Q9. Fine-tune a CLIP-style model so editors can search a title's shots by text ("rain on a window at night"). Write the loss and recall@k, and handle false negatives

Round: Domain depth. AI policy: Unknown. Type: Concept plus code. Difficulty: Hard. Topic: Contrastive learning, in-video search.

<details>
<summary>Show answer</summary>

Task: given a batch of shot embeddings and caption embeddings, write the symmetric contrastive loss and recall@k. Then explain what goes wrong with in-batch negatives on video.

What they grade: the loss written correctly, the temperature, the false-negative problem that is specific to video, and an evaluation that matches how editors search.

Approach: normalize both embeddings and compute the similarity matrix S = shots times captions transposed, divided by the temperature. Use cross-entropy over rows (shot to text) and over columns (text to shot) with the diagonal as the target, then average the two. Learn the temperature (CLIP starts near 0.07) or tune it. Too low makes training unstable, and too high blurs ranking.

The video problem: a batch from one title holds many shots of the same scene with nearly the same caption. Treating them as negatives pushes apart things that should match. That hurts recall and teaches the model to separate on lighting noise. Three fixes:

1. Mask negatives that share a scene id or have near-duplicate captions (this answer's code).
2. Sample batches across titles so duplicates are rare. That is cheap, but it gives fewer hard negatives.
3. Use multi-positive targets that spread the target mass over all matching shots.

I start with masking plus cross-title sampling, and I add mined hard negatives from the same title once the loss is stable.

Evaluation: recall@1, 5 and 10 for text-to-shot search on held-out titles with captions written by editors. Count a hit if any shot in the right scene is returned, because editors accept any of those. Break results down by genre, animation versus live action, and night scenes.

Code:

```python
import math


def norm(v):
    s = math.sqrt(sum(x * x for x in v)) or 1.0
    return [x / s for x in v]


def sim(A, B, temp):
    A, B = [norm(a) for a in A], [norm(b) for b in B]
    return [[sum(x * y for x, y in zip(a, b)) / temp for b in B] for a in A]


def xent(row, target, mask):
    m = max(v for j, v in enumerate(row) if mask[j])
    lse = m + math.log(sum(math.exp(v - m) for j, v in enumerate(row) if mask[j]))
    return lse - row[target]


def clip_loss(shots, caps, scene, temp=0.07):
    S = sim(shots, caps, temp)
    n = len(S)
    keep = [[i == j or scene[i] != scene[j] for j in range(n)] for i in range(n)]
    l_st = sum(xent(S[i], i, keep[i]) for i in range(n)) / n
    l_ts = sum(xent([S[r][j] for r in range(n)], j, keep[j]) for j in range(n)) / n
    return (l_st + l_ts) / 2


def recall_at_k(caps, shots, scene, k):
    S = sim(caps, shots, 1.0)
    hits = 0
    for i, row in enumerate(S):
        top = sorted(range(len(row)), key=lambda j: -row[j])[:k]
        hits += any(scene[j] == scene[i] for j in top)
    return hits / len(S)


shots = [[1, 0, 0], [0.98, 0.2, 0], [0, 1, 0], [0, 0, 1]]
caps = [[1, 0.1, 0], [1, 0.1, 0], [0, 1, 0.1], [0.1, 0, 1]]
scene = ["rain", "rain", "car", "kitchen"]
masked = clip_loss(shots, caps, scene)
unmasked = clip_loss(shots, caps, list(range(4)))
assert masked < unmasked          # duplicates no longer punished as negatives
assert recall_at_k(caps, shots, scene, 1) == 1.0
print(f"masked {masked:.3f} unmasked {unmasked:.3f} q9 ok")
```

Verify: the masked loss is lower when scene duplicates exist, and recall@1 is 1.0 on aligned data.

Pitfalls:
- Forgetting to normalize, so the loss rewards vector length.
- Reporting recall on generated captions, which flatters the model.

Follow-ups: how do you fine-tune without forgetting general concepts? (A lower learning rate on the backbone, or train only adapters.)

Why this is likely: Netflix's MediaFM post describes a CLIP-style model over shots and links Netflix's in-video search work, and the team owns media understanding models; medium confidence.

</details>

<a id="q10"></a>

#### Q10. A transformer encoder reads every shot of a two-hour title. Size its attention memory, then say when a KV cache helps and when it does not

Round: Domain depth. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: Attention cost, KV cache, long sequences.

<details>
<summary>Show answer</summary>

Task: estimate the sequence length, the attention score memory and the KV cache size, and choose an architecture.

What they grade: correct formulas, sensible assumptions stated up front, and knowing that a KV cache serves autoregressive decoding, not a bidirectional encoder pass.

Approach: assume an average shot of about 3 seconds, so a two-hour title has about 2,400 shots. With one token per shot (a fused shot embedding, as in a shot-level foundation model) the sequence length is 2,400. Naive attention materializes an L x L score matrix per head. In fp16 with 16 heads that is 2,400 x 2,400 x 16 x 2 bytes, about 184 MB per layer. That is fine for inference and manageable in training with activation checkpointing.

With 16 patch tokens per shot, L is 38,400 and the same matrix is about 47 GB per layer. That rules out naive attention. The options:

1. A fused attention kernel such as FlashAttention. It never stores the matrix, so memory is O(L), but compute still grows with L squared.
2. A hierarchy: pool patches into one token per shot, then run attention over shots. That cuts both memory and compute by 256x, at the cost of detail inside each shot.
3. Local windows plus a few global tokens.

For title-level tasks such as scene grouping I choose the hierarchy, and I keep a patch-level model only for tasks that need detail inside a shot.

KV cache: an encoder sees the whole title at once, so there is nothing to cache. Every pass recomputes K and V. A KV cache helps a causal model that consumes shots one at a time, for example scoring each new shot as dailies arrive, or a decoder generating text. Its size is 2 x layers x L x d_model x bytes, which is about 118 MB for 12 layers, d_model 1,024 and 2,400 shots in fp16. Grouped-query attention with 4 KV heads instead of 16 cuts that by 4x.

Code:

```python
def scores_bytes(L, heads, b=2):
    return L * L * heads * b


def kv_bytes(layers, L, d_model, b=2, kv_ratio=1.0):
    return 2 * layers * L * d_model * b * kv_ratio


shots = 2 * 3600 // 3
assert shots == 2400
one_tok = scores_bytes(shots, 16) / 1e6
patch_tok = scores_bytes(shots * 16, 16) / 1e9
kv = kv_bytes(12, shots, 1024) / 1e6
gqa = kv_bytes(12, shots, 1024, kv_ratio=4 / 16) / 1e6
print(f"1 token/shot: {one_tok:.0f} MB/layer, 16 tokens/shot: {patch_tok:.0f} GB/layer")
print(f"KV cache {kv:.0f} MB, with GQA {gqa:.0f} MB")
assert round(one_tok) == 184 and round(patch_tok) == 47 and round(kv) == 118
print("q10 ok")
```

Verify: measure peak memory for one layer at L = 2,400 and compare it with the estimate.

Pitfalls:
- Saying "use a KV cache" for an encoder.
- Forgetting the head count in the score matrix.

Follow-ups: how would you place shots in time (position by shot index or by timestamp)?

Why this is likely: an October 2026 Netflix Research Engineer screen report lists KV cache and attention among its ML fundamentals, and Netflix's MediaFM post describes a transformer encoder over shot sequences; medium confidence.

</details>

<a id="q11"></a>

#### Q11. A generative relighting model (like the tools Netflix acquired with InterPositive) is about to reach compositors. Design the evaluation that decides whether artists can rely on it

Round: Domain depth. AI policy: Unknown. Type: Concept plus code. Difficulty: Hard. Topic: Eval design for generative media tools.

<details>
<summary>Show answer</summary>

Task: define offline metrics, a human study with artists, slices and a ship rule, with sample sizes.

What they grade: metrics tied to what artists need, a power calculation, attention to temporal and identity failures, and a clear go or no-go rule.

Approach: work in three layers.

1. Offline, against ground truth. Use CG scenes rendered under source and target lighting, plus light-stage captures where available. Score LPIPS and PSNR against the target render. Synthetic data proves the model follows the requested light. It does not prove the output looks real on film plates.
2. Failure checks on real plates. A temporal flicker score: how much more the output changes frame to frame than the input does, computed on static regions or after motion compensation. Face identity drift: the cosine between face embeddings before and after. Skin-tone shift measured in a perceptual color space.
3. Artists. A blinded pairwise test against the current workflow on the same shots: "which would you send to review?" Also track task metrics in a pilot: time to an approved shot, and the share of outputs used without paint fixes.

Sample size: to show a 60% win rate against 50% at 95% confidence and 80% power takes about 194 comparisons, or 20 artists rating 10 shots each. Report the win rate with a Wilson interval.

Slices: skin tones, faces, hair, motion blur, night exteriors, animation versus live action. A good average can hide a bad slice.

Ship rule: go to a pilot when the lower bound of the win-rate interval is above 50%, flicker is no worse than the current workflow on 95% of shots, and no slice shows a severe identity or skin-tone failure.

Code:

```python
import math


def wilson(wins, n, z=1.96):
    p = wins / n
    d = 1 + z * z / n
    c = (p + z * z / (2 * n)) / d
    h = z * math.sqrt(p * (1 - p) / n + z * z / (4 * n * n)) / d
    return c - h, c + h


def n_for_winrate(p1, p0=0.5, za=1.96, zb=0.8416):
    num = za * math.sqrt(p0 * (1 - p0)) + zb * math.sqrt(p1 * (1 - p1))
    return math.ceil((num / (p1 - p0)) ** 2)


def flicker(inp, out):
    d_in = sum(abs(b - a) for a, b in zip(inp, inp[1:])) / (len(inp) - 1)
    d_out = sum(abs(b - a) for a, b in zip(out, out[1:])) / (len(out) - 1)
    return d_out - d_in


assert n_for_winrate(0.6) == 194
lo, hi = wilson(124, 194)
assert lo > 0.5
assert flicker([0.5] * 5, [0.6] * 5) == 0
assert flicker([0.5] * 5, [0.6, 0.7, 0.6, 0.7, 0.6]) > 0
print(f"win rate 124/194 interval {lo:.3f} to {hi:.3f}; q11 ok")
```

Verify: test the metric code on clips with known flicker, and check that the ratings from two artists agree on a shared subset before trusting the totals.

Pitfalls:
- Judging on still frames only.
- Letting artists see which output came from the model.

Follow-ups: how do you keep evaluating after launch, and how do you handle a tool that artists like but supervisors reject?

Why this is likely: press reports on Netflix's July 2026 CPTO interview describe its acquisition of a generative filmmaking startup for relighting, reframing and reshoots in post, and this role builds artist-facing ML tools; medium confidence on the exact prompt.

</details>

### ML system design

<a id="q12"></a>

#### Q12. Your new match-cut ranker lifts precision@10 from 0.42 to 0.50 offline. What do you check and run before editors see it?

Round: ML system design. AI policy: Unknown. Type: Design. Difficulty: Medium. Topic: Launch decisions for internal ML tools.

<details>
<summary>Show answer</summary>

Task: decide what has to be true before launch, and design the online test for a small user base.

What they grade: whether you trust the offline gain only after checking it, power math for few users, guardrails, and a rollback plan.

Approach: first, check the offline result. Are the labelled pairs from titles the old model never saw? Are the rater pools the same? Does the gain hold in every slice (animation, documentary, dark scenes), or does one genre carry it? Bootstrap the 0.08 gain over titles, not over pairs, because pairs within a title are correlated.

Next, check cost and behavior. Latency of the editor's request (the p95 for a full ranked list should stay under about 2 seconds), GPU cost per title, and how much the top 10 changes. A large change means editors have to learn the tool again.

Then run shadow mode for a week, scoring both rankers on live requests and logging both. After that, run an online test whose primary metric is suggestions accepted per session, with guardrails on time per session and on reports of bad suggestions.

Numbers:

```python
def n_per_arm(sd, delta, z=1.96 + 0.8416):
    return 2 * z * z * sd * sd / (delta * delta)


print(round(n_per_arm(0.2, 0.03)), "sessions per arm, randomized by session")
print(round(n_per_arm(0.1, 0.03)), "editors per arm, randomized by editor")
mde = (2 * (1.96 + 0.8416) ** 2 * 0.1 ** 2 / 75) ** 0.5
print(f"with 75 editors per arm the minimum detectable lift is {mde:.3f}")
```

Assume an acceptance rate around 0.30 per session. Detecting +0.03 needs about 700 sessions per arm if sessions are independent. But one editor's sessions are correlated, and editors learn the tool, so randomize by editor. With a per-editor standard deviation of 0.10 that needs about 174 editors per arm. With 150 editors in total, the test can detect only about +0.046.

Alternatives: interleaving merges both rankers into one list and credits whichever one produced the accepted pair. It removes most variance between editors and suits small user bases. A plain editor-level test is cleaner to explain but underpowered. Interleaving is the main test here, with an editor-level holdout of a few weeks to confirm that the session gain is real.

Risks: novelty effects, a few heavy users dominating the result, and tracking that counts a click as an acceptance. Keep a feature flag for rollback.

Why this is likely: an August 2026 Reddit post on Netflix MLE interviews says they "care a lot about what happens after your offline metric improves", and Netflix published its match cutting work; medium confidence.

</details>

<a id="q13"></a>

#### Q13. Design a service that proposes match cuts across every shot in a season, then across the whole catalog

Round: ML system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Embedding retrieval, pairwise ranking at scale.

<details>
<summary>Show answer</summary>

Task: given shots, return pairs that would make good match cuts (similar framing, silhouette, motion or color), ranked, for an editor's tool. Start with one season, then scale to the catalog for trailers and promos.

What they grade: using the math to pick brute force or an index, a two-stage design, removing trivial matches, and an evaluation.

Numbers:

```python
season_shots = 10 * 2_400
pairs = season_shots * (season_shots - 1) // 2
flops = season_shots ** 2 * 512 * 2
print(f"{pairs / 1e6:.0f}M pairs, {flops / 1e12:.2f} TFLOP brute force at 512 dims")
catalog_shots = 50_000_000
print(f"catalog index: {catalog_shots * 512 * 2 / 1e9:.0f} GB fp16, {catalog_shots * 64 / 1e9:.1f} GB with 64-byte PQ codes")
rerank = season_shots * 50 * 2e-3 / 60
print(f"rerank top 50 per shot at 2 ms per pair: {rerank:.0f} GPU minutes per season")
```

A season of 10 episodes at about 2,400 shots each is 24,000 shots and 288 million pairs. Brute-force similarity at 512 dimensions is about 0.6 TFLOP, under a second on one GPU. So no index is needed inside a season. A catalog of an assumed 50 million shots needs an approximate index: about 51 GB of raw vectors, or 3.2 GB with product quantization.

Approach: there are two stages. Stage 1 retrieves the top 50 candidates per shot using several embeddings (overall image, silhouette or pose, motion). Stage 2 reranks each candidate pair with a model trained on editor-accepted pairs, using features such as pose alignment at the boundary frames, optical flow direction and color histogram distance. That costs about 40 GPU minutes per season. Remove trivial matches first: shots from the same setup, shots next to each other, and near duplicates.

Alternatives: one pairwise model over every pair is the most accurate but costs 288 million inferences per season. Embeddings alone are fast, but they rank "same location" above "good transition". Two stages win. For the catalog, IVF-PQ beats HNSW on memory at this size (about 3.2 GB of codes versus more than 51 GB of raw vectors plus graph links for HNSW). HNSW gives higher recall at the same latency, so it suits a smaller index that fits in RAM. Pick by the recall needed, measured against exact search on a sample.

Risks: popular looks crowding the top results (add diversity in the ranking), ranking that favors the latest season, and leaking unreleased titles into promo searches. Filter by access rights at query time.

Follow-ups: how do you collect training labels without asking editors to label pairs?

Why this is likely: Netflix's tech blog describes match cutting as a pairwise shot problem, and the role asks for reusable algorithms and scalable designs; medium-high confidence on the domain.

</details>

<a id="q14"></a>

#### Q14. Design the batch job that computes shot embeddings for every new or re-delivered title, and backfills the catalog when the encoder changes. Show the GPU-hour math

Round: ML system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Batch inference pipelines, idempotency, backfills.

<details>
<summary>Show answer</summary>

Task: new assets arrive all week, and each needs embeddings within hours. A new encoder version must replace every vector without mixing versions in search.

What they grade: the size of the job, the trigger and idempotency design, a backfill that does not starve new work, and a safe switch-over.

Numbers:

```python
hours_week = 2_000
shots_per_hour = 3600 // 3
frames_per_shot = 8
gpu_fps = 2_000
weekly_frames = hours_week * shots_per_hour * frames_per_shot
print(f"weekly: {weekly_frames / 1e6:.1f}M frames, {weekly_frames / gpu_fps / 3600:.1f} GPU hours")
catalog_hours = 100_000
backfill = catalog_hours * shots_per_hour * frames_per_shot / gpu_fps / 3600
print(f"backfill: {backfill:.0f} GPU hours, {backfill / 64:.1f} hours on 64 GPUs")
print(f"storage: {catalog_hours * shots_per_hour * 512 * 2 / 1e9:.0f} GB of fp16 vectors")
```

These are assumptions to confirm with the team: 2,000 hours of new footage a week, 3 second shots, 8 frames per shot, and 2,000 frames per second per GPU (Q7's tuned pipeline). That gives 19.2 million frames, or about 2.7 GPU hours a week. A catalog backfill of 100,000 hours takes about 133 GPU hours, about 2 hours on 64 GPUs, and the vectors fit in about 123 GB. Decode and storage reads cost more than the GPUs, so decode from proxies.

Approach: an ingest event (asset delivered, or a new version) publishes a job keyed by (asset_id, asset_version, encoder_version). Workers check the feature store first, so retries and duplicate events cost nothing. A Ray or batch cluster runs shots in chunks of about 500 and writes vectors plus a manifest. Then a separate step loads them into the search index. Use two queues: new titles get priority, and the backfill only uses spare capacity.

Version switch: write the new vectors into a new index, check retrieval quality on a fixed query set, and then move the alias in one step. Search reads one encoder version at a time, because vectors from different encoders are not comparable.

Alternatives: computing on demand at query time skips the backfill, but it adds seconds of latency and repeats the work. Streaming each shot as decode produces it saves little, since assets arrive in batches. Precomputing in batches wins.

Risks: corrupt assets retrying forever (cap retries and move them to a dead-letter queue), a silent quality drop after a decoder upgrade (sample a check every week), and cost spikes during a backfill (set a budget limit).

Why this is likely: Netflix's media ML blog describes compute triggered when new media assets arrive, with features stored per asset, and its MediaFM post describes shot-level embeddings used across the catalog; medium-high confidence on the domain.

</details>

<a id="q15"></a>

#### Q15. Design an ML matte (rotoscoping) tool that compositors use inside Nuke: click on an object, get a matte for the whole shot, then correct it

Round: ML system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Artist-facing ML tools, latency, deployment.

<details>
<summary>Show answer</summary>

Task: decide where inference runs, how artists stay in control, how results reproduce, and how to keep unreleased footage safe.

What they grade: a latency budget backed by numbers, a design that keeps the artist in the loop, version pinning, and knowing how compositing tools work.

Numbers:

```python
w, h, ch, half = 3840, 2160, 4, 2
frame_mb = w * h * ch * half / 1e6
print(f"4K EXR frame {frame_mb:.1f} MB")
for name, mb_s in [("1 GbE", 125), ("10 GbE", 1250)]:
    print(f"{name}: {frame_mb / mb_s * 1000:.0f} ms to upload one frame")
proxy_mb = frame_mb / 16
print(f"quarter-resolution proxy {proxy_mb:.1f} MB, {proxy_mb / 125 * 1000:.0f} ms on 1 GbE")
```

A 4K half-float EXR frame with 4 channels is about 66 MB. Sending it over 1 GbE takes about 530 ms, which is too slow for a click-to-preview target of about 200 ms. A quarter-resolution proxy is about 4 MB and takes about 33 ms.

Approach: build a Nuke tool with two modes. It can be a gizmo or Python-scripted group that calls a model service, NukeX's Inference node running a .cat file built from a TorchScript model with CatFileCreator, or a C++ NDK plugin. Interactive mode runs a prompt-based segmentation model on the artist's workstation GPU, or on a nearby GPU service, at proxy resolution. Clicks and strokes are the prompts, and the result shows in under 200 ms. Propagate mode tracks the matte through the shot as a background job on the render farm at full resolution, so the Nuke UI thread never blocks. Artists fix keyframes, and propagation reruns from the fixed frames.

The node stores the model version, the prompts and the corrections in the Nuke script. A later re-render then gives the same matte, which supervisors need. Mattes come out as a standard alpha channel, so the rest of the comp is unchanged.

Alternatives: everything local is fastest and keeps footage on the machine, but it limits model size to the workstation GPU and makes updates harder. A central service is easy to update but adds network cost and needs approved content storage. A hybrid wins: local or nearby GPUs for interaction, the farm for full resolution.

Risks: unreleased footage stays in approved storage and runs only through in-house models. Edge quality on hair and motion blur is weak, so offer soft mattes and a confidence overlay. Artist corrections are valuable training data, but use them only with the right agreement in place.

Follow-ups: how do you measure success? (Time to final matte compared with manual roto, and the share of shots finished with the tool.)

Why this is likely: the job asks for a track record of deploying artist-facing ML tools and lists Nuke, Houdini, Maya and Unreal Engine, and a related Netflix job for the same team mentions human-in-the-loop inference; high confidence on the topic.

</details>

### Past project and culture

<a id="q16"></a>

#### Q16. Walk me through an ML tool you shipped to artists or creators: the architecture, the scale, adoption, and what you would redesign for 10x the users

Round: Past-project deep dive. AI policy: Unknown. Type: Design. Difficulty: Medium. Topic: Reverse system design, adoption.

<details>
<summary>Show answer</summary>

Task: explain one real system end to end, with numbers, your own decisions, and what you would change at ten times the users.

What they grade: depth on your own decisions, numbers that hold together, adoption measured honestly, and a redesign that targets the real bottleneck.

Approach: an example structure. Use your own project. Context: a background-removal tool for a game studio's 60 concept and marketing artists, running as a Photoshop panel that calls an internal GPU service. My part: I owned the service, the evaluation and the rollout. A researcher owned the model.

Architecture: the panel uploads a downscaled image (2048 px on the long edge) and gets a matte back. A FastAPI service on 2 GPUs runs the model, then edge refinement at full resolution on the client side. Median latency was 900 ms, and p95 was 2.1 seconds.

Numbers: about 1,500 requests a day at peak, under 10% GPU utilization. Adoption went from 8 pilot artists to 45 weekly users in 3 months. The share of mattes used without manual fixes went from 55% to 78% after we retrained on 2,000 corrected examples that artists sent through a "send fix" button.

Alternatives considered: running the model on artists' laptops (no GPU on half of them) and batch-only processing (artists wanted it interactive). The service won.

Redesign at 10x (600 artists, about 15,000 requests a day): the GPUs would still have spare capacity. The problems would be versioning and trust. I would pin model versions per project so re-exports match, add per-team rollout flags, and keep a fixed evaluation set per asset type, because one global accuracy number hid the hair failures. I would also move to an asynchronous API for batch exports.

Risks: in a story, claiming team results as your own. In the design, ignoring the support load. At our scale, one engineer spent 20% of their time on artist questions.

Why this is likely: candidate guides on Netflix describe a deep dive where you explain your largest system and how you would scale it, an older Netflix MLE report lists a past-experience round, and the job asks for deployed artist-facing ML tools; medium confidence.

</details>

<a id="q17"></a>

#### Q17. Tell me about a time an AI assistant gave you wrong code or analysis that looked right. How did you catch it, and what do you do differently now?

Round: Director and culture. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: Judgment with AI tools, candor.

<details>
<summary>Show answer</summary>

Task: one specific story with the stakes, how you caught the error, what you changed, and how you use AI day to day.

What they grade: judgment about when to trust the tool, a checking habit you actually follow, candor about your own miss, and what changed for the team.

Approach: an example structure. Situation: I asked an assistant to write a script that computed per-title color statistics for a model's training data and to summarize which titles were outliers. The code ran, the chart looked plausible, and the summary said night scenes were well covered.

How I caught it: my habit is to check any generated analysis against one number I can compute by hand. I picked a title I knew was mostly night exteriors, and the script reported a mean luma that was far too bright. The assistant had read frames with a library call that converts color spaces silently. The math ran on gamma-encoded values while the comments claimed linear light, so dark frames looked brighter than they were. It had also skipped frames that failed to decode without counting them.

Action: I fixed the conversion and added a test on a synthetic black and white clip with known values. I also added a count of skipped frames, then reran the analysis. Night scenes turned out to be under-represented by about half. We added 300 night shots to training, and the night-scene error on the matte eval dropped by about a third.

What I do now: I give the assistant small, scoped tasks with the interfaces pasted in. I read every diff. I ask it to write tests first and run them myself, and I keep one hand-checked number for any analysis. I also told the team about my miss in our weekly review, and two people added the same check to their scripts.

Pitfalls:
- A story where the AI was wrong but nothing was at stake.
- Blaming the tool rather than your own process.

Follow-ups: when would you not use AI at all? How do you review a teammate's AI-written pull request?

Why this is likely: Netflix's CPTO said in July 2026 that she expects AI fluency at every level, and a Netflix-verified commenter on Blind said candidates are judged on when and how they push back on AI; medium confidence.

</details>

## Prep checklist

- [ ] Code a timed cache and an undo and redo history from scratch, with an injected clock and tests, in 25 minutes each.
- [ ] Review three real pull requests from open media or ML repos and write down what their tests could not catch.
- [ ] Practice one AI-assisted session: read the code first, give one scoped prompt, review the diff, run the tests, and catch one wrong suggestion out loud.
- [ ] Be able to explain shot boundary detection, its failure modes (flashes, fades, dissolves) and how to measure it against cut lists.
- [ ] Write a symmetric contrastive loss and recall@k by hand, and explain false negatives in video batches.
- [ ] Size attention memory and a KV cache for a long sequence, and know when each applies.
- [ ] Debug distributed training: uneven batches, learning-rate warmup, NCCL timeouts, slow ranks.
- [ ] Profile an input-bound GPU job and predict the speedup before changing it.
- [ ] Design one media batch pipeline and one artist-facing tool with latency, storage and GPU-hour numbers.
- [ ] Prepare a deep dive on an ML tool you shipped to users, with adoption numbers and a 10x redesign.
- [ ] Prepare one story about catching wrong AI output, and one about giving or receiving hard feedback.
- [ ] Ask the recruiter which rounds allow or provide AI, and whether there is an online assessment.

## Sources

- Blind, Netflix AI-assisted interview thread (July 2026): https://www.teamblind.com/post/netflix-ai-assisted-interview-7o1ukyfe
- Reddit, Netflix senior SWE interview process, August 2026 (timed cache, undo and redo, behavioral rounds): https://www.reddit.com/r/OfferEngineering/comments/1vv14ji/netflix_senior_swe_interview_process_august_2026
- Reddit, Netflix Research Engineer phone screen (ML fundamentals and pipeline PR review): https://www.reddit.com/r/OfferEngineering/comments/1wzkqd8/netflix_research_engineer_phone_screen_all_the
- Reddit, Netflix MLE interviews and what happens after the offline metric improves: https://www.reddit.com/r/OfferEngineering/comments/1w2nwjo/netflix_mle_interviews_care_a_lot_about_what/
- Glassdoor, Netflix Machine Learning Engineer interviews: https://www.glassdoor.com/Interview/Netflix-Machine-Learning-Engineer-Interview-Questions-EI_IE11891.0,7_KO8,33.htm
- Lenny's Podcast, Netflix CPTO on AI and the future of product and tech roles: https://www.lennysnewsletter.com/p/netflix-cpto-on-ai-and-the-future
- Dealroom, summary of the Netflix CPTO interview: https://app.dealroom.co/news/note/systems-thinkers-over-specialists-netflix-cpto-elizabeth-stone-on-rebuilding-the-org-for-an-ai-era
- The Next Web, Netflix and AI fluency: https://thenextweb.com/news/netflix-ai-fluency-elizabeth-stone
- The Next Web, Netflix acquires a generative filmmaking startup: https://thenextweb.com/news/netflix-buys-ben-afflecks-ai-filmmaking-startup
- interviewing.io, Netflix hiring process guide: https://interviewing.io/guides/hiring-process/netflix
- 1point3acres, Netflix interview trends 2026 Q2: https://www.1point3acres.com/interview/guides/interview-trend-netflix-2026q2
- LockedIn AI, Netflix candidate reports (low confidence): https://www.lockedinai.com/company-details/NFLX
- prepare.sh, Netflix interview process 2026 (vendor, low confidence): https://prepare.sh/articles/netflix-interview-process-2026
- datainterview, Netflix MLOps engineer interview (vendor, low confidence): https://www.datainterview.com/blog/netflix-mlops-engineer-interview
- Blind, Netflix interview prep 2025 thread: https://www.teamblind.com/post/netflix-interview-prep-2025-oi6mk43k
- Netflix Tech Blog, Scaling Media Machine Learning at Netflix: https://netflixtechblog.com/scaling-media-machine-learning-at-netflix-f19b400243
- Netflix Tech Blog, MediaFM, the multimodal AI foundation for media understanding: https://netflixtechblog.com/mediafm-the-multimodal-ai-foundation-for-media-understanding-at-netflix-e8c28df82e2d
- Netflix Tech Blog, Match Cutting at Netflix: https://netflixtechblog.com/match-cutting-at-netflix-finding-cuts-with-smooth-visual-transitions-31c3fc14ae59
- Netflix Tech Blog, Creating Media with Machine Learning series: https://netflixtechblog.com/new-series-creating-media-with-machine-learning-5067ac110bcd
- Netflix match cutting code: https://github.com/Netflix/matchcut
- Netflix Research, large-scale distributed training of media models on Amazon FSx: https://research.netflix.com/publication/netflix-uses-amazon-fsx-for-large-scale-distributed-training-of-media
- Crowd-reported LeetCode company tags, Netflix, six-month list (low confidence, practice links only): https://github.com/liquidslr/leetcode-company-wise-problems/tree/main/Netflix
