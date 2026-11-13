# Databricks: AI Engineer, Forward Deployed Engineering (AI FDE)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Databricks publishes one company-wide frame: a recruiter call, a pre-onsite screen, an onsite of four to six interviews, and a presentation for some roles, all virtual on Google Meet. The AI FDE team lead has described the technical core as three rounds: GenAI depth, EDA and classical ML, and productionization. Candidate reports fill in the rest. The loop varies by region and level, so confirm it with your recruiter.

1. Recruiter screen. 30 minutes. Background, why Databricks, why forward deployed work, customer-facing experience. AI: not applicable.
2. Hiring manager screen. 30 to 45 minutes. Project ownership from first customer call to production; some AI FDE candidates got LLM fundamentals here. AI: no published rule, treat as not allowed.
3. Technical coding screen. About 60 minutes, pairing with an engineer. Build to a written spec, extend in stages, write tests, debug code that partly works. For AI FDE it leans toward applied data work and model evaluation, not algorithm puzzles. AI: allowed in some reported loops (an LLM or Databricks Assistant, and you must vet its output); ask the recruiter.
4. GenAI technical deep dive. 45 to 60 minutes. Transformers, fine-tuning, RAG, vector search, evaluation, and "why this model, why not X" on systems you built. AI: not allowed.
5. EDA and classical ML. 45 to 60 minutes, likely hands-on in a notebook. Statistical intuition and modeling fundamentals. AI: no published rule.
6. Productionization and LLMOps design. 45 to 60 minutes on a whiteboard or shared doc. Evals, CI/CD, monitoring, drift, governance, with progressive curveballs (batch to streaming, a Spark join that runs out of memory). AI: no published rule; expect no AI and confirm with the recruiter.
7. Decomposition or customer scenario. 45 to 60 minutes. A vague business goal; you find stakeholders and KPIs before any architecture. AI: not allowed.
8. Build, demo and panel (some loops). Build a small solution, demo it, take questions. AI: ask the recruiter.
9. Behavioral and culture (one or two). Ownership, customer obsession, pushing back on a customer. AI: not allowed.
10. Executive round for senior levels, references, then a decision. AI: not applicable.

