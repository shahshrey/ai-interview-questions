# Meta: Software Engineer, Machine Learning

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Meta's official prep guide for this title lists 45-minute coding, ML system design, and behavioral interviews. The details below come from that guide plus candidate reports from 2025 and 2026, and individual loops vary.

1. Pre-screen. This is a written questionnaire on Meta's candidate portal, a recruiter call, or both. Several reports say senior candidates are asked about team size, scope, the split between coding and leadership, and how they use AI tools and judge their output.
2. Technical screen. This is 45 minutes in CoderPad with two data structures and algorithms problems and no code execution. Some senior candidates instead get a 60-minute AI-native screen. Reported problems include merging two sorted interval lists, Nested List Weight Sum, Range Sum of BST, and path simplification.
3. Full loop of 4 to 5 interviews:
   - One standard coding round with two medium problems in about 40 minutes. Meta's guide says it does not ask dynamic programming. The guide scores problem solving, coding, verification, and communication.
   - One AI-enabled coding round of 60 minutes in a multi-file CoderPad project with a built-in assistant. It usually moves from fixing a bug, to implementing the core function, to optimizing against larger tests. Some tracks run an ML coding variant instead, with sparse matrix-vector products and k-nearest-neighbor search.
   - One or two ML system design rounds. Prompts center on ranking and recommendation, ads, and integrity. Reports say E6 and above loops often include two, and that a weak design round is the usual reason for down-leveling.
   - One behavioral round scored on conflict, ambiguity, driving results, communication, and growth.
4. Hiring committee review, then team matching. Candidate reports say a pass in the debrief does not guarantee an offer, and that team matching can take weeks.

For a role at this scope, expect the design rounds to go deep into training efficiency, sharding, calibration, and data integrity, because those are the problems the listing names.

## Questions and answers

### Coding

#### Q1. Minimum remove to make valid parentheses (LeetCode 1249)

Given a string of lowercase letters, `(` and `)`, remove the minimum number of parentheses so the result is valid, and return any valid result. Valid means every `)` closes an earlier unmatched `(`, and no `(` is left open.

Each unmatched `)` must go, and you find those in one left-to-right scan by keeping a stack of indexes of open parentheses. Whatever is left on the stack at the end is an unmatched `(`, and those go too. Removing exactly the unmatched characters is minimal, because each one has to be removed in any valid result.

```python
def min_remove_to_make_valid(s: str) -> str:
    chars = list(s)
    open_idx = []                 # indexes of unmatched "("
    for i, ch in enumerate(chars):
        if ch == "(":
            open_idx.append(i)
        elif ch == ")":
            if open_idx:
                open_idx.pop()
            else:
                chars[i] = ""     # unmatched ")"
    for i in open_idx:
        chars[i] = ""             # unmatched "("
    return "".join(chars)

assert min_remove_to_make_valid("lee(t(c)o)de)") == "lee(t(c)o)de"
assert min_remove_to_make_valid("a)b(c)d") == "ab(c)d"
assert min_remove_to_make_valid("))((") == ""
```

Time is O(n) and space is O(n) for the character list and the stack. The standard screen runs no code, so trace these edge cases out loud: an empty string, letters only, all closing parentheses, and nested input such as `(((`.

Expect these follow-ups:

- Use O(1) extra space beyond the output. Make two passes with a balance counter. Left to right, drop any `)` that would push the balance negative. Then go right to left and drop the last `balance` copies of `(`.
- Handle three bracket types. Push the expected closer onto the stack and compare on each closer. A related reported variant asks you to count the mismatched brackets instead of removing them.
- Return every minimal result (LeetCode 301). This needs a BFS over removals with deduplication, which costs exponential time in the worst case. Say so before you start coding.

Why this is likely: 1249 is the most frequently reported Meta problem in a 1point3acres aggregation covering March 2025 to October 2026, and a bracket-counting variant appears in a candidate-sourced MLE guide.

#### Q2. Binary tree vertical order traversal (LeetCode 314)

Given a binary tree, return its node values column by column from left to right. Inside a column, list nodes from top to bottom. Nodes in the same row and column go left to right.

BFS already visits nodes top to bottom and left to right, so each column comes out in the right order with no sort. Track the minimum and maximum column so that you can output the columns in order without sorting the keys.

```python
from collections import defaultdict, deque

class TreeNode:
    def __init__(self, val, left=None, right=None):
        self.val, self.left, self.right = val, left, right

def vertical_order(root):
    if root is None:
        return []
    cols = defaultdict(list)
    lo = hi = 0
    q = deque([(root, 0)])
    while q:
        node, c = q.popleft()
        cols[c].append(node.val)
        lo, hi = min(lo, c), max(hi, c)
        if node.left:
            q.append((node.left, c - 1))
        if node.right:
            q.append((node.right, c + 1))
    return [cols[c] for c in range(lo, hi + 1)]

t = TreeNode(3, TreeNode(9, TreeNode(4), TreeNode(0)),
                TreeNode(8, TreeNode(1), TreeNode(7)))
assert vertical_order(t) == [[4], [9], [3, 0, 1], [8], [7]]
```

