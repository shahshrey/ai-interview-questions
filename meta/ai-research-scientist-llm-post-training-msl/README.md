# Meta: AI Research Scientist, LLM Post-Training (Meta Superintelligence Labs)

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Meta says schedules are personalized per role and the recruiter is the source of truth, and recent candidate summaries say Superintelligence Labs research scientist loops use a different round mix from software engineering. No report specific to the Modeling and ToolCalling team was found, so the loop below combines Meta's hiring page, recent Meta research scientist reports and recent lab reports.

1. Recruiter screen. About 30 minutes, phone or video. Background, research interests, team fit. AI: not applicable.
2. Online assessment (some roles only). About 70 minutes, CodeSignal style, algorithm problems. Low confidence for this role, so ask the recruiter. AI: not allowed.
3. Technical or research screen. About 45 minutes in CoderPad. One recent research scientist report covers experiment design, ML fundamentals and coding in a single screen. The coding part is either two Meta-tagged algorithm problems or, on some newer screens, a talk about an AI system you built followed by AI-assisted coding. AI: not allowed in the classic variant; provided (CoderPad assistant) in the newer one.
4. Onsite classic coding. 45 minutes, usually two medium problems in 35 to 40 minutes. A recent research scientist report names a BFS maze and lowest common ancestor. AI: not allowed.
5. Onsite AI-enabled coding. About 60 minutes, a multi-file CoderPad project: fix a bug, write your own tests, then extend. The built-in assistant has a model picker (Claude, ChatGPT, Gemini, Meta models), and you are graded on task breakdown, verification and explaining the AI's output. AI: provided (CoderPad assistant); outside AI tools not allowed.
6. Research or ML design. 45 to 60 minutes in CoderPad with Mermaid. A lab report describes a research design round on building a coding LLM (framing, data, training); for this team expect post-training design for tool-use or search agents: data, reward, RL environment and evals. AI: provided (CoderPad assistant with Mermaid).
7. Research presentation or deep dive. 20 to 60 minutes. You defend one line of work: motivation, baselines, ablations, failures, and an open-ended research problem. AI: not allowed.
8. LLM fundamentals and ML coding. 45 minutes. Explain and hand-code post-training pieces such as RLHF objectives, DPO-style losses and attention. Reports say this lab leans toward hand-coded models and past research. AI: not allowed.
9. Behavioral. 45 minutes, the classic Meta behavioral (impact, conflict, ownership, technical leadership) plus an AI-oriented behavioral. AI: not allowed.
10. Hiring committee, then team match and offer. AI: not applicable.