No current report mentions a classic algorithm round for this role, so there are no LeetCode-style questions here. The mix below: four practical coding tasks (build, debug, review of AI-written code, trace debug), three EDA and classical ML tasks, three GenAI deep dive questions, four productionization and evaluation designs, one decomposition case and one behavioral question.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Build a document chunker and keyword retriever to a spec, then extend it in two stages | Build | Technical coding screen | Allowed | Medium |
| [Q2](#q2) | The RAG eval harness reports 31% accuracy on an echo model. Find and fix the bugs | Debug | Technical coding screen | Allowed | Medium |
| [Q3](#q3) | Review this AI-written hybrid search function before it ships to a customer | Review | Technical coding screen | Allowed | Medium |
| [Q4](#q4) | A feature pipeline's shuffle join kills executors. Read the plan and task stats, fix it | Trace debug | Technical coding screen | Allowed | Hard |
| [Q5](#q5) | Build a churn model on this customer table, and find why validation AUC is 0.99 | Concept plus code | EDA and classical ML | Unknown | Medium |
| [Q6](#q6) | The new RAG prompt lifts thumbs-up from 62% to 65%. Is the lift real? | Concept plus code | EDA and classical ML | Unknown | Medium |
| [Q7](#q7) | Forecast weekly demand for a product launching next month with no sales history | Concept plus code | Decomposition or EDA and classical ML | Unknown | Medium |
| [Q8](#q8) | A contract assistant answers from a superseded document. Read the retrieval trace, fix it | Trace debug | GenAI technical deep dive | Not allowed | Hard |
| [Q9](#q9) | Fine-tune a model or improve the context for a support assistant? Decide with numbers | Design | GenAI technical deep dive | Not allowed | Medium |
| [Q10](#q10) | Raise a text-to-SQL assistant from 58% to production accuracy, and prove it | Concept plus code | GenAI technical deep dive | Not allowed | Hard |
| [Q11](#q11) | Take a customer's GenAI proof of concept to production with eval gates and monitoring | Design | Productionization and LLMOps design | Not allowed | Hard |
| [Q12](#q12) | Your LLM judge agrees with humans 85% of the time. Is it good enough to gate releases? | Concept plus code | Productionization and LLMOps design | Not allowed | Hard |
| [Q13](#q13) | Offline evals pass, but production answer quality fell for three weeks. Read the monitor | Trace debug | Productionization and LLMOps design | Not allowed | Hard |
| [Q14](#q14) | Design a supervisor agent that routes between a text-to-SQL agent and a document agent | Design | Productionization and LLMOps design | Not allowed | Hard |
| [Q15](#q15) | A VP says "use AI to cut claims processing time". Scope the engagement | Design | Decomposition or customer scenario | Not allowed | Medium |
| [Q16](#q16) | A customer wants to ship an AI agent next week without evals. How do you push back? | Behavioral | Behavioral and culture | Not allowed | Medium |

### Practical coding (AI assistant allowed in some reported loops; ask the recruiter)

<a id="q1"></a>

#### Q1. Build a document chunker and keyword retriever to a spec, then extend it in two stages

Round: Technical coding screen. AI policy: Allowed. Type: Build. Difficulty: Medium. Topic: RAG ingestion and retrieval.

Spec handed to you in the shared editor:

```text
rag_mini/
  chunk.py        # chunk(text, max_words, overlap) -> list[str]   (you write this)
  retrieve.py     # BM25(chunks).search(query, k) -> list[int]      (you write this)
  test_chunk.py   # 5 tests
  test_retrieve.py# 4 tests

Stage 1: split on whitespace; every chunk has at most max_words words;
         consecutive chunks share exactly `overlap` words; the last word of
         the document is in the last chunk; empty text gives [].
         overlap >= max_words raises ValueError.
Stage 2: BM25 keyword search (k1=1.5, b=0.75), lowercase alphanumeric
         tokens, ties broken by lower chunk index, zero-score chunks dropped.
Stage 3: chunks never cross a markdown heading ("# ..."); each chunk
         carries its heading so a search hit can cite its section.
```

<details>
<summary>Show answer</summary>

Task: implement stage 1 and 2 with tests green, then stage 3 without breaking them.

What they grade: reading the spec before typing, edge cases, how you direct and check the assistant, clean extension.

Approach: I write the edge-case tests first (empty text, text shorter than one chunk, length exactly max_words, overlap equal to max_words) because they pin the spec. Then I delegate: "Implement chunk() in chunk.py to pass test_chunk.py. Use a step of max_words minus overlap. Do not change the tests." I read the diff for the loop bound. The assistant's first version used `range(0, len(words), step)`, which emits a trailing chunk made only of overlap words (for 50 words, max 50, overlap 10 it returned two chunks). My exact-length test caught it, and the fix stops at `len(words) - overlap`.

For BM25 I give the formula in the prompt rather than "implement BM25", since assistants mix up the idf variants. I check that idf stays positive for a term that appears in every chunk (the `log(1 + ...)` form does).

Stage 3 is a wrapper that splits on headings, chunks each section and attaches the heading. Two alternatives: carry the heading as metadata (clean citations, no token cost) or prepend it to each chunk's text (helps retrieval when the body never repeats the section topic, costs a few tokens per chunk). I prepend it for retrieval and also keep it as metadata, because short policy sections often lack their own keywords.

Code:

```python
import math, re
from collections import Counter

def chunk(text, max_words=50, overlap=10):
    if overlap >= max_words:
        raise ValueError("overlap must be smaller than max_words")
    words = text.split()
    if not words:
        return []
    step = max_words - overlap
    stop = max(len(words) - overlap, 1)
    return [" ".join(words[s:s + max_words]) for s in range(0, stop, step)]

def chunk_sections(markdown, max_words=50, overlap=10):
    out, heading, buf = [], "", []
    def flush():
        for c in chunk(" ".join(buf), max_words, overlap):
            out.append({"section": heading, "text": (heading + " " + c).strip()})
    for line in markdown.splitlines():
        if line.startswith("#"):
            flush(); buf = []; heading = line.lstrip("#").strip()
        else:
            buf.append(line)
    flush()
    return out

TOK = re.compile(r"[a-z0-9]+")
def tokens(s):
    return TOK.findall(s.lower())

class BM25:
    def __init__(self, chunks, k1=1.5, b=0.75):
        self.docs = [tokens(c) for c in chunks]
        self.tf = [Counter(d) for d in self.docs]
        self.n = len(self.docs)
        self.avg = sum(map(len, self.docs)) / max(self.n, 1)
        self.df = Counter(t for d in self.docs for t in set(d))
        self.k1, self.b = k1, b

    def idf(self, t):
        n = self.df.get(t, 0)
        return math.log(1 + (self.n - n + 0.5) / (n + 0.5))

    def search(self, query, k=3):
        scored = []
        for i, d in enumerate(self.docs):
            s = 0.0
            for t in tokens(query):
                f = self.tf[i].get(t, 0)
                if f:
                    norm = self.k1 * (1 - self.b + self.b * len(d) / self.avg)
                    s += self.idf(t) * f * (self.k1 + 1) / (f + norm)
            if s > 0:
                scored.append((-s, i))
        return [i for _, i in sorted(scored)[:k]]

words = " ".join(f"w{i}" for i in range(120))
cs = chunk(words, 50, 10)
assert len(cs) == 3 and cs[-1].split()[-1] == "w119"
assert cs[0].split()[-10:] == cs[1].split()[:10]
assert len(chunk(" ".join(f"w{i}" for i in range(50)), 50, 10)) == 1
assert chunk("", 50, 10) == []
try:
    chunk("a b", 5, 5); raise AssertionError("expected ValueError")
except ValueError:
    pass
docs = ["refund within 30 days of purchase", "shipping takes 5 days", "the refund policy covers refund requests"]
assert BM25(docs).search("refund", k=2) == [2, 0]
assert BM25(docs).search("warranty") == []
md = "# Refunds\nmoney back in 30 days\n# Shipping\nships in 5 days"
secs = chunk_sections(md)
assert [s["section"] for s in secs] == ["Refunds", "Shipping"]
assert BM25([s["text"] for s in secs]).search("shipping")[0] == 1
print("ok")
```

Verify: overlap equality between neighbors, last word present, exact-length input gives one chunk, ties and zero scores, headings never merge.

Pitfalls: trailing overlap-only chunk; negative idf; letting the assistant edit tests to pass.

Follow-ups: chunk by tokens instead of words; add vector search and fuse the two (see Q3).

Why this is likely: candidate reports describe the Databricks FDE technical screen as building to a spec in stages with tests and AI tools allowed, and the AI FDE job lists RAG work (medium confidence).

</details>

<a id="q2"></a>

#### Q2. The RAG eval harness reports 31% accuracy on an echo model. Find and fix the bugs

Round: Technical coding screen. AI policy: Allowed. Type: Debug. Difficulty: Medium. Topic: GenAI evaluation harness.

A customer's harness scores their RAG endpoint. A test model that returns the expected answer scores 31%, and one test crashes.

```text
eval_harness/
  load.py         # load_rows(path) -> [{"id", "question", "expected", "category"}]
  run.py          # run_all(rows, call) -> predictions
  score.py        # accuracy(rows, preds)
  report.py       # per-category accuracy table
  test_score.py   # 6 tests, 2 failing

# run.py
def run_all(rows, call, workers=8):
    with ThreadPoolExecutor(workers) as ex:
        futs = [ex.submit(call, r) for r in rows]
        return [f.result() for f in as_completed(futs)]

# score.py
def normalize(s):
    return s.strip().lower()

def accuracy(rows, preds):
    hits = sum(normalize(p) == normalize(r["expected"]) for r, p in zip(rows, preds))
    return hits / len(rows)

FAILED test_echo_model_scores_one: 0.31 != 1.0
FAILED test_timeout_counts_as_wrong: AttributeError: 'NoneType' object has no attribute 'strip'
```

<details>
<summary>Show answer</summary>

Task: explain both failures, fix them, and harden the harness so it cannot silently misalign again.

What they grade: reasoning from the symptom, a targeted prompt, checking the diff, a regression test.

Approach: An echo model must score 1.0, so the scores and the rows are misaligned. `as_completed` yields futures in finish order, and `zip(rows, preds)` assumes input order. With eight workers and uneven latency most pairs are wrong, hence 31%. The second failure comes from the endpoint wrapper, which returns None on timeout, and `normalize` then calls `.strip()` on it.

Prompt to the assistant: "In run.py, return predictions in input order, keyed by row id. In score.py, treat None as a wrong answer and count it as an error. Show only the diff." The first diff it gave sorted results by `id(future)`, which is a memory address and not input order. I rejected that and asked for `ex.map` or a dict keyed by row id. I prefer keying by id over `ex.map`: it survives retries and partial reruns, and the scorer can assert that every row has exactly one prediction.

Two more things I check before calling it fixed. A zero-row file divides by zero, so empty input raises a clear error. And the per-category report must use the same keyed join, or it will repeat the bug.

Code:

```python
import random, time
from concurrent.futures import ThreadPoolExecutor, as_completed

def run_all(rows, call, workers=8):
    with ThreadPoolExecutor(workers) as ex:
        futs = {ex.submit(call, r): r["id"] for r in rows}
        return {futs[f]: f.result() for f in as_completed(futs)}

def normalize(s):
    return " ".join(str(s).lower().split()).rstrip(".")

def score(rows, preds):
    if not rows:
        raise ValueError("no eval rows")
    missing = {r["id"] for r in rows} - preds.keys()
    assert not missing, f"no prediction for {sorted(missing)}"
    hits = errors = 0
    for r in rows:
        p = preds[r["id"]]
        if p is None:
            errors += 1
        elif normalize(p) == normalize(r["expected"]):
            hits += 1
    return {"accuracy": hits / len(rows), "errors": errors}

rows = [{"id": i, "question": f"q{i}", "expected": f"Answer {i}."} for i in range(40)]
def echo(r):
    time.sleep(random.random() / 200)
    return r["expected"]
assert score(rows, run_all(rows, echo))["accuracy"] == 1.0
def flaky(r):
    return None if r["id"] % 10 == 0 else r["expected"]
res = score(rows, run_all(rows, flaky))
assert res == {"accuracy": 0.9, "errors": 4}
print("ok")
```

Verify: echo model scores 1.0 with random latency; timeouts count as wrong and are reported; empty input fails loudly.

Pitfalls: fixing only the crash; trusting a 31% baseline without a sanity model; normalizing so hard that wrong answers match.

Follow-ups: replace exact match with an LLM judge (see Q12); add retries without double counting.

Why this is likely: candidate reports say the FDE coding screen includes debugging code that partly works, and the AI FDE job centers on evaluation (medium confidence).

</details>

<a id="q3"></a>

#### Q3. Review this AI-written hybrid search function before it ships to a customer

Round: Technical coding screen. AI policy: Allowed. Type: Review. Difficulty: Medium. Topic: Hybrid retrieval and reciprocal rank fusion.

A teammate asked an assistant for "hybrid vector plus keyword search with RRF, filtered to the user's region". Results look plausible in a demo. Review it.

```python
def hybrid_search(index, sql, query, user_region, k=10, rrf_k=60):
    # vector results, already reranked
    vec = index.similarity_search(query_text=query, num_results=k, rerank=True)
    # keyword results, region filtered
    kw = sql(f"SELECT id FROM docs WHERE region = '{user_region}' "
             f"AND text ILIKE '%{query}%' LIMIT {k}")
    scores = {}
    for rank, doc in enumerate(vec):
        scores[doc["id"]] = 1 / (rrf_k + rank)
    for rank, doc in enumerate(kw):
        scores[doc["id"]] = 1 / (rrf_k + rank)
    return sorted(scores, key=scores.get)[:k]
```

<details>
<summary>Show answer</summary>

Task: find the defects, rank them by harm, and write a tested fusion function.

What they grade: security first, catching a made-up API, the RRF math, tests that would have caught each bug.

Approach: in order of harm:

1. Data leak. The region filter applies only to keyword hits. Vector hits from other regions flow through, so an EU user can see US documents. The filter has to go into the vector query too, or results must be filtered after fusion against a trusted region.
2. SQL injection. `query` and `user_region` are pasted into SQL. A query containing a quote breaks it, and a crafted one reads other tables. Use bound parameters.
3. Made-up parameter and wrong return shape. The client has no `rerank` argument. The real option is `reranker=DatabricksReranker(columns_to_rerank=["text"])`, imported from `databricks.ai_search.reranker` per the current docs (SDK 0.57 or later). Setting `query_type="hybrid"` runs keyword plus vector search with RRF server side, which may replace this function entirely. I check the installed signature with `inspect.signature` and the docs before trusting any argument. The code also treats the response as a list of dicts. `similarity_search` returns a dict, and the rows are in `response["result"]["data_array"]`. Each row is a list of the requested columns followed by the score, so `enumerate(vec)` loops over the dict's keys and `doc["id"]` fails.
4. Fusion bugs. The second loop overwrites instead of adding, so a document found by both retrievers gets no boost, which defeats RRF. Rank starts at 0, while RRF uses 1 / (k + rank) with rank from 1 (minor, but it changes ties). The sort is ascending, so it returns the worst documents first.
5. `ILIKE '%whole query%'` matches only the exact phrase, so multi-word queries get almost no keyword hits. Use a real keyword index (BM25 from Q1).

Alternatives for fusion: RRF needs no score calibration; a weighted sum of normalized scores can be tuned but breaks when score scales drift. I keep RRF and tune k later on the eval set.

Code:

```python
def rrf(result_lists, k=60, top=10, allowed=None):
    scores = {}
    for results in result_lists:
        for rank, doc_id in enumerate(results, start=1):
            if allowed is not None and doc_id not in allowed:
                continue
            scores[doc_id] = scores.get(doc_id, 0.0) + 1.0 / (k + rank)
    return sorted(scores, key=lambda d: (-scores[d], d))[:top]

vec = ["a", "b", "c", "x_us"]
kw = ["c", "d", "a"]
fused = rrf([vec, kw], allowed={"a", "b", "c", "d"})
assert fused[:2] == ["a", "c"]
assert "x_us" not in fused
assert rrf([[], []]) == []
assert rrf([["z"], ["y"]]) == ["y", "z"]
print("ok")
```

Verify: a document in both lists outranks one in a single list; an out-of-region id never appears; a query with a quote character runs safely through the parameterized query.

Pitfalls: approving because the demo looked right; fixing fusion and missing the leak; trusting arguments the assistant invented.

Follow-ups: add a cross-encoder reranker on the top 50 and measure the latency cost; pick k for RRF on the eval set.

Why this is likely: reports say AI tools are allowed in some FDE coding rounds and candidates must vet AI output, and Databricks has shipped hybrid search with RRF in Vector Search (medium confidence).

</details>

<a id="q4"></a>

#### Q4. A feature pipeline's shuffle join kills executors. Read the plan and task stats, fix it

Round: Technical coding screen. AI policy: Allowed. Type: Trace debug. Difficulty: Hard. Topic: Spark joins and data skew.

The nightly job that builds features for the customer's churn model started failing after a new event source was added.

```text
== Physical Plan ==
SortMergeJoin [customer_id], [customer_id], LeftOuter
:- Exchange hashpartitioning(customer_id, 200)
:  +- FileScan delta events   size 1.8 TB, rows 9.1B
+- Exchange hashpartitioning(customer_id, 200)
   +- FileScan delta customers size 3.2 GB, rows 41M

Stage 14: 199/200 tasks done in 2 to 4 min; task 137 running 48 min
task 137 shuffle read 212 GB   (median task 9.0 GB)
ExecutorLostFailure (executor 31 exited): container killed, exceeding memory limits
spark.sql.adaptive.enabled = false
SELECT count(*) FROM events WHERE customer_id IS NULL   -> 1.07B
top non-null keys: cust_000417 (38M rows), cust_009113 (21M rows)
```

<details>
<summary>Show answer</summary>

Task: name the root cause from the evidence and propose fixes in order of effort.

What they grade: reading task-level skew before reaching for more memory, knowing the join options, numbers.

Approach: one task reads 212 GB while the median reads 9 GB, so this is key skew, not an undersized cluster. The new source writes about 1.07B rows with a null customer_id. Hash partitioning sends every null to one partition, and nulls never match in a join anyway, so all that data is shuffled for nothing. At about 200 bytes per row (1.8 TB over 9.1B rows), 1.07B nulls is about 212 GB, which matches task 137.

Fixes, cheapest first:

1. Split off the null rows before the join and union them back with null customer columns. Same result for a left outer join, and the hot partition disappears.
2. Turn on adaptive query execution with skew join handling, so Spark splits oversized partitions, and size shuffle partitions to the data (1.8 TB at about 128 MB each is roughly 14,000, which AQE can coalesce).
3. The two whale customers (38M and 21M rows) still create hot partitions. Salting splits those keys over N buckets on the events side and replicates the matching customer rows N times.

Alternatives: broadcasting customers avoids the shuffle, but at 3.2 GB it is far above the default broadcast threshold and risks driver and executor memory; I would only broadcast after projecting to the needed columns, if that shrinks it under a few hundred MB. Adding memory just moves the failure to the next spike.

With an assistant I ask for the null split and salting code, then check the row count is unchanged and that nulls are not dropped from the left side.

Code:

```python
from collections import Counter

def partition_sizes(keys, n=200, salt=None):
    sizes = Counter()
    for i, k in enumerate(keys):
        if k is None:
            k = "__null__"
        if salt and k in salt:
            k = f"{k}#{i % salt[k]}"
        sizes[hash(k) % n] += 1
    return sizes

keys = [None] * 5000 + ["whale"] * 2000 + [f"c{i}" for i in range(20000)]
before = max(partition_sizes(keys).values())
non_null = [k for k in keys if k is not None]
after_null = max(partition_sizes(non_null).values())
after_salt = max(partition_sizes(non_null, salt={"whale": 32}).values())
assert before >= 5000 and after_null >= 2000 and after_salt < 400
assert len(non_null) + keys.count(None) == len(keys)
print(before, after_null, after_salt)
```

Verify: output row count equals the left table's row count; stage 14 max task read close to the median; runtime and spill compared before and after.

Pitfalls: raising executor memory and calling it fixed; an inner join that silently drops null rows; salting without replicating the right side.

Follow-ups: the same job moves to streaming; what changes for state size and late data.

Why this is likely: a prep guide built from candidate transcripts lists an out-of-memory shuffle join as a curveball in Databricks FDE coding and design rounds (medium confidence).

</details>

### EDA and classical ML (no published AI rule; ask the recruiter)

<a id="q5"></a>

#### Q5. Build a churn model on this customer table, and find why validation AUC is 0.99

Round: EDA and classical ML. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: Feature leakage and model evaluation.

The customer's data team shares a notebook. Their first model gets 0.99 AUC on a random split, and they want to deploy it.

```text
table: subscriptions_snapshot  (one row per customer, built last night)
columns:
  customer_id, signup_date, plan, seats, monthly_active_users_90d,
  usage_change_30d, support_tickets_90d, last_login_days_ago,
  cancellation_page_views, account_status, churned (label: cancelled in
  the 60 days after the cutoff date)
rows: 48,200    churn rate: 7.9%
split: train_test_split(test_size=0.2, random_state=0)
gradient boosted trees, validation AUC 0.991
top features: last_login_days_ago (0.41), account_status (0.33), cancellation_page_views (0.12)
```

<details>
<summary>Show answer</summary>

Task: explain the 0.99, rebuild the evaluation so it predicts the deployed number, and give an honest baseline.

What they grade: spotting leakage from the schema, time-aware splits, the right metric for a 7.9% base rate, explaining it to a customer.

Approach: The snapshot was built last night, but the label is about the 60 days after an earlier cutoff. So features like `last_login_days_ago`, `account_status` and `cancellation_page_views` were measured after many customers had already cancelled. A cancelled customer has a large last-login gap and a "closed" status, so the model is reading the answer. The top three features are those same columns.

The fix is a point-in-time feature table: for each cutoff date, compute features only from events before it, and label from the 60 days after. Then split by time: train on cutoffs from earlier months, validate on the latest month. Random splits also leak when the same customer appears at several cutoffs, so group by customer.

Metric: with 7.9% churn, report PR AUC and precision at the top decile next to ROC AUC, because the retention team can call maybe 500 accounts a month. A realistic honest AUC is 0.70 to 0.80; I tell the customer the lower number is the one production will see.

Alternatives: logistic regression on a few well-built features is the baseline and the explainable choice; gradient boosted trees usually add a few AUC points. I ship the trees only if the gain holds on the time split.

Code:

```python
import numpy as np

def auc(y, s):
    order = np.argsort(s)
    ranks = np.empty(len(s)); ranks[order] = np.arange(1, len(s) + 1)
    pos = y == 1
    return (ranks[pos].sum() - pos.sum() * (pos.sum() + 1) / 2) / (pos.sum() * (~pos).sum())

def fit_logreg(X, y, lr=0.5, steps=600):
    X = np.c_[np.ones(len(X)), (X - X.mean(0)) / X.std(0)]
    w = np.zeros(X.shape[1])
    for _ in range(steps):
        p = 1 / (1 + np.exp(-X @ w))
        w -= lr * X.T @ (p - y) / len(y)
    return w, X

rng = np.random.default_rng(0)
n = 20000
usage_drop = rng.normal(size=n)
tickets = rng.poisson(1.0, n)
logit = -2.8 + 0.9 * usage_drop + 0.3 * tickets
y = (rng.random(n) < 1 / (1 + np.exp(-logit))).astype(int)
login_gap_after_cutoff = np.where(y == 1, rng.normal(40, 5, n), rng.normal(5, 5, n))

honest = np.c_[usage_drop, tickets]
leaky = np.c_[usage_drop, tickets, login_gap_after_cutoff]
for name, X in [("honest", honest), ("leaky", leaky)]:
    w, Xs = fit_logreg(X, y)
    print(name, round(auc(y, Xs @ w), 3))
w, Xs = fit_logreg(honest, y); a_honest = auc(y, Xs @ w)
w, Xs = fit_logreg(leaky, y); a_leaky = auc(y, Xs @ w)
assert a_leaky > 0.98 and 0.65 < a_honest < 0.85
assert abs(auc(np.array([0, 1, 0, 1]), np.array([0.1, 0.9, 0.2, 0.8])) - 1.0) < 1e-9
```

Verify: no feature timestamp later than its cutoff (assert it in the feature job); time-split AUC within a few points of the first month in production.

Pitfalls: dropping only the top leaky feature and missing the others; a random split with repeated customers; reporting ROC AUC alone.

Follow-ups: how to pick a call threshold with a retention budget; how to show the model causes saves (holdout group).

Why this is likely: the AI FDE team lead names an EDA and classical ML round, and recruiters described the AI FDE coding round as applied data science with model evaluation (medium confidence).

</details>

<a id="q6"></a>

#### Q6. The new RAG prompt lifts thumbs-up from 62% to 65%. Is the lift real?

Round: EDA and classical ML. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: Statistics for GenAI A/B tests.

The customer ran an A/B test on their support assistant: 2,000 rated answers per arm, from about 250 users per arm (about 8 ratings each). Control 62% thumbs-up, new prompt 65%. Their analyst says p = 0.049, ship it.

<details>
<summary>Show answer</summary>

Task: decide whether the evidence supports shipping, show the math, and propose a test that would settle it.

What they grade: noticing the unit of randomization, design effects, power, and a cheaper offline check.

Approach: The analyst's z-test treats 2,000 ratings as independent. They are not: users were randomized, and each gave about 8 ratings, and a user's ratings are correlated (some people never click thumbs-up). With an intraclass correlation of 0.1, the design effect is 1 + (8 minus 1) x 0.1 = 1.7, so the effective sample is about 1,176 per arm. The z score drops from 1.97 to 1.51 and p rises to about 0.13, which is not significant.

Power: detecting 62% to 65% at 80% power and alpha 0.05 needs about 4,040 independent ratings per arm, or about 6,870 with this clustering. The current test is underpowered by about 3.4 times.

Alternatives for settling it:

1. Run longer with user-level analysis (a cluster bootstrap over users, or a mixed model). Accurate, but takes weeks.
2. Paired offline evaluation: run both prompts on the same 500 logged questions and have a calibrated judge (Q12) compare them. Pairing removes question difficulty from the variance, so a McNemar test on discordant pairs needs far fewer samples. It is fast, but measures the judge's preference, not users'.

I would do the paired offline eval now, and keep the online test running to the computed sample size before claiming a user-facing win. I also check thumbs-up rate: only a minority of users rate, so a prompt that changes who rates can move the number without changing quality.

Code:

```python
import math

def two_prop(p1, p2, n, deff=1.0):
    n_eff = n / deff
    pb = (p1 + p2) / 2
    se = math.sqrt(pb * (1 - pb) * 2 / n_eff)
    z = (p2 - p1) / se
    return z, math.erfc(z / math.sqrt(2))

def n_per_arm(p1, p2, za=1.96, zb=0.8416):
    pb = (p1 + p2) / 2
    num = za * math.sqrt(2 * pb * (1 - pb)) + zb * math.sqrt(p1 * (1 - p1) + p2 * (1 - p2))
    return num ** 2 / (p2 - p1) ** 2

deff = 1 + (8 - 1) * 0.1
z0, p0 = two_prop(0.62, 0.65, 2000)
z1, p1 = two_prop(0.62, 0.65, 2000, deff)
assert abs(p0 - 0.049) < 0.002 and abs(p1 - 0.131) < 0.002
assert 4000 < n_per_arm(0.62, 0.65) < 4100
print(round(p0, 3), round(p1, 3), round(n_per_arm(0.62, 0.65) * deff))
```

Verify: estimate the real ICC from the data instead of assuming 0.1; an A/A test on the same pipeline should give p above 0.05 about 95% of the time.

Pitfalls: treating ratings as independent; stopping when p first dips below 0.05; ignoring who chooses to rate.

Follow-ups: what if the lift is real but latency rose 300 ms; how to combine several metrics without fishing.

Why this is likely: the AI FDE team lead says the EDA round tests statistical intuition, and prep guides list A/B questions for Databricks deep dives (medium confidence).

</details>

<a id="q7"></a>

#### Q7. Forecast weekly demand for a product launching next month with no sales history

Round: Decomposition or EDA and classical ML. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: Cold-start forecasting.

A retail customer launches a new mid-range blender in five weeks and needs a weekly forecast for the first eight weeks to set stock per region. There is no sales history for it, but there are three past launches in the same category and the product's attributes.

<details>
<summary>Show answer</summary>

Task: produce a usable forecast with uncertainty now, and a plan that improves it as sales arrive.

What they grade: asking what the forecast decides, using analogs and attributes, uncertainty, a fast update rule.

Approach: First the decision. Stock per region is set weekly, and a stockout costs more than overstock for a launch, so they need a high quantile (say P80), not only the mean. Then three pieces:

1. Total volume prior. Fit a simple model of eight-week totals on past launches using attributes (price band, category, marketing spend, number of stores). With few launches, use the category median scaled by price and store count, and treat the spread of analogs as the uncertainty.
2. Launch shape. Normalize each analog's weekly sales to shares of its eight-week total and average them. Launch curves have a common shape (ramp, peak around week 3, slow decay) even when totals differ.
3. Update. After each week, compare actuals with the forecast. The implied total is cumulative sales divided by the cumulative share expected so far. Blend it with the prior, weighting the actuals more as more of the curve is observed.

Alternatives: a Bass diffusion model fits the launch shape with two parameters, but needs a market size guess and is unstable with little data. A global gradient boosted model over all products with attribute features scales to thousands of SKUs, which is where I would go for a full catalog; for one launch, analogs are faster and easier to explain to the merchandising team.

Code:

```python
import numpy as np

analogs = np.array([
    [120, 300, 410, 380, 350, 330, 310, 300],
    [80, 190, 260, 250, 240, 230, 220, 215],
    [200, 480, 600, 560, 520, 500, 480, 470]], float)
shares = analogs / analogs.sum(1, keepdims=True)
shape = shares.mean(0)

def forecast(prior_total, actuals=(), tau=0.25):
    seen = len(actuals)
    cum = shape[:seen].sum()
    total = prior_total
    if seen:
        implied = sum(actuals) / cum
        w = cum / (cum + tau)
        total = w * implied + (1 - w) * prior_total
    return total * shape[seen:]

prior = 2400.0
f0 = forecast(prior)
assert abs(f0.sum() - prior) < 1e-6 and f0.argmax() == 2
hot = forecast(prior, actuals=[2 * prior * shape[0], 2 * prior * shape[1]])
assert hot.sum() > 1.3 * prior * shape[2:].sum()
print(np.round(f0), np.round(hot))
```

Verify: backtest by pretending each past launch was new (leave one out) and measure weekly error and P80 coverage.

Pitfalls: one point forecast with no interval; using a seasonal model that needs a year of history; ignoring stockouts, which censor observed demand.

Follow-ups: how to split the forecast across regions; what to do when early sales are low because stock was low.

Why this is likely: a first-hand Databricks FDE report describes a case to forecast demand with no history, and the role covers classical ML (medium confidence).

</details>

### GenAI technical deep dive (no AI, as reported for Databricks deep dives)

<a id="q8"></a>

#### Q8. A contract assistant answers from a superseded document. Read the retrieval trace, fix it

Round: GenAI technical deep dive. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: RAG retrieval and long context.

```text
query: "What is the termination notice period for EU contractors under MSA v3?"
retriever: vector, top_k=40, chunk_size=2,000 tokens, prompt context 81,000 tokens
rank 1   msa_v2.pdf#s14            score 0.83  "...notice period of 30 days..."
rank 2   msa_v3.pdf#s14            score 0.82  "...contractors: 60 days notice..."
rank 3   msa_v2_faq.md#3           score 0.80  "...30 days notice applies to all..."
...
rank 23  msa_v3_amend_eu.pdf#s2    score 0.71  "...supersedes Section 14 for EU ... 90 days"
answer: "30 days" (cites msa_v2.pdf)
eval (412 questions): correct 58%; correct when gold chunk rank <= 5: 81%;
                      correct when gold chunk rank > 10: 44%; median context 74k tokens
```

<details>
<summary>Show answer</summary>

Task: explain the wrong answer from the trace and fix retrieval and context building.

What they grade: separating retrieval failures from generation failures, document versioning, long-context behavior, measurable fixes.

Approach: Three failures stack up here.

1. Versioning. The v2 contract and its FAQ are still indexed and score as high as v3, because the text is nearly the same. Nothing tells the retriever which document is current. Fix with metadata: doc family, version, effective date, superseded_by. Filter to the current version by default, and link amendments to the sections they change.
2. Rank of the right evidence. The amendment that decides the answer sits at rank 23 of 40. The eval shows accuracy falls from 81% to 44% when the gold chunk is below rank 10. Models use the start and end of a long context much better than the middle, so 81,000 tokens of near-duplicates hides the answer. Fix with smaller chunks (about 400 to 600 tokens, by clause), hybrid search so "EU" and "Section 14" match exactly, then a reranker that keeps the top 8.
3. Generation. The prompt never says to prefer the latest effective document or to report conflicts. Add that, and ask for the clause and version in the citation.

Alternatives: putting the whole contract set in a long-context model avoids retrieval but costs roughly 80,000 input tokens per question and still suffers from the middle-of-context problem; fine-tuning on contracts would freeze facts that change with each amendment. Better retrieval wins.

Code:

```python
def build_context(hits, query_scope=None, budget_tokens=4000, keep=8):
    superseded = {h["supersedes"] for h in hits
                  if h.get("supersedes") and h.get("scope") in (None, query_scope)}
    current = [h for h in hits if h["doc"] not in superseded and not h.get("obsolete")]
    current.sort(key=lambda h: -h["rerank"])
    out, used = [], 0
    for h in current[:keep]:
        if used + h["tokens"] > budget_tokens:
            break
        out.append(h); used += h["tokens"]
    return out

hits = [
    {"doc": "msa_v2#s14", "rerank": 0.62, "tokens": 500, "obsolete": True},
    {"doc": "msa_v3#s14", "rerank": 0.70, "tokens": 500},
    {"doc": "msa_v3_amend_eu#s2", "rerank": 0.91, "tokens": 400,
     "supersedes": "msa_v3#s14", "scope": "EU"},
    {"doc": "msa_v2_faq#3", "rerank": 0.55, "tokens": 300, "obsolete": True},
]
ctx = build_context(hits, query_scope="EU")
assert [h["doc"] for h in ctx] == ["msa_v3_amend_eu#s2"]
us = [h["doc"] for h in build_context(hits, query_scope="US")]
assert "msa_v3#s14" in us
assert build_context([]) == []
print("ok")
```

Verify: report retrieval recall at 5 and answer accuracy separately, sliced by gold rank; add 30 conflict questions where only the amendment is right. Add a test where a US question retrieves the EU amendment and keeps msa_v3#s14, since an amendment supersedes its base section only inside its own scope.

Pitfalls: raising top_k to "give the model more"; deleting old versions that some questions legitimately ask about; judging only final answers.

Follow-ups: how to version an index without downtime; how to handle a question about the old contract.

Why this is likely: the AI FDE team lead lists RAG, vector search and transformer depth for the deep dive, and prep guides report questions on where long-context RAG fails (high confidence).

</details>

<a id="q9"></a>

#### Q9. Fine-tune a model or improve the context for a support assistant? Decide with numbers

Round: GenAI technical deep dive. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Fine-tuning versus retrieval and prompting.

A customer's support assistant handles 50,000 questions a day. Prompts average 6,000 tokens (instructions, 12 examples, retrieved docs). Answers are correct 78% of the time, but the format often breaks their ticketing tool, and product docs change weekly. Their CTO wants to fine-tune.

<details>
<summary>Show answer</summary>

Task: recommend what to change, in what order, with numbers on quality, cost and upkeep.

What they grade: matching the method to the failure type, cost math, an eval plan that decides.

Approach: Split the 22% failures before choosing. I would label 200 failed answers: wrong or missing facts, ignored instructions, and broken format. Fine-tuning teaches behavior (format, tone, tool calls); it does a poor job of teaching facts that change weekly, because every doc change means retraining.

Numbers: 50,000 x 6,000 = 300M input tokens a day. A fine-tuned model that no longer needs the 12 examples and long instructions could run on about 1,500 tokens, or 75M a day, a 4x cut, and a smaller fine-tuned model is cheaper per token too. That saving is real, but only if the retrieved context still fits.

Plan:

1. Fix format first with structured output (a JSON schema enforced at decode time) and a validator with one retry. That usually takes format errors close to zero with no training.
2. Fix facts with retrieval: hybrid search, a reranker, fresher index sync. Measure retrieval recall separately.
3. Fine-tune (LoRA on a smaller model) for tone and policy once 1 and 2 are done, using a few thousand reviewed transcripts, and keep retrieval for facts. Gate it on the same eval set: equal or better accuracy, lower cost and latency.

Alternatives: prompt only (fast, but 300M tokens a day); retrieval plus structured output (fixes most failures, cheapest to maintain); fine-tune plus retrieval (lowest serving cost, adds a training pipeline and model versioning). Distilling a large model into a small one is the end state if volume grows.

Numbers: target accuracy above 90% on 500 labeled questions, format validity above 99.5%, p95 latency under 3 seconds.

Risks: fine-tuning on transcripts copies agents' mistakes; a tuned model can lose general abilities, so keep a regression set; retraining cadence becomes a team job the customer must own.

Why this is likely: prep guides report "fine-tuning or better context" deep-dive questions for Databricks FDE candidates, and the AI FDE job lists fine-tuning work (medium confidence).

</details>

<a id="q10"></a>

#### Q10. Raise a text-to-SQL assistant from 58% to production accuracy, and prove it

Round: GenAI technical deep dive. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Text-to-SQL evaluation and improvement.

A finance team uses a text-to-SQL assistant over 40 tables. Their 120-question benchmark says 58% correct by exact string match. Analysts complain about wrong totals for "net revenue" and fiscal quarters.

<details>
<summary>Show answer</summary>

Task: fix the measurement, then raise accuracy with changes you can attribute.

What they grade: execution-based scoring, error analysis before fixes, using business definitions, a release bar.

Approach: Exact string match is the wrong metric, because `SELECT SUM(amount) - SUM(refund)` and `SELECT SUM(amount - refund)` are both right but differ as strings. (If column order should not matter either, compare each row as a sorted tuple of values, or map columns by name, before comparing.) Score by execution: run the generated and gold SQL on a fixed snapshot and compare result sets (ignore row order unless the question asks for it, round floats). That change usually moves the number, in either direction, and gives the true baseline.

Then error analysis on failures, sorted into buckets: wrong business definition ("net revenue" excludes refunds and intercompany), wrong date logic (fiscal year starts in February), wrong join path, wrong filter value, and ambiguous question. The analyst complaints point at definitions and fiscal dates.

Fixes, each measured on the benchmark:

1. Definitions: write metric views or certified SQL for "net revenue" and a fiscal calendar table, and point the model at them. Databricks' Genie guidance recommends this kind of curated instructions, example SQL and trusted assets.
2. Schema context: column descriptions and sample values for the 40 tables, retrieved per question rather than all at once.
3. Ambiguity: let the assistant ask a clarifying question instead of guessing.
4. Grow the benchmark to 300 questions from real logs, with a held-out set so prompt tuning does not overfit.

Release bar: above 90% execution accuracy on the held-out set, and 100% on a 30-question "money" subset that finance signs off.

Code:

```python
import sqlite3

def execution_match(conn, pred_sql, gold_sql, ordered=False):
    try:
        pred = conn.execute(pred_sql).fetchall()
    except sqlite3.Error:
        return False
    gold = conn.execute(gold_sql).fetchall()
    norm = lambda rows: [tuple(round(v, 2) if isinstance(v, float) else v for v in r) for r in rows]
    pred, gold = norm(pred), norm(gold)
    return pred == gold if ordered else sorted(pred) == sorted(gold)

c = sqlite3.connect(":memory:")
c.execute("CREATE TABLE sales(region TEXT, amount REAL, refund REAL)")
c.executemany("INSERT INTO sales VALUES (?,?,?)", [("EU", 100, 10), ("US", 200, 0), ("EU", 50, 5)])
gold = "SELECT region, SUM(amount - refund) FROM sales GROUP BY region"
assert execution_match(c, "SELECT region, SUM(amount) - SUM(refund) FROM sales GROUP BY region ORDER BY region DESC", gold)
assert not execution_match(c, "SELECT region, SUM(amount) FROM sales GROUP BY region", gold)
assert not execution_match(c, "SELEC broken", gold)
print("ok")
```

Verify: a few gold queries reviewed by finance; check the scorer against 20 hand-judged pairs.

Pitfalls: tuning prompts on the test questions; counting a query that returns the right number by luck on an empty snapshot; no clarifying path.

Follow-ups: use DSPy to optimize the prompt against execution accuracy; route out-of-scope questions to a human.

Why this is likely: the AI FDE job lists Text2SQL work, and Databricks publishes production-readiness guidance for Genie text-to-SQL spaces built around benchmarks (medium confidence).

</details>

### Productionization and LLMOps design (no published AI rule; expect no AI and confirm with the recruiter)

<a id="q11"></a>

#### Q11. Take a customer's GenAI proof of concept to production with eval gates and monitoring

Round: Productionization and LLMOps design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: LLMOps, CI/CD and governance.

An insurer has a notebook proof of concept: a RAG assistant over 30,000 policy documents that impressed executives in a demo. They want it live for 2,000 internal agents in eight weeks. Today it has no tests, a hard-coded API key, one prompt in a string, and an index built by hand.

<details>
<summary>Show answer</summary>

Task: design the path from notebook to a governed production service, with what blocks a release.

What they grade: evaluation as the release gate, separation of environments, governance, monitoring that leads to action, a realistic plan.

Approach: I treat it as four pipelines, each versioned and owned.

1. Data and index. A scheduled job ingests documents into governed tables (Unity Catalog for permissions and lineage), chunks them, and syncs a vector index. Document permissions travel as metadata so retrieval filters by the agent's entitlements.
2. Application. Prompt, retriever settings and model choice live in code, logged as one versioned model with MLflow, with tracing on every call. Secrets move to a secret scope.
3. Evaluation and CI/CD. A golden set of about 300 questions with reviewed answers, built with the insurer's subject matter experts, plus scorers: correctness and groundedness judges, retrieval recall, and safety checks. Every change (prompt, chunking, model) runs in CI, and the release is blocked if correctness falls more than 2 points or groundedness falls below 95%. Dev, staging and prod are separate workspaces or catalogs.
4. Serving and monitoring. A model serving endpoint with rate limits and logging of requests, retrieved docs and feedback. Production traces are sampled and scored by the same judges daily, so offline and online numbers are comparable.

Numbers: 2,000 agents at about 20 questions a day is 40,000 a day, a peak near 5 per second; at 4,000 input tokens each that is 160M tokens a day, which sets endpoint sizing and cost. Target p95 under 4 seconds.

Alternatives: shipping the notebook behind an API is fastest but unreviewable and ungoverned; a fully custom stack on raw cloud services gives control but takes the customer past eight weeks and leaves them with more to own. Managed serving with MLflow evaluation and tracing wins for an eight-week deadline.

Risks: experts not available to label (book their time in week one); permission leaks through retrieval (test with a user who lacks access); judge drift (recalibrate quarterly, see Q12).

Why this is likely: the AI FDE team lead describes a productionization round on evals, CI/CD, MLOps and LLMOps, and MLflow 3 added GenAI evaluation and tracing that the role would use (high confidence).

</details>

<a id="q12"></a>

#### Q12. Your LLM judge agrees with humans 85% of the time. Is it good enough to gate releases?

Round: Productionization and LLMOps design. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: LLM judge calibration.

Experts labeled 300 production answers pass or fail. The judge agrees on 255 of them. The confusion table:

```text
                  judge pass   judge fail
human pass (240)      228           12
human fail  (60)       33           27
```

<details>
<summary>Show answer</summary>

Task: decide whether the judge can gate releases, fix it if not, and estimate the true pass rate either way.

What they grade: looking past raw agreement, chance-corrected agreement, per-class error, and correcting estimates for judge error.

Approach: 85% agreement sounds good, but 80% of answers pass, so a judge that always says pass agrees 80% of the time. Cohen's kappa corrects for that and comes out at about 0.46, which is moderate. The bigger problem shows up per class. The judge catches 95% of good answers but only 45% of bad ones (27 of 60). A release gate exists to catch bad answers, so this judge misses more than half of regressions.

Fixes, measured on a held-out half of the labels:

1. Make the criteria concrete. Replace "is this answer good" with separate binary checks: is every claim supported by the retrieved text, does it answer the question asked, does it follow policy.
2. Give the judge the reference answer and the retrieved context, and ask for reasons before the verdict.
3. Add a few labeled examples of subtle failures to the judge prompt.
4. Target above 80% catch rate on failures and kappa above 0.7 before gating.

Even an imperfect judge can estimate the true pass rate if its error rates are known: true rate = (judged rate + specificity minus 1) / (sensitivity + specificity minus 1). That turns a judged 87% into the true 80%.

Alternatives: human review of every release is accurate but slow and costly; code-based checks (citation present, JSON valid) are exact but narrow. I use code checks where possible, the calibrated judge for meaning, and humans on a sample of disagreements each week.

Code:

```python
def judge_stats(tp, fn, fp, tn):
    n = tp + fn + fp + tn
    po = (tp + tn) / n
    h_pass, j_pass = (tp + fn) / n, (tp + fp) / n
    pe = h_pass * j_pass + (1 - h_pass) * (1 - j_pass)
    sens, spec = tp / (tp + fn), tn / (tn + fp)
    return {"agree": po, "kappa": (po - pe) / (1 - pe), "sens": sens, "spec": spec}

def corrected_rate(judged, sens, spec):
    return min(1.0, max(0.0, (judged + spec - 1) / (sens + spec - 1)))

s = judge_stats(tp=228, fn=12, fp=33, tn=27)
assert abs(s["agree"] - 0.85) < 1e-9 and abs(s["kappa"] - 0.46) < 0.01
assert abs(s["spec"] - 0.45) < 1e-9
assert abs(corrected_rate(261 / 300, s["sens"], s["spec"]) - 0.80) < 1e-9
print({k: round(v, 3) for k, v in s.items()})
```

Verify: report kappa and failure catch rate with a bootstrap interval; recheck on fresh labels after every judge prompt change.

Pitfalls: quoting raw agreement; calibrating and testing on the same labels; one judge model grading its own outputs.

Follow-ups: how many labels you need for a tight interval on the catch rate (60 failures gives roughly plus or minus 13 points); using DSPy to optimize the judge prompt against expert labels.

Why this is likely: the AI FDE job lists evaluation as core work, and the Databricks productionization round covers evals; MLflow documents LLM judge scorers and aligning them with human feedback (high confidence).

</details>

<a id="q13"></a>

#### Q13. Offline evals pass, but production answer quality fell for three weeks. Read the monitor

Round: Productionization and LLMOps design. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Production monitoring and drift.

```text
week  requests  judge_pass  retrieval_hit@5  share_german  index_lag_hours  new_sku_share
w1     41k        0.86          0.91             4%               2              3%
w2     43k        0.85          0.90             5%               2              4%
w3     44k        0.79          0.81             5%              31              9%
w4     46k        0.74          0.75            12%              77             14%
w5     47k        0.72          0.73            13%             125             16%
offline golden set (fixed 400 questions): judge_pass 0.88 every week
w5 slices: questions about products added after w2: 0.41; other products: 0.83
           German questions: 0.69; English questions: 0.73
index sync job: last success end of w2; later runs "succeeded" with 0 rows upserted
```

<details>
<summary>Show answer</summary>

Task: find the causes from the monitor, rank them, and fix both the incident and the monitoring gap.

What they grade: reading slices, separating data freshness from model quality, noticing why offline missed it.

Approach: The golden set is fixed, so it cannot see new products or new users; that is why offline stays at 0.88. The production slices show the causes.

1. Stale index, the main cause. Index lag climbs from 2 to 125 hours, and the sync job has upserted zero rows since week 2 while reporting success. Questions about products added after week 2 score 0.41 against 0.83 for others, and new products grew to 16% of traffic. That 16% losing 42 points explains about 7 of the 14 lost points.
2. Language shift, a smaller cause. German grew from 4% to 13%. German scores are 4 points lower and German traffic grew by 9 points of share, so the mix shift costs under half a point (0.09 x 0.04 = 0.36); it is small now but will grow.
3. Retrieval hit rate fell with freshness, not with the model, so swapping the LLM would not have helped.

Fixes: repair the sync (the job likely lost its source filter or permissions), backfill, and make "0 rows upserted for 24 hours" an alert. Add index lag and new-content share to the dashboard with thresholds. Refresh the golden set monthly with sampled production questions, including German ones, and keep a frozen subset for trend lines. For German, test a multilingual embedding model on a German slice before switching.

Alternatives for detection: alert on judge pass rate alone (catches it late, gives no cause); alert on input drift metrics such as PSI on language and product mix (early, but noisy); alert on pipeline health such as lag and row counts (cheapest and most direct here). I use all three, with pipeline health paging and drift as a daily report.

Code:

```python
import math

def psi(expected, actual, eps=1e-6):
    keys = set(expected) | set(actual)
    return sum((actual.get(k, 0) - expected.get(k, 0)) *
               math.log((actual.get(k, 0) + eps) / (expected.get(k, 0) + eps)) for k in keys)

w1 = {"en": 0.96, "de": 0.04}
w5 = {"en": 0.87, "de": 0.13}
assert 0.1 < psi(w1, w5) < 0.25
assert psi(w1, w1) == 0
drop_from_stale = 0.16 * (0.83 - 0.41)
assert 0.06 < drop_from_stale < 0.08
print(round(psi(w1, w5), 3), round(drop_from_stale, 3))
```

Verify: after backfill, new-product slice back near 0.8 and index lag under 4 hours; a test that fails the sync job on zero upserts.

Pitfalls: blaming the model; trusting a job's "succeeded" status; a golden set that never changes.

Follow-ups: how to sample and label production traffic cheaply; what to alert on for an agent with tools.

Why this is likely: prep guides report "offline metrics fine but production degrades" questions for Databricks FDE candidates, and the team lead lists monitoring and drift in the productionization round (high confidence).

</details>

<a id="q14"></a>

#### Q14. Design a supervisor agent that routes between a text-to-SQL agent and a document agent

Round: Productionization and LLMOps design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Multi-agent systems.

A manufacturer wants one assistant for plant managers. Some questions need numbers from the data warehouse ("scrap rate on line 4 last month"), some need the maintenance manuals ("torque spec for the B7 pump"), and some need both ("is line 4's scrap above the limit in our quality policy?").

<details>
<summary>Show answer</summary>

Task: design the routing, the agents, how combined questions work, and how you know it works.

What they grade: when to use multiple agents at all, routing evaluation, failure handling, permissions, cost and latency.

Approach: A supervisor model reads the question and plans: call the SQL agent, the document agent, or both in sequence, then compose the answer with citations to the query and the manual section. Each sub-agent is its own tested component: the SQL agent with its own benchmark (Q10), the document agent with retrieval and answer evals (Q8). The supervisor sees short descriptions of each agent's scope and returns a structured plan, so routing is inspectable.

For combined questions the supervisor passes results forward: the SQL agent returns 3.4% scrap, the document agent finds the 3% limit, and the supervisor compares them. It must show both numbers and sources, not just "yes".

Evaluation: a routing set of about 200 labeled questions (SQL, docs, both, out of scope), scored for route accuracy, plus end-to-end correctness. Route errors and sub-agent errors are reported separately so we fix the right part.

Numbers: each extra model call adds about 1 to 2 seconds; a both-agents question with planning and composing is four calls, so p95 near 8 seconds. Route single-source questions directly, and cache schema and manual lookups.

Alternatives: one agent with both tools is simpler and often enough with two tools; I would start there and move to a supervisor once sub-agents need their own prompts, permissions and owners. A rule-based router (keywords like "rate", "last month") is fast and cheap, but brittle on mixed questions. Databricks' Agent Bricks Supervisor Agent offers this pattern managed, which saves building the orchestration.

Risks: the supervisor answers from its own knowledge instead of calling an agent (require citations or refuse); permission bypass, where a manager sees another plant's data (the SQL agent runs with the user's identity); loops between agents (cap at 5 steps); cost growth from retries.

Follow-ups: in loops with a build, demo and panel round, you may build a small version of this agent, demo it, and take hostile questions ("why not one agent?", "what happens when the warehouse is down?", "show me a wrong answer"). Bring the routing eval numbers and one failure you found and fixed (Blind FDE thread, Sep 2026, medium confidence).

Why this is likely: the AI FDE job lists multi-agent systems and Text2SQL, and Databricks made a supervisor agent for orchestrating enterprise agents generally available (medium confidence).

</details>

### Decomposition and customer scenario (no AI)

<a id="q15"></a>

#### Q15. A VP says "use AI to cut claims processing time". Scope the engagement

Round: Decomposition or customer scenario. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Problem decomposition with a customer.

You are on a call with an insurer's VP of claims. The brief is one sentence. You have 45 minutes to turn it into a plan.

<details>
<summary>Show answer</summary>

Task: turn a vague goal into a scoped, measurable first project, out loud, asking before designing.

What they grade: questions before architecture, a baseline metric, picking the right first slice, honest risks.

Approach: I start with questions and leave the diagram for later.

1. Outcome: which time? Time to first contact, to decision, or to payment? Who is measured on it? Suppose the answer is "average days from first notice of loss to decision, now 11 days".
2. Where the time goes: ask for the process map and the event log. Typical breakdown: waiting in queues, document collection, manual review of documents, approvals. Suppose 4 of the 11 days are adjusters reading and summarizing documents, and 3 days are waiting for missing documents.
3. Volume and mix: 20,000 claims a month, 60% simple auto claims.
4. Constraints: regulation on automated decisions, data residency, what systems hold the documents.

Then I pick the first slice. Fully automated claim decisions are the biggest prize and the biggest risk. A narrower first project: extract and summarize claim documents for simple auto claims and flag missing documents on day one. It attacks 7 of the 11 days, keeps an adjuster deciding, and is measurable.

Numbers: if summaries save 2 of the 4 reading days and early missing-document requests save 2 of the 3 waiting days on 60% of claims, the average falls by about 2.4 days, about 22%. That is the hypothesis to test.

Plan: two weeks to baseline from the event log and label 200 claims; four weeks to build extraction with evals (field accuracy above 95% on key fields); a pilot with one adjuster team against a control team, measuring days to decision and rework.

Alternatives: a chat assistant for adjusters (easy to ship, hard to measure); straight-through processing for the simplest claims (big gain, needs legal sign-off). I would propose the latter as phase two once extraction is proven.

Risks: no clean event log (measure a sample by hand); adjusters not trusting summaries (show sources per field); scope creep to all claim types.

Why this is likely: first-hand reports describe a vague decomposition round for Databricks FDE candidates where they had to find stakeholders and KPIs first (medium confidence).

</details>

### Behavioral (no AI)

<a id="q16"></a>

#### Q16. A customer wants to ship an AI agent next week without evals. How do you push back?

Round: Behavioral and culture. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Customer pushback and responsible AI delivery.

Tell me about a time you pushed back on a customer, or on your own team, about shipping AI work before it was checked. How do you use AI tools yourself, and how do you check their output?

<details>
<summary>Show answer</summary>

Task: one specific story with numbers, the disagreement handled with respect, and a clear habit for checking AI output.

What they grade: customer obsession with spine, ownership, judgment about AI quality, how you check AI-generated work.

Approach: Use one situation, your task, the actions and the result, and keep it to about three minutes.

Example shape: "A retail customer's executive wanted their returns agent live for the holiday peak, eight days out, with no evaluation. I did not argue in principle. I asked for two days, pulled 150 real returns conversations, and had their support lead label them. The agent was right on 71%, and 9 answers promised refunds outside policy. I showed the executive those 9 next to the policy. We agreed on a smaller launch: the agent handled order status and return labels only, refunds went to humans, and the full scope shipped three weeks later at 92% on the same set. The narrow version still deflected about 30% of contacts over the peak."

Then I cover AI use. I use coding assistants for boilerplate, tests and data plumbing, and I keep business rules, metrics and anything touching customer data under my own review. One habit from that project: the assistant wrote the label-matching script for the eval, and it compared order ids as strings with a trailing space, which counted 14 correct answers as wrong. A test on 10 hand-checked rows caught it. Now every AI-written eval script gets a small set of known answers before I trust its numbers.

Verify: the story has a number for the risk found, the compromise, and the outcome; the AI example has a concrete error and the check that caught it.

Pitfalls: a story where you simply refused; a story where the customer was foolish; claiming AI tools are never wrong for you.

Follow-ups: what if the executive had overruled you; how you would explain the risk in their terms.

Why this is likely: Databricks' interview-prep page lists behavioral interviews, prep guides report pushback-on-customer questions for FDE roles, and candidates report being expected to vet AI output in coding rounds (medium confidence).

</details>

## Prep checklist

- [ ] Build a small RAG pipeline end to end (chunker, BM25 plus vector retrieval, fusion, reranker) and time yourself extending it in stages with tests.
- [ ] Practice one paired coding session with an AI assistant where you write the tests first and review every diff line out loud.
- [ ] Hand-write AUC, a two-proportion test with a design effect, and Cohen's kappa in plain Python.
- [ ] Rebuild a churn or demand model with point-in-time features and a time-based split, and explain the leakage you avoided.
- [ ] Prepare a 10-minute deep dive on one GenAI system you shipped, with "why this model, why not X" answers for each component.
- [ ] Learn MLflow evaluation, tracing and LLM judge scorers well enough to sketch an eval gate in CI.
- [ ] Read Databricks' posts on hybrid search, Genie production readiness, Agent Bricks Supervisor Agent and DSPy, and form one opinion on each.
- [ ] Rehearse a Spark skew and out-of-memory join diagnosis from a physical plan and task statistics.
- [ ] Practice two vague-goal decomposition cases out loud: questions first, then a baseline metric, then a narrow first slice.
- [ ] Prepare one pushback story and one AI-use story, each with numbers.
- [ ] Ask the recruiter whether AI tools are allowed in the coding and build rounds, and whether there is a build and demo panel.

## Sources

- Databricks careers, interview preparation: https://www.databricks.com/company/careers/interview-prep
- Databricks engineering blog, a hiring manager's guide to engineering interviews: https://www.databricks.com/blog/2020/01/22/engineering-intervews-a-hiring-managers-guide.html
- Interview with the Head of AI FDE EMEA on the AI FDE loop: https://www.youtube.com/watch?v=fKr6mDHfoIM
- Blind, FDE interview at Databricks: https://www.teamblind.com/post/fde-interview-at-databricks-4b2y5fqe
- Reddit, FDE interview at Databricks: https://www.reddit.com/r/leetcode/comments/1w8bs6j/fde_interview_at_databricks
- Reddit, AI FDE interviews: https://www.reddit.com/r/cscareerquestionsuk/comments/1vhctj7/anyone_been_through_interviews_for_ai_fde_at
- Exponent, Databricks forward deployed engineer interview experiences: https://www.tryexponent.com/experiences/databricks-forward-deployed-engineer-interview-ef1a4b
- Prepfully, Databricks forward deployed engineer guide: https://prepfully.com/interview-guides/databricks-forward-deployed-engineer
- DataInterview, Databricks FDE guide (low confidence): https://www.datainterview.com/blog/databricks-forward-deployed-engineer-interview
- Community notes on Databricks AI engineer questions (low confidence): https://github.com/ombharatiya/AI-Engineer-Interview-Questions/blob/main/14-company-interview-questions/databricks.md
- Databricks, MLflow 3.0: https://www.databricks.com/blog/mlflow-30-unified-ai-experimentation-observability-and-governance
- Databricks docs, MLflow GenAI scorers: https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/concepts/scorers
- Databricks, Agent Bricks Supervisor Agent: https://www.databricks.com/blog/agent-bricks-supervisor-agent-now-ga-orchestrate-enterprise-agents
- Databricks, production-ready Genie spaces: https://www.databricks.com/blog/how-build-production-ready-genie-spaces-and-build-trust-along-way
- Databricks, hybrid search in Mosaic AI Vector Search: https://www.databricks.com/blog/announcing-hybrid-search-general-availability-mosaic-ai-vector-search
- Databricks, DSPy on Databricks: https://www.databricks.com/blog/dspy-databricks