Time and space are both O(n). Columns are contiguous integers, so `range(lo, hi + 1)` never hits a missing key.

Plain DFS breaks the top-to-bottom rule, because it visits a deeper left node before a shallower right node in the same column. If you use DFS, record `(row, order)` and sort each column, which costs O(n log n).

Expect these follow-ups:

- Break ties by value inside the same row and column (LeetCode 987). Collect `(row, value)` per column and sort.
- Return one flattened list instead of a list of lists. This variant appears in a 2025 E5 candidate report.
- Explain the order of growth for a skewed tree. The queue holds O(width) nodes and the column map holds O(n) values.

Why this is likely: 314 is the second most reported Meta problem in the 1point3acres aggregation, and a modified version appears in a 2025 E5 MLE onsite report.

#### Q3. Merge two sorted lists of disjoint intervals into their union

You get two lists of closed intervals. Each list is sorted by start, and the intervals inside each list do not overlap. Return the sorted union as non-overlapping intervals. For example, `[[1,3],[6,8]]` and `[[2,4],[8,10],[12,13]]` give `[[1,4],[6,10],[12,13]]`.

Walk both lists the way the merge step of merge sort does. Always take the interval with the smaller start, then either extend the last output interval or append a new one. This avoids concatenating and sorting, which costs O((m+n) log(m+n)) and throws away the fact that the inputs are already sorted.

```python
def union_intervals(a, b):
    i = j = 0
    out = []
    while i < len(a) or j < len(b):
        if j == len(b) or (i < len(a) and a[i][0] <= b[j][0]):
            cur = a[i]; i += 1
        else:
            cur = b[j]; j += 1
        if out and cur[0] <= out[-1][1]:       # overlaps or touches
            out[-1][1] = max(out[-1][1], cur[1])
        else:
            out.append([cur[0], cur[1]])
    return out

assert union_intervals([[1, 3], [6, 8]], [[2, 4], [8, 10], [12, 13]]) == [[1, 4], [6, 10], [12, 13]]
assert union_intervals([], [[1, 2]]) == [[1, 2]]
assert union_intervals([[1, 5]], [[2, 3]]) == [[1, 5]]
```

Time is O(m + n) and extra space is O(1) beyond the output. Append a copy of `cur`, because extending `out[-1]` would otherwise mutate the caller's input.

Edge cases to raise:

- Whether touching intervals such as `[1,2]` and `[2,3]` merge. They do with closed intervals and do not with half-open ones, so ask.
- Either list empty.
- One interval containing several from the other list.

Expect these follow-ups:

- Return the intersection instead (LeetCode 986). Use the same two pointers and advance whichever interval ends first.
- Merge k lists. Use a heap of list heads, which costs O(N log k).
- Merge inputs that arrive as iterators too large for memory. The algorithm is already streaming, so explain that it holds only the current output interval.

Why this is likely: this exact union variant appears in a January 2025 Meta MLE phone screen report on LeetCode Discuss and in a 2025 E5 onsite report, and Exponent lists it among recent screen questions.

#### Q4. Sparse vector dot product and sparse matrix-vector multiplication

Part one: represent sparse vectors as sorted `(index, value)` pairs and compute their dot product. Part two: given a sparse matrix as COO triples `(row, col, value)` and a dense vector `x`, compute `y = A x` in time proportional to the number of nonzeros.

```python
def sparse_dot(a, b):
    """a, b: lists of (index, value) sorted by index."""
    i = j = 0
    acc = 0.0
    while i < len(a) and j < len(b):
        if a[i][0] == b[j][0]:
            acc += a[i][1] * b[j][1]; i += 1; j += 1
        elif a[i][0] < b[j][0]:
            i += 1
        else:
            j += 1
    return acc

def coo_to_csr(n_rows, entries):
    indptr = [0] * (n_rows + 1)
    for r, _, _ in entries:
        indptr[r + 1] += 1
    for r in range(n_rows):
        indptr[r + 1] += indptr[r]          # prefix sums give row offsets
    cols = [0] * len(entries)
    vals = [0.0] * len(entries)
    nxt = indptr[:-1].copy()
    for r, c, v in entries:                  # counting sort by row
        k = nxt[r]; cols[k] = c; vals[k] = v; nxt[r] += 1
    return indptr, cols, vals

def csr_matvec(indptr, cols, vals, x):
    y = [0.0] * (len(indptr) - 1)
    for r in range(len(y)):
        acc = 0.0
        for k in range(indptr[r], indptr[r + 1]):
            acc += vals[k] * x[cols[k]]
        y[r] = acc
    return y

ip, c, v = coo_to_csr(3, [(2, 0, 1.0), (0, 1, 2.0), (0, 2, 3.0), (2, 2, 4.0)])
assert csr_matvec(ip, c, v, [1, 1, 2]) == [8.0, 0.0, 9.0]
assert sparse_dot([(0, 1.0), (5, 2.0)], [(5, 3.0), (9, 1.0)]) == 6.0
```

The dot product costs O(nnz_a + nnz_b). The COO to CSR conversion is a counting sort costing O(nnz + rows), and the matrix-vector product costs O(nnz + rows). Empty rows fall out naturally as zeros. Touch only the nonzero entries, because that is what the interviewer is checking.

