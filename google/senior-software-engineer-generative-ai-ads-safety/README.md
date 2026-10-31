# Google: Senior Software Engineer, Generative AI, Ads Safety

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop is pieced together from public guides and anecdotal candidate reports, so the exact loop for this team may differ.

1. A recruiter screen covers your background, level fit, and the ML track.
2. A technical phone screen gives you one algorithms problem in a shared editor, sometimes with an ML follow-up.
3. The onsite has 4 to 5 rounds of about 45 minutes each, most likely in person.
   - You get 1 to 2 coding rounds on data structures and algorithms.
   - One ML system design round is likely framed around moderation or model refresh at scale.
   - One ML domain round covers multimodal models, distillation, evaluation, and prompting.
   - One round covers Googleyness and Leadership.
4. A hiring committee reviews the packet.
5. Team matching happens with hiring managers.
6. The offer comes last.

## Questions and answers

### LLM moderation at scale

#### Q1. You can't afford to run an LLM on every ad. How would you still use one to catch policy violations at scale?

Start from the cost. A strong multimodal model can't score billions of creatives, so each LLM call has to cover thousands of ads. Google's published Ads Safety work follows this pattern.

The first stage is a funnel. Cheap classifiers, rules, and actor signals pick likely candidates, and exact or perceptual hashing removes duplicates. Next you cluster the candidates with embeddings and pick one representative per cluster, plus a few diverse samples from the large clusters. The LLM reviews only those representatives.

Its decisions then spread to the other cluster members by similarity. Use a tighter threshold for propagation than for clustering, because every wrongly propagated label lands on a real advertiser. The confirmed labels go back into the funnel so the next round finds lookalikes.

Be ready to talk about where the tradeoffs sit. Propagation precision pulls against coverage, and freshness costs compute. Cross-modal embeddings tend to work better than single-modality ones for image ads.

Give a metric for each stage: funnel recall, LLM precision, propagation precision, and cost per enforced ad. Also give a rule for when a propagated label needs a human audit sample.

#### Q2. A new ad policy is announced today. How do you get enforcement live in minutes rather than days?

Collecting labels, training a classifier, and running a launch review takes days. The faster route uses in-context learning. Put the policy text and a dozen in-scope and out-of-scope examples into a prompt for a strong multimodal model. If the policy is long or changes often, retrieve policy passages and earlier decisions through RAG. Run the prompt against a small expert-labeled golden set within the hour.

Launch in shadow mode and compare its decisions with human ones. Then turn on enforcement for high-confidence decisions only and send the uncertain middle band to reviewers. While that runs, the LLM's labels and rationales become training data for a cheap student model, which later takes over the high-volume traffic.

The prompt phase has real costs. Prompts can be brittle across locales and formats, every call is expensive, and over-enforcement hurts legitimate advertisers.

Name the gates between shadow, partial, and full enforcement, and explain how rollback works. Then describe how you retire the prompt path once the distilled model meets the bar.

#### Q3. Bad advertisers submit many near-identical versions of a rejected ad. How do you catch the variants?

Treat this as an adversarial near-duplicate problem and layer your signals.

Start with content similarity. Perceptual hashes catch exact copies and light edits. Multimodal embeddings catch paraphrased text, cropped images, and recut video, and you index them in an approximate nearest neighbor store. Add actor signals next. Shared accounts, payment instruments, domains, landing pages, and creation timing link variants that look different on the surface. Then reason at the entity level by judging the advertiser and the destination along with each creative. Public Google work on advertiser content profiles supports this.

When a new ad lands close to a known violator, raise its review priority. If similarity and actor overlap are both high, apply the existing decision directly.

The main tradeoff is collateral damage. Legitimate businesses in the same vertical produce similar-looking ads, so propagation needs a high bar and an audit sample.

Cover the evasion moves you expect, such as small pixel noise, text inside images, and cloaked landing pages. Explain how the index updates in near real time, and how you measure false positives through appeal outcomes.

### Multimodal and video understanding

#### Q4. Design a multimodal teacher model for ad video understanding, and explain how it feeds production classifiers.

The teacher's job is to produce high-quality labels and rationales for training. Smaller models handle the request traffic.

Its inputs are sampled frames, the ASR transcript, on-screen text from OCR, audio events, and landing page context. Sample frames at scene changes so short flashes get captured. A Gemini-class model, prompted or lightly tuned per policy, outputs a label, a confidence, timestamps for the evidence, and a short rationale. Those outputs train compact student models that score all video traffic.

