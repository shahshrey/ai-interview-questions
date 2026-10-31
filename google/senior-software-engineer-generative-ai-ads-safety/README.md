# Google: Senior Software Engineer, Generative AI, Ads Safety

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop is assembled from public hiring guides and 2025 to 2026 candidate reports for Google L5 and ML-track SWE roles. The loop for this team may differ.

1. Recruiter screen (about 30 minutes). Level fit, ML track, and production impact with numbers such as latency, dataset size, and precision gains. Candidates report being downleveled from L5 to L4 after the screen, so level comes up early.
2. Technical phone screen (45 minutes). One medium or medium-hard algorithm problem in a shared doc with no autocomplete, plus a follow-up that adds a constraint, often a streaming or real-time version of the same problem.
3. Onsite, 4 to 5 rounds of 45 minutes, in person in most cases according to the listing.
   - Two coding rounds at the standard SWE L5 bar. Reports mention graphs, BFS and shortest paths with obstacles, sliding windows, heaps, intervals, and DP, with follow-ups that change the constraints.
   - One ML system design round. Expect a moderation or classification pipeline, with pressure on serving cost, latency, feedback loops, and refresh. Model architecture gets less of the time.
   - One ML domain round. For this team, expect depth on distillation, multimodal video inputs, embeddings and near-duplicate detection, calibration, and LLM-based evaluation.
   - One Googleyness and Leadership round.
4. Hiring committee review of the full packet.
5. Team matching. Because the listing names the team, the match call will likely go deep on teacher models and agentic evaluation.

## Questions and answers

### Coding

#### Q1. Given a stream of policy-violation events, each a (violation_type, timestamp) pair, return after every event the most frequent violation type among the last N events.

Problem. Implement `add(key)` that appends an event to a window holding the most recent N events, evicts the oldest when the window is full, and returns a key with the highest count in the window. Ties may return any key with the maximum count.

The naive approach rescans the counts on every eviction, which costs O(distinct keys) per event. The fix is a count-to-keys bucket map. A sliding window changes one count by +1 and one by -1 per event, so the maximum can only rise by one or fall by one, and it falls only when the last key at the old maximum leaves that bucket.

```python
from collections import defaultdict, deque

class WindowMode:
    def __init__(self, n):
        self.n = n
        self.window = deque()
        self.count = defaultdict(int)     # key -> count in window
        self.bucket = defaultdict(set)    # count -> keys with that count
        self.max_count = 0

    def _move(self, key, delta):
        c = self.count[key]
        if c:
            self.bucket[c].discard(key)
            if not self.bucket[c]:
                del self.bucket[c]
        c += delta
        if c:
            self.count[key] = c
            self.bucket[c].add(key)
        else:
            del self.count[key]
        if delta > 0:
            self.max_count = max(self.max_count, c)
        elif self.max_count not in self.bucket:
            self.max_count -= 1           # can only drop by one

    def add(self, key):
        self.window.append(key)
        self._move(key, +1)
        if len(self.window) > self.n:
            self._move(self.window.popleft(), -1)
        return next(iter(self.bucket[self.max_count]))
```

Complexity. O(1) amortized per event and O(N) memory. The increment happens before the eviction, so the window briefly holds N + 1 items and the bucket for the true maximum is never empty when `add` returns.

Edge cases. N = 1, a window made of one repeated key, every key distinct (the maximum stays at 1), and many tied keys. If the interviewer wants deterministic ties such as the most recent key, keep an ordered structure per bucket.

Follow-ups. Change the window from count-based to time-based (evict while `ts < now - T`) and the same structure works, since evictions are still -1 moves. If the interviewer asks for top k instead of top 1, walk buckets downward from `max_count`. For a sharded stream too large for one machine, keep a count-min sketch per time slice with a small heap of heavy hitters, and accept the sketch's overcount bound of about e/w times the stream length per cell.

Why this is likely: a 2025 LeetCode Discuss report of a Google SWE III ML coding round gave this exact shape, most frequent event with timestamps followed by a fixed-size real-time queue follow-up.

#### Q2. Given L2-normalized embeddings for N ad creatives, group them into near-duplicate clusters where two ads are linked if cosine similarity is at least t. Do it without comparing all pairs.

Problem. Return the connected components of the graph whose edges are pairs with cosine similarity of at least t. N is large enough that O(N^2) comparisons are too slow.

Use random-hyperplane LSH to generate candidate pairs, confirm each candidate with the exact cosine, and merge with union-find. Each hyperplane bit agrees for two vectors with probability 1 - theta/pi, where theta is the angle between them. With b bands of r bits, a pair becomes a candidate with probability 1 - (1 - p^r)^b.