Expect these follow-ups:

- One vector is far denser than the other. Binary search the larger vector for each index of the smaller one, which costs O(a log b), or store the larger one in a hash map.
- Handle duplicate COO coordinates. Sum them during conversion.
- Explain when CSR beats COO. CSR has O(1) row access and better locality for row-parallel work, and it is the natural layout for one GPU warp per row.
- Connect this to recommendation models. A pooled embedding lookup (EmbeddingBag) is a sparse matrix of IDs times a dense embedding table, which is why the same memory-bound access pattern dominates embedding kernels.

Why this is likely: two 2026 Meta candidate reports describe an AI coding round with sparse matrix times vector and k-nearest-neighbor prompts, and Exponent lists sparse matrix-vector multiplication and a sparse vector data structure among reported ML coding questions.

### Ranking and retrieval system design

#### Q5. Design the ranking system for Facebook or Instagram feed

Start with requirements and scale. Assume about 2 billion daily users and about 10 feed loads each, which is roughly 230K requests per second on average and about 500K at peak. Each request returns a few dozen posts chosen from tens of thousands of eligible ones, within a ranking budget of about 100 to 200 ms at p99. The objective is long-term user value, which raw click counts measure poorly.

Use a funnel, as Meta's Instagram posts describe:

1. Sourcing. Combine several candidate sources: connected content, two-tower embedding retrieval with ANN search, recent engagement neighbors, and trending or local pools. Sources can be real-time or precomputed off-peak. Mix them with tunable weights to get a few thousand candidates.
2. Early-stage ranking. Use a cacheable two-tower model trained to predict whether the late-stage model would put an item in its top K. That makes it a distillation of the heavy model, and it cuts the set to about 500.
3. Late-stage ranking. A multi-task DLRM or sequence model predicts P(like), P(comment), P(share), expected watch time, and P(hide). A value model combines them, for example `score = sum(w_i * p_i) - w_hide * p_hide`.
4. Final pass. Apply diversity rules, author and format caps, integrity demotions, and insertion of ads and new-creator content.

For features, use user and item counters with several time windows, pairwise affinity features, and the user's recent event sequence. Meta has moved ads ranking toward event-based sequence features with attention pooling, which costs O(MN) instead of O(N^2). Training uses logged impressions joined with labels inside a fixed window and retrains every few hours. The Explore post describes continual training on new data every hour.

The hard parts:

- Late-stage throughput. 500K requests times 500 items is about 2.5 x 10^8 item scorings per second. Compute the user side once per request and batch the candidates through the model.
- Position and selection bias. Logged data reflects the old ranker. Add position as a training feature and fix it at serving, and keep a small exploration slice.
- Multi-objective weights. They encode product judgment. Tune them with online experiments, and keep integrity and hide-rate guardrails.
- Cold start. New creators and new posts lack counters, so use content embeddings and a boosted exploration quota.

Measure NE per task offline. Online, measure sessions, time spent, and meaningful interactions, with guardrails on hides, reports, and latency.

Why this is likely: Meta's official MLE prep guide names "design a personalized news ranking system", and the LeetCode Discuss thread collecting Meta MLE design questions reports "design a feed like Instagram or Facebook".

#### Q6. Design ads retrieval that selects candidates from hundreds of millions of ads within a few milliseconds

Requirements: for each ad request, apply advertiser targeting, score the eligible ads, and pass a few thousand to the ranking stage. The pool now runs to hundreds of millions of ads, because generative creative tools multiply variants. Retrieval has to stay within a single-digit-millisecond budget and must handle ads being created, edited, and expiring all day.

A classical design starts with index matching on targeting rules. A two-tower model then computes a user embedding online, and dot-product ANN search runs over ad embeddings that are rebuilt every few hours. It is cheap and cacheable, but the model can only see user-ad interactions through a dot product, so retrieval quality trails ranking quality.

Meta's Andromeda posts describe how to close that gap.

- The 2024 version moved retrieval onto NVIDIA Grace Hopper with hierarchical indexing and GPU-native operators. That grew model capacity by four orders of magnitude.
- The 2026 version replaces the single user tower with an ensemble of user expert towers. It encodes ads jointly with serving context and routes each candidate through an MoE attention-based gating network to the top-K relevant experts. Meta reports near ranking-level quality with about 300x fewer inference FLOPs than the 2024 model.
- At billions of ads per second, the bottleneck becomes memory traffic for intermediate tensors and network traffic between CPU hosts and accelerators. Meta fuses embedding gather, interaction, and scoring into one kernel that keeps intermediates in registers and shared memory, and runs at over 10 billion ads per second on one GH200.
- Index matching, scoring, and top-K selection share one GPU server, so a request carries only user features and a targeting query. Meta reports that this cuts network traffic about 100x and reaches over 1 billion ads per second per server. INT8 adds up to 2x over FP16 with neutral accuracy.

The hard parts to discuss:

- Moving targeting logic onto the GPU while supporting real-time index updates.
- Keeping the retrieval objective aligned with ranking, for example by distilling from ranking scores so that retrieval keeps the ads the ranker would value.
- Fairness to new ads that have no engagement history.
- Measuring retrieval itself. Use recall of the ranking stage's eventual top ads, and run online tests on conversions.

