# Netflix: AI Engineer 6 - AI Foundation & Tooling, Ads Platform

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Netflix lets each hiring team write its own interviews. No public report covers this exact role, which is the team's first dedicated AI engineer. The shape below combines the Netflix Tech Blog description of its backend loop with 2026 reports from the Ads org, which hires through a centralized pipeline and matches candidates to teams after the onsite.

1. Recruiter call (about 30 minutes, virtual). Background, why Netflix, the culture memo, and level. Ads recruiters describe the centralized pipeline and later team match. AI: not relevant.
2. Hiring manager screen (30 to 45 minutes, virtual). One or two AI systems you shipped, and how you would stand up AI tooling for a team that has champions but no foundation layer. AI: not relevant.
3. Technical phone screen (about 60 minutes, virtual). One coding problem with several follow-ups. Ads reports describe medium-difficulty problems in Netflix framing, such as topological sort over dependencies and caches with expiry. Staff-level reports mention a system design screen. AI: not allowed in any Ads report.
4. Onsite coding (45 to 60 minutes). Practical coding judged on clean, tested code: tree and graph traversal, caches, tradeoff discussion. AI: not allowed in Ads reports.
5. AI-assisted practical (about 60 minutes, team-dependent). Some 2026 Netflix onsites added a round in a real codebase with an inline assistant and a live interviewer, graded on how you prompt, review, test, and push back on the AI. Whether this team runs it is unknown. AI: allowed.
6. Onsite system design (45 to 60 minutes, one or two rounds). For this role, expect an AI platform design (context layer, pre-review, incident triage) and possibly the Ads design the org reuses (frequency capping). AI: unknown; no report mentions an assistant.
7. Onsite data modeling and business impact (45 to 60 minutes). Ads loops model the advertiser, campaign, line item, creative and impression hierarchy and discuss the business tradeoffs. It may be swapped for an AI depth round for this role. AI: unknown.
8. Second onsite round (2 to 3 interviews of about 45 minutes). A director, a partner engineer, and another leader probe partnering, judgment, adoption and the culture memo. AI: not relevant.
9. Team match and decision. Ads candidates report one or two extra hiring manager chats before an offer. AI: not relevant.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Design the centralized context layer that gives every coding and ops agent grounded... | Design | Onsite system design | Unknown | Hard |
| [Q2](#q2) | Our code-search retrieval cites a function renamed two weeks ago and never returns... | Debug | Onsite coding | Not allowed | Medium |
| [Q3](#q3) | Design a PR pre-review agent that flags drift from the team's architecture decisions... | Design | Onsite system design | Unknown | Hard |
| [Q4](#q4) | An assistant wrote this webhook handler for the pre-review bot. Review it like a PR... | Review | AI-assisted practical | Allowed | Medium |
| [Q5](#q5) | With an assistant, build CI test selection and a gate for AI-generated tests in this... | Build | AI-assisted practical | Allowed | Medium |
| [Q6](#q6) | Design an agent-assisted deployment validation step for Ads services. What does the... | Design | Onsite system design | Unknown | Hard |
| [Q7](#q7) | Design the eval harness for the pre-review agent, and the hill-climbing loop you would... | Design | Onsite system design | Unknown | Hard |
| [Q8](#q8) | Pre-review v2 catches 224 of 400 seeded defects and v1 catches 212. Do you ship v2? | Concept plus code | Onsite system design | Unknown | Medium |
| [Q9](#q9) | Debug this incident triage agent trace: it blamed the wrong change for an ad decisioning... | Trace debug | Onsite system design | Unknown | Medium |
| [Q10](#q10) | Build the change-correlation step for incident triage: rank recent changes by how... | Build | Onsite coding | Not allowed | Medium |
| [Q11](#q11) | This agent loop hangs some sessions and hides tool failures. Find the bugs and fix... | Debug | Onsite coding | Not allowed | Medium |
| [Q12](#q12) | Design multi-agent orchestration where parallel agents implement, test and document... | Design | Onsite system design | Unknown | Hard |
| [Q13](#q13) | Design an ads frequency cap service: a member sees a line item at most N times per... | Design | Onsite system design | Unknown | Hard |
| [Q14](#q14) | Model the ads demand hierarchy and write the pacing query that flags line items... | Concept plus code | Onsite data modeling and business impact | Unknown | Medium |
| [Q15](#q15) | (phone screen, no AI) Agent pipeline jobs have dependencies. Return the minimum number... | Classic algorithm | Technical phone screen | Not allowed | Medium |
| [Q16](#q16) | (onsite coding, no AI) Build a cache for LLM responses with a per-entry TTL and a total... | Classic algorithm | Onsite coding | Not allowed | Medium |
| [Q17](#q17) | Tell me how you got a skeptical team to adopt AI tooling, and how you check AI output... | Behavioral | Second onsite round | Unknown | Medium |

### Context layer and retrieval

<a id="q1"></a>

#### Q1. Design the centralized context layer that gives every coding and ops agent grounded, team-specific knowledge of the Ads Platform codebase, decisions and runbooks.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Context engineering and RAG for engineering agents.

<details>
<summary>Show answer</summary>

Task: Design one shared context service that IDE assistants, CI agents and incident agents all use, with freshness, access control and a way to measure quality.

What they grade: concrete sources and freshness rules; hybrid retrieval with reasons; one interface for many agents; evals tied to real tasks.

Numbers: Assume about 30 services, 1.5 million lines of code, and 2,000 docs (architecture decision records, runbooks, postmortems, API specs). At 40-line chunks with 10 lines of overlap that is about 50,000 code chunks plus about 20,000 doc chunks. At 1,024 float32 dimensions each vector is 4 KB, so about 280 MB of vectors, which fits in memory on one node.

Approach: Ingest four kinds of context. (1) Code, chunked on symbol boundaries from a parser where possible and on line windows as a fallback, keyed by path and blob hash. (2) Decisions: ADRs and the canonical patterns the team picked. (3) Operations: runbooks, postmortems, service ownership, dashboards. (4) Live state through tools, not the index: current deploys, flags, on-call. Reindex on each merge from a webhook, delete chunks whose blob hash no longer matches HEAD, and record index lag as a metric.

Retrieval is hybrid: BM25 for identifiers and error strings, vectors for intent, then merge with reciprocal rank fusion and cap chunks per file so one big file cannot fill the context. Decisions get a fixed slot in every coding session, which is how Netflix's UI Platform team describes seeding agent sessions with architecture decisions before code is written. Expose it through one MCP server (search_code, get_decision, get_runbook) so every agent shares one source of truth and one permission check.

Alternatives: Vector-only search misses exact symbols. Long-context stuffing of whole services costs more per call and dilutes attention. A fine-tuned model goes stale on every merge. Hybrid retrieval with an MCP front wins on freshness and cost.

Risks: Stale chunks produce confident wrong answers, so the index must answer "as of which commit". Filter secrets out of docs at ingest.

Verify: Build a golden set of 200 questions from real PR comments and incident threads with known source files, and track recall at 10 per release.

Why this is likely: Netflix job listing (centralized context layer, RAG, grounding) plus a Netflix engineer's QCon San Francisco 2026 abstract describing a context layer that runs the length of the SDLC; high confidence on topic, medium on the round.

</details>

<a id="q2"></a>

#### Q2. Our code-search retrieval cites a function renamed two weeks ago and never returns short config files. Here is the indexer. Find the bugs and fix them.

Round: Onsite coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Chunking and index freshness.

```python
def chunk_lines(lines, size=40, overlap=10):
    chunks = []
    start = 0
    while start + size < len(lines):
        chunks.append((start, lines[start:start + size]))
        start += size - overlap
    return chunks


def retrieve(index, query_vec, k=5):
    scored = [(dot(query_vec, c["vec"]), c) for c in index]
    scored.sort(key=lambda x: x[0], reverse=True)
    return [c for _, c in scored[:k]]


def reindex(repo, index, changed_paths):
    for path in changed_paths:
        for start, chunk in chunk_lines(repo.read(path).splitlines()):
            index.append({"path": path, "blob": repo.blob(path),
                          "vec": embed("\n".join(chunk)), "text": chunk})
```

<details>
<summary>Show answer</summary>

Task: Explain both symptoms from the code, fix them, and add tests that would have caught them.

What they grade: tracing the loop bound by hand; spotting append-only reindexing; a fix with a coverage test; a note on top-k crowding.

Approach: Run the chunker by hand. A 40-line file gives `0 + 40 < 40`, which is false, so it produces no chunks at all: every file of 40 lines or fewer is invisible, which explains the missing config files. A 45-line file yields lines 0 to 39, then `30 + 40 < 45` is false, so lines 40 to 44 are lost. The bound must cover the tail.

The stale function comes from `reindex`. It appends new chunks but never removes the old ones, and `retrieve` never checks whether a chunk's blob is still the one at HEAD. The old `getBidFloor` chunk is still in the index and still scores well.

A third issue: with overlapping chunks, one large file can fill all k slots. Cap chunks per file.

```python
def chunk_lines(lines, size=40, overlap=10):
    if not 0 <= overlap < size:
        raise ValueError("overlap must be in [0, size)")
    chunks, start = [], 0
    while start < len(lines):
        chunks.append((start, lines[start:start + size]))
        if start + size >= len(lines):
            break
        start += size - overlap
    return chunks


def dot(a, b):
    return sum(x * y for x, y in zip(a, b))


def retrieve(index, query_vec, head_blob, k=5, per_file=2):
    live = [c for c in index if head_blob.get(c["path"]) == c["blob"]]
    live.sort(key=lambda c: dot(query_vec, c["vec"]), reverse=True)
    out, used = [], {}
    for c in live:
        if used.get(c["path"], 0) < per_file:
            out.append(c)
            used[c["path"]] = used.get(c["path"], 0) + 1
        if len(out) == k:
            break
    return out


if __name__ == "__main__":
    for n in (1, 39, 40, 41, 45, 100):
        lines = [f"l{i}" for i in range(n)]
        covered = {s + j for s, ch in chunk_lines(lines) for j in range(len(ch))}
        assert covered == set(range(n)), n
    assert chunk_lines([]) == []
    index = [
        {"path": "pacing.py", "blob": "old", "vec": [1.0, 0.0], "text": "def getBidFloor"},
        {"path": "pacing.py", "blob": "new", "vec": [0.9, 0.1], "text": "def get_floor_price"},
        {"path": "pacing.py", "blob": "new", "vec": [0.8, 0.0], "text": "chunk 2"},
        {"path": "pacing.py", "blob": "new", "vec": [0.7, 0.0], "text": "chunk 3"},
        {"path": "caps.yaml", "blob": "c1", "vec": [0.5, 0.5], "text": "max_per_day: 3"},
    ]
    head = {"pacing.py": "new", "caps.yaml": "c1"}
    got = [c["text"] for c in retrieve(index, [1.0, 0.0], head, k=3)]
    assert got == ["def get_floor_price", "chunk 2", "max_per_day: 3"], got
    print("ok")
```

In `reindex`, also delete every chunk for the path before appending, so the index stays small.

Verify: A coverage property test over file lengths 0 to 100, a rename test where the old symbol must not come back, and a crowding test.

Pitfalls: Fixing only the loop and leaving stale chunks; filtering by timestamp instead of blob hash, which breaks on reverts; per-file caps so tight that a big file's best chunk is skipped.

Why this is likely: Ads onsite reports describe practical coding judged on clean code, and the listing asks for document indexing and retrieval pipelines; medium confidence on format.

</details>

### Agentic SDLC

<a id="q3"></a>

#### Q3. Design a PR pre-review agent that flags drift from the team's architecture decisions before a human opens the diff. Keep it useful enough that engineers do not mute it.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: AI code review in CI.

<details>
<summary>Show answer</summary>

Task: Design the pipeline from PR opened to comments posted, decide what the LLM does versus deterministic checks, and set quality targets.

What they grade: a layered pipeline; grounding in decisions and patterns; a precision-first noise budget; measurement.

Numbers: Assume 60 PRs a day, a median diff of 300 lines, and about 30,000 input tokens of context per review. That is under 2 million tokens a day of input. The latency target is under 5 minutes from push, before reviewers usually look. The quality targets are at most 3 comments per PR and a comment acceptance rate of at least 50%, where accepted means a code change or a thumbs-up within 48 hours.

Approach: (1) Deterministic first: linters, type checks, and a pattern catalog of known anti-patterns matched by regex or symbol. Netflix's performance engineering agents work this way: a cheap match in CI decides whether an LLM is called at all. (2) Retrieve the context: the ADRs and canonical patterns for touched modules from the context layer (Q1), plus callers of changed symbols. (3) The LLM reviews each hunk against the retrieved decisions and must cite the decision id and the exact line for every finding, as structured output. (4) A filter drops findings without a valid citation, findings on unchanged lines, duplicates of linter output, and anything below a confidence threshold tuned on the eval set (Q7). (5) Post as a single review with "suggested change" blocks, and let authors react with useful or not useful.

Alternatives: One big prompt with the full diff is simpler but produces long, generic reviews. Per-check specialist prompts (security, decisions, tests) cost more calls but are easier to evaluate one at a time. Start with one decision-drift checker and add specialists only when eval data says they pay off.

Risks: Noise is the main failure mode: once engineers learn to ignore it, real findings get ignored too. Treat diff text as untrusted data, because a PR can contain text that tries to steer the reviewer. The agent only comments; a human approves every merge.

Verify: Track acceptance rate, comments per PR, and escaped defects in shadow mode for two weeks before posting publicly.

Why this is likely: Netflix job listing (PR pre-review) plus a QCon San Francisco 2026 abstract from Netflix's UI Platform team describing a pre-review agent that flags drift against architecture decisions; high confidence on topic.

</details>

<a id="q4"></a>

#### Q4. An assistant wrote this webhook handler for the pre-review bot. Review it like a PR: find the defects, say whether you would merge it, and write the tests that would have caught them.

Round: AI-assisted practical. AI policy: Allowed. Type: Review. Difficulty: Medium. Topic: Reviewing AI-generated code.

```python
import hashlib
import hmac
import json


def handle_pr_event(headers, body, secret, gh, llm):
    event = json.loads(body)
    try:
        sig = headers["X-Hub-Signature-256"]
        expected = "sha256=" + hmac.new(secret, body, hashlib.sha256).hexdigest()
        if not hmac.safe_compare(sig, expected):
            return 401
    except Exception:
        pass
    pr = event["pull_request"]
    repo = event["repository"]["full_name"]
    files = gh.get(f"/repos/{repo}/pulls/{pr['number']}/files")
    prompt = "Review this diff and reply with JSON findings.\n" + "\n".join(f["patch"] for f in files)
    for finding in json.loads(llm(prompt)):
        line = map_line(finding["hunk_start"], finding["hunk"], finding["offset"])
        gh.post_review_comment(pr["number"], finding["path"], line, finding["text"])
    return 200


def map_line(new_start, hunk_lines, offset):
    line = new_start
    for i, text in enumerate(hunk_lines):
        if i == offset:
            return line
        line += 1
```

<details>
<summary>Show answer</summary>

Task: Find the planted defects, decide on merge, and add tests.

What they grade: catching the hallucinated API and the auth bypass it causes; the line-mapping off-by-one; missing edge cases; tests per defect.

Approach: Request changes. Defects, worst first. (1) `hmac.safe_compare` does not exist; the real function is `hmac.compare_digest`. The AttributeError is swallowed by `except Exception: pass`, so every request, signed or not, gets through. A hallucinated API plus a broad except turns into an auth bypass. A missing header also falls through. The handler parses the body before verifying the signature; verify first, then json.loads. (2) `map_line` counts deleted lines (starting with "-") toward new-file line numbers, so every comment below a deletion lands one line too low per deletion. (3) The files endpoint is paginated (30 per page by default, 100 max), so big PRs are only partly reviewed. Files with no `patch` key, such as binaries, raise KeyError. (4) The diff is pasted into the instruction text, so PR content can steer the reviewer, and model output goes to `json.loads` with no schema check.

How I would use the assistant here: ask it to list every external call with a link to its docs, which surfaces `safe_compare` quickly. Then ask for tests first, one per defect, and read each test before the fix. One wrong suggestion to catch: it proposed `if sig == expected`, which works but is not constant-time. Keep `compare_digest`.

```python
import hashlib
import hmac


def verify(headers, body, secret):
    sig = headers.get("X-Hub-Signature-256", "")
    expected = "sha256=" + hmac.new(secret, body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(sig, expected)


def map_line(new_start, hunk_lines, offset):
    if not 0 <= offset < len(hunk_lines) or hunk_lines[offset].startswith("-"):
        return None
    return new_start + sum(1 for t in hunk_lines[:offset] if not t.startswith("-"))


def list_files(gh, repo, number, per_page=100):
    files, page = [], 1
    while True:
        batch = gh.get(f"/repos/{repo}/pulls/{number}/files", per_page=per_page, page=page)
        files.extend(batch)
        if len(batch) < per_page:
            return files
        page += 1


if __name__ == "__main__":
    secret, body = b"s3cret", b'{"x": 1}'
    good = "sha256=" + hmac.new(secret, body, hashlib.sha256).hexdigest()
    assert verify({"X-Hub-Signature-256": good}, body, secret)
    assert not verify({"X-Hub-Signature-256": "sha256=00"}, body, secret)
    assert not verify({}, body, secret)
    hunk = [" a", "-b", "+c", " d"]
    assert [map_line(10, hunk, i) for i in range(5)] == [10, None, 11, 12, None]

    class FakeGH:
        def get(self, path, per_page, page):
            start = (page - 1) * per_page
            return [{"filename": f"f{i}"} for i in range(start, min(start + per_page, 250))]

    assert len(list_files(FakeGH(), "netflix/ads", 7)) == 250
    print("ok")
```

Verify: The tests above, plus a handler test where an unsigned request must get 401, and a schema test that drops findings with a missing path or a non-integer offset.

Pitfalls: Fixing the API name but keeping the broad except; trusting that the green tests the assistant wrote cover the defect it introduced.

Why this is likely: A 2026 Blind report of a Netflix AI-assisted onsite round graded on reviewing and testing AI output and pushing back on it; medium confidence that this team runs that round.

</details>

<a id="q5"></a>

#### Q5. With an assistant, build CI test selection and a gate for AI-generated tests in this repo. Changed modules must run every test that depends on them, directly or transitively.

Round: AI-assisted practical. AI policy: Allowed. Type: Build. Difficulty: Medium. Topic: Test generation and selection in CI.

```text
ads-pacing/
  pacing/__init__.py
  pacing/clock.py
  pacing/budget.py          imports pacing.clock
  pacing/caps.py
  pacing/throttle.py        imports pacing.budget, pacing.caps
  pacing/api.py             imports pacing.throttle
  tests/test_budget.py      imports pacing.budget
  tests/test_caps.py        imports pacing.caps
  tests/test_throttle.py    imports pacing.throttle
  tests/test_api.py         imports pacing.api
  ci/select_tests.py        TODO select_tests(changed, imports, is_test)
  ci/gen_tests.py           TODO accept_generated_test(runs_on_head, fails_on_mutants)
```

<details>
<summary>Show answer</summary>

Task: Implement both TODOs. A generated test is accepted only if it is stable and actually detects bugs.

What they grade: a delegation plan; reading the dependency direction before accepting code; a gate that rejects flaky or vacuous tests; tests for the selector.

Approach: Prompt plan. First: "Read ci/ and the import lines above, then restate what select_tests must return for a change to pacing/clock.py before writing any code." That forces the assistant to state its model. Then: "Write select_tests as a BFS over reverse imports, plus unit tests for clock, caps and an unknown module."

The wrong output to catch: the first draft walked forward imports (what the changed file imports). For clock.py that selects nothing, because clock imports nothing. Its own test passed only because it asserted the empty list. The fix is to invert the graph: test_budget depends on budget, which depends on clock.

For generated tests, accept a test only if it passes 3 of 3 runs on HEAD (no flakes) and fails on at least one mutant of the code under test, such as a flipped comparison or a dropped branch. A test that never fails is vacuous.

```python
from collections import deque


def select_tests(changed, imports, is_test):
    rev = {}
    for mod, deps in imports.items():
        for d in deps:
            rev.setdefault(d, set()).add(mod)
    seen, queue = set(changed), deque(changed)
    while queue:
        for parent in rev.get(queue.popleft(), ()):
            if parent not in seen:
                seen.add(parent)
                queue.append(parent)
    return sorted(m for m in seen if is_test(m))


def accept_generated_test(runs_on_head, fails_on_mutants):
    return len(runs_on_head) >= 3 and all(runs_on_head) and any(fails_on_mutants)


if __name__ == "__main__":
    imports = {
        "pacing.budget": ["pacing.clock"],
        "pacing.throttle": ["pacing.budget", "pacing.caps"],
        "pacing.api": ["pacing.throttle"],
        "tests.test_budget": ["pacing.budget"],
        "tests.test_caps": ["pacing.caps"],
        "tests.test_throttle": ["pacing.throttle"],
        "tests.test_api": ["pacing.api"],
    }
    is_test = lambda m: m.startswith("tests.")
    assert select_tests(["pacing.clock"], imports, is_test) == [
        "tests.test_api", "tests.test_budget", "tests.test_throttle"]
    assert select_tests(["pacing.caps"], imports, is_test) == [
        "tests.test_api", "tests.test_caps", "tests.test_throttle"]
    assert select_tests(["docs.readme"], imports, is_test) == []
    assert accept_generated_test([True] * 3, [False, True])
    assert not accept_generated_test([True, False, True], [True])
    assert not accept_generated_test([True] * 3, [False, False])
    print("ok")
```

Verify: In the diff, check that the graph is inverted, that cycles cannot loop forever (the seen set), and that unknown modules return nothing rather than everything. Add a fallback: if the import graph is older than HEAD, run the full suite.

Pitfalls: Trusting a green test that asserts the wrong expectation; accepting generated tests that only assert no exception was raised.

Why this is likely: Netflix job listing (automated test creation, CI integration) and the 2026 Blind report of an AI-assisted practical round in a real codebase; medium confidence.

</details>

<a id="q6"></a>

#### Q6. Design an agent-assisted deployment validation step for Ads services. What does the agent decide, what does code decide, and how does it fail safe?

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Deployment validation and canary analysis.

<details>
<summary>Show answer</summary>

Task: Design the post-deploy gate for services such as ad decisioning and event ingestion, including where an LLM adds value.

What they grade: a statistical judge that stays deterministic; the agent picking metrics and explaining; fail-safe defaults; Ads-specific signals.

Approach: Keep the pass or fail decision in code. Netflix open-sourced Kayenta with Google for automated canary analysis: canary and baseline clusters get equal traffic and a judge compares metric distributions with a statistical test. Use that shape. The agent adds value in three places. (1) Before the deploy, it reads the diff and the context layer and proposes extra metrics. A change to frequency cap logic should add cap-violation rate and impressions per member per day to the default latency and error metrics. (2) On a failed or marginal canary, it writes a summary that links the failing metrics, the log lines and the hunks most likely responsible. (3) After rollback, it opens a ticket with that evidence. Promotion stays with the code judge and the deploy owner.

Numbers: Ads requests are bursty around content starts, so compare canary and baseline at the same time, not before and after. Assume a 30-minute canary at 5% of traffic. If the decision service handles 20,000 requests a second, the canary sees 1,000 a second, about 1.8 million requests, enough to detect a 0.1 percentage point error rate increase over a 0.5% baseline.

Alternatives: Fixed thresholds are simple but noisy for bursty metrics. A pure LLM judge is unrepeatable and hard to audit. A statistical judge plus an LLM explainer gives repeatable decisions and useful explanations.

Risks: If the agent or the metric store is down, the gate should fall back to the default metric set and the statistical judge, and hold the deploy if those are down too. Extra metrics proposed by the agent can add false failures, so require a human to approve new metrics before they block. Revenue metrics lag by hours, so pair the fast canary with a slower guardrail check.

Verify: Replay the last 50 deploys, including known bad ones, and measure detection and false-block rates before turning on blocking.

Why this is likely: Netflix job listing (deployment validation) plus Netflix's public canary analysis work (Kayenta); medium confidence on the round.

</details>

### Evaluation

<a id="q7"></a>

#### Q7. Design the eval harness for the pre-review agent, and the hill-climbing loop you would run to improve it without fooling yourself.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Eval design and hill-climbing.

<details>
<summary>Show answer</summary>

Task: Define datasets, metrics, graders and the improvement loop for an agent that comments on PRs.

What they grade: data from real history plus seeded defects; precision and recall measured separately; a calibrated judge; held-out splits and regression gates.

Approach: Datasets. (1) Seeded defects: take 400 merged PRs and inject one known defect each (a drift from an ADR, a missing null check, a wrong cap window). The answer is known, so recall is exact and needs no judge. (2) Historical PRs with human review comments that led to changes, which are real issues. (3) Clean PRs that humans approved without comment, used to measure noise.

Metrics. Recall on seeded defects, graded by code: did a finding cite the defect's file and line within 3 lines? Precision on historical and clean PRs, graded by an LLM judge that compares each finding with the human comments and the later diff. Comments per PR, latency and cost.

Judge calibration. Have two engineers label 150 findings. Accept the judge only if its agreement with them is close to their agreement with each other, for example a Cohen's kappa of at least 0.6. Recalibrate when the model or prompt changes.

Hill-climbing loop. Split the data 60/20/20 into dev, validation and a locked test set. Change one thing at a time (retrieval depth, prompt, filter threshold), iterate on dev, confirm on validation, and only touch the test set before a release. Keep a failure log of misses, tagged by cause, and pick the next change from the largest bucket. Watch for overfitting: if dev improves 5 points and validation 0, revert.

Alternatives: Only human ratings are accurate but too slow to run on every change. Only an LLM judge is fast but drifts. Code graders where the answer is known plus a calibrated judge for the rest gives fast feedback that stays honest.

Risks: Seeded defects are easier than real ones, so report the two sources separately. Leakage: if retrieval can see the fix commit, recall is inflated, so freeze the index at each PR's base commit.

Verify: Gate releases on no drop in seeded recall, at most 3 comments per PR, and a paired significance test (Q8).

Why this is likely: Netflix job listing (evaluation frameworks, hill-climbing methodologies); high confidence on topic for an AI/ML role.

</details>

<a id="q8"></a>

#### Q8. Pre-review v2 catches 224 of 400 seeded defects and v1 catches 212. v2 caught 31 that v1 missed and missed 19 that v1 caught. Do you ship v2?

Round: Onsite system design. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: Comparing agent versions statistically.

<details>
<summary>Show answer</summary>

Task: Decide whether a 3-point gain (53.0% to 56.0%) is real, with a test that matches the paired design.

What they grade: using a paired test, not two independent proportions; a confidence interval; a decision rule that also weighs cost and noise.

Approach: Both versions ran on the same 400 items, so the data is paired. Only the discordant items carry information: 31 where v2 wins and 19 where v1 wins. McNemar's exact test asks whether a 31 to 19 split is likely if both versions were equally good, which is a binomial with n = 50 and p = 0.5. A paired bootstrap of the per-item difference gives an interval for the gain.

```python
import random
from math import comb


def mcnemar_exact(b, c):
    n, k = b + c, min(b, c)
    tail = sum(comb(n, i) for i in range(k + 1)) / 2 ** n
    return min(1.0, 2 * tail)


def paired_bootstrap(pairs, iters=5000, seed=7):
    rng = random.Random(seed)
    diffs = sorted(
        sum(b - a for a, b in (rng.choice(pairs) for _ in pairs)) / len(pairs)
        for _ in range(iters))
    return diffs[int(0.025 * iters)], diffs[int(0.975 * iters) - 1]


if __name__ == "__main__":
    pairs = [(1, 1)] * 193 + [(0, 1)] * 31 + [(1, 0)] * 19 + [(0, 0)] * 157
    assert sum(a for a, _ in pairs) == 212 and sum(b for _, b in pairs) == 224
    p = mcnemar_exact(31, 19)
    lo, hi = paired_bootstrap(pairs)
    print(f"p={p:.3f} ci=({lo:+.3f}, {hi:+.3f})")
    assert 0.11 < p < 0.13
    assert lo < 0 < hi
    assert mcnemar_exact(62, 38) < 0.05
    print("ok")
```

The p-value is about 0.12 and the 95% interval for the gain runs from about -0.5 to +6.5 points. The data does not show v2 is better. If the same 31 to 19 ratio held over twice as many items (62 to 38), p would be about 0.02.

Decision: treat the recall gain as unproven. Ship v2 if it is no worse on precision, comments per PR, and cost, and the change has another reason to exist (simpler prompt, cheaper model). Otherwise grow the seeded set to about 800 items and rerun. Also read the 19 regressions: if they cluster in one defect class, such as cap window bugs, that is a real regression hidden in the average.

Verify: Assert the counts match the table before testing, and check the bootstrap with a fixed seed so results reproduce.

Pitfalls: A two-proportion z-test that ignores pairing; picking the best of many runs, since sampling at nonzero temperature varies run to run, so average several runs per item.

Why this is likely: Netflix job listing (evaluation frameworks, hill-climbing methodologies), the same source as Q7; medium confidence on this exact format.

</details>

### AI operations

<a id="q9"></a>

#### Q9. Debug this incident triage agent trace: it blamed the wrong change for an ad decisioning latency alert. Find the root causes, fix the system, and design the eval.

Round: Onsite system design. AI policy: Unknown. Type: Trace debug. Difficulty: Medium. Topic: Incident triage agent failure analysis.

```json
{
  "alert": {"service": "ads-decisioning", "metric": "p99_latency_ms",
            "value": 410, "slo": 120, "fired_at": "14:05Z"},
  "steps": [
    {"n": 1, "tool": "list_deploys", "args": {"since": "13:35Z"},
     "result": [{"service": "ads-reporting", "at": "14:02Z", "id": "d-881"},
                {"service": "ads-targeting", "at": "13:40Z", "id": "d-877"}]},
    {"n": 2, "tool": "get_metric",
     "args": {"service": "ads-reporting", "metric": "error_rate"},
     "result": {"before": 0.002, "after": 0.004}},
    {"n": 3, "tool": "search_runbook", "args": {"q": "latency after deploy"},
     "result": "Roll back the most recent deploy."},
    {"n": 4, "tool": "final",
     "args": {"root_cause": "d-881 (ads-reporting)", "confidence": 0.86,
              "action": "rollback d-881"}}
  ],
  "unseen": {"flag_change": {"service": "ads-decisioning", "flag": "cap_check_v2",
                             "at": "13:58Z", "rollout": "50%"}}
}
```

<details>
<summary>Show answer</summary>

Task: Explain why the agent picked d-881, fix the tools and the reasoning path, and prove the fix.

What they grade: noticing the agent never looked at the alerting service; the missing change source; dependency direction; a replay eval.

Approach: Read each step against the alert. Three problems. (1) Coverage gap: `list_deploys` only returns deploys, so the flag change to ads-decisioning at 13:58Z, which ramped a new cap check to 50% of traffic, was invisible. Most incidents follow some change, and in many systems flags and config cause as many as deploys. (2) Wrong direction: ads-reporting consumes events downstream of decisioning, so its deploy cannot slow decisioning's request path. The agent ranked by recency without checking the dependency graph. (3) Wrong evidence: step 2 checked reporting's error rate, not decisioning's latency. A rise from 0.2% to 0.4% in a different metric on a different service does not support the conclusion, yet the agent reported 0.86 confidence and proposed a rollback.

Fixes. Replace `list_deploys` with `list_changes(service, since)`, which returns deploys, flag changes and config pushes for the alerting service and its dependencies (services it calls), resolved from the service graph. Require at least one piece of evidence on the alerting metric itself before naming a root cause. Rank changes by metric shift around each change time (Q10). Make the confidence number come from the evidence, for example zero if no evidence touches the alert metric. Rollback stays a suggestion that a human confirms.

Verify: Turn this incident into a replay case with recorded tool outputs. Add variations: a flag change only, a config push only, a downstream deploy that is a decoy, and a case where the dependency's deploy really is the cause. Grade with code: did the named change match the labeled cause, did any cited evidence touch the alert metric, and did the agent ever call a write tool. Track top-1 accuracy and the rate of confident wrong answers, which matter more than unconfident ones.

Pitfalls: Fixing the prompt ("consider flags") without giving the agent a tool that returns flags; accepting the agent's confidence number as calibrated.

Why this is likely: Netflix job listing (incident triage, log and metric correlation, root cause analysis); high confidence on topic, medium on format.

</details>

<a id="q10"></a>

#### Q10. Build the change-correlation step for incident triage: rank recent changes by how much the alerting metric moved after each one, considering only the alerting service and its dependencies.

Round: Onsite coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Log and metric correlation.

```text
Spec of existing inputs
changes:  list of (minute, service, change_id), minute = index into the metric series
series:   list of floats, alerting metric per minute for the alerting service
calls:    dict service -> list of services it calls (its dependencies)
service:  the alerting service
window:   minutes to compare before and after each change (default 10)
Return:   change ids with a positive shift, largest shift first; ties go to the later change
Ignore:   changes in services the alerting service does not call, directly or transitively
Ignore:   changes too close to the series edges to have a full before window
Example:  flag change in ads-decisioning at minute 58 should rank first
Example:  deploy of ads-reporting (a consumer, not a dependency) must not appear
```

<details>
<summary>Show answer</summary>

Task: Implement `rank_suspects` from the spec, with tests that mirror the Q9 incident.

What they grade: a transitive dependency closure; a clean before and after comparison; edge handling; a discussion of better statistics.

Approach: First compute the set of services that can affect the alerting service: itself plus everything it calls, transitively, by BFS. Then for each change in that set, compare the mean of the metric in the window after the change with the mean in the window before. Keep positive shifts and sort. This is a simple step-change detector. It is deterministic and cheap, so it runs before any LLM reasoning and hands the agent a ranked shortlist with numbers.

```python
from collections import deque


def dependencies(calls, service):
    seen, queue = {service}, deque([service])
    while queue:
        for d in calls.get(queue.popleft(), ()):
            if d not in seen:
                seen.add(d)
                queue.append(d)
    return seen


def rank_suspects(changes, series, calls, service, window=10):
    allowed = dependencies(calls, service)
    scored = []
    for minute, svc, cid in changes:
        if svc not in allowed or minute < window or minute >= len(series):
            continue
        before = series[minute - window:minute]
        after = series[minute:minute + window]
        shift = sum(after) / len(after) - sum(before) / len(before)
        if shift > 0:
            scored.append((shift, minute, cid))
    scored.sort(key=lambda s: (-s[0], -s[1]))
    return [cid for _, _, cid in scored]


if __name__ == "__main__":
    series = [100.0] * 58 + [400.0] * 20
    calls = {"ads-decisioning": ["ads-targeting", "ads-caps"], "ads-targeting": ["catalog"]}
    changes = [
        (40, "ads-targeting", "d-877"),
        (58, "ads-decisioning", "flag:cap_check_v2"),
        (62, "ads-reporting", "d-881"),
        (3, "catalog", "d-870"),
    ]
    assert rank_suspects(changes, series, calls, "ads-decisioning") == ["flag:cap_check_v2"]
    assert "catalog" in dependencies(calls, "ads-decisioning")
    assert rank_suspects([], series, calls, "ads-decisioning") == []
    print("ok")
```

In the test, d-877 at minute 40 sees no shift, d-881 is excluded because reporting is not a dependency, and d-870 at minute 3 lacks a full before window.

Alternatives: Mean difference is easy to explain but sensitive to spikes. A median shift or a rank test (Mann-Whitney) is more robust for latency. A change-point detector over the whole series finds when the shift happened, then matches it to the nearest change; it is better with many overlapping changes.

Verify: Overlapping changes within one window, a change at the last minute, a dependency cycle in `calls`, and a negative shift (an improvement) that must not be returned.

Pitfalls: Using wall-clock strings instead of aligned minute indexes; skipping the closure and only checking direct dependencies.

Why this is likely: Netflix job listing (log and metric correlation), plus Ads onsite reports of practical coding with BFS and tradeoff discussion; medium confidence.

</details>

### Tool calling and orchestration

<a id="q11"></a>

#### Q11. This agent loop hangs some sessions and hides tool failures from the model. Find the bugs and fix the loop.

Round: Onsite coding. AI policy: Not allowed. Type: Debug. Difficulty: Medium. Topic: Tool calling and structured outputs.

```python
import json


def run_agent(llm, tools, task):
    messages = [{"role": "user", "content": task}]
    while True:
        out = llm(messages)
        if out.get("final"):
            return out["final"]
        call = out["tool_call"]
        try:
            result = tools[call["name"]](**json.loads(call["args"]))
        except Exception:
            result = ""
        messages.append({"role": "tool", "content": str(result)})
```

<details>
<summary>Show answer</summary>

Task: Name each bug, explain the symptom it causes, and rewrite the loop with tests.

What they grade: the missing budget; swallowed errors; a message history the model can follow; tests with a scripted fake model.

Approach: Bugs. (1) `while True` with no step budget: a model that keeps calling tools never stops, which explains the hung sessions. (2) Every exception becomes an empty string: an unknown tool name, malformed JSON arguments, and wrong keyword arguments all look like "no results" to the model, so it retries the same call, which feeds bug 1. (3) The assistant's own tool call is never appended, and tool results carry no call id, so the model cannot tell which result belongs to which call. (4) `out.get("final")` treats an empty-string final answer as missing. (5) A blanket except also hides real faults, such as a timeout in a write tool, that the orchestrator should see.

Fix: a budget, typed and structured errors the model can act on, a complete history, and a status the caller can check.

```python
import json


class ToolError(Exception):
    pass


def run_agent(llm, tools, task, max_steps=8):
    messages = [{"role": "user", "content": task}]
    for _ in range(max_steps):
        out = llm(messages)
        if "final" in out:
            return {"status": "done", "answer": out["final"], "messages": messages}
        call = out["tool_call"]
        messages.append({"role": "assistant", "tool_call": call})
        try:
            if call["name"] not in tools:
                raise ToolError(f"unknown tool {call['name']}")
            data = tools[call["name"]](**json.loads(call["args"]))
            result = {"ok": True, "data": data}
        except (ToolError, json.JSONDecodeError, TypeError) as e:
            result = {"ok": False, "error": f"{type(e).__name__}: {e}"}
        messages.append({"role": "tool", "id": call["id"], "content": json.dumps(result)})
    return {"status": "budget_exhausted", "answer": None, "messages": messages}


if __name__ == "__main__":
    tools = {"get_cap": lambda line_item: {"line_item": line_item, "cap": 3}}
    script = iter([
        {"tool_call": {"id": "c1", "name": "get_cap", "args": "{line_item: 7}"}},
        {"tool_call": {"id": "c2", "name": "get_cap", "args": '{"line_item": 7}'}},
        {"final": "cap is 3"},
    ])
    res = run_agent(lambda m: next(script), tools, "what is the cap for 7?")
    assert res["status"] == "done" and res["answer"] == "cap is 3"
    first = json.loads(res["messages"][2]["content"])
    assert first["ok"] is False and "JSONDecodeError" in first["error"]
    loop = lambda m: {"tool_call": {"id": "x", "name": "nope", "args": "{}"}}
    res = run_agent(loop, tools, "loop", max_steps=4)
    assert res["status"] == "budget_exhausted" and len(res["messages"]) == 9
    print("ok")
```

Unexpected exceptions now propagate, so a timeout reaches the orchestrator's retry policy instead of the model.

Verify: The scripted tests above; add a loop detector that stops after the same call with the same arguments fails twice.

Pitfalls: Raising max_steps as the only fix; passing raw stack traces to the model, which wastes tokens and can leak internals.

Why this is likely: Netflix job listing (tool use and function calling, structured outputs, agent architectures) and Ads onsite reports of practical coding; medium confidence on format.

</details>

<a id="q12"></a>

#### Q12. Design multi-agent orchestration where parallel agents implement, test and document a change as one coordinated workflow. How do you keep it from producing slop?

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Multi-agent orchestration for the SDLC.

<details>
<summary>Show answer</summary>

Task: Design the orchestrator, the agents' boundaries, how work merges, and where humans stay accountable.

What they grade: isolation and merge strategy; independence between implementer and tester; budgets and stop rules; clear human ownership.

Approach: A planner turns a ticket into a task graph with acceptance criteria written as checkable statements. The human owner approves this plan before any code is written, because a wrong plan executed in parallel wastes the most. Each agent works in its own git worktree on its own branch, so agents never edit the same checkout.

Roles. The implementer writes code against the plan. The test agent writes tests from the acceptance criteria and the public interface only, without reading the implementation, so its tests are an independent check rather than a copy of the code's assumptions. The doc agent updates ADR references and runbooks after the code stabilizes. The orchestrator merges the test branch into the implementation branch, runs the suite, and loops failures back to the implementer with the failing output, up to a budget.

Numbers: Assume a medium change takes an engineer 6 hours. Sequential agent runs might take 40 minutes, and parallel runs about 15. But each implement and fix cycle costs perhaps 200,000 tokens, so cap at 3 fix rounds and escalate after that. Parallelism pays off only when the subtasks touch different files; if the planner predicts more than 20% file overlap, run sequentially.

Alternatives: One agent doing everything in sequence is simpler and avoids merge conflicts but lets the same blind spot shape both code and tests. Parallel agents with independent tests catch more defects for a higher token cost.

Risks: Slop at scale. Gates against it: the pre-review agent (Q3) runs on every branch; changed lines need tests; a human reviewer owns the merge and is named in the PR. Track defects found after merge in agent-authored changes against human-authored ones, and slow the rollout if agent changes are worse.

Verify: Pilot on low-risk tickets, compare cycle time and post-merge defects against a matched set of human-only tickets for a month.

Why this is likely: Netflix job listing (multi-agent orchestration for implementation, testing and documentation; avoiding slop and accountability gaps); high confidence on topic.

</details>

### Ads platform domain

<a id="q13"></a>

#### Q13. Design an ads frequency cap service: a member sees a line item at most N times per day and per week, checked on every ad decision.

Round: Onsite system design. AI policy: Unknown. Type: Design. Difficulty: Hard. Topic: Ads frequency capping.

<details>
<summary>Show answer</summary>

Task: Design the read path in ad decisioning and the write path from impressions, with consistency and scale tradeoffs.

What they grade: separating decision-time checks from impression counting; windowed counters; memory and latency math; a stance on over- versus under-delivery.

Numbers: Assume 50,000 ad decisions per second at peak, each checking about 20 candidate line items, so about 1 million counter reads per second. The check budget inside decisioning is about 5 ms. Assume 300 million active (member profile, line item) pairs in a 7-day window. Seven daily buckets of 2 bytes plus a 16-byte key is about 30 bytes per pair, so about 9 GB, or 27 GB with three replicas.

Approach: Read path: decisioning fetches all of the profile's counters in one batched call to an in-memory key-value store keyed by profile. It then filters candidates whose daily or weekly sum reaches the cap. Write path: confirmed impression events (the ad actually rendered) flow through a stream to a counter updater that increments the current day's bucket. Old buckets drop on write, and keys carry a TTL.

Decision-to-impression lag causes most cap violations. A member watching back to back can get two decisions before the first impression is counted. Add a short-lived reservation: on decision, write a provisional count with a TTL of a few minutes, and replace it with the confirmed count when the impression arrives. If the impression never arrives, the reservation expires.

Alternatives: Counting at decision time is simple but overcounts ads that never played, which under-delivers paid campaigns. Counting only at impression time undercounts during the lag and over-exposes members. Reservations plus impression counting cost a second write per decision and give tight caps both ways.

Risks: If the counter store is unreachable, decide whether to fail open (serve and risk over-exposure) or fail closed (skip capped line items and lose revenue). Default to a brief fail-open with an alert; legally sensitive caps fail closed. Define each campaign's day boundary time zone.

Verify: Replay a binge session in a load test and measure the cap violation rate, the p99 check latency, and under-delivery against the booked plan.

Why this is likely: 1point3acres' 2026 Q2 Netflix trend report lists ads frequency cap as a top system design topic, and a 2026 Blind Netflix L5 report names it; high confidence for Ads loops, medium for this role.

</details>

<a id="q14"></a>

#### Q14. Model the ads demand hierarchy and write the pacing query that flags line items delivering under 90% of their expected impressions to date.

Round: Onsite data modeling and business impact. AI policy: Unknown. Type: Concept plus code. Difficulty: Medium. Topic: Ads data modeling.

<details>
<summary>Show answer</summary>

Task: Define tables for advertiser, campaign, line item, creative and daily impressions, then flag under-pacing line items.

What they grade: entities and cardinalities that match how ads are sold; constraints; money as integers; a query that a business owner can act on.

Approach: The hierarchy is advertiser to campaign (budget and flight dates) to line item (price, booked impressions, frequency cap) to creative (asset and review status). Delivery facts land in an impression table that is aggregated daily, because the raw event stream is far too large to query for pacing. Store money as integer micros to avoid float rounding. Linear pacing expects booked × elapsed days ÷ total days.

```python
import sqlite3

SCHEMA = """
CREATE TABLE advertiser (id INTEGER PRIMARY KEY, name TEXT NOT NULL);
CREATE TABLE campaign (id INTEGER PRIMARY KEY,
  advertiser_id INTEGER NOT NULL REFERENCES advertiser(id),
  budget_micros INTEGER NOT NULL, start_date TEXT NOT NULL, end_date TEXT NOT NULL,
  CHECK (end_date >= start_date));
CREATE TABLE line_item (id INTEGER PRIMARY KEY,
  campaign_id INTEGER NOT NULL REFERENCES campaign(id),
  cpm_micros INTEGER NOT NULL, booked_impressions INTEGER NOT NULL,
  freq_cap INTEGER, freq_window_hours INTEGER);
CREATE TABLE creative (id INTEGER PRIMARY KEY,
  line_item_id INTEGER NOT NULL REFERENCES line_item(id), duration_s INTEGER NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('pending', 'approved', 'rejected')));
CREATE TABLE impression_daily (line_item_id INTEGER NOT NULL REFERENCES line_item(id),
  day TEXT NOT NULL, impressions INTEGER NOT NULL, PRIMARY KEY (line_item_id, day));
"""

PACING = """
SELECT li.id,
       COALESCE(SUM(d.impressions), 0) AS delivered,
       li.booked_impressions * (julianday(:today) - julianday(c.start_date) + 1)
         / (julianday(c.end_date) - julianday(c.start_date) + 1) AS expected
FROM line_item li
JOIN campaign c ON c.id = li.campaign_id
LEFT JOIN impression_daily d ON d.line_item_id = li.id AND d.day <= :today
WHERE :today BETWEEN c.start_date AND c.end_date
GROUP BY li.id
HAVING delivered < 0.9 * expected
ORDER BY delivered * 1.0 / expected
"""

if __name__ == "__main__":
    db = sqlite3.connect(":memory:")
    db.executescript(SCHEMA)
    db.execute("INSERT INTO advertiser VALUES (1, 'Acme')")
    db.execute("INSERT INTO campaign VALUES (1, 1, 50000000000, '2026-01-01', '2026-01-10')")
    db.executemany("INSERT INTO line_item VALUES (?, 1, 30000000, 1000, 3, 24)", [(1,), (2,), (3,)])
    rows = [(1, "2026-01-0%d" % d, 96) for d in range(1, 6)]
    rows += [(2, "2026-01-0%d" % d, 60) for d in range(1, 6)]
    db.executemany("INSERT INTO impression_daily VALUES (?, ?, ?)", rows)
    flagged = db.execute(PACING, {"today": "2026-01-05"}).fetchall()
    assert [(r[0], r[1], round(r[2])) for r in flagged] == [(3, 0, 500), (2, 300, 500)], flagged
    print("ok")
```

Line item 1 delivered 480 of an expected 500 (96%), so it is fine. Line item 2 is at 60% and line item 3 has no delivery, which usually means no approved creative. Join `creative` to tell the business owner why.

Alternatives: Linear pacing is easy to explain but wrong for viewing patterns that peak on weekends. Forecast-weighted pacing uses expected inventory per day and flags fewer false alarms. Start linear, then switch once a forecast exists.

Pitfalls: Float money; an inner join that hides line items with zero delivery; ignoring flight dates.

Why this is likely: 1point3acres' 2026 Q2 trend report lists ads data modeling (advertiser, campaign, ad group, ad, impression) as the most frequent Netflix topic, and a 2026 Blind Ads report describes a data modeling and business impact round; high confidence for Ads loops.

</details>

### Coding fundamentals

<a id="q15"></a>

#### Q15. (phone screen, no AI) Agent pipeline jobs have dependencies. Return the minimum number of parallel stages to run all n jobs, or -1 if the dependencies contain a cycle. Then extend it to jobs with durations.

Round: Technical phone screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Topological sort.

Related practice: [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) (Medium), [Parallel Courses III](https://leetcode.com/problems/parallel-courses-iii/) (Hard)

<details>
<summary>Show answer</summary>

Task: Given jobs 1 to n and pairs (before, after), return the stage count; then return the minimum total time when each job has a duration and unlimited jobs run in parallel.

What they grade: Kahn's algorithm by levels; cycle detection by count; the longest-path extension; clean code and complexity.

Approach: Count incoming edges. All jobs with none form stage 1. Process the queue one level at a time: each finished job decrements its dependents, and those reaching zero join the next level. If fewer than n jobs were processed, a cycle blocked the rest. For durations, the answer is the longest path weighted by duration: each job finishes at its own duration plus the latest finish among its prerequisites, computed in topological order.

```python
from collections import deque


def _graph(n, deps):
    out = {i: [] for i in range(1, n + 1)}
    indeg = {i: 0 for i in range(1, n + 1)}
    for a, b in deps:
        out[a].append(b)
        indeg[b] += 1
    return out, indeg


def min_stages(n, deps):
    out, indeg = _graph(n, deps)
    queue = deque(i for i in indeg if indeg[i] == 0)
    stages = done = 0
    while queue:
        stages += 1
        for _ in range(len(queue)):
            u = queue.popleft()
            done += 1
            for v in out[u]:
                indeg[v] -= 1
                if indeg[v] == 0:
                    queue.append(v)
    return stages if done == n else -1


def min_time(n, deps, duration):
    out, indeg = _graph(n, deps)
    finish = {i: duration[i - 1] for i in range(1, n + 1)}
    queue = deque(i for i in indeg if indeg[i] == 0)
    done = 0
    while queue:
        u = queue.popleft()
        done += 1
        for v in out[u]:
            finish[v] = max(finish[v], finish[u] + duration[v - 1])
            indeg[v] -= 1
            if indeg[v] == 0:
                queue.append(v)
    return max(finish.values()) if done == n else -1


if __name__ == "__main__":
    assert min_stages(3, [(1, 3), (2, 3)]) == 2
    assert min_stages(3, [(1, 2), (2, 3), (3, 1)]) == -1
    assert min_stages(4, []) == 1
    assert min_time(3, [(1, 3), (2, 3)], [3, 2, 5]) == 8
    assert min_time(5, [(1, 5), (2, 5), (3, 5), (3, 4), (4, 5)], [1, 2, 3, 4, 5]) == 12
    print("ok")
```

Complexity: O(n + e) time and space for both.

Verify: No dependencies (one stage), a single chain (n stages), a cycle, duplicate edges, and a job with no edges.

Pitfalls: DFS recursion depth on long chains in Python; counting levels by popping one node at a time; returning a partial answer when a cycle exists.

Follow-ups: At most k jobs per stage (it becomes much harder; explain why greedy fails); a job that fails and is retried.

Why this is likely: Blind 2026 Netflix Ads reports describe Parallel Courses style phone screens, 1point3acres' 2026 Q2 trend report lists topological sort as the top Ads coding topic, and Course Schedule II and Parallel Courses appear in the liquidslr Netflix list; high confidence for Ads screens.

</details>

<a id="q16"></a>

#### Q16. (onsite coding, no AI) Build a cache for LLM responses with a per-entry TTL and a total weight limit (token count). get returns live entries only; put evicts expired entries first, then least recently used ones.

Round: Onsite coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Cache design with expiry and weights.

Related practice: [Cache With Time Limit](https://leetcode.com/problems/cache-with-time-limit/) (Medium), [LRU Cache](https://leetcode.com/problems/lru-cache/) (Medium)

<details>
<summary>Show answer</summary>

Task: Implement `get(key)` and `put(key, value, weight, ttl)` with a capacity in total weight, an injectable clock, and tests.

What they grade: correct weight accounting on overwrite and eviction; lazy expiry; recency updates on get; complexity and follow-ups.

Approach: An `OrderedDict` gives LRU order with O(1) moves. Each entry stores value, weight and expiry time. `get` drops an expired entry and otherwise moves it to the end. `put` rejects items heavier than the whole cache, removes any old version of the key (subtracting its weight), sweeps expired entries, then evicts from the front until the new item fits.

```python
from collections import OrderedDict


class WeightedTTLCache:
    def __init__(self, capacity, clock):
        self.capacity, self.clock = capacity, clock
        self.data = OrderedDict()
        self.used = 0

    def _drop(self, key):
        _, weight, _ = self.data.pop(key)
        self.used -= weight

    def get(self, key):
        item = self.data.get(key)
        if item is None:
            return None
        if item[2] <= self.clock():
            self._drop(key)
            return None
        self.data.move_to_end(key)
        return item[0]

    def put(self, key, value, weight, ttl):
        if weight > self.capacity:
            return False
        if key in self.data:
            self._drop(key)
        now = self.clock()
        for k in [k for k, (_, _, exp) in self.data.items() if exp <= now]:
            self._drop(k)
        while self.used + weight > self.capacity:
            self._drop(next(iter(self.data)))
        self.data[key] = (value, weight, now + ttl)
        self.used += weight
        return True


if __name__ == "__main__":
    t = [0.0]
    c = WeightedTTLCache(100, clock=lambda: t[0])
    assert c.put("a", "A", 40, ttl=10) and c.put("b", "B", 40, ttl=100)
    assert c.get("a") == "A"
    assert c.put("c", "C", 40, ttl=100)
    assert c.get("b") is None and c.get("a") == "A" and c.used == 80
    t[0] = 11
    assert c.get("a") is None and c.used == 40
    assert c.put("c", "C2", 90, ttl=5) and c.used == 90 and c.get("c") == "C2"
    assert not c.put("huge", "X", 101, ttl=5)
    print("ok")
```

Complexity: get is O(1). put is O(n) because of the expiry sweep. Alternatives: a min-heap of expiry times makes the sweep O(log n) per removed entry at the cost of stale heap entries after overwrites; a background sweeper keeps put fast but needs locking. For a prompt cache with thousands of entries, the simple sweep is fine; past about 100,000 entries, use the heap.

Verify: Overwrite with a different weight, an item exactly at capacity, expiry at the exact boundary, and eviction order after a get.

Pitfalls: Forgetting to subtract the old weight on overwrite; using wall-clock time, which jumps, instead of a monotonic clock.

Why this is likely: Cache With Time Limit is the top item in the liquidslr Netflix six-month list, and a 2026 Blind Netflix L5 onsite report describes a weighted cache with weight-based eviction; high confidence for the format.

</details>

### Adoption and judgment

<a id="q17"></a>

#### Q17. Tell me how you got a skeptical team to adopt AI tooling, and how you check AI output before it reaches production.

Round: Second onsite round. AI policy: Unknown. Type: Behavioral. Difficulty: Medium. Topic: Driving AI adoption with quality.

<details>
<summary>Show answer</summary>

Task: Give one real story with numbers, then your personal routine for verifying AI output.

What they grade: starting from the team's pain, not the tool; trust built through pairing; measured results; candor about what did not work; a concrete checking routine.

Approach: Structure the story in four parts. Context: the team, its biggest time sink, and why people were skeptical (a bad earlier tool, fear of noisy reviews). Action: pick one painful workflow, pair with its most skeptical owner, and ship a narrow tool that saves them time in week one. For example, triage summaries for the on-call before automating any fixes. Run it in shadow mode, publish its misses openly, and let engineers turn it off per repo. Result: numbers, such as median time to first triage note falling from 40 to 10 minutes, or 70% of engineers using the assistant weekly after a quarter. Say what failed too: one workflow you dropped because its precision never got above 40%.

Then the checking routine, which Netflix interviewers in AI-assisted rounds reportedly probe ("when and how you push back on the AI"). Before prompting, I write down the expected behavior and the tests that would prove it. I ask the model to plan before coding and reject plans that touch files I did not expect. In the diff, I check every external API and library call against its docs, because invented functions are the most common failure. I check boundaries and error paths, and I run the tests myself. Give one example where you caught a wrong output, such as an assistant that swallowed an exception and made an auth check pass (Q4).

Verify: Prepare the numbers in advance and be ready for "what would you do differently" and "how did you measure that".

Pitfalls: Leading with tool names; claiming adoption without a metric; describing AI as always right or always wrong. Netflix's culture memo values candor and context over control, so show how you gave engineers the context to choose the tool rather than mandating it.

Why this is likely: Netflix job listing (drive team-wide AI adoption through pairing, architecture reviews and demonstrations) and the Netflix Tech Blog description of a second round on partnering and non-technical skills; high confidence.

</details>

## Prep checklist

- [ ] Sketch a context layer end to end: sources, chunking, hybrid retrieval, freshness by blob hash, and an MCP interface.
- [ ] Practice one pre-review agent design with a noise budget, citations, and shadow-mode rollout.
- [ ] Write an eval plan with seeded defects, a calibrated LLM judge, and dev, validation and test splits.
- [ ] Run McNemar and a paired bootstrap by hand on a small agent comparison.
- [ ] Debug two agent traces aloud: one with a wrong id, one with a missing change source.
- [ ] Review AI-generated code under a timer and write one test per defect before fixing it.
- [ ] Solve Course Schedule II and a topological sort with durations in under 25 minutes, with no AI.
- [ ] Build a TTL plus weight cache from scratch and explain its complexity and follow-ups.
- [ ] Prepare an ads frequency cap design with memory and latency numbers.
- [ ] Model the advertiser, campaign, line item, creative and impression tables and write a pacing query.
- [ ] Prepare one adoption story with numbers and a personal AI checking routine.
- [ ] Read the Netflix culture memo and pick one part you would push back on.

## Sources

- Netflix Tech Blog, Demystifying Interviewing for Backend Engineers @ Netflix: https://netflixtechblog.com/demystifying-interviewing-for-backend-engineers-netflix-aceb26a83495
- QCon San Francisco 2026, How Netflix Retrofitted Its SDLC for AI Coding Agents: https://qconsf.com/presentation/nov2026/how-netflix-retrofitted-its-sdlc-ai-coding-agents
- ByteByteGo, How Netflix Uses AI Agents for Performance Engineering: https://blog.bytebytego.com/p/how-netflix-uses-ai-agents-for-performance
- ZenML LLMOps database, Netflix AI agents for performance engineering: https://www.zenml.io/llmops-database/ai-agents-for-automated-performance-engineering-and-code-optimization
- Blind, Netflix AI Assisted Interview: https://www.teamblind.com/post/netflix-ai-assisted-interview-7o1ukyfe
- Blind, LeetCode vs AI-Assisted Coding by Company megathread: https://www.teamblind.com/post/megathread-leetcode-vs-ai-assisted-coding-by-company-7ovjpqyf
- Blind, Netflix Ads Onsite: https://www.teamblind.com/post/netflix-ads-onsite-v4jbp830
- Blind, Netflix onsite interview - pls help: https://www.teamblind.com/post/netflix-onsite-interview-pls-help-k0gba5he
- Blind, Netflix Ads Phone Screen: https://www.teamblind.com/post/netflix-ads-phone-screen-pu35vfpn
- Blind, Netflix Onsite-L5: https://www.teamblind.com/post/netflix-onsite-l5-rxdq5p26
- Blind, Netflix L6 phone screen - system design: https://www.teamblind.com/post/netflix-l6-phone-screen-system-design-sgv2zg2u
- Blind, Finally on schedule for a Netflix interview: https://www.teamblind.com/post/finally-on-schedule-for-a-netflix-interview-c0mi43oc
- Reddit r/leetcode, Netflix and Google interview experience: https://www.reddit.com/r/leetcode/comments/1u8luln/netflix_and_google_interview_experience_offer/
- 1point3acres, Netflix Ads full loop thread: https://www.1point3acres.com/bbs/thread-1164299-1-1.html
- 1point3acres, Netflix Interview Trend Analysis (2026 Q2): https://www.1point3acres.com/interview/guides/interview-trend-netflix-2026q2
- Hello Interview, System Design Lessons from Netflix's Ad Tracking Launch: https://hellointerview.substack.com/p/system-design-lessons-from-netflixs
- Netflix newsroom, Netflix's Third Season of Ads: https://about.netflix.com/news/netflix-third-season-of-ads-and-a-look-ahead-at-whats-next
- Kayenta, automated canary analysis: https://github.com/spinnaker/kayenta
- liquidslr, LeetCode company-wise problems (Netflix): https://github.com/liquidslr/leetcode-company-wise-problems/tree/main/Netflix