For quality, measure agreement with expert reviewers by policy and locale. Check that the cited timestamps actually show the problem, and track the gap between teacher and student.

More frames and longer context raise recall on brief violations but cost more. One teacher per policy is more accurate, while a single shared teacher is easier to maintain.

Spend some time on the refresh loop. When the teacher is updated or a policy changes, regenerate labels for a targeted slice, retrain the students, and hold the launch until every slice passes its regression check.

#### Q5. How do you distill a large multimodal model into one cheap enough to score every ad?

Start from the serving budget, meaning latency and cost per item, because that fixes the student's size and architecture.

Build training data from teacher outputs on a stratified sample. Safety data is very imbalanced, so oversample rare and hard policy cases. Train on soft labels as well as hard ones so the student learns the teacher's uncertainty. Google's distilling step-by-step work suggests also training the student to predict the teacher's rationale as an auxiliary task, which improves data efficiency. Mix in human-labeled examples where the teacher is known to be weak.

Evaluate the student against human labels as well as against the teacher. A student that matches the teacher perfectly has also copied the teacher's errors.

One multi-policy student is cheaper to run, but the policies interfere with each other. Per-policy heads are a common middle ground. Pick calibration and thresholds per policy.

Cover drift as well. Students go stale as adversaries adapt, so schedule refreshes from new teacher labels and watch live score distributions for shifts.

### Agents for the ML lifecycle

#### Q6. You build an agent that tunes prompts and runs evals with no human in the loop. What keeps it from shipping a regression?

The risk is an optimizer that games its own metric, so keep tuning data and gating data separate. The agent searches prompt variants against a tuning set, in the style of OPRO-like optimization. Promotion depends on a held-out gating set the agent never sees, refreshed regularly with new adversarial examples.

Set gates per slice as well as overall. Each policy, locale, format, and advertiser segment gets a no-regression threshold on precision and recall, and every candidate also has cost and latency budgets to meet. Cap the agent's search budget and the scope of its changes so a single run touches only a bounded part of the system.

Every candidate goes through a canary or shadow phase on live traffic before full rollout, with automatic rollback when online metrics or appeal rates move.

Humans stay off the critical path. They own the gate definitions and get an audit trail of every change and its evidence. Be ready to explain why changing a gate needs human sign-off while ordinary prompt changes go through automatically.

#### Q7. Design an automated model refresh pipeline for safety classifiers.

A refresh starts from one of three triggers: a schedule, detected drift, or a policy change. Drift shows up as shifts in score distributions, rising appeal overturns, or new adversarial clusters.

The pipeline pulls fresh labels from reviewers and teacher models, then validates the data for schema, label balance, and leakage against eval sets. It trains candidates and runs the standard evaluation suite. Promotion uses the same per-slice gates as Q6, followed by shadow scoring on live traffic and then a staged rollout with an automatic rollback path.

Version the data snapshot, code, config, and thresholds. That lets you trace any decision to the model that made it, which appeals and audits depend on.

Frequent refreshes track adversaries better but bring instability and more churn for advertisers. Rare refreshes are stable and go stale. A practical middle path is frequent threshold recalibration with less frequent full retrains.

Round out the design with observability, meaning dashboards per policy and alerts on precision drops from audit samples. Give each failure mode an owner, and add a way to freeze refreshes during incidents.

### Evaluation and quality

#### Q8. How do you build and maintain a golden evaluation set for ad policy decisions?

Every automated change is judged against the golden set, so run it like a product. Policy experts should do the labeling. Stratify by policy, locale, ad format, and difficulty, and include deliberately borderline and adversarial cases.

Put multiple labelers on hard items and record the adjudicated answer with its reason. Those disagreements also show you where the policy itself is unclear.

Version the set and keep a changelog. Changing the set moves the metrics, and the changelog lets you separate those moves from model changes. Refresh the set on a schedule with recent evasion patterns, and retire stale items carefully so you can still compare against history.

Keep golden items out of prompts, few-shot examples, and training data, and enforce that with automated hash checks. Any leak makes the set useless as a measure.

A bigger set gives tighter confidence intervals but costs expert time, so size each slice by the decision it supports. Report confidence intervals with your metrics, and keep a separate hidden set for gating agents.

#### Q9. When would you use an LLM as a judge for safety labels, and how do you calibrate it?