The tradeoff is cacheable dot products, which are cheap but less expressive, against brute-force scoring of every eligible ad with a richer model, which only pays off after kernel fusion and co-location. Meta attributes a 2.4% conversion lift on Facebook to Andromeda since 2025.

Why this is likely: Meta published the next-generation Andromeda retrieval post in October 2026, and candidate guides list ads ranking and click prediction among the most common Meta ML design prompts.

#### Q7. Design an evaluation framework for ads ranking models

Requirements: many teams ship changes to CTR, CVR, and quality models every week. The framework has to catch regressions before launch and predict online advertiser value. It also has to respect the auction, where prediction errors change prices as well as rankings.

Offline evaluation:

- Train on data up to day T and evaluate on day T+1. Random splits leak future information.
- Use normalized entropy as the primary metric. NE is log loss divided by the entropy of the background rate. Report it per task and per major slice, such as surface, country, ad objective, and new or established advertiser.
- Report calibration, the sum of predictions over the sum of labels, overall and by prediction bucket. AUC can appear as a secondary number. It ignores scale, so launch decisions rest on NE and calibration. The 2014 Facebook ads paper chose NE and calibration over AUC for exactly this reason.
- Check freshness by measuring how NE decays when the model is evaluated one, two, and three days after its training cutoff. That decay tells you the retraining cadence.

Auction-aware checks:

- Expected value is roughly bid x pCTR x pCVR plus quality terms. Replay logged auctions with the new predictions to estimate changes in winners, prices, and spend.
- A calibration shift of 5% can move spend more than a small NE gain, so require both to pass before launch.

Online evaluation:

- Advertiser budgets couple the treatment and control groups, because a better model in one arm can drain budgets the other arm would have spent. Use budget-split experiments or advertiser-side randomization for large changes, and user-side splits for small ones.
- Primary metrics are advertiser value (conversions, cost per result, ROAS) and revenue. Guardrails are ad hides and reports, organic engagement, and latency.
- Long-running holdouts capture effects that two-week tests miss.

Platform pieces:

- Frozen benchmark datasets with versions.
- Shared metric definitions.
- Automatic slice reports with confidence intervals.
- A ledger that links each offline result to its later online result, so you can measure how well offline NE predicts online gains and retire gates that do not.
- A model registry with prediction-stability SLOs, as Instagram describes in its post about scaling to 1000 models.

The tradeoff is launch speed against rigor. Tier the gates by risk, so that weight tweaks run fast and new architectures face the full suite.

Why this is likely: Meta's official MLE prep guide lists "design an evaluation framework for ads ranking", and NE and calibration come from Meta's own ads paper.

### ML platform design

#### Q8. Design a feature platform for thousands of ranking models, including user event sequences

Requirements: about ten thousand features and thousands of models. Online reads need a p99 under about 10 ms per request across hundreds of features. Training data must be correct as of each impression's timestamp, and a new feature should go from idea to experiment in days.

Components:

1. A definition registry. Each feature has one owner and one definition written in a DSL or a restricted Python subset. The definition compiles to both a streaming job and a batch backfill job. It also records the default value, its version, and its privacy purpose.
2. Streaming aggregation. Counters over 1 hour, 1 day, 7 days, and 28 days, plus last-N event lists, are computed from event logs with event-time windows and watermarks. Results land in an online key-value store sharded by entity ID.
3. A sequence store. Following Meta's event-based feature design, store per-user event streams with an event type, entity IDs, attributes, and a timestamp. Truncate each stream to a length set by its importance. A user with 2,000 events at about 64 bytes each is about 128 KB, so cap the length and push attribute embeddings into the model rather than the store.
4. Serving-time logging. The ranker logs the exact feature values it used. An online joiner attaches labels later. Training tables store features as maps so that features can be added without schema changes, which matches Meta's data ingestion post.
5. Point-in-time backfill for new features. Join the event log as of each impression timestamp, using the value that existed at that moment instead of the latest one.

The hard parts:

- Skew between the streaming and batch paths. Generating both from one definition removes most of it. Logging at serving time covers the rest for features that already exist.
- Cost. Logging thousands of features for every impression is expensive. Sample negatives, and drop features with low importance scores on a schedule. Meta's sequence learning post makes the same argument about redundant aggregations.
- Freshness against cost. Assign freshness tiers per feature, so that only the counters that need second-level updates pay for them.
- Deletion. Expire user data on deletion requests in both the store and the training tables, and track lineage so that you know which models consumed it.

Tradeoffs: logging at serving time gives exact parity but slows feature iteration, because a new feature has no history until you backfill it. Backfill gives history but risks leakage. Most mature platforms use both, and add a parity check that compares backfilled values with logged values on overlapping days.

Why this is likely: the listing names feature engineering platforms and feature consistency, and Meta has published its event-based feature and data ingestion designs.

#### Q9. Design the training data and training pipeline for a recommendation model that retrains every few hours

Requirements: train on petabytes of logged impressions. Warm-start from the previous checkpoint every few hours, validate automatically, and publish to serving. The pipeline has to stop failures in data, hardware, or code before they reach production.