```python
import math, random
from collections import defaultdict

class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]   # path halving
            x = self.parent[x]
        return x
    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        return True

def cosine(u, v):
    dot = sum(a * b for a, b in zip(u, v))
    nu = math.sqrt(sum(a * a for a in u)); nv = math.sqrt(sum(b * b for b in v))
    return dot / (nu * nv) if nu and nv else 0.0

def near_dup_clusters(vecs, threshold=0.9, bands=16, rows=8, seed=0):
    rng = random.Random(seed)
    dim = len(vecs[0])
    planes = [[rng.gauss(0, 1) for _ in range(dim)] for _ in range(bands * rows)]
    dsu = DSU(len(vecs))
    buckets = defaultdict(list)
    for i, v in enumerate(vecs):
        bits = [1 if sum(p * x for p, x in zip(pl, v)) >= 0 else 0 for pl in planes]
        for b in range(bands):
            buckets[(b, tuple(bits[b * rows:(b + 1) * rows]))].append(i)
    checked = set()
    for members in buckets.values():
        for x in range(len(members)):
            for y in range(x + 1, len(members)):
                a, b = members[x], members[y]
                if (a, b) in checked or dsu.find(a) == dsu.find(b):
                    continue
                checked.add((a, b))
                if cosine(vecs[a], vecs[b]) >= threshold:
                    dsu.union(a, b)
    groups = defaultdict(list)
    for i in range(len(vecs)):
        groups[dsu.find(i)].append(i)
    return list(groups.values())
```

Complexity. Hashing costs O(N * b * r * d). Verification costs the sum of squared bucket sizes, and union-find is near O(1) amortized per operation. With b = 16 and r = 8, a pair at cosine 0.9 becomes a candidate with probability about 0.996, a pair at 0.5 about 0.47, and an orthogonal pair about 0.06. Raise r to cut false candidates and raise b to recover recall.

Edge cases. Zero vectors, N = 0 or 1, and giant buckets from template ads. Cap bucket size or split hot buckets with more bits.

Follow-ups. Transitive chaining can merge unrelated ads through a path of near neighbors, so add a check against the cluster centroid or limit cluster diameter. For streaming, hash each new ad, compare it only against its buckets, and union on the fly.

Why this is likely: candidate reports say graph and union-find problems dominate Google coding rounds, and Google's Ads Safety papers rely on similarity graphs and dedup over embeddings.

#### Q3. Ad videos arrive for human review as [start, end) time intervals of reviewer occupancy. Return the minimum number of reviewers needed so that no reviewer handles two overlapping tasks, plus a valid assignment of reviewer IDs to tasks.

Problem. This is the Meeting Rooms II pattern with an extra output, the assignment. A task that ends at time t frees its reviewer for a task starting at t.

Sort tasks by start time. Keep a min-heap of (end_time, reviewer_id) for busy reviewers and a min-heap of free reviewer IDs. Before each task, release every reviewer whose end time is at or before the task's start.

```python
import heapq

def assign_reviewers(tasks):
    order = sorted(range(len(tasks)), key=lambda i: tasks[i][0])
    busy = []              # (end_time, reviewer_id)
    free = []              # reusable reviewer ids
    assignment = [-1] * len(tasks)
    next_id = 0
    for i in order:
        start, end = tasks[i]
        while busy and busy[0][0] <= start:
            _, rid = heapq.heappop(busy)
            heapq.heappush(free, rid)
        if free:
            rid = heapq.heappop(free)
        else:
            rid, next_id = next_id, next_id + 1
        assignment[i] = rid
        heapq.heappush(busy, (end, rid))
    return next_id, assignment
```

For tasks [(0, 30), (5, 10), (15, 20), (10, 15)] this returns 2 reviewers with assignment [0, 1, 1, 1].

Complexity. O(n log n) time and O(n) space. The answer equals the maximum number of overlapping intervals at any point, which is also computable with a sweep over +1 and -1 events sorted with ends before starts at equal timestamps.

Edge cases. Empty input, zero-length tasks, identical intervals, and the touching case where end equals start. State the half-open convention out loud before coding.

Follow-ups. A fixed number of reviewers K with tasks that wait until someone is free (Meeting Rooms III) needs a second heap keyed by (free_time, id) and delayed task starts. A second variant asks you to merge flagged segments within one video when they are within g seconds of each other. Sort by start and extend the last segment while `start <= last_end + g`, which is O(n log n).

Why this is likely: 2025 LeetCode Discuss Google onsite reports list interval and heap problems, and a Google onsite thread for Meeting Rooms 3 shows this family recurs with follow-ups.

#### Q4. A teacher model can only afford k frames per ad video. Each candidate frame has a timestamp and a saliency score. Pick at most k frames, any two at least d seconds apart, to maximize total saliency.

Problem. Given times[i], scores[i], k, and d, return the maximum total score and the chosen frames. Greedy by score fails because one high-score frame can block two neighbors whose sum is larger.

This is weighted interval scheduling with a cardinality limit. Sort by time. For frame i, let prev[i] be the number of frames with time at most times[i] - d, found by binary search. Then dp[j][i] = max(dp[j][i-1], dp[j-1][prev[i]] + score[i]).

