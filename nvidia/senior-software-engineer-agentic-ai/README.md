# NVIDIA: Senior Software Engineer, Agentic AI

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

NVIDIA publishes the outline of its process, and each hiring team designs its own rounds inside it. The listing describes a senior engineer building open-source agent runtime libraries in Rust, Python, Go and Node.js, with async and event-driven systems, tool calls and cross-language APIs. NVIDIA's open-source NeMo Relay runtime (Rust core, Python and Node.js bindings, agent trajectory events) matches that scope, so the technical rounds below lean on that kind of work. That match is an inference. Candidates on Blind say there is no standard process across teams, so treat this as a likely shape and confirm it with the recruiter.

1. Recruiter or application review. A short call; NVIDIA says a resume can be matched to up to five roles. Virtual. AI: not relevant.
2. Hiring manager screen. 30 to 60 minutes by phone or video. A close look at past projects ("what was your own contribution?"), sometimes a design talk about the team's project. Some teams skip this and open with a live coding round with an engineer. AI: not allowed.
3. Technical screen, one or two rounds. 45 to 60 minutes, live, in HackerRank or a shared editor, usually Python or C++. Recent reports describe practical, team-written problems: a multi-part Python parsing task, a REST API task with JSON, retries and rate limits, or parsing log records and returning the top N. Some teams use LeetCode easy to medium problems or a 60 to 70 minute online assessment instead. AI: not allowed.
4. Onsite coding, one to three rounds. 45 to 60 minutes each, on a whiteboard or an NVIDIA laptop, at least one round in person. Data structures built without the standard library, async and concurrency work, and performance-aware code. AI: not allowed.
5. Domain or project deep dive. 45 to 60 minutes with team engineers. For this team, expect agent runtime internals: language bindings, serialization cost, tracing and exporters, tool-call and LLM middleware, backpressure, and agent evaluation. AI: not allowed.
6. System design. 45 to 60 minutes. Infrastructure-leaning prompts: scheduling, isolation, observability, latency against throughput, failure handling. For an agent team, a tracing pipeline, a multi-provider LLM layer or a benchmark platform are plausible. AI: not allowed.
7. Hiring manager or behavioral round, about 45 to 50 minutes, on ownership, ambiguity and open-source collaboration, plus an optional 15 minute Insider Chat that NVIDIA says does not affect the decision. AI: not allowed.
8. Decision. NVIDIA says "within a matter of weeks", and feedback can be shared across teams. AI: not relevant.

AI tools: NVIDIA's How We Hire page says that using unapproved outside tools such as ChatGPT during an interview disqualifies the candidate, and that an in-person onsite at an NVIDIA office comes before any offer. No source shows an AI-assisted round at NVIDIA. Every question below assumes no AI, as at NVIDIA, so you write and check all the code yourself.