Data path:

1. An online joiner matches impressions with clicks and conversions inside a label window. The 2014 Facebook ads paper describes this tradeoff. A short window mislabels late clicks as negatives, and a long window delays training and uses more memory. Delayed conversions need either a longer window or a model of the delayed feedback.
2. Negative downsampling at a rate w cuts volume. Record w so that predictions can be recalibrated later.
3. Store the data in partitioned warehouse tables. Meta uses Hive tables in a DWRF columnar format on the Tectonic file system.
4. A disaggregated preprocessing tier reads, decodes, and transforms data, including normalization, bucketization, truncation, and feature crosses. It scales independently of the GPUs. Meta's DPP checkpoints data cursors and replaces failed reader nodes without stopping the job. Size the reader tier so the GPUs always have input ready. Meta reports recommendation jobs using tens to hundreds of reader nodes.

Training path:

1. Each job warm-starts from the last good checkpoint and trains on the new partitions. Sparse and dense state are checkpointed separately, because the embedding state is much larger.
2. Before publishing, compare the candidate with the current model on the same holdout, using NE, calibration, and per-slice deltas. Also check prediction distribution drift on a fixed probe set, and check weight and gradient norms.
3. Publish through a model registry that records data range, code version, and parent checkpoint, so that every model can be reproduced and rolled back.

Reliability:

- Meta reports that over 66% of training interruptions on Llama 3 came from hardware faults.
- Silent data corruption is the subtle case. It produces wrong gradients that stay inside numeric bounds, so training looks healthy while it diverges. Meta estimates it at about one fault per thousand devices.
- Defenses include periodic fleet tests, comparing replicas in data-parallel training, and reductive triage, which bisects on smaller clusters to isolate a NaN source.

The hard parts:

- Warm-starting can lock in a bad update, so keep several older checkpoints and run a slow-path full retrain now and then.
- Schema changes upstream can silently zero a feature. Alert on feature coverage as well as job success.

Why this is likely: the listing asks for training pipelines, training stability, and data integrity, and Meta has published its data ingestion pipeline and its hardware reliability practices.

### Distributed training depth

#### Q10. How do you shard trillion-parameter embedding tables, and what limits scaling to thousands of GPUs?

The basic scheme is DLRM's hybrid parallelism. Embedding tables are model parallel and the dense MLPs are data parallel. In the forward pass, an all-to-all sends each sample's IDs to the GPUs that own the rows and returns pooled embeddings. The backward pass reverses that exchange for gradients.

A table with 1 billion rows at dimension 256 in FP16 is 512 GB before optimizer state, more than any single device can hold.

TorchRec's sharding options:

- Table-wise puts a whole table on one GPU. It is simple, but large tables cause imbalance.
- Row-wise splits rows across GPUs. It balances memory, but pooled partial sums need a reduce-scatter.
- Column-wise splits the embedding dimension. The results are concatenated, and it balances compute for wide tables.
- Table-row-wise keeps the row split inside a node to use NVLink.
- Grid sharding splits both rows and columns, for the largest tables.
- Data parallel replicates small tables, where an all-reduce is cheaper than lookups over the network.

The planner enumerates options per table. It estimates storage and step time from pooling factors, embedding dimension, ID distribution, and topology, then partitions to minimize the slowest rank. Hot IDs matter too. A power-law ID distribution makes some row shards much busier, so the planner needs access statistics as well as sizes. Caching hot rows in HBM, with the rest in host memory through UVM, extends capacity when tables outgrow HBM.

Meta's PyTorch post on 2D sparse parallelism names three problems at thousands of GPUs:

1. Imbalance and stragglers.
2. All-to-all bandwidth that drops across nodes.
3. Input feature memory that grows with GPU count.

The fix is to divide ranks into sharding groups. Inside each group, tables are model parallel. Across groups, the same shards are replicated and synchronized with an all-reduce. Smaller groups mean cheaper all-to-all, and the replicas sit inside one host to use intra-node bandwidth. In one production example, Meta ran 1024 GPUs with sharding groups of 256.

Optimizer state needs separate handling. FBGEMM fuses the optimizer into the embedding backward pass, so embedding gradients are never materialized. With no gradients to all-reduce, 2D parallelism synchronizes the weights. Meta synchronizes optimizer state on a delayed schedule, and syncing every N steps behaves like local SGD.

The failure modes to name:

- One hot table pinned to one rank, which sets the step time.
- Row-wise sharding with poor ID hashing.
- Treating the delayed sync as free, when it can shift convergence and needs a quality check.

Why this is likely: the listing names training efficiency and data loading, and Meta's TorchRec and 2D sparse parallelism work is the public description of how its recommendation training scales.

#### Q11. Compare FSDP, tensor parallelism, and pipeline parallelism. How would you combine them for a large model?

Start with memory. Mixed-precision Adam keeps about 16 bytes per parameter: FP16 or BF16 weights and gradients, plus FP32 master weights and two moment buffers. A 405B model therefore needs about 6.5 TB before activations, so plain data parallel replication is impossible.

