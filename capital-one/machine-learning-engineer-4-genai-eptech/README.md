# Capital One: Machine Learning Engineer 4 (Python, AWS, SQL, GenAI), Enterprise Platforms Technology

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Capital One publishes one interview guide for its AI and ML roles. 2026 Machine Learning Engineer candidate reports match it closely: a CodeSignal assessment, then a virtual Power Day of four equally weighted rounds. There is no team-specific loop for the marketing and messaging team, so the domain focus below comes from the posting. Confirm the rounds with your recruiter.

1. Recruiter screen (about 30 minutes, phone or video). Background, ML experience, Python and cloud tools, and how you have deployed models. AI: not allowed.
2. CodeSignal online assessment (70 minutes, four data structures and algorithms questions, pass or fail). MLE reports describe two easy, one medium and one medium-hard question, with screen recording to prove no outside help. One report describes a longer multi-part assessment instead. AI: not allowed.
3. Hiring manager interview (about 30 minutes). Your resume and past projects. Some loops appear to skip it. AI: not allowed.
4. Power Day, technical interview (60 minutes). 30 minutes of live coding in CodeSignal, then 30 minutes going deep on ML skills tied to the team: response and ranking models, GenAI for message copy, and running models in production. AI: not provided in MLE reports.
5. Power Day, ML system design (60 minutes). An end-to-end system built on modern LLM concepts. For this team, expect personalized messaging across owned and paid channels, with guardrails. AI: not allowed.
6. Power Day, behavioral (60 minutes). STAR stories probed with a second layer of "why" questions. 2026 engineering reports add a question on how you use AI at work. AI: not allowed.
7. Power Day, case study (60 minutes). A business case on a Capital One product. Autumn 2026 engineering reports describe a case that moves into reading, dry-running and changing given code. AI: not allowed.
8. Possible agentic coding round. From late 2026, engineering loops added a CodeSignal round with Claude Code: fix a bug in an unfamiliar codebase, then build a feature. No MLE report shows it yet. AI: provided (Claude Code).

