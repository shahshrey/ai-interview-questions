# NVIDIA: Senior Software Engineer, Agentic AI

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

NVIDIA's process is decentralized. Candidates on Blind and interviewing.io describe the general shape as stable, while each team picks its own rounds and questions. Every interviewer usually comes from the hiring team, so expect the questions to track what the team builds. For this role that is agent runtime libraries: a Rust core with Python, Node.js and Go bindings, lifecycle events, middleware, and OpenTelemetry export.

1. Recruiter call, about 30 minutes. Background, level fit (the listing has two level bands), and logistics. Referred candidates sometimes skip it.
2. Technical screen, 45 to 60 minutes. Recent candidate reports describe practical Python more often than puzzles: a multi-part parsing problem, a "call a REST API, validate JSON, handle retries, timeouts and 429s" exercise, and top-N counting over a log stream. Expect a resume deep dive in the same call. NVIDIA says coding usually runs on HackerRank and that using unapproved tools such as ChatGPT disqualifies you.
3. Hiring manager call, 30 to 60 minutes. Past projects, why agent infrastructure, and how you work with open-source users. Some candidates get this call first.
4. Final loop, 4 to 5 rounds of 45 to 60 minutes. NVIDIA requires an in-person onsite before an offer. Based on reports, the mix is likely:
   - Two coding rounds: one LeetCode-medium data structure problem (LRU cache and hashmap-from-scratch appear in NVIDIA reports) and one practical concurrency problem in Python or Rust (rate limiter, bounded executor, backpressure).
   - A system architecture round built around the team's own project. One 2026 senior candidate reported exactly this format.
   - A domain deep dive on runtime internals: asyncio and Tokio, PyO3 boundaries, context propagation, tracing overhead.
   - A values or behavioral round. Some guides say NVIDIA's values come up in every round.
5. Decision. NVIDIA says a few weeks. Candidates report two to three weeks of quiet after the onsite.

## Questions and answers

### Async and concurrency coding

#### Q1. Implement an async token-bucket rate limiter for LLM provider calls.

Problem. Build a limiter that allows `rate` units per second, with bursts up to `capacity`. `acquire(cost)` waits until enough tokens exist and `try_acquire(cost)` returns immediately with a bool. Many coroutines share one limiter on a single asyncio event loop. Waiters should be served roughly in FIFO order. A cost can stand for one request or for an estimated token count when the provider limits tokens per minute.

```python
import asyncio, time

class TokenBucket:
    def __init__(self, rate: float, capacity: float, clock=time.monotonic):
        if rate <= 0 or capacity <= 0:
            raise ValueError("rate and capacity must be positive")
        self.rate = rate
        self.capacity = capacity
        self._tokens = capacity
        self._clock = clock
        self._last = clock()
        self._lock = asyncio.Lock()   # FIFO fairness among waiters

    def _refill(self) -> None:
        now = self._clock()
        self._tokens = min(self.capacity, self._tokens + (now - self._last) * self.rate)
        self._last = now

    def try_acquire(self, cost: float = 1.0) -> bool:
        self._refill()
        if self._tokens >= cost:
            self._tokens -= cost
            return True
        return False

    async def acquire(self, cost: float = 1.0) -> None:
        if cost > self.capacity:
            raise ValueError("cost exceeds bucket capacity")
        async with self._lock:
            while True:
                self._refill()
                if self._tokens >= cost:
                    self._tokens -= cost
                    return
                await asyncio.sleep((cost - self._tokens) / self.rate)
```

A test with `rate=10, capacity=5` and 15 sequential acquires finishes in about 1.0 second. That matches the expected time, because 5 tokens go out as the initial burst and the other 10 refill at 10 per second.

Complexity. Each call does O(1) work apart from sleeping. Refill is lazy and computed from elapsed time, so there's no background timer task.

Why it's built this way. `time.monotonic` doesn't jump when the wall clock changes. The lock makes waiters line up, so a stream of small requests can't starve one large request. `try_acquire` skips the lock because it never awaits, and on a single event loop nothing can interleave inside it. Cancellation is also safe. If a waiter is cancelled during `sleep`, `async with` releases the lock and no tokens are spent.

Edge cases. A cost larger than capacity would wait forever, so reject it. Use a float cost for token-based limits. Injecting the clock makes tests deterministic.

Likely follow-ups:
- Per-tenant or per-model limits: keep a dict of buckets keyed by `(provider, model, api_key)` and evict idle ones.
- Token limits where the true cost is only known after the call: reserve an estimate, then refund or charge the difference when usage comes back.
- Server-side 429s: honor `Retry-After` by pausing the whole bucket, so every waiting call backs off together.
- Several processes: move the state into Redis with a Lua script, or use GCRA, which stores one timestamp per key.
- Threads: `asyncio.Lock` does not protect state across threads. Use one limiter per loop, or a `threading.Lock` and no awaits under it.

Why this is likely: candidate reports from 2026 describe NVIDIA screens that ask about rate limiting and 429 handling when calling an API, and the listing asks for async programming and request lifecycles.

#### Q2. Run N async tool calls with at most K in flight, a per-call timeout, and results returned in input order.

Problem. Input: a list of zero-argument async callables, a concurrency limit `K >= 1`, and a timeout in seconds. Output: a list the same length as the input, where entry `i` holds either the value of call `i` or its exception. The other calls keep running when one fails. Cancelling the whole operation must cancel everything still running.

```python
import asyncio
from dataclasses import dataclass
from typing import Any, Optional

@dataclass
class Result:
    index: int
    value: Any = None
    error: Optional[BaseException] = None

async def run_bounded(calls, limit: int, timeout: float):
    if limit < 1:
        raise ValueError("limit must be >= 1")
    sem = asyncio.Semaphore(limit)
    out = [None] * len(calls)

    async def one(i, fn):
        async with sem:
            try:
                out[i] = Result(i, value=await asyncio.wait_for(fn(), timeout))
            except asyncio.CancelledError:
                raise
            except Exception as e:
                out[i] = Result(i, error=e)

    async with asyncio.TaskGroup() as tg:
        for i, fn in enumerate(calls):
            tg.create_task(one(i, fn))
    return out
```

