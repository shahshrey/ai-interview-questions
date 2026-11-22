# OpenAI: Applied AI Engineer, Sign in with ChatGPT

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

No interview report for the Sign in with ChatGPT team was found. This loop combines OpenAI's official interview guide with candidate reports from the closest customer-facing engineering tracks (Applied AI Engineer, AI Deployment Engineer, Forward Deployed Engineer). OpenAI's guide says skills-assessment "formats vary by team" (pair coding, take-homes, technical tests) and that some formats allow AI while others are built to assess you without it, so ask your recruiter for the prep document and follow it closely. The team's work is identity and permissioning for partners: Sign in with ChatGPT, connector OAuth, account linking, MCP servers and CLIs, so expect those topics in design and debugging.

1. Recruiter or intro call (30 to 60 minutes, virtual). Background, why OpenAI, why partner-facing work, hands-on LLM API experience, and sometimes your views on AI safety. AI: not relevant.
2. Technical screen (60 minutes, virtual). Most candidates get the traditional option: a practical, multi-part build-and-extend problem in CoderPad with hidden tests. Some are offered an agentic option instead: an unfamiliar Python codebase where AI tools are part of the interview and you are graded on planning, using AI well and validating its output. AI: not allowed in the traditional option, provided in the agentic option.
3. Take-home (customer-facing tracks, reported in several forms). A build on OpenAI's APIs with a recorded walkthrough, a longer app build, or a design document. Reports on the deployment track say there is no LeetCode-style round there. AI: not stated, so ask.
4. Take-home review or technical deep dive (60 minutes, virtual). Defend your design, then production questions: retries, rate limits, robustness, evals, latency and failure debugging. AI: not allowed.
5. System design (60 minutes, Excalidraw). OpenAI-flavoured systems with pushes on idempotency, retries and multi-tenancy; for this team expect sign-in, connector OAuth and MCP gateways. AI: not allowed.
6. Customer case or solution design (60 minutes). An interviewer plays a partner; scoping questions come before boxes. AI: not allowed.
7. Onsite coding (60 minutes). Another multi-part practical problem, sometimes with a part that changes or fixes working code. AI: not allowed unless the interviewer says otherwise.
8. Project deep dive (45 to 60 minutes). One project you led end to end: decisions, rejected alternatives, numbers, how you measured it. AI: not allowed.
9. Hiring manager and behavioral (45 to 60 minutes). Partner stories, saying no to an executive, ambiguity, ownership. AI: not allowed.

