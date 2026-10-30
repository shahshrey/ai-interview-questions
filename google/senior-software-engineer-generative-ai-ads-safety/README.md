# Google: Senior Software Engineer, Generative AI, Ads Safety

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This is an estimate based on public guides and anecdotal candidate reports. The exact loop for this team may differ.

1. Recruiter screen: background, level fit, and the ML track.
2. Technical phone screen: one algorithms problem in a shared editor, sometimes with an ML follow-up.
3. Onsite of 4 to 5 rounds, about 45 minutes each, most likely in person:
   - 1 to 2 coding rounds on data structures and algorithms.
   - 1 ML system design round, likely framed around moderation or model refresh at scale.
   - 1 ML domain round on multimodal models, distillation, evaluation, and prompting.
   - 1 Googleyness and Leadership round.
4. Hiring committee review.
5. Team matching with hiring managers.
6. Offer.

## Questions and answers

### LLM moderation at scale

#### Q1. You cannot afford to run an LLM on every ad. How do you still use one to catch policy violations at scale?

Begin with the cost. A strong multimodal model can't score billions of creatives, so the job is to make each LLM call cover thousands of ads. Google's own published Ads Safety work follows this pattern. First comes a funnel: cheap classifiers, rules, and actor signals pick likely candidates, and exact or perceptual hashing removes duplicates. Then cluster the candidates with embeddings and choose one representative per cluster, plus a few diverse samples from large clusters. The LLM reviews only the representatives. Its decisions spread to cluster members through similarity, using a tighter threshold for propagation than for clustering, because every wrongly propagated label hits a real advertiser. Finally, feed the confirmed labels back into the funnel so the next round finds lookalikes. The tradeoffs to discuss are propagation precision versus coverage, cross-modal versus single-modality embeddings (cross-modal tends to work better for image ads), and freshness versus compute. A strong answer gives metrics for each stage (funnel recall, LLM precision, propagation precision, cost per enforced ad) and a rule for when a propagated label needs a human audit sample.

#### Q2. A new ad policy is announced today. How do you get enforcement live in minutes instead of days?

The slow path is collecting labels, training a classifier, and running a launch review, which takes days. The fast path rests on in-context learning. Write the policy text and a dozen in-scope and out-of-scope examples into a prompt for a strong multimodal model, or retrieve policy passages and earlier decisions through RAG when the policy is long or changes often. Run it against a small expert-labeled golden set within the hour. Launch in shadow mode first and compare against human decisions, then turn on enforcement for high-confidence decisions only, sending the uncertain middle band to reviewers. Meanwhile, the LLM's labels and rationales become training data for a cheap student model that later takes over the high-volume traffic. The tradeoffs are consistency (prompts can be brittle across locales and formats), cost per call during the prompt phase, and the risk of over-enforcing on legitimate advertisers. A strong answer names the gates between shadow, partial, and full enforcement, how rollback works, and how the prompt path is retired once the distilled model meets the bar.

#### Q3. Bad advertisers submit many near-identical versions of a rejected ad. How do you catch the variants?

This is an adversarial near-duplicate problem. Layer three things. First, content similarity: perceptual hashes for exact and light edits, and multimodal embeddings for paraphrased text, cropped images, or recut video, indexed in an approximate nearest neighbor store. Second, actor signals: shared accounts, payment instruments, domains, landing pages, and creation timing link variants that look different on the surface. Third, entity-level reasoning: judge the advertiser and the destination rather than each creative alone, which public Google work on advertiser content profiles supports. When a new ad lands close to a known violator, raise its review priority or apply the existing decision if similarity and actor overlap are both high. The main tradeoff is collateral damage. Legitimate businesses in the same vertical produce similar-looking ads, so propagation needs a high bar and an audit sample. A strong answer covers evasion moves (small pixel noise, text in images, cloaked landing pages), how the index updates in near real time, and how false positives are measured through appeal outcomes.

### Multimodal and video understanding

#### Q4. Design a multimodal teacher model for ad video understanding and explain how it feeds production classifiers.