An LLM judge helps with scale. It can grade thousands of candidate outputs, write rationales, and find likely disagreements. Its labels carry their own errors, so treat it as a measurement instrument with error bars.

Before relying on it, measure agreement with expert labels on each slice as well as overall. Judges are often good on common policies and weak on rare or culture-specific ones. Look for the known biases, which include position and verbosity effects, sensitivity to prompt wording, and over-flagging of unusual but legitimate content.

To calibrate, map judge scores to observed precision on a labeled sample, then set thresholds per policy. Use the judge mainly for triage and for evaluating candidate models. Send the cases where the judge and the production model disagree to human reviewers, since those produce the most informative labels.

Re-check agreement whenever the judge model or its prompt changes, because a judge upgrade shifts every metric built on it. Keep humans auditing a random sample the whole time.

#### Q10. Your classifier's precision dropped right after a model refresh. How do you debug it?

Mitigate first. If audit samples or appeal overturns confirm the drop, roll back or raise thresholds to protect advertisers, then investigate.

Work from the data outward. Check for training-serving skew, where a feature, preprocessing step, or embedding version changed in one place and stayed the same in the other. Check the labels, since a new label source such as a teacher model update can add noise or change the policy interpretation. Check whether the eval set changed, because that can make offline numbers look fine while production differs.

Then compare the old and new models slice by slice across policy, locale, format, and advertiser segment. Regressions usually concentrate somewhere specific. Look at the score distributions as well. If the distribution shifted while ranking quality held steady, you have a calibration or threshold issue rather than a modeling one.

Once you find the root cause, add a gate or test that would have caught it. Keep detection, mitigation, and root cause as separate steps when you explain it, and finish with a process fix alongside the model fix.

### Coding

#### Q11. Given a stream of ad embeddings, group items whose similarity is above a threshold into clusters.

Ask first whether the relation is transitive, which gives connected components, or whether clusters need a maximum diameter. Assume connected components.

Comparing all pairs is O(n squared), which is too slow at scale. Generate candidate pairs with locality-sensitive hashing, using random hyperplanes for cosine similarity, or with an approximate nearest neighbor index. Confirm each candidate with the exact similarity check. Merge confirmed pairs with union-find using path compression and union by rank, which makes each operation nearly constant time.

For streaming, insert each new item into its buckets, compare it only with items in those buckets, and union on matches. Clusters then update incrementally without a full recompute.

Talk about memory. Buckets for popular templates can grow very large, so cap them or use more hash bits. Transitive merging can also chain unrelated items into one giant cluster, which you can fix with a diameter check or a split pass.

Write clean union-find code and state the complexity. Then explain how LSH loses recall and how the number of hash tables tunes that loss.

#### Q12. Return the k domains with the most policy violations over a sliding time window.

On a single machine with exact counts, keep a deque of (timestamp, domain) events and a hash map of counts. Append each new event and increment its count. Before answering a query, pop expired events from the front, decrement their counts, and delete any count that reaches zero. For the top k, run a heap selection over the map at query time, which costs O(m log k) for m distinct domains. If queries are frequent, maintain bucketed counts for O(1) updates. Which operation dominates decides the design, so ask about it.

At Google scale the stream is distributed and memory is limited, so switch to approximate methods. Keep a count-min sketch per time bucket with a small heap of heavy hitters, and rotate buckets out as the window slides. Sharding by domain keeps the merge step simple. You trade exactness for memory and accept the sketch's error bounds.

Handle events that arrive out of order, ties, and empty windows, and write explicit tests for the expiry logic.

### Design judgment

#### Q13. Compare RAG, in-context learning, and fine-tuning for policy classification. When do you pick each?

Each one trades off speed, cost, and consistency differently.

In-context learning is the fastest to deploy. Put the policy and examples in the prompt, and a new policy can be enforced the same day. Each call is expensive, results depend on which examples you choose, and behavior can vary across formats.

RAG helps when the policy is long, changes often, or depends on precedent. For each ad you retrieve the relevant policy sections and similar past decisions, which keeps the prompt short and current. It adds retrieval latency, and a retrieval miss becomes a new failure mode.

Fine-tuning or distillation gives the lowest cost and the most consistent behavior at very high volume. It takes data and time, and it locks in the policy interpretation until the next refresh.

In practice the three form a lifecycle. Use ICL or RAG to launch quickly, then distill into a tuned model for volume, and keep the prompt path for the hardest or newest cases. Tie each choice to measurable requirements, and explain how you would move traffic between them safely.