Two questions are classic algorithm questions because candidates still report a traditional, no-AI coding screen with algorithmic cores; they are framed in this role's domain. The rest are practical builds, reviews, traces, designs and cases.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build the one-time sign-in transaction store behind a "Continue with ChatGPT"... | Build | Technical screen (traditional) | Not allowed | Medium |
| [Q2](#q2) | Build a session manager for a partner app: generate a session with a time to... | Classic algorithm | Technical screen (traditional) | Not allowed | Medium |
| [Q3](#q3) | Build a webhook delivery scheduler that notifies partners when a user links or... | Build | Onsite coding | Not allowed | Medium |
| [Q4](#q4) | Build a consent store for connector OAuth: record grants and revocations of... | Classic algorithm | Technical screen (traditional) | Not allowed | Medium |
| [Q5](#q5) | This partner's MCP server runs anonymously today. Using the AI agent, add OAuth... | Build | Technical screen (agentic option) | Provided (AI coding agent) | Medium |
| [Q6](#q6) | The agent wrote this ID token check for a partner's "Continue with ChatGPT"... | Review | Technical screen (agentic option) | Provided (AI coding agent) | Medium |
| [Q7](#q7) | Partners report that users get "Reconnect your account" a few times a day even... | Debug | Onsite coding (refactor part) | Not allowed | Medium |
| [Q8](#q8) | A partner moved its MCP server to a new identity provider. Since then, about 40%... | Trace debug | Take-home review | Not allowed | Medium |
| [Q9](#q9) | Design the backend for a large consumer partner adding "Continue with ChatGPT" to... | Design | System design | Not allowed | Hard |
| [Q10](#q10) | A partner with 40 internal services wants ChatGPT and Codex agents to act for its... | Design | System design | Not allowed | Hard |
| [Q11](#q11) | A developer-tools partner ships a CLI that coding agents run on laptops, CI... | Design | System design | Not allowed | Medium |
| [Q12](#q12) | The interviewer plays a travel company's CTO: "We want ChatGPT agents to search... | Design | Customer case | Not allowed | Medium |
| [Q13](#q13) | An enterprise partner's security team blocks the launch: "We will not allow... | Design | Customer case | Not allowed | Hard |
| [Q14](#q14) | A partner is replacing its email sign-up form with "Continue with ChatGPT"... | Concept plus code | Project deep dive | Not allowed | Medium |
| [Q15](#q15) | Before a partner's MCP plugin launches, you must show that agents use its tools... | Concept plus code | Project deep dive | Not allowed | Hard |
| [Q16](#q16) | Tell me about a time you used an AI coding agent to deliver a partner integration... | Behavioral | Hiring manager / behavioral | Not allowed | Medium |

### Practical coding (no AI, as at OpenAI's traditional screen and onsite coding)

<a id="q1"></a>

#### Q1. (Technical screen, no AI, as at OpenAI's traditional coding screen) Build the one-time sign-in transaction store behind a "Continue with ChatGPT" button: start a transaction, build the authorization URL, and consume the callback exactly once. Then extend it so two open tabs do not break each other.

Round: Technical screen (traditional). AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: OAuth state, PKCE, nonce, expiry.

Spec of the existing code you extend (the interviewer gives this in the pad):

```text
CLIENT_ID = "oaiapp_example"
AUTHORIZE = "https://auth.openai.com/api/accounts/authorize"
TTL_SECONDS = 600                      # transactions expire after 10 minutes

class TransactionStore:
    def __init__(self, clock): ...     # clock() returns seconds, injectable for tests
    def begin(self, browser_session_id: str, redirect_uri: str) -> str:
        """Create state, PKCE verifier, S256 challenge and nonce.
        Keep them server side. Return the authorization URL."""
    def consume(self, browser_session_id: str, callback_url: str) -> dict:
        """Return {code, verifier, nonce, redirect_uri} or raise SignInError.
        Reject missing, expired, reused and mismatched transactions."""

Part 2: a user opens sign-in in two tabs. Today the second begin() overwrites
the first, so the first tab fails with state_mismatch. Fix it without letting
one browser session hold unbounded transactions.
```

<details>
<summary>Show answer</summary>

Task: a store that makes every sign-in attempt single use and short lived, plus the multi-tab extension, with tests for each rejection path.

What they grade: correct order of checks, single-use semantics, constant-time comparison, clean extension in part 2, and tests written as you go.

Approach: begin() generates three independent random values: state (CSRF protection for the callback), a PKCE verifier with its S256 challenge (so a stolen code cannot be redeemed without the verifier), and a nonce (bound into the ID token, so a replayed token fails later). Only the challenge leaves the server. The verifier stays in the store, keyed by the browser session id that lives in an HttpOnly, Secure, SameSite=Lax cookie.

consume() removes the transaction before validating anything. That makes it single use even when validation fails, so an attacker cannot retry states. Then it checks, in order: transaction exists, not expired, returned state matches, no error parameter, a code is present, and the callback path matches the stored redirect URI. Only after all of that does the caller redeem the code.

Part 2: key transactions by (browser session, state) instead of browser session alone, and cap each session at a small number of pending transactions, evicting the oldest. Alternatives: a signed, encrypted state cookie removes server storage but puts the verifier in the browser, which OpenAI's guide says to keep server side; one global map keyed only by state works but loses the binding to the browser that started the flow, which is what stops login CSRF. The keyed map with a cap of 5 is the one to build, because storage stays bounded and each tab keeps its own transaction.

Code:

```python
import base64, hashlib, secrets
from urllib.parse import urlencode, urlsplit, parse_qs

CLIENT_ID = "oaiapp_example"
AUTHORIZE = "https://auth.openai.com/api/accounts/authorize"
TTL_SECONDS = 600
MAX_PENDING = 5


class SignInError(Exception):
    pass


def b64url(raw):
    return base64.urlsafe_b64encode(raw).rstrip(b"=").decode()


def s256(verifier):
    return b64url(hashlib.sha256(verifier.encode()).digest())


class TransactionStore:
    def __init__(self, clock):
        self.clock = clock
        self.pending = {}  # browser_session_id -> {state: transaction}

    def begin(self, browser_session_id, redirect_uri):
        tx = {
            "state": b64url(secrets.token_bytes(32)),
            "nonce": b64url(secrets.token_bytes(32)),
            "verifier": b64url(secrets.token_bytes(32)),
            "redirect_uri": redirect_uri,
            "expires_at": self.clock() + TTL_SECONDS,
        }
        bucket = self.pending.setdefault(browser_session_id, {})
        now = self.clock()
        for s in [s for s, t in bucket.items() if t["expires_at"] <= now]:
            del bucket[s]
        while len(bucket) >= MAX_PENDING:
            del bucket[min(bucket, key=lambda s: bucket[s]["expires_at"])]
        bucket[tx["state"]] = tx
        query = urlencode({
            "response_type": "code", "client_id": CLIENT_ID,
            "redirect_uri": redirect_uri, "scope": "openid profile email",
            "state": tx["state"], "nonce": tx["nonce"],
            "code_challenge": s256(tx["verifier"]), "code_challenge_method": "S256",
        })
        return AUTHORIZE + "?" + query

    def consume(self, browser_session_id, callback_url):
        parts = urlsplit(callback_url)
        q = {k: v[0] for k, v in parse_qs(parts.query).items()}
        returned = q.get("state", "")
        bucket = self.pending.get(browser_session_id, {})
        tx = None
        for s in list(bucket):
            if secrets.compare_digest(s, returned):
                tx = bucket.pop(s)
        if tx is None:
            raise SignInError("missing, reused or mismatched transaction")
        if self.clock() >= tx["expires_at"]:
            raise SignInError("expired")
        if "error" in q:
            raise SignInError("authorization error: " + q["error"])
        if not q.get("code"):
            raise SignInError("no code")
        if f"{parts.scheme}://{parts.netloc}{parts.path}" != tx["redirect_uri"]:
            raise SignInError("redirect_uri mismatch")
        return {"code": q["code"], "verifier": tx["verifier"],
                "nonce": tx["nonce"], "redirect_uri": tx["redirect_uri"]}


# RFC 7636 Appendix B test vector for S256
assert s256("dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk") == "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM"

now = [1000.0]
store = TransactionStore(lambda: now[0])
CB = "https://partner.example/auth/openai/callback"
url = store.begin("sess1", CB)
params = {k: v[0] for k, v in parse_qs(urlsplit(url).query).items()}
assert "verifier" not in url and params["code_challenge_method"] == "S256"
ok = store.consume("sess1", f"{CB}?code=abc&state={params['state']}")
assert ok["code"] == "abc" and s256(ok["verifier"]) == params["code_challenge"]

for bad, why in [(f"{CB}?code=abc&state={params['state']}", "reused")]:
    try:
        store.consume("sess1", bad); raise AssertionError(why)
    except SignInError:
        pass

state2 = parse_qs(urlsplit(store.begin("sess1", CB)).query)["state"][0]
now[0] += TTL_SECONDS
try:
    store.consume("sess1", f"{CB}?code=x&state={state2}"); raise AssertionError("expired")
except SignInError as e:
    assert "expired" in str(e)

s_a = parse_qs(urlsplit(store.begin("sess2", CB)).query)["state"][0]
s_b = parse_qs(urlsplit(store.begin("sess2", CB)).query)["state"][0]
assert store.consume("sess2", f"{CB}?code=b&state={s_b}")["code"] == "b"
assert store.consume("sess2", f"{CB}?code=a&state={s_a}")["code"] == "a"
try:
    store.consume("other", f"{CB}?code=a&state={s_a}"); raise AssertionError("cross session")
except SignInError:
    pass

s_c = parse_qs(urlsplit(store.begin("sess3", CB)).query)["state"][0]
try:
    store.consume("sess3", f"{CB}?error=access_denied&state={s_c}"); raise AssertionError("error")
except SignInError as e:
    assert "access_denied" in str(e)

for _ in range(8):
    store.begin("sess4", CB)
assert len(store.pending["sess4"]) == MAX_PENDING
print("q1 ok")
```

Verify: the RFC 7636 S256 vector; a happy path; reuse of the same callback; expiry exactly at the TTL; two tabs completing in reverse order; a state from another browser session; an error=access_denied callback; the pending cap.

Pitfalls: deleting the transaction only on success, which lets an attacker retry; comparing state with == (timing leak, minor but graded); redeeming the code before checking the error parameter; putting the verifier in the URL or a readable cookie.

Follow-ups: where does this live with 20 web servers (a shared store such as Redis with the same TTL)? What changes for a confidential client (the client secret goes only in the HTTP Basic header of the token request, PKCE stays)?

Why this is likely: OpenAI's Sign in with ChatGPT website guide spells out exactly these steps and tests (fresh state, PKCE and nonce, a 10 minute transaction, rejecting missing, expired, reused and mismatched state), and candidate reports describe OpenAI's traditional screen as a multi-part build-and-extend problem with hidden tests. Medium confidence on this exact topic, high on the format.

</details>

<a id="q2"></a>

#### Q2. (Technical screen, no AI) Build a session manager for a partner app: generate a session with a time to live, renew it only if it is still live, count live sessions, and revoke every session tied to one OpenAI session reference (sid) when OpenAI sends an account security notification (an OIDC Back-Channel Logout token) for that sid.

Round: Technical screen (traditional). AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Hash map, heap with lazy deletion.
Related practice: [Design Authentication Manager](https://leetcode.com/problems/design-authentication-manager/) (Medium)

Spec:

```text
SessionManager(ttl)
  generate(session_id, sid, now)   -> session expires at now + ttl
  renew(session_id, now)           -> new expiry now + ttl, only if expiry > now
  count_live(now)                  -> sessions with expiry > now
  revoke_sid(sid)                  -> remove every session created with that sid
Rules: expiry happens before any other action at the same time.
Calls: up to 200,000, with count_live called often.
```

<details>
<summary>Show answer</summary>

Task: all four operations, fast enough for 200,000 calls, with expiry and revocation semantics exactly as specified.

What they grade: the boundary rule (expiry at time t happens first), an efficient count, a clean extension for revoke_sid, and tests on the boundaries.

Approach: keep a dict from session id to expiry. A naive count_live scans the whole dict, which costs O(n) per call and 200,000 times 200,000 in the worst case, which is far too slow. A min-heap of (expiry, session id) fixes that. count_live pops every heap entry whose expiry is at or before now and deletes the session from the dict if the dict still holds that same expiry. Renewals push a new heap entry and leave the old one behind; the dict check makes the stale entry harmless. That is lazy deletion. Each entry is pushed and popped once, so the total cost is O(n log n) and count_live is the dict size after cleanup.

revoke_sid needs a second index: sid to the set of its session ids. Revocation deletes those sessions from the dict; their heap entries become stale and are skipped later.

Alternatives: a sorted list with bisect gives O(log n) search but O(n) inserts; an ordered map (a balanced tree) gives the same bounds as the heap but Python has none built in. The heap with lazy deletion wins on simplicity at the same complexity. If renew dominated, a doubly linked list ordered by expiry works only because every new expiry is now + ttl, which is monotonic, so appends stay sorted. Mention it as an O(1) variant.

Code:

```python
import heapq


class SessionManager:
    def __init__(self, ttl):
        self.ttl = ttl
        self.expiry = {}
        self.sid_of = {}
        self.by_sid = {}
        self.heap = []

    def _expire(self, now):
        while self.heap and self.heap[0][0] <= now:
            exp, sess = heapq.heappop(self.heap)
            if self.expiry.get(sess) == exp:
                self._drop(sess)

    def _drop(self, sess):
        del self.expiry[sess]
        sid = self.sid_of.pop(sess)
        self.by_sid[sid].discard(sess)

    def generate(self, sess, sid, now):
        self._expire(now)
        self.expiry[sess] = now + self.ttl
        self.sid_of[sess] = sid
        self.by_sid.setdefault(sid, set()).add(sess)
        heapq.heappush(self.heap, (now + self.ttl, sess))

    def renew(self, sess, now):
        self._expire(now)
        if sess in self.expiry:
            self.expiry[sess] = now + self.ttl
            heapq.heappush(self.heap, (now + self.ttl, sess))

    def count_live(self, now):
        self._expire(now)
        return len(self.expiry)

    def revoke_sid(self, sid):
        for sess in list(self.by_sid.get(sid, ())):
            self._drop(sess)


m = SessionManager(5)
m.generate("a", "sid1", 1)
m.renew("a", 6)                 # expired at 6, renew ignored
assert m.count_live(6) == 0
m.generate("b", "sid1", 7)
m.generate("c", "sid2", 8)
m.renew("b", 11)                # live until 12, renewed to 16
assert m.count_live(12) == 2    # b to 16, c to 13
assert m.count_live(13) == 1
m.revoke_sid("sid1")
assert m.count_live(13) == 0
m.renew("b", 13)                # revoked sessions cannot be renewed
assert m.count_live(14) == 0

import random, time
m = SessionManager(100)
t0 = time.time()
for i in range(200_000):
    op = random.random()
    if op < 0.4:
        m.generate(f"s{i}", f"u{i % 500}", i)
    elif op < 0.7:
        m.renew(f"s{random.randrange(max(i, 1))}", i)
    elif op < 0.99:
        m.count_live(i)
    else:
        m.revoke_sid(f"u{random.randrange(500)}")
assert time.time() - t0 < 5
print("q2 ok")
```

Verify: renew exactly at expiry is ignored; two sessions with different expiries counted at each boundary; revoke then renew; a 200,000-call random run under a few seconds.

Pitfalls: treating expiry at time t as still live; forgetting that renew pushes a duplicate heap entry, which double-deletes without the dict check; letting revoke leave the sid index pointing at dead sessions.

Follow-ups: how do you run this across 20 servers (a shared store with native TTLs, and sid revocation as a fan-out message)? What happens when clocks disagree between servers?

Why this is likely: Design Authentication Manager is on the OpenAI list of a public company-wise LeetCode repository (six-month window, crowd reported, practice signal only), candidate reports say OpenAI's traditional screen uses practical multi-part problems with algorithmic cores, and the sid claim and session handling come from OpenAI's Sign in with ChatGPT guide. Medium confidence.

</details>

<a id="q3"></a>

#### Q3. (Onsite coding, no AI) Build a webhook delivery scheduler that notifies partners when a user links or unlinks their ChatGPT account: retries with capped exponential backoff and jitter, honours Retry-After, never delivers the same event twice, and dead-letters after the last attempt.

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Retries, idempotency, scheduling.

Spec:

```text
Scheduler(send, clock, rng, max_attempts=6, base=1.0, cap=300.0)
  enqueue(event_id, endpoint, payload)  # duplicates of event_id are ignored
  tick()                                # runs every attempt due at clock()
send(endpoint, payload, event_id) returns (status, retry_after_seconds or None)
Retry: 408, 429 and 5xx. Do not retry other 4xx; dead-letter them at once.
Part 2: keep events for one endpoint in order (a later event waits while an
earlier one for the same endpoint is still retrying).
```

<details>
<summary>Show answer</summary>

Task: a deterministic, testable scheduler with clear retry rules, then per-endpoint ordering.

What they grade: correct retry classification, backoff math, dedupe, the ordering extension without stalling other endpoints, and tests driven by a fake clock.

Approach: a heap of (due_time, sequence, event_id) drives attempts; sequence breaks ties so order is stable. A seen set dedupes event ids at enqueue time, and the receiver also gets the event id in a header so it can dedupe its side, because "exactly once" across a network is at-least-once delivery plus an idempotent receiver.

Backoff with full jitter: delay = rng.uniform(0, min(cap, base * 2^(attempt - 1))), where attempt counts tries so far. Full jitter spreads a burst of failures instead of retrying them in lockstep. With base 1 s, cap 300 s and 6 attempts, the worst-case schedule is 1, 2, 4, 8, 16 seconds of upper bounds, so a dead endpoint is given up on within about 31 seconds of retries; production values are larger (hours), but the shape is the same. Retry-After from a 429 or 503 overrides the computed delay when it is longer.

Part 2: keep a FIFO queue per endpoint and only schedule the head. When the head succeeds or is dead-lettered, schedule the next one. Other endpoints are unaffected. Alternatives: a global FIFO is simpler but one slow partner blocks everyone; no ordering at all is fastest but a partner can see "unlinked" before "linked". Per-endpoint queues win because each partner gets its events in order and one slow partner delays only itself.

Code:

```python
import heapq, random


class Scheduler:
    def __init__(self, send, clock, rng, max_attempts=6, base=1.0, cap=300.0):
        self.send, self.clock, self.rng = send, clock, rng
        self.max_attempts, self.base, self.cap = max_attempts, base, cap
        self.seen, self.events, self.queues = set(), {}, {}
        self.heap, self.seq = [], 0
        self.delivered, self.dead = [], []

    def _schedule(self, event_id, due):
        self.seq += 1
        heapq.heappush(self.heap, (due, self.seq, event_id))

    def enqueue(self, event_id, endpoint, payload):
        if event_id in self.seen:
            return
        self.seen.add(event_id)
        self.events[event_id] = {"endpoint": endpoint, "payload": payload, "attempt": 0}
        q = self.queues.setdefault(endpoint, [])
        q.append(event_id)
        if len(q) == 1:
            self._schedule(event_id, self.clock())

    def _finish(self, event_id, ok):
        ev = self.events.pop(event_id)
        (self.delivered if ok else self.dead).append(event_id)
        q = self.queues[ev["endpoint"]]
        q.pop(0)
        if q:
            self._schedule(q[0], self.clock())

    def tick(self):
        now = self.clock()
        while self.heap and self.heap[0][0] <= now:
            _, _, eid = heapq.heappop(self.heap)
            ev = self.events[eid]
            ev["attempt"] += 1
            status, retry_after = self.send(ev["endpoint"], ev["payload"], eid)
            if 200 <= status < 300:
                self._finish(eid, True)
            elif (status in (408, 429) or status >= 500) and ev["attempt"] < self.max_attempts:
                delay = self.rng.uniform(0, min(self.cap, self.base * 2 ** (ev["attempt"] - 1)))
                if retry_after is not None:
                    delay = max(delay, retry_after)
                self._schedule(eid, now + delay)
            else:
                self._finish(eid, False)


now = [0.0]
calls = []
script = {"e1": [503, 429, 200], "e2": [200], "e3": [400], "e4": [500] * 10, "e5": [200]}

def send(endpoint, payload, eid):
    calls.append((round(now[0], 2), eid))
    status = script[eid].pop(0)
    return status, (30 if status == 429 else None)

s = Scheduler(send, lambda: now[0], random.Random(7))
s.enqueue("e1", "partnerA", {"type": "linked"})
s.enqueue("e2", "partnerA", {"type": "unlinked"})
s.enqueue("e1", "partnerA", {"type": "linked"})       # duplicate ignored
s.enqueue("e3", "partnerB", {})
s.enqueue("e4", "partnerC", {})
s.enqueue("e5", "partnerC", {})
while s.heap and now[0] < 10_000:
    now[0] = s.heap[0][0]
    s.tick()
assert s.delivered.index("e1") < s.delivered.index("e2")
assert "e3" in s.dead and [c for c in calls if c[1] == "e3"] == [(0.0, "e3")]
assert "e4" in s.dead and sum(1 for c in calls if c[1] == "e4") == 6
assert "e5" in s.delivered
e1_times = [t for t, e in calls if e == "e1"]
assert e1_times[2] - e1_times[1] >= 30                 # Retry-After honoured
assert sum(1 for c in calls if c[1] == "e1") == 3
print("q3 ok", s.delivered, s.dead)
```

Verify: a 503 then 429 then success (Retry-After honoured); a 400 tried once and dead-lettered; a permanently failing endpoint tried exactly 6 times; a duplicate enqueue; ordering on one endpoint while another endpoint keeps moving.

Pitfalls: retrying 400 or 401 forever; a jitter-free backoff that synchronises retries after an outage; dedupe only on the sender side; blocking all partners behind one failing endpoint.

Follow-ups: how do you sign payloads so partners can verify them (an HMAC over timestamp and body, with a tolerance window to stop replays)? How does a partner replay missed events after a long outage?

Why this is likely: candidate reports and prep vendors list webhook delivery with retries and idempotency as an OpenAI system design and coding theme, 1point3acres posts describe payment idempotency questions, and partner notifications fit this role's integration work. Medium confidence.

</details>

<a id="q4"></a>

#### Q4. (Technical screen, no AI) Build a consent store for connector OAuth: record grants and revocations of scopes per user and client as they happen, and answer "which scopes did this user grant this client at time t" for audits, including events that arrive late.

Round: Technical screen (traditional). AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Binary search, versioned key-value store.
Related practice: [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) (Medium)

Spec:

```text
ConsentStore()
  grant(user, client, scopes: set, ts)    # adds scopes at ts
  revoke(user, client, scopes: set, ts)   # removes scopes at ts; empty set = all
  scopes_at(user, client, ts) -> set      # state after every event with time <= ts
Part 1: events arrive in timestamp order.
Part 2: events can arrive out of order (queue replays), and two events can share
a timestamp; apply same-timestamp events in arrival order.
Load: millions of events, audits far more frequent than writes.
```

<details>
<summary>Show answer</summary>

Task: a versioned store with fast point-in-time reads, then out-of-order writes without breaking reads.

What they grade: picking snapshots versus deltas deliberately, binary search on the right boundary, and handling late events correctly.

Approach: for each (user, client) key keep a list of versions sorted by (ts, arrival sequence). Each version stores the full scope set after that event, so a read is one bisect_right on the timestamp and a lookup: O(log n). Storing deltas instead makes writes cheap but each read replays history, O(n). Audits dominate here, so snapshots win; scope sets are small (a handful of strings), so the memory cost is modest.

Part 1 is the classic time-based key-value store: append and bisect. Part 2: a late event lands in the middle, so every later snapshot is now wrong. Insert it at its sorted position and recompute snapshots from the event before it to the end. That is O(k) for k later versions per late event, which is fine when late events are rare and recent. If late events were common, store deltas and add periodic checkpoints every 64 versions: reads replay at most 64 deltas, and a late write rebuilds only the checkpoints after it. Alternatives ranked: snapshot list (fast reads, simple), delta log plus checkpoints (balanced), full event sourcing per read (simplest writes, too slow for audits).

The boundary matters: scopes_at(ts) includes events at exactly ts, so use bisect_right on the key (ts, infinity).

Code:

```python
import bisect, itertools

INF = float("inf")


class ConsentStore:
    def __init__(self):
        self.events = {}      # key -> sorted list of (ts, seq, op, scopes)
        self.snaps = {}       # key -> list of frozenset, parallel to events
        self.seq = itertools.count()

    def _add(self, key, ts, op, scopes):
        ev = self.events.setdefault(key, [])
        sn = self.snaps.setdefault(key, [])
        item = (ts, next(self.seq), op, frozenset(scopes))
        i = bisect.bisect_right(ev, (item[0], item[1]))
        ev.insert(i, item)
        sn.insert(i, frozenset())
        state = sn[i - 1] if i else frozenset()
        for j in range(i, len(ev)):
            _, _, o, s = ev[j]
            if o == "grant":
                state = state | s
            else:
                state = frozenset() if not s else state - s
            sn[j] = state

    def grant(self, user, client, scopes, ts):
        self._add((user, client), ts, "grant", scopes)

    def revoke(self, user, client, scopes, ts):
        self._add((user, client), ts, "revoke", scopes)

    def scopes_at(self, user, client, ts):
        ev = self.events.get((user, client), [])
        i = bisect.bisect_right(ev, (ts, INF))
        return set(self.snaps[(user, client)][i - 1]) if i else set()


c = ConsentStore()
c.grant("u1", "acme", {"files:read"}, 10)
c.grant("u1", "acme", {"files:write"}, 20)
c.revoke("u1", "acme", {"files:write"}, 30)
assert c.scopes_at("u1", "acme", 9) == set()
assert c.scopes_at("u1", "acme", 10) == {"files:read"}
assert c.scopes_at("u1", "acme", 25) == {"files:read", "files:write"}
assert c.scopes_at("u1", "acme", 30) == {"files:read"}
c.revoke("u1", "acme", set(), 15)                 # late event: revoke all at 15
assert c.scopes_at("u1", "acme", 15) == set()
assert c.scopes_at("u1", "acme", 25) == {"files:write"}
c.grant("u1", "acme", {"calendar:read"}, 30)      # same ts, arrives after the revoke
assert c.scopes_at("u1", "acme", 30) == {"calendar:read"}
assert c.scopes_at("u2", "acme", 30) == set()
print("q4 ok")
```

Verify: reads before the first event, exactly at an event, and between events; a late revoke-all that changes later answers; two events with the same timestamp in arrival order; an unknown user.

Pitfalls: bisect_left, which misses events at exactly ts; recomputing only the inserted snapshot and leaving later ones stale; mutable sets shared between snapshots, so one write silently changes history.

Follow-ups: how would you store this in a database (an append-only events table with an index on key and time, plus materialized snapshots)? What retention does an audit trail need, and how do you delete a user's data on request?

Why this is likely: a recent 1point3acres roundup of OpenAI reports lists a KV store build and a credits ledger with expiry and out-of-order operations among common practical coding questions, and connector OAuth consent with scopes is this team's domain. The LeetCode link is a topic pick for practice, not on OpenAI's crowd list. Medium confidence.

</details>

### Agentic coding (AI agent provided, as in OpenAI's agentic option)

<a id="q5"></a>

#### Q5. (Agentic coding round, AI agent provided) This partner's MCP server runs anonymously today. Using the AI agent, add OAuth so ChatGPT can link user accounts: publish protected resource metadata, challenge unauthenticated calls correctly, verify access tokens, and require a scope per tool. Narrate your prompts and what you check in each diff.

Round: Technical screen (agentic option). AI policy: Provided (AI coding agent). Type: Build. Difficulty: Medium. Topic: MCP authorization, OAuth 2.1 resource server.

The repo you are given:

```text
acme-mcp/
  pyproject.toml
  acme_mcp/
    __init__.py
    server.py          # routes: POST /mcp ; tools: list_files, read_file, upload_file
    tools.py           # tool implementations, currently use a shared demo account
    config.py          # RESOURCE = "https://mcp.acme.example"
                       # ISSUER   = "https://auth.acme.example"
    jwks.py            # fetch_jwks(issuer) -> keys (exists, unused)
  tests/
    test_tools.py      # 9 passing tests, no auth
    conftest.py
  README.md            # "auth: TODO"
```

<details>
<summary>Show answer</summary>

Task: a working resource server that ChatGPT can discover and link to, with tests, built by delegating small steps to the agent and checking each one.

What they grade: reading the code before prompting, small delegated steps, checking each diff against the spec, tests that fail before the fix, and catching at least one wrong agent change.

Approach: I read server.py and config.py myself first, then ask the agent for a plan only ("list the files you would change for MCP authorization; no code yet"). Then four steps, one diff each:

1. Serve /.well-known/oauth-protected-resource from config. Check: resource is the exact URL ChatGPT sends as the resource parameter, and authorization_servers holds the exact issuer string.
2. Return 401 with WWW-Authenticate: Bearer resource_metadata=<url> on every auth failure. Check: the header is always present; it is how ChatGPT finds the metadata and re-runs OAuth.
3. Verify signature through jwks.py, then iss, aud equal to RESOURCE, exp and nbf with small leeway. Check: aud may be a string or a list.
4. Map tools to scopes: files:read for list_files and read_file, files:write for upload_file.

Wrong agent output I caught: in step 4 the agent wrote `if required in claims["scope"]`, a substring test, so a files:readonly token passed the files:read check. I asked for split-and-compare and added a test for that token. It also put a trailing slash on the authorization_servers entry; OpenAI's docs say the issuer is compared as an exact string, so linking would fail. I fixed it to match ISSUER.

Alternatives: introspection on every call gives instant revocation but adds a network round trip (tens of milliseconds) against well under a millisecond for local JWT checks; an auth gateway in front keeps tools clean but splits the challenge from the metadata. Local checks with short-lived tokens win.

Code (the verification core the agent's diff must match; the signature check is done by the existing JWKS library code and is out of scope here):

```python
import time

RESOURCE = "https://mcp.acme.example"
ISSUER = "https://auth.acme.example"
TOOL_SCOPES = {"list_files": "files:read", "read_file": "files:read", "upload_file": "files:write"}
LEEWAY = 5


def protected_resource_metadata():
    return {"resource": RESOURCE, "authorization_servers": [ISSUER],
            "scopes_supported": sorted(set(TOOL_SCOPES.values()))}


def challenge(error=None):
    value = f'Bearer resource_metadata="{RESOURCE}/.well-known/oauth-protected-resource"'
    if error:
        value += f', error="{error}"'
    return 401, {"WWW-Authenticate": value}


def authorize(tool, claims, now=None):
    """claims: payload of a token whose signature already verified, or None."""
    now = time.time() if now is None else now
    if claims is None:
        return challenge()
    aud = claims.get("aud")
    auds = aud if isinstance(aud, list) else [aud]
    if claims.get("iss") != ISSUER or RESOURCE not in auds:
        return challenge("invalid_token")
    if claims.get("exp", 0) + LEEWAY <= now or claims.get("nbf", 0) - LEEWAY > now:
        return challenge("invalid_token")
    granted = set(str(claims.get("scope", "")).split())
    if TOOL_SCOPES[tool] not in granted:
        return 403, {"WWW-Authenticate": f'Bearer error="insufficient_scope", scope="{TOOL_SCOPES[tool]}"'}
    return 200, {"sub": claims["sub"]}


NOW = 1_800_000_000
good = {"iss": ISSUER, "aud": RESOURCE, "sub": "user-42", "exp": NOW + 900, "scope": "files:read"}
assert protected_resource_metadata()["authorization_servers"] == ["https://auth.acme.example"]
assert authorize("read_file", good, NOW) == (200, {"sub": "user-42"})
status, headers = authorize("read_file", None, NOW)
assert status == 401 and "resource_metadata=" in headers["WWW-Authenticate"]
assert authorize("read_file", dict(good, aud="https://other.example"), NOW)[0] == 401
assert authorize("read_file", dict(good, aud=["x", RESOURCE]), NOW)[0] == 200
assert authorize("read_file", dict(good, iss=ISSUER + "/"), NOW)[0] == 401
assert authorize("read_file", dict(good, exp=NOW - 61), NOW)[0] == 401
assert authorize("read_file", dict(good, scope="files:readonly"), NOW)[0] == 403
assert authorize("upload_file", good, NOW)[0] == 403
print("q5 ok")
```

Verify: the 9 existing tests pass with a token fixture; new tests for no token, wrong audience, list audience, trailing-slash issuer, expiry, files:readonly, and a write tool with a read scope; then one manual link from ChatGPT.

Pitfalls: returning 401 without the WWW-Authenticate header; accepting any token your IdP signed regardless of audience, which lets a token for another API call your tools; one-shot prompting the whole feature and reviewing a 400-line diff.

Follow-ups: how do tools act as the user instead of the shared demo account? How would you support a user linking both a personal and a work account?

Why this is likely: a first-hand Blind report describes OpenAI's agentic coding option as an unfamiliar Python codebase graded on planning, using AI well and validating its output, and OpenAI's plugin authentication docs define exactly this MCP authorization contract, which the job description names as core work. Medium confidence.

</details>

<a id="q6"></a>

#### Q6. (Agentic coding round, AI agent provided) The agent wrote this ID token check for a partner's "Continue with ChatGPT" callback. Review it like a pull request: find the defects, decide whether to merge, and write the tests that would have caught them.

Round: Technical screen (agentic option). AI policy: Provided (AI coding agent). Type: Review. Difficulty: Medium. Topic: OIDC ID token verification, account linking, security.

```text
import jwt, requests, logging

ISSUER = "https://auth.openai.com"
CLIENT_ID = "oaiapp_example"
JWKS = requests.get(ISSUER + "/.well-known/jwks.json").json()      # loaded once at import

def verify_and_login(id_token, expected_nonce, db):
    header = jwt.get_unverified_header(id_token)
    key = next(k for k in JWKS["keys"] if k["kid"] == header["kid"])
    try:
        claims = jwt.decode(id_token, jwt.PyJWK(key).key,
                            algorithms=[header["alg"]],
                            options={"verify_aud": False},
                            leeway=3600,
                            verify_nonce=expected_nonce)
    except Exception:
        logging.error("bad token %s", id_token)
        return None
    if not claims["iss"].startswith(ISSUER) or CLIENT_ID not in claims["aud"]:
        return None
    user = db.find_by_email(claims.get("email"))
    if user:
        return user
    return db.create(sub=claims["sub"], email=claims.get("email"))
```

<details>
<summary>Show answer</summary>

Task: a block or approve decision, each defect with its impact, the fixed logic, and one regression test per defect.

What they grade: spotting the hallucinated parameter and the security holes; knowing OpenAI's documented checks (signature against JWKS, issuer, audience, expiry, original nonce); a clear merge decision.

Approach: block the merge. Defects, most severe first:

1. Hallucinated API: PyJWT's decode has no verify_nonce option, so the nonce is never checked (depending on the version the argument is ignored with a warning or rejected). Replayed ID tokens pass. Compare claims["nonce"] to the transaction's nonce yourself.
2. Account takeover: it links by email without checking email_verified and before looking up the OpenAI subject. OpenAI's guide says look up the stored (issuer, client, sub) mapping first and link by email only when email_verified is true.
3. Issuer check uses startswith, so https://auth.openai.com.evil.example passes. Use exact equality with the discovery issuer.
4. Audience check is turned off and replaced with a substring test on a string aud. Let the library verify audience against the client id.
5. Algorithm comes from the attacker-controlled header. Pin the algorithm to the key's declared algorithm (RS256 here).
6. leeway=3600 accepts tokens an hour past expiry. OpenAI says allow only a small clock skew, and its example uses 5 seconds; use a few seconds.
7. JWKS is fetched once at import and never refreshed, so the next key rotation causes a total sign-in outage; next() also raises StopIteration on an unknown kid. Refresh on an unknown kid, rate limited.
8. Logging the raw ID token leaks personal data and a usable credential into logs. Log the kid, error class and request id.

Alternatives: a maintained JWT library with explicit options (chosen) versus a full OIDC client library that also handles discovery and JWKS caching, better for new code but a larger change to review. Both keep JWT parsing inside a maintained library.

Code (claim checks and JWKS caching after the fix; the signature check itself stays in the maintained library and is passed in, so this block runs with the standard library only):

```python
import time

ISSUER = "https://auth.openai.com"
CLIENT_ID = "oaiapp_example"
LEEWAY = 5


class JwksCache:
    def __init__(self, fetch, min_refresh_s=60, clock=time.time):
        self.fetch, self.min_refresh_s, self.clock = fetch, min_refresh_s, clock
        self.keys, self.fetched_at = {}, -1e18

    def get(self, kid):
        if kid not in self.keys and self.clock() - self.fetched_at >= self.min_refresh_s:
            self.keys = {k["kid"]: k for k in self.fetch()["keys"]}
            self.fetched_at = self.clock()
        return self.keys.get(kid)


def check_claims(claims, expected_nonce, now):
    aud = claims.get("aud")
    auds = aud if isinstance(aud, list) else [aud]
    problems = []
    if claims.get("iss") != ISSUER:
        problems.append("iss")
    if CLIENT_ID not in auds:
        problems.append("aud")
    if claims.get("exp", 0) + LEEWAY <= now:
        problems.append("exp")
    if claims.get("nonce") != expected_nonce:
        problems.append("nonce")
    if not claims.get("sub"):
        problems.append("sub")
    return problems


def resolve_account(claims, db):
    key = (claims["iss"], CLIENT_ID, claims["sub"])
    if key in db["links"]:
        return db["links"][key], "linked"
    email = claims.get("email")
    if email and claims.get("email_verified") is True and email in db["by_email"]:
        db["links"][key] = db["by_email"][email]
        return db["by_email"][email], "linked_by_verified_email"
    if email and email in db["by_email"]:
        return None, "needs_ownership_check"
    user = f"user{len(db['links']) + 1}"
    db["links"][key] = user
    return user, "created"


NOW = 1_800_000_000
base = {"iss": ISSUER, "aud": [CLIENT_ID], "sub": "abc", "exp": NOW + 300, "nonce": "n1"}
assert check_claims(base, "n1", NOW) == []
assert check_claims(dict(base, iss=ISSUER + ".evil.example"), "n1", NOW) == ["iss"]
assert check_claims(dict(base, aud="oaiapp_example_2"), "n1", NOW) == ["aud"]
assert check_claims(dict(base, exp=NOW - 120), "n1", NOW) == ["exp"]
assert check_claims(base, "other-nonce", NOW) == ["nonce"]

fetches = []
t = [0.0]
def fetch():
    fetches.append(t[0])
    return {"keys": [{"kid": "k2" if t[0] > 100 else "k1", "alg": "RS256"}]}
cache = JwksCache(fetch, clock=lambda: t[0])
assert cache.get("k1")["alg"] == "RS256"
t[0] = 200
assert cache.get("k2") is not None and len(fetches) == 2      # rotation picked up
assert cache.get("bogus") is None and len(fetches) == 2       # rate limited

db = {"links": {}, "by_email": {"alex@example.com": "user-alex"}}
victim = dict(base, sub="attacker", email="alex@example.com", email_verified=False)
assert resolve_account(victim, db) == (None, "needs_ownership_check")
owner = dict(base, sub="alex-sub", email="alex@example.com", email_verified=True)
assert resolve_account(owner, db) == ("user-alex", "linked_by_verified_email")
moved = dict(owner, email="alex@new.example")
assert resolve_account(moved, db) == ("user-alex", "linked")
print("q6 ok")
```

Verify: lookalike issuer, foreign audience, expired token, wrong nonce, key rotation, unknown-kid flood, unverified email matching an account, and a returning user with a new email.

Pitfalls: approving because the happy-path test passes; fixing only the crypto and missing the account-linking hole; caching JWKS forever; logging tokens.

Follow-ups: how do you store the session afterwards (a first-party HttpOnly cookie holding a session id; discard the raw ID token after verification)? What breaks when OpenAI rotates keys during a traffic peak?

Why this is likely: OpenAI's agentic round is reported to grade validating AI output, other companies' AI-assisted rounds (for example Canva's) explicitly grade critical review of AI-written code, and every check here is spelled out in OpenAI's Sign in with ChatGPT guide. Medium confidence.

</details>

### Debug and trace in the identity stack (no AI)

<a id="q7"></a>

#### Q7. (Onsite coding, refactor part, no AI) Partners report that users get "Reconnect your account" a few times a day even though refresh tokens last 30 days. Here is the token manager and the production log. Find the root cause, fix it, and prove the fix with a test.

Round: Onsite coding (refactor part). AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Refresh token rotation, concurrency.

```text
class TokenManager:
    def __init__(self, store, auth_server):
        self.store, self.auth = store, auth_server

    def access_token(self, link_id):
        cred = self.store.get(link_id)
        if cred["expires_at"] <= time.time():
            try:
                new = self.auth.refresh(cred["refresh_token"])
            except InvalidGrant:
                self.store.delete(link_id)        # forces "Reconnect your account"
                raise
            self.store.put(link_id, new)
            cred = new
        return cred["access_token"]

log (one link, times in seconds):
  12.001 tool=list_files  link=L9 access expired -> refresh rt_41
  12.003 tool=read_file   link=L9 access expired -> refresh rt_41
  12.140 auth: rt_41 rotated -> rt_42 issued
  12.151 auth: rt_41 reuse detected -> invalid_grant, family revoked
  12.152 link=L9 deleted (InvalidGrant)
  12.160 tool=list_files ok
  12.161 tool=read_file  error: reconnect required
```

<details>
<summary>Show answer</summary>

Task: explain the failure from the log, fix it in the token manager, and add a test that fails before and passes after.

What they grade: reading the log as evidence, naming the race precisely, a fix that is correct under concurrency, and real revocations that still delete the link.

Approach: the agent issues two tool calls in parallel for the same link. Both see an expired access token and both refresh with rt_41. The authorization server rotates refresh tokens: the first call gets rt_42, the second reuses rt_41, which looks like token theft, so the server returns invalid_grant (and many servers revoke the whole token family). The manager then deletes the link, so the user must reconnect. The bug is the unsynchronised check-then-refresh plus treating any invalid_grant as "user revoked".

Fix, three parts: (1) single-flight refresh per link: take a per-link lock, re-read the credential inside the lock, and refresh only if it is still expired, so the second caller uses the first caller's new token. (2) Refresh early, 60 seconds before expiry, so calls near the boundary do not all refresh together. (3) On invalid_grant, delete the link only if the refresh token you used is still the stored one; if it changed, another refresh won and you just retry with the new token.

Alternatives: a per-link lock in one process is enough for one server; with many servers, use a short lease in the shared store (for example a set-if-absent key with a 10 second expiry) or route each link to one worker. Disabling rotation would hide the bug and weaken security. Asking the authorization server for a reuse grace period helps but you do not control every partner's server. The lock plus re-check needs no change on the authorization server, so the partner can ship it alone.

Code:

```python
import threading, time


class InvalidGrant(Exception):
    pass


class AuthServer:
    """Rotating refresh tokens with reuse detection, like many real servers."""
    def __init__(self):
        self.valid, self.n, self.lock = {"rt_41"}, 41, threading.Lock()

    def refresh(self, rt):
        time.sleep(0.05)
        with self.lock:
            if rt not in self.valid:
                self.valid.clear()
                raise InvalidGrant(rt)
            self.valid.discard(rt)
            self.n += 1
            new = f"rt_{self.n}"
            self.valid.add(new)
            return {"access_token": f"at_{self.n}", "refresh_token": new, "expires_at": time.time() + 3600}


class TokenManager:
    EARLY = 60

    def __init__(self, store, auth):
        self.store, self.auth = store, auth
        self.locks, self.locks_guard = {}, threading.Lock()

    def _lock(self, link_id):
        with self.locks_guard:
            return self.locks.setdefault(link_id, threading.Lock())

    def access_token(self, link_id):
        cred = self.store[link_id]
        if cred["expires_at"] - self.EARLY > time.time():
            return cred["access_token"]
        with self._lock(link_id):
            cred = self.store[link_id]
            if cred["expires_at"] - self.EARLY > time.time():
                return cred["access_token"]
            used = cred["refresh_token"]
            try:
                new = self.auth.refresh(used)
            except InvalidGrant:
                if self.store.get(link_id, {}).get("refresh_token") == used:
                    del self.store[link_id]
                raise
            self.store[link_id] = new
            return new["access_token"]


store = {"L9": {"access_token": "at_40", "refresh_token": "rt_41", "expires_at": time.time() - 1}}
auth = AuthServer()
tm = TokenManager(store, auth)
results, errors = [], []

def call():
    try:
        results.append(tm.access_token("L9"))
    except InvalidGrant as e:
        errors.append(e)

threads = [threading.Thread(target=call) for _ in range(8)]
for th in threads:
    th.start()
for th in threads:
    th.join()
assert not errors and set(results) == {"at_42"} and "L9" in store
store["L9"]["expires_at"] = time.time() + 30          # inside the early window
assert tm.access_token("L9") == "at_43"
auth.valid.clear()                                     # user really revoked access
store["L9"]["expires_at"] = 0
try:
    tm.access_token("L9"); raise AssertionError("should fail")
except InvalidGrant:
    assert "L9" not in store
print("q7 ok")
```

Verify: 8 concurrent calls on an expired token produce one refresh and no deletion; a token inside the early window refreshes; a real revocation still deletes the link. Before the fix, the same 8-thread test produces invalid_grant and a deleted link.

Pitfalls: a global lock that serialises all users; checking expiry only outside the lock; retrying invalid_grant blindly, which can trip reuse detection again; logging refresh tokens while debugging.

Follow-ups: how do you make this work across 30 servers? How do you tell a partner's server bug apart from a real user revocation in metrics?

Why this is likely: candidate reports describe OpenAI onsite coding with a part that changes or fixes working code, OpenAI's plugin docs note that access and refresh tokens can expire or rotate while a connection stays in use, and the job asks for tracing failures end to end in partner integrations. Medium confidence on topic.

</details>

<a id="q8"></a>

#### Q8. (Take-home review or technical deep dive, no AI) A partner moved its MCP server to a new identity provider. Since then, about 40% of new account links fail and some existing users hit errors after a month. Here are the traces from both sides. Find every root cause and the fix for each.

Round: Take-home review. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: OAuth discovery, issuer identification, client registration.

```text
# partner auth server metadata (new IdP)
GET https://auth.partner.example/.well-known/oauth-authorization-server
{ "issuer": "https://auth.partner.example/",
  "authorization_response_iss_parameter_supported": true,
  "registration_endpoint": "https://auth.partner.example/register",
  "code_challenge_methods_supported": ["S256"], ... }

# partner MCP server protected resource metadata
{ "resource": "https://mcp.partner.example",
  "authorization_servers": ["https://auth.partner.example"] }

# link attempt A (fails)
callback ?code=...&state=...&iss=https%3A%2F%2Fauth.partner.example%2F
host: iss "https://auth.partner.example/" != recorded issuer "https://auth.partner.example" -> rejected

# link attempt B (succeeds, older IdP pod not yet drained)
callback ?code=...&state=...&iss=https%3A%2F%2Fauth.partner.example

# existing link, day 31
POST /token grant_type=refresh_token client_id=dcr_7f2... -> 401 {"error":"invalid_client"}
IdP config: dynamic clients expire after 30 days of inactivity OR 30 days after creation
```

<details>
<summary>Show answer</summary>

Task: separate the independent failures, tie each to evidence in the trace, and give the partner a fix and a check for each.

What they grade: using the traces as evidence, knowing the documented contract, separating two bugs that look like one, and giving the partner something they can verify.

Approach: there are two root causes.

1. Issuer mismatch (the 40% of new links). The new IdP publishes issuer with a trailing slash, while the protected resource metadata lists it without one. OpenAI's plugin docs say the metadata issuer and the authorization_servers entry must be the same exact string, and that the returned iss is compared by exact string, with no normalising of trailing slashes. Attempts that hit new IdP pods return iss with a slash and are rejected; attempts on old pods still pass, which is why only some attempts fail today; the share will climb toward 100% as the old pods drain. Fix: pick one canonical issuer string and use it in the IdP config, the authorization server metadata, the protected resource metadata, and the iss response parameter.
2. Expiring dynamic clients (failures after a month). ChatGPT registers a DCR client once per MCP server connection and reuses it. The new IdP deletes dynamic clients 30 days after creation, so every refresh then gets invalid_client. OpenAI's docs say to keep the registered client valid while the connection is in use. Fix: disable creation-based expiry for these clients, or move to CIMD, where ChatGPT presents a stable client id document and there is no registration to expire.

Alternatives for the second fix: a 365-day expiry only delays the outage; inactivity cleanup is safe only if it counts token activity; CIMD removes the problem and gives one stable client identity, so I recommend it.

Code (a small classifier I would run over the partner's exported logs to size each cause; the sample below is illustrative):

```python
import re
from collections import Counter

LOG = """\
link L1 iss=https://auth.partner.example/ recorded=https://auth.partner.example rejected
link L2 iss=https://auth.partner.example recorded=https://auth.partner.example ok
link L3 iss=https://auth.partner.example/ recorded=https://auth.partner.example rejected
refresh R1 client=dcr_7f2 status=401 error=invalid_client age_days=31
refresh R2 client=dcr_9a1 status=200 age_days=12
refresh R3 client=dcr_1c4 status=400 error=invalid_grant age_days=3
link L4 iss=https://auth.partner.example/ recorded=https://auth.partner.example rejected
link L5 iss=https://auth.partner.example recorded=https://auth.partner.example ok
"""

def classify(line):
    m = re.search(r"iss=(\S+) recorded=(\S+) (\w+)", line)
    if m:
        iss, rec, outcome = m.groups()
        if outcome == "ok":
            return "ok"
        return "issuer_trailing_slash" if iss.rstrip("/") == rec.rstrip("/") else "issuer_other"
    m = re.search(r"status=(\d+)(?: error=(\w+))? age_days=(\d+)", line)
    if m:
        status, err, age = m.groups()
        if status == "200":
            return "ok"
        if err == "invalid_client" and int(age) >= 30:
            return "dcr_client_expired"
        return err or "other"
    return "unparsed"

counts = Counter(classify(l) for l in LOG.splitlines())
print(counts)
links = [l for l in LOG.splitlines() if l.startswith("link")]
assert counts["issuer_trailing_slash"] == 3 and counts["dcr_client_expired"] == 1
assert counts["invalid_grant"] == 1
assert round(counts["issuer_trailing_slash"] / len(links), 2) == 0.6
print("q8 ok")
```

Verify: compare both metadata issuer strings byte for byte; link fresh accounts across every IdP pod; force a refresh on a 31-day-old link in staging; watch link success and invalid_client counts daily.

Pitfalls: "fixing" it by asking OpenAI to normalise slashes; blaming the 40% on flakiness because some attempts pass; treating invalid_grant (a revoked or reused refresh token, a separate issue) as the same bug as invalid_client.

Follow-ups: what alert would have caught this before partners did? What would you add to the partner launch checklist?

Why this is likely: a first-person OpenAI FDE report describes a take-home review followed by production questions on debugging and robustness, the job asks for tracing failures end to end, and both failure modes come straight from OpenAI's plugin authentication docs (exact issuer matching, keeping DCR clients valid). Medium confidence.

</details>

### System design (no AI)

<a id="q9"></a>

#### Q9. (System design, no AI) Design the backend for a large consumer partner adding "Continue with ChatGPT" to its website and app: sign-in transactions, token exchange, account resolution, sessions, sign-out, and what happens when the OpenAI auth service is slow or down.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: OIDC sign-in at scale.

<details>
<summary>Show answer</summary>

Task: an architecture, the request path, the data model, capacity numbers, and the failure plan, presented as you would to a partner's engineering lead.

Approach: four parts. (1) A sign-in service that owns /auth/openai (create transaction, redirect) and the callback (consume transaction, exchange code, verify ID token). (2) A transaction store with TTLs. (3) An identity table mapping (issuer, client id, OpenAI sub) to the local user, with a unique index on that triple. (4) The partner's existing first-party session service. OpenAI's discovery document is fetched at startup and cached; JWKS is cached and refreshed on an unknown kid. The ID token is verified and then discarded; the partner keeps only the mapping, the profile fields it needs, and the sid with each session so it can find sessions tied to one OpenAI session.

Numbers: assume 30 million monthly users and 3 million sign-ins a day. That is about 35 per second on average; plan for a 10x peak of 350 per second. Transactions live 10 minutes, so at peak at most 350 x 600 = 210,000 live transactions at about 300 bytes each, roughly 63 MB, which fits one small Redis primary with a replica. The callback does one token request to OpenAI (budget 300 ms), one local signature check (under 1 ms with cached keys), and two database reads. Target p95 under 800 ms for the whole callback.

Alternatives: transaction state in a signed, encrypted cookie removes Redis but moves the PKCE verifier to the browser, against OpenAI's guidance; a relational table works but needs a TTL sweeper and adds write load. Redis with TTL wins. For account resolution, keying only by email is simpler but breaks when users change email and invites takeover; the (issuer, client, sub) key with a verified-email link step is what OpenAI documents.

Risks: OpenAI auth slow or down: keep other sign-in methods visible, time out the token call at 5 seconds with one retry; existing first-party sessions are unaffected. JWKS refresh storms: rate limit refreshes to one a minute per instance. Duplicate accounts from double submits: the unique index makes the second insert fail and fall back to lookup. Monitoring: callback outcome by error class (state, nonce, exchange, verification), with request ids from OpenAI's response headers for support tickets.

Follow-ups: how does sign-out work (clear only the partner's session)? How would you support the same button inside a ChatGPT plugin?

Why this is likely: candidate reports say OpenAI design rounds probe idempotency, retries and multi-tenancy on OpenAI-flavoured products, and the job's core product is this exact sign-in flow, documented step by step in OpenAI's guide. Medium confidence on the topic, high on a design round.

</details>

<a id="q10"></a>

#### Q10. (System design, no AI) A partner with 40 internal services wants ChatGPT and Codex agents to act for its users through one MCP endpoint. Design the gateway: per-tool permissions, token checks, write safety, audit, rate limits, and multi-account users.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: MCP gateway, authorization, multi-tenancy.

<details>
<summary>Show answer</summary>

Task: a gateway design with the auth path, the permission model, capacity numbers, and the controls an enterprise security review will ask for.

Approach: one public MCP server (the resource server) in front of internal services. It publishes protected resource metadata, verifies every bearer token locally (signature through cached JWKS, issuer, audience equal to the resource, expiry, scopes), then maps each tool to a required scope and an internal service. It calls internal services with a short-lived internal token that carries the user id and the scopes actually granted, never the public token, so internal services cannot be called with a token minted for someone else. Write tools (create, delete, pay) carry a write scope and return a preview first; the agent must call a confirm tool with the preview id, which gives the user a checkpoint. Optionally the gateway checks ChatGPT's mTLS client certificate to confirm the caller is OpenAI, while OAuth still identifies the user.

Numbers: 2 million tool calls a day is 23 per second, about 230 at a 10x peak. Local JWT checks cost under 1 ms, so 3 small instances suffice; introspection per call would add 10 to 30 ms and make the IdP a hard dependency. Audit: about 1 KB per call, 2 GB a day, 730 GB a year before compression, fine for an object store with a 13 month retention and a queryable index on user and time.

Alternatives: 40 separate MCP servers give teams autonomy but 40 OAuth integrations and 40 security reviews; a generated gateway from service schemas saves effort but exposes internal APIs one to one, which makes poor tools for a model. A curated gateway with about 15 task-shaped tools wins.

Risks: confused deputy (the agent fetching data for one account into another): every call carries the selected connection's credentials, and the gateway never takes a user id as a tool argument. Multi-account users: expose a read-only profile tool returning a stable opaque id, as OpenAI's docs describe, so personal and work accounts stay distinct. Prompt-injected content triggering writes: the confirm step plus per-user rate limits on write tools. Revocation: access tokens of 15 minutes, so revoking the refresh token ends access within that window.

Follow-ups: how do you roll out a new scope without breaking existing links? How do you let an enterprise admin restrict which workspaces can link?

Why this is likely: the job describes building MCP servers, connector OAuth and agent-auth primitives with partners, and candidate reports describe OpenAI design rounds that push on multi-tenancy and reliability. Medium confidence on topic.

</details>

<a id="q11"></a>

#### Q11. (System design, no AI) A developer-tools partner ships a CLI that coding agents run on laptops, CI machines and remote SSH boxes. Design how its users sign in with ChatGPT from that CLI and how the CLI stores and refreshes credentials.

Round: System design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Native and CLI OAuth, token storage.

<details>
<summary>Show answer</summary>

Task: choose the sign-in grant per environment, define where tokens live and how they refresh, and state what the partner must ask OpenAI to support.

Approach: CLIs are public clients: anything shipped in the binary is not secret, so PKCE is mandatory and there is no client secret. Each environment gets its own grant:

1. Laptop with a browser: authorization code with PKCE and a loopback redirect (http://127.0.0.1 on an ephemeral port), the standard pattern for native apps in RFC 8252. The CLI opens the browser, listens once, checks state, exchanges the code.
2. Remote SSH box without a browser: the device authorization grant (RFC 8628), where the user enters a short code on another device, if the authorization server supports it; otherwise print the authorization URL and accept the redirect URL pasted back.
3. CI and unattended agents: no interactive user, so no user sign-in at all. Use a scoped, revocable machine credential from the partner's own platform, never a copied user refresh token.

Storage: the OS keychain on laptops; a file with 0600 permissions as a fallback, with a warning. Access tokens short lived, refresh tokens rotated on every use, and a single-flight refresh so parallel agent processes do not trip reuse detection (see Q7).

Numbers: a sign-in costs one browser round trip and one token call, about 2 seconds of user time. With 1 hour access tokens and 20 short agent runs a day, each run refreshes at most once, so at most 20 refreshes per user per day (24 even under continuous use), which is negligible load. A device code that expires in 10 minutes with a 5 second polling interval means at most 120 polls per sign-in.

Alternatives: pasting a long-lived API key is simplest but leaks into shell history and CI logs and never expires; embedding a client secret in the CLI is false security. Loopback plus PKCE wins where a browser exists.

Risks: another local process grabbing the loopback port (bind to 127.0.0.1 only, random port, verify state); tokens in agent transcripts (the CLI never prints them, and redacts in debug logs); headless agents prompting for sign-in in the middle of a task (fail fast with a clear message).

Follow-ups: how does sign-out revoke tokens on the server as well as deleting the local copy? What changes if the CLI is open source?

Why this is likely: the job lists CLIs, coding harnesses and agent-auth primitives among the integration paths this team builds with partners, and OpenAI's design rounds are reported to push on trade-offs. Low to medium confidence on this exact topic.

</details>

### Customer case (no AI)

<a id="q12"></a>

#### Q12. (Customer case, no AI) The interviewer plays a travel company's CTO: "We want ChatGPT agents to search and book trips for our members, using their member accounts. Launch in six weeks." Scope the first integration, pick the integration path, and write the API contract for the first tools.

Round: Customer case. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Partner scoping, MVP, tool contracts.

<details>
<summary>Show answer</summary>

Task: ask the right questions first, then a scoped plan with a path, milestones, risks and a first tool contract.

Approach: ask scoping questions before drawing any boxes. Who are the users (members only, or also guests)? What does "book" include: payment on file, loyalty points, cancellations? What is their identity provider and does it support OAuth 2.1 with PKCE and discovery? What are their hard rules: payment confirmation, fraud checks, regulators? How will they measure success: bookings, conversion, support tickets?

Then the path. Two options. (a) Sign in with ChatGPT on their website, which lets members sign in to the partner site with ChatGPT but does not let an agent act in ChatGPT. (b) An MCP server published as a plugin, with connector OAuth linking the member account, so agents in ChatGPT and Codex can call their tools. The ask is (b); (a) is an optional later step for new member sign-ups.

Scope for six weeks: week 1 to 2, read-only tools (search_trips, get_trip, list_bookings) with account linking; week 3 to 4, a two-step booking (create_booking_quote returns a priced quote id with an expiry; confirm_booking takes the quote id), so the user sees price and rules before anything is charged; week 5, evals and a staged rollout to employees; week 6, partner beta. Cancellations wait for phase two.

Numbers: if 1% of their 5 million members try it in month one, that is 50,000 users; at 10 tool calls per session and 2 sessions each, 1 million calls a month, about 0.4 per second average. That load is trivial, so plan around correctness and trust.

Alternatives: one generic "book" tool is faster to build but lets the model charge cards in one step; exposing their full internal API is complete but gives the model 80 poorly named endpoints. Four task-shaped tools with a quote-then-confirm write path win.

Risks: price changes between quote and confirm (quotes expire in 15 minutes and confirm re-checks the price); wrong member account (stable profile id per linked account); their IdP missing discovery or S256 support, checked in week 1, which leaves weeks 2 to 5 to close any gap.

Follow-ups: the CTO pushes back: "Just let it book in one step." What do you say? What do you need from their security team this week?

Why this is likely: a first-hand Blind report on OpenAI's deployment track describes a live case where an engineer plays a fictional customer and scoping questions come before design, an FDE candidate blog describes a solution design round, and the job asks for scoping projects, MVPs and API contracts with partners. Medium confidence.

</details>

<a id="q13"></a>

#### Q13. (Customer case, no AI) An enterprise partner's security team blocks the launch: "We will not allow dynamic client registration, and we need proof that calls come from OpenAI and that corporate users cannot link our app from personal ChatGPT accounts." Propose a design they can approve.

Round: Customer case. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: CIMD, DCR, mTLS, workspace domain restrictions.

<details>
<summary>Show answer</summary>

Task: address each concern with a concrete mechanism, give the trade-offs, and leave the security team with steps to verify.

Approach: take the three concerns one at a time.

1. "No DCR": their worry is unbounded client registrations they cannot audit. Offer Client ID Metadata Documents (CIMD). OpenAI's plugin docs describe CIMD as the preferred method: ChatGPT presents a stable HTTPS metadata document as its client id, so there is no registration step and one stable client identity the partner can attach policies, redirect allowlists and rate limits to. For token endpoint authentication, use private_key_jwt: ChatGPT signs the client assertion with a managed key whose public JWKS is published from the metadata origin, so the partner verifies the signature and no shared secret exists. Fallback: a predefined OAuth client created by their admins, at the cost of manual secret rotation.
2. "Prove calls come from OpenAI": ChatGPT presents an OpenAI-managed client certificate on TLS connections to MCP servers. The partner validates that the chain leads to OpenAI's connectors intermediate CA and that the leaf's SAN is mtls.prod.connectors.openai.com, without pinning the leaf, which rotates. mTLS authenticates the client; OAuth still authenticates the user.
3. "No personal workspaces": enable workspace domain restrictions. The partner advertises openid and email scopes and a UserInfo endpoint returning email and email_verified true; an Enterprise workspace that has verified the corporate domain then stops that identity from linking in a personal or outside workspace.

Alternatives: DCR with an approval queue meets the letter of their policy but adds manual work per connection; an IP allowlist of OpenAI egress ranges is easy but weaker than mTLS on its own. CIMD plus private_key_jwt plus mTLS is the strongest set and is all documented.

Numbers: CIMD means one client record instead of one per connection; with 20,000 employees that is 1 record instead of up to 20,000 DCR clients to audit.

Risks: their IdP may not support CIMD yet (check in the first call; fall back to a predefined client); a strict firewall in front of the MCP server may terminate TLS before the app sees the client certificate (validate at the edge and forward a verified header).

Follow-ups: how would you test all three controls in staging before their review? What do you tell them if their IdP vendor needs a quarter to add CIMD?

Why this is likely: the job names enterprise constraints, connector OAuth and identity and permissioning as core work, OpenAI's plugin authentication docs describe CIMD, DCR, predefined clients, mTLS and workspace domain restrictions in exactly these terms, and candidate reports describe customer case rounds. Medium confidence.

</details>

### Evals and rollout (no AI)

<a id="q14"></a>

#### Q14. (Project deep dive or hiring manager round, no AI) A partner is replacing its email sign-up form with "Continue with ChatGPT". Design the staged rollout: the metrics, the sample size needed to catch a 2-point drop in sign-in completion from a 72% baseline, and the rules for moving between stages or rolling back.

Round: Project deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Staged rollout, experiment sizing.

<details>
<summary>Show answer</summary>

Task: a rollout plan with a primary metric, guardrails, a sample size you can defend, and stage gates.

What they grade: picking metrics that tie to partner value, correct power math, guardrails that catch silent failures, and rollback rules decided before launch.

Approach: primary metric: sign-in completion, meaning callbacks that end in a session divided by button clicks. Guardrails: callback error rate by class (state, nonce, exchange, verification), duplicate accounts created per 1,000 sign-ins, support tickets tagged sign-in, and p95 callback latency. The partner's business metric (members who finish onboarding in 7 days) is tracked but too slow to gate stages.

Sample size for a two-sided test at alpha 0.05 and 80% power, detecting 72% falling to 70%: n per arm = (1.96 + 0.84)^2 x (0.72 x 0.28 + 0.70 x 0.30) / 0.02^2, about 8,100 clicks per arm (the code computes it exactly). At 40,000 sign-in clicks a day, a 25% treatment arm collects that in under a day; a 1% canary (about 400 clicks a day against a very large control) still needs about 4,100 treatment clicks, roughly 10 days, so the canary gates on guardrails (errors, duplicates), not on completion.

Stages: 1% for 2 days (guardrails only: error rate under 1% and zero duplicate accounts), 10% for 2 days, 50% for 3 days (powered completion test), then 100% with the old form kept behind a flag for 2 weeks. Rollback is automatic if callback errors exceed 2% for 15 minutes.

Alternatives: launching to 100% with before/after comparison is faster but confounded by weekday and campaign effects; a full 50/50 test from day one powers the test fastest but exposes half of users to unknown failure modes. A staged rollout with guardrail-only early stages wins.

Code:

```python
from math import sqrt, ceil
from statistics import NormalDist

def n_per_arm(p1, p2, alpha=0.05, power=0.8):
    z = NormalDist().inv_cdf(1 - alpha / 2) + NormalDist().inv_cdf(power)
    return ceil(z * z * (p1 * (1 - p1) + p2 * (1 - p2)) / (p1 - p2) ** 2)

def diff_ci(success_a, n_a, success_b, n_b, level=0.95):
    pa, pb = success_a / n_a, success_b / n_b
    se = sqrt(pa * (1 - pa) / n_a + pb * (1 - pb) / n_b)
    z = NormalDist().inv_cdf(0.5 + level / 2)
    return pb - pa, (pb - pa - z * se, pb - pa + z * se)

n = n_per_arm(0.72, 0.70)
print("clicks per arm:", n)
assert 8000 <= n <= 8200
diff, (lo, hi) = diff_ci(7200, 10000, 7050, 10000)
print(f"diff {diff:+.3f}, 95% CI ({lo:+.3f}, {hi:+.3f})")
assert hi < 0                      # a real 1.5-point drop at this size
diff, (lo, hi) = diff_ci(7200, 10000, 7230, 10000)
assert lo < 0 < hi                 # no detectable change
print("q14 ok")
```

Verify: the sample size lands near 8,100; a 1.5-point drop on 10,000 per arm gives an interval entirely below zero; a 0.3-point rise gives an interval spanning zero.

Pitfalls: counting page views instead of button clicks as the denominator; peeking daily at the completion test without correction; no guardrail for duplicate accounts, the costliest silent failure.

Follow-ups: completion rises but 7-day onboarding falls: what do you do? How do you attribute a drop to OpenAI's side versus the partner's?

Why this is likely: the job prefers experience with evaluations, staged rollouts and launch operations, and candidate reports on OpenAI's customer-facing tracks describe deep dives that ask how you knew a launch worked, with numbers. Medium confidence.

</details>

<a id="q15"></a>

#### Q15. (Project deep dive, no AI) Before a partner's MCP plugin launches, you must show that agents use its tools correctly. Design the eval: the task set, graders, the reliability metric, and the bar for launch. Then compute the metrics from these trial results.

Round: Project deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Agent and tool-use evals.

```text
task_id  category        trials (1 = pass, graded on final account state)
T01      search          1 1 1 1 1
T02      search          1 1 1 1 0
T03      booking_quote   1 0 1 1 1
T04      booking_confirm 1 1 0 1 0
T05      booking_confirm 1 1 1 1 1
T06      multi_account   0 1 0 0 1
T07      auth_expired    1 1 1 1 1
T08      cancel_refused  1 1 1 0 1
```

<details>
<summary>Show answer</summary>

Task: an eval plan a partner and OpenAI can both trust, plus the numbers from the sample results.

What they grade: graders tied to real outcomes, a reliability metric suited to actions with side effects, honest handling of small samples, and a launch bar set before looking at results.

Approach: build about 200 tasks from the partner's real support logs and product flows, in categories: read-only search, quote, confirm, multi-account (personal versus work), auth edge cases (expired token, revoked link), and refusals (actions the tools must not allow). Each task runs in a sandbox account with a known starting state.

Graders, in order of trust: (1) state checks in the sandbox (was exactly one booking created, with the right dates and price), deterministic and cheap; (2) tool-call checks (right tool, valid arguments, quote before confirm, correct account); (3) an LLM judge only for the final message's clarity, calibrated against 100 human-labelled transcripts and used only if agreement is high, for example Cohen's kappa of 0.8 or more.

Reliability: for actions that charge money, average pass rate hides flakiness. Report pass^k, the share of tasks that pass all k trials. With 5 trials per task it shows whether the user can rely on the agent every time.

Launch bar, set in advance: every write category at pass^5 of 0.9 or higher on 200 tasks, zero wrong-account writes, zero unconfirmed charges. On the sample above, pass^5 is far below that bar, and multi_account is the worst category, so the next step is reading those transcripts before changing any prompt.

Alternatives: LLM-judge-only grading is fast to set up but drifts and misses side effects; human review of every transcript is accurate but costs about 2 minutes per transcript, 2,000 minutes per run. The winning mix puts state checks first, the judge second, and humans on calibration.

Code:

```python
from collections import defaultdict

RAW = """T01 search 1 1 1 1 1
T02 search 1 1 1 1 0
T03 booking_quote 1 0 1 1 1
T04 booking_confirm 1 1 0 1 0
T05 booking_confirm 1 1 1 1 1
T06 multi_account 0 1 0 0 1
T07 auth_expired 1 1 1 1 1
T08 cancel_refused 1 1 1 0 1"""

tasks = []
for line in RAW.splitlines():
    tid, cat, *trials = line.split()
    tasks.append((tid, cat, [int(x) for x in trials]))

pass_rate = sum(sum(t) for _, _, t in tasks) / sum(len(t) for _, _, t in tasks)
pass_hat_5 = sum(all(t) for _, _, t in tasks) / len(tasks)
by_cat = defaultdict(list)
for _, cat, t in tasks:
    by_cat[cat].append(sum(t) / len(t))
worst = min(by_cat, key=lambda c: sum(by_cat[c]) / len(by_cat[c]))
print(f"mean pass rate {pass_rate:.2f}, pass^5 {pass_hat_5:.2f}, worst category {worst}")
assert round(pass_rate, 2) == 0.80 and pass_hat_5 == 3 / 8 and worst == "multi_account"
print("q15 ok")
```

Numbers: on the sample, the mean trial pass rate is 0.80, which looks fine, but pass^5 is 0.375: only 3 of 8 tasks pass every time. With 8 tasks the uncertainty is huge, which is why the real set needs about 200.

Pitfalls: grading only the final message; changing the bar after seeing results; letting the LLM judge see the expected answer and rubber-stamp it.

Follow-ups: the partner changes a tool's schema a week before launch: what reruns? How do you keep the eval from overfitting to the prompt?

Why this is likely: the job prefers experience with evaluations and asks for turning production learnings into durable platform improvements, and an FDE candidate blog describes OpenAI interviewers pressing on how you know a system works. Medium confidence.

</details>

### Hiring manager and behavioral

<a id="q16"></a>

#### Q16. (Hiring manager or behavioral, no AI) Tell me about a time you used an AI coding agent to deliver a partner integration under deadline pressure and the agent produced something wrong that looked right. How did you catch it, what did you tell the partner, and what did you change afterwards?

Round: Hiring manager / behavioral. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI use and verification, partner communication.

<details>
<summary>Show answer</summary>

Task: one specific story covering the defect, how you caught it, the partner conversation, and a lasting change in how you work.

What they grade: ownership of AI-written code, verification habits, honest and calm partner communication, and a change other people adopted.

Approach: use a situation, action, result shape but spend most of the time on the catch and on the partner. A strong answer sounds like this:

"We had nine days to ship an OAuth account-linking integration for a partner's launch. I used an agent to write the token refresh path and the webhook signature check. The tests passed. When I read the diff I saw the signature check compared HMACs with ==, and the refresh code retried on any 4xx, including invalid_grant. The second one mattered: under load it would keep hammering a revoked token and the partner's server would lock the account. I wrote tests for both, watched them fail, fixed them with a constant-time compare and a retry list limited to timeouts and 5xx, and ran a load test with parallel calls. That cost a day. I told the partner's lead that day, with the reason and the new date for the staging build, and offered to cut the 'cancel booking' tool from launch to keep their date. They agreed. We launched on time with zero auth incidents in the first month. Afterwards I changed how I delegate: I write the edge-case tests before prompting, I keep each agent task to one reviewable change, and I read every line touching auth, retries or money. The team turned that into a checklist for AI-written pull requests."

Verify: the story has a concrete defect, your own role, a number, a partner conversation and a lasting change.

Pitfalls: a story where AI was wrong and you rewrote everything by hand; hiding the slip from the partner; no process change; claiming you never use AI.

Follow-ups: when do you let an agent run without review? How do you tell a partner executive their date is not realistic? Recruiter round: be ready for "Why OpenAI?" and your view on AI safety (recent 1point3acres roundup).

Why this is likely: OpenAI's agentic round reportedly grades validating AI output and owning the result, the official guide says AI expectations vary by round, and the hiring manager round on customer-facing tracks focuses on partner stories and ownership. Medium confidence.

</details>

## Prep checklist

- [ ] Read OpenAI's interview guide and ask your recruiter which rounds allow AI, whether you get the agentic option, and what the take-home looks like.
- [ ] Implement "Continue with ChatGPT" end to end on a toy site from OpenAI's guide: state, PKCE, nonce, code exchange, ID token checks, account linking (Q1, Q6, Q9).
- [ ] Build a small authenticated MCP server: protected resource metadata, the 401 challenge, token checks and per-tool scopes (Q5, Q10).
- [ ] Learn the client registration choices (CIMD, DCR, predefined) and when an enterprise picks each, plus mTLS client checks (Q8, Q13).
- [ ] Practise multi-part stateful builds in 60 minutes without AI, with expiry, late events and retries (Q1 to Q4).
- [ ] Do one timed agentic session on an unfamiliar repo: plan first, delegate small steps, read every diff, run tests (Q5).
- [ ] Practise reviewing AI-written auth code for hallucinated options, loose comparisons and logging of secrets (Q6).
- [ ] Rehearse a partner case out loud: five scoping questions before any design, then a six-week plan (Q12).
- [ ] Prepare rollout math (sample size, confidence intervals) and an agent eval with pass^k (Q14, Q15).
- [ ] Prepare a deep dive on an integration you shipped, with numbers, plus one story about catching wrong AI output (Q16).

## Sources

- OpenAI interview guide (official): https://openai.com/interview-guide/ (high)
- Sign in with ChatGPT website guide (official docs): https://developers.openai.com/siwc/website (high)
- Plugin and MCP authentication (official docs): https://developers.openai.com/plugins/build/auth (high)
- Blind, first-hand report on the agentic coding option: https://www.teamblind.com/post/openai-agentic-code-interview-up8jhuiq (medium)
- Blind, AI Deployment Engineer and Solutions Architect thread: https://www.teamblind.com/post/open-ai-ai-deployment-engineer-solutions-architect-gylqsu7u (medium)
- Glassdoor OpenAI interview reviews: https://www.glassdoor.com/Interview/OpenAI-Interview-Questions-E2210885.htm (medium)
- 1point3acres roundup of OpenAI interview reports: https://www.1point3acres.com/bbs/thread-1190155-1-1.html (medium)
- DataInterview, OpenAI Forward Deployed Engineer interview: https://www.datainterview.com/blog/openai-forward-deployed-engineer-interview (medium)
- Gaijineer, OpenAI Forward Deployed Engineer interview process: https://gaijineer.co/openai-forward-deployed-engineer-interview-process (medium)
- Reddit, OpenAI Applied AI Engineer interview prep thread: https://www.reddit.com/r/leetcode/comments/1xp7eeb/openai_applied_ai_engineer_interview_prep/ (low)
- Coditioning, OpenAI system design questions: https://www.coditioning.com/blog/16/openai-system-design-interview-questions (low)
- PracHub OpenAI software engineer guide, counter-evidence on AI-assisted coding: https://prachub.com/interview-guide/openai-software-engineer-interview-guide (medium)
- RFC 7636, PKCE: https://www.rfc-editor.org/rfc/rfc7636 (high)
- RFC 8252, OAuth for native apps: https://www.rfc-editor.org/rfc/rfc8252 (high)
- RFC 8628, device authorization grant: https://www.rfc-editor.org/rfc/rfc8628 (high)
- RFC 9207, authorization server issuer identification: https://www.rfc-editor.org/rfc/rfc9207 (high)
- Company-wise LeetCode lists, practice signal only: https://github.com/liquidslr/leetcode-company-wise-problems (low)