I tested it with five calls, K=2 and a 0.4 second timeout. Peak concurrency stayed at 2, a 0.5 second call came back as `TimeoutError`, a failing call came back as `RuntimeError`, and every other call kept its result in its own slot.

Complexity. O(N) tasks and O(N) memory. Wall time is about the sum of durations divided by K, bounded below by the slowest call.

Details interviewers look for:
- Errors are caught inside `one`, so the `TaskGroup` never sees them and doesn't cancel the siblings. `CancelledError` is re-raised on purpose. Swallowing it breaks structured concurrency, because the parent never learns that cancellation finished.
- The timeout starts after the semaphore is acquired, so queueing time doesn't count against it. Say this out loud and ask which behavior the interviewer wants.
- On Python 3.11 and later, `asyncio.TimeoutError` is the built-in `TimeoutError`.

Edge cases. Empty input returns `[]`. K larger than N is fine. A callable that raises before its first await is still caught, because `fn()` runs inside `wait_for`. A coroutine that suppresses cancellation can hang the group. Mention it and leave it out of scope.

Likely follow-ups:
- 100,000 calls: creating every task up front costs memory, so switch to K worker tasks pulling from an `asyncio.Queue`.
- A whole-batch deadline: wrap the group in `async with asyncio.timeout(total)`.
- Fail-fast mode: let the exception escape so the `TaskGroup` cancels the rest.
- Streaming results as they finish: use `asyncio.as_completed` and keep the index.
- The Rust version uses `tokio::sync::Semaphore`, `tokio::time::timeout`, and a `JoinSet`, writing results into a pre-sized `Vec` by index.

Why this is likely: the listing names async programming and callbacks as core requirements, parallel tool execution is a standard agent harness feature, and NVIDIA candidate reports in 2026 mention concurrency questions in coding rounds.

#### Q3. Build an LRU cache with TTL, then make concurrent misses for the same key share one backend call.

Problem. Part 1: `get(key)` and `put(key, value)` in O(1), evicting the least recently used entry above `capacity`, with entries expiring `ttl` seconds after they are written. Part 2: an async `get_or_call(key, fn)` where 20 concurrent misses on the same key run `fn` once and every caller gets the result. In an agent harness, this is how you cache embedding or tool results when many sub-agents ask for the same thing.

```python
import asyncio, time
from collections import OrderedDict

class TTLCache:
    def __init__(self, capacity: int, ttl: float, clock=time.monotonic):
        self.capacity, self.ttl, self.clock = capacity, ttl, clock
        self._d = OrderedDict()   # key -> (expires_at, value)

    def get(self, key, default=None):
        item = self._d.get(key)
        if item is None:
            return default
        expires, value = item
        if expires <= self.clock():
            del self._d[key]
            return default
        self._d.move_to_end(key)
        return value

    def put(self, key, value) -> None:
        if self.capacity <= 0:
            return
        self._d[key] = (self.clock() + self.ttl, value)
        self._d.move_to_end(key)
        while len(self._d) > self.capacity:
            self._d.popitem(last=False)

_MISS = object()

class SingleFlight:
    def __init__(self, cache: TTLCache):
        self.cache = cache
        self._inflight = {}

    async def get_or_call(self, key, fn):
        hit = self.cache.get(key, _MISS)
        if hit is not _MISS:
            return hit
        fut = self._inflight.get(key)
        if fut is None:
            fut = asyncio.get_running_loop().create_future()
            self._inflight[key] = fut
            try:
                val = await fn()
                self.cache.put(key, val)
                fut.set_result(val)
            except asyncio.CancelledError:
                fut.cancel()
                raise
            except Exception as e:
                fut.set_exception(e)
                fut.exception()   # mark retrieved to avoid "never retrieved" warnings
                raise
            finally:
                del self._inflight[key]
            return val
        return await asyncio.shield(fut)
```

Tests covered LRU order, expiry with a fake clock, 20 concurrent callers producing exactly one backend call, errors reaching every waiter, and a cancelled waiter leaving the shared call alone.

Complexity. Every operation is O(1). `OrderedDict` is a hash map plus a doubly linked list, the same structure you'd build by hand, and you should be ready to build it by hand. NVIDIA reports include "LRU cache" and "hashmap without STL".

Edge cases. Expiry is lazy, so expired entries keep using memory until someone reads them or they get evicted. Add a periodic sweep or a min-heap of expiry times if that matters. `_MISS` is a sentinel so that cached `None` values still count as hits. `shield` means a waiter that gets cancelled stops waiting without cancelling the shared future.

Likely follow-ups:
- If the leader is cancelled, every waiter fails. Fix it by running `fn` in its own task that the leader also awaits through `shield`.
- Negative caching for errors, with a short TTL.
- Thread safety: one lock per cache, with the backend call made outside the lock.
- Writing the linked list by hand: sentinel head and tail nodes, plus `_remove` and `_add_front` helpers.

Why this is likely: LRU cache shows up repeatedly in NVIDIA candidate reports and company-tagged problem lists (Interview Query, fastprep, Crackr), and the async extension ties it to the agent harness work in the listing.

### Data structures and performance coding

#### Q4. Rebuild span trees from an unordered event stream and compute self time and the critical path of an agent run.

Problem. Input: lifecycle events `{"kind": "start"|"end", "id", "parent", "name", "ts"}` in any order, the way they arrive from several threads or exporters. Output: (a) the root spans, with spans that never closed reported separately, (b) self time per span, meaning time not covered by any child, where children can overlap because tools run in parallel, (c) the critical path: the chain of spans that decided the run's wall time.