#### Q14. How would you decide which enforcement decisions run fully automated and which need human review?

Decide with explicit criteria. Start with severity. Harm to users, such as scams or dangerous products, justifies more aggressive automation on the blocking side, while account-level suspensions deserve more caution. Reversibility comes next. Disapproving a single ad is cheap to undo, and suspending an advertiser is much harder to reverse. Automate only the score ranges where audited precision meets a per-policy target. Weigh advertiser impact too, since high-spend or long-standing accounts with good history may get a review before drastic actions. Appeal overturn rates by policy show where automation is getting it wrong.

Turn these criteria into a routing policy with three bands per policy: auto-enforce, human review, and auto-allow. Audit random samples from every band, including auto-allow, because that band is where missed violations hide. Revisit the bands when models or policies change.

You are trading reviewer capacity against error cost. In this role the goal is to shrink the human band while holding precision. Treat reviewers as a scarce resource and spend their time on the most uncertain, highest-stakes cases.

### Leadership and behavior

#### Q15. Tell me about a time you removed a manual bottleneck from an ML workflow, and how you got other teams to adopt your fix.

This question tests the core theme of the role, which is taking humans off the critical path without losing safety in an organization with many stakeholders.

Pick a real story where a manual step slowed every model update. Hand-run evals, manual threshold tuning, and ad hoc label requests all fit. Describe the situation and put a number on the cost, whether that is days per release, engineer hours, or missed incidents. Explain the design you chose and the alternatives you rejected. Then describe the safety checks that made automation acceptable to skeptics, such as gates, audit logs, and rollback.

Spend most of your time on adoption. Say who resisted and why, how you used data or a pilot to win them over, and what you changed based on their feedback. Close with measured results and what you would do differently.

Interviewers look for ownership, influence without authority, and honest reflection on mistakes. Name the specific decisions you made yourself, and connect the outcome to user or business impact.

## Prep checklist

- [ ] Read "Scaling Up LLM Reviews for Google Ads Content Moderation" until you can sketch its funnel, cluster, label, and propagate design from memory.
- [ ] Read "Advertiser Content Understanding via LLMs for Google Ads Safety" and think through when to classify at the advertiser level and when at the ad level.
- [ ] Skim Google's latest Ads Safety Report for the scale of the work and the problems Gemini is used on.
- [ ] Practice a 45 minute ML system design that takes new policy enforcement from days to minutes, with shadow mode and a staged rollout.
- [ ] Review distillation basics, including soft labels, rationale supervision, class imbalance, and evaluating students against human labels.
- [ ] Work out your view on agent-driven evals and prompt optimization, covering tuning and gating sets, slice gates, and rollback.
- [ ] Practice medium and hard algorithm problems on union-find, hashing, heaps, sliding windows, and streaming.
- [ ] Prepare 4 behavioral stories about a cross-team disagreement, removing a manual bottleneck, a launch that went wrong, and leading without authority.
- [ ] Plan for an in-person onsite, since the listing says one is required in most instances.

## Sources

- https://arxiv.org/html/2402.14590v1 (Google Ads Safety paper on scaling LLM reviews with funneling, clustering, and label propagation)
- https://arxiv.org/pdf/2409.15343 (Google Ads Safety paper on classifying advertiser intent with LLMs)
- https://blog.google/products/ads-commerce/2025-ads-safety-report (summary of Google's 2025 Ads Safety Report on Gemini-powered enforcement)
- https://support.google.com/adspolicy/answer/13584894 (Google help page on how automation and human review work together in ad moderation)
- https://research.google/blog/distilling-step-by-step-outperforming-larger-language-models-with-less-training-data-and-smaller-model-sizes (Google Research post on distilling LLM rationales into small models)
- https://arxiv.org/abs/2309.03409 (Google DeepMind OPRO paper on using LLMs to optimize prompts)
- https://igotanoffer.com/blogs/tech/google-machine-learning-engineer-interview (public guide to Google ML engineer interview rounds)
- https://www.datainterview.com/blog/google-machine-learning-engineer-interview (public guide to onsite structure and level expectations)
- https://igotanoffer.com/en/advice/google-team-matching (public guide to the Google hiring committee and team matching)
- https://www.reddit.com/r/leetcode/comments/1r4dq0g/senior_sofware_engineer_ml_applied_interview (anecdotal candidate discussion of the Senior SWE Applied ML loop)