```python
import bisect

def select_frames(times, scores, k, d):
    n = len(times)
    order = sorted(range(n), key=lambda i: times[i])
    t = [times[i] for i in order]
    s = [scores[i] for i in order]
    prev = [bisect.bisect_right(t, t[i] - d) for i in range(n)]
    dp = [[0.0] * (n + 1) for _ in range(k + 1)]
    take = [[False] * (n + 1) for _ in range(k + 1)]
    for j in range(1, k + 1):
        for i in range(1, n + 1):
            skip = dp[j][i - 1]
            pick = dp[j - 1][prev[i - 1]] + s[i - 1]
            if pick > skip:
                dp[j][i], take[j][i] = pick, True
            else:
                dp[j][i] = skip
    chosen, j, i = [], k, n
    while j > 0 and i > 0:
        if take[j][i]:
            chosen.append(order[i - 1])
            i, j = prev[i - 1], j - 1
        else:
            i -= 1
    return dp[k][n], sorted(chosen, key=lambda x: times[x])
```

Complexity. O(n log n + nk) time and O(nk) space. If you only need the value, keep two rows and drop to O(n) space. This solution was checked against brute force on random inputs.

Edge cases. n = 0, k larger than n, duplicate timestamps (only one can be chosen if d > 0), negative scores (never chosen because skipping is allowed), and d = 0.

Follow-ups. If k is large, drop the cardinality dimension with a Lagrangian penalty on each pick and binary search the penalty. For coverage instead of raw saliency, such as at least one frame per scene, add scene IDs and a constraint, or switch to a submodular greedy with its 1 - 1/e guarantee.

Why this is likely: candidates report DP problems in Google onsites, and the listing's multimodal teacher models for video make frame budgeting a natural domain wrapper for a standard DP.

### ML system design

#### Q5. Design an LLM review system that finds policy-violating image ads across all traffic before they reach the auction, when you can afford LLM calls on only a tiny fraction of ads.

Requirements. Scan every new and active creative, aim for high precision on enforcement, raise recall on one policy (for example Non-Family Safe), and keep LLM calls within a fixed daily budget. As an assumption, plan for hundreds of millions of creatives per day against an LLM budget around one million calls.

Components and data flow, following the structure of Google's published approach.
1. Candidate selection. Pull candidates using content similarity to known violators (an embedding graph over past human or model labels), actor similarity (other creatives from accounts with violations), and cheap non-LLM classifier scores. Candidate selection can run at low precision because it only feeds the next stage.
2. Volume reduction. Cross-day dedup removes anything the LLM has already reviewed, intra-batch dedup by hash removes copies, and inactive or already-labeled creatives are filtered out. Cluster the rest and pick one representative per cluster using diversity-maximizing sampling over the similarity graph.
3. LLM labeling. A prompt-engineered or parameter-efficiently tuned multimodal LLM labels the representatives.
4. Label propagation. Push the label to cluster members whose similarity to the representative clears a stricter threshold than the clustering threshold.
5. Feedback. Newly labeled positives become seeds for the next round of candidate selection, so coverage grows over time.

The hard parts. The representation sets the ceiling, and the paper reports cross-modal embeddings beating unimodal ones for clustering and propagation. Propagation is where precision is lost, so measure it separately on an audited sample of propagated labels. Over-merged clusters spread one bad decision widely, so cap cluster size or require a second representative for large clusters.

Numbers to quote. The published system reduced LLM reviews by more than three orders of magnitude and doubled recall against a non-LLM baseline. In budget terms, if 200 million candidates collapse to 400 thousand clusters, a one-million-call budget covers every cluster and leaves room for a second representative on the large ones.

Tradeoffs. Tight propagation thresholds keep precision but cost coverage. Daily batch rounds are cheap, but violations stay live until the next round. Pre-auction blocking needs a fast path, so newly created ads close to a known violator should go straight to review.

Why this is likely: Google's WSDM 2024 paper "Scaling Up LLM Reviews for Google Ads Content Moderation" describes this exact pipeline, and the listing asks for "high-scale data pipelining and funneling."

#### Q6. A new scam pattern appears today. Design a system that enforces against it within hours, without waiting to collect labels and retrain a classifier.

Requirements. Go from a handful of confirmed examples to live enforcement the same day, hold precision at least at the level of the existing classifier, and make the result reversible.

Design. Use retrieval-augmented in-context learning, the approach Google describes for enforcing new violation patterns in O(1) days.
1. Example store. Index embeddings of labeled ads (violating and benign) together with reviewer rationales and policy passages. Adding the new pattern means inserting a few dozen labeled examples, with no training run.
2. Retrieval. For each candidate ad, retrieve the top k nearest labeled examples plus the relevant policy text. Retrieval must cover in-scope and out-of-scope neighbors, otherwise the model learns a one-sided prompt.
3. LLM classification. The prompt holds the policy, the retrieved examples with their rationales, and the ad, and it asks for a label, a confidence, and the evidence. Google's RAG moderation paper reports better memorization and generalization than parametric and non-parametric baselines when reasoning information is added to the examples.
4. Rollout. Run in shadow mode against a golden set and recent traffic. Turn on enforcement for the high-confidence band and route the middle band to reviewers. Every reviewer decision goes back into the example store, so the system improves within the day.
5. Hand-off. Once a few thousand labels exist, distill into a cheap classifier and keep the RAG path for the newest patterns.