The teacher exists to produce high-quality labels and rationales, not to serve every request. The inputs are sampled frames (scene-change based, not uniform, so short flashes are not missed), ASR transcript, on-screen text through OCR, audio events, and landing page context. A Gemini-class model, prompted or lightly tuned per policy, outputs a label, a confidence, timestamps of the evidence, and a short rationale. Those outputs train compact student models that score all video traffic. Quality controls include agreement with expert reviewers by policy and locale, a check that the cited timestamps actually show the problem, and tracking the gap between teacher and student. Tradeoffs: more frames and longer context raise recall on brief violations but increase cost; one teacher per policy is more accurate while a single shared teacher is easier to maintain. A strong answer explains the refresh loop. When the teacher is updated or a policy changes, regenerate labels for a targeted slice, retrain the students, and gate the launch on slice-level regressions.

#### Q5. How do you distill a large multimodal model into one cheap enough to score every ad?

Begin with the serving budget, latency and cost per item, because that fixes the student's size and architecture. Build training data from teacher outputs on a stratified sample that oversamples rare and hard policy cases, since safety data is very imbalanced. Use soft labels rather than just hard ones so the student learns the teacher's uncertainty. Google's distilling step-by-step work suggests training the student to also predict the teacher's rationale as an auxiliary task, which improves data efficiency. Mix in human-labeled examples where the teacher is known to be weak. Evaluate the student against human labels, not only against the teacher, because copying teacher errors faithfully is not success. Tradeoffs: one multi-policy student is cheaper to run but policies interfere with each other; per-policy heads are a common middle ground. Pick calibration and thresholds per policy. A strong answer also covers drift. Students go stale as adversaries adapt, so schedule refreshes from new teacher labels and watch live score distributions for shifts.

### Agents for the ML lifecycle

#### Q6. You build an agent that tunes prompts and runs evals with no human in the loop. What stops it from shipping a regression?

The danger is an optimizer that games its own metric. Keep the data used for tuning separate from the data used for gating. The agent searches prompt variants, in the style of OPRO-like optimization, against a tuning set, but promotion depends on a held-out gating set it never sees, refreshed regularly with new adversarial examples. Gates are per slice, not just the overall number: each policy, locale, format, and advertiser segment has a no-regression threshold on precision and recall, plus cost and latency budgets. Limit the agent's search budget and the scope of its changes so a single run cannot rewrite everything. Every candidate gets a canary or shadow phase on live traffic before full rollout, with automatic rollback when online metrics or appeal rates move. Humans stay off the critical path but own the gate definitions and get an audit trail of every change and its evidence. A strong answer explains why gate changes need human sign-off while ordinary prompt changes do not.

#### Q7. Design an automated model refresh pipeline for safety classifiers.

Triggers come from three sources: schedules, detected drift (shifts in score distributions, rising appeal overturns, new adversarial clusters), and policy changes. The pipeline pulls fresh labels from reviewers and teacher models, validates the data (schema, label balance, leakage against eval sets), trains candidates, and runs the standard evaluation suite. Promotion uses the same per-slice gates as Q6, followed by shadow scoring on live traffic, then a staged rollout with an automatic rollback path. Version everything: data snapshot, code, config, thresholds. Then any decision can be traced to the model that made it, which matters for appeals and audits. Tradeoffs: refreshing often tracks adversaries better but risks instability and more churn for advertisers; refreshing rarely is stable but stale. A practical middle path is frequent threshold recalibration with less frequent full retrains. A strong answer includes observability (dashboards per policy, alerts on precision drops from audit samples), ownership of each failure mode, and a way to freeze refreshes during incidents.

### Evaluation and quality

#### Q8. How do you build and maintain a golden evaluation set for ad policy decisions?

The golden set is the contract every automated change is judged against, so treat it as a product. Have policy experts label it, not general raters, and stratify by policy, locale, ad format, and difficulty, including deliberately borderline and adversarial cases. Use multiple labelers on hard items and record the adjudicated answer plus the reason, which also tells you where the policy itself is unclear. Version the set and keep a changelog, because metric moves caused by changing the set are easy to mistake for model changes. Refresh it on a schedule with recent evasion patterns, and retire stale items carefully so you can still compare against history. Guard against leakage: golden items must never appear in prompts, few-shot examples, or training data, which needs automated hash checks. Tradeoffs: a bigger set gives tighter confidence intervals but costs expert time, so size each slice by the decision it supports. A strong answer mentions reporting confidence intervals and keeping a separate hidden set for gating agents.

#### Q9. When would you use LLM-as-a-judge for safety labels, and how do you calibrate it?