FSDP (ZeRO stage 3) shards parameters, gradients, and optimizer state across the data-parallel group. That divides those 16 bytes per parameter by N. Each layer is all-gathered just in time for the forward and the backward pass, and gradients are reduce-scattered. Per step that moves about 1.5 times the traffic of DDP, but it overlaps with compute when prefetch is set up well. FSDP stops scaling when the per-GPU batch gets small, because communication then outweighs compute. Hybrid sharding (HSDP) shards inside a node and replicates across nodes to keep the large all-gathers on NVLink.

Tensor parallelism splits weight matrices. In a transformer MLP, the first matmul is split by columns and the second by rows, so each block needs one all-reduce in the forward pass and one in the backward pass. That traffic is on the critical path, so TP stays inside a node.

Pipeline parallelism splits layers into stages and feeds microbatches through them. With p stages and m microbatches, the idle bubble fraction for GPipe or 1F1B is (p - 1) / (m + p - 1). Interleaved schedules with v virtual stages per rank shrink it by about a factor of v, and zero-bubble schedules split weight-gradient work to fill the gaps. 1F1B also caps in-flight activations at about p microbatches on the first stage, so the first stage has the highest memory use.

Llama 3 405B combined all of them as 4D parallelism, ordered [TP, CP, PP, DP] from the highest to the lowest bandwidth need. TP ran on NVLink inside the server, context parallelism split long sequences, PP spanned nodes, and FSDP was the outermost dimension. Meta removed one layer each from the first and last pipeline stages, giving 126 layers instead of 128, to balance memory and compute.

Recommendation models use a different mix. Meta's GEM training uses 2D FSDP plus expert parallelism for the dense side and fully sharded 2D model parallelism for the sparse side, mapped onto the network hierarchy. It also uses SM-free collectives, so communication leaves the SMs free for compute.

Pick the layout from three things: the memory each GPU must hold, which collectives sit on the critical path, and which link each collective uses. Then confirm the choice with a trace.

Why this is likely: the listing asks for distributed training and foundation model training across clusters, and Meta has published the Llama 3 parallelism design and the GEM training recipe.

#### Q12. Define MFU. Why might a large recommendation model sit near 20% MFU, and how do you find what to fix?

Model FLOPs utilization is the useful model FLOPs completed per second, divided by the hardware's peak FLOPs per second at the precision you use. Recomputation does not count. Counting it gives hardware FLOPs utilization, which flatters a run that recomputes activations. For dense transformers, training costs about 6 FLOPs per parameter per token: 2 in the forward pass and 4 in the backward pass, plus the attention terms. So MFU is about 6 x N x tokens per second, divided by peak.

For recommendation models, count only the dense compute. Embedding lookups are memory-bandwidth bound and contribute almost no FLOPs, so MFU understates how hard the hardware is working, and you should report bandwidth use for the embedding kernels alongside it.

Meta's GEM post gives the framework: end-to-end MFU = local MFU x scaling ratio. Local MFU comes from running each layer alone on one GPU, with no recomputation and no exposed communication. The scaling ratio is end-to-end MFU divided by local MFU. Step latency is the maximum over ranks of max(compute time, communication time), so the slowest rank sets the pace. Near-linear scaling needs four things: compute much larger than communication, communication hidden under compute, little recomputation, and balanced ranks.

Why recommendation models score low:

- Jagged inputs. User histories vary from hundreds to tens of thousands of events, and padding to the maximum wastes up to half the compute. Meta built jagged flash attention to work on unpadded tensors.
- Asymmetric attention shapes. Long queries with short keys, and the reverse, break the tiling assumptions behind FlashAttention. Meta saw a 2.6x gap between production and synthetic benchmarks.
- Memory-bound pieces. Small embedding dimensions and frequent normalizations keep tensor cores idle.
- Precision sensitivity. CTR and CVR objectives regress under naive FP8, so low precision needs block scaling and validation against NE.

Meta reports doubling GEM to 20 to 25% MFU while growing training FLOPs 4x in a year. The levers were custom kernels, MXFP8 attention and MLP (over 1.3x faster forward and 1.5x faster backward), and topology-aware parallelism.

How to find the fix:

1. Profile one step and split the time into compute, exposed communication, data loading stalls, and idle time.
2. If local MFU is low, work on kernels, fusion, padding, and precision.
3. If the scaling ratio is low, work on overlap, load balance across embedding shards, collective placement, and input prefetch.
4. Rerun quality checks after every change, because a speedup only counts if NE holds.

Why this is likely: the listing asks for performance work across training, data loading, and hardware utilization, and Meta's GEM training post defines this exact decomposition.

### Model quality and data integrity

#### Q13. Explain normalized entropy and calibration for click prediction. How do negative downsampling and delayed labels break calibration, and how do you fix it?

Normalized entropy is the average log loss divided by the entropy of a constant predictor that always outputs the background rate p̄:

NE = [ -(1/N) Σ ( y log p + (1 - y) log(1 - p) ) ] / [ -( p̄ log p̄ + (1 - p̄) log(1 - p̄) ) ]

Lower is better, and 1.0 means no better than the base rate. The normalization matters because raw log loss falls as the base rate approaches 0, so without it a seasonal drop in CTR would look like a model gain. For example, at a 2% base rate the constant predictor's loss is about 0.098 nats, so a model at 0.090 has NE of about 0.92.