The hard parts. Retrieval misses are silent failures, so track the share of positives whose nearest neighbors include at least one positive. Adversaries probe new paths quickly, so refresh the store continuously. Bad examples poison many decisions, which means every example needs provenance and a way to remove it.

Numbers. Retrieve k = 8 to 16 examples. A 30-second video ad costs about 3,000 tokens on Gemini 3 defaults (70 tokens per frame at 1 fps plus 32 audio tokens per second), so run RAG only on the output of a candidate funnel.

Tradeoffs. RAG is fast to launch and easy to undo, but each call costs more and the results depend on which examples come back from retrieval. Fine-tuning gives cheaper, more consistent calls and takes longer to ship.

Why this is likely: Google's CIKM 2025 paper "Google Ads Content Moderation with RAG" targets O(1)-day enforcement, which matches the team's stated goal of cutting enforcement latency from days to minutes.

#### Q7. Design the pipeline that uses a multimodal LLM teacher to label ad videos at scale and trains the production student classifiers from those labels.

Requirements. Cover tens of policies across millions of new video ads per day. The student must score every video within a tight latency budget. The teacher runs offline on a sampled and prioritized subset.

Components.
1. Preprocessing. Decode once and extract shot boundaries, keyframes (fixed rate plus scene-change frames), ASR transcript, OCR text, audio events, and landing-page text. Cache everything by content hash so each re-encode or duplicate is processed once.
2. Teacher sampling. Choose videos for teacher labeling by stratifying across policy, locale, and format, then upweight uncertain student scores, new advertisers, and videos near known violators. Uniform sampling wastes the teacher on easy negatives at a 0.1% base rate.
3. Teacher inference. A Gemini-class model gets the policy text, few-shot examples, and the media, and returns per-policy labels, a probability, evidence timestamps, and a short rationale, as structured JSON with schema validation.
4. Label store. Version labels by teacher model, prompt version, and policy version so any student can be traced to the labels it learned from.
5. Student training. Train a compact video encoder with per-policy heads on teacher soft labels plus human labels, and calibrate each head after training.
6. Gates. Measure the teacher and the student against expert golden sets per policy and locale. Promote a model only when every slice holds steady or improves.

Numbers. On Gemini 2.x defaults, video costs about 300 tokens per second at 258 tokens per frame at 1 fps. On Gemini 3 defaults it is about 102 tokens per second (70 per frame plus 32 audio). Labeling 500 thousand 30-second videos per day comes to about 1.5 billion input tokens per day, so batch it and use a Flash-tier teacher for most labels and a Pro-tier teacher only for disagreements.

The hard parts. Frame sampling at 1 fps misses sub-second flashes, so add scene-change frames or raise fps for short ads. Teacher rationales must point at real evidence, so spot-check the cited timestamps. Teacher upgrades shift the label distribution, which means every teacher change triggers relabeling of a regression slice.

Tradeoffs. One teacher per policy gives better accuracy, while a shared teacher is cheaper to maintain. More frames buy recall at linear cost in tokens.

Why this is likely: the listing explicitly asks for "multi-modal LLM teacher models for video understanding for ads at scale," and Google's video understanding docs publish the per-frame token costs that drive this budget.

#### Q8. Design a toolkit where AI agents run model evaluation and prompt optimization for safety classifiers with no human on the critical path, and keep it from shipping regressions.

Requirements. Agents propose prompt or model changes, evaluate them, and promote winners automatically. Humans set the gates and audit the results, and individual changes go through without a human approval step. Refresh triggers include a schedule, drift alarms, and policy updates.

Components.
1. Optimizer agent. An LLM proposes prompt edits in the style of OPRO, conditioned on past candidates and their scores, plus error categories from misclassified examples. The Advertiser Content Understanding paper describes manual error binning, and the agent automates that step.
2. Data splits. A tuning set the agent can see, a gating set it never sees, and a fresh canary slice drawn from recent traffic. Rotate the gating set on a schedule and hash-check that no gating item appears in prompts or few-shot pools.
3. Evaluator. Compute per-slice precision, recall, and calibration for each policy, locale, format, and advertiser segment, with bootstrap confidence intervals. LLM-judge scores count only after their agreement with experts has been measured per slice.
4. Gates. Promote only if the lower confidence bound on precision stays above target on every slice, recall stays within a fixed delta of the current model, and cost and latency stay within budget.
5. Deployment. Shadow, then a 1% canary, then staged rollout, with automatic rollback on appeal overturn rate or audit precision.
6. Audit log. Record every candidate, its diff, its scores, and who or what approved it.

The hard parts. The agent overfits the tuning set, so cap its search budget and count every gating-set evaluation against a fixed allowance. Multiple comparisons across many candidates inflate false wins, so apply a correction or confirm on a fresh sample. Drift signals are noisy, so require two signals before an automatic retrain.