An LLM judge is useful for scale: grading thousands of candidate outputs, writing rationales, and finding likely disagreements. It is not ground truth. Before relying on it, measure agreement with expert labels on each slice, not only overall, because judges are often good on common policies and weak on rare or culture-specific ones. Look for known biases: position and verbosity effects, sensitivity to prompt wording, and over-flagging of unusual but legitimate content. Calibrate by mapping judge scores to observed precision on a labeled sample, then set thresholds per policy. Use the judge mainly for triage and for evaluating candidate models, and send the cases where the judge and the production model disagree to human reviewers, since that is where the most informative labels come from. Re-check agreement whenever the judge model or its prompt changes, because a judge upgrade quietly shifts every metric built on it. A strong answer treats the judge as a measurement instrument with its own error bars, and keeps humans auditing a random sample.

#### Q10. Your classifier's precision dropped right after a model refresh. Walk through how you debug it.

Mitigate first. If the drop is confirmed through audit samples or appeal overturns, roll back or raise thresholds to protect advertisers, then investigate. Debug from the data outward. Check for training-serving skew: did a feature, preprocessing step, or embedding version change in one place but not the other? Check the labels: did a new label source, such as a teacher model update, add noise or change the policy interpretation? Check whether the eval set changed, which can make offline numbers look fine while production differs. Then compare the old and new models slice by slice: policy, locale, format, advertiser segment. Regressions usually concentrate somewhere specific. Look at the score distributions too. A shift with stable ranking quality points to a calibration or threshold issue, not a modeling one. Once you find the root cause, add a gate or test that would have caught it. A strong answer is systematic, separates detection from mitigation from root cause, and ends with a process fix, not just a model fix.

### Coding

#### Q11. Given a stream of ad embeddings, group items whose similarity is above a threshold into clusters.

Clarify first: is the relation transitive (connected components) or do clusters need a maximum diameter? Assume connected components. Comparing all pairs is O(n squared) and fails at scale, so generate candidate pairs with locality-sensitive hashing (random hyperplanes for cosine similarity) or an approximate nearest neighbor index, then confirm each candidate with the exact similarity check. Merge confirmed pairs with union-find using path compression and union by rank, which makes each operation nearly constant time. For streaming, insert each new item into its buckets, compare it only with items in those buckets, and union on matches. Clusters update incrementally without recomputing. Discuss memory: bucket sizes can blow up for popular templates, so cap them or use more hash bits. Also note that transitive merging can chain unrelated items into one giant cluster, so you may need a diameter check or a split pass. A strong answer writes clean union-find code, states the complexity, and explains recall loss from LSH and how to tune the number of hash tables.

#### Q12. Return the k domains with the most policy violations over a sliding time window.

For a single machine with exact counts, keep a deque of (timestamp, domain) events and a hash map of counts. On each new event, append it and increment its count. Before answering a query, pop expired events from the front and decrement their counts, deleting zeros. For the top k, either run a heap selection over the map at query time, O(m log k) for m distinct domains, or maintain bucketed counts for O(1) updates if queries are frequent. Discuss which operation dominates, since that decides the design. At Google scale the stream is distributed and memory is limited, so move to approximate methods: a count-min sketch per time bucket with a small heap of heavy hitters, and buckets that rotate out as the window slides. Sharding by domain makes the merge step simple. Tradeoffs are exactness versus memory and the error bounds of the sketch. A strong answer handles events that arrive out of order, ties, and empty windows, and tests the expiry logic explicitly.

### Design judgment

#### Q13. Compare RAG, in-context learning, and fine-tuning for policy classification. When do you pick each?

They solve different problems along the speed, cost, and consistency axes. In-context learning is the fastest to deploy: put the policy and examples in the prompt, and a new policy can be enforced the same day. It is expensive per call, sensitive to example choice, and can be inconsistent across formats. RAG helps when the policy is long, changes often, or depends on precedent: retrieve the relevant policy sections and similar past decisions per ad, so the prompt stays short and current. It adds retrieval latency and a new failure mode if retrieval misses. Fine-tuning or distillation gives the lowest cost and most consistent behavior at very high volume, but takes data and time, and it locks in the policy interpretation until the next refresh. In practice they form a lifecycle: ICL or RAG to launch quickly, then distill into a tuned model for volume, keeping the prompt path for the hardest or newest cases. A strong answer ties each choice to measurable requirements and explains how to move traffic between them safely.