Classic algorithm questions are limited to two (Q3 and Q5), because 2026 reports show an easy to medium LeetCode-style screen or online assessment on some teams, while most senior loops use practical problems.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a client for a device telemetry API that survives retries and rate limits | Build | Technical screen | Not allowed | Medium |
| [Q2](#q2) | Parse an agent run log into spans, then report tool latency and self time | Build | Technical screen | Not allowed | Medium |
| [Q3](#q3) | Parse a service log, count errors per component and return the top N components | Classic algorithm | Technical screen | Not allowed | Medium |
| [Q4](#q4) | Implement a string-keyed hash map in C++ without the standard containers | Build | Onsite coding | Not allowed | Hard |
| [Q5](#q5) | Build an O(1) LRU cache with a time-to-live for tool results, without OrderedDict | Classic algorithm | Onsite coding | Not allowed | Medium |
| [Q6](#q6) | These async tool calls sometimes return nothing, and the agent gets stuck. Find the bugs | Debug | Onsite coding | Not allowed | Medium |
| [Q7](#q7) | A teammate's coding assistant wrote this tracing middleware. Review it before it ships... | Review | Onsite coding | Not allowed | Medium |
| [Q8](#q8) | Build an in-process event bus where a slow telemetry exporter can never stall the agent | Build | Onsite coding | Not allowed | Hard |
| [Q9](#q9) | This support agent loops between two tools in 18% of sessions. Read the trace, find... | Trace debug | Domain deep dive | Not allowed | Medium |
| [Q10](#q10) | Design the evaluation that proves a new context-compaction middleware does not hurt... | Concept plus code | Domain deep dive | Not allowed | Hard |
| [Q11](#q11) | Python calls into the Rust core on every event. Where does the time go, and how do... | Optimize | Domain deep dive | Not allowed | Hard |
| [Q12](#q12) | Sub-agents running concurrently get the wrong parent spans. Explain why and fix it | Debug | Domain deep dive | Not allowed | Medium |
| [Q13](#q13) | One runtime ships Rust, Python, Node.js and Go bindings. How do you keep behavior... | Design | Domain deep dive | Not allowed | Hard |
| [Q14](#q14) | Design the telemetry path for an agent runtime so tracing never slows or breaks the agent | Design | System design | Not allowed | Hard |
| [Q15](#q15) | Design the provider layer that lets one agent harness call OpenAI Chat, OpenAI... | Design | System design | Not allowed | Hard |
| [Q16](#q16) | Design a platform that runs 10,000 agent benchmark episodes per night against... | Design | System design | Not allowed | Hard |
| [Q17](#q17) | Tell me about a change to a library others depended on, made with incomplete information | Behavioral | Hiring manager | Not allowed | Medium |

### Technical screen coding (no AI, as at NVIDIA)

<a id="q1"></a>

#### Q1. Build a client for a device telemetry API that survives retries and rate limits

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: HTTP retries, backoff, testable I/O.

<details>
<summary>Show answer</summary>

Task: spec given in the round. `GET /devices` returns a JSON list of `{"id", "temp_c", "util"}`. The service sometimes returns 503, drops connections, or returns 429 with a `Retry-After` header in seconds. Write `get_json(url)` and `hot_devices(devices, max_temp_c)`, which returns devices above the temperature limit, busiest first.

What they grade: which errors you retry, a backoff that does not stampede, tests without a network, and clean handling of bad data.

Approach: separate I/O from logic first, because that is what makes the retry policy testable. `get_json` takes the transport and the sleep function as parameters, so a test can script responses and record delays without waiting.

Retry 429, 5xx and connection errors; fail fast on other 4xx, because a 400 or 404 will not fix itself. Use exponential backoff with full jitter (a random delay between 0 and min(cap, base x 2^attempt)), so a fleet of clients does not retry in lockstep. On 429 the server's `Retry-After` wins when it is longer. Malformed JSON on a 200 raises a clear error and is not retried, since the same body will come back.

Code:

```python
import json, random

RETRYABLE = {429, 500, 502, 503, 504}

class Response:
    def __init__(self, status, body="", headers=None):
        self.status, self.body, self.headers = status, body, headers or {}

def get_json(get, url, max_attempts=5, base=0.2, cap=5.0, sleep=None, rng=random.random):
    for attempt in range(max_attempts):
        try:
            r = get(url)
        except OSError:
            r = None
        if r is not None and 200 <= r.status < 300:
            try:
                return json.loads(r.body)
            except json.JSONDecodeError as e:
                raise ValueError(f"malformed JSON from {url}") from e
        if r is not None and r.status not in RETRYABLE:
            raise RuntimeError(f"HTTP {r.status} from {url}")
        if attempt == max_attempts - 1:
            break
        delay = rng() * min(cap, base * 2 ** attempt)
        if r is not None and r.status == 429 and r.headers.get("Retry-After", "").isdigit():
            delay = max(delay, float(r.headers["Retry-After"]))
        sleep(delay)
    raise TimeoutError(f"gave up on {url} after {max_attempts} attempts")

def hot_devices(devices, max_temp_c):
    ok = [d for d in devices if isinstance(d.get("temp_c"), (int, float)) and isinstance(d.get("util"), (int, float))]
    return sorted((d for d in ok if d["temp_c"] > max_temp_c), key=lambda d: (-d["util"], d["id"]))

def scripted(responses):
    it = iter(responses)
    def get(url):
        r = next(it)
        if isinstance(r, Exception):
            raise r
        return r
    return get

slept = []
body = json.dumps([{"id": "g0", "temp_c": 91, "util": 0.4}, {"id": "g2", "temp_c": 88, "util": 0.95},
                   {"id": "g1", "temp_c": 70, "util": 0.9}, {"id": "g3", "util": 0.99}])
get = scripted([Response(503), OSError("reset"), Response(429, headers={"Retry-After": "2"}), Response(200, body)])
assert [d["id"] for d in hot_devices(get_json(get, "/devices", sleep=slept.append, rng=lambda: 0.5), 85)] == ["g2", "g0"]
assert slept == [0.1, 0.2, 2.0]
try:
    get_json(scripted([Response(404)]), "/x", sleep=slept.append)
    raise AssertionError
except RuntimeError:
    pass
try:
    get_json(scripted([Response(500)] * 3), "/x", max_attempts=3, sleep=lambda s: None)
    raise AssertionError
except TimeoutError:
    pass
try:
    get_json(scripted([Response(200, 'not json')]), '/x', sleep=lambda s: None)
    raise AssertionError
except ValueError:
    pass
```

Verify: the scripted sequence above (503, reset, 429 with Retry-After, then success), a 404 that raises at once, three 500s that exhaust the attempts, and a 200 with a broken body. Records missing a field are skipped; in production, log the skip count.

Pitfalls:
- Retrying every 4xx, which hammers the service on a request that can never succeed.
- Ignoring `Retry-After`, or treating it as a minimum without a cap on total time.
- Calling `time.sleep` directly, which makes the test slow and flaky.

Follow-ups: a fixed delay is simpler but synchronizes retries across clients; a library such as `tenacity` saves code, but writing the 20 lines shows you know the policy. For a real service, also cap total time (for example 30 seconds) as well as attempts. How would this look with `asyncio` and a shared rate limiter?

Why this is likely: a first-hand 2026 Reddit report describes an NVIDIA practical coding round with this shape (REST call, JSON filtering and sorting, then retries, 429, backoff with jitter and testable I/O); medium confidence.

</details>

<a id="q2"></a>

#### Q2. Parse an agent run log into spans, then report tool latency and self time

Round: Technical screen. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Multi-part parsing, interval union.

<details>
<summary>Show answer</summary>

Task: you get a text log, one event per line, possibly out of order and with junk lines. Part 1: count and median latency per tool. Part 2: report spans that never ended and lines you could not parse. Part 3: self time per span (its duration minus the time covered by its children).

```text
t=0.000 span=s1 kind=agent name=planner ev=start
t=0.100 span=s2 parent=s1 kind=llm name=chat ev=start
t=0.900 span=s2 ev=end
t=1.000 span=s3 parent=s1 kind=tool name=search ev=start
t=1.250 span=s4 parent=s1 kind=tool name=search ev=start
t=1.400 span=s3 ev=end
t=1.950 span=s4 ev=end
t=2.000 span=s5 parent=s1 kind=tool name=fetch ev=start
garbage line
t=2.300 span=s1 ev=end
```

What they grade: robust parsing, keeping state keyed by span ID, and spotting the overlap trap in part 3.

Approach: parse each line into a dict of `key=value` tokens and key spans by ID, so an `end` that arrives before its `start` still pairs up. Part 3 is the trap. `s3` and `s4` run at the same time, so subtracting each child's duration double counts the overlap and can give negative self time. Subtract the union of child intervals instead: sort children by start and merge.

Code:

```python
from collections import defaultdict
from statistics import median

def parse(lines):
    spans, bad = {}, 0
    for line in lines:
        try:
            f = dict(tok.split("=", 1) for tok in line.split())
            t, sid, ev = float(f["t"]), f["span"], f["ev"]
        except (ValueError, KeyError):
            bad += 1
            continue
        s = spans.setdefault(sid, {"start": None, "end": None, "parent": None, "kind": None, "name": None})
        if ev == "start":
            s.update(start=t, parent=f.get("parent") or None, kind=f.get("kind"), name=f.get("name"))
        elif ev == "end":
            s["end"] = t
    return spans, bad

def tool_latency(spans):
    by = defaultdict(list)
    for s in spans.values():
        if s["kind"] == "tool" and s["start"] is not None and s["end"] is not None:
            by[s["name"]].append(s["end"] - s["start"])
    return {k: (len(v), round(median(v), 3)) for k, v in by.items()}

def self_time(spans):
    kids = defaultdict(list)
    for s in spans.values():
        if s["parent"] and s["start"] is not None and s["end"] is not None:
            kids[s["parent"]].append((s["start"], s["end"]))
    out = {}
    for sid, s in spans.items():
        if s["start"] is None or s["end"] is None:
            continue
        covered, cur_s, cur_e = 0.0, None, None
        for a, b in sorted(kids[sid]):
            if cur_e is None or a > cur_e:
                covered += (cur_e - cur_s) if cur_e is not None else 0.0
                cur_s, cur_e = a, b
            else:
                cur_e = max(cur_e, b)
        covered += (cur_e - cur_s) if cur_e is not None else 0.0
        out[sid] = round(s["end"] - s["start"] - covered, 3)
    return out

log = """t=0.000 span=s1 kind=agent name=planner ev=start
t=0.100 span=s2 parent=s1 kind=llm name=chat ev=start
t=0.900 span=s2 ev=end
t=1.000 span=s3 parent=s1 kind=tool name=search ev=start
t=1.250 span=s4 parent=s1 kind=tool name=search ev=start
t=1.400 span=s3 ev=end
t=1.950 span=s4 ev=end
t=2.000 span=s5 parent=s1 kind=tool name=fetch ev=start
garbage line
t=2.300 span=s1 ev=end""".splitlines()
spans, bad = parse(log)
assert bad == 1 and tool_latency(spans) == {"search": (2, 0.55)}
assert self_time(spans)["s1"] == 0.55
assert [k for k, s in spans.items() if s["end"] is None] == ["s5"]
spans2, _ = parse(list(reversed(log)))
assert self_time(spans2)["s1"] == 0.55
```

Verify: the sample (one bad line, one unfinished span, overlapping tool calls), the same log reversed, a span with no children (self time equals duration), and an unfinished child that must not count toward its parent's coverage.

Pitfalls:
- Pairing start and end by line order, which breaks on out-of-order logs.
- Summing child durations, which gives s1 a self time of 0.4 instead of 0.55 here.
- Letting one bad line crash the whole parse.

Follow-ups: keep all durations for an exact median (fine up to millions of spans), or use a streaming sketch such as t-digest when the log does not fit in memory. How would you report self time per tool name across many runs?

Why this is likely: a 2026 senior software engineer report on LeetCode Discuss describes a multi-part Python parsing problem in NVIDIA's technical screen, and the listing centers on agent harnesses and tracing; medium confidence.

</details>

<a id="q3"></a>

#### Q3. (Technical screen, no AI) Parse a service log, count errors per component and return the top N components

Round: Technical screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Hashing, top-k selection.
Related practice: [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) (Medium), [Group Anagrams](https://leetcode.com/problems/group-anagrams/) (Medium).

<details>
<summary>Show answer</summary>

Task: each line is `<timestamp> <LEVEL> <component> <message>`. Return the N components with the most `ERROR` lines, highest first, ties broken by component name. Skip lines that do not parse and report how many.

```text
2026-08-01T10:00:00Z INFO  gateway request ok
2026-08-01T10:00:01Z ERROR scheduler queue full
2026-08-01T10:00:01Z ERROR gateway upstream 503
2026-08-01T10:00:02Z WARN  exporter slow flush
2026-08-01T10:00:02Z ERROR scheduler queue full
2026-08-01T10:00:03Z ERROR exporter dropped 12 events
2026-08-01T10:00:03Z
2026-08-01T10:00:04Z ERROR gateway upstream timeout
2026-08-01T10:00:05Z ERROR scheduler lease lost
2026-08-01T10:00:06Z error tokenizer bad utf-8
```

What they grade: clean parsing, the right counting structure, a top-N selection with a stated complexity, and deterministic ties.

Approach: one pass with a hash map from component to count, so counting is O(m) for m lines. Then select. Three options: sort all K components, O(K log K); a heap of size N, O(K log N); bucket by count, O(m). For K in the thousands and small N, the heap is the usual answer and `heapq.nsmallest` with key `(-count, name)` gives the order and the tie rule in one call. Bucket sort wins only when K is huge and counts are small. Decide up front whether `error` in lowercase counts; I normalize the level with `upper()` and say so.

Code:

```python
import heapq
from collections import Counter

def top_error_components(lines, n):
    counts, bad = Counter(), 0
    for line in lines:
        parts = line.split(None, 3)
        if len(parts) < 3:
            bad += 1
            continue
        _, level, comp = parts[:3]
        if level.upper() == "ERROR":
            counts[comp] += 1
    top = heapq.nsmallest(n, counts.items(), key=lambda kv: (-kv[1], kv[0]))
    return top, bad

log = """2026-08-01T10:00:00Z INFO  gateway request ok
2026-08-01T10:00:01Z ERROR scheduler queue full
2026-08-01T10:00:01Z ERROR gateway upstream 503
2026-08-01T10:00:02Z WARN  exporter slow flush
2026-08-01T10:00:02Z ERROR scheduler queue full
2026-08-01T10:00:03Z ERROR exporter dropped 12 events
2026-08-01T10:00:03Z
2026-08-01T10:00:04Z ERROR gateway upstream timeout
2026-08-01T10:00:05Z ERROR scheduler lease lost
2026-08-01T10:00:06Z error tokenizer bad utf-8""".splitlines()
top, bad = top_error_components(log, 2)
assert top == [("scheduler", 3), ("gateway", 2)] and bad == 1
assert top_error_components(log, 10)[0][-2:] == [("exporter", 1), ("tokenizer", 1)]
assert top_error_components([], 3) == ([], 0)
```

Verify: the sample, N larger than the number of components, an empty log, a tie at the cut line, and a malformed line. Mention a property test against a full sort on random inputs.

Pitfalls:
- `Counter.most_common(n)` breaks ties by insertion order, not by name, so the output depends on log order.
- Splitting on single spaces, which breaks on the padded `INFO  ` column.
- Loading a multi-gigabyte log into memory instead of streaming lines.

Follow-ups: the log is 50 GB across 20 files (count per file in parallel, merge the counters, then select); top N over the last 5 minutes only (a ring of per-minute counters); approximate top N with bounded memory (Space-Saving or count-min sketch plus a heap).

Why this is likely: a 2026 r/OfferEngineering report on an NVIDIA senior software engineer phone screen describes parsing log records, counting items and returning the top N, and Glassdoor reviews from October 2026 mention a simple LeetCode-style screen on some teams; medium confidence.

</details>

### Onsite coding (no AI, as at NVIDIA)

<a id="q4"></a>

#### Q4. Implement a string-keyed hash map in C++ without the standard containers

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Open addressing, memory layout.
Related practice: [Design HashMap](https://leetcode.com/problems/design-hashmap/) (Easy).

<details>
<summary>Show answer</summary>

Task: write `put`, `get` and `erase` for string keys and 64-bit values with no `std::unordered_map`, `std::string` or `std::vector`. Explain how deletes affect probing, and test it.

What they grade: a defensible collision strategy, correct deletes, memory ownership, and tests that hit the edge cases.

Approach: open addressing with linear probing keeps all slots in one array, which is cache friendly and avoids a heap allocation per entry. Separate chaining is the alternative: deletes are simpler, but every insert allocates a node and lookups chase pointers, typically 2x to 3x slower at the same load factor. Linear probing needs tombstones for deletes; emptying a slot outright makes later keys in the same probe run unreachable. Tombstones pile up, so the map counts them and rehashes once full plus deleted slots pass 75% of capacity. A rehash triggered mostly by tombstones keeps the same capacity.

Code:

```cpp
#include <cassert>
#include <cstdint>
#include <cstdlib>
#include <cstring>
#include <cstdio>

class StrMap {
  struct Slot { char* key; int64_t val; uint8_t state; };   // 0 empty, 1 full, 2 deleted
  Slot* slots_ = nullptr;
  size_t cap_ = 0, size_ = 0, used_ = 0;                     // used_ counts full + deleted

  static uint64_t hash(const char* s) {                       // FNV-1a
    uint64_t h = 1469598103934665603ull;
    while (*s) { h ^= (unsigned char)*s++; h *= 1099511628211ull; }
    return h;
  }
  void rehash(size_t new_cap) {
    Slot* old = slots_; size_t old_cap = cap_;
    slots_ = (Slot*)calloc(new_cap, sizeof(Slot)); cap_ = new_cap; size_ = used_ = 0;
    for (size_t i = 0; i < old_cap; i++)
      if (old[i].state == 1) insert_owned(old[i].key, old[i].val);   // tombstones are dropped
    free(old);
  }
  void insert_owned(char* key, int64_t val) {
    size_t i = hash(key) & (cap_ - 1);
    while (slots_[i].state == 1) i = (i + 1) & (cap_ - 1);
    slots_[i] = {key, val, 1}; size_++; used_++;
  }
  long find(const char* key) const {
    if (!cap_) return -1;
    size_t i = hash(key) & (cap_ - 1);
    for (size_t n = 0; n < cap_ && slots_[i].state != 0; n++, i = (i + 1) & (cap_ - 1))
      if (slots_[i].state == 1 && strcmp(slots_[i].key, key) == 0) return (long)i;
    return -1;
  }
 public:
  StrMap() { rehash(8); }
  ~StrMap() { for (size_t i = 0; i < cap_; i++) if (slots_[i].state == 1) free(slots_[i].key); free(slots_); }
  StrMap(const StrMap&) = delete; StrMap& operator=(const StrMap&) = delete;
  size_t size() const { return size_; }

  void put(const char* key, int64_t val) {
    long f = find(key);
    if (f >= 0) { slots_[f].val = val; return; }
    if ((used_ + 1) * 4 > cap_ * 3) rehash(size_ * 2 >= cap_ / 2 ? cap_ * 2 : cap_);  // grow, or just clear tombstones
    size_t i = hash(key) & (cap_ - 1);
    while (slots_[i].state == 1) i = (i + 1) & (cap_ - 1);    // reuse first empty or tombstone
    if (slots_[i].state == 0) used_++;
    slots_[i] = {strdup(key), val, 1}; size_++;
  }
  bool get(const char* key, int64_t* out) const {
    long f = find(key);
    if (f < 0) return false;
    *out = slots_[f].val; return true;
  }
  bool erase(const char* key) {
    long f = find(key);
    if (f < 0) return false;
    free(slots_[f].key); slots_[f].key = nullptr; slots_[f].state = 2; size_--;
    return true;
  }
};

int main() {
  StrMap m; int64_t v;
  m.put("search", 3); m.put("fetch", 5); m.put("search", 7);
  assert(m.size() == 2 && m.get("search", &v) && v == 7);
  assert(m.erase("fetch") && !m.get("fetch", &v) && !m.erase("fetch"));
  char buf[32];
  for (int i = 0; i < 20000; i++) { snprintf(buf, sizeof buf, "k%d", i); m.put(buf, i); }
  for (int i = 0; i < 20000; i += 2) { snprintf(buf, sizeof buf, "k%d", i); assert(m.erase(buf)); }
  for (int i = 0; i < 20000; i++) {
    snprintf(buf, sizeof buf, "k%d", i);
    assert(m.get(buf, &v) == (i % 2 == 1) && (i % 2 == 0 || v == i));
  }
  for (int i = 0; i < 100000; i++) { m.put("churn", i); m.erase("churn"); }   // tombstone churn
  assert(m.size() == 10001);
  puts("ok");
}
```

Verify: build with `clang++ -std=c++17 -fsanitize=address,undefined` to catch leaks and out-of-bounds access. Tests cover overwrite, erase then lookup, 20,000 inserts with half erased (keys after a tombstone must still be found), and 100,000 put and erase cycles on one key.

Pitfalls:
- Clearing a slot on erase instead of leaving a tombstone, which loses keys further along the probe run.
- Never cleaning tombstones; the churn test then turns every probe into a full scan.
- Storing the caller's pointer instead of copying the key, or allowing copies that double free.

Follow-ups: why capacity is a power of two (modulo becomes a mask, but a weak hash then clusters); Robin Hood hashing to cut probe-length variance; making it safe for concurrent readers.

Why this is likely: a 2026 senior software engineer report on LeetCode Discuss describes C and C++ string problems without the STL, including a hash map from scratch, and calibrd's NVIDIA guide lists core data structures without the standard library; medium confidence.

</details>

<a id="q5"></a>

#### Q5. (Onsite coding, no AI) Build an O(1) LRU cache with a time-to-live for tool results, without OrderedDict

Round: Onsite coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Hash map plus doubly linked list.
Related practice: [LRU Cache](https://leetcode.com/problems/lru-cache/) (Medium), [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/) (Medium).

<details>
<summary>Show answer</summary>

Task: the harness caches deterministic tool results keyed by tool name and arguments. Implement `ToolCache(capacity, ttl_s, clock)` with `get(tool, args)` and `put(tool, args, value)`, both O(1). Entries older than `ttl_s` are misses. When full, evict the least recently used entry. The interviewer bans `OrderedDict` and `functools.lru_cache`.

What they grade: the hash map plus linked list structure, correct pointer updates, a canonical cache key, and an injectable clock for tests.

Approach: a dict maps key to node; a doubly linked list with sentinel head and tail keeps recency, most recent next to the head. `get` moves the node to the front; `put` inserts at the front and evicts `tail.prev` when over capacity. TTL is checked lazily on `get`: an expired node is unlinked and counted as a miss. The key is `(tool, json.dumps(args, sort_keys=True))`, so `{"a":1,"b":2}` and `{"b":2,"a":1}` hit the same entry.

Alternatives: lazy expiry (chosen) costs nothing extra but lets expired entries hold memory until touched or evicted; a min-heap by expiry removes them eagerly at O(log n) per write; a timer wheel does it in O(1) amortized but adds a background task. With a capacity cap, lazy expiry is enough.

Code:

```python
import json

class _Node:
    __slots__ = ("key", "value", "expires", "prev", "next")
    def __init__(self, key=None, value=None, expires=0.0):
        self.key, self.value, self.expires = key, value, expires
        self.prev = self.next = None

class ToolCache:
    def __init__(self, capacity, ttl_s, clock):
        self.cap, self.ttl, self.clock, self.map = capacity, ttl_s, clock, {}
        self.head, self.tail = _Node(), _Node()
        self.head.next, self.tail.prev = self.tail, self.head
        self.hits = self.misses = 0

    @staticmethod
    def _key(tool, args):
        return (tool, json.dumps(args, sort_keys=True, default=str))

    def _unlink(self, n):
        n.prev.next, n.next.prev = n.next, n.prev

    def _push_front(self, n):
        n.prev, n.next = self.head, self.head.next
        self.head.next.prev = n
        self.head.next = n

    def get(self, tool, args):
        n = self.map.get(self._key(tool, args))
        if n is None or n.expires <= self.clock():
            if n is not None:
                self._unlink(n); del self.map[n.key]
            self.misses += 1
            return None
        self._unlink(n); self._push_front(n)
        self.hits += 1
        return n.value

    def put(self, tool, args, value):
        k = self._key(tool, args)
        if k in self.map:
            self._unlink(self.map.pop(k))
        n = _Node(k, value, self.clock() + self.ttl)
        self.map[k] = n
        self._push_front(n)
        if len(self.map) > self.cap:
            lru = self.tail.prev
            self._unlink(lru); del self.map[lru.key]

now = [0.0]
c = ToolCache(2, ttl_s=10, clock=lambda: now[0])
c.put("search", {"q": "a", "k": 3}, "A"); c.put("search", {"q": "b"}, "B")
assert c.get("search", {"k": 3, "q": "a"}) == "A"
c.put("fetch", {"url": "x"}, "X")
assert c.get("search", {"q": "b"}) is None and len(c.map) == 2
now[0] = 10.0
assert c.get("fetch", {"url": "x"}) is None and len(c.map) == 1
c.put("search", {"q": "a", "k": 3}, "A2")
assert c.get("search", {"q": "a", "k": 3}) == "A2" and len(c.map) == 1
```

Verify: argument order does not matter, recency is updated by `get`, the right key is evicted, an entry expires exactly at `ttl`, and overwriting a key does not grow the map. A randomized test against a slow reference model (a list scan) catches pointer bugs.

Pitfalls:
- Forgetting to delete the evicted key from the dict, which leaks and returns stale nodes.
- Caching tools with side effects or user-specific results (`send_email`, per-user auth).
- Using `time.time()` directly, which makes TTL tests sleep.

Follow-ups: make it safe across asyncio tasks (a single-threaded loop needs no lock; threads do); coalesce concurrent misses for the same key so 10 agents trigger one tool call; cap by bytes instead of entries.

Why this is likely: calibrd's NVIDIA guide cites core data structure coding without the standard library at the onsite, and LRU Cache appears in the crowd-reported NVIDIA LeetCode list for the last three months (low confidence that this team uses it).

</details>

<a id="q6"></a>

#### Q6. These async tool calls sometimes return nothing, and the agent gets stuck. Find the bugs

Round: Onsite coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: asyncio, error surfacing, bounded concurrency.

<details>
<summary>Show answer</summary>

Task: the harness runs a batch of model-requested tool calls with this function, and `test_tool_error_is_visible_to_model` fails. Production also hits the search provider's rate limit when the model asks for 40 searches at once.

```python
import asyncio

async def run_tools(calls, registry, timeout=10):
    results = {}
    async def one(call):
        try:
            fn = registry[call["name"]]
            results[call["id"]] = await asyncio.wait_for(fn(**call["args"]), timeout)
        except Exception:
            results[call["id"]] = None
    await asyncio.gather(*[one(c) for c in calls])
    return [results[c["id"]] for c in calls]

# FAILED test_tool_error_is_visible_to_model:
#   expected {"ok": False, "error": "ValueError: index offline"}, got None
```

What they grade: finding every bug, including the ones the failing test misses; keeping cancellation intact; and a concurrency limit in the right place.

Approach: there are four bugs. First, every failure becomes `None`, so a crashed tool, an unknown tool and a timeout all look like "no results" to the model, which then retries; that is how loops start. Second, concurrency is unbounded, so 40 calls hit the provider at once. Third, results are keyed by call ID, and models do sometimes repeat an ID, so one result overwrites another. Fourth, a missing tool raises `KeyError` inside the same `try`, which hides a harness bug as a tool failure.

Code:

```python
import asyncio

async def run_tools(calls, registry, max_in_flight=4, timeout=10.0):
    sem = asyncio.Semaphore(max_in_flight)

    async def one(call):
        fn = registry.get(call["name"])
        if fn is None:
            return {"id": call["id"], "ok": False, "error": f"unknown tool {call['name']}"}
        async with sem:
            try:
                value = await asyncio.wait_for(fn(**call["args"]), timeout)
                return {"id": call["id"], "ok": True, "value": value}
            except asyncio.TimeoutError:
                return {"id": call["id"], "ok": False, "error": f"timeout after {timeout}s"}
            except Exception as e:                       # CancelledError is not caught
                return {"id": call["id"], "ok": False, "error": f"{type(e).__name__}: {e}"}

    return await asyncio.gather(*(one(c) for c in calls))

async def search(q):
    await asyncio.sleep(0.01)
    if q == "boom":
        raise ValueError("index offline")
    return [] if q == "nothing" else [q.upper()]

calls = [{"id": "c1", "name": "search", "args": {"q": "a"}},
         {"id": "c1", "name": "search", "args": {"q": "nothing"}},
         {"id": "c3", "name": "search", "args": {"q": "boom"}},
         {"id": "c4", "name": "nope", "args": {}}]
out = asyncio.run(run_tools(calls, {"search": search}))
assert out[0]["value"] == ["A"] and out[1] == {"id": "c1", "ok": True, "value": []}
assert out[2] == {"id": "c3", "ok": False, "error": "ValueError: index offline"}
assert out[3]["error"] == "unknown tool nope"
```

Verify: the failing test, a duplicate call ID, an unknown tool, a timeout, and a test that 40 calls never exceed 4 in flight (a counter inside a fake tool). `gather` returns results in input order, so the dict goes away.

Pitfalls:
- Catching `BaseException`, which swallows `CancelledError` (a `BaseException` since Python 3.8) and stops the agent from being cancelled.
- An empty list and an error looking the same to the model.

Follow-ups: a semaphore per batch is simple; a shared per-provider token bucket is better when several agents share one rate limit, and it belongs in the tool client. How would you add per-tool timeouts?

Why this is likely: candidate reports and NVIDIA prep guides describe practical debugging and concurrency coding onsite, and the listing asks for async programming in agent harnesses and tool calls; medium confidence.

</details>

<a id="q7"></a>

#### Q7. A teammate's coding assistant wrote this tracing middleware. Review it before it ships in the library

Round: Onsite coding. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: Review of AI-generated code, tracing, secrets.

<details>
<summary>Show answer</summary>

Task: review the pull request by hand, with no AI, as at NVIDIA. The middleware wraps every tool call and buffers events; an exporter drains the buffer every second.

```python
import asyncio, json, time

class ToolTracer:
    def __init__(self, max_buffer=1000):
        self.buf = [None] * max_buffer
        self.n = 0

    def record(self, event):
        self.buf[self.n % len(self.buf)] = event
        self.n += 1

    def drain(self):
        if self.n >= len(self.buf):
            start = (self.n + 1) % len(self.buf)
            out = self.buf[start:] + self.buf[:start]
        else:
            out = self.buf[:self.n]
        self.n = 0
        return out

    async def wrap(self, name, fn, **kwargs):
        start = time.time()
        async with asyncio.timeout_after(30):
            result = await fn(**kwargs)
        self.record({"tool": name, "args": json.dumps(kwargs), "ms": (time.time() - start) * 1000})
        return result
```

What they grade: ranking findings by severity, catching the hallucinated API, the secret leak and the missing failure path, and a tested fix.

Approach: findings, worst first.
1. Hallucinated API. `asyncio.timeout_after` does not exist. Python 3.11 has `asyncio.timeout`, and `asyncio.wait_for` works on every supported version. Every wrapped call raises `AttributeError`.
2. Security. Raw `kwargs` go into telemetry, and tool arguments often carry API keys or `Authorization` headers. Redact known secret keys recursively before serializing.
3. Missing edge case. The event is recorded only on success, so failed, timed-out and cancelled calls vanish from traces. Record in `finally` with a status.
4. Off-by-one. After the buffer wraps, the oldest event sits at `n % len`, not `(n + 1) % len`, so `drain` returns events out of order.
5. Smaller: `time.time()` jumps with clock changes, so use `time.perf_counter()`; dropped events are not counted.

Verdict: request changes. Using `deque(maxlen=...)` removes the index math; a preallocated array is only worth it if profiling shows the deque on the hot path.

Code:

```python
import asyncio, json, time
from collections import deque

SECRET_KEYS = {"api_key", "authorization", "token", "password", "secret"}

def redact(obj):
    if isinstance(obj, dict):
        return {k: "[redacted]" if k.lower() in SECRET_KEYS else redact(v) for k, v in obj.items()}
    if isinstance(obj, list):
        return [redact(v) for v in obj]
    return obj

class ToolTracer:
    def __init__(self, max_buffer=1000):
        self.buf = deque(maxlen=max_buffer)
        self.dropped = 0

    def record(self, event):
        if len(self.buf) == self.buf.maxlen:
            self.dropped += 1
        self.buf.append(event)

    def drain(self):
        out, self.buf = list(self.buf), deque(maxlen=self.buf.maxlen)
        return out

    async def wrap(self, name, fn, timeout=30.0, **kwargs):
        start, status = time.perf_counter(), "ok"
        try:
            return await asyncio.wait_for(fn(**kwargs), timeout)
        except BaseException as e:
            status = type(e).__name__
            raise
        finally:
            self.record({"tool": name, "args": json.dumps(redact(kwargs), default=str),
                         "status": status, "ms": round((time.perf_counter() - start) * 1000, 3)})

async def echo(q, api_key=None, headers=None):
    return q

async def fail(q):
    raise ValueError("down")

async def main():
    t = ToolTracer(max_buffer=3)
    await t.wrap("echo", echo, q="hi", api_key="sk-123", headers={"Authorization": "Bearer abc"})
    ev = t.drain()[0]
    assert "sk-123" not in ev["args"] and "Bearer abc" not in ev["args"]
    try:
        await t.wrap("fail", fail, q="x")
    except ValueError:
        pass
    assert t.drain()[0]["status"] == "ValueError"
    for i in range(5):
        await t.wrap("echo", echo, q=str(i))
    assert [json.loads(e["args"])["q"] for e in t.drain()] == ["2", "3", "4"] and t.dropped == 2

asyncio.run(main())
```

Verify: an import-time smoke test (catches finding 1), a secret-in-args test, a failing tool that still records a span, and a wraparound order test. Each would have failed on the original.

Pitfalls:
- Approving because the happy path "looks right"; the hallucinated call means it never worked.
- Redacting only top-level keys, which misses nested headers.

Follow-ups: how would you redact secrets that appear inside free-text arguments? Should the tracer sample instead of buffering everything?

Why this is likely: the role builds open-source agent observability libraries, so reviewing tracing code is core work, and NVIDIA bans outside AI in interviews, so the review is by hand (inferred format, medium confidence).

</details>

<a id="q8"></a>

#### Q8. Build an in-process event bus where a slow telemetry exporter can never stall the agent

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Backpressure, event-driven design.

<details>
<summary>Show answer</summary>

Task: many subscribers. An audit subscriber must see every event in order. A telemetry subscriber may lose events, but the producer (the agent loop) must never wait on it. A subscriber that throws must not stop the others.

What they grade: choosing backpressure per subscriber, isolation between subscribers, clean shutdown, and a test that proves the producer is not blocked.

Approach: there are three policies. Blocking (the producer waits for queue space) is lossless but lets the slowest consumer set the agent's speed. Drop-oldest keeps the newest data and never blocks. Drop-newest keeps a contiguous prefix, useful for replay. One global policy is wrong for this mix, so each subscriber gets its own bounded queue, policy and drain task.

Code:

```python
import asyncio

class EventBus:
    def __init__(self):
        self.subs = []

    def subscribe(self, handler, maxsize=1024, policy="drop_oldest"):
        q = asyncio.Queue(maxsize)
        sub = {"q": q, "policy": policy, "dropped": 0, "errors": 0}
        async def pump():
            while True:
                ev = await q.get()
                try:
                    await handler(ev)
                except Exception:
                    sub["errors"] += 1
                finally:
                    q.task_done()
        sub["task"] = asyncio.get_running_loop().create_task(pump())
        self.subs.append(sub)
        return sub

    async def emit(self, ev):
        for s in self.subs:
            if s["policy"] == "block":
                await s["q"].put(ev)
            else:
                if s["q"].full():
                    s["q"].get_nowait(); s["q"].task_done(); s["dropped"] += 1
                s["q"].put_nowait(ev)

    async def close(self):
        for s in self.subs:
            await s["q"].join()
            s["task"].cancel()
        await asyncio.gather(*(s["task"] for s in self.subs), return_exceptions=True)

async def main():
    bus, audit, tele = EventBus(), [], []
    async def audit_h(ev): audit.append(ev)
    async def slow_tele(ev): await asyncio.sleep(0.01); tele.append(ev)
    async def bad(ev): raise ValueError
    bus.subscribe(audit_h, maxsize=2, policy="block")
    t = bus.subscribe(slow_tele, maxsize=4)
    b = bus.subscribe(bad, maxsize=4)
    loop = asyncio.get_running_loop(); t0 = loop.time()
    for i in range(200):
        await bus.emit(i)
    assert loop.time() - t0 < 0.5
    await bus.close()
    assert audit == list(range(200)) and tele[-1] == 199 and t["dropped"] > 150 and b["errors"] > 0

asyncio.run(main())
```

Verify: 200 events through a 10 ms telemetry handler would take 2 seconds if blocking; with drop-oldest the emit loop finishes in milliseconds, audit gets all 200 in order, telemetry ends on the newest event, `dropped` reports the loss, and the throwing subscriber does not affect the others.

Pitfalls:
- A blocking audit subscriber with a slow handler still stalls the agent; it needs its own budget or a durable spill to disk.
- Forgetting `task_done` on a dropped item, which makes `join` hang at shutdown.
- No `dropped` metric, so loss is invisible.

Follow-ups: in a Rust core the same shape is a bounded `tokio::sync::mpsc` per subscriber with `try_send` for lossy and `send().await` for lossless subscribers; `broadcast` is the alternative, but all receivers share one ring buffer. A slow receiver gets `RecvError::Lagged` and silently loses events, and you cannot make one subscriber lossless while another is lossy. How would close work with a deadline?

Why this is likely: the listing asks for event-driven systems, and Blind replies on NVIDIA senior loops say interviewers push on latency against throughput and failure handling; medium confidence.

</details>

### Agent runtime depth

<a id="q9"></a>

#### Q9. This support agent loops between two tools in 18% of sessions. Read the trace, find the cause and add a detector

Round: Domain deep dive. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Agent traces, tool errors, loop guards.

<details>
<summary>Show answer</summary>

Task: three failing traces look like this one (trimmed). Find the root cause, fix it, and add a runtime guard.

```text
{"type":"tool_call","tool":"search_kb","args":{"q":"refund status"}}
{"type":"tool_result","tool":"search_kb","result":{"docs":["refund-policy"]}}
{"type":"tool_call","tool":"get_ticket","args":{"id":881}}
{"type":"tool_result","tool":"get_ticket","result":{}}
{"type":"llm","text":"I could not find the ticket details. Let me search again."}
{"type":"tool_call","tool":"search_kb","args":{"q":"refund status"}}
{"type":"tool_result","tool":"search_kb","result":{"docs":["refund-policy"]}}
{"type":"tool_call","tool":"get_ticket","args":{"id":881}}
{"type":"tool_result","tool":"get_ticket","result":{}}
{"type":"tool_call","tool":"search_kb","args":{"q":"refund status"}}
{"type":"tool_result","tool":"search_kb","result":{"docs":["refund-policy"]}}
{"type":"tool_call","tool":"get_ticket","args":{"id":881}}
{"type":"tool_result","tool":"get_ticket","result":{}}
{"type":"end","reason":"max_steps"}
```

Ticket service log for the same session (trimmed):

```text
2026-08-01T10:00:02Z ticket-svc GET /tickets/881 403 missing scope tickets.refunds req=7f3a1
2026-08-01T10:00:05Z ticket-svc GET /tickets/881 403 missing scope tickets.refunds req=7f3a2
2026-08-01T10:00:08Z ticket-svc GET /tickets/881 403 missing scope tickets.refunds req=7f3a3
```

What they grade: reading the trace for the first wrong signal, joining it with service logs, fixing the root cause, and proving the fix.

Approach: `get_ticket` returns `{}` every time. Join the trace to the service log by request ID. The log shows 403 because the agent's service token lacks the scope for that ticket queue. The tool wrapper turns any non-200 into an empty result (the same pattern as Q6), so the model reads it as "no data yet" and repeats the plan with identical arguments.

Fixes: return a structured error (`{"ok": false, "error": "403 forbidden: missing scope tickets.refunds"}`) so the model can tell the user or escalate; fix the token scope; add a guard that stops a repeated cycle of identical calls and hands off.

Code:

```python
import json

def find_loop(events, min_repeats=3):
    calls = [(e["tool"], json.dumps(e["args"], sort_keys=True)) for e in events if e["type"] == "tool_call"]
    for period in (1, 2, 3):
        for start in range(len(calls) - period * min_repeats + 1):
            cycle = calls[start:start + period]
            if all(calls[start + k * period:start + (k + 1) * period] == cycle for k in range(min_repeats)):
                return {"start": start, "cycle": [c[0] for c in cycle]}
    return None

tr = [{"type": "tool_call", "tool": "search_kb", "args": {"q": "refund status"}},
      {"type": "tool_result", "tool": "search_kb", "result": {"docs": ["refund-policy"]}},
      {"type": "tool_call", "tool": "get_ticket", "args": {"id": 881}},
      {"type": "tool_result", "tool": "get_ticket", "result": {}}] * 3
assert find_loop(tr) == {"start": 0, "cycle": ["search_kb", "get_ticket"]}
assert find_loop(tr[:8]) is None
assert find_loop([{"type": "tool_call", "tool": "search_kb", "args": {"q": f"q{i}"}} for i in range(6)]) is None
```

Verify: two rounds are allowed, because one retry after a transient error is normal; new arguments count as progress. Rerun the failing sessions plus a regression set and check the loop rate falls toward zero while the resolution rate holds.

Pitfalls:
- Only raising `max_steps`, which already exists and fires after the session wasted its budget; the cycle check fires after about 6 calls instead of 30.
- Prompting the model "do not repeat yourself" while the tool still hides the 403.

Follow-ups: how do you find this pattern across a million traces (group by tool and empty-result rate)? What should the user see when the guard trips?

Why this is likely: the listing names agent harnesses, tool calls and observability, and NeMo Relay emits agent trajectory events; the scenario is adapted from a generic agent trace-debug format, not an NVIDIA report (low to medium confidence).

</details>

<a id="q10"></a>

#### Q10. Design the evaluation that proves a new context-compaction middleware does not hurt agent task success

Round: Domain deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Agent eval design, power analysis.

<details>
<summary>Show answer</summary>

Task: the middleware summarizes old turns when the context passes 60% of the window, and it cut tokens per task by about 35% in testing. Product wants proof that task success does not fall by more than 3 points from a 70% baseline.

What they grade: a paired design, a sample size you can defend, slices where the change should hurt, and a trustworthy grader.

Approach: run the same task set with and without the middleware, same seeds and model version, and score with deterministic checks where possible (did the change pass its tests, did the API end state match). Use an LLM judge only for tasks without a checkable end state. Pairing is far cheaper, because most tasks succeed or fail either way.

Code:

```python
from statistics import NormalDist

# One-sided non-inferiority, margin 3 points, powered for a true drop of 0.
def n_per_arm(p0, margin, alpha=0.05, power=0.8):
    z = NormalDist().inv_cdf(1 - alpha) + NormalDist().inv_cdf(power)
    return int(z ** 2 * 2 * p0 * (1 - p0) / margin ** 2) + 1

def paired_n(discordant_rate, margin, alpha=0.05, power=0.8):
    z = NormalDist().inv_cdf(1 - alpha) + NormalDist().inv_cdf(power)
    return int(z ** 2 * discordant_rate / margin ** 2) + 1

print(n_per_arm(0.70, 0.03), paired_n(0.10, 0.03))
assert paired_n(0.10, 0.03) == 687 and n_per_arm(0.70, 0.03) == 2886
```

Numbers: use a one-sided non-inferiority test with a 3 point margin, since a test that misses a drop is no proof the drop is absent. With alpha 0.05 one-sided, power 0.8 and a true drop of 0, independent samples need about 2,886 episodes per arm. If about 10% of tasks flip outcome between versions, the paired test needs about 687 tasks. The flip rate comes from a pilot, and agent nondeterminism raises it, so run each task twice per arm and check that the flip rate between two runs of the same arm is well below the flip rate across arms.

Verify: report slices next to the average. Compaction should hurt long tasks most, so include a few hundred tasks that cross the 60% threshold and report success by length bucket. Add targeted probes: facts given early and needed late, and tool results referenced by ID many turns later. Check the LLM judge against about 200 human labels and use it only if agreement stays above about 85% on both arms.

Pitfalls:
- An average over mostly short tasks that never trigger compaction, which hides the effect.
- A judge that prefers shorter contexts and biases the result.

Follow-ups: ship rule: the upper bound of the one-sided 95% confidence interval on the drop is below 3 points, no slice drop above 5 points, and the token saving confirmed on the same runs. How do you keep this eval from drifting as the model changes?

Why this is likely: the listing asks for evaluation and benchmarking of agent workflows, and NVIDIA's NeMo Agent Toolkit course centers on agent evaluation and profiling (low to medium confidence on the round format).

</details>

<a id="q11"></a>

#### Q11. Python calls into the Rust core on every event. Where does the time go, and how do you make it cheap?

Round: Domain deep dive. AI policy: Not allowed. Type: Optimize. Difficulty: Hard. Topic: FFI cost, PyO3, batching.

<details>
<summary>Show answer</summary>

Task: a profile shows 18% of agent CPU in the binding layer at 50,000 events per second. Explain the costs and choose a fix.

What they grade: a cost model with numbers, knowing what crosses the boundary (call, conversion, GIL, allocation), and matching the fix to the event's latency needs.

Approach: measure a bare foreign call first, then account for the rest.

Code:

```python
import ctypes, time
libc = ctypes.CDLL(None)
labs = libc.labs
labs.argtypes, labs.restype = [ctypes.c_long], ctypes.c_long
n = 200_000
t0 = time.perf_counter()
for i in range(n):
    labs(-i)
print(f"ctypes call: {(time.perf_counter() - t0) / n * 1e6:.2f} us per call")
for name, us in [("per-event at 1 us", 1.0), ("per-event at 5 us", 5.0), ("batch of 256 at 20 us", 20 / 256)]:
    print(f"{name}: {50_000 * us / 1e6 * 100:.1f}% of one core")
```

Numbers: the bare call costs about 0.2 microseconds on a laptop. Real binding calls cost more, because each event converts Python objects into Rust types (dict to struct, str to UTF-8), may acquire or release the GIL, and may allocate. At 1 to 5 microseconds per event, 50,000 events a second costs 5% to 25% of a core, which matches the profile.

Alternatives:
1. Batch. Python appends to a local list and crosses into Rust every 256 events or 10 ms, cutting call overhead per event by about 100x. Cost: up to 10 ms extra delivery latency, fine for telemetry, wrong for middleware that must run before the call proceeds.
2. Cheaper conversion. Pass pre-encoded bytes, or use typed classes defined in Rust so fields are already native.
3. Release the GIL during heavy Rust work. This helps other threads but does not cut per-call cost.
4. Move the loop. If Python only forwards provider stream chunks, let the core own the stream and hand Python finished results (Q15).

Choice: batching for observational events, typed Rust-backed classes for events synchronous middleware must see.

Verify: rerun the same profile, aiming for binding overhead under 2% of CPU, and add a microbenchmark to CI so a regression shows up as a number.

Pitfalls:
- Batching events that middleware must see synchronously.
- Optimizing the call while the real cost is dict-to-struct conversion.

Follow-ups: what changes with free-threaded Python? How does Node.js N-API compare?

Why this is likely: the listing asks for performance across language bindings and serialization, and NeMo Relay's repo guidance names Rust as the runtime with Python bindings (inferred team match, medium confidence).

</details>

<a id="q12"></a>

#### Q12. Sub-agents running concurrently get the wrong parent spans. Explain why and fix it

Round: Domain deep dive. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Context propagation across async tasks.

<details>
<summary>Show answer</summary>

Task: the tracing scope stack is a module-level list. With two sub-agents started by `asyncio.gather`, the recorded parents are wrong.

```python
import asyncio

STACK, records = [], []

class scope:
    def __init__(self, name):
        self.name = name
    def __enter__(self):
        records.append((self.name, STACK[-1] if STACK else None))
        STACK.append(self.name)
    def __exit__(self, *exc):
        STACK.pop()

async def sub_agent(name):
    with scope(name):
        await asyncio.sleep(0.01)
        with scope(f"{name}.llm"):
            await asyncio.sleep(0)

async def main():
    with scope("root"):
        await asyncio.gather(sub_agent("a"), sub_agent("b"))

asyncio.run(main())
print(dict(records))
```

What they grade: explaining the interleaving precisely, a per-task fix, and knowing how each binding handles it.

Approach: running it prints `{'root': None, 'a': 'root', 'b': 'a', 'a.llm': 'b', 'b.llm': 'a.llm'}`. Both tasks push onto one shared list, so whichever ran last decides the parent. The fix is per-task state. `contextvars` gives each asyncio task a copy of the context at creation, so siblings start with the same parent and cannot see each other's pushes. Store the stack as an immutable tuple so a child never mutates its parent's view.

Code:

```python
import asyncio, contextvars

_stack = contextvars.ContextVar("scope_stack", default=())
records = []

class scope:
    def __init__(self, name):
        self.name = name
    def __enter__(self):
        parent = _stack.get()[-1] if _stack.get() else None
        records.append((self.name, parent))
        self.token = _stack.set(_stack.get() + (self.name,))
        return self
    def __exit__(self, *exc):
        _stack.reset(self.token)

async def sub_agent(name):
    with scope(name):
        await asyncio.sleep(0.01)
        with scope(f"{name}.llm"):
            await asyncio.sleep(0)

async def main():
    with scope("root"):
        await asyncio.gather(sub_agent("a"), sub_agent("b"))

asyncio.run(main())
assert dict(records) == {"root": None, "a": "root", "b": "root", "a.llm": "a", "b.llm": "b"}
```

Verify: the assert above, plus a thread started inside a scope and a deeply nested gather.

Pitfalls:
- Node.js needs `AsyncLocalStorage`; Rust needs the parent passed explicitly or a task-local, because a thread-local breaks when Tokio moves tasks between threads.
- Threads started from Python need `contextvars.copy_context().run(...)`, because a new thread starts with an empty context.
- Using a mutable list inside the ContextVar, which reintroduces sharing.

Follow-ups: passing a scope handle explicitly is the most reliable alternative, but it changes every user-facing signature, so use implicit context with an explicit override. How do you link spans across a process boundary?

Why this is likely: NeMo Relay's guidance says scope stacks establish ownership and event parentage across bindings, and the listing asks for async programming and cross-language APIs; medium confidence.

</details>

<a id="q13"></a>

#### Q13. One runtime ships Rust, Python, Node.js and Go bindings. How do you keep behavior consistent and avoid breaking users?

Round: Domain deep dive. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Cross-language API design, versioning.

<details>
<summary>Show answer</summary>

Task: the Rust core is the source of truth. Python and Node.js are primary bindings; Go and a raw C interface are experimental. A field renamed in Rust broke Node.js users silently, while Python raised an error.

What they grade: finding the process cause, a contract that fails the build, conformance testing, and a versioning policy users can trust.

Approach: each binding hand-maps the core types, so a rename lands differently in each. Fix the process in three parts.

Contract. Define public types and events once, in Rust or a schema, and generate binding stubs (Python type stubs, TypeScript declarations, Go structs) from it. A rename then fails the build in every binding, including the lenient ones.

Conformance tests. One shared set of scenario files (inputs and expected event sequences) runs against every binding in CI. When a public behavior changes in Rust, the scenario changes, and every binding that exposes it must pass.

Versioning. Semantic versioning, all bindings released together under one version, so users need no compatibility matrix. Keep an old field name as an alias, warn in each language's normal way, and list it in the changelog. The event schema carries its own version for exporters.

Alternatives: hand-written bindings with conformance tests give each language a more natural API, but drift is caught only as well as tests cover it; fully generated bindings never drift but feel foreign. Generate the data types and hand-write the small idiomatic layer on top (context managers in Python, promises in Node.js).

Risks: async models diverge most. Rust futures map to Python awaitables and Node.js promises, and cancellation maps to `CancelledError` in Python and `AbortSignal` in Node.js; those mappings belong in the conformance scenarios. Releasing in lockstep means a Go-only fix waits for a full release; patch releases can cover that.

Numbers: codegen covers about 60 public types and 25 event kinds; 120 conformance scenarios x 4 bindings at about 2 seconds each is about 16 minutes, about 4 minutes in parallel per binding. Deprecated names stay 2 minor releases (about 12 weeks).

Follow-ups: how do you mark the experimental Go binding so users do not depend on it? How do you deprecate an event type consumers already parse?

Why this is likely: NeMo Relay's repository guidance says a public runtime change requires identifying which bindings expose it and keeping them aligned, and the listing asks for cross-language APIs with backwards compatibility; medium confidence.

</details>

### System design

<a id="q14"></a>

#### Q14. Design the telemetry path for an agent runtime so tracing never slows or breaks the agent

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Observability pipeline, sampling.

<details>
<summary>Show answer</summary>

Task: the runtime emits lifecycle events (scope start and end, LLM call, tool call, middleware decisions) from Rust, Python and Node.js hosts. Users export to OpenTelemetry backends and to files. Size it for 2,000 concurrent agents.

What they grade: volume math, a hot path that cannot block, a sampling choice that keeps the traces people need, and failure behavior.

Numbers:

```python
agents, steps_per_s, events_per_step, bytes_per_event = 2_000, 5, 6, 600
eps = agents * steps_per_s * events_per_step
print(f"{eps:,} events/s, {eps * bytes_per_event / 1e6:.1f} MB/s, {eps * bytes_per_event * 86400 / 1e12:.2f} TB/day")
for name, keep in [("keep all", 1.0), ("head sample 10%", 0.10), ("tail: errors + p99 + 5%", 0.07)]:
    print(f"{name}: {eps * bytes_per_event * 86400 * keep / 1e12:.2f} TB/day")
```

That is 60,000 events per second and about 3.1 TB a day unsampled. Prompts and outputs make events large, which is why the 600 byte average matters.

Approach: the agent thread only appends a compact event to a bounded per-process buffer (Q8's drop-oldest policy) and returns. Serialization, batching and export run on a separate thread or task. Capture overhead stays under about 1 microsecond per event and under 1% of agent CPU. Full prompts are recorded as references or truncated by default, with opt-in full capture, because they are both the bulk of the bytes and the main privacy risk.

Alternatives: exporting straight from the library to each backend has fewer moving parts, but every vendor client runs inside the agent process and a slow backend becomes memory growth there. A local OpenTelemetry collector (OTLP over gRPC) handles retries, sampling and fan-out, so the library keeps one exporter. Head sampling is cheap but drops the rare failing run; tail sampling in the collector keeps every error trace, every trace above p99 duration and 5% of the rest, about 7% of traces with a 1% error rate, or about 0.22 TB a day against 0.31 TB for 10% head sampling. Choose the collector and tail sampling, with the runtime marking errors and loop-guard trips on spans.

Risks: when the collector is down, the buffer drops oldest and the `dropped` counter is exported when the link recovers; the agent never waits. Shutdown flushes with a deadline (for example 2 seconds) so a stuck exporter cannot block exit. Tail sampling needs whole traces in one collector, so route by trace ID.

Follow-ups: how do you trace across sub-agents in other processes? What do you redact by default?

Why this is likely: Blind replies on NVIDIA senior system design describe infrastructure prompts that probe observability, latency and failure handling, and NeMo Relay ships OpenTelemetry-style exporters for agent events (medium confidence on the exact prompt).

</details>

<a id="q15"></a>

#### Q15. Design the provider layer that lets one agent harness call OpenAI Chat, OpenAI Responses and Anthropic Messages APIs

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Provider codecs, streaming tool calls.

<details>
<summary>Show answer</summary>

Task: the harness wants one internal request and response type, streaming, tool calls, retries and token accounting across providers whose wire formats differ in tool calls, system prompts and streamed deltas.

What they grade: a canonical model that does not lose data, correct streaming tool-call assembly, error normalization, and where the work runs.

Approach: one canonical internal model (messages with typed content parts: text, tool call, tool result, image) plus a codec per provider that converts both ways. Codecs are pure functions, so they get table-driven tests from recorded provider payloads, including streaming chunks.

The hard parts:
1. Streaming tool calls. Providers stream tool-call arguments as partial JSON fragments with different event shapes. The codec accumulates fragments per tool-call index and emits a complete call only when the provider signals the end of that block. Emitting partial arguments to middleware is a frequent source of bugs.
2. Semantic gaps. Some fields have no equivalent elsewhere (reasoning content, cache-control hints). Keep them as provider-specific extensions instead of dropping them, so a round trip loses nothing.
3. Errors. Map each provider's errors onto a small set: rate limited (with retry-after), overloaded, invalid request, context too long, auth. The retry policy (Q1) keys off the canonical class.

Alternatives: a lowest-common-denominator API is simple but throws away features users want; passing raw payloads through with light wrappers loses nothing but pushes format handling into every middleware. Choose the canonical model with extensions, because guardrails and tracing are then written once.

Numbers: at 2,000 agents streaming about 50 chunks a second, that is 100,000 chunk conversions a second. In Rust, at a few microseconds each, that is a small fraction of a core. Doing it per chunk across a Python binding costs more (Q11), so streaming conversion stays in the core and the binding receives batched deltas.

Risks: providers change formats without notice; nightly contract tests against live sandboxes catch it. Token counts differ by tokenizer, so accounting stores the provider's reported usage, not an estimate.

Follow-ups: how do you add a self-hosted NIM endpoint? How do you fail over between providers mid-conversation?

Why this is likely: a NeMo Relay GitHub issue (#548) discusses middleware and codecs across OpenAI, Anthropic and OCI providers, and the listing asks for model-provider APIs (inferred team match, medium confidence).

</details>

<a id="q16"></a>

#### Q16. Design a platform that runs 10,000 agent benchmark episodes per night against self-hosted model endpoints

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Benchmark scheduling, isolation, reproducibility.

<details>
<summary>Show answer</summary>

Task: teams want nightly benchmark runs of their agents (each episode is 5 to 50 LLM calls plus sandboxed tool use) against GPU-backed model endpoints, with results comparable night to night.

What they grade: capacity math that finds the real bottleneck, a scheduler that fits varied episode sizes, isolation choices with costs, and comparability.

Numbers: 10,000 episodes x about 20 calls x about 3,000 tokens is about 600M tokens a night. At an aggregate 50,000 tokens per second across the endpoint pool, that is about 3.3 hours, so the GPU pool sets the schedule and everything else follows.

Approach: a job API (suite, agent version, model endpoint, seed); a scheduler that admits episodes against a token budget per endpoint, not a fixed concurrency, because episodes differ by 10x in size; sandboxed workers that run the agent and its tools; and a results store of traces, scores and cost.

Alternatives: containers per episode cost a few hundred milliseconds to start and are fine at this rate. MicroVMs isolate untrusted tool code better at a higher start cost. Shared long-lived workers are cheapest but leak state between episodes, which ruins comparability. Use containers, with microVMs only for suites that execute model-written code.

Risks: comparability drifts. Pin endpoint versions, set temperature and seeds where supported, record every LLM response so a failed episode replays without GPU cost, and run a fixed control agent nightly; if the control's score moves, the platform or endpoint changed, not the agents. Endpoint errors retry with backoff and do not count as agent failures. Episodes are idempotent and checkpointed by step, so a lost worker resumes. Per-team quotas stop one large suite from starving the rest.

Follow-ups: how do you report a regression with confidence when scores are noisy (see Q10)? How do you cut cost by reusing recorded responses?

Why this is likely: Blind replies on NVIDIA senior design rounds describe distributed inference or training platforms with scheduling, isolation and observability, and the listing asks for benchmarking agents to find bottlenecks (illustrative numbers, medium confidence).

</details>

### Hiring manager

<a id="q17"></a>

#### Q17. Tell me about a change to a library other people depended on that you had to make with incomplete information. How did you decide, and what happened?

Round: Hiring manager. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Ownership, open-source judgment.

<details>
<summary>Show answer</summary>

Task: tell one specific story with the trade-off, the decision rule and the measured result.

What they grade: how you reduce unknowns, the options you weighed, your own contribution, and what you would change.

Approach: example structure. Situation: our open-source tracing library had one async API, and two large users needed a synchronous one for their frameworks. We could not see most users' code, only download counts and issues. Task: add the sync API without breaking async users and without maintaining two code paths forever.

Action: I wrote a short design note with three options: a separate sync package, a sync wrapper that runs the async core on a private event loop, or a sync core with async wrappers. I picked the wrapper because it reused the tested core, and measured its overhead (about 15 microseconds per call against a 2 millisecond median call), which the two users accepted. To reduce the unknowns, I searched public repositories that imported the package to see how people used the API, and asked both users to run a release candidate in CI for a week. One found that the private loop broke inside Jupyter, which already runs a loop; we detected that case and raised a clear error pointing to the async API instead of hanging. I used an AI coding assistant to draft the sync wrapper and the test matrix, then reviewed every line. Its first draft called `asyncio.run` inside an already running loop; the Jupyter release-candidate test caught it, and I added that case to the matrix.

Result: shipped in one minor release with no breaking changes, the sync API issue closed, and one framework's adoption brought new contributors.

Pitfalls:
- A story where "we" did everything and your own decision is unclear.
- No numbers, or no result.

Follow-ups: how did you handle disagreement in review? What did you leave out? How did you tell users? What would you change? (Here: publish the release candidate to all users, because the Jupyter case was common.) How do you use AI coding tools day to day, and how do you check their output? (NVIDIA bans AI in the interview itself; this is about daily work.)

Why this is likely: NVIDIA's hiring page and 2026 candidate reports describe hiring manager rounds that dig into past projects and decisions, and the listing asks for open-source library work used by others; medium-high confidence.

</details>

## Prep checklist

- [ ] Write a REST client with retries, backoff with jitter, `Retry-After` and fail-fast 4xx handling from memory, with tests that need no network.
- [ ] Practice multi-part Python parsing and log top-N tasks under time, with malformed lines, ties and out-of-order events.
- [ ] Implement a hash map and an LRU cache without standard containers, and run the C++ one under AddressSanitizer.
- [ ] Rehearse bounded concurrency, cancellation and timeouts in asyncio and in Tokio.
- [ ] Review a pull request by hand for hallucinated APIs, secrets in logs, missing failure paths and ring buffer index errors.
- [ ] Prepare an agent trace debugging story: the symptom in traces, the root cause, the fix and the eval that proved it.
- [ ] Be able to size an agent eval with a paired design and explain the flip rate.
- [ ] Size telemetry volume and choose a sampling strategy with numbers.
- [ ] Know the costs of crossing a Python or Node.js binding, and when to batch.
- [ ] Explain how context propagates across async tasks in Python, Node.js and Rust.
- [ ] Prepare one story about changing a library other people depend on, with the trade-off and the result.
- [ ] Plan to do every round without AI tools and bring a physical photo ID for the onsite.

## Sources

- NVIDIA, How We Hire (official process, AI tool rule, in-person onsite): https://www.nvidia.com/en-us/about-nvidia/careers/how-we-hire
- NeMo Relay repository: https://github.com/NVIDIA/NeMo-Relay
- NeMo Relay repository guidance: https://github.com/NVIDIA/NeMo-Relay/blob/main/AGENTS.md
- NeMo Relay provider codecs issue: https://github.com/NVIDIA/NeMo-Relay/issues/548
- Candidate report, NVIDIA practical coding round (Reddit): https://www.reddit.com/r/AskProgrammers/comments/1ve7bzw/nvidia_swe_interview_2026_a_practical_coding
- Candidate report, NVIDIA senior software engineer process (Reddit): https://www.reddit.com/r/OfferEngineering/comments/1vycmly/nvidia_senior_software_engineer_interview_process
- Candidate report, NVIDIA senior software engineer loop (LeetCode Discuss post 7542688)
- Glassdoor, NVIDIA senior software engineer interviews: https://www.glassdoor.com/Interview/NVIDIA-Senior-Software-Engineer-Interview-Questions-EI_IE7633.0,6_KO7,31.htm
- Blind, NVIDIA senior software engineer interview thread: https://www.teamblind.com/post/interviewing-at-nvidia-for-a-senior-software-engineer-role-ugq0ba2b
- Blind, NVIDIA in-person panel thread: https://www.teamblind.com/post/upcoming-nvidia-in-person-panel-interview-rpu16d4v
- Blind, NVIDIA interviews thread (no standard process across teams): https://www.teamblind.com/post/nvidia-interviews-ocnyi2kd
- calibrd, NVIDIA interview guide: https://www.calibrd.com/interview-prep/nvidia-interview
- prepare.sh, NVIDIA interview process: https://prepare.sh/articles/nvidia-interview-process-2026
- datainterview, NVIDIA software engineer interview: https://www.datainterview.com/blog/nvidia-software-engineer-interview
- DeepLearning.AI, NeMo Agent Toolkit course: https://www.deeplearning.ai/courses/nvidia-nat-making-agents-reliable
- Dataford, NVIDIA agentic AI engineer guide (vendor, low confidence): https://dataford.io/interview-guides/nvidia/agentic-ai-engineer
- Crowd-reported LeetCode company tags, NVIDIA (low confidence, practice links only): https://github.com/liquidslr/leetcode-company-wise-problems/tree/main/Nvidia