Numbers. At 1% prevalence, a gating slice needs about 1,000 flagged items to estimate precision within roughly plus or minus 2 to 3 points at 95% confidence, which means sampling around 100 thousand items or enriching with a funnel and reweighting.

Tradeoffs. More automation shortens enforcement latency and raises the cost of a bad gate. Gate definitions therefore stay human-owned while candidate changes flow automatically.

Why this is likely: the listing asks for agents that "automate model evaluation and prompt optimization" and remove humans from the critical path, and OPRO is a Google DeepMind method.

### Domain depth

#### Q9. Explain knowledge distillation precisely. How would you distill a multimodal LLM teacher into a video classifier, and where does it fail?

Classic distillation trains the student on a mix of hard labels and the teacher's softened distribution:

L = alpha * CE(y, softmax(z_s)) + (1 - alpha) * T^2 * KL(softmax(z_t / T) || softmax(z_s / T))

The temperature T > 1 exposes the teacher's relative probabilities across wrong classes, which carry more information per example than a one-hot label. The T^2 factor keeps gradient magnitudes comparable as T changes, because the soft-target gradient scales as 1/T^2. For binary policy heads, the equivalent is binary cross-entropy against the teacher's probability.

An LLM teacher usually returns text without exposing logits, so you build soft targets in one of three ways: from token probabilities of the label tokens, from agreement across several sampled answers, or from a verbalized confidence that has been calibrated afterward. Google's distilling step-by-step adds a second objective, L = L_label + lambda * L_rationale, where the student also generates the teacher's rationale, and this let a much smaller student match or beat the teacher with less data. For a video classifier without a text decoder, use rationales differently. Turn the evidence timestamps into auxiliary frame-level targets, or distill into intermediate features.

The transfer data matters more than the loss. Sample it by teacher uncertainty and policy rarity, because a 0.1% base rate means random samples teach the student almost nothing about positives. Add human labels for slices where the teacher is known to be weak.

Failure modes.
- Copied errors. The student matches the teacher's mistakes, so always evaluate against expert labels and treat teacher agreement as a secondary signal.
- Capacity gap. A much larger teacher can produce targets the student cannot represent. A medium teacher-assistant or higher T can help.
- Confirmation loops. If the student's scores pick which items the teacher labels next, blind spots never get labeled. Keep a random exploration slice.
- Calibration. Soft-label training and class reweighting shift probabilities, so recalibrate each head.
- Staleness. Adversaries adapt faster than refresh cycles.

Why this is likely: the listing pairs teacher models with production-scale classifiers, and candidates report that Google ML domain rounds probe distillation math and failure modes.

#### Q10. How do multimodal LLMs encode video, and what does that mean for detecting violations in short ad videos?

Gemini-style models turn video into a sequence of tokens. Frames are sampled at a fixed rate, 1 fps by default in the public API. A vision encoder turns each frame into a fixed number of tokens: 258 per frame at default resolution on Gemini 2.x (66 at low), and 70 per frame by default on Gemini 3 (280 at high). Audio is tokenized separately at about 32 tokens per second, and timestamps are interleaved so the model can refer to moments. The tokens enter the same transformer context as the text prompt, so cost and attention scale with duration times frame rate times tokens per frame.

What this means for ads.
- Temporal aliasing. At 1 fps, a 300 ms flash of prohibited content can fall between samples. Raise fps for short ads, add frames at shot boundaries found by histogram or embedding deltas, or run a cheap per-frame detector at full frame rate and send only flagged spans to the LLM.
- Resolution tradeoff. Low resolution saves about 4 times the tokens but loses small on-screen text such as disclaimers, phone numbers, and crypto wallet addresses. Run OCR separately and pass the text in.
- Cross-modal violations. Many policy violations exist only in combination. The visuals can be benign while the voice-over makes a health claim, or the audio can be clean while on-screen text promises guaranteed returns. Keep audio and the ASR transcript in context, because frame-only students miss these.
- Context length. A 30-second ad costs about 3,000 tokens on Gemini 3 defaults and about 9,000 on Gemini 2.x defaults, which sets teacher throughput.

Student encoders differ. Production students typically use a frame encoder with temporal pooling or a small video transformer over 8 to 32 frames, plus text and audio towers combined by late fusion. Early fusion catches cross-modal interactions but costs more. Late fusion is cheap and modular, but misses a claim that only means something in light of the visuals.

Failure modes. Adversarial edits such as frame insertion and mirroring, letterboxing, and padding a short violation inside a long benign video. Evaluate recall on short-duration violations as its own slice.

Why this is likely: the listing names "multi-modal LLM teacher models for video understanding," and Google's public video understanding docs publish the sampling rates and token counts this question depends on.

#### Q11. Walk through near-duplicate detection for ad creatives with embeddings and LSH. How do you pick the hash parameters and the similarity threshold?

Exact copies are caught by cryptographic hashes and light edits by perceptual hashes (pHash, dHash). Adversarial variants such as crops, overlays, re-encodes, paraphrased text, and recut video need learned embeddings plus approximate nearest neighbor search.