#### Q14. How would you decide which enforcement decisions run fully automated and which need human review?

Use explicit criteria rather than instinct. Severity: decisions involving harm to users, such as scams or dangerous products, justify more aggressive automation on the blocking side, while account-level suspensions deserve more caution. Reversibility: disapproving a single ad is cheap to undo, suspending an advertiser is not. Confidence and calibration: automate only score ranges where audited precision meets a per-policy target. Advertiser impact: high-spend or long-standing accounts with good history may get a review before drastic actions. Feedback: appeal overturn rates by policy show where automation is wrong. Put this into a routing policy with three bands per policy: auto-enforce, human review, and auto-allow. Audit random samples from every band, including auto-allow, which is where missed violations hide. Revisit the bands when models or policies change. The tradeoff is reviewer capacity versus error cost, and the goal in this role is to shrink the human band without losing precision. A strong answer frames humans as a scarce resource to spend on the most uncertain, highest-stakes cases.

### Leadership and behavior

#### Q15. Tell me about a time you removed a manual bottleneck from an ML workflow, and how you got other teams to adopt it.

This tests the core theme of the role: taking humans off the critical path without losing safety, in an organization with many stakeholders. Pick a real story where a manual step, such as hand-run evals, manual threshold tuning, or ad hoc label requests, slowed every model update. Use a clear structure. Describe the situation and put a number on the cost (days per release, engineer hours, missed incidents). Explain the design you chose and the alternatives you rejected. Describe the safety checks that made automation acceptable to skeptics, such as gates, audit logs, and rollback. Then focus on adoption: who resisted and why, how you used data or a pilot to win them over, and what you changed based on their feedback. Close with measured results and what you would do differently. Interviewers look for ownership, influence without authority, and honest reflection on mistakes. A strong answer avoids vague team credit, shows your specific decisions, and connects the outcome to user or business impact.

## Prep checklist

- [ ] Read "Scaling Up LLM Reviews for Google Ads Content Moderation" and be able to sketch its funnel, cluster, label, and propagate design from memory.
- [ ] Read "Advertiser Content Understanding via LLMs for Google Ads Safety" and think through advertiser-level versus ad-level classification.
- [ ] Skim Google's latest Ads Safety Report for the scale and the problems Gemini is used on.
- [ ] Practice a 45 minute ML system design for new policy enforcement going from days to minutes, including shadow and staged rollout.
- [ ] Review distillation basics: soft labels, rationale supervision, class imbalance, and evaluating students against humans.
- [ ] Prepare a clear view on agent-driven evals and prompt optimization: tuning versus gating sets, slice gates, and rollback.
- [ ] Practice medium and hard algorithm problems on union-find, hashing, heaps, sliding windows, and streaming.
- [ ] Prepare 4 behavioral stories: cross-team disagreement, removing a manual bottleneck, a launch that went wrong, and leading without authority.
- [ ] Plan for an in-person onsite, since the listing says that is required in most instances.

## Sources

- https://arxiv.org/html/2402.14590v1 - Google Ads Safety paper on scaling LLM reviews with funneling, clustering, and label propagation.
- https://arxiv.org/pdf/2409.15343 - Google Ads Safety paper on classifying advertiser intent with LLMs.
- https://blog.google/products/ads-commerce/2025-ads-safety-report - Google's 2025 Ads Safety Report summary on Gemini-powered enforcement.
- https://support.google.com/adspolicy/answer/13584894 - Google help page on how automation and human review combine in ad moderation.
- https://research.google/blog/distilling-step-by-step-outperforming-larger-language-models-with-less-training-data-and-smaller-model-sizes - Google Research on distilling LLM rationales into small models.
- https://arxiv.org/abs/2309.03409 - Google DeepMind OPRO paper on using LLMs to optimize prompts.
- https://igotanoffer.com/blogs/tech/google-machine-learning-engineer-interview - Public guide to Google ML engineer interview rounds.
- https://www.datainterview.com/blog/google-machine-learning-engineer-interview - Public guide on onsite structure and level expectations.
- https://igotanoffer.com/en/advice/google-team-matching - Public guide on Google hiring committee and team matching.
- https://www.reddit.com/r/leetcode/comments/1r4dq0g/senior_sofware_engineer_ml_applied_interview - Anecdotal candidate discussion of the Senior SWE Applied ML loop.