Calibration is the sum of predicted probabilities divided by the number of observed positives, and the target is 1.0. Check it overall, per prediction bucket with a reliability curve, and per slice. AUC cannot see calibration, because any monotone rescaling of the scores leaves AUC unchanged. In an auction, expected value is about bid x pCTR x pCVR, so a 10% overprediction for one advertiser segment wins impressions it would otherwise lose and moves prices. Meta's ads work reports NE and calibration for this reason.

Negative downsampling keeps a fraction w of the negatives. The model then predicts the odds inflated by a factor of 1/w, and the exact correction is:

q = p / (p + (1 - p) / w)

In logit space, that is the raw logit plus ln(w). Apply the correction before the auction, and record w with the model, because a wrong w produces a systematic bias that looks like drift.

Other ways calibration breaks:

- Delayed conversions. Recent impressions look negative because their conversions have not arrived, so a model trained on fresh data underpredicts CVR. Fix it with a longer label window, an explicit delayed-feedback model, or importance weights for label maturity.
- Distribution shift. New surfaces, new advertisers, and holiday traffic all move the base rate.
- Selection bias. The model trains only on impressions the previous model won.
- Model changes. Ensembling, distillation, and multi-task heads can each shift calibration.

Fixes:

- A post-hoc calibration layer, such as isotonic regression or Platt scaling per segment, refit on recent data.
- A calibration term in the loss.
- Monitoring calibration per segment on fresh traffic, with alerts on a band such as 0.95 to 1.05.

Isotonic regression can overfit small slices, so pool sparse segments.

Why this is likely: NE and calibration come from Meta's 2014 ads click prediction paper and still appear in Meta's engineering posts, and prep sites list "design an ads ranking system with calibration" among Meta questions.

#### Q14. What causes train-serve skew in ranking systems, and how do you detect and prevent it at platform scale?

Train-serve skew is any difference between the inputs or computation a model saw in training and what it sees at serving. It raises no errors, and the only symptom is worse predictions. Name the specific mechanisms:

1. Two code paths. Batch SQL and online C++ compute the "same" feature differently, for example with different bucket boundaries, hash seeds, text normalization, or time zones.
2. Time leakage. A backfilled feature uses data from after the impression, such as a counter that includes the click being predicted. Offline NE improves and online results do not.
3. Freshness gaps. Training sees counters updated each second, while serving reads a cache refreshed hourly, or the reverse.
4. Defaults and failures. At serving, a feature lookup times out and returns 0. The model learned that 0 means "no engagement", so it demotes healthy items without any error, and this gets worse under load.
5. Version mismatch. A model trained on version 3 of a feature reads version 4 after a platform migration.
6. Population differences. Training data is downsampled or filtered differently from serving traffic, or it excludes surfaces the model now serves.

Detection:

- Log the features used at serving for a sample of requests. Every day, recompute them through the offline path and report a parity rate per feature, the fraction within tolerance.
- Replay logged features through the offline model and compare scores with the logged predictions. Any gap means the model or its preprocessing differs.
- Monitor per-feature coverage and default rates online. A jump in defaults is often the first sign of a broken upstream source.
- Track a prediction stability metric per model, as Instagram did in its post about scaling to 1000 models, and alert on calibration drift by slice.

Prevention:

- Compile one feature definition into both paths.
- Train on logged features when you can.
- Use point-in-time joins for backfills.
- Pin feature versions in the model artifact, and refuse to serve if they differ.
- Make "missing" an explicit value distinct from zero.
- Shadow every new model on live traffic before canary, and compare its live predictions with offline predictions for the same requests.

Platform owners also have to drive adoption. Block launches whose features fail parity, and publish the parity dashboard so that teams see their own numbers.

Why this is likely: the listing names feature consistency and serving correctness among the cross-system failures this role resolves, and candidate guides list offline-online gaps as a standard Meta ML follow-up.

### Behavioral

#### Q15. Tell me about a time you set technical direction across several teams that disagreed. How did you decide, and what happened?

At this level the interviewer is checking for scope, judgment, and influence without authority. Meta's guide scores five signals: resolving conflict, embracing ambiguity, driving results, communicating effectively, and growing continuously. Pick one story that touches most of them. Good candidates are a platform migration, a shared modeling standard, or a training infrastructure bet that affected several organizations.

Structure:

1. Context in two sentences. Cover the systems involved, the number of teams, and why it mattered in numbers, such as training cost, launch velocity, or an incident rate.
2. The disagreement. State each side's position fairly, including the reasonable case against yours. Interviewers notice how you describe people who disagreed with you.
3. How you decided. Describe the evidence you gathered, such as a benchmark, a prototype, or a cost model. Then describe the written proposal with alternatives and risks, and who reviewed it. Name one point where the critics were right and you changed the plan.
4. How you drove it. Cover milestones, migration support for the teams that bore the cost, an intermediate win that kept the effort funded, and how you handled a team that resisted the change.
5. Results with numbers. For example: step time down 30%, launch time from weeks to days, or a class of incidents removed. Also say plainly which parts missed.
6. What you would do differently.