LSH mechanics. For cosine similarity, random-hyperplane LSH sets h(x) = sign(w . x) with w drawn from a Gaussian, and P[h(x) = h(y)] = 1 - theta/pi. Concatenate r bits into a band and use b bands. A pair becomes a candidate when any band matches, with probability

P(candidate) = 1 - (1 - p^r)^b

which forms an S-curve. With r = 8 and b = 16, cosine 0.9 (p about 0.856) gives about 0.996, cosine 0.5 gives about 0.47, and orthogonal vectors give about 0.06. Raising r sharpens rejection of dissimilar pairs, and raising b recovers recall. For Jaccard similarity on token shingles of ad text, MinHash plays the same role with P = J. At production scale, graph-based or quantized ANN indexes such as ScaNN usually replace raw LSH, but the recall and cost tradeoffs are the same.

Threshold selection. Build a labeled set of pairs (same campaign variant, adversarial variant, and same vertical but a different business) and plot precision and recall of "is near-duplicate" against similarity. Choose two thresholds, a looser one for retrieval and candidate generation and a strict one for propagating an enforcement label, because a false propagation lands on a real advertiser. Google's scaling paper uses a tighter similarity for propagation than for candidate selection.

Embedding choice. Cross-modal embeddings beat unimodal ones for this task in Google's published results. Image-only similarity merges different ads that share a stock template, while text-only similarity misses a reused image with new copy.

Failure modes.
- Hub ads. Template creatives sit near thousands of others, so cap neighbors and normalize by local density.
- Transitive chaining. A to B to C links unrelated ads, so check against the cluster centroid.
- Embedding version drift. Changing the encoder invalidates the index and the thresholds, so re-index and re-tune together.
- Evasion. Adversaries optimize against your embedding, so combine content similarity with actor signals such as shared payment instruments, domains, and creation timing.

Why this is likely: dedup, clustering, and similarity-based propagation sit at the center of Google's Ads Safety LLM papers, and candidate-reported Google ML design prompts include near-duplicate video detection.

#### Q12. Your violation classifier will run on traffic with a 0.1% base rate. How do you calibrate it and choose an enforcement threshold?

Start from the base rate. With prevalence pi = 0.001, recall 0.9, and a false positive rate of 0.5%, precision = 0.9 * 0.001 / (0.9 * 0.001 + 0.005 * 0.999), which is about 15%. ROC-AUC looks excellent here and hides the problem, so report precision-recall curves and precision at fixed recall on the natural distribution.

Calibration methods.
- Platt scaling fits p = sigmoid(A * s + B) on held-out data. It has two parameters, so it is stable on small sets and assumes a sigmoid-shaped distortion.
- Isotonic regression fits a monotone step function. It is more flexible but needs more data and overfits sparse tails, which is a real risk with few positives.
- Temperature scaling divides logits by one learned T. It preserves ranking and is standard for neural nets, which tend to be overconfident.

Prior correction. If you trained with negatives downsampled at keep-rate w, the model's odds are inflated by 1/w. Correct with p = q / (q + (1 - q) / w). Class weights distort probabilities in the same way, so recalibrate on data that reflects the true prevalence.

Measuring. Use reliability diagrams and ECE with equal-mass bins (equal-width bins put nearly all items in the first bin at this prevalence), plus Brier score and log loss. Check calibration per slice too, since a globally calibrated model can be badly off for one locale or format.

Choosing the threshold. With calibrated p and known costs, enforce when C_FN * p > C_FP * (1 - p), so t* = C_FP / (C_FP + C_FN). In practice ads enforcement uses bands: auto-enforce above the threshold where the lower 95% confidence bound of audited precision meets the policy target (for example 95% or higher), send the middle band to human review sized to reviewer capacity, and auto-allow below, with a random audit sample from every band. Choose thresholds per policy, because harm and reversibility differ.

Failure modes. Prior shift during scam waves moves the threshold's real precision, so monitor score distributions and audited precision on a schedule. Delayed labels such as appeals bias recent estimates. Refreshing the model silently changes what any fixed threshold means, so re-fit the calibrator with every refresh.

Why this is likely: rare-positive thresholding is a standard ML domain probe in candidate-reported guides, and Google's 2025 Ads Safety Report highlights an 80% drop in incorrect advertiser suspensions, which is a precision and threshold result.

#### Q13. Explain zero-shot policy classification with text descriptions and image-text co-embeddings. Why would Ads Safety use it, and how do you keep it precise?

Mechanism. A contrastively trained image-text model maps images and text into one embedding space, where the cosine similarity of matching pairs is high. To build a classifier without labeled training data, write a set of text descriptions of the policy's modes, such as "lingerie product photo on a model" or "suggestive pose with partial nudity." Embed them, embed each incoming ad image, and score the ad by its similarity to the nearest in-scope description, minus or gated by its similarity to out-of-scope descriptions such as "swimwear in a sports context." This is CLIP-style zero-shot classification, extended with explicit negative descriptions.