```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Span:
    id: str
    parent: Optional[str] = None
    name: str = ""
    start: Optional[float] = None
    end: Optional[float] = None
    children: list = field(default_factory=list)

def build_tree(events):
    spans = {}
    for e in events:                       # events may arrive in any order
        s = spans.setdefault(e["id"], Span(e["id"]))
        if e["kind"] == "start":
            s.parent, s.name, s.start = e.get("parent"), e["name"], e["ts"]
        else:
            s.end = e["ts"]
    roots, incomplete = [], []
    for s in spans.values():
        if s.start is None or s.end is None:
            incomplete.append(s.id)
            continue
        p = spans.get(s.parent) if s.parent else None
        if p is None or p.start is None or p.end is None:
            roots.append(s)                # orphan: parent missing or still open
        else:
            p.children.append(s)
    return roots, incomplete

def self_time(span):
    covered, cur_s, cur_e = 0.0, None, None
    for a, b in sorted((c.start, c.end) for c in span.children):
        if cur_e is None or a > cur_e:
            if cur_e is not None:
                covered += cur_e - cur_s
            cur_s, cur_e = a, b
        else:
            cur_e = max(cur_e, b)
    if cur_e is not None:
        covered += cur_e - cur_s
    return (span.end - span.start) - covered

def critical_path(span):
    path, stack = [], [span]
    while stack:
        s = stack.pop()
        path.append(s.name)
        cursor, chosen = s.end, []
        for c in sorted(s.children, key=lambda c: c.end, reverse=True):
            if c.end <= cursor:
                chosen.append(c)
                cursor = c.start
        stack.extend(chosen)   # latest-first, so pop() expands in time order
    return path
```

Test run: an agent spends 0 to 3 s on `llm1`, runs `toolA` (3 to 5 s) and `toolB` (3 to 9 s) in parallel, then `llm2` (9 to 10 s), and ends at 10.5 s. The critical path is `agent -> llm1 -> toolB -> llm2`, the root's self time is 0.5 s, and an unclosed span is reported as incomplete.

How it works. Self time is interval-union length, the "merge intervals" pattern. The critical path walks backward from the parent's end. At each step it takes the child that finished last before the cursor, then moves the cursor to that child's start. This is the same idea as the nested-stack bottleneck analysis in NVIDIA's NeMo Agent Toolkit profiler.

Complexity. Building the tree is O(E). Self time and the critical path are O(c log c) per span for sorting, so O(S log S) overall. The traversal is iterative, so a deep tree of nested sub-agents can't hit Python's recursion limit.

Edge cases. An end that arrives before its start. A span whose parent was dropped by a full exporter queue: promote it to a root so it stays in the output. Clock skew between processes: prefer monotonic timestamps from a single clock, or record a per-process offset. Duplicate events from at-least-once delivery: dedupe on `(id, kind)`.

Likely follow-ups:
- Streaming version with bounded memory: keep only open spans and finalize subtrees when their root closes.
- Detecting idle gaps inside the critical path, where the harness was waiting on itself.
- Attributing tokens and cost along the path.

Why this is likely: the listing asks for benchmarking agents to find bottlenecks, NVIDIA's NeMo Agent Toolkit ships exactly this kind of nested-stack analysis, and candidate reports describe practical parsing problems in screens.

#### Q5. This Python event emitter costs 7 microseconds per event on the hot path. Make it fast without changing what subscribers see.

Problem. Every LLM and tool callback in a harness calls `emit`. Profiling shows the instrumentation adding measurable end-to-end latency. Requirements: lower the per-call cost, keep emission order, keep I/O off the caller's path, and bound memory.

```python
import json, threading, time, uuid
from collections import deque

class NaiveEmitter:
    def __init__(self, sink):
        self.lock = threading.Lock()
        self.subscribers = []
        self.sink = sink

    def subscribe(self, fn):
        self.subscribers.append(fn)

    def emit(self, kind, name, payload):
        event = {"id": str(uuid.uuid4()), "ts": time.time(),
                 "kind": kind, "name": name, "payload": payload}
        line = json.dumps(event)              # serialize on the hot path
        with self.lock:                       # lock held during I/O
            self.sink.write(line + "\n")
            for fn in self.subscribers:
                fn(json.loads(line))          # every subscriber re-parses

class Event:
    __slots__ = ("seq", "ts_ns", "kind", "name", "payload")
    def __init__(self, seq, ts_ns, kind, name, payload):
        self.seq, self.ts_ns, self.kind, self.name, self.payload = seq, ts_ns, kind, name, payload

class FastEmitter:
    def __init__(self, sink, max_queue=65536, batch=512):
        self._q = deque()
        self._max = max_queue
        self._batch = batch
        self._seq = 0
        self._subs = ()                       # immutable tuple: copy-on-write
        self._sink = sink
        self.dropped = 0
        self._ns = time.perf_counter_ns       # bind hot lookups once

    def subscribe(self, fn):
        self._subs = self._subs + (fn,)

    def emit(self, kind, name, payload):
        q = self._q
        if len(q) >= self._max:               # bounded: drop, never block the caller
            self.dropped += 1
            return
        self._seq += 1
        q.append(Event(self._seq, self._ns(), kind, name, payload))
        if len(q) >= self._batch:
            self.flush()

    def flush(self):
        q, subs, dumps = self._q, self._subs, json.dumps
        out = []
        for _ in range(len(q)):
            e = q.popleft()
            for fn in subs:
                fn(e)                         # subscribers get the object, no re-parse
            out.append(dumps((e.seq, e.ts_ns, e.kind, e.name, e.payload)))
        if out:
            self._sink.write("\n".join(out) + "\n")   # one write per batch
```

Measured on CPython 3.14 with a null sink, two subscribers and 200,000 events: 7.16 microseconds per event naive, 1.65 microseconds optimized, about 4.3 times faster. Most of the gain comes from serializing once per batch where the naive version serialized and then re-parsed JSON for every subscriber, from using a sequence number in place of UUID4, and from making one sink write per batch.

What changed and why:
- `uuid4` reads OS randomness, and a per-process counter is enough for ordering and dedupe.
- `__slots__` objects allocate less than dicts.
- Subscribers get the object instead of a re-parsed copy, so the contract becomes "events are immutable". Write that down, because a subscriber that mutates an event now affects the others.
- Snapshotting the subscriber tuple removes the lock from the read path.
- `perf_counter_ns` is monotonic. Keep one wall-clock anchor per batch if exporters need epoch time.

Complexity. O(1) amortized per emit. Memory is bounded by `max_queue`.