Several 2026 reports say Meta also asks how you use AI tools to drive impact and how you check their output. If your story includes using LLM tooling, such as automated triage of failed jobs or generated experiment reports, describe how you measured its error rate and where a person still made the call.

Expect follow-ups on the hardest conversation, on a decision you reversed, and on how you knew the direction was right beyond the fact that people accepted it. Keep the story to about four minutes so there is room for probing.

Why this is likely: Meta's official prep guide defines the behavioral signals, and candidate reports from 2025 and 2026 describe senior behavioral rounds that probe cross-team conflict and AI use.

## Prep checklist

- [ ] Solve the 20 most frequent problems on the 1point3acres-derived Meta list (1249, 314, 680, 227, 199, 560, 543, 56, 408, 215 and the rest), two per 40 minutes, without running code, and trace each by hand.
- [ ] Practice interval, tree, and string-parsing variants with a twist: union of interval lists, vertical order flattened into one list, and cd with `..` and `~`.
- [ ] Do two timed sessions in Meta's CoderPad practice environment with the AI assistant: fix a bug, implement a core function in a multi-file project, then optimize it. Practice rejecting a hallucinated function.
- [ ] Implement from scratch: sparse dot product, CSR matrix-vector multiply, k-nearest-neighbor search with a heap, and a softmax attention forward pass.
- [ ] Rehearse feed ranking and ads click prediction designs end to end in 35 minutes, with QPS, candidate counts, and latency budgets written down.
- [ ] Read Meta's Andromeda 2024 and 2026 posts, the GEM training post, the sequence learning post, and the Instagram Explore post. Write one paragraph per post on the bottleneck it solved.
- [ ] Derive by hand: NE for a 2% base rate, the downsampling correction q = p / (p + (1 - p) / w), the pipeline bubble (p - 1) / (m + p - 1), and 16 bytes per parameter for Adam.
- [ ] Explain TorchRec's sharding types and 2D sparse parallelism, and give one example where table-wise sharding creates a straggler.
- [ ] Prepare a train-serve skew incident and a silent training divergence incident, each with detection time, root cause, and the systemic fix.
- [ ] Prepare four behavioral stories, two of which involve AI tools with a measured result, and practice each in under four minutes.

## Sources

- https://d3no4ktch0fdq4.cloudfront.net/public/course/files/Meta_ML_onsite_interview_prep.pdf
- https://tryexponent.com/guides/meta-machine-learning-engineer-interview
- https://www.interviewquery.com/guides/meta-ml-engineer
- https://prepfully.com/interview-guides/meta-machine-learning-engineer
- https://www.aced.io/experiences/meta-facebook-machine-learning-engineer-interview-32229f
- https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7
- https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-9efbbd
- https://leetcode.com/discuss/interview-question/6344393/Meta-or-E4-or-screen-or-MLE-or-US
- https://leetcode.com/discuss/interview-question/6295593/Meta-MLE-E4E5-Phone-screening-or-Passed-to-Full-loop
- https://leetcode.com/discuss/interview-experience/6300660/Meta-MLE-system-design-questions
- https://leetcode.com/discuss/interview-question/5236394/Meta-E6-or-Offer
- https://roundz.substack.com/p/interview-experience-183-meta-machine-learning-engineer
- https://www.teamblind.com/post/meta-e6-hc-for-ml-swe-jwwm3dr2
- https://www.bigocodes.com/meta
- https://www.fastprep.io/problems/meta-coo-sparse-matrix-multiplication
- https://www.hellointerview.com/guides/meta/e6
- https://prachub.com/companies/meta/positions/machine-learning-engineer/categories/ml-system-design
- https://engineering.fb.com/2026/10/28/ml-applications/meta-andromeda-next-gen-ads-retrieval-gpu-co-design
- https://engineering.fb.com/2024/12/02/production-engineering/meta-andromeda-advantage-automation-next-gen-personalized-ads-retrieval-engine
- https://engineering.fb.com/2026/08/03/ml-applications/training-gem-at-llm-scale-meta-ads-recommendation-foundation-model
- https://engineering.fb.com/2024/11/19/data-infrastructure/sequence-learning-personalized-ads-recommendations
- https://engineering.fb.com/2023/08/09/ml-applications/scaling-instagram-explore-recommendations-system
- https://engineering.fb.com/2025/05/21/production-engineering/journey-to-1000-models-scaling-instagrams-recommendation-system
- https://engineering.fb.com/2022/09/19/ml-applications/data-ingestion-machine-learning-training-meta
- https://engineering.fb.com/2025/07/22/data-infrastructure/how-meta-keeps-its-ai-hardware-reliable
- https://pytorch.org/blog/scaling-recommendation-2d-sparse-parallelism
- https://meta-pytorch.org/torchrec/high-level-arch.html
- https://arxiv.org/pdf/1906.00091
- https://quinonero.net/Publications/predicting-clicks-facebook.pdf
- https://johaupt.github.io/blog/downsampling_recalibration.html
- https://aisystemcodesign.github.io/papers/Llama3-ISCA25.pdf
- https://www.intoai.pub/p/distributed-training-of-llama-explained