Google's published version adds three pieces.
1. LLM-assisted description generation. An LLM breaks the policy into specific visual signals, suggests blind spots, and captions known violating images to produce more descriptions. Domain experts add their own.
2. Validation. For each description, experts review the nearest retrieved images and mark the description in-scope or out-of-scope. Then descriptions are matched against a labeled corpus, and in-scope descriptions that mostly retrieve benign images are removed, as are out-of-scope descriptions that mostly retrieve violating ones.
3. Enforcement. Fast approximate kNN matches each image against all descriptions, and an LLM reviews the nuanced cases.

The paper reports 90.8% precision against 89.1% for the supervised baseline, with relative recall of 63.4% against 48.2%.

Why use it. A new classifier becomes as cheap as writing descriptions, with no training run. One ANN index serves many policies, and iteration takes hours.

Keeping it precise.
- Calibrate per description. Raw cosine scores are not probabilities, and each description has its own score distribution, so set thresholds per description on a labeled sample.
- Watch for over-broad descriptions. One vague description can fire on a whole vertical, so track each description's precision and retire weak ones.
- Text bias. Contrastive models react to rendered text in the image, so an ad with the word "sexy" can match without any visual content. Pair with OCR-aware checks.
- Encoder updates invalidate every threshold, so re-validate the full description set when the encoder changes.

Why this is likely: Google's WSDM 2025 paper "Zero-Shot Image Moderation in Google Ads with LLM-Assisted Textual Descriptions and Cross-modal Co-embeddings" comes from the same Ads Safety group this role sits in.

#### Q14. When should a policy decision be made at the advertiser level instead of the ad level, and how would you build an LLM classifier over advertiser profiles?

Ad-level classifiers lack context. "Intimate collection now 40% off" is ambiguous on its own, and clear once you know the advertiser is a mainstream department store or a known adult retailer. Bad actors exploit ad-level inconsistency by submitting many near-identical variants and hoping some slip through. Advertiser-level classification fixes both problems. It removes over-flagging for good advertisers and makes enforcement against bad ones consistent.

Pipeline, following Google's published approach.
1. Candidate selection. Running an LLM over every advertiser is too expensive, so select advertisers whose ads are likely over-flagged by ad-level classifiers, or under-flagged for the reverse task.
2. Content summarization. An advertiser can have thousands of ads. Keep the most relevant ones, already-labeled ones, and known false positives.
3. Feature aggregation. Combine ad text and images, landing-domain content, targeting information, and Knowledge Graph facts about the brand, deduplicated into a content profile that fits a long-context prompt.
4. Prompt. Include the policy text with in-scope and out-of-scope examples, and ask the model to summarize the advertiser, describe their products, and then decide. Asking for the summary first forces grounding before the verdict.
5. Prompt tuning. Hand-label a few hundred advertisers split into three sets: two for iterative tuning, where each error is binned by type to find under-constrained parts of the prompt, and one held out for final validation.

Reported results. About 95% accuracy on a small test set with Gemini 1.0 Ultra, and a 65% reduction in incorrectly flagged advertisers at roughly unchanged recall.

When to use which. Use advertiser-level decisions when intent is stable across an account and context removes ambiguity: brand legitimacy, regulated verticals, and scam operations. Keep ad-level decisions for content that violates policy no matter who posts it, such as a single explicit image, and for compromised accounts, where a good advertiser's history would hide a takeover.

Failure modes.
- Account takeover and slow pivots, where the profile is stale, so refresh it on activity changes.
- LLM prior knowledge about well-known brands can be wrong or outdated.
- One wrong advertiser label affects every ad in the account, so advertiser verdicts need higher precision and an audit trail for appeals.

Why this is likely: Google's CIKM 2024 paper "Advertiser Content Understanding via LLMs for Google Ads Safety" covers this design, and the 2025 Ads Safety Report credits intent understanding for an 80% drop in incorrect suspensions.

### Leadership and behavior

#### Q15. Tell me about a time you disagreed with a lead or partner team about a model quality threshold or launch decision. What did you do, and what happened?

This tests whether you can hold a data-backed position on quality while still moving a cross-functional launch forward, which matters on a team that automates decisions with direct revenue and advertiser impact.

Structure the story in four parts.
1. Stakes in numbers. Example: a proposed threshold would raise recall by 12% but cut precision from 96% to 89% on one locale, which meant thousands of wrongful disapprovals a week for legitimate advertisers.
2. How you found the disagreement. Name the evidence, such as a slice analysis, an audit sample, or appeal overturn rates. Interviewers want to see that you checked your own position before pushing it.
3. How you resolved it. Good patterns include proposing a per-slice threshold instead of a global one, running a shadow launch to replace argument with data, and agreeing on a decision rule up front, such as "launch if the lower confidence bound on precision stays above 95% on every slice." Explain how you kept the relationship working and how you escalated if you did.
4. Outcome and reflection. Give the measured result, what you got wrong, and the process change it led to, such as adding the slice gate to the automated pipeline so the pipeline settled the same question automatically from then on.