Likely follow-ups:
- Flush still runs on the caller's thread when a batch fills. Move it to a background thread or an asyncio task, and cap the time per flush.
- Thread safety: `deque.append` and `popleft` are atomic under the GIL, but `self._seq += 1` is not. On free-threaded 3.14, use `itertools.count()` or a lock.
- Wrap each subscriber call in try/except so one bad subscriber can't break the rest or the flush.
- Report `dropped` as a metric.
- The next step is moving the queue and serialization into Rust behind PyO3 (see Q7). The emit path then becomes one boundary crossing that only stores Python references.

Why this is likely: the listing asks for profiling and optimizing library overhead "across ... serialization, tracing, or middleware pipelines" and for instrumenting frameworks "without changing user-visible behavior".

### Runtime internals

#### Q6. Your tracing wrapper around a streaming LLM call parents spans under the wrong agent when sub-agents run concurrently. Explain why and fix it.

OpenTelemetry Python stores the current span in a `contextvars.ContextVar`. Each asyncio task runs inside a copy of the context taken when the task was created. `tracer.start_as_current_span` sets the variable on entry and resets it with a token on exit. A coroutine is fine, because the set and the reset happen in the same task. An async generator breaks this. If you set the span and then `yield` while it is still set, control returns to the consumer with your span still current. Any span the consumer starts between chunks gets parented under the stream instead of under its own agent.

I reproduced it in a small test. A leaky wrapper sets the variable, then does `async for c in agen: yield c`. The consumer sees `'llm.stream'` as the current value on every chunk, when it should see its own context. When the generator is resumed from a different task (common under a `TaskGroup`, or when a framework hands the stream to another task), `reset(token)` raises `ValueError: Token ... was created in a different Context`. A Google ADK issue from 2026 documents this exact bug with OpenTelemetry. A first fix using `context.attach`/`detach` around yields was reverted two days later, because `detach` raised during cancellation.

The fix that holds up is to make each step of the generator self-contained. Set the context around each `__anext__` call and reset it before yielding:

```python
from opentelemetry import context, trace

async def traced_stream(agen, span):
    it = agen.__aiter__()
    try:
        while True:
            token = context.attach(trace.set_span_in_context(span))
            try:
                chunk = await it.__anext__()
            except StopAsyncIteration:
                break
            finally:
                context.detach(token)
            yield chunk
    finally:
        span.end()
```

The span object outlives the yields, but it is only "current" while upstream code runs. With this version the consumer sees its own context. Related points an expert would bring up:

- Start a span and make it current in separate steps (`start_span` versus `start_as_current_span`) when the lifetime doesn't match a lexical block.
- End the span in `finally`, and handle the consumer abandoning the stream: `aclose()` raises `GeneratorExit` inside the generator.
- Threads don't inherit contextvars. `run_in_executor` needs `contextvars.copy_context().run`, and `asyncio.to_thread` already does this.
- Callback-style frameworks such as LangChain handlers get start and end callbacks, often on different tasks. Keep an explicit `run_id -> span` map and parent spans by id, without relying on the ambient context.
- In Rust `tracing`, the same bug is holding a `Span::enter()` guard across `.await`. Use `future.instrument(span)` instead.

Why this is likely: the listing asks for instrumenting third-party frameworks "without changing user-visible behavior" and for OpenTelemetry and tracing experience, and this failure mode is publicly documented in a major agent framework.

#### Q7. Where does time go when Python calls into a Rust core through PyO3, and how do you make a hot path cheap?

The costs, from biggest to smallest in the cases I've seen:

1. Conversion at the boundary. `Vec<String>` or `HashMap` arguments copy every element out of Python objects. A `dict` payload converted into a Rust struct on every event can cost more than the work itself. Fixes: accept `&Bound<'_, PyAny>` and convert lazily, keep payloads as `Py<PyAny>` until an exporter needs them, or take `bytes` or buffer-protocol objects for bulk data.
2. Error construction. The PyO3 performance guide notes that chained `extract` calls build a `PyErr` for every failed attempt. When you only need a type test, `cast` skips converting `PyDowncastError` into a `PyErr`.
3. String allocation. Each `&str` to Python conversion allocates a new `PyString`. The `intern!` macro caches attribute names and dict keys at each call site.
4. Calling back into Python. PyO3 uses the faster `vectorcall` protocol when arguments are a Rust tuple and falls back to `tp_call` for `Bound<PyTuple>`. Subscribers written in Python and invoked from Rust pay this on every event.
5. Interpreter attachment. With the GIL, only one attached thread runs Python at a time. Wrap long Rust-only work in `Python::detach` (named `allow_threads` in older PyO3) so other Python threads keep running. On the free-threaded build, which CPython 3.14 officially supports, detaching still matters because stop-the-world events such as GC wait for every attached thread.
6. Reference counting. Dropping a `Py<T>` while detached pushes the decref into a global reference pool, which is synchronized and drained on the next attach. The guide lists the `pyo3_disable_reference_pool` flag to remove that cost, with the risk of an abort if you drop while detached.

Design rules that follow:
- Cross the boundary once per batch or per lifecycle event, passing all fields in that one call.
- Keep state in Rust behind `#[pyclass]` handles.
- Do serialization and export on Rust threads, with Tokio living in the extension. Bridge to asyncio with something like `pyo3-async-runtimes` so Python gets an awaitable without blocking the event loop.

On the free-threaded build, `#[pyclass]` types must be `Sync` or marked `unsendable`. Shared mutable state needs real locks, because the GIL no longer serializes access.

How to measure: `criterion` for the pure Rust function, `pyperf` or `pytest-benchmark` for the Python call, and `perf` or `py-spy --native` to see whether time sits in conversion, in `PyO3` glue, or in your code. Report nanoseconds per call for an empty function, for a typical payload, and for a batch of 512. Those three numbers tell you whether batching is worth its added latency and its shutdown-flush complexity.

Why this is likely: the listing explicitly names PyO3, maturin and Python/Rust bindings, and "profiling or optimizing runtime and library overhead across language boundaries".

#### Q8. In a Tokio-based event pipeline, how do you apply backpressure, and what breaks with cancellation and blocking?

Backpressure means slow consumers push back on producers before memory grows without limit. In Tokio:

- `mpsc::channel(n)` is bounded. `send().await` parks the producer when the buffer is full. Inside it's a semaphore of permits, and Tokio's async semaphore keeps waiters in an intrusive linked list, so waiting doesn't allocate.
- `try_send` returns `Full` right away, so the caller can drop, sample or count.
- `unbounded_channel` has no backpressure at all. Use it only when the producer is already bounded by something else.
- `Semaphore` limits concurrent work, such as in-flight LLM calls per provider, without a queue between stages.

The design question is which side pays when the queue fills. The agent's speed comes first, so telemetry takes the loss. OpenTelemetry's spec says `Span.End()` must not do blocking I/O, and BatchSpanProcessor drops spans when the queue is full by default. A 2026 spec issue proposing an opt-in "block on overflow" mode drew pushback for exactly this reason: blocking inside a single-threaded event loop (asyncio or Node) stalls the application. So the emit path uses `try_send` and counts drops:

```rust
use std::sync::atomic::{AtomicU64, Ordering};
use tokio::sync::mpsc::{self, error::TrySendError};

// Event and export_batch are defined elsewhere in the crate.
static DROPPED: AtomicU64 = AtomicU64::new(0);

fn emit(tx: &mpsc::Sender<Event>, ev: Event) {
    match tx.try_send(ev) {
        Ok(()) => {}
        Err(TrySendError::Full(_)) => { DROPPED.fetch_add(1, Ordering::Relaxed); }
        Err(TrySendError::Closed(_)) => {}
    }
}

async fn exporter(mut rx: mpsc::Receiver<Event>) {
    let mut buf = Vec::with_capacity(512);
    while rx.recv_many(&mut buf, 512).await > 0 {
        export_batch(&buf).await;
        buf.clear();
    }
}
```

Work that must not be lost, like a guardrail decision or a tool result, gets `send().await` so the caller waits.

Cancellation. Dropping a future cancels it at its last `.await`. Inside `tokio::select!`, a branch that loses gets dropped, so every branch must be cancel safe. `mpsc::Receiver::recv` is cancel safe. Reading into a partial buffer across an await, or popping an item into a local variable before an await, is not, and the data is lost. Async Rust has no async `Drop`, so cleanup that needs I/O (flushing an exporter) needs an explicit `shutdown().await` with a deadline, and a `Drop` that only signals.

Blocking. A synchronous call inside a task stalls that worker thread and every task queued on it. Examples: a big `serde_json` serialization, file I/O, or calling into Python. Use `spawn_blocking` or a dedicated thread. Under PyO3, release Python attachment before any `.await`.

Tracing. `Span::enter()` across an await misattributes events. Use `.instrument(span)`.

Why this is likely: the listing names async Rust, Tokio and runtime state management, and the task mix calls for backpressure. The OpenTelemetry drop-versus-block debate is current and shows the tradeoff directly.

#### Q9. You maintain one runtime with Rust, Python, Node.js and Go bindings. How do you keep the API consistent and backward compatible?

Start from one source of truth for behavior. NVIDIA's open-source NeMo Relay, which matches this listing closely, states that Rust is the source of truth for runtime behavior, Python and Node.js are the primary bindings, and Go and the raw C FFI are experimental. It also asks contributors to check parity with the other bindings whenever they add behavior to one. Adopt the same policy.

Layering:
- A Rust core crate with the real semantics: scopes, events, middleware ordering, error types.
- A stable C ABI made of opaque handles, `extern "C"` functions, explicit ownership (`*_new` and `*_free`), integer error codes plus a "last error" accessor, and only C-compatible types crossing the boundary. Go uses this through cgo.
- Native bindings for Python (PyO3) and Node (napi-rs) that call the core directly and skip the C layer, which avoids double conversion.
- Payloads crossing the boundary are JSON-compatible values or a versioned schema. Relay's events carry an `atof_version` field for this reason.