Questions marked "no AI, as at Capital One" follow the proctored assessment and the MLE Power Day reports. Two classic algorithm questions are included, because the assessment and the 30-minute live coding block still use them. Q6 practices the agentic format in case your loop includes it. The rest focus on what this team builds: personalized messaging, GenAI copy with guardrails, Spark and SQL pipelines, and models in production.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | You get a row of n bits, all 0, and a list of commands: "L" turns the leftmost 0... | Classic algorithm | Online assessment | Not allowed | Medium |
| [Q2](#q2) | Build a request tracker in levels. Level 1: count the calls in the last 3,000 ms... | Classic algorithm | Power Day technical 1 | Not allowed | Medium |
| [Q3](#q3) | This Spark SQL builds training rows for a message response model. The model looks... | Debug | Power Day technical 1 | Not allowed | Medium |
| [Q4](#q4) | An AI assistant wrote this helper that turns LLM output into a final marketing... | Review | Power Day case study | Not allowed | Medium |
| [Q5](#q5) | Read this message eligibility function, dry-run the three inputs below without... | Build | Power Day case study | Not allowed | Medium |
| [Q6](#q6) | In this scoring service, `test_score_matches_training` fails: online scores do not... | Debug | Agentic technical interview | Provided (Claude Code) | Medium |
| [Q7](#q7) | A 90-day response model for credit card offers scores train AUC 0.93, random-split... | Concept plus code | Power Day technical 1 | Not allowed | Medium |
| [Q8](#q8) | Your team ranks next-best messages with gradient-boosted trees on 400 aggregate... | Design | Power Day technical 1 | Not allowed | Hard |
| [Q9](#q9) | An LLM writes personalized message copy, and an LLM judge checks it for brand and... | Concept plus code | Power Day technical 1 | Not allowed | Hard |
| [Q10](#q10) | The daily monitoring log below shows sends jumping and conversion dropping on day 9... | Trace debug | Power Day technical 1 | Not allowed | Medium |
| [Q11](#q11) | A generated email told customers they earn 3x miles for 15 months, but the live... | Trace debug | Power Day technical 1 | Not allowed | Medium |
| [Q12](#q12) | Design a platform that picks the next best message for each customer across email... | Design | Power Day ML system design | Not allowed | Hard |
| [Q13](#q13) | Should the LLM write each customer's message in real time, or should you generate... | Design | Power Day ML system design | Not allowed | Medium |
| [Q14](#q14) | Design how a retrained response model goes from training to 100% of traffic on... | Design | Power Day ML system design | Not allowed | Medium |
| [Q15](#q15) | Size the compute for scoring: nightly batch for scheduled campaigns plus real-time... | Concept plus code | Power Day ML system design | Not allowed | Medium |
| [Q16](#q16) | Design the test that decides whether the new message model beats the current one... | Concept plus code | Power Day case study | Not allowed | Medium |
| [Q17](#q17) | Model risk review asks two things before launch: per-customer reasons for each... | Concept plus code | Power Day technical 1 | Not allowed | Medium |
| [Q18](#q18) | Tell me about a time you improved an ML pipeline you owned. Where did you use AI... | Behavioral | Power Day behavioral | Not allowed | Medium |

### Coding rounds (CodeSignal)

<a id="q1"></a>

#### Q1. (Online assessment, no AI, as at Capital One) You get a row of n bits, all 0, and a list of commands: "L" turns the leftmost 0 into 1 and outputs its index (or -1 if there is no 0), and "C i" turns bit i back to 0. Return the outputs fast enough for 100,000 commands.

Round: Online assessment (CodeSignal). AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Stateful simulation, heaps.
Related practice: [Design Memory Allocator](https://leetcode.com/problems/design-memory-allocator/) (Medium), [Simple Bank System](https://leetcode.com/problems/simple-bank-system/) (Medium), [Number of Adjacent Elements With the Same Color](https://leetcode.com/problems/number-of-adjacent-elements-with-the-same-color/) (Medium).

<details>
<summary>Show answer</summary>

Task: write `run(n, commands)` that returns the index produced by every "L" command. A "C" on a bit that is already 0 does nothing.

What they grade: a correct simulation first, then a data structure that avoids scanning the array, and edge cases (array full, clearing a 0 twice).

Approach: the naive answer scans from the left for every "L", which is O(n) per command and O(n times q) overall: 10 billion steps for n = q = 100,000, which times out. The question only ever asks for the smallest index that is 0, so keep a min-heap that holds exactly the indices that are currently 0. "L" pops the smallest index and sets it. "C i" pushes i back, but only if bit i was 1; otherwise the heap would hold a duplicate and a later "L" would hand out the same index twice. Each command is O(log n), and memory is O(n).

Alternatives: a sorted set (or `bisect` on a sorted list) gives the same answer but list inserts are O(n) in Python; a segment tree over the bits answers "first zero" in O(log n) and also supports range queries, but it is about 30 lines more and easy to get wrong in a 70-minute, four-question test. The heap is the quickest to write correctly.

Code:

```python
import heapq
import random


class BitArray:
    def __init__(self, n):
        self.bits = [0] * n
        self.zeros = list(range(n))  # min-heap holding exactly the indices that are 0
        heapq.heapify(self.zeros)

    def set_leftmost(self):
        if not self.zeros:
            return -1
        i = heapq.heappop(self.zeros)
        self.bits[i] = 1
        return i

    def clear(self, i):
        if self.bits[i] == 1:  # clearing a 0 must not push a duplicate
            self.bits[i] = 0
            heapq.heappush(self.zeros, i)


def run(n, commands):
    arr, out = BitArray(n), []
    for cmd in commands:
        if cmd[0] == "L":
            out.append(arr.set_leftmost())
        else:
            arr.clear(cmd[1])
    return out


def brute(n, commands):
    bits, out = [0] * n, []
    for cmd in commands:
        if cmd[0] == "L":
            i = bits.index(0) if 0 in bits else -1
            if i >= 0:
                bits[i] = 1
            out.append(i)
        else:
            bits[cmd[1]] = 0
    return out


assert run(3, [("L",), ("L",), ("C", 0), ("L",), ("L",), ("L",)]) == [0, 1, 0, 2, -1]
assert run(2, [("C", 1), ("C", 1), ("L",), ("L",), ("L",)]) == [0, 1, -1]
random.seed(7)
for _ in range(500):
    n = random.randint(1, 8)
    cmds = [("L",) if random.random() < 0.6 else ("C", random.randrange(n)) for _ in range(30)]
    assert run(n, cmds) == brute(n, cmds)
print("ok")
```

Verify: the hand case, the double-clear case (which fails without the guard), and 500 random command lists checked against the brute-force scan.

Pitfalls: pushing on every clear, returning 0 instead of -1 when full, and reading the command format wrong. Spend a minute on the samples before coding.

Follow-ups: add "R" that returns the rightmost 1 (a second, max-heap with lazy deletion), or count the 1s inside a range (a Fenwick tree).

Why this is likely: Capital One's AI/ML recruiting page describes a 70-minute CodeSignal assessment with four data structures and algorithms questions, and a 2026 MLE candidate report describes a proctored assessment with a stateful bit-array command question (medium confidence on the exact wording).

</details>

<a id="q2"></a>

#### Q2. (Power Day live coding, no AI, as at Capital One) Build a request tracker in levels. Level 1: count the calls in the last 3,000 ms. Level 2: turn it into a message frequency cap, at most 1 send per customer per channel in 24 hours and 3 sends per customer across channels in 7 days.

Round: Power Day technical 1 (30 minutes of live coding). AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Sliding window, object design.
Related practice: [Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/) (Easy).

<details>
<summary>Show answer</summary>

Task: `RecentCounter(window).ping(t)` returns how many pings fall in [t - window, t]. Then `FrequencyCap.try_send(customer, channel, t)` records the send and returns True only when both caps allow it. Times never decrease.

What they grade: a clean level 1 in under 10 minutes, a level 2 design that reuses it, the boundary rule said out loud, and tests.

Approach: a deque of timestamps per key. On each call, pop from the left while the oldest timestamp is older than t - window, then compare the length with the cap. Each timestamp is pushed and popped once, so the cost is amortized O(1) per call and memory is bounded by the cap times the number of active keys. For level 2 keep two deques: one per (customer, channel) for the 24-hour rule and one per customer for the 7-day rule. A blocked send must not be recorded, or a customer who keeps triggering events stays blocked forever.

Alternatives: a fixed-bucket counter (one count per hour) uses constant memory but only approximates the window edge; a database query per send is exact but adds a network round trip to every decision. Use the deque in the interview and in a single process; in production the same logic sits in Redis sorted sets so every scoring pod sees the same counts.

Code:

```python
from collections import defaultdict, deque

DAY = 86_400


def trim(q, t, window):
    while q and q[0] < t - window:  # keep [t - window, t]
        q.popleft()


class RecentCounter:
    def __init__(self, window=3000):
        self.window, self.q = window, deque()

    def ping(self, t):
        self.q.append(t)
        trim(self.q, t, self.window)
        return len(self.q)


class FrequencyCap:
    def __init__(self, per_channel=1, total=3):
        self.per_channel, self.total = per_channel, total
        self.chan = defaultdict(deque)  # (customer, channel) -> send times
        self.cust = defaultdict(deque)  # customer -> send times

    def try_send(self, customer, channel, t):
        qc, qa = self.chan[(customer, channel)], self.cust[customer]
        trim(qc, t, DAY)
        trim(qa, t, 7 * DAY)
        if len(qc) >= self.per_channel or len(qa) >= self.total:
            return False
        qc.append(t)
        qa.append(t)
        return True


rc = RecentCounter()
assert [rc.ping(t) for t in (1, 100, 3001, 3002)] == [1, 2, 3, 3]
fc = FrequencyCap()
assert fc.try_send("a", "email", 0)
assert not fc.try_send("a", "email", DAY)        # exactly 24 hours later is still inside
assert fc.try_send("a", "email", DAY + 1)
assert fc.try_send("a", "sms", DAY + 2)
assert not fc.try_send("a", "push", DAY + 3)     # third send in 7 days already used
assert fc.try_send("a", "push", 7 * DAY + 1)     # the send at 0 has aged out
assert fc.try_send("b", "email", DAY)            # customers are independent
print("ok")
```

Verify: the LeetCode sample for level 1, the exact-boundary send, a blocked send that is not recorded, and two customers that do not interfere.

Pitfalls: an off-by-one on the boundary (decide whether a send exactly 24 hours ago counts, and say it), recording blocked sends, and `defaultdict` creating empty deques on every read.

Follow-ups: level 3, evict idle customers to bound memory; level 4, share state across pods and make check-and-record atomic.

Why this is likely: Capital One's AI/ML page describes 30 minutes of live CodeSignal coding in a Power Day technical round, a 2026 MLE report calls it "30 min leetcode", and another 2026 MLE report names a "RecentRequestTracker" data structure task (medium confidence).

</details>

### Code reading and debugging

<a id="q3"></a>

#### Q3. (Power Day, no AI, as at Capital One) This Spark SQL builds training rows for a message response model. The model looks great offline, but the row count is 40% below the eligible population. Find the bugs and fix the query.

Round: Power Day technical 1 (ML depth) or case study. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Feature pipelines, leakage, joins.

```sql
-- one row per customer who got the message at snapshot_day
SELECT c.customer_id,
       COUNT(e.event_id)  AS opens_90d,
       MAX(e.event_day)   AS last_open_day,
       l.converted        AS label
FROM customers c
JOIN email_events e ON e.customer_id = c.customer_id
JOIN labels l       ON l.customer_id = c.customer_id
WHERE e.event_type = 'open'
  AND e.event_day >= l.snapshot_day - 90
GROUP BY c.customer_id
```

<details>
<summary>Show answer</summary>

Task: explain why rows disappear and why offline metrics are too good, then fix it.

What they grade: reading SQL as a data flow, naming leakage precisely, and proving the fix with a tiny dataset.

Approach: three bugs. First, the inner join to `email_events` drops every customer with no qualifying open. Those customers are mostly non-converters, so the model trains on a base rate that is too high and never sees the coldest customers. Second, there is no upper bound on `event_day`, so opens after the snapshot count. A customer who opened the message they were scored for gets a higher `opens_90d`, which leaks the label and explains the strong offline AUC. Third, `label` is not in the `GROUP BY`. SQLite accepts that, Spark rejects it, and other engines can pick an arbitrary value.

The naive fix, switching to `LEFT JOIN` and keeping the `WHERE`, still drops the rows, because a `WHERE` filter on the right table turns the left join back into an inner join. The filters belong in the `ON` clause. Then check the row count against the labels table and compute features "as of" the snapshot.

Alternatives: a point-in-time join in a feature store (for example Feast or SageMaker Feature Store) removes this class of bug for every model, but takes a platform change; a window function over a daily events table works too but scans more data. For one model, fix the SQL and add a row-count assertion.

Code:

```python
import sqlite3

BUGGY = """SELECT c.customer_id, COUNT(e.event_id), l.converted FROM customers c
JOIN email_events e ON e.customer_id = c.customer_id
JOIN labels l ON l.customer_id = c.customer_id
WHERE e.event_type = 'open' AND e.event_day >= l.snapshot_day - 90
GROUP BY c.customer_id"""
NAIVE_LEFT = BUGGY.replace("JOIN email_events", "LEFT JOIN email_events")
FIXED = """SELECT c.customer_id, COUNT(e.event_id), l.converted FROM customers c
JOIN labels l ON l.customer_id = c.customer_id
LEFT JOIN email_events e ON e.customer_id = c.customer_id
 AND e.event_type = 'open'
 AND e.event_day >= l.snapshot_day - 90
 AND e.event_day < l.snapshot_day
GROUP BY c.customer_id, l.converted ORDER BY c.customer_id"""

db = sqlite3.connect(":memory:")
db.executescript("""
CREATE TABLE customers(customer_id INT);
CREATE TABLE labels(customer_id INT, snapshot_day INT, converted INT);
CREATE TABLE email_events(event_id INT, customer_id INT, event_type TEXT, event_day INT);
INSERT INTO customers VALUES (1), (2), (3);
INSERT INTO labels VALUES (1, 100, 1), (2, 100, 0), (3, 100, 0);
INSERT INTO email_events VALUES (10, 1, 'open', 50), (11, 1, 'open', 105),
  (12, 1, 'click', 60), (13, 2, 'open', 5);
""")
assert db.execute(BUGGY).fetchall() == [(1, 2, 1)]       # 2 rows lost, post-snapshot open counted
assert len(db.execute(NAIVE_LEFT).fetchall()) == 1       # WHERE still drops rows
assert db.execute(FIXED).fetchall() == [(1, 1, 1), (2, 0, 0), (3, 0, 0)]
print("ok")
```

Verify: one row per labelled customer, zero counts for silent customers, and no event on or after the snapshot.

Pitfalls: `COUNT(*)` instead of `COUNT(e.event_id)` (counts 1 for customers with no events), and fixing the leak only in training while serving still computes the feature differently.

Follow-ups: how to catch this automatically (assert row counts against the labels table, and compare offline and online feature distributions).

Why this is likely: the posting asks for Spark and SQL data pipelines that feed ML models, and Capital One's Power Day case and technical rounds are reported to include reading code and explaining what it does without running it (medium confidence).

</details>

<a id="q4"></a>

#### Q4. (Power Day, no AI, as at Capital One) An AI assistant wrote this helper that turns LLM output into a final marketing message. Review it like a pull request: find the defects, decide whether to merge, and write the tests that would have caught them.

Round: Power Day case study or technical 1. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: GenAI guardrails, code review.

```python
import json
import logging
import string

log = logging.getLogger("msg")
BANNED = ["guaranteed approval", "risk-free", "act now"]
TEMPLATE = string.Template("Hi $first_name, $body")


def build_prompt(customer, offer):
    return ("Write a short marketing message for this customer.\n"
            f"Customer: {json.dumps(customer)}\n"
            f"Offer: {offer['summary']}\n")


def finalize(customer, llm_text, max_chars=160):
    log.info("generated for %s: %s", customer, llm_text)
    text = TEMPLATE.safe_format(first_name=customer.get("first_name"), body=llm_text)
    if any(p in text for p in BANNED):
        return None
    return text[: max_chars + 1]
```

<details>
<summary>Show answer</summary>

Task: list each defect with its impact, give a merge decision, and show the fix with tests.

What they grade: catching the hallucinated API, the privacy issue, the off-by-one and the missing edge cases, and ranking them by risk.

Approach: do not merge. In order of risk:

1. Privacy and security: `json.dumps(customer)` sends the whole customer record to the LLM, and the log line writes it in plain text. A bank customer record can hold account and identity data. Send an allowlist of fields and log only an id.
2. Hallucinated API: `string.Template` has `safe_substitute`, not `safe_format`, so every call raises `AttributeError`. A test that mocks `finalize` would never see it.
3. Off-by-one: `text[: max_chars + 1]` returns 161 characters. Truncating is also wrong here, because cutting text can drop a required disclosure. Reject over-length text and regenerate.
4. Missing edge cases: a missing name renders "Hi None,", and the banned-phrase check is case-sensitive, so "Act   now" passes.

Alternatives: a block list is cheap but weak on paraphrases; an LLM or classifier compliance check catches paraphrases at extra latency and cost; pre-approved templates with slot filling are the safest. I would keep the block list as a fast first layer and add a reviewed check behind it.

Code:

```python
import logging
import re
import string

log = logging.getLogger("msg")
BANNED = ["guaranteed approval", "risk-free", "act now"]
TEMPLATE = string.Template("Hi $first_name, $body")
PROMPT_FIELDS = ("first_name", "segment", "preferred_channel")


def build_prompt(customer, offer):
    safe = {k: customer[k] for k in PROMPT_FIELDS if customer.get(k)}
    return ("Write a short marketing message for this customer.\n"
            f"Customer: {safe}\nOffer: {offer['summary']}\n")


def finalize(customer, llm_text, max_chars=160):
    name = (customer.get("first_name") or "").strip() or "there"
    text = TEMPLATE.safe_substitute(first_name=name, body=llm_text.strip())
    norm = re.sub(r"\s+", " ", text).lower()
    if any(p in norm for p in BANNED) or len(text) > max_chars:
        log.info("rejected message for customer_id=%s", customer.get("customer_id"))
        return None
    return text


c = {"customer_id": 42, "first_name": None, "tax_id": "000-00-0000", "segment": "travel"}
assert finalize(c, "your travel offer is here.") == "Hi there, your travel offer is here."
assert finalize({"first_name": "Ana"}, "ACT   NOW for miles") is None
assert finalize({"first_name": "Ana"}, "x" * 153) is None        # 161 characters
assert len(finalize({"first_name": "Ana"}, "x" * 152)) == 160
prompt = build_prompt(c, {"summary": "2x miles"})
assert "tax_id" not in prompt and "000-00" not in prompt
print("ok")
```

Verify: one test per defect, each failing on the original code.

Pitfalls: stopping at the first crash, and approving with "minor comments" when there is a data leak.

Follow-ups: how would you stop prompt injection through a customer-editable field such as a nickname?

Why this is likely: candidate reports describe Capital One's 2026 engineering case rounds as reading and reasoning about given code for an AI feature, and the posting asks for Responsible AI and secure, well-managed code (medium confidence). The AI-generated-code framing is a practice format; no MLE report describes reviewing AI output in the case round (low confidence on that part).

</details>

<a id="q5"></a>

#### Q5. (Power Day case study, no AI, as at Capital One) Read this message eligibility function, dry-run the three inputs below without running the code, then extend it for a new rule: paid social retargeting must skip anyone who has opted out of any channel or is behind on payments at all.

Round: Power Day case study. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Code reading, business rules.

```python
QUIET_START, QUIET_END = 21, 8  # local hours, 9 pm to 8 am


def eligible(c, channel, local_hour, sends_7d):
    if c["opted_out"].get(channel):
        return False, "opted_out"
    if c["delinquent_days"] > 30 and channel != "email":
        return False, "collections_hold"
    if QUIET_START <= local_hour or local_hour < QUIET_END:
        if channel in ("sms", "push"):
            return False, "quiet_hours"
    if sends_7d >= 3:
        return False, "frequency_cap"
    return True, "ok"

# A: {"opted_out": {"email": True}, "delinquent_days": 0}, "sms", 22, 0
# B: {"opted_out": {}, "delinquent_days": 45}, "email", 10, 3
# C: {"opted_out": {}, "delinquent_days": 0}, "push", 8, 2
```

<details>
<summary>Show answer</summary>

Task: give the output of each call, raise the business questions the code hides, then add the paid social rule with tests.

What they grade: careful tracing, spotting behavior that is legal Python but questionable policy, and a small, tested change.

Approach: trace each input. A: the opt-out is for email, not SMS, so it passes the first check; hour 22 is quiet and the channel is SMS, so the result is `(False, "quiet_hours")`. B: 45 days late, but email is exempt from the collections hold; hour 10 is not quiet; 3 sends in 7 days hits the cap, so `(False, "frequency_cap")`. C: hour 8 is not quiet because the end is exclusive, so `(True, "ok")`.

Then say what you would ask the business: should a customer 45 days late really get marketing email? Should an email opt-out also stop SMS? Only the first failing reason is returned, which makes audit reports undercount the other reasons.

For the new rule, add a check at the top for `paid_social`: block if any opt-out flag is true or `delinquent_days > 0`. Put it first so the reason is specific.

Alternatives: a rules table (rule name, channel, predicate) is easier for compliance to review and makes every reason visible, but it is a bigger change; a separate function per channel duplicates the shared checks. In a 60-minute case, make the small tested change and propose the rules table as the next step.

Code:

```python
QUIET_START, QUIET_END = 21, 8


def eligible(c, channel, local_hour, sends_7d):
    if channel == "paid_social":
        if any(c["opted_out"].values()) or c["delinquent_days"] > 0:
            return False, "paid_social_exclusion"
    if c["opted_out"].get(channel):
        return False, "opted_out"
    if c["delinquent_days"] > 30 and channel != "email":
        return False, "collections_hold"
    if QUIET_START <= local_hour or local_hour < QUIET_END:
        if channel in ("sms", "push"):
            return False, "quiet_hours"
    if sends_7d >= 3:
        return False, "frequency_cap"
    return True, "ok"


A = {"opted_out": {"email": True}, "delinquent_days": 0}
B = {"opted_out": {}, "delinquent_days": 45}
C = {"opted_out": {}, "delinquent_days": 0}
assert eligible(A, "sms", 22, 0) == (False, "quiet_hours")
assert eligible(B, "email", 10, 3) == (False, "frequency_cap")
assert eligible(C, "push", 8, 2) == (True, "ok")
assert eligible(A, "paid_social", 12, 0) == (False, "paid_social_exclusion")
assert eligible({"opted_out": {}, "delinquent_days": 1}, "paid_social", 12, 0)[0] is False
assert eligible({"opted_out": {"sms": False}, "delinquent_days": 0}, "paid_social", 12, 0) == (True, "ok")
print("ok")
```

Verify: the three dry runs as tests, plus an opt-out flag stored as False, which must not block.

Pitfalls: reading `local_hour < QUIET_END` as inclusive, and treating any key in `opted_out` as an opt-out.

Follow-ups: where does `local_hour` come from for a customer who travels, and how would you backfill the new rule for messages already queued?

Why this is likely: Capital One's AI/ML page describes a case round on its own products, and 2026 Power Day reports describe a business case that moves into reading given code, dry-running inputs and changing it for a new requirement (medium confidence).

</details>

<a id="q6"></a>

#### Q6. (Agentic round with Claude Code, AI assistant provided; documented for engineering tracks, not yet confirmed for MLE) In this scoring service, `test_score_matches_training` fails: online scores do not match offline scores for the same customer. Fix the bug with the agent, then add per-channel send thresholds read from config.

Round: Agentic technical interview (CodeSignal with Claude Code), if your loop includes it. AI policy: Provided (Claude Code). Type: Debug. Difficulty: Medium. Topic: Training and serving skew, multi-file change.

```
scoring-svc/
  pyproject.toml
  config/thresholds.yaml        # {default: 0.5}; feature task adds per-channel keys
  scoring/__init__.py
  scoring/features.py           # build_vector(raw) -> [float(v) for v in raw.values()]
  scoring/model.py              # load() -> weights, bias, feature_names, defaults
  scoring/thresholds.py         # load_thresholds(path) -> dict
  scoring/api.py                # POST /score {customer features, channel} -> {score, send}
  train/export.py               # writes model.json with feature_names = sorted(columns)
  tests/test_features.py
  tests/test_api.py             # FAILING: test_score_matches_training
  tests/fixtures/request.json   # keys in the client's order, one feature missing
```

<details>
<summary>Show answer</summary>

Task: make online scoring use the same feature order and missing-value handling as training, then add `send` decisions with a threshold per channel and a default.

What they grade: reading the code before prompting, a precise delegation plan, checking the diff instead of trusting it, tests, and catching a wrong fix.

Approach: I read `export.py`, `features.py` and the fixture myself first. The root cause is that training writes weights in sorted column order while serving builds the vector in the request's key order, and a missing key shortens the vector so every later weight is misaligned. My prompt to the agent: "In scoring/features.py, build the vector from model.feature_names in that order, use model.defaults for missing keys, return the missing names, and do not edit tests. Then run pytest." In the diff I check three things: the order comes from the model artifact, not a new sort; missing values use the training defaults; no test was changed.

The AI mistake to catch: the agent's first diff sorted `raw.keys()` in `features.py`. The failing test passed because the fixture happened to sort the same way, but a missing feature still shifted every position. I rejected it and asked for the feature-names version plus a test with a missing key. For the feature task I asked for `thresholds.get(channel, thresholds["default"])` and validation that each threshold is between 0 and 1, then checked that an unknown channel falls back to the default.

Alternatives: pinning the vector format in a shared library used by both training and serving fixes skew for good, but it is a bigger change than the round allows; a feature store with one definition gives the same guarantee at platform cost. For a 20-minute fix, read the names from the model artifact.

Code:

```python
import math


def build_vector(raw, feature_names, defaults):
    missing = [f for f in feature_names if f not in raw]
    return [float(raw.get(f, defaults[f])) for f in feature_names], missing


def score(vec, weights, bias):
    return 1 / (1 + math.exp(-(sum(w * x for w, x in zip(weights, vec)) + bias)))


def should_send(p, channel, thresholds):
    t = thresholds.get(channel, thresholds["default"])
    if not 0 < t < 1:
        raise ValueError(f"bad threshold for {channel}: {t}")
    return p >= t


names, w, b = ["balance_util", "opens_90d", "tenure_m"], [-1.2, 0.08, 0.01], -2.0
defaults = {"balance_util": 0.3, "opens_90d": 0.0, "tenure_m": 24.0}
train_row = {"balance_util": 0.5, "opens_90d": 4, "tenure_m": 36}
offline = score(build_vector(train_row, names, defaults)[0], w, b)
request = {"tenure_m": 36, "opens_90d": 4, "balance_util": 0.5}  # client order
assert abs(score(build_vector(request, names, defaults)[0], w, b) - offline) < 1e-12
vec, missing = build_vector({"tenure_m": 36, "opens_90d": 4}, names, defaults)
assert vec == [0.3, 4.0, 36.0] and missing == ["balance_util"]
th = {"default": 0.5, "sms": 0.7}
assert should_send(0.6, "email", th) and not should_send(0.6, "sms", th)
print("ok")
```

Verify: the original failing test, a reordered request, a missing key, and the threshold fallback.

Pitfalls: letting the agent edit the test, and accepting a fix that passes only the given fixture.

Follow-ups: how would you detect skew in production? Log served vectors and compare them daily with offline values.

Why this is likely: 2026 Capital One engineering candidates describe an agentic CodeSignal round with Claude Code (fix a bug in an unfamiliar codebase, then build a feature), and prep guides say experienced engineers get it too; no MLE report shows it yet, so treat it as possible (low confidence for this role).

</details>

### ML and GenAI depth

<a id="q7"></a>

#### Q7. (Power Day ML depth, no AI, as at Capital One) A 90-day response model for credit card offers scores train AUC 0.93, random-split validation AUC 0.90, and out-of-time AUC 0.74 on the next quarter. Diagnose it and write the check that finds the cause.

Round: Power Day technical 1 (ML depth). AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Bias and variance, validation, leakage.

<details>
<summary>Show answer</summary>

Task: decide whether this is overfitting, leakage or drift, and write a per-feature check.

What they grade: reading the three numbers correctly, picking the right validation scheme for a time-ordered problem, and a concrete check.

Approach: train 0.93 versus random-split 0.90 is a small gap, so variance is not the main problem. The 16-point drop out of time is. A random split lets rows from the same weeks, campaigns and even the same customer appear on both sides, so it rewards features that encode "this campaign" or information gathered after the snapshot. Out of time, those features stop working. The two candidates are leakage (a feature built with data after the snapshot) and a real shift (a new offer, a pricing change or a new channel mix).

The check: compute each feature's single-feature AUC in time and out of time. A feature whose AUC collapses is either leaky or tied to a regime that ended. Then read how that feature is built. For leakage, rebuild it as of the snapshot; for a regime change, retrain with recent data or add the campaign attributes as explicit inputs. Going forward, validate with a rolling, out-of-time split and report the out-of-time number as the headline.

Alternatives: adversarial validation (train a model to tell the two periods apart and read its top features) finds shifts across combinations of features, but needs a model per run; permutation importance on the out-of-time set shows which features the model relies on but not why. Start with the single-feature AUC table, because it is the fastest first pass.

Code:

```python
import random


def auc(y, s):
    pairs = sorted(zip(s, y))
    ranks, i = [0.0] * len(pairs), 0
    while i < len(pairs):
        j = i
        while j + 1 < len(pairs) and pairs[j + 1][0] == pairs[i][0]:
            j += 1
        for k in range(i, j + 1):
            ranks[k] = (i + j) / 2 + 1  # average rank for ties
        i = j + 1
    pos = sum(y)
    neg = len(y) - pos
    rsum = sum(r for r, (_, lab) in zip(ranks, pairs) if lab == 1)
    return (rsum - pos * (pos + 1) / 2) / (pos * neg)


def drift_report(rows_in, rows_oot, features, drop=0.10):
    flagged = []
    for f in features:
        a_in = auc([r["y"] for r in rows_in], [r[f] for r in rows_in])
        a_oot = auc([r["y"] for r in rows_oot], [r[f] for r in rows_oot])
        if a_in - a_oot > drop:
            flagged.append((f, round(a_in, 2), round(a_oot, 2)))
    return flagged


random.seed(0)

def make(n, leaky):
    rows = []
    for _ in range(n):
        y = int(random.random() < 0.2)
        rows.append({"y": y, "tenure": y * 0.5 + random.gauss(0, 1),
                     "opens_after_send": (y * 2 if leaky else 0) + random.gauss(0, 1)})
    return rows


assert auc([0, 0, 1, 1], [0.1, 0.4, 0.35, 0.8]) == 0.75
report = drift_report(make(4000, True), make(4000, False), ["tenure", "opens_after_send"])
assert [f for f, _, _ in report] == ["opens_after_send"], report
print(report)
```

Verify: a hand-checked AUC, and a simulation where only the leaky feature is flagged.

Pitfalls: adding regularization because "it is overfitting", and tuning on the out-of-time set until it is no longer out of time.

Follow-ups: how would you set the retraining cadence? Plot out-of-time AUC by months since training.

Why this is likely: the posting lists bias and variance, feature selection and validation among the modeling issues the role must understand, and Capital One's AI/ML page says the second half of the technical round goes deep on ML skills (medium confidence).

</details>

<a id="q8"></a>

#### Q8. (Power Day ML depth, no AI, as at Capital One) Your team ranks next-best messages with gradient-boosted trees on 400 aggregate features. Someone proposes a transformer over each customer's last 200 events. Decide whether to switch, with latency and compute numbers.

Round: Power Day technical 1 (ML depth) or ML system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Sequential recommenders, serving cost.

<details>
<summary>Show answer</summary>

Task: compare the two on expected lift, serving cost and risk, and propose a migration path.

What they grade: knowing what a sequence model buys, back-of-envelope compute, and a staged plan instead of a rewrite.

Approach: trees on aggregates lose order and recency: "opened three travel emails this week" and "opened three travel emails last quarter" can look the same. A transformer over events learns order and timing directly and replaces much of the hand-built feature work. Capital One has written publicly about moving a recommender from tree-based models to a transformer, so the idea is credible here. The cost is compute, explainability and a harder serving path.

Numbers: a 4-layer encoder with width 128 over 200 events costs about 24 times 128 squared FLOPs per token per layer for projections and the feed-forward block, plus 4 times 200 times 128 for attention. That is about 99 million FLOPs per layer and about 0.4 GFLOP per customer. Scoring 50 million customers nightly is about 2 times 10 to the 16 FLOPs, a few minutes on one GPU at a realistic 100 TFLOPS. Online, 0.4 GFLOP on a CPU core at a realistic 40 GFLOPS is about 10 ms per customer. Encoding the history once per request and sharing it across the 30 candidates fits a 50 ms budget. It still takes a fifth of the budget, a CPU core per request at peak, and an online fetch of 200 raw events. A candidate-conditioned model runs the encoder once per candidate, about 30 times 10 ms = 300 ms, which does not fit. A 500-tree depth-6 model scores 30 candidates in well under a millisecond.

Alternatives: (1) keep trees and add sequence features such as recency-weighted counts: cheapest, smallest lift. (2) Full transformer online: best fit to the data, highest cost and latency. (3) Hybrid: compute the transformer's customer embedding nightly in batch, then feed it as features to the tree ranker online. I would pick the hybrid. It captures most of the sequence signal, keeps online latency flat, and keeps reason codes for model risk review.

Risks: stale embeddings for customers active today (refresh on major events), and governance: the embedding needs documentation, monitoring and a fallback to the old features.

Plan: offline out-of-time comparison first, then a holdout test against the current ranker on conversion and unsubscribe rate.

Why this is likely: Capital One's AI page describes a transformer-based recommender replacing tree-based models, and the posting asks for choices of model, data and features informed by infrastructure (medium confidence).

</details>

<a id="q9"></a>

#### Q9. (Power Day ML depth, no AI, as at Capital One) An LLM writes personalized message copy, and an LLM judge checks it for brand and compliance rules before human review. Design the offline eval that decides whether the judge can replace part of human review, and compute the numbers on this sample.

Round: Power Day technical 1 (ML depth) or ML system design. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: LLM evaluation, judge calibration.

Sample: 300 generated messages, labelled by compliance reviewers and by the judge.

| | Human pass | Human fail |
|---|---|---|
| Judge pass | 240 | 6 |
| Judge fail | 24 | 30 |

<details>
<summary>Show answer</summary>

Task: 300 generated messages were labelled pass or fail by compliance reviewers and by the judge. Both pass: 240. Judge pass, human fail: 6. Judge fail, human pass: 24. Both fail: 30. Is the judge good enough, and what is the rollout rule?

What they grade: choosing the metric that matches the cost of errors, honest uncertainty, and a gate that a compliance partner would sign.

Approach: raw agreement is 90%, but most of that comes from the many easy passes. Cohen's kappa corrects for chance agreement and comes out about 0.61, moderate. The expensive error is a judge pass on a message humans fail: 6 of the 36 human fails slipped through, so recall on violations is 83%. For copy going to customers of a bank, that is not good enough to remove human review.

The eval set: stratify messages by product, channel and segment, include adversarial cases (prompt-injected names, near-miss claims such as "no annual fee" when one applies), and keep a frozen test set that prompt authors never see. Score each rule separately, because a single pass or fail hides which rule the judge misses.

Rollout rule: let the judge auto-reject (it is conservative there), keep humans on everything it passes, and only reduce human review once the Wilson lower bound on violation recall is at least 95% on a fresh sample. With 36 failures that bound is about 68%, so the next step is collecting at least 200 more labelled failures.

Alternatives: exact-match rules catch known phrases with no false passes on them but miss paraphrases; a fine-tuned classifier is cheaper per call than an LLM judge but needs thousands of labels. While labels are scarce, use the LLM judge with a recall gate.

Code:

```python
from math import sqrt


def kappa(both_pass, j_pass_h_fail, j_fail_h_pass, both_fail):
    n = both_pass + j_pass_h_fail + j_fail_h_pass + both_fail
    po = (both_pass + both_fail) / n
    j_pass = (both_pass + j_pass_h_fail) / n
    h_pass = (both_pass + j_fail_h_pass) / n
    pe = j_pass * h_pass + (1 - j_pass) * (1 - h_pass)
    return (po - pe) / (1 - pe)


def wilson_lower(k, n, z=1.96):
    p = k / n
    centre = p + z * z / (2 * n)
    margin = z * sqrt(p * (1 - p) / n + z * z / (4 * n * n))
    return (centre - margin) / (1 + z * z / n)


k = kappa(240, 6, 24, 30)
recall = 30 / 36
low = wilson_lower(30, 36)
print(f"kappa={k:.2f} violation recall={recall:.2f} wilson_lower={low:.2f}")
assert abs(k - 0.61) < 0.01 and abs(low - 0.68) < 0.01
assert kappa(150, 150, 0, 0) == 0  # always-pass judge on balanced data
assert wilson_lower(10, 12) < wilson_lower(30, 36)  # same recall, fewer failures, wider interval
print("ok")
```

Verify: kappa is 0 for a judge that always says pass on balanced data, and the Wilson bound widens as n shrinks.

Pitfalls: reporting accuracy, and letting the judge see the human label in its prompt.

Follow-ups: how to detect judge drift after a model upgrade (rerun the frozen set on every judge change).

Why this is likely: the posting is for a GenAI role on a marketing and messaging team and asks for Responsible and Explainable AI, and Capital One's AI/ML page says platform roles go deep on guardrails and LLM safety (medium confidence).

</details>

<a id="q10"></a>

#### Q10. (Power Day ML depth, no AI, as at Capital One) The daily monitoring log below shows sends jumping and conversion dropping on day 9 with no model release. Find the root cause from the log and write the drift check that should have paged someone.

Round: Power Day technical 1 (ML depth). AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Production monitoring, drift.

```
day  scored  mean_score  sent   conv_rate  pred/actual  psi_days_since_open  null_rate_days_since_open  psi_income_band
1    48.1M   0.0212      1.92M  0.0208     1.02         0.01                 0.200                      0.00
2    48.0M   0.0211      1.91M  0.0207     1.02         0.01                 0.201                      0.01
3    48.2M   0.0213      1.93M  0.0209     1.02         0.00                 0.199                      0.00
4    48.1M   0.0212      1.92M  0.0206     1.03         0.01                 0.200                      0.01
5    48.3M   0.0210      1.90M  0.0205     1.02         0.02                 0.202                      0.00
6    48.2M   0.0212      1.92M  0.0208     1.02         0.01                 0.200                      0.01
7    48.1M   0.0211      1.91M  0.0207     1.02         0.01                 0.199                      0.00
8    48.3M   0.0210      1.91M  0.0207     1.01         0.01                 0.199                      0.01
9    48.2M   0.0297      2.71M  0.0151     1.61         1.19                 0.001                      0.01
10   48.1M   0.0298      2.72M  0.0150     1.62         1.19                 0.001                      0.00
events: day 9 02:00 feature-pipeline release v2.3 ("unify null handling"); model artifact unchanged
```

<details>
<summary>Show answer</summary>

Task: name the cause, the evidence, the immediate fix and the alert.

What they grade: reading the log in the right order, separating model problems from data problems, and a check with a threshold.

Approach: volume scored is flat, so it is not a population change. The model artifact did not change. On day 9 the mean score jumps 40%, more customers clear the fixed threshold (sends up 42%), and conversion among those sent drops, so predicted over actual goes from 1.02 to 1.61: the model is now overconfident. One feature moves: `days_since_last_open` has a PSI of 1.19 (above 0.25 is usually treated as a major shift), and its null rate falls from 20% to 0.1%. The pipeline release "unify null handling" lands at 02:00 on day 9. The likely cause: nulls (customers who never opened an email) are now filled with 0, which the model reads as "opened today", one of the strongest positive signals. The other feature, `psi_income_band`, stays flat, which rules out a broad data outage.

Fix now: roll back the pipeline release or restore the training-time missing value, then rescore. Fix for good: the model artifact records the expected null rate and fill value for each feature, and serving checks them.

Alternatives: alerting on conversion rate catches this too, but only after conversions mature, days later; a KS test is a fair choice for continuous features but needs care with large samples, where tiny shifts look significant. PSI plus null-rate checks per feature, run before scoring, catch it in minutes.

Code:

```python
from math import log


def psi(expected, actual, eps=1e-4):
    total = 0.0
    for e, a in zip(expected, actual):
        e, a = max(e, eps), max(a, eps)
        total += (a - e) * log(a / e)
    return total


# bins: 0-7 days, 8-30, 31-90, 91+, missing
baseline = [0.20, 0.25, 0.20, 0.15, 0.20]
day9 = [0.399, 0.25, 0.20, 0.15, 0.001]  # missing values now land in the 0-7 bin
stable = [0.21, 0.24, 0.20, 0.15, 0.20]
assert abs(psi(baseline, day9) - 1.19) < 0.01
assert psi(baseline, stable) < 0.01
print(round(psi(baseline, day9), 2), round(psi(baseline, stable), 4))
```

Verify: PSI near 0 for a stable day and 1.19 for the day 9 bins.

Pitfalls: retraining on the broken data, which bakes the bug in, and alerting only on model metrics.

Follow-ups: who owns the contract between the feature pipeline and the model?

Why this is likely: the posting says the role will "retrain, maintain, and monitor models in production" and ensure models are well governed, and MLOps monitoring is a common theme in Capital One MLE prep guides (medium confidence).

</details>

<a id="q11"></a>

#### Q11. (Power Day ML depth, no AI, as at Capital One) A generated email told customers they earn 3x miles for 15 months, but the live offer is 2x miles for 12 months. Use this trace to find the root cause, then propose a fix and a check that blocks this class of error.

Round: Power Day technical 1 (ML depth) or ML system design. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Retrieval grounding, GenAI guardrails.

```
{"step": "request", "offer_id": "O-17", "segment": "travel", "channel": "email"}
{"step": "retrieve", "cache": "hit", "key": "offer:O-17", "doc_version": 2, "age_hours": 70}
{"step": "offer_store", "offer_id": "O-17", "current_version": 3, "published_hours_ago": 30}
{"step": "facts_v2", "earn": "3x miles on travel", "term": "15 months"}
{"step": "facts_v3", "earn": "2x miles on travel", "term": "12 months"}
{"step": "generate", "prompt_tokens": 812, "output": "Earn 3x miles on travel for 15 months..."}
{"step": "guardrail", "check": "banned_phrases", "result": "pass"}
{"step": "guardrail", "check": "toxicity", "result": "pass"}
{"step": "guardrail", "check": "length", "result": "pass"}
{"step": "send", "status": "queued", "batch_size": 182000}
```

<details>
<summary>Show answer</summary>

Task: explain what failed, in order, and what to change.

What they grade: reading the trace instead of blaming the model, separating retrieval, generation and guardrail failures, and a deterministic check.

Approach: the model restated the facts it was given correctly. The retrieval cache returned version 2 of the offer, 70 hours old, while version 3 was published 30 hours ago. The cache key is the offer id only, so a new version never invalidates it. Then every guardrail passed, because none of them compares claims with the current offer. The failure is a missing contract, not a hallucination, and 182,000 customers were queued.

Fix: key the cache by (offer id, version), invalidate on the offer store's publish event, and have the generation step fetch the current version number on each batch. Add a claim check: every number, percentage and multiplier in the output must appear in the current offer facts. Block the batch and alert when it fails. Hold the queued batch now and regenerate it.

Alternatives: removing the cache fixes freshness but adds offer-store load and latency on every message; asking the LLM to double-check its own claims is cheaper to build but not deterministic. I would use versioned cache keys plus a deterministic claim check, because offer terms are exactly the kind of fact a regex can check.

Code:

```python
import re

NUM = re.compile(r"\d+(?:\.\d+)?(?:%|x)?")


def unsupported_claims(text, facts):
    allowed = set(NUM.findall(" ".join(facts.values())))
    return [n for n in NUM.findall(text) if n not in allowed]


v3 = {"earn": "2x miles on travel", "term": "12 months"}
bad = "Earn 3x miles on travel for 15 months."
good = "Earn 2x miles on travel for 12 months."
assert unsupported_claims(bad, v3) == ["3x", "15"]
assert unsupported_claims(good, v3) == []
assert unsupported_claims("Earn 2x miles for 12 months, 0% intro APR.", v3) == ["0%"]
print("ok")
```

Verify: replay the trace with the stale cache and confirm the batch is blocked; add 50 seeded offer changes to the eval set and require zero unsupported claims.

Pitfalls: fixing only the cache and leaving no check, and a claim check that ignores spelled-out numbers such as "twelve months".

Follow-ups: how would you handle claims that are not numbers, such as "no annual fee"?

Why this is likely: Capital One's AI page describes a retrieval pipeline built to reduce hallucinations, and the posting asks for GenAI work under Responsible AI rules on a marketing and messaging team (medium confidence).

</details>

### ML system design

<a id="q12"></a>

#### Q12. (Power Day ML system design, no AI, as at Capital One) Design a platform that picks the next best message for each customer across email, push, in-app and paid ads, writes the copy with an LLM inside brand and compliance rules, and learns from the results.

Round: Power Day ML system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Personalization, ranking, LLM generation, guardrails.

<details>
<summary>Show answer</summary>

Task: an end-to-end design with data, models, serving, guardrails and feedback, sized with numbers.

What they grade: scoping questions first, a clear split between batch and real time, guardrails as part of the design, and numbers.

Approach: first ask which channels need an in-session decision, whether the goal is incremental conversions or engagement, and who approves copy. Then five stages. (1) Eligibility: hard rules for opt-outs, collections holds, quiet hours and frequency caps (as in Q5 and Q2), run before any model. (2) Candidate generation: active offers matching each customer's products and segment, about 30 per customer. (3) Ranking: a model predicting incremental conversion per (customer, message, channel), with tree models on features plus a nightly sequence embedding (Q8). (4) Copy: choose from a pre-approved variant library, with the LLM used offline to write variants and online only for low-risk slot filling (Q13). (5) Feedback: sends, opens, conversions and unsubscribes flow back to the feature store and to training labels, with a random holdout to measure lift.

Numbers (assumptions): 50 million marketable customers, about 1 billion events a day (11,600 a second on average), so Kinesis or Kafka feeds Spark. Nightly batch: 50 million times 30 candidates is 1.5 billion scores, minutes on a modest Spark cluster (Q15). Online features: 50 million customers times 400 features times 4 bytes is 80 GB, which fits an online store such as DynamoDB or Redis. In-app decisions: about 3,000 requests a second at peak with a 100 ms budget, so the ranker gets about 40 ms.

Alternatives: one model per channel is simpler but cannot trade channels off for a customer; a single multi-task ranker with channel as a feature shares data across channels, so I would use it. A contextual bandit adapts faster than a supervised ranker but needs logged propensities and is harder for model risk to validate; I would start supervised with 10% exploration traffic and logged propensities, then move to a bandit.

Risks: feedback loops (keep exploration), compliance (every claim traces to an approved variant), and paid-channel cost (rank by expected value minus cost).

Why this is likely: Capital One's AI/ML page describes an end-to-end ML system design round built on modern LLM concepts, a 2026 MLE report lists an ML design round, and the team builds personalized omnichannel messaging on owned and paid channels (high confidence on the round, medium on the prompt).

</details>

<a id="q13"></a>

#### Q13. (Power Day ML system design, no AI, as at Capital One) Should the LLM write each customer's message in real time, or should you generate and review a library of variants offline and personalize by choosing among them? Decide with cost, latency and review numbers.

Round: Power Day ML system design. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: GenAI serving, compliance review, cost.

<details>
<summary>Show answer</summary>

Task: compare the options for about 2 million sends a day and recommend one.

What they grade: seeing that compliance review, not tokens, is the binding constraint, and doing the arithmetic.

Approach: in a bank, marketing copy about credit products needs review against approved claims. Real-time generation makes every message unique, so nothing a human approved is what the customer sees. That is the decisive issue; cost and latency come second.

Numbers (assumptions): real time, 2 million messages a day at 800 input and 120 output tokens is 1.6 billion input and 240 million output tokens a day. At an assumed 0.50 dollars per million input tokens and 2 dollars per million output tokens, that is about 1,280 dollars a day, affordable. Latency adds 0.5 to 2 seconds per message, fine for email, too slow for in-app. Review: 2 million unique messages a day cannot be reviewed by people; even a 1% sample is 20,000 a day. Variant library: 50 offers times 8 segments times 3 channels times 5 tones is 6,000 variants. At 1 minute each that is 100 reviewer hours per full refresh, done monthly, and an LLM judge (Q9) pre-screens so reviewers see fewer rejects.

Alternatives: (1) real time: most personal, unreviewable, and a hallucination reaches customers directly. (2) Offline library plus selection by the ranker: reviewable, zero added latency, and lift comes from choosing the right variant. (3) Hybrid: library text with deterministic slots (first name, preferred merchant category) filled from validated data, never from the LLM. I would ship (3). It keeps every claim reviewed, and the ranker can learn which variant works for which customer, which is where most of the lift is.

Risks: a library goes stale (track variant fatigue and refresh the worst-performing), and slots can still break rules (validate slot values against allowlists).

Verify: compare the hybrid against today's templates in a holdout test on conversion and unsubscribe rate before scaling.

Why this is likely: the posting pairs GenAI with Responsible and Explainable AI for a marketing and messaging team, and Capital One's AI/ML page says design rounds use modern LLM concepts and go deep on guardrails for platform roles (medium confidence).

</details>

<a id="q14"></a>

#### Q14. (Power Day ML system design, no AI, as at Capital One) Design how a retrained response model goes from training to 100% of traffic on Kubernetes, with one-box testing, blue/green deployment and gradual dial-up, and say what triggers an automatic rollback.

Round: Power Day ML system design or technical 1. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: MLOps, deployment, monitoring.

<details>
<summary>Show answer</summary>

Task: the release path, the gates between stages and the rollback triggers, with the traffic needed to trust each gate.

What they grade: fast technical gates before slow business gates, statistics on the dial-up, and model risk governance built into the pipeline.

Approach: the pipeline is CI/CD for models. Training writes an artifact to a model registry with its data snapshot, feature list, expected null rates and offline metrics. An automated validation step compares it with the champion on an out-of-time set and blocks if AUC drops or calibration moves. Model risk review signs off on material changes. Then deployment: (1) shadow, where the new model scores live traffic without acting and we compare score distributions and decision agreement for a day; (2) one-box, a single pod taking about 1% of real traffic; (3) blue/green with dial-up 10%, 50%, 100%, where the old deployment stays warm for instant rollback.

Numbers: fast rollback triggers need no conversions: error rate above 0.1%, p99 latency above 50 ms, a score PSI above 0.25 against shadow, or send volume moving by more than 10%. Conversion is slow. To detect a 10% relative drop from 2.0% to 1.8% (one-sided, 5% level, 80% power) needs about 57,600 sends per arm. At 2 million sends a day, 10% traffic is 200,000 sends a day, so volume arrives in one day, but conversions take about 7 days to mature. Hold at 50% for that week, then go to 100%.

Alternatives: a straight cutover is simplest but makes every bug full scale; a canary with automated analysis (for example Argo Rollouts) does the dial-up for you but needs well-chosen metrics. I would use Argo Rollouts or an equivalent with the fast technical metrics, plus a manual gate for the conversion check.

Risks: the old and new models both send to the same customers during dial-up, so randomize by customer, not by request; and retraining on data the current model chose creates feedback, so keep a small exploration slice.

Verify: rehearse a rollback in staging every release.

Why this is likely: the posting's preferred qualifications name one-box testing, blue/green deployment, gradual dial-up, monitoring and alarms, and a Capital One MLE question bank includes deployment, monitoring and low-latency inference (high confidence on topic).

</details>

<a id="q15"></a>

#### Q15. (Power Day ML system design, no AI, as at Capital One) Size the compute for scoring: nightly batch for scheduled campaigns plus real-time scoring for in-app moments. How many pods, how long does the batch take, and what does it cost per month?

Round: Power Day ML system design. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Capacity planning, AWS, Kubernetes.

<details>
<summary>Show answer</summary>

Task: state assumptions, compute pods and batch time, and choose what runs in batch and what runs online.

What they grade: explicit assumptions, headroom for spikes and zone failure, and the batch versus real-time decision.

Approach: assumptions to confirm: 3,000 in-app requests a second at peak, each ranking 30 candidates; a load test shows one 4-vCPU pod sustains 400 requests a second at a p99 of 40 ms; target 60% utilization so autoscaling has time to react; 3 availability zones, surviving the loss of one. Batch: 50 million customers times 30 candidates, and a tree model scores about 20,000 rows a second per vCPU.

Pods: 3,000 divided by 400 times 0.6 is 12.5, so 13 pods. To survive a zone loss, the two remaining zones must hold 13, so 7 per zone and 21 in total. At an assumed 0.04 dollars per vCPU-hour that is 84 vCPUs and about 2,450 dollars a month at full size; a horizontal pod autoscaler with a floor near 9 pods cuts that roughly in half off-peak. Batch: 1.5 billion scores is 75,000 vCPU-seconds, about 21 vCPU-hours, or about 6 minutes on 200 vCPUs before Spark overhead, so the cost is small.

Alternatives: scoring everything in real time at send time removes staleness but moves 1.5 billion scores into the latency path and needs several times the pods; serverless functions remove capacity planning but cold starts hurt p99 at this volume; all-batch is cheapest but cannot react within a session. I would score scheduled channels in batch and keep a small real-time tier for in-app.

Code:

```python
from math import ceil


def realtime_pods(peak_rps, pod_rps, target_util=0.6, zones=3):
    base = ceil(peak_rps / (pod_rps * target_util))
    per_zone = ceil(base / (zones - 1))  # survive the loss of one zone
    return base, per_zone * zones


def batch(customers, candidates, rows_per_vcpu_s, vcpus):
    vcpu_s = customers * candidates / rows_per_vcpu_s
    return vcpu_s / 3600, vcpu_s / vcpus / 60  # vCPU-hours, wall-clock minutes


base, pods = realtime_pods(3000, 400)
monthly = pods * 4 * 0.04 * 730
vcpu_h, minutes = batch(50e6, 30, 20_000, 200)
assert (base, pods) == (13, 21)
assert round(monthly) == 2453 and round(vcpu_h, 1) == 20.8 and minutes == 6.25
print(base, pods, round(monthly), round(vcpu_h, 1), minutes)
```

Verify: rerun with the measured pod throughput from a load test, and check the p99 at 60% utilization.

Pitfalls: sizing to average load, and forgetting the feature store's read capacity (3,000 requests a second times 30 candidates may mean 90,000 reads a second unless features are fetched per customer).

Why this is likely: the posting asks for cloud architectures that deliver ML models at scale on AWS and Kubernetes, and Capital One's design round is reported to probe end-to-end infrastructure (medium confidence).

</details>

### Experiments and governance

<a id="q16"></a>

#### Q16. (Power Day case study or ML depth, no AI, as at Capital One) Design the test that decides whether the new message model beats the current one. Conversion is 2.0% today and the business cares about a lift to 2.2%. Give the unit, the metrics, the sample size and the run length.

Round: Power Day case study or technical 1. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Experiment design, uplift.

<details>
<summary>Show answer</summary>

Task: a test plan with a primary metric, guardrails, sample size and decision rule.

What they grade: randomizing at the right unit, measuring incremental effect, and power math.

Approach: randomize by customer, not by message, so one customer never sees both models' choices. Primary metric: conversions within 14 days per assigned customer (intent to treat), which counts customers the new model chose not to message. Guardrails: unsubscribe and complaint rates, with a non-inferiority margin agreed up front, plus send volume. Keep a small long-running universal holdout that gets no marketing messages, so the team can measure the total incremental effect of messaging, not just model against model.

Sample size: 2.0% against 2.2% at a two-sided 5% level with 80% power needs about 80,700 customers per arm. With about 2 million eligible customers a day, that is reached on the first day, but run at least two full weeks: conversions need 14 days to mature, and weekly cycles in spending matter.

Alternatives: a multi-armed bandit shifts traffic to the winner faster, but makes a clean effect estimate harder, which model risk review will ask for; for paid media, where individuals often cannot be randomized, a geo experiment (matched regions) is the fallback. For owned channels, use the customer-level A/B test.

Code:

```python
from math import ceil, sqrt
from statistics import NormalDist


def n_per_arm(p1, p2, alpha=0.05, power=0.8, two_sided=True):
    z = NormalDist().inv_cdf
    za = z(1 - alpha / 2) if two_sided else z(1 - alpha)
    zb = z(power)
    pbar = (p1 + p2) / 2
    num = (za * sqrt(2 * pbar * (1 - pbar)) + zb * sqrt(p1 * (1 - p1) + p2 * (1 - p2))) ** 2
    return ceil(num / (p1 - p2) ** 2)


n = n_per_arm(0.020, 0.022)
assert 80_000 < n < 81_000, n
assert n_per_arm(0.020, 0.018, two_sided=False) < n  # rollback check in Q14
print(n, n_per_arm(0.020, 0.018, two_sided=False))
```

Verify: the formula against a simulation, or an online calculator, before you present it.

Pitfalls: measuring response (opened, clicked) instead of incremental conversion, peeking daily without a sequential method, and counting a customer in both arms when they switch devices.

Follow-ups: how would you test on paid ad channels where the platform controls delivery?

Why this is likely: a Capital One question bank lists analytics and experimentation as its largest category, the case round ties ML to business metrics, and the posting asks for models that solve real business problems (medium confidence).

</details>

<a id="q17"></a>

#### Q17. (Power Day ML depth or case study, no AI, as at Capital One) Model risk review asks two things before launch: per-customer reasons for each targeting decision, and evidence that the model does not select one group of customers at a much lower rate. Build both checks.

Round: Power Day technical 1 (ML depth) or case study. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Explainability, fairness, model governance.

Given: a logistic targeting model with three features.

| Feature | Weight | Population mean |
|---|---|---|
| travel_spend | 0.9 | 1.0 |
| opens_90d | 0.3 | 2.0 |
| balance_util | -1.1 | 0.4 |

Selected for the campaign: group A 1,200 of 10,000; group B 450 of 5,000. Sample customer: travel_spend 3.0, opens_90d 5, balance_util 0.2.

<details>
<summary>Show answer</summary>

Task: reason codes per decision and a selection-rate comparison across groups, with what you would do if the check fails.

What they grade: picking an explanation method that fits the model, honest limits of fairness metrics, and knowing how a bank governs models.

Approach: banks follow interagency supervisory guidance on model risk management. The current version is SR 26-2, issued in April 2026 by the Federal Reserve with the OCC (as Bulletin 2026-13) and the FDIC, and it replaced the long-standing SR 11-7. It expects documentation, independent validation and ongoing monitoring, scaled to how much risk each model carries. For a logistic model, each feature's contribution is its weight times the customer's distance from the population average. This is exact on the log-odds scale. The contributions add up to the customer's logit minus the logit at the average; the sigmoid makes them non-additive on the probability scale. The top positive contributions become reason codes. For tree models, TreeSHAP gives consistent attributions at a cost that grows with tree depth.

For selection rates, compare each group's rate with the highest group's. A ratio below 0.8 (a rule of thumb borrowed from US employment guidance) means investigate, not "illegal". Marketing of credit products can raise fair lending risk, for example if targeting skips neighborhoods. Card issuers often do not collect demographic data, so teams estimate it with proxy methods such as BISG and treat the result as approximate. If the check fails, find the features driving the gap (often geography-like features), test removing or constraining them, and measure the lift lost.

Alternatives: LIME explains any model but its explanations change between runs; global feature importance is stable but says nothing about one customer. Use exact linear contributions for the logistic model and TreeSHAP for tree models.

Code:

```python
def reason_codes(x, weights, means, top=2):
    contrib = {f: weights[f] * (x[f] - means[f]) for f in weights}
    return [f for f, v in sorted(contrib.items(), key=lambda kv: -kv[1]) if v > 0][:top]


def impact_ratios(selected, total):
    rates = {g: selected[g] / total[g] for g in total}
    best = max(rates.values())
    return {g: round(r / best, 2) for g, r in rates.items()}


w = {"travel_spend": 0.9, "opens_90d": 0.3, "balance_util": -1.1}
mu = {"travel_spend": 1.0, "opens_90d": 2.0, "balance_util": 0.4}
assert reason_codes({"travel_spend": 3.0, "opens_90d": 5, "balance_util": 0.2}, w, mu) == ["travel_spend", "opens_90d"]
ratios = impact_ratios({"A": 1200, "B": 450}, {"A": 10_000, "B": 5_000})
assert ratios == {"A": 1.0, "B": 0.75}
x = {"travel_spend": 3.0, "opens_90d": 5, "balance_util": 0.2}
logit = lambda v: sum(w[f] * v[f] for f in w) - 0.5
assert abs(sum(w[f] * (x[f] - mu[f]) for f in w) - (logit(x) - logit(mu))) < 1e-9
print(ratios, [g for g, r in ratios.items() if r < 0.8])
```

Verify: the asserts check the hand-computed reason codes and impact ratios, and that the contributions add up exactly to the logit difference.

Pitfalls: dropping the protected attribute and calling the model fair, since proxies remain; and reason codes that name a feature a customer cannot understand.

Follow-ups: how do you monitor these ratios after launch?

Why this is likely: the posting asks that models be well governed from a risk perspective and follow Responsible and Explainable AI practices (high confidence on topic).

</details>

### Behavioral

<a id="q18"></a>

#### Q18. (Power Day behavioral, no AI, as at Capital One) Tell me about a time you improved an ML pipeline you owned. Where did you use AI tools, how did you check their output, and what would you do differently?

Round: Power Day behavioral. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Ownership, AI use, judgment.

<details>
<summary>Show answer</summary>

Task: one STAR story, about 3 minutes, with your own decisions in it and a clear account of AI use.

What they grade: ownership, the reasoning behind each choice (interviewers ask a second layer of "why"), measurable results, and responsible AI use.

Approach: pick a story with a number at each step. A sample shape: Situation, a nightly feature pipeline for a response model took 5 hours and failed about twice a month, delaying campaigns. Task, I owned getting it under 2 hours and stopping silent failures. Action, I profiled the Spark job and found a skewed join on a few very active customers; I salted the join key and added row-count and null-rate checks between stages. I used an AI coding assistant to draft the salting change and the data-quality tests. I reviewed every diff, ran the tests on a day of production data, and caught one wrong suggestion: the assistant wrote the null-rate check against the whole table instead of per partition, which would have hidden a failure in a single day's data. I rewrote it and added a test that failed on the old version. Result, runtime fell to 1 hour 40 minutes, failures stopped for the next quarter, and the checks caught one upstream schema change before it reached the model.

Then say what you would change: I would have added the data-quality checks before optimizing, because they found more problems than the speedup solved.

Verify: before the interview, check that every number is true and that you can explain each decision two levels deep.

Pitfalls: saying "we" for every action, a story with no AI use when asked, or the opposite: claiming the AI did the work without saying how you checked it. Keep confidential details out.

Follow-ups: what do you never delegate to an AI tool? How does your team review AI-written code?

Why this is likely: Capital One's AI/ML page says the behavioral round probes ownership with a second layer of questions, and a 2026 Power Day report says the final behavioral question was about the use of AI (medium confidence on the AI part).

</details>

## Prep checklist

- [ ] Do 4 timed CodeSignal-style sets of four problems in 70 minutes (two easy, one medium, one medium-hard) with no AI and your camera on.
- [ ] Practice stateful simulations and "design a class" problems in levels: recent-call counters, allocators, bank systems, tries.
- [ ] Write SQL that builds point-in-time features and catch join and leakage bugs on paper.
- [ ] Prepare a 45-minute ML design for a personalized messaging platform with numbers for events, scores, features and latency.
- [ ] Be able to argue real-time LLM generation against a reviewed variant library, with token cost and review hours.
- [ ] Practice reading unfamiliar Python and dry-running it out loud, then extending it for a new rule with tests.
- [ ] Be able to derive and explain AUC, PSI, Cohen's kappa, a Wilson interval and an A/B sample size on a whiteboard, and say when each one misleads.
- [ ] Prepare a release plan with shadow, one-box, blue/green and dial-up, and the rollback metrics for each stage.
- [ ] Do one timed drill with an AI coding agent on a small repo: fix a bug, add a feature, and review each diff before accepting it.
- [ ] Prepare two STAR stories, one about improving an ML pipeline and one about catching a wrong result, each with how you used and checked AI.
- [ ] Ask the recruiter which rounds you will get, whether the coding round allows AI tools, and which team runs the design round.

## Sources

- https://www.capitalonecareers.com/insider-tips-for-ai-and-machine-learning-interviews-tech-101-cdev
- https://www.capitalone.com/tech/ai/
- https://www.glassdoor.com/Interview/Capital-One-Machine-Learning-Engineer-Interview-Questions-EI_IE3736.0,11_KO12,37.htm
- https://www.reddit.com/r/learnmachinelearning/comments/1xcucvn/capital_one_mle_power_day_interview_experience (snippet only)
- https://www.teamblind.com/post/capital-one-senior-lead-software-engineer-interview-power-day-experience-ojk3afdm
- https://prachub.com/interview-experiences/capital-one-new-grad-software-engineer-interview-experience-power-day-with-behavioral-case-study-and-agentic-coding
- https://prachub.com/resources/capital-one-software-engineer-interview-guide-2026-codesignal-power-day-case-interviews-and-timeline (low confidence, prep vendor)
- https://prachub.com/companies/capital-one/positions/machine-learning-engineer (low confidence, crowd question bank)
- https://www.calibrd.com/interview-prep/capital-one-interview (low confidence, prep vendor)
- https://www.1point3acres.com/interview/thread/1190403 (summary only)
- https://www.interviewquery.com/interview-guides/capital-one-machine-learning-engineer (low confidence, prep vendor)
- https://www.datainterview.com/blog/capital-one-machine-learning-engineer-interview (low confidence, prep vendor)
- https://www.federalreserve.gov/supervisionreg/srletters/SR2602.htm
- https://github.com/liquidslr/leetcode-company-wise-problems