Interviewers watch how you treated the partner team as closely as whether you were right. Pick a story where someone actually pushed back, and own at least one mistake you made.

Have two backup stories ready on the same themes: removing a manual bottleneck from an ML workflow and getting other teams to adopt the fix, and a launch that went wrong where you led the rollback and the postmortem.

Why this is likely: a 2026 candidate report of a Google L5 MLE onsite described a behavioral question about disagreeing with a team lead on model quality thresholds, and Googleyness guides list conflict and feedback prompts as standard.

## Prep checklist

- [ ] Solve 20 Google-tagged problems from 2025 reports across graphs (BFS, Dijkstra, union-find), heaps, intervals, sliding windows, and DP, in a plain doc with no autocomplete, finishing a working baseline within 20 to 25 minutes.
- [ ] For every coding problem, practice the streaming or real-time follow-up and say the complexity of each version out loud.
- [ ] Read "Scaling Up LLM Reviews for Google Ads Content Moderation" and redraw the funnel, dedup, cluster, LLM label, propagate, and feedback loop from memory with a number at each stage.
- [ ] Read the Google Ads RAG moderation paper, the zero-shot co-embedding paper, and the advertiser content understanding paper, and write one tradeoff each paper does not discuss.
- [ ] Derive the distillation loss with temperature, explain the T^2 factor, and explain how distilling step-by-step adds a rationale loss.
- [ ] Compute token budgets for video at 1 fps under Gemini 2.x (258 tokens per frame) and Gemini 3 (70 tokens per frame) defaults, and size a daily teacher labeling job.
- [ ] Work through the LSH S-curve 1 - (1 - p^r)^b for three (r, b) settings and pick one for a target cosine of 0.9.
- [ ] Practice the precision-at-low-prevalence calculation, prior correction after negative downsampling, and Platt versus isotonic versus temperature scaling.
- [ ] Sketch an agentic evaluation and prompt optimization loop with tuning, gating, and canary splits, per-slice gates, and rollback, and explain how OPRO-style search can overfit.
- [ ] Skim the 2025 Ads Safety Report numbers (8.3B+ ads blocked or removed, 24.9M+ accounts suspended, over 99% stopped before serving, 80% fewer incorrect suspensions).
- [ ] Prepare three behavioral stories: a threshold or launch disagreement, removing a manual bottleneck, and a rollback with a postmortem.

## Sources

- https://arxiv.org/html/2402.14590v1
- https://research.google/pubs/scaling-up-llm-reviews-for-google-ads-content-moderation
- https://research.google/pubs/google-ads-content-moderation-with-rag
- https://dl.acm.org/doi/10.1145/3746252.3761435
- https://arxiv.org/html/2412.16215v1
- https://arxiv.org/html/2409.15343v1
- https://research.google/pubs/advertiser-content-understanding-via-llms-for-google-ads-safety
- https://services.google.com/fh/files/blogs/global_2025_adssafetyreport.pdf
- https://blog.google/products/ads-commerce/2025-ads-safety-report
- https://ppc.land/googles-2025-ads-safety-report-gemini-blocked-8-3-billion-bad-ads
- https://support.google.com/adspolicy/answer/7187501?hl=en
- https://research.google/blog/distilling-step-by-step-outperforming-larger-language-models-with-less-training-data-and-smaller-model-sizes
- https://arxiv.org/abs/2309.03409
- https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/capabilities/video-understanding
- https://github.com/delfer/gemini-api-proxy/blob/main/gemini-api-reference/gemini-video-understanding.md
- https://en.wikipedia.org/wiki/Locality-sensitive_hashing
- https://dswok.com/Metrics-and-losses/Calibration
- https://prachub.com/concepts/classification-thresholds-imbalanced-risk-and-fraud
- https://leetcode.com/discuss/post/6973035/google-first-round-swe-iii-ml-bengaluru-d9rdx
- https://interviewexperiences.in/experience/google/google-swe-iii-ml-l4-interview-experience-may-2025
- https://leetcode.com/discuss/post/6185127/2024-google-interview-questions-compilat-mjrf
- https://leetcode.com/discuss/interview-question/6066801/Google-Onsite-Interview-experience(L3)
- https://leetcode.com/discuss/interview-question/613816/google-onsite-meeting-rooms-3
- https://leetcode.com/discuss/interview-experience/6010240/Google-MLEor-L6or-Interview-Experience-and-Resources-for-preparation
- https://www.finalroundai.com/community/t/google-l5-ml-engineer-onsite-2026-realized-too-late-its-basically-a-swe-interview-with-ml-on-top/150
- https://www.interviewquery.com/guides/google-ml-engineer
- https://interviewpilot.dev/interview-experiences/google
- https://igotanoffer.com/blogs/tech/google-machine-learning-engineer-interview
- https://prepfully.com/interview-guides/google-software-engineer
- https://katkat.com/interview-questions/design-large-scale-near-duplicate-video-detection
- https://interviewing.io/guides/hiring-process/google