Where the languages differ:
- Async. Rust futures need a runtime. Python wants awaitables on its own loop. Node wants Promises, and callbacks from Rust worker threads must go through a `ThreadsafeFunction` back to the JS main thread. Go wants blocking calls in goroutines. Define the async contract once (what runs where, what can block, what happens on cancel) and map it into each binding.
- Callbacks into the host language. Each call costs a boundary crossing plus lock acquisition (GIL, the JS event loop, Go's cgo callback). Batch, or let hosts poll.
- Errors. Map a closed set of core error kinds to idiomatic types: Python exception subclasses, JS `Error` with a `code`, Go `error` values. Wrap every exported function in `catch_unwind` so a panic stops at the FFI boundary.
- Memory. Go's cgo rules forbid C keeping Go pointers. Python objects held in Rust need the attach rules from Q7.

Compatibility practice:
- Semver per package, with versions released in lockstep.
- Additive changes first: new optional fields, new event kinds that old subscribers ignore. Readers tolerate unknown fields.
- Deprecation warnings in every language for at least one minor release before removal.
- A shared conformance suite of golden event streams, run against all bindings in CI, so "same input, same events" is checked by tests.
- API diff checks: `cargo-semver-checks` for Rust, plus stub or `.d.ts` diffs for Python and Node.

Why this is likely: the listing requires "design or extend cross-language APIs with attention to consistency, usability, stability, and backwards compatibility", and the team's public repo ships exactly this binding set.

### System design

#### Q10. Design a framework-agnostic agent runtime library that captures scopes and lifecycle events from LangChain, LangGraph, and custom harnesses, with a Rust core.

Requirements. Plugs into existing frameworks as they are. Captures nested scopes (session, turn, agent, LLM call, tool call, sub-agent). Lets middleware observe or control calls. Exports to OpenTelemetry, raw JSONL and trajectory formats. Overhead budget: under about 5 microseconds per event on the caller's thread, and under 1 percent of end-to-end latency. LLM calls take 0.5 to 30 seconds, so the budget is mostly about CPU at high event rates. Expect 1,000 to 10,000 events per second per process for busy multi-agent runs.

Components:
1. Scope stack. A tree of scopes with ids and parent ids. The active scope is tracked per task (Python contextvars, Node AsyncLocalStorage, Tokio task-local). Scope-local registrations disappear when the scope closes.
2. Three integration modes, chosen by where the real call happens. Managed execution: the runtime invokes the tool or LLM function itself, so middleware can block or rewrite. Lifecycle hooks: the framework keeps control and reports start and end through callbacks, so the runtime observes only. Gateway: a local proxy for provider traffic when neither hook exists. NeMo Relay documents exactly these three modes.
3. Middleware chain. See Q12.
4. Event bus. Canonical, versioned lifecycle events go into a bounded queue. Subscribers and exporters consume off the hot path. NVIDIA's NeMo Agent Toolkit uses the same subject-observer shape, with an `IntermediateStepManager` publishing to a reactive stream and async exporters.
5. Exporters. OpenTelemetry using the GenAI semantic conventions (`gen_ai.*` attributes, agent and tool spans), OpenInference, raw event JSONL, and trajectory files for replay and evals.
6. Plugin config. Declarative files that install middleware and exporters, so users enable features without code changes.

Data flow: framework callback -> adapter maps it to a core call -> core updates scope state, runs the middleware snapshot -> event goes to the queue -> worker thread fans out to subscribers -> exporters batch and send.

Hard parts:
- Ordering. Events from one scope must be delivered in order. Use a per-process sequence number and FIFO publication. Re-entrancy: if a subscriber emits an event, queue it for the next dispatch round.
- Correct parenting under concurrency (Q6).
- Failure isolation. Exporter failures stay inside the exporter. Security middleware fails closed.
- Version drift in adapters. Pin tested framework ranges, run a CI matrix, and degrade to "observe only" on unknown versions.
- Shutdown. Flush with a deadline, from `atexit` and signal handlers.

Tradeoffs. A lowest-common-denominator event schema is portable but loses framework detail, so add an attributes map. Managed execution gives control but needs more integration work. Hooks are easy but read-only.

Why this is likely: this is the team's charter in the listing (reusable building blocks for agent harnesses, middleware, event emission), and NVIDIA publishes two open-source projects with this architecture.

#### Q11. Design the telemetry export path so tracing can never take down or slow down the agent.

Requirements. At-most-once delivery is acceptable for telemetry. The application keeps running while export happens. Memory is bounded, overhead is measurable, and every dropped item is counted. Assume bursts of 20,000 spans per second while a swarm of sub-agents fans out, and an OTLP collector that is sometimes slow or down.

Design, inside the process:
- `Span.End()` only enqueues. Python's BatchSpanProcessor defaults are a 2,048-span queue, 512-span batches, a 5-second schedule delay and a 30-second export timeout. At 20,000 spans per second, a 2,048 queue fills in about 100 ms of collector stall, so size the queue from burst rate times the outage you want to absorb (for example 64k spans at about 1 KB each, roughly 64 MB), or accept drops.
- Export runs on a dedicated thread (Python) or a Tokio task (Rust core), off the event loop.
- Drop policy: drop newest on overflow (the OTel default), or drop by priority so error spans and root spans survive. Expose `dropped_spans` as a metric.
- Head sampling at the root keeps whole traces. Tail sampling (keep slow or errored traces) belongs in the collector.

Outside the process: a local collector agent with `memory_limiter`, a persistent sending queue, and `block_on_overflow` set where blocking a collector is fine. A 2026 OpenTelemetry spec discussion supports making blocking an explicit opt-in for SDKs only, because `Span.End` must not block on a single-threaded loop.

Payload control:
- Prompts and completions are large and sensitive. Capture them only when the user opts in, truncate at a set size, and redact in a sanitizer stage before export.
- Put large bodies in span events or a side channel, and keep indexed attributes small.

Measuring overhead:
- Microbenchmarks per span (target low single-digit microseconds).
- Macrobenchmarks: the same agent workload with tracing off, sampled, and on, compared at p50 and p99 latency and CPU. The XPerf paper reports about 0.1 percent end-to-end overhead from its OpenTelemetry tracer, which is a fair target for a well-built exporter.
- Run the framework's own test suite with instrumentation enabled, to show behavior stays the same.

Failure handling:
- Collector down: exponential backoff with jitter, then drop.
- Shutdown: `force_flush` with a deadline.
- Fork: reinitialize processors in the child, because threads don't survive `fork`.

Why this is likely: the listing lists OpenTelemetry, exporters and observability pipelines, and the drop-versus-block question is an open OpenTelemetry spec discussion in 2026.

#### Q12. Design a middleware system for tool and LLM calls that supports guardrails, request rewriting, retries, and streaming.

Requirements. Middleware can (a) block a call, (b) rewrite the request, (c) wrap execution (retry, cache, fall back to another model), (d) enrich or redact emitted events while the application sees the same data. Works across Python, Node and Rust callbacks, async throughout. Adds little latency next to a 1-second LLM call, with a target under 1 ms for the whole chain, excluding guardrails that call models.

Split middleware into families by lifecycle meaning, because with one generic hook type every bug becomes an ordering bug. The families are:
1. Conditional guardrails run before execution and return allow or reject. A rejection is normal control flow and comes back as a return value.
2. Request intercepts transform the request.
3. Execution intercepts get `(request, next)` and may call `next` zero, one or several times: retries, fan-out, caching.
4. Stream intercepts wrap chunk delivery and finalization.
5. Event sanitizers and metadata injectors run on the publication path, after the fact, so the call proceeds without waiting on them.

NeMo Relay's middleware docs follow this split. They also specify behaviors worth copying: entries run sequentially in priority order with a name tie-break; guardrails and intercepts fail closed when their callback throws; metadata injectors fail open and only their own output is dropped; middleware callbacks must not re-enter the runtime; each concurrent `next` call gets an isolated snapshot of scope state; and every `next` must finish before the intercept returns.

Hard parts:
- Snapshot semantics. Take the middleware list as a snapshot when a call starts, so a running chain keeps the list it started with even when middleware is registered or removed mid-call.
- Streaming. A guardrail on output either buffers (safe, but adds latency to the first token) or checks incrementally and cuts the stream off on a violation. Client-visible chunks already sent can't be recalled, so document that and send a terminal error event. For tool calls inside a stream, check the assembled arguments before the tool runs.
- Retries inside execution intercepts. Retry only idempotent tools by default, cap attempts, share the request's deadline, and record each attempt as a child span.
- Cross-language cost. A Python guardrail invoked from a Rust core pays a boundary crossing (Q7). Let simple policies (allowlists, regex redaction, size limits) run natively in Rust from config.
- Testing. Run each middleware in isolation against recorded requests, plus property tests for ordering (A before B in all paths), and record guardrail decisions as events so evals can replay them.

Tradeoffs. Fail-closed improves safety and costs availability, so choose per family and make it visible in config. Several middleware kinds make the API larger, but each kind has clearer semantics than one generic hook.

Why this is likely: the listing names middleware, plugin systems, guardrails, policy engines and request/response interception. NVIDIA documents this design publicly.

#### Q13. Design a benchmarking harness that finds bottlenecks in agent workloads and catches performance regressions in CI.

Requirements. Compare frameworks, library versions, models and serving backends on the same agent tasks. Report latency (p50, p95, p99), cost (tokens), throughput, and quality. Results must be reproducible despite nondeterministic LLM outputs. Runs per commit in CI in under 15 minutes, with a nightly full run.

The main idea is to separate harness overhead from model behavior. Agent control flow depends on model output, so two live runs take different paths. XPerf solves this with trace replay: record real agent runs as dependency graphs of LLM calls and tool calls, then replay the exact input token IDs (which keeps prefix-cache behavior) and force the recorded outputs, honoring dependencies and tool durations. That gives a deterministic workload for comparing serving systems or runtime libraries.

Three modes:
1. Microbenchmarks for the library itself: event emit cost, middleware chain cost, boundary crossings, with `criterion` and `pyperf`. CI-gated with a noise threshold, for example fail on a slowdown over 5 percent with p < 0.01 across repeated runs.
2. Replay benchmarks: recorded trajectories run against a mock or real serving backend. These measure harness overhead, parallelism and cache behavior.
3. Live end-to-end evals on a fixed task set (correctness rubric, LLM judge calibrated against human labels), each task run 3 to 5 times and reported with confidence intervals. NVIDIA's NeMo Agent Toolkit `nat eval` already combines evaluators, latency stats and bottleneck analysis in one config, and AgentRace benchmarks framework efficiency the same way.

What to record per run: the full span tree (Q4) with timestamps, token counts per call, cache hit rate if the server reports it, retries, and environment (commit, model, server version, hardware).

Analysis:
- Critical path and self time per step.
- Time to first token versus decode time per call.
- Idle gaps where the harness waited on itself (sequential tool calls that could run in parallel, slow callbacks).
- Cost per successful task, so failed runs count toward the cost.

Hard parts:
- Noise. Pin CPU frequency, use dedicated runners, warm up, and prefer many repetitions to a single long run.
- Benchmark rot. Refresh traces from production regularly and keep a frozen set for long-term comparisons.
- Overfitting to the suite. Keep a hidden holdout set.

Why this is likely: the listing asks to "benchmark current agents to find bottlenecks", and lists evaluation and benchmarking systems as a plus. NVIDIA's toolkit and recent papers show the methods.

#### Q14. Design a model gateway for agent harnesses that routes requests across providers and self-hosted models, enforces rate limits, and keeps prefix caches warm.

Requirements. OpenAI-compatible and Anthropic-compatible endpoints (`/v1/chat/completions`, `/v1/responses`, `/v1/messages`), because harnesses are moving to typed content blocks for interleaved thinking and tool calls. Per-tenant token rate limits. Routing by policy (cost, latency, capability). Streaming passthrough. Added latency under 5 ms at p99. Scale: 5,000 concurrent sessions, each sending a request every few seconds with a growing context of 20k to 200k tokens.

Why caching matters. NVIDIA's Dynamo team reports that coding-agent sessions hit 85 to 97 percent prefix cache after the first call, with about 11.7 cache reads per token written, so each prefix is written once and read many times. Without cache-aware routing, turn 2 has roughly a 1/N chance of reaching the worker holding turn 1's KV cache, and every miss recomputes the whole prefix. That's seconds of prefill on long contexts, and it wastes GPU time.

Components:
1. Protocol frontend. Parses all three APIs into one internal representation of typed blocks.
2. Admission. Token-bucket limits (Q1) per tenant and per upstream, keyed on estimated input tokens, with reconcile-after-response. Returns 429 with `Retry-After` when there's no capacity. Queues stay bounded.
3. Router. Policy first (model choice, the kind of thing NVIDIA's Switchyard library does), then placement. For self-hosted pools, route to the worker with the longest matching prefix, scoring by block-hash overlap against current load. Use session affinity as a cheap fallback (hash on session id). For hosted providers, keep prompts stable so provider-side caching works: put dynamic content at the end and keep the tool order fixed.
4. Agent hints. Let harnesses pass metadata such as session id, priority, or "this sub-agent is ephemeral". Dynamo's `nvext.agent_hints` is an example, used so the cache can evict short-lived sub-agent KV first.
5. Streaming proxy. Zero-copy passthrough of server-sent events, with backpressure from the client socket to upstream.
6. Observability. One span per request with routing decision, cache hit tokens, TTFT and tokens per second.

Hard parts:
- Load versus affinity. Pure prefix affinity creates hot spots. Cap per-worker queue depth and spill over when the expected queueing cost exceeds the prefill cost.
- Failover. Fail over mid-stream only before the first token is sent. Afterwards, return an error event.
- Accounting with cached tokens, which providers price differently.
- Security. Per-tenant keys stay in the gateway, and request and response logging goes through redaction.

Tradeoffs. A gateway adds a hop and becomes a production dependency. Run it as a sidecar or library mode for single-process apps, and as a shared service for fleets.

Why this is likely: the listing says to "cut cost and reduce latency" and to work with GPU-optimized inference teams, and NVIDIA's Dynamo and Switchyard publications describe these exact mechanisms for agent harnesses.

### Behavioral

#### Q15. Tell me about a change you made to a library other people depend on that caused a regression for users. How did you find it, fix it, and keep it from happening again?

The interviewer is checking three things at once: systems debugging, ownership, and how you treat users of an open-source or shared library. Pick a story where the cause crossed a layer: a serialization change, a binding upgrade, an instrumentation hook that changed timing, an async change that broke ordering.

Structure it in this order:
1. Impact in numbers. Who noticed, how (an issue, a dashboard, a benchmark), and how bad: "p99 for users with streaming enabled went from 40 ms to 300 ms after 2.3.0."
2. Triage. How you reproduced it with the smallest case, and how you separated your code from theirs. Name the tools: `git bisect`, `py-spy` or `perf` flame graphs, tracing spans, a microbenchmark.
3. Root cause, stated precisely: "the new callback wrapper held a lock while calling user code, so a slow subscriber blocked every emit," or "we converted a dict to a Rust struct per event."
4. Fix and tradeoffs. What you shipped first (a revert or patch release) versus the real fix, and how you told users (changelog, issue reply, migration note).
5. Prevention. The benchmark gate, conformance test or compatibility matrix you added, and whether it caught anything later.

What makes a strong answer at senior level: you shipped a patch release quickly instead of waiting for a perfect fix; you credit the user who reported it; you can explain the mechanism down to threads, the event loop, or memory; and you changed the process (CI gates, deprecation policy) along with the code. If your experience is with internal code, an SDK used by several teams works, as long as you show the same care for people who had your code handed to them.

Why this is likely: NVIDIA candidates report project deep dives in every round, and the listing asks for "strong collaboration with the open-source community" plus systems-level debugging across bindings and callbacks.

## Prep checklist

- [ ] Read the NeMo Relay docs end to end: concepts (scopes, events, middleware, subscribers, codecs), the ecosystem page, and the three integration modes. Be ready to critique one design choice.
- [ ] Run the NeMo Agent Toolkit profiler and `nat eval` on a small LangChain workflow and read the nested-stack bottleneck output.
- [ ] Write Q1 to Q4 from memory in Python with no IDE, then rewrite Q2 in Rust with `tokio::sync::Semaphore`, `tokio::time::timeout` and `JoinSet`.
- [ ] Practice LeetCode 146 (LRU Cache) with a hand-written doubly linked list, 56 (Merge Intervals), 1195 (Fizz Buzz Multithreaded) and 347 (Top K Frequent Elements). All four match patterns in NVIDIA reports.
- [ ] Build a tiny PyO3 extension with maturin. Benchmark an empty call, a dict-payload call and a batched call, then try `intern!`, `cast` versus `extract`, and `Python::detach`.
- [ ] Reproduce the async generator contextvars leak from Q6 and fix it, then read google/adk-python issue 5722 and its revert history.
- [ ] Read the OpenTelemetry BatchSpanProcessor spec and issue 5178 on blocking versus dropping. Know the Python SDK defaults (2048 queue, 512 batch, 5 s delay).
- [ ] Read the OpenTelemetry GenAI semantic conventions for agent and tool spans (`gen_ai.*` attributes).
- [ ] Read the Tokio docs on bounded `mpsc`, `Semaphore`, cancel safety in `select!`, and `tracing`'s `Instrument`.
- [ ] Read the Dynamo agentic inference post (KV-aware routing, agent hints) and the XPerf paper (trace replay for agent benchmarks).
- [ ] Prepare one regression story and one cross-layer performance story, each with before and after numbers.

## Sources

- https://www.nvidia.com/en-us/about-nvidia/careers/how-we-hire/
- https://interviewing.io/nvidia-interview-questions
- https://igotanoffer.com/en/advice/nvidia-software-engineer-interview
- https://www.interviewquery.com/guides/nvidia-software-engineer
- https://leetcode.com/discuss/post/7542688
- https://www.reddit.com/r/OfferEngineering/comments/1vycmly/nvidia_senior_software_engineer_interview_process
- https://www.reddit.com/r/AskProgrammers/comments/1ve7bzw/nvidia_swe_interview_2026_a_practical_coding
- https://www.reddit.com/r/leetcode/comments/1q06zz6/2026_interview_prep
- https://www.reddit.com/r/leetcode/comments/1akv8i9/1700_questions_solved_nvidia_panel_round
- https://www.teamblind.com/post/nvidia-interview-process-t01yxggv
- https://www.teamblind.com/post/nvidia-software-engineer-interview-xrq6ekjg
- https://www.teamblind.com/post/looking-for-tips-for-interview-prep-at-nvidia-ebf382jn
- https://dataford.io/interview-guides/nvidia/agentic-ai-engineer
- https://www.interview101.com/interviews/nvidia/software-engineer
- https://www.fastprep.io/problems/nvidia-lru-key-value-cache
- https://interviewsolver.com/interview-questions/nvidia
- https://github.com/NVIDIA/NeMo-Relay
- https://docs.nvidia.com/nemo/relay
- https://docs.nvidia.com/nemo/relay/about-nemo-relay/ecosystem
- https://docs.nvidia.com/nemo/relay/about-nemo-relay/concepts/middleware
- https://github.com/NVIDIA/NeMo-Agent-Toolkit
- https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/profiler.md
- https://docs.nvidia.com/nemo/agent-toolkit/latest/run-workflows/observe/observe.html
- https://www.hoangndst.com/blog/nvidia-nemo-agent-toolkit
- https://github.com/NVIDIA-NeMo/Switchyard
- https://docs.nvidia.com/dynamo/digest/agentic-inference
- https://arxiv.org/html/2608.20370v1
- https://agent-race.github.io/home
- https://github.com/google/adk-python/issues/5722
- https://www.querystack.tech/post/instrumenting-an-async-python-service-with-opentelemetry-traces-metrics-and-context-propagation-c77f76
- https://github.com/open-telemetry/opentelemetry-specification/issues/5178
- https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md
- https://pyo3.rs/main/performance.html
- https://pyo3.rs/v0.30.1/free-threading
- https://its-saeed.github.io/learn-by-building/async/23-backpressure.html
- https://github.com/hawkw/thingbuf/blob/main/mpsc_perf_comparison.md
- https://docs.python.org/3/library/asyncio-task.html