Every report found ran the loop virtually. The questions below follow that mix: two classic problems, three AI-enabled coding tasks, four hand-coded ML tasks, seven research design, evaluation and debugging questions, two research deep dive questions and one behavioral question.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Find the shortest clear path through a binary grid and return the cells on it, not... | Classic algorithm | Onsite classic coding or phone screen | Not allowed | Medium |
| [Q2](#q2) | A model emitted a tool-call argument with unbalanced parentheses. Remove the fewest... | Classic algorithm | Phone screen or onsite classic coding | Not allowed | Medium |
| [Q3](#q3) | Fix the tool-call parser and rollout harness so multi-call turns stop crashing, then... | Debug | Onsite AI-enabled coding | Provided (CoderPad assistant) | Medium |
| [Q4](#q4) | Review this AI-generated reward function for tool-call correctness before it goes... | Review | Onsite AI-enabled coding | Provided (CoderPad assistant) | Medium |
| [Q5](#q5) | Extend a tool-plan executor in three parts: references, dependency order, parallel... | Build | Onsite AI-enabled coding | Provided (CoderPad assistant) | Hard |
| [Q6](#q6) | Implement the DPO loss from policy and reference logits, with prompt and padding... | Build | LLM fundamentals and ML coding | Not allowed | Medium |
| [Q7](#q7) | Write GRPO group advantages and the clipped loss for multi-turn tool trajectories... | Build | LLM fundamentals and ML coding | Not allowed | Hard |
| [Q8](#q8) | Implement causal multi-head attention with a KV cache and prove decoding matches the... | Build | LLM fundamentals and ML coding | Not allowed | Medium |
| [Q9](#q9) | Your RLHF dashboard shows negative KL on some batches. Fix the estimator, then pick... | Concept plus code | LLM fundamentals and ML coding | Not allowed | Medium |
| [Q10](#q10) | Design the post-training recipe for a search agent: data, environment, reward and... | Design | Research or ML design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q11](#q11) | Design a long-horizon tool-use benchmark, and decide what to report: pass@1, pass@16... | Concept plus code | Research or ML design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q12](#q12) | Train reward climbs while held-out accuracy falls. Read the RL log and the sample... | Trace debug | Research or ML design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q13](#q13) | Add a thinking-length penalty without breaking hard problems, then explain an... | Design | Research or ML design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q14](#q14) | Size an agentic RL run where tools are slow: GPU split, step time and synchronous... | Design | Research or ML design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q15](#q15) | Post-train an assistant to use personal memory in tool calls without leaking it or... | Design | Research or ML design | Provided (CoderPad assistant with Mermaid) | Hard |
| [Q16](#q16) | Decide which SFT data to drop before online RL, write the filter, and design the... | Concept plus code | Research or ML design | Provided (CoderPad assistant with Mermaid) | Medium |
| [Q17](#q17) | Defend your strongest post-training result against a skeptical interviewer... | Design | Research presentation or deep dive | Not allowed | Hard |
| [Q18](#q18) | Plan a six-month research program for long-horizon tool use, with milestones and... | Design | Research presentation or deep dive | Not allowed | Hard |
| [Q19](#q19) | How do you use AI tools in your research and post-training work, and where do you... | Behavioral | Behavioral | Not allowed | Medium |

### Classic coding (no AI, as at Meta)

<a id="q1"></a>

#### Q1. Find the shortest clear path through a binary grid and return the cells on it, not just the length

Round: Onsite classic coding or phone screen. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: BFS on grids.
Related practice: [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) (Medium), [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) (Medium), [Number of Islands](https://leetcode.com/problems/number-of-islands/) (Medium).

An n by n grid holds 0 (open) and 1 (blocked). You may move to any of the 8 neighbours. Return the list of cells on a shortest clear path from the top-left to the bottom-right corner, or an empty list if none exists. Follow-up: the interviewer asks for the length only, but without a visited set or parent map.

<details>
<summary>Show answer</summary>

Task: unit-cost shortest path on a grid, then reconstruct the path itself.

What they grade: choosing BFS and saying why, marking visited at the right moment, path reconstruction, edge cases, talking while coding.

Approach: Every move costs 1, so breadth-first search pops cells in order of distance, and the first time it pops the goal the path is optimal. Dijkstra gives the same answer with a heap and an extra log factor, so it only wins when moves have different costs (for example sqrt(2) for diagonals). A* with the Chebyshev distance as heuristic expands fewer cells on open grids but adds code to get right in 20 minutes. BFS is O(n squared) time and memory.

Mark a cell visited when you push it, not when you pop it; otherwise one cell can enter the queue up to 8 times. To return the path, store a parent pointer for each discovered cell and walk back from the goal. For the follow-up, write the distance into the grid itself (for example as negative numbers) so you need no visited set or parent map; the queue still holds the frontier, so memory is O(frontier), not O(1). Strict O(1) extra memory means rescanning the grid once per level, marking every open neighbour of a level-d cell as level d+1, which costs O(n^2 * L) time, where L is the path length. Either way, say out loud that this mutates the input.

Code:

```python
from collections import deque

def shortest_clear_path(grid):
    n = len(grid)
    if grid[0][0] or grid[n - 1][n - 1]:
        return []
    goal = (n - 1, n - 1)
    parent = {(0, 0): None}
    q = deque([(0, 0)])
    while q:
        cell = q.popleft()
        if cell == goal:
            path = []
            while cell is not None:
                path.append(cell)
                cell = parent[cell]
            return path[::-1]
        r, c = cell
        for dr in (-1, 0, 1):
            for dc in (-1, 0, 1):
                nr, nc = r + dr, c + dc
                if 0 <= nr < n and 0 <= nc < n and not grid[nr][nc] and (nr, nc) not in parent:
                    parent[(nr, nc)] = cell
                    q.append((nr, nc))
    return []

assert shortest_clear_path([[0, 1], [1, 0]]) == [(0, 0), (1, 1)]
assert shortest_clear_path([[0, 0, 0], [1, 1, 0], [1, 1, 0]]) == [(0, 0), (0, 1), (1, 2), (2, 2)]
assert shortest_clear_path([[1, 0], [0, 0]]) == []
assert shortest_clear_path([[0]]) == [(0, 0)]
assert shortest_clear_path([[0, 1, 0], [1, 1, 0], [0, 0, 0]]) == []
print("ok")
```

Verify: a 1 by 1 open grid, a blocked start, a blocked goal, an unreachable goal, and a grid where only a diagonal step works.

Pitfalls: LeetCode 1091 counts cells, not moves, so its answer equals len(path); skipping the start and goal checks; the self-loop when dr and dc are both 0 (harmless here because the start is already in parent, but say so).

Follow-ups: weighted diagonals (switch to Dijkstra or A*); many start cells (multi-source BFS, as in Rotting Oranges); a 10,000 by 10,000 grid (bidirectional BFS to cut the frontier).

Why this is likely: a recent Glassdoor Meta research scientist report names a BFS maze problem in the no-AI onsite coding round, and grid BFS problems sit in the recent Meta-tagged lists (medium confidence).

</details>

<a id="q2"></a>

#### Q2. A model emitted a tool-call argument with unbalanced parentheses. Remove the fewest characters to make it valid

Round: Phone screen or onsite classic coding. AI policy: Not allowed. Type: Classic algorithm. Difficulty: Medium. Topic: Stack and string parsing.
Related practice: [Minimum Remove to Make Valid Parentheses](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/) (Medium), [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) (Easy), [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) (Medium).

The string mixes letters with "(" and ")", for example `filter(a)b(c))d(`. Remove the minimum number of parentheses so the result is valid, and return any valid result. Letters must stay in order.

<details>
<summary>Show answer</summary>

Task: one pass to find unmatched parentheses, one pass to rebuild the string.

What they grade: the stack idea, O(n) time, correct handling of both kinds of unmatched parenthesis, and clean tests.

Approach: Scan left to right with a stack of indices of open parentheses. A ")" with an empty stack has no partner, so mark it for removal. A ")" with a non-empty stack pops its partner. After the scan, every index left on the stack is an unmatched "(", so mark those too. Rebuild the string, skipping marked indices. Each removed character is unavoidable: an unmatched ")" has no "(" to its left, and an unmatched "(" has no ")" to its right, so the result is minimal.

Alternatives: a two-pass counter version (left to right drops extra ")", right to left drops extra "(") uses no index set but builds two intermediate strings; it is equally O(n). Trying every subset of removals is exponential and only worth naming to dismiss. I pick the stack because it reads clearly and extends to reporting which positions were removed, which helps when you log repairs to model output.

Code:

```python
def min_remove(s):
    stack, drop = [], set()
    for i, ch in enumerate(s):
        if ch == "(":
            stack.append(i)
        elif ch == ")":
            if stack:
                stack.pop()
            else:
                drop.add(i)
    drop.update(stack)
    return "".join(ch for i, ch in enumerate(s) if i not in drop)

def valid(s):
    depth = 0
    for ch in s:
        depth += (ch == "(") - (ch == ")")
        if depth < 0:
            return False
    return depth == 0

for s, removed in [("filter(a)b(c))d(", 2), ("lee(t(c)o)de)", 1), ("))((", 4), ("", 0), ("a(b)c", 0)]:
    out = min_remove(s)
    assert valid(out) and len(s) - len(out) == removed, (s, out)
assert min_remove("a)b(c)d") == "ab(c)d"
print("ok")
```

Verify: empty string, no parentheses, only ")" characters, only "(" characters, nested valid input left unchanged.

Pitfalls: removing from the string while iterating (O(n squared) and index bugs); forgetting the leftover "(" on the stack; returning only the count when the prompt asks for the string.

Follow-ups: three bracket types where a ")" can only close "(" (a minimum-removal answer then needs interval DP, O(n cubed)); stream the input in O(1) memory if only ")" can be unmatched.

Why this is likely: recent Meta reports list stack and expression parsing problems among recurring classic topics, and this problem is in Meta's three-month tagged list (medium confidence for the topic, low for this exact problem).

</details>

### AI-enabled coding (AI assistant allowed, as in Meta's AI-enabled round)

<a id="q3"></a>

#### Q3. Fix the tool-call parser and rollout harness so multi-call turns stop crashing, then write the missing tests

Round: Onsite AI-enabled coding. AI policy: Provided (CoderPad assistant). Type: Debug. Difficulty: Medium. Topic: Agent rollout harness.

The repo has `agent/parser.py`, `agent/harness.py`, `agent/tools.py` and `tests/test_parser.py`. The one existing test passes, but RL rollouts crash when a model makes two tool calls in one turn, calls an unknown tool, or pretty-prints its JSON. Find the bugs, fix them and add tests.

```text
# agent/parser.py
import json, re

CALL_RE = re.compile(r"<tool_call>(.*)</tool_call>")

def parse_calls(text):
    calls = []
    for m in CALL_RE.finditer(text):
        obj = json.loads(m.group(1))
        calls.append((obj["name"], obj["arguments"]))
    return calls

# agent/harness.py
from agent.parser import parse_calls

def run_episode(model, tools, prompt, max_turns=8):
    msgs = [{"role": "user", "content": prompt}]
    for turn in range(1, max_turns):
        out = model(msgs)
        msgs.append({"role": "assistant", "content": out})
        calls = parse_calls(out)
        if not calls:
            return msgs
        for name, args in calls:
            result = tools[name](**args)
            msgs.append({"role": "tool", "content": result})
    return msgs
```

<details>
<summary>Show answer</summary>

Task: reproduce each crash with a failing test, fix the parser and harness, keep the passing test green.

What they grade: tests before fixes, a clear prompt to the assistant, reading every line of its diff, and catching its mistakes.

Approach: First I run the existing test, then ask the assistant: "Write failing pytest cases for parse_calls with two calls in one string, a call whose JSON spans lines, and malformed JSON. Do not change the source." I read the tests, run them, and confirm they fail for the reasons I expect. Then I list the bugs myself: (1) the greedy `.*` swallows everything between the first open tag and the last close tag, so two calls become one invalid JSON blob; (2) without `re.S` the dot does not match newlines, so pretty-printed calls vanish silently; (3) malformed JSON raises and kills the rollout instead of becoming a reward-visible error; (4) `range(1, max_turns)` runs one turn fewer than the budget; (5) an unknown tool raises KeyError; (6) a tool exception or a non-string result crashes or corrupts the transcript.

The assistant's first fix changed `.*` to `.*?` but did not add `re.S`. The multi-line test still failed, and because the tests existed first, I saw it within seconds. Every error now becomes an observation message, because during RL the policy has to see its mistakes to learn from them, and a crash would drop the whole trajectory.

Code:

```python
import json, re

CALL_RE = re.compile(r"<tool_call>(.*?)</tool_call>", re.S)

def parse_calls(text):
    calls = []
    for m in CALL_RE.finditer(text):
        try:
            obj = json.loads(m.group(1))
            calls.append((obj["name"], obj.get("arguments", {}), None))
        except (json.JSONDecodeError, KeyError, TypeError) as e:
            calls.append((None, None, f"parse error: {type(e).__name__}"))
    return calls

def run_episode(model, tools, prompt, max_turns=8):
    msgs = [{"role": "user", "content": prompt}]
    for _ in range(max_turns):
        out = model(msgs)
        msgs.append({"role": "assistant", "content": out})
        calls = parse_calls(out)
        if not calls:
            return msgs
        for name, args, err in calls:
            if err is None and name not in tools:
                err = f"unknown tool: {name}"
            if err is None:
                try:
                    res = tools[name](**args)
                    content = res if isinstance(res, str) else json.dumps(res)
                except Exception as e:
                    content = f"tool error: {type(e).__name__}: {e}"
            else:
                content = err
            msgs.append({"role": "tool", "content": content})
    return msgs

two = '<tool_call>{"name": "add", "arguments": {"a": 1, "b": 2}}</tool_call> and <tool_call>{"name": "add", "arguments": {"a": 3, "b": 4}}</tool_call>'
assert [c[:2] for c in parse_calls(two)] == [("add", {"a": 1, "b": 2}), ("add", {"a": 3, "b": 4})]
pretty = '<tool_call>{\n  "name": "add",\n  "arguments": {"a": 1, "b": 1}\n}</tool_call>'
assert parse_calls(pretty)[0][:2] == ("add", {"a": 1, "b": 1})
assert parse_calls("<tool_call>{oops}</tool_call>")[0][2].startswith("parse error")

tools = {"add": lambda a, b: {"sum": a + b}, "boom": lambda: 1 / 0}
script = iter([two, '<tool_call>{"name": "nope", "arguments": {}}</tool_call><tool_call>{"name": "boom", "arguments": {}}</tool_call>', "done"])
msgs = run_episode(lambda m: next(script), tools, "hi", max_turns=3)
tool_msgs = [m["content"] for m in msgs if m["role"] == "tool"]
assert tool_msgs == ['{"sum": 3}', '{"sum": 7}', "unknown tool: nope", "tool error: ZeroDivisionError: division by zero"]
calls_seen = []
run_episode(lambda m: calls_seen.append(1) or two, tools, "hi", max_turns=4)
assert len(calls_seen) == 4
print("ok")
```

Verify: two calls in one turn, multi-line JSON, malformed JSON, unknown tool, tool exception, dict results, and the turn budget used exactly.

Pitfalls: catching bare `Exception` in the parser and hiding real bugs; letting the assistant edit the tests to match its code; dropping crashed trajectories, which silently biases the RL batch toward easy prompts.

Follow-ups: add call ids so each observation maps to its call; cap tool output length; make tool calls with no side effects run in parallel.

Why this is likely: Meta's careers FAQ describes AI-enabled rounds about reading, debugging and building on existing code, and a recent research scientist report describes a whole project with a bug and tests you write yourself (high confidence for the format, medium for the topic).

</details>

<a id="q4"></a>

#### Q4. Review this AI-generated reward function for tool-call correctness before it goes into an RL run

Round: Onsite AI-enabled coding. AI policy: Provided (CoderPad assistant). Type: Review. Difficulty: Medium. Topic: Reward design and reward hacking.

A teammate asked the assistant: "score a trajectory's tool calls against the gold calls; 1.0 if all match; also check the final numeric answer". It produced the code below. Find every defect, say which ones the policy would exploit, and fix it.

```text
# reward.py (AI-generated)
import json

def grade_calls(pred_calls, gold_calls):
    if len(pred_calls) == 0:
        return 1.0                      # nothing to do
    hits = 0
    for i in range(len(gold_calls) - 1):
        p, g = pred_calls[i], gold_calls[i]
        args = json.loads(p["arguments"], ignore_order=True)
        if p["name"] == g["name"] and args == g["arguments"]:
            hits += 1
    return hits / len(pred_calls)

def check_answer(answer_text, gold):
    return eval(answer_text) == gold
```

<details>
<summary>Show answer</summary>

Task: list defects by severity, mark the exploitable ones, ship a fixed version with tests.

What they grade: spotting the reward hacks first, not only the crashes; knowing which APIs exist; security sense; tests that pin the fix.

Approach: I read it as the policy would. Defects, worst first:

1. Empty prediction scores 1.0. A policy learns to call nothing within a few hundred steps. Exploitable.
2. `eval(answer_text)` executes model output inside the reward worker. That is remote code execution from a policy you are optimizing to find any reward path. Security issue and exploitable.
3. The loop stops at `len(gold_calls) - 1`, so the last gold call is never checked. Off-by-one, and exploitable: the final call is often the one that commits an action.
4. Dividing by `len(pred_calls)` rewards fewer calls: with three gold calls, a prediction that makes only the first two correctly scores 1.0. It raises IndexError only when the prediction has two or more fewer calls than gold.
5. `json.loads` has no `ignore_order` parameter; the assistant invented it, so every call raises TypeError. Hallucinated API.
6. Malformed JSON from the policy crashes the reward instead of scoring 0.

My fix scores matched calls in order and divides by the longer list, so both missing and extra calls cost reward. Malformed calls score 0. The answer check parses a number with `float` and compares with a tolerance. Alternatives: order-free F1 over call multisets (right when calls are independent), or an LLM judge for free-form arguments (flexible, but slower and itself hackable). I keep exact matching in order for dependent calls and add a judge later for string arguments only.

Code:

```python
import json, math

def grade_calls(pred_calls, gold_calls):
    if not gold_calls:
        return 1.0 if not pred_calls else 0.0
    hits = 0
    for p, g in zip(pred_calls, gold_calls):
        try:
            args = p["arguments"] if isinstance(p["arguments"], dict) else json.loads(p["arguments"])
        except (json.JSONDecodeError, TypeError, KeyError):
            continue
        if p.get("name") == g["name"] and args == g["arguments"]:
            hits += 1
    return hits / max(len(pred_calls), len(gold_calls))

def check_answer(answer_text, gold, tol=1e-6):
    try:
        return math.isclose(float(answer_text.strip()), gold, rel_tol=tol, abs_tol=tol)
    except (ValueError, AttributeError):
        return False

g = [{"name": "search", "arguments": {"q": "x"}}, {"name": "book", "arguments": {"id": 7}}]
assert grade_calls([], g) == 0.0
assert grade_calls([{"name": "search", "arguments": '{"q": "x"}'}], g) == 0.5
assert grade_calls([{"name": "search", "arguments": {"q": "x"}}, {"name": "book", "arguments": {"id": 8}}], g) == 0.5
assert grade_calls(g + [{"name": "book", "arguments": {"id": 7}}], g) == 2 / 3
assert grade_calls([{"name": "search", "arguments": "{bad"}], g) == 0.0
assert grade_calls([], []) == 1.0
assert check_answer(" 42.0 ", 42) and not check_answer("__import__('os')", 42)
print("ok")
```

Verify: empty prediction, extra calls, wrong final call, malformed JSON, code injection string, whitespace in answers.

Pitfalls: fixing crashes but leaving the empty-call hack; trusting "it runs" as proof the API exists; an LLM judge with no tests of its own.

Follow-ups: argument normalization (dates, units); partial credit for the right tool with wrong arguments, and how the policy might game it.

Why this is likely: Meta's AI-enabled round grades reading and verifying AI output, and recent candidate summaries stress catching incorrect assumptions in generated code; reward functions are core work for a post-training team (medium confidence).

</details>

<a id="q5"></a>

#### Q5. Extend a tool-plan executor in three parts: references, dependency order, parallel batches

Round: Onsite AI-enabled coding. AI policy: Provided (CoderPad assistant). Type: Build. Difficulty: Hard. Topic: Agent planning and execution.

A planner model emits a plan as a list of calls: `{"id": "f", "tool": "fetch", "args": {"url": "$s.top_url"}}`. An argument of the form `$id` or `$id.key` refers to an earlier result. Part 1: run calls in list order with references resolved. Part 2: the planner now emits calls in any order; run them in dependency order and reject cycles and unknown references. Part 3: return the calls grouped into batches that could run in parallel.

<details>
<summary>Show answer</summary>

Task: build the executor in stages, keeping every earlier test green.

What they grade: splitting the task, tests per part, a graph algorithm stated in plain words, reviewing what the assistant writes.

Approach: Part 1 is a resolver plus a loop. For part 2, each call depends on the ids its arguments reference, so the plan is a graph and the run order is a topological sort. For part 3, Kahn's algorithm by levels gives batches: level 0 holds calls with no dependencies, level k holds calls whose dependencies all sit in earlier levels. Alternatives: DFS topological sort (one order, no batches) or running everything with futures and awaiting references (fast, but cycles become deadlocks instead of clear errors). I pick levels because they give a cycle check and batches together.

Delegation: I ask the assistant for the resolver and its tests, and write the level function myself because it is the graded part. Its resolver treated any string starting with "$" as a reference, so a price argument of "$5" raised KeyError. I tightened the pattern to `$<identifier>` and added that case as a test.

Code:

```python
import re
REF = re.compile(r"^\$([A-Za-z_]\w*)(?:\.(\w+))?$")

def resolve(v, results):
    m = REF.match(v) if isinstance(v, str) else None
    if not m:
        return v
    out = results[m.group(1)]
    return out[m.group(2)] if m.group(2) else out

def deps_of(call):
    return {REF.match(v).group(1) for v in call["args"].values() if isinstance(v, str) and REF.match(v)}

def levels(calls):
    deps = {c["id"]: deps_of(c) for c in calls}
    for cid, d in deps.items():
        if d - deps.keys():
            raise ValueError(f"{cid} references unknown {sorted(d - deps.keys())}")
    done, out = set(), []
    while len(done) < len(deps):
        ready = sorted(c for c in deps if c not in done and deps[c] <= done)
        if not ready:
            raise ValueError(f"cycle among {sorted(deps.keys() - done)}")
        out.append(ready)
        done.update(ready)
    return out

def run(calls, tools):
    by_id, results = {c["id"]: c for c in calls}, {}
    for batch in levels(calls):
        for cid in batch:
            c = by_id[cid]
            results[cid] = tools[c["tool"]](**{k: resolve(v, results) for k, v in c["args"].items()})
    return results

tools = {"search": lambda q: {"top_url": f"u/{q}"}, "fetch": lambda url: f"page {url}",
         "summarize": lambda text, budget: f"sum({text})", "weather": lambda city: "sunny"}
plan = [{"id": "sum", "tool": "summarize", "args": {"text": "$f", "budget": "$5"}},
        {"id": "f", "tool": "fetch", "args": {"url": "$s.top_url"}},
        {"id": "w", "tool": "weather", "args": {"city": "Paris"}},
        {"id": "s", "tool": "search", "args": {"q": "llama"}}]
assert levels(plan) == [["s", "w"], ["f"], ["sum"]]
assert run(plan, tools)["sum"] == "sum(page u/llama)"
for bad, msg in [([{"id": "a", "tool": "fetch", "args": {"url": "$b"}}, {"id": "b", "tool": "fetch", "args": {"url": "$a"}}], "cycle"),
                 ([{"id": "a", "tool": "fetch", "args": {"url": "$zz"}}], "unknown")]:
    try:
        levels(bad); assert False
    except ValueError as e:
        assert msg in str(e)
print("ok")
```

Verify: out-of-order plan, a literal "$5" argument, a two-node cycle, an unknown reference, independent calls landing in the same batch.

Pitfalls: the level loop rescans every call on each level, so it is O(V * (V + E)) in the worst case (a long chain); an in-degree queue (Kahn's algorithm) makes it O(V + E) if plans grow past a few hundred calls. Duplicate ids silently overwrite each other; reject them in production.

Follow-ups: run each batch with a thread pool and per-call timeouts; partial failure (skip dependants, keep independent branches); feed the error back to the planner as an observation.

Why this is likely: candidate reports say Meta's AI-enabled round is a progressive multi-part build in CoderPad, and long-horizon tool use is the team's stated focus (medium confidence for format, low for this exact task).

</details>

### LLM fundamentals and ML coding (no AI, as at Meta)

<a id="q6"></a>

#### Q6. Implement the DPO loss from policy and reference logits, with prompt and padding masked out

Round: LLM fundamentals and ML coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Preference optimization.

You get token ids of shape (B, T), a loss mask that is 1 on response tokens and 0 on prompt and padding, and logits of shape (B, T, V) from the policy and from a frozen reference model, for both the chosen and the rejected response. Write the sequence log-probability function and the DPO loss in NumPy, and show tests that would catch the usual bugs.

<details>
<summary>Show answer</summary>

Task: per-sequence log-probabilities with the right shift and mask, then the DPO objective.

What they grade: the one-token shift, masking, a numerically stable log-sigmoid, and tests that prove each piece.

Approach: The logit at position t predicts token t+1, so compare `logits[:, :-1]` with `ids[:, 1:]` and use `mask[:, 1:]`. Gather the log-softmax at the target ids and sum over response tokens. DPO treats beta times the log-ratio between policy and reference as an implicit reward, so the loss per pair is minus log sigmoid of beta times the difference between the chosen and rejected log-ratios. Compute minus log sigmoid(m) as `logaddexp(0, -m)` so large margins do not overflow.

Alternatives: averaging log-probs per token instead of summing removes the length effect but changes the objective (it is closer to SimPO-style length normalization, which drops the reference); IPO swaps the logistic loss for a squared loss and resists overfitting on deterministic preferences. I keep the summed DPO loss as the baseline and report the length of chosen versus rejected responses, because DPO is known to drift toward longer outputs.

Code:

```python
import numpy as np

def log_softmax(x):
    x = x - x.max(-1, keepdims=True)
    return x - np.log(np.exp(x).sum(-1, keepdims=True))

def seq_logps(logits, ids, mask):
    lp = log_softmax(logits[:, :-1])
    tok = np.take_along_axis(lp, ids[:, 1:, None], -1)[..., 0]
    return (tok * mask[:, 1:]).sum(-1)

def dpo_loss(pol_c, pol_r, ref_c, ref_r, beta=0.1):
    margin = beta * ((pol_c - ref_c) - (pol_r - ref_r))
    return np.logaddexp(0.0, -margin).mean(), (margin > 0).mean()

rng = np.random.default_rng(0)
B, T, V = 2, 6, 11
ids = rng.integers(0, V, (B, T))
mask = np.array([[0, 0, 0, 1, 1, 1], [0, 0, 1, 1, 1, 0]])
logits = rng.normal(size=(B, T, V))
lp = seq_logps(logits, ids, mask)

bumped = logits.copy(); bumped[:, 0] += 5 * rng.normal(size=V)
assert np.allclose(seq_logps(bumped, ids, mask), lp)
bumped = logits.copy(); bumped[0, 3] += 5 * rng.normal(size=V)
assert not np.isclose(seq_logps(bumped, ids, mask)[0], lp[0])
bumped = logits.copy(); bumped[1, 4] += 5 * rng.normal(size=V)
assert np.isclose(seq_logps(bumped, ids, mask)[1], lp[1])  # position 4 predicts padding token 5

loss, acc = dpo_loss(lp, lp, lp, lp)
assert np.isclose(loss, np.log(2))
better, _ = dpo_loss(lp + 1.0, lp, lp, lp)
assert better < np.log(2)
big, _ = dpo_loss(np.array([1e4]), np.array([-1e4]), np.zeros(1), np.zeros(1))
assert np.isfinite(big)
print("ok")
```

Verify: policy equal to reference gives log 2; changing logits that only predict prompt tokens or only predict padding leaves the loss unchanged; raising the chosen log-prob lowers the loss; huge margins stay finite.

Pitfalls: no shift (each token scores itself); including prompt tokens; running the reference with gradients on; mean over tokens when the paper's loss sums; beta far too large, which overfits to the preference labels.

Follow-ups: why chosen log-probs often fall during DPO training; when to prefer online RL over offline DPO; how to build the pairs from on-policy samples.

Why this is likely: Llama 4 post-training ended with a lightweight DPO stage, and candidate summaries say this lab's loops lean toward hand-coded models (medium confidence).

</details>

<a id="q7"></a>

#### Q7. Write GRPO group advantages and the clipped loss for multi-turn tool trajectories, masking tool outputs

Round: LLM fundamentals and ML coding. AI policy: Not allowed. Type: Build. Difficulty: Hard. Topic: Policy-gradient RL for LLMs.

Each prompt gets a group of G sampled trajectories. A trajectory interleaves model tokens and tool-output tokens. You get per-token `logp_new` and `logp_old` of shape (N, T), a scalar reward per trajectory, and a mask that is 1 only on model-generated tokens. Write the group-relative advantage and the clipped surrogate loss in NumPy.

<details>
<summary>Show answer</summary>

Task: group-normalized advantages, a PPO-style clipped token loss, correct masking and aggregation.

What they grade: why tool tokens are masked, the min in the clipped objective, how the loss is averaged, and tests.

Approach: GRPO drops the value network. For each prompt, the advantage of trajectory i is its reward minus the group mean, divided by the group standard deviation, and every model token in that trajectory shares it. The ratio is exp(logp_new minus logp_old). The per-token objective is the minimum of ratio times advantage and the clipped ratio times advantage; the loss is its negative. Tool-output tokens were not sampled by the policy, so their log-probs carry no policy-gradient signal; leaving them in trains the model to predict tool output, which is wrong for the policy and often leaks the environment into the model.

Aggregation is a real choice. The original GRPO averages tokens within each sequence and then across sequences, which gives each token in a short answer more weight. DAPO averages over all tokens in the batch, and Dr. GRPO also removes the standard-deviation division. I use a token-level mean and keep the std division as a flag, because groups where every reward is equal give zero advantage either way and are better filtered out before the step.

Code:

```python
import numpy as np

def group_advantages(rewards, G, scale=True, eps=1e-6):
    r = rewards.reshape(-1, G)
    a = r - r.mean(1, keepdims=True)
    if scale:
        a = a / (r.std(1, keepdims=True) + eps)
    return a.reshape(-1)

def grpo_loss(logp_new, logp_old, adv, mask, clip=0.2):
    ratio = np.exp(logp_new - logp_old)
    a = adv[:, None]
    obj = np.minimum(ratio * a, np.clip(ratio, 1 - clip, 1 + clip) * a)
    return -(obj * mask).sum() / mask.sum()

rng = np.random.default_rng(1)
rewards = np.array([1, 0, 0, 1, 1, 1, 1, 1], dtype=float)
adv = group_advantages(rewards, G=4)
assert np.allclose(adv[4:], 0) and np.isclose(adv[:4].sum(), 0)

N, T = 8, 10
mask = np.ones((N, T)); mask[:, 3:6] = 0
old = rng.normal(size=(N, T))
assert np.isclose(grpo_loss(old, old, adv, mask), -(adv[:, None] * mask).sum() / mask.sum())
new = old.copy(); new[:, 3:6] += 3.0
assert np.isclose(grpo_loss(new, old, adv, mask), grpo_loss(old, old, adv, mask))
one = np.ones((1, 1))
assert np.isclose(grpo_loss(old[:1, :1] + np.log(1.5), old[:1, :1], np.array([2.0]), one), -1.2 * 2.0)
assert np.isclose(grpo_loss(old[:1, :1] + np.log(0.5), old[:1, :1], np.array([-2.0]), one), -0.8 * -2.0)
print("ok")
```

Verify: a group with identical rewards gives zero advantage; with ratio 1 the loss is minus the masked mean advantage; changing tool-token log-probs changes nothing; the clip binds at 1.2 for positive and 0.8 for negative advantages.

Pitfalls: masking prompt tokens but not tool outputs; computing the std across the whole batch instead of per group; dropping the clipped term (plain ratio times advantage), which lets a positive-advantage token push its ratio without bound. A second bug is keeping only the clipped term without the min, which removes the corrective gradient when a negative-advantage token's ratio has already risen above 1 + clip.

Follow-ups: add a KL penalty to a reference (and why many recent recipes drop it); asymmetric clipping; trajectories truncated at the turn limit.

Why this is likely: the job asks for hands-on post-training runs, Meta's Muse Spark post describes a rebuilt large-scale RL stack, and candidate summaries say the lab's loops include hand-coded models (medium confidence).

</details>

<a id="q8"></a>

#### Q8. Implement causal multi-head attention with a KV cache and prove decoding matches the full pass

Round: LLM fundamentals and ML coding. AI policy: Not allowed. Type: Build. Difficulty: Medium. Topic: Transformers from scratch.

Write one attention layer in NumPy with weight matrices Wq, Wk, Wv, Wo of shape (d, d) and h heads. It must support a full causal pass over T tokens and incremental decoding with a cache of past keys and values. Show a test that the two paths agree.

<details>
<summary>Show answer</summary>

Task: split heads, scaled dot product, causal mask with an offset for cached positions, concatenate heads, project.

What they grade: shapes said out loud, the mask offset bug, the 1 over sqrt(d_head) scale, a stable softmax, and a test that compares cached and uncached outputs.

Approach: Project x of shape (T, d) to q, k and v, reshape each to (h, T, d_head). With a cache of P past positions, concatenate the cached k and v in front, so keys have length P + T. Query i sits at absolute position P + i and may attend to keys 0 through P + i, so the mask blocks key j when j is greater than P + i. Without the offset, the first new token can only see the first cached token, a common bug in cache code.

Alternatives: grouped-query attention shares each k and v head across several query heads, cutting cache memory by the group factor (Llama models use it); multi-query attention shares one k and v head, saving more memory at some quality cost. Numbers: for an 8B-class model with 32 layers, 8 KV heads of size 128 and bf16, the cache costs 2 x 32 x 8 x 128 x 2 bytes, about 131 KB per token, so a 32K-token context needs about 4.3 GB. Full multi-head with 32 KV heads would need four times that.

Code:

```python
import numpy as np

def attention(x, W, h, cache=None):
    Wq, Wk, Wv, Wo = W
    T, d = x.shape
    dh = d // h
    split = lambda m: m.reshape(T, h, dh).transpose(1, 0, 2)
    q, k, v = split(x @ Wq), split(x @ Wk), split(x @ Wv)
    P = 0
    if cache is not None:
        P = cache[0].shape[1]
        k = np.concatenate([cache[0], k], 1)
        v = np.concatenate([cache[1], v], 1)
    s = q @ k.transpose(0, 2, 1) / np.sqrt(dh)
    blocked = np.arange(P + T)[None, :] > (P + np.arange(T))[:, None]
    s = np.where(blocked, -np.inf, s)
    s = np.exp(s - s.max(-1, keepdims=True))
    out = (s / s.sum(-1, keepdims=True)) @ v
    return out.transpose(1, 0, 2).reshape(T, d) @ Wo, (k, v)

rng = np.random.default_rng(0)
d, h, T = 16, 4, 7
W = [rng.normal(size=(d, d)) / np.sqrt(d) for _ in range(4)]
x = rng.normal(size=(T, d))
full, _ = attention(x, W, h)

out, cache = attention(x[:4], W, h)
steps = [out]
for t in range(4, T):
    o, cache = attention(x[t:t + 1], W, h, cache)
    steps.append(o)
assert np.allclose(np.concatenate(steps), full)

x2 = x.copy(); x2[-1] += 10
assert np.allclose(attention(x2, W, h)[0][:-1], full[:-1])
print("ok")
```

Verify: prefill then one token at a time equals the full pass; changing the last token leaves earlier outputs unchanged (causality); T = 1 works with and without a cache.

Pitfalls: masking with j > i instead of j > P + i; scaling by sqrt(d) instead of sqrt(d_head); softmax without subtracting the max; transposing heads back in the wrong order.

Follow-ups: add rotary embeddings (rotate q and k by absolute position before caching k); convert to grouped-query attention; estimate decode speed from memory bandwidth.

Why this is likely: recent Meta ML engineer question banks and ML interview threads report writing multi-head attention from scratch, and the lab's loops lean toward hand-coded models (medium confidence).

</details>

<a id="q9"></a>

#### Q9. Your RLHF dashboard shows negative KL on some batches. Fix the estimator, then pick PPO, DPO or GRPO for three goals

Round: LLM fundamentals and ML coding. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: RLHF objectives and KL control.

The logged "KL to reference" is the batch mean of `logp_policy - logp_ref` on sampled tokens, and it sometimes goes negative. Explain why, write a better estimator, and say what the KL term does in the RLHF objective. Then choose a method for: (a) matching style preferences from 200K human pairs, (b) maths with a checkable answer, (c) a tool-use agent with a learned reward model.

<details>
<summary>Show answer</summary>

Task: explain the KL-regularized objective, fix the estimator, make three method choices with reasons.

What they grade: the objective in words, why a per-sample estimator can be negative, unbiasedness versus variance, and choices tied to the data and reward.

Approach: RLHF maximizes expected reward minus beta times KL from the policy to the reference. The KL term keeps the policy near the distribution where the reward model is accurate; without it the policy finds outputs the reward model overrates (reward over-optimization). The logged quantity is the naive estimator k1 = log p minus log q on samples from p. It is unbiased, but single samples are often negative, so small batches can average below zero. The k3 estimator, (r minus 1) minus log r with r = q over p, is also unbiased, is never negative, and has lower variance when the two distributions are close. I log k3 and keep k1 as a check.

Choices. (a) DPO: offline pairs, no reward model to train, stable and cheap; its weakness, no fresh on-policy samples, matters less for style. (b) GRPO or another online RL method with a verifier: the reward is exact, so on-policy exploration pays off, and no value network is needed when groups of 8 to 16 samples give a baseline. (c) PPO with a value model or GRPO with KL to the reference: a learned reward model is hackable, so keep the KL penalty, hold out a second reward model or human evals, and stop on divergence.

Code:

```python
import numpy as np

rng = np.random.default_rng(0)
p = np.array([0.5, 0.3, 0.15, 0.05])
q = np.array([0.4, 0.35, 0.2, 0.05])
true_kl = float((p * np.log(p / q)).sum())

x = rng.choice(4, size=200_000, p=p)
logr = np.log(q[x]) - np.log(p[x])
k1 = -logr
k3 = np.exp(logr) - 1 - logr

assert (k1 < 0).mean() > 0.3
assert (k3 >= 0).all()
assert abs(k1.mean() - true_kl) < 3e-3 and abs(k3.mean() - true_kl) < 3e-3
assert k3.std() < k1.std()
print(round(true_kl, 4), round(k1.mean(), 4), round(k3.mean(), 4))
```

Verify: both estimators average to the true KL, k1 is negative on a large share of samples, k3 never is, and k3 has the smaller spread on this pair.

Pitfalls: putting KL in the reward and in the loss at once by accident; computing it over prompt tokens; reading a falling KL as success when the policy has collapsed to the reference.

Follow-ups: adaptive beta targeting a fixed KL; why DPO's implicit reward can drift from real preferences; what reward over-optimization looks like on a plot of reward versus KL.

Why this is likely: the only research scientist question listed on Glassdoor in recent reports is to explain how RLHF works in LLMs, and fundamentals rounds at this lab ask for hand-written pieces (medium confidence).

</details>

### Research and ML design (AI assistant allowed, as in Meta's design rounds)

<a id="q10"></a>

#### Q10. Design the post-training recipe for a search agent: data, environment, reward and evals, with numbers

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Design. Difficulty: Hard. Topic: RL for search agents.

Goal: a model that answers multi-hop questions by calling a `search` tool several times and citing what it found. You have a strong instruction-tuned base, a GPU budget for online RL, and no human labels for search behaviour. Design the recipe end to end.

<details>
<summary>Show answer</summary>

Task: a recipe that gives a reproducible reward, cheap rollouts and an eval you trust.

What they grade: environment choice with alternatives, a reward that is hard to game, numbers that add up, and a plan for what can go wrong.

Approach: I draw four boxes in Mermaid: prompts, environment, RL trainer, evals. The assistant drafts the diagram and a first list of reward terms. I check each term; it suggested rewarding the number of searches, which teaches the model to search for the sake of it, so I removed it.

Data: about 40K question and short-answer pairs from multi-hop QA sets plus synthetic questions built by chaining facts from the corpus, filtered so the current model solves each one between 10% and 90% of the time over 8 samples. A short SFT warm-up on a few thousand traces in the tool-call format makes calls parse before RL starts.

Environment alternatives: (a) live web search: realistic, but slow, costly, non-deterministic and hard to replay; (b) a frozen corpus snapshot with BM25 plus dense retrieval: deterministic, cheap, replayable; (c) an LLM that simulates search results: very cheap, but the policy can learn the simulator's quirks. I train on (b) and evaluate on (a), so the paper claim is "trained offline, works live".

Reward: exact match or token F1 on the final answer, 0 when the format is invalid, no reward for calls. Retrieved tokens are masked from the loss, as in Search-R1, because the policy did not generate them.

Numbers: 256 prompts per step times 8 samples is 2,048 trajectories; at about 5 searches each that is 10,240 retrievals. At 50 ms on 64 retrieval workers that is about 8 seconds per step; a live API at 1 second per call would take about 160 seconds. Each search returns 3 passages of about 300 tokens, so a 5-call trajectory carries about 4,500 retrieved tokens; with about 250 generated tokens per turn it reaches about 5,750 tokens of context.

Risks: answers leaking through the corpus in the question's own wording (deduplicate); the model learning to skip search on memorized facts (track calls per question by difficulty); corpus staleness against live eval.

Why this is likely: a recent lab report describes a research design round on building a coding LLM (framing, data, training), and the job names search and agentic tool use as the team's focus (medium confidence for format, low for this exact prompt).

</details>

<a id="q11"></a>

#### Q11. Design a long-horizon tool-use benchmark, and decide what to report: pass@1, pass@16 or pass^k

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Concept plus code. Difficulty: Hard. Topic: Agent evaluation.

You need a benchmark for agents that take 15 or more tool calls in a sandbox (files, a calendar, a mock store) to finish a task. Leadership wants one headline number. Design the tasks, the checker, the contamination controls and the metrics, and implement the estimators.

<details>
<summary>Show answer</summary>

Task: a benchmark whose number moves only when the agent really improves, plus the right summary statistics.

What they grade: state-based checking, contamination controls, statistics with intervals, and matching the metric to the claim.

Approach: Tasks: 300 tasks written by people after the training-data cutoff, each with a seeded sandbox and a checker that inspects the final sandbox state (the right file edited, the right order placed), not the transcript, so different valid paths all pass. Hold out 100 tasks that nobody tunes on. Contamination: canary strings in every task file, a search of training data for task text and paraphrases, and a fresh slice each quarter.

Metrics answer different questions. pass@1 is the expected success of one attempt, the product number. pass@k (at least one of k samples succeeds) measures how much capability RL can still sharpen; Meta's Muse Spark post reports RL gains on both pass@1 and pass@16. pass^k (all k attempts succeed, from tau-bench) measures reliability, which is what users of a long-horizon agent feel. I report all three from n = 16 samples per task using unbiased estimators, and the headline is pass@1 with a 95% interval.

I ask the assistant to draft the Mermaid diagram and the estimator functions, and I write the edge-case asserts myself. Its first pass_at_k computed the success rate of k fresh samples per task, a high-variance estimate, so I replaced it with the unbiased formula and checked that k = 1 equals c / n.

Numbers: with 300 tasks at a pass@1 near 0.4, the interval is about plus or minus 5.5 points, so a 3-point gain needs a paired test across the same tasks, or more tasks. Task-level variance dominates sample-level variance, so 16 samples per task matter less than 300 more tasks.

Code:

```python
from math import comb

def pass_at_k(n, c, k):
    return 1.0 if n - c < k else 1 - comb(n - c, k) / comb(n, k)

def pass_hat_k(n, c, k):
    return comb(c, k) / comb(n, k)

def summarize(results, k):
    return {m: sum(f(n, c, k) for n, c in results) / len(results) for m, f in
            [("pass@1", lambda n, c, k: c / n), (f"pass@{k}", pass_at_k), (f"pass^{k}", pass_hat_k)]}

assert pass_at_k(16, 0, 4) == 0.0 and pass_at_k(16, 16, 4) == 1.0
assert abs(pass_at_k(16, 4, 1) - 0.25) < 1e-12 and abs(pass_hat_k(16, 4, 1) - 0.25) < 1e-12
assert pass_hat_k(16, 3, 4) == 0.0
a = summarize([(16, 12), (16, 0), (16, 8)], 4)
b = summarize([(16, 10), (16, 2), (16, 8)], 4)
assert abs(a["pass@1"] - b["pass@1"]) < 1e-12 and b["pass@4"] > a["pass@4"] and a["pass^4"] > b["pass^4"]
print(a, b)
```

Verify: the edge cases of all failures and all passes, k = 1 matching the success rate, and a case where two models tie on pass@1 but split on pass@4 and pass^4.

Pitfalls: estimating pass@k as the success rate of k fresh samples (high variance) instead of the unbiased formula; grading transcripts with an LLM judge that the agent can flatter; reporting the best of several runs.

Follow-ups: cost per solved task; tool-call efficiency; how to score partial progress without letting it hide failures.

Why this is likely: the team works on long-horizon agentic tool use per the job post, and Meta's Muse Spark post reports pass@1 and pass@16 under RL scaling; no Meta report names this exact prompt (low to medium confidence).

</details>

<a id="q12"></a>

#### Q12. Train reward climbs while held-out accuracy falls. Read the RL log and the sample, find the cause, fix it

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Trace debug. Difficulty: Hard. Topic: Reward hacking in agentic RL.

```text
grader: reward = float(normalize(gold) in normalize(final_text))
step  train_reward  heldout_em  calls/traj  resp_tok  kl_k3  answers/traj
 100      0.41        0.38         3.2        910     0.01      1.0
 200      0.49        0.43         3.4        880     0.02      1.0
 300      0.55        0.46         3.3        850     0.04      1.1
 400      0.61        0.47         3.0        800     0.06      1.4
 500      0.68        0.44         2.4        610     0.11      2.3
 600      0.74        0.39         1.8        450     0.17      3.6
 700      0.80        0.33         1.3        320     0.26      5.2
 800      0.84        0.30         1.1        260     0.33      6.9
sample (step 800, gold "1947"):
  <tool_call>{"name": "search", "arguments": {"q": "year of independence"}}</tool_call>
  Final answer: 1945, 1946, 1947, 1948 or 1949
```

<details>
<summary>Show answer</summary>

Task: name the root cause from the evidence, stop the damage, fix the grader and prove the fix.

What they grade: reading several columns together, a precise cause, a fix that closes the hole rather than patching one symptom, and monitoring that would have caught it.

Approach: Train reward and held-out exact match agree until step 400 and then split: train reward keeps rising while held-out falls from 0.47 to 0.30. The answers-per-trajectory column climbs from 1.0 to 6.9 in step with the split, and the sample shows why: the grader checks whether the gold string appears anywhere in the final text, so listing five candidate years earns full reward. Searches fall from 3.2 to 1.1 because guessing a list is cheaper than looking things up, responses get shorter, and KL rises as the policy moves into this new mode. The held-out metric uses exact match, so it sees through the trick. I ask the assistant to tabulate the gap between train reward and held-out exact match per step and to draft grader unit tests, but I read the log myself. Its first grader fix normalized case and whitespace yet kept substring matching, so the five-year list still scored 1.0; the list test caught it.

Fix, in order: stop the run and keep the step 300 or 400 checkpoint; change the grader to extract exactly one answer from an answer tag and compare with exact match after normalization, scoring 0 when there are several candidates; rescore logged trajectories with the new grader to measure how much reward was fake; restart from the last good checkpoint.

Alternatives: an LLM judge that extracts one answer (handles free text, but adds cost and its own exploits) versus a strict answer format (cheap, exact, needs a format reward early on). I pick the strict format for short-answer tasks and keep the judge only for long-form answers.

Verify: unit tests for the grader with lists, hedges ("either 1946 or 1947"), and the gold answer inside a search query; a rerun where train reward and held-out move together past step 400.

Pitfalls: raising the KL coefficient, which slows the hack without removing it; trusting a single train-reward plot; tuning the grader on the held-out set.

Follow-ups: an alert when train reward and held-out diverge for two evals in a row; red-teaming every new reward with a policy trained to exploit it.

Why this is likely: reward hacking is a core failure in the RL runs this team owns, and Meta's Muse Spark post stresses RL stability and checks that RL gains generalize to a held-out set; no Meta report names this exact format (low to medium confidence).

</details>

<a id="q13"></a>

#### Q13. Add a thinking-length penalty without breaking hard problems, then explain an accuracy collapse after it was turned on

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Design. Difficulty: Hard. Topic: Reasoning efficiency in RL.

A reasoning model averages 9,000 thinking tokens on maths and agent tasks; product wants about a third of that. After a teammate added `reward -= 0.0001 * thinking_tokens`, average length fell to 2,500 tokens, easy-set accuracy held, and hard-set accuracy fell by 12 points. Design a better penalty and explain what happened.

<details>
<summary>Show answer</summary>

Task: a length term that cuts waste on easy problems while keeping long reasoning where it pays.

What they grade: the diagnosis, a reward that always ranks a correct answer above a wrong one, alternatives, and a way to measure the trade-off.

Approach: The flat penalty costs 0.9 reward at 9,000 tokens, almost as much as a correct answer is worth. On a hard problem where long thinking succeeds 40% of the time, the expected reward of thinking is 0.4 minus 0.9, which is negative, while a short guess that is almost always wrong scores about 0. The policy did the rational thing and stopped thinking on hard problems. The penalty applied to wrong answers too, so failing fast became the best option.

Better design: penalize length only on correct answers, relative to the group. For each prompt, take the shortest correct length in the group of samples; a correct answer gets 1 minus lambda times its excess length over that, capped so a correct answer always scores at least 0.5 and a wrong one 0. Hard problems, where few samples are correct, get little pressure; easy ones, where many short correct samples exist, get a lot. Warm lambda up over a few hundred steps. I ask the assistant to draft length_reward and its asserts, then check the bounds myself. Its first version still subtracted the penalty from wrong answers, the same bug as the original, and the assert that a wrong answer scores exactly 0 caught it.

Code:

```python
def length_reward(correct, length, group_correct_lengths, lam=0.5, cap=0.5):
    if not correct:
        return 0.0
    best = min(group_correct_lengths)
    return 1.0 - min(cap, lam * (length - best) / best)

assert length_reward(False, 100, [100]) == 0.0
assert length_reward(True, 100, [100, 300]) == 1.0
assert length_reward(True, 300, [100, 300]) == 0.5
assert min(length_reward(True, n, [50, n]) for n in (60, 500, 50_000)) >= 0.5
print("ok")
```

Alternatives: (a) a hard token budget with truncation counted as wrong (simple, but blunt across difficulty levels); (b) a group-relative length reward like the one in the Kimi k1.5 report (close to my design); (c) user-set thinking budgets learned by conditioning on a budget token (gives product control at inference). I pick the relative reward and add (c) later if product wants a dial.

Numbers: report accuracy against mean tokens per difficulty bucket before and after; the target is within 1 point on hard problems at less than half the tokens on easy ones.

Risks: the model hides reasoning in the answer section (count all generated tokens); length collapse later in training (keep the cap); groups with no correct answer get no length signal, which is intended.

Why this is likely: Meta's Muse Spark post describes a thinking-time penalty during RL that produced "thought compression", and the job asks for post-training work on frontier capabilities (medium confidence).

</details>

<a id="q14"></a>

#### Q14. Size an agentic RL run where tools are slow: GPU split, step time and synchronous versus asynchronous rollouts

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Design. Difficulty: Hard. Topic: RL infrastructure at scale.

An 8B policy is trained with GRPO on 64 H100 GPUs. Each step uses 256 prompts times 8 samples. A trajectory has 10 turns of about 300 generated tokens and 500 tool-output tokens, on a 1,000-token prompt. Tool calls take 2 seconds at the median and 20 seconds at p99. Estimate step time and pick an architecture.

<details>
<summary>Show answer</summary>

Task: back-of-envelope numbers for training, generation and tool waits, then the design that removes the bottleneck.

What they grade: numbers that add up, spotting the tail-latency trap, alternatives with their staleness costs.

Approach: Count tokens first. 2,048 trajectories of 9,000 tokens each; 6.1M of those tokens are generated. Training plus a reference forward costs about 8 times parameters times tokens: 8 x 8e9 x 2,048 x 9,000, about 1.2e18 FLOPs. At 40% of an H100's roughly 989 dense bf16 TFLOPS, 24 trainer GPUs need about 124 seconds. Generation at an assumed 3,000 decoded tokens per second per GPU on 40 rollout GPUs takes about 51 seconds of pure decoding. I ask the assistant to build this arithmetic table (tokens, FLOPs, seconds per phase), then I recheck each row by hand. It first used 6 times parameters times tokens, which drops the reference forward pass, and understated training time by a quarter; I corrected the factor to 8.

Tool latency dominates the step. If all trajectories advance turn by turn together, each turn waits for the slowest of 2,048 calls, and the chance that none of them hits the p99 tail is about 1 in a billion. So each of 10 turns waits about 20 seconds: 200 seconds of mostly idle GPUs. A synchronous step is about 51 + 200 + 124 = 375 seconds, and rollout GPUs are busy about 14% of the time.

Alternatives: (a) synchronous and on-policy: simple and exact, 375 seconds per step. (b) per-trajectory continuous batching with a 30-second tool timeout and asynchronous training one step behind, with the importance ratio against the behaviour policy already in the GRPO loss: rollouts for step k+1 overlap training of step k. (c) fully streaming with partial rollouts resumed under new weights: fastest, but more off-policy and harder to debug. I pick (b). Then rebalance to 32 and 32 GPUs: training about 93 seconds, decoding about 64 seconds plus tool overlap, so a step is about 95 seconds.

Numbers: about 230 steps a day synchronous versus about 900 asynchronous, close to 4 times the throughput on the same hardware.

Risks: staleness bias (cap it at one step and log the mean importance ratio); timeouts that hit hard prompts more often and bias the batch (log the timeout rate by prompt difficulty); tool servers that cannot handle 2,048 concurrent calls.

Why this is likely: the job asks for experience with the full post-training stack, and ML interview threads that include this lab describe reasoning about distributed inference and throughput (low to medium confidence).

</details>

<a id="q15"></a>

#### Q15. Post-train an assistant to use personal memory in tool calls without leaking it or over-personalizing, and show how you would measure both

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Design. Difficulty: Hard. Topic: Personalization and privacy in agents.

The assistant has a memory store of user facts (diet, family, health notes, home city) and tools that reach third parties (web search, restaurant booking, email). Design the data, rewards and evals so memory helps when it is relevant, stays out when it is not, and never flows to a third party without need.

<details>
<summary>Show answer</summary>

Task: training signals for three behaviours (use, restraint, privacy) and metrics for each.

What they grade: separating the failure modes, rewards that cannot be satisfied by ignoring memory, evals with paired controls, and statistics on rare events.

Approach: Three failures matter. Under-use: the memory says vegetarian and the booking ignores it. Over-personalization: a question about tax rules gets answered with the user's home city when it should not be. Leakage: a sensitive memory, like a health condition, appears in a web search query or an email to someone else.

Data: synthetic personas with 20 to 50 memory entries, each labelled with a sensitivity level, and tasks of three kinds: memory needed, memory irrelevant, memory present but must not leave the device. Rewards: task success from a state checker; a leak penalty when a high-sensitivity entry, or an entity extracted from it, appears in any outgoing tool argument; a relevance penalty when an irrelevant memory changes the answer, judged by comparing against a no-memory run.

Alternatives: (a) a hard rule layer that strips sensitive entities from tool arguments (exact and auditable, but blocks legitimate use such as booking a gluten-free table); (b) trained behaviour only (flexible, not guaranteed); (c) both, with the rule layer as a backstop and logging. I pick (c) and train the model so the backstop rarely fires. I ask the assistant to draft the Mermaid data-flow diagram and the canary sample-size math, and I set the sensitivity labels myself. It first sized the canary set at 300 trials for a 0.1% bound, ten times too few; I fixed it with 3 divided by 0.001.

Numbers: to claim a leak rate under 0.1% with zero observed leaks, the rule of three needs about 3,000 canary trials (3 divided by n). Over-personalization is measured on paired prompts with and without memory; target no change on at least 98% of irrelevant pairs. Helpfulness lift is the win rate of memory-on against memory-off on relevant tasks.

Risks: a judge that cannot tell useful from intrusive personalization (calibrate on human labels first); personas that are too clean compared with real memory, which is noisy and stale; users who want memory to flow (a consent flag in the tool schema).

Why this is likely: the job names personalization among the team's frontier capabilities, and Meta frames its lab's goal as personal superintelligence (medium confidence for the topic, low for the exact framing).

</details>

<a id="q16"></a>

#### Q16. Decide which SFT data to drop before online RL, write the filter, and design the ablation that proves it helped

Round: Research or ML design. AI policy: Provided (CoderPad assistant with Mermaid). Type: Concept plus code. Difficulty: Medium. Topic: Data curation for post-training.

You have 1M SFT examples and a plan for online RL afterwards. A colleague argues that heavy SFT on easy data makes RL worse. Decide what to keep, implement a pass-rate filter, and design the experiment that settles the argument.

<details>
<summary>Show answer</summary>

Task: a curation rule, a filter, and a fair ablation.

What they grade: reasoning about what SFT does to exploration, the zero-advantage problem in group RL, and an experiment with matched compute and seeds.

Approach: Heavy SFT on easy, uniform answers narrows the output distribution, so RL samples fewer diverse attempts and has less to reinforce. Llama 4's post-training removed more than half of the SFT data that Llama-model judges tagged as easy, ran a lightweight SFT on the rest, and then focused online RL on harder prompts. I follow the same idea: score each prompt by the current model's pass rate over 8 samples. Prompts at 100% teach SFT little and give GRPO zero signal; prompts at 0% give no RL signal either; keep the middle for RL and a small, diverse share of the easy set for SFT so formats stay stable.

With group size G and pass rate p, a group is all-same (zero advantage) with probability p to the G plus (1 minus p) to the G. At p = 0.9 and G = 8 that is about 43% of groups wasted; at p = 0.5 it is under 1%. I ask the assistant to write the filter and this waste formula, and I review the boundaries. Its first zero_signal returned only p to the G, which matches at p = 0.9 but calls a p = 0.1 prompt almost free of waste when about 43% of its groups are all wrong, so I added the (1 minus p) to the G term.

Code:

```python
def zero_signal(p, G):
    return p ** G + (1 - p) ** G

def split(pass_rates, lo=0.1, hi=0.9):
    keep_rl = [i for i, p in enumerate(pass_rates) if lo <= p <= hi]
    too_easy = [i for i, p in enumerate(pass_rates) if p > hi]
    too_hard = [i for i, p in enumerate(pass_rates) if p < lo]
    return keep_rl, too_easy, too_hard

assert abs(zero_signal(0.9, 8) - 0.4305) < 1e-4
assert zero_signal(0.5, 8) < 0.01
assert split([1.0, 0.5, 0.0, 0.875, 0.95]) == ([1, 3], [0, 4], [2])
print("ok")
```

Verify: ablation arms with the same total compute: (A) full SFT then RL, (B) filtered SFT then RL, (C) no SFT, only a format warm-up, then RL. Compare after RL, not after SFT, because B can look worse after SFT and better after RL. Use 3 seeds, held-out benchmarks, and pass@k at the end of SFT as a diversity check.

Pitfalls: judging easy by length or source instead of the model's own pass rate; measuring pass rates once and never refreshing them as the policy improves; comparing arms with different RL step counts.

Follow-ups: refresh the filter every N steps (dynamic sampling, as in DAPO); what to do with the too-hard bucket (curriculum or hints).

Why this is likely: Meta's Llama 4 post describes dropping more than half the easy SFT data before online RL, and the job asks for data curation strategies (medium confidence).

</details>

### Research deep dive (no AI, as at Meta)

<a id="q17"></a>

#### Q17. Defend your strongest post-training result against a skeptical interviewer: baselines, ablations and what failed

Round: Research presentation or deep dive. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Research rigor.

Example pushback: "Your method beats DPO by 3 points on a tool-use benchmark. How do I know that is not tuning budget, a weak baseline or one lucky seed? What did you try that did not work?"

<details>
<summary>Show answer</summary>

Task: show the claim survives the obvious attacks, and that you know where it stops.

What they grade: honest baselines, matched compute, seed variance, ablations that isolate the idea, failures you learned from, and how the work would scale with Meta's resources.

Approach: Build the talk around one claim and its scope, stated in a single sentence: "On multi-turn tool-use tasks, our reward shaping improves pass@1 by 3 points over DPO at equal compute, and the gain disappears on single-turn tasks." Then answer the obvious attacks inside the talk itself.

Alternatives (baselines I compared against and why): DPO with the same data and a tuned beta; online GRPO with the same reward; and the base model with prompting only. Each baseline got the same hyperparameter search budget, for example 12 runs, and I show the search grid. Reviewers check tuning budgets first, so every baseline gets the same search as your method.

Numbers: 3 seeds per arm, mean and spread, and a paired test over the same tasks. If the seed spread is 1.5 points and the gain is 3, say so: "consistent, but not large". Show compute in GPU-hours per arm. Show the ablation table: remove each component and report the drop.

Failures: pick one and tell it plainly, for example: "Our first version rewarded intermediate tool success; the policy learned to make easy, useless calls. We removed it and kept final-state reward plus a format term."

Scaling: say what changes at 10 to 100 times the compute. Which part breaks first: reward noise, data diversity or staleness? Which experiment would you run first at Meta?

Risks: a general claim resting on one benchmark (test a second one or narrow the claim); talk numbers that drift from the paper (check them line by line before the interview); thin related work (read the three closest papers and say how yours differs).

Why this is likely: recent Meta research scientist reports describe a research presentation and technical discussions about prior projects, and a recent job-search blog from a research scientist describes deep dives across labs (medium confidence).

</details>

<a id="q18"></a>

#### Q18. Plan a six-month research program for long-horizon tool use, with milestones and kill criteria

Round: Research presentation or deep dive. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Research planning and leadership.

The interviewer says: "Our agents fail on tasks that need more than about 30 tool calls. You lead two researchers and have a fixed GPU budget. What do you work on, in what order, and when do you stop?"

<details>
<summary>Show answer</summary>

Task: turn an open problem into staged bets with measurable gates.

What they grade: problem framing from data, prioritization, milestones with numbers, the courage to kill work, and leading a small team.

Approach: Spend month 1 on diagnosis. Collect 500 failed trajectories over 30 calls and label the first fatal error: lost state (forgot an earlier result), compounding small errors, bad planning, context overflow, or tool errors handled badly. The split between those errors decides where the budget goes.

Then three bets, ordered by expected value per GPU-hour. (1) Context management: train the model to write and read its own notes or summaries, so the working context stays under a fixed size. Cheap to test with SFT on synthetic traces. (2) Credit assignment over long horizons: process rewards from state checks at sub-goals versus outcome-only reward with more samples. (3) Curriculum: train on horizons of 10, then 20, then 40 calls, moving up when pass rate crosses 50%.

Alternatives: scaling context length alone (simple, but cost grows with length and models still lose track in long contexts); a planner and executor split into two models (clear roles, but harder to train end to end). I test notes first because the diagnosis usually shows lost state.

Numbers: milestones as pass@1 on a held-out set of 40 to 60 call tasks: from a baseline of 10% to 20% by month 3, 35% by month 6, with pass^4 reported alongside for reliability. Each bet gets about a third of the budget at first.

Kill criteria: written down before starting. A bet dies if it shows no gain beyond seed noise after 25% of its budget, or if its gain disappears on the live-tool eval. Then its budget moves to the leading bet.

Risks: the benchmark saturating before the real problem is solved (refresh it), and one researcher stuck on infrastructure (assign infra ownership explicitly).

Why this is likely: research scientist reports describe an open-ended research problem in the loop, and the job asks for long-horizon agentic tool use and technical leadership of complex projects (medium confidence).

</details>

### Behavioral (no AI, as at Meta)

<a id="q19"></a>

#### Q19. How do you use AI tools in your research and post-training work, and where do you refuse to trust them?

Round: Behavioral. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI-assisted research practice.

The interviewer wants one concrete story: a time an AI tool sped up your work, a time it was wrong, and the process you follow now.

<details>
<summary>Show answer</summary>

Task: one specific story with numbers, plus a rule you apply every time.

What they grade: real use, verification habits, judgment about where AI fails in research, ownership of results.

Approach: Use one situation, your task, your actions and the result, in about three minutes.

Example shape: "We needed an evaluation harness for 200 multi-turn tool-use tasks in a sandbox. I used a coding assistant to write the sandbox adapters, the result logging and the plots, which took two days instead of about a week. I wrote the state checkers myself, because they decide the result, and I asked the assistant to write tests against 20 recorded trajectories whose outcomes I had checked by hand. Three adapters failed. One reset the sandbox after the final tool call instead of before the checker ran, so every task scored as failed. Another accepted a partial order as complete, which inflated success by about 4 points. Neither looked wrong in the diff."

Then the rule. AI drafts boilerplate, tests, plots and literature leads. I write or re-derive anything that defines a result: reward functions, metrics, data splits, loss code. Every claim it makes about a paper gets checked against the paper. The team now keeps recorded trajectories with known outcomes as a regression suite for any AI-written eval code.

Verify: the story has a number for the time saved, a number for the error, and the change you made afterwards.

Pitfalls: claiming you avoid AI (Meta expects candidates to use the provided assistant); a story where the AI is never wrong; a vague "I always double-check".

Follow-ups: how you would use the CoderPad assistant in the AI-enabled round; how you review AI-written reward code before an RL run.

Why this is likely: candidate summaries report that Meta's loop now includes an AI-oriented behavioral round, and Meta's careers FAQ says candidates are expected to use the provided assistant (medium confidence).

</details>

## Prep checklist

- [ ] Solve grid BFS, stack parsing and lowest common ancestor problems in 20 minutes each, out loud, with no AI.
- [ ] Practise one multi-file CoderPad-style task: run the tests first, prompt the assistant, then review every line of its diff.
- [ ] Hand-write the DPO loss, GRPO advantages with a clipped loss, and causal attention with a KV cache in NumPy or PyTorch.
- [ ] Be able to explain the KL term in RLHF and the k1 and k3 estimators on a whiteboard.
- [ ] Prepare a 3-minute and a 20-minute version of your strongest post-training paper, including one failed experiment.
- [ ] For each paper, write down the baseline and ablation a skeptical reviewer would ask for, and whether you ran it.
- [ ] Rehearse one RL design for a tool-use or search agent with numbers for rollouts, tokens, tool latency and GPU-hours.
- [ ] Know the unbiased pass@k estimator, pass^k, and how wide a confidence interval is on a 300-task benchmark.
- [ ] Read Meta's Muse Spark and Llama 4 posts and form one opinion you can defend on each post-training choice.
- [ ] Prepare one AI-use story with a measured time saving and a caught error.
- [ ] Ask the recruiter which rounds are AI-enabled and whether there is a research presentation.

## Sources

- Meta careers, hiring process and FAQ on AI-enabled interviews: https://www.metacareers.com/hiring-process
- Glassdoor, Meta research scientist interview reports: https://www.glassdoor.com/Interview/Meta-Research-Scientist-Interview-Questions-EI_IE40772.0,4_KO5,23.htm
- 1point3acres, Meta interview trend report: https://www.1point3acres.com/interview/guides/meta-interview-trend-2026q4
- Reddit, Meta Superintelligence Labs senior MLE interview experience (low confidence, summary only): https://www.reddit.com/r/OfferEngineering/comments/1unndt6/meta_msl_jun_2026_senior_mle_interview_experience
- Reddit, Meta interviews with AI coding: https://www.reddit.com/r/OfferEngineering/comments/1wubnl2/meta_swe_interviews_in_2026_ai_coding
- First-hand research scientist job search blog: https://yongzx.github.io/blog/2026/06/24/job-search/
- Blind, Meta ML research scientist interview thread: https://www.teamblind.com/post/meta-machine-learning-research-scientist-interview-passedoral-offer-received-but-i-am-confused-help-please-fqu8o6d5
- Blind, ML and AI interview thread (low confidence): https://www.teamblind.com/post/been-grinding-mlai-interviews-for-the-past-6-months-oratewjm
- PracHub, Meta machine learning engineer questions (low confidence): https://prachub.com/companies/meta/positions/machine-learning-engineer
- Meta AI, Introducing Muse Spark: https://ai.meta.com/blog/introducing-muse-spark-msl
- Meta AI, Llama 4 post-training: https://ai.meta.com/blog/llama-4-multimodal-intelligence
- Cameron Wolfe, Llama 4 analysis: https://cameronrwolfe.substack.com/p/llama-4
- Company-wise LeetCode lists (crowd-reported tags): https://github.com/liquidslr/leetcode-company-wise-problems
