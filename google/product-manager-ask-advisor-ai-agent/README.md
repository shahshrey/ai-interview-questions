# Google: Product Manager, Ask Advisor AI Agent

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

No first-hand report exists for this exact role. The loop below combines Google's own PM candidate-prep and interview-tips pages with recent candidate and coach reports. A Google AI PM director quoted in Aakash Gupta's 2026 guide describes two routes for PMs: a standardized competency loop followed by team matching, and a specialized loop where the hiring manager picks the rounds. Ask Advisor is an agent product, so ask the recruiter early which route you are on.

1. Google Hiring Assessment (some candidates, about 20 to 30 minutes, online). Work-style statements and situational-judgment scenarios. AI: not allowed.
2. Recruiter screen (30 minutes). Background, motivation, level fit, a few behavioral and PM-philosophy questions. AI: not relevant.
3. PM or hiring manager screen (45 minutes, video). One product case, usually product vision plus problem space understanding, with follow-ups on metrics, estimation and how the product works technically. AI: not allowed. Google's interview-tips page says Search and AI "are not permitted to be used during your interviews".
4. Onsite: Product vision (45 minutes). Design or improve a product from user needs to a long-term vision; on an AI team, expect it to center on an AI product. AI: not allowed.
5. Onsite: Product analysis (45 minutes). Estimation, metric choice, root-cause diagnosis, and an A/B test with a counter-metric. AI: not allowed.
6. Onsite: Strategic insights (45 minutes). Market, competition, monetization, often with heavy pushback from a group PM. AI: not allowed.
7. Onsite: Execute with judgment (45 minutes, often paired with leadership stories). Launch, delay or sunset calls, prioritization, influence without authority. AI: not allowed.
8. Onsite: Problem space understanding (45 minutes). Frame competing constraints (technical versus business, a big customer versus the roadmap) before choosing; can run as two or three short scenarios. AI: not allowed.
9. AI fluency and AI product integration. These are new scored components on Google's PM page, and Google publishes no sample questions, so expect them as follow-ups inside rounds 4 to 8: judging AI output accuracy, AI limits, and adding AI features while balancing performance, reliability and usability. AI: not allowed.
10. Possible AI technical or AI system design round (45 minutes, some AI-team loops). Model choice, grounding, evals, latency and cost, agent architecture. AI: not allowed.
11. Possible prototyping round (about 45 minutes, specialized hiring-manager loops only; low likelihood for this team). Build a rough prototype with an AI builder while narrating trade-offs. AI: provided, the builder is the point of the round.
12. Team matching, then hiring committee. AI: not relevant.

No coding or classic algorithm round appears in any current Google PM report, so this set has no LeetCode questions. Most questions below are design, diagnosis and worked math, with two short code checks for metrics and evals that you would do on a whiteboard.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | Design Ask Advisor for a small business owner who has never run a paid search campaign... | Design | Product vision | Not allowed | Medium |
| [Q2](#q2) | Ask Advisor now spans Google Ads, Analytics and Merchant Center with shared memory. Pick... | Design | Product vision | Not allowed | Hard |
| [Q3](#q3) | Your largest agency customer wants Ask Advisor to make bulk autonomous changes across 400... | Design | Problem space understanding | Not allowed | Hard |
| [Q4](#q4) | Advertisers want a kill switch while leadership pushes for more automation. Design the... | Design | Problem space understanding | Not allowed | Hard |
| [Q5](#q5) | Recommendation acceptance fell 15% over a weekend. Read the dashboard and the log, find... | Trace debug | Product analysis | Not allowed | Medium |
| [Q6](#q6) | Estimate Ask Advisor's daily model calls and monthly serving cost, then name the two... | Design | Product analysis | Not allowed | Medium |
| [Q7](#q7) | Pick the north-star and guardrail metrics for Ask Advisor, then size the A/B test for a... | Concept plus code | Product analysis | Not allowed | Medium |
| [Q8](#q8) | Should Ask Advisor act on its own or only recommend, when more ad spend is also Google's... | Design | Strategic insights | Not allowed | Hard |
| [Q9](#q9) | Meta Advantage+ and agency tools also want to own the advertiser's daily workflow. Where... | Design | Strategic insights | Not allowed | Hard |
| [Q10](#q10) | General availability is one month away. CSAT is 3.9 against a 4.2 target and the... | Design | Execute with judgment | Not allowed | Hard |
| [Q11](#q11) | Advertisers reject most AI-generated image assets and policy flags are rising. Scale... | Design | Execute with judgment | Not allowed | Medium |
| [Q12](#q12) | Design the evaluation for Ask Advisor's answers, and decide whether the LLM judge can... | Concept plus code | AI fluency | Not allowed | Hard |
| [Q13](#q13) | Read this Ask Advisor trace. The agent told an advertiser to raise budget after a... | Trace debug | AI fluency | Not allowed | Medium |
| [Q14](#q14) | An AI tool drafted this one-page brief for an Ads VP on why the agent makes mistakes.... | Review | AI fluency | Not allowed | Medium |
| [Q15](#q15) | Set the latency and cost budget for a multi-step diagnosis answer, and show where the... | Design | AI product integration | Not allowed | Hard |
| [Q16](#q16) | Design the architecture of a cross-product ads agent: tools over Ads, Analytics and... | Design | AI system design | Not allowed | Hard |
| [Q17](#q17) | In 45 minutes, prototype a campaign diagnosis flow with an AI builder and narrate what... | Build | Prototyping (specialized loops) | Provided (AI builder) | Medium |
| [Q18](#q18) | Tell me about an AI feature you refused to ship, or shipped smaller, and how you... | Behavioral | Execute with judgment | Not allowed | Medium |

### Product vision

<a id="q1"></a>

#### Q1. Design Ask Advisor for a small business owner who has never run a paid search campaign. Who is the user, what is the first version, and how does it earn trust?

Round: Product vision. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Agent product design for novice advertisers.

<details>
<summary>Show answer</summary>

Task: Pick a sharp user, the jobs they hire the agent for, a first version, and a trust plan.

What they grade: a specific user with real pain, a prioritized set of jobs, a scoped first version, and metrics tied to the advertiser's outcome rather than usage.

Approach: I would pick a local service owner, for example a plumber spending a few hundred dollars a month, who set up a campaign once, does not know what a bid strategy is, and checks the account on a phone between jobs. The jobs, ranked by pain: "am I wasting money", "why did my calls drop", "set up something that works", and "fix the disapproved ad". The first two are diagnosis, which is where a novice is most stuck and where a wrong answer costs only time.

First version: a weekly plain-language check-up that answers "is this working" in three sentences with one suggested fix, plus a chat that can explain any number in the account. Every suggestion shows the evidence (the metric, the date range, the comparison) and an Apply button, matching the approval-first model on Google's Ask Advisor help page. In version one, every change waits for the advertiser's click.

Alternatives: a full campaign builder first (high value, but a novice cannot judge a generated campaign and early mistakes burn trust), or a pure education chatbot (safe, but does not touch their money, so it does not change outcomes). Diagnosis plus one-click fixes wins because it is useful in week one and every fix is reviewable.

Verify: success is the share of novice accounts still active after 90 days and their cost per lead, compared with a holdout of similar new accounts. Guardrails: undo rate on applied suggestions, thumbs-down rate, and policy disapprovals caused by agent edits.

Risks: overconfident answers on thin data (an account with 6 clicks a week needs "too early to tell"), and suggestions that raise spend without raising results.

Why this is likely: Google's PM prep page lists product vision cases for a named user group, and Search Engine Land reports that Ask Advisor targets marketers without deep analytics expertise (high confidence for format, medium for this exact prompt).

</details>

<a id="q2"></a>

#### Q2. Ask Advisor now spans Google Ads, Analytics and Merchant Center with shared memory. Pick the next capability to build and defend it against two alternatives.

Round: Product vision. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Cross-product agent roadmap.

<details>
<summary>Show answer</summary>

Task: Choose one capability that only a cross-product agent can deliver and argue for it with a sizing.

What they grade: a choice grounded in a real cross-product pain, alternatives compared on the same yardstick, and a way to tell if it worked.

Approach: The unique asset is that one agent can see campaign data, site analytics and the product feed together. The most common expensive question an advertiser asks is "why did my results drop", and the root cause often sits outside Google Ads: a broken conversion tag in Analytics, or products disapproved in the Merchant Center feed. Today the advertiser has to check three products by hand.

I would build cross-product root-cause diagnosis: when a performance drop is detected, the agent checks tag health, feed disapprovals and campaign changes, and returns a ranked list of causes with evidence and the fix it can apply with approval.

Alternatives: (a) campaign launch from the product feed, which Google already demoed at Google Marketing Live (a prompt like "find new customers for my hair care products" pulls product details from Merchant Center); valuable, but it already exists as a path, so the marginal gain is lower. (b) Proactive weekly insights across all three products; good for engagement, but without diagnosis depth it becomes another notification people ignore. Diagnosis wins because it answers the highest-stakes question and builds the plumbing (cross-product tool calls, evidence display) that (a) and (b) both reuse.

Numbers: if 1 in 5 active advertisers hits a drop of 20% or more in a month, and in an assumed 30% of those cases the cause is a tag or feed issue, then at several hundred thousand monthly users that is tens of thousands of fixable cases a month that today often end in a support ticket or lost spend.

Verify: time from drop to fix, share of diagnoses the advertiser confirms as correct, and recovered conversions versus a matched holdout.

Why this is likely: Google's prep page includes "next great feature" vision prompts, and the Google Marketing Live 2026 coverage describes the shared layer across Ads, Analytics and Merchant Center (medium confidence).

</details>

### Problem space understanding

<a id="q3"></a>

#### Q3. Your largest agency customer wants Ask Advisor to make bulk autonomous changes across 400 client accounts. It is not on the roadmap, and their Sales contact went straight to Engineering. What do you do?

Round: Problem space understanding. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Big customer versus roadmap.

<details>
<summary>Show answer</summary>

Task: Frame the competing constraints before deciding, then give a decision and a message to each party.

What they grade: naming the real constraints, separating the need from the requested solution, a decision with a reversible path, and how you repair the Sales to Engineering bypass.

Approach: First I would map the constraints out loud. Business: a large agency spends a lot and can move budget to a competitor. Product: Ask Advisor is built for a single account and, per Google's help page, is not available in manager (MCC) accounts, so multi-account work is a new surface to build. Risk: one wrong bulk change across 400 accounts is 400 incidents with 400 end clients. Team: Engineering has been pulled off plan without a PM decision.

Then I would talk to the agency to find the job behind the request. Usually it is "my account managers spend Mondays applying the same fixes"; they want those hours back.

Decision: offer a scoped pilot now: read-only multi-account diagnosis (which accounts need attention and why) plus a batch of suggested changes that an account manager approves per account in one review screen. That serves the job, exercises the manager-account plumbing, and keeps a human on every write.

Alternatives: (a) build what they asked, which creates a one-customer fork and the largest blast radius in the product; (b) say no and lose the learning; the pilot sits between them.

Numbers: if each account manager covers 25 accounts and spends 4 hours a week on routine fixes, 16 managers spend 64 hours a week; cutting that by half is the pilot's success bar.

Pitfalls: punishing the Sales rep; instead I would agree with Sales leadership on an intake path where big-customer asks come to the PM with revenue at stake, and tell Engineering their roadmap stands.

Why this is likely: Google's PM prep page lists problem space prompts about a big customer versus the roadmap and Sales going to Engineering directly, and aced.io describes the same scenario (high confidence for format).

</details>

<a id="q4"></a>

#### Q4. Advertisers want a kill switch while leadership pushes for more automation. Design the control model: autonomy levels, approvals, undo and audit.

Round: Problem space understanding. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Agent autonomy and control.

<details>
<summary>Show answer</summary>

Task: Resolve the trust versus automation tension with a concrete control model.

What they grade: autonomy set by the risk of each action; reversible actions; an audit story; numbers on blast radius.

Approach: Digiday's coverage of Google's 2026 ad agents frames trust as the product, and Search Engine Land names "how much operational control advertisers are willing to hand over" as the open question. Both sides are right, so I would make autonomy a per-action setting the advertiser controls.

Levels: L0 answer only. L1 suggest, and the advertiser applies with one click, which is the current model on the help page ("implement the suggested changes with your approval"). L2 auto-apply inside guardrails the advertiser sets, for example budget moves of at most 20% a day, pausing only keywords with zero conversions over 30 days, never touching bid strategy. L3 goal-driven autonomy, later and opt-in.

Risk tiers decide the default: renaming a campaign is low risk; a budget change spends real money; a final URL change can break conversion tracking. Higher tiers stay at L1 regardless of the setting.

Controls: a global pause that stops all agent writes instantly; an undo window of 7 days for every agent change, built on the account change history; a weekly digest of what the agent did and what it was worth.

Alternatives: one global autonomy toggle (simple, but forces all-or-nothing), or no autonomy at all (safe, caps the product). Per-action levels win because advertisers can grant trust gradually.

Numbers: cap blast radius per day. A 20% cap on a 100 dollar budget adds 20 dollars a day on average, but because Google Ads can spend up to twice the daily budget on a given day, one day can run up to 40 dollars over the old ceiling (240 versus 200); over a month the extra is capped at about 608 dollars (30.4 times 20). Both figures show in the digest.

Verify: share of advertisers who move a category to L2, undo rate per action type (above 5% means the default is wrong), and kill-switch usage.

Risks: guardrails that are too complex to set; ship three presets first.

Why this is likely: research maps a kill switch versus automation prompt to Google's problem space component and Digiday's trust coverage of the Google Marketing Live 2026 agents (medium confidence).

</details>

### Product analysis and metrics

<a id="q5"></a>

#### Q5. Recommendation acceptance fell 15% over a weekend. Read the dashboard and the log, find the cause, and say what you check before you call it.

Round: Product analysis. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Metric root-cause diagnosis.

```text
metric: accepted recommendations / shown recommendations (accepted = apply succeeded)
period          shown     apply_clicks   accepted   rate
Fri (baseline)  410,000   93,500         90,000     21.95%
Sat-Sun avg     402,000   92,800         75,300     18.73%

by recommendation type   share   rate_before   rate_after
keywords                 30%     24.0%         23.6%
creative                 30%     21.0%         20.8%
budget                   25%     23.0%         11.0%
bidding                  15%     18.0%         17.7%

log sample (Sat):
apply rec=budget  campaign=c_8812  status=FAILED  err=CONFIRMATION_REQUIRED
apply rec=budget  campaign=c_1173  status=FAILED  err=CONFIRMATION_REQUIRED
apply rec=keyword campaign=c_5521  status=OK
release note (Fri 18:00): budget changes above 20% now need a confirm step
```

<details>
<summary>Show answer</summary>

Task: Find the root cause from the evidence, confirm it, and decide the fix.

What they grade: checking the metric definition first, segmenting before guessing, reading clicks and outcomes together, and ruling out alternatives.

Approach: First, the metric itself: "accepted" counts only successful applies, so a failure in the apply path looks like users rejecting suggestions. Then the funnel: shown fell 2% and apply clicks fell less than 1%, so users still want the suggestions; the drop is between click and success. Segmenting by type, only budget fell (23.0% to 11.0%); the others moved by less than half a point. The weighted rates reproduce the headline: 21.95% before, about 18.7% after, a 14.7% relative drop. The log shows budget applies failing with CONFIRMATION_REQUIRED, and the Friday release added a confirm step for budget changes above 20%. My hypothesis: the agent's Apply path does not render or send the new confirmation, so the change fails silently.

Before I call it: compare with the last three weekends to rule out a weekend mix effect (agencies are less active on weekends); check whether the failure share of budget applies is near 100% for changes above 20% and near 0% below it; check that no logging change happened Friday.

Fix: roll back or patch the confirm step in the agent surface, then re-offer the failed suggestions to those advertisers, because their intent was clear.

Alternatives considered: a model change that made budget suggestions worse (would lower clicks, but clicks held), or seasonality (would hit all types).

Pitfalls: reporting "users trust budget suggestions less" from the rate alone; forgetting that failed applies may have left advertisers thinking a change happened.

Follow-ups: what alert would have caught this in an hour (apply failure rate by type), and who owns the agent's write path when another team changes the API.

Why this is likely: Google's PM prep page uses metric-drop diagnosis cases such as received email down 15% over a weekend, and research maps an acceptance-rate drop to that format (high confidence for format).

</details>

<a id="q6"></a>

#### Q6. Estimate Ask Advisor's daily model calls and monthly serving cost, then name the two biggest levers to cut it.

Round: Product analysis. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Estimation and unit economics.

<details>
<summary>Show answer</summary>

Task: Build a transparent estimate from users to tokens to dollars, then find the levers.

What they grade: a clear chain of assumptions, sanity checks, and levers ranked by impact.

Approach: Start from the job listing's scale: hundreds of thousands of advertisers a month. I state every assumption so the interviewer can change one.

Numbers:
- 400,000 monthly active advertisers, 25% active on a given day: 100,000 daily users.
- 2 conversations each, 6 turns each: 1.2 million turns a day.
- An agent turn makes about 4 model calls (plan, two tool-result reads, answer): 4.8 million calls a day, about 56 per second on average and roughly 170 at a 3x peak.
- Each call reads about 8,000 input tokens (account context, tool results, history) and writes 400: 38.4 billion input and 1.9 billion output tokens a day.
- At assumed internal prices of 0.30 dollars per million input tokens and 2.50 per million output tokens: about 11,500 dollars a day for input and 4,800 for output, about 16,300 dollars a day, or about 0.49 million dollars a month.

Sanity check: input dominates, because agents re-read context every call. That tells me where to cut.

Lever 1, prefix caching: the account summary and system instructions repeat across calls. If 70% of input tokens hit a cache billed at a quarter of the price, input cost falls from 11,500 to about 5,500 dollars a day, and the total from about 16,300 to about 10,300.

Lever 2, routing: classify the turn first; simple lookups ("how did campaign A do last week") go to a smaller model or a direct report query with no generation beyond a template.

Alternatives: a smaller model everywhere (cheapest, but diagnosis quality drops and wrong advice costs more than tokens), or precomputing a daily account digest (cuts repeated diagnosis calls, but wastes compute on accounts nobody opens). I would do caching first, routing second, digests only for the most-opened accounts.

Risks: cost per conversation can fall while answers get worse; track cost per resolved question.

Why this is likely: Google's PM prep page uses estimation cases such as Gmail messages per second, and AI PM question banks ask for serving cost and cost per request (medium confidence).

</details>

<a id="q7"></a>

#### Q7. Pick the north-star and guardrail metrics for Ask Advisor, then size the A/B test for a new suggestion ranker.

Round: Product analysis. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Metrics and experiment design.

<details>
<summary>Show answer</summary>

Task: Define metrics that reward advertiser outcomes, then design and size a fair test.

What they grade: a north star that cannot be gamed by raising spend, guardrails that catch harm, the right unit of randomization, and a correct sample size.

Approach: North star: advertisers who applied at least one suggestion in a week and whose goal metric (cost per conversion or return on ad spend, whichever they set) was no worse 14 days later. Acceptance rate alone misleads: a ranker that pushes easy budget increases gets accepted often and can still hurt results.

Guardrails: undo rate within 7 days, thumbs-down rate, policy disapprovals caused by agent edits, and the goal metric of accepted changes versus a holdout. Counter-metric for the business: spend can rise, but only when the advertiser's goal metric holds.

Test design: randomize by advertiser account, not by conversation, because one advertiser sees many suggestions and learns from them. Run at least two weeks to cover weekly cycles and the conversion lag. Spillover through the auction is small when treatment covers a small share of each auction.

Sample size for the primary rate: baseline 20%, minimum detectable effect 1 point, two-sided alpha 0.05, power 0.8.

Code:

```python
from math import ceil

def n_per_arm(p1, p2, z_alpha=1.959964, z_beta=0.841621):
    var = p1 * (1 - p1) + p2 * (1 - p2)
    return ceil((z_alpha + z_beta) ** 2 * var / (p2 - p1) ** 2)

n = n_per_arm(0.20, 0.21)
assert 25_000 < n < 26_000, n
assert n_per_arm(0.20, 0.22) < n / 3.5
print("advertisers per arm:", n)
```

Verify: about 25,600 advertisers per arm; doubling the effect to 2 points cuts the need by about 4x, so the first check is whether the ranker is worth testing at 1 point.

Pitfalls: counting suggestions as independent (inflates significance); stopping early when the curve looks good; ignoring the conversion lag.

Follow-ups: how to measure the long-term effect (a long-running holdout of 1% of accounts).

Why this is likely: aced.io's product analysis section and AI PM guides describe north-star plus guardrail and A/B test questions with a counter-metric (high confidence for format).

</details>

### Strategy

<a id="q8"></a>

#### Q8. Should Ask Advisor act on its own or only recommend, when more ad spend is also Google's revenue?

Round: Strategic insights. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Monetization and incentive alignment.

<details>
<summary>Show answer</summary>

Task: Take a position on autonomy given the conflict of interest, and show how the product earns trust anyway.

What they grade: naming the incentive problem directly, a long-term business argument, and mechanisms that make the incentive visible.

Approach: The advertiser suspects that an agent built by the company that sells the ads will suggest spending more. That suspicion is the main risk to adoption, bigger than model quality. So the agent acts on its own only where its objective is the advertiser's stated goal, and the product shows proof of that.

Position: recommend by default, act autonomously only on actions that are goal-bound and capped (Q4's L2), and keep every increase in total spend behind explicit approval.

Long-term business argument: advertisers who hit their target cost per acquisition keep spending and grow; advertisers who feel upsold cut budgets or leave. A spend-raising suggestion that hurts results is a short-term gain and a churn risk.

Mechanisms: (a) every suggestion shows the expected effect on the advertiser's goal metric as well as on clicks; (b) the agent also recommends cutting spend where it is wasted, and we track the ratio of spend-down to spend-up suggestions; (c) a quarterly report of outcomes of accepted suggestions against holdouts.

Alternatives: full autonomy for speed (wins on effort saved, loses trust), or recommend only forever (safe, but caps the value for small advertisers who will never review every suggestion).

Numbers: if 10% of advertisers who feel upsold cut spend by half, that loss can exceed the lift from aggressive spend suggestions; I would model it with the holdout data before choosing defaults.

Risks: internal pressure to tune the ranker on revenue; make advertiser goal metrics the launch criteria.

Why this is likely: Google's PM prep page lists monetization cases under strategic insights, and Digiday's coverage of the 2026 agents centers advertiser trust and control (medium confidence).

</details>

<a id="q9"></a>

#### Q9. Meta Advantage+ and agency tools also want to own the advertiser's daily workflow. Where should Ask Advisor compete, and where should it not?

Round: Strategic insights. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Competitive strategy.

<details>
<summary>Show answer</summary>

Task: Map the competitors, pick where to win, and name what to concede.

What they grade: a clear view of each player's advantage, a focused bet, and how you would know if it is working.

Approach: Three groups compete for the advertiser's daily attention. Meta's Advantage+ automates campaigns inside Meta's own inventory. Agency and third-party tools manage spend across many platforms and own the agency relationship. General AI assistants can read exported reports and give advice without account access.

Ask Advisor's advantages: first-party account data in real time, the ability to act inside Google Ads with approval, and the cross-product view across Analytics and Merchant Center. Its weakness: it only sees Google, and many advertisers spend on several platforms.

Where to compete: the in-account work for Google spend, especially for small and mid-size advertisers with no agency, where diagnosis plus one-click fixes beats any outside tool. Deep cross-product diagnosis (Q2) is something no outside tool can match.

Where not to compete: cross-platform budget allocation. A Google agent recommending how much to spend on Meta will never be trusted. Instead, expose clean APIs and let agency tools call Ask Advisor's diagnosis inside their workflow.

Alternatives: (a) build a cross-platform planner (credibility problem); (b) stay a chat helper (commodity); (c) the focused in-account agent plus open APIs, which I pick.

Numbers: measure share of Google Ads active accounts that use Ask Advisor weekly, segmented by agency-managed versus self-managed; win first where there is no agency.

Risks: agencies see the agent as a threat to their role; position it as leverage for account managers (Q3's pilot).

Why this is likely: aced.io's strategic insights section uses competitor prompts, and the Google Marketing Live 2026 recap positions Gemini as the layer across Google's ad stack (medium confidence).

</details>

### Execution

<a id="q10"></a>

#### Q10. General availability is one month away. CSAT is 3.9 against a 4.2 target and the hallucination rate on the eval set is 4% against 2%. Launch, delay or something else?

Round: Execute with judgment. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Launch decision under quality gaps.

<details>
<summary>Show answer</summary>

Task: Make a launch call with data, and say what you would need to see to change it.

What they grade: breaking the averages apart, a decision that limits harm, a clear path to the bar, and how you communicate it.

Approach: First, segment both metrics by intent. Averages hide where the risk is. Suppose the breakdown shows reporting and education answers at a 1.5% hallucination rate and CSAT 4.3, while billing, policy and budget-change answers sit at 8% and CSAT 3.4.

Decision: a split launch. Take reporting, explanation and education intents to general availability on schedule. Keep billing, policy appeals and write actions in beta, with the answer linking to the source page instead of summarizing it. Google's own help page already tells users to verify billing status and policy appeal outcomes directly in account settings, so this matches the product's stated limits.

Numbers: at 1 million answers a week, a 4% rate is 40,000 wrong answers. Splitting brings the launched slice to about 1.5%, while the risky slice stays with beta users.

Alternatives: (a) launch everything (meets the date, ships known harm where errors cost money); (b) delay everything (protects quality, but holds back the slice that already beats the bar and stalls learning). The split wins.

Path to the bar for the held intents: ground billing and policy answers on the account's own data and official policy pages, add refusal when grounding is missing, and re-test on a fresh eval sample; exit criteria are under 2% hallucination and CSAT above 4.2 for two consecutive weeks.

Verify: confirm the eval set is representative of live traffic before trusting 4% (Q12).

Pitfalls: moving the target to make the date; announcing general availability without telling sales teams which intents are excluded.

Why this is likely: Google's PM prep page lists execute-with-judgment prompts about launch decisions, and aced.io reports a launch one month out with CSAT below target (high confidence for format).

</details>

<a id="q11"></a>

#### Q11. Advertisers reject most AI-generated image assets and policy flags are rising. Scale it back, fix it or sunset it?

Round: Execute with judgment. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Sunset decisions for AI features.

<details>
<summary>Show answer</summary>

Task: Decide the future of a weak AI capability using evidence, and plan the exit or the fix.

What they grade: separating usage problems from value problems, cost of keeping it, and a decision with a dated review.

Approach: Ask Advisor can suggest new text and image creatives, and the help page notes that generated images come either from the landing page or from Google AI. I would first split the data by source, because these are two different features. Suppose landing-page images are accepted 35% of the time and generated images 8%, and generated images cause most of the policy flags (for example, invented product details or text rendering errors).

Questions before deciding: do accepted images perform? Compare click-through and conversion rate of accepted generated images against the advertiser's existing assets. If accepted ones perform as well, the problem is selection and quality, not value. Who uses it? If small advertisers with no designer accept more, the feature serves a real need for that segment.

Decision, given those numbers: scale back, not sunset. Keep landing-page images on by default. Move fully generated images behind an explicit request ("make me new images"), add a pre-check that rejects images with rendered text or products not in the feed, and limit to verticals with low policy risk.

Alternatives: sunset generated images (removes risk, loses value for advertisers with no creative assets), or keep it as is (rising policy flags hurt trust in the whole agent).

Numbers: target generated-image acceptance above 20% and policy flags below the rate for advertiser-uploaded images within one quarter; if not met, sunset that mode.

Pitfalls: judging by acceptance alone; forgetting to tell users why a capability changed.

Why this is likely: Google's PM prep page includes sunset decisions among execute-with-judgment examples, and research maps scaling back an agent capability to that prompt (medium confidence).

</details>

### AI fluency

<a id="q12"></a>

#### Q12. Design the evaluation for Ask Advisor's answers, and decide whether the LLM judge can replace human raters.

Round: AI fluency. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Agent evaluation.

<details>
<summary>Show answer</summary>

Task: Build an eval that tracks answer quality per intent, and test whether an automated judge agrees with humans well enough.

What they grade: a representative eval set, a rubric with grounding, judge calibration against humans, and how the eval gates launches.

Approach: Eval set: sample real conversations stratified by intent (reporting, diagnosis, optimization, policy, billing), plus hard cases written by ads specialists: thin data, conversion lag, conflicting metrics. Each case stores the account snapshot, so answers can be checked against facts.

Rubric, scored per answer: numbers match the account data; the cause is supported by evidence shown; the suggested action fits the advertiser's stated goal; refusal when data is missing. A hallucination is any stated fact not supported by the snapshot or an official policy page.

Judge calibration: 200 answers rated by both human ads specialists and the LLM judge. Raw agreement overstates it, so use Cohen's kappa, which corrects for chance.

Code:

```python
def kappa(both_pass, both_fail, judge_only_pass, human_only_pass):
    n = both_pass + both_fail + judge_only_pass + human_only_pass
    po = (both_pass + both_fail) / n
    pj = (both_pass + judge_only_pass) / n
    ph = (both_pass + human_only_pass) / n
    pe = pj * ph + (1 - pj) * (1 - ph)
    return (po - pe) / (1 - pe)

k = kappa(120, 40, 30, 10)
assert abs(k - 0.529) < 1e-3
assert abs(kappa(100, 100, 0, 0) - 1.0) < 1e-9
print(round(k, 3))
```

Here raw agreement is 80%, but kappa is 0.53, and the judge passes 30 answers humans failed, so it is lenient. Humans keep gating launches for now. Fix the rubric prompt using those 30 disagreements, re-test on a fresh 200, and require kappa above 0.7 per intent before the judge replaces humans for that intent. Humans keep rating a weekly sample.

Alternatives: thumbs-down rate only (cheap, but users cannot spot wrong numbers), or humans only (accurate, too slow for daily releases).

Pitfalls: an eval set built from easy chats; letting the judge see the expected answer and grade style instead of facts.

Why this is likely: Google's PM prep page describes AI fluency as judging AI output accuracy, and AI PM question banks ask how to evaluate a prompt and an agent's answers (medium confidence).

</details>

<a id="q13"></a>

#### Q13. Read this Ask Advisor trace. The agent told an advertiser to raise budget after a conversion drop. What went wrong, and what is the product fix?

Round: AI fluency. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Agent failure analysis in ads data.

```text
user: why did my conversions drop last week? should I spend more?
plan: get_performance(range=LAST_7_DAYS) vs PREVIOUS_7_DAYS
tool get_performance -> {
  "this_week": {"cost": 1410, "conversions": 31},
  "last_week": {"cost": 1395, "conversions": 52},
  "notes": ["conversion_lag_days_p50=4", "last 4 days incomplete"]
}
tool get_budget_status -> {"daily_budget": 250, "avg_daily_spend": 201,
  "limited_by_budget": false}
answer: Conversions fell 40% (52 to 31) while cost stayed flat. Your
  campaign is losing auctions. I suggest raising the daily budget from
  250 to 325 dollars to regain volume. [Apply]
user: clicked Apply
```

<details>
<summary>Show answer</summary>

Task: Find every error in the agent's reasoning and propose product-level fixes (tool contracts, guards) as well as prompt changes.

What they grade: ads domain knowledge, reading tool outputs against the answer, and fixes that prevent the class of error.

Approach: Three errors. First, conversion lag: the tool said the median lag is 4 days and the last 4 days are incomplete. Conversions in Google Ads are reported against the click date, so the most recent days fill in later. The 40% drop is mostly missing data, not a real drop. Second, the budget advice contradicts the tool: the campaign is not budget-limited (spending 201 of 250 a day), so raising the budget would not buy more volume. The agent invented "losing auctions" with no auction data. Third, the agent offered a spend increase with one click on a shaky claim.

Fixes, in order of leverage:
- Tool contract: return data freshness as structured fields, and have the tool itself return a comparison of complete periods by default.
- Planner rule: compare only complete periods, or compare same-lag windows.
- Action guard: a budget increase is offered only if the budget status says limited; otherwise the agent cannot attach an Apply button.
- Answer pattern: state uncertainty ("the last 4 days are still filling in; check again Thursday").

Alternatives: fix with a prompt instruction only (fast, but fails silently on the next phrasing), or remove budget suggestions entirely (safe, loses a high-value action). Structured fields plus an action guard win because they make the wrong answer impossible rather than less likely.

Verify: add 30 conversion-lag cases and 20 not-budget-limited cases to the eval set (Q12); the fix passes when no answer suggests budget increases on those.

Pitfalls: blaming the model; the tool gave the right signal and the system ignored it.

Why this is likely: research maps AI fluency to judging output accuracy, and Google's Ask Advisor help page lists "Why do I see a big drop in conversions?" as a core use case (medium confidence).

</details>

<a id="q14"></a>

#### Q14. An AI tool drafted this one-page brief for an Ads VP on why the agent makes mistakes. Find the errors before you send it.

Round: AI fluency. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: Explaining LLM limits to leadership.

```text
Brief: Ask Advisor quality (draft)
1. LLMs predict likely text, so they sometimes state facts not in the data.
2. Fix: add retrieval (RAG). RAG eliminates hallucinations because the
   model only answers from retrieved documents.
3. We will also set temperature to 0, which makes answers deterministic
   and therefore correct.
4. Our golden set shows 98% accuracy (49 of 50 cases), so we are ready.
5. Hallucinations fell from 4% to 2% last month, a 2% improvement.
6. Plan: fine-tune monthly on last month's chats so the agent learns
   new ads policies as they change.
7. Gemini carries a 99.9% factuality SLA for enterprise use.
8. Ask: approve general availability for all intents next month.
```

<details>
<summary>Show answer</summary>

Task: Catch the false or misleading claims and rewrite the brief so the VP can make a sound decision.

What they grade: understanding real LLM limits, statistics on small samples, spotting invented facts, and an honest ask.

Approach: Line 1 is fine. Line 2 is wrong: grounding on retrieved sources reduces unsupported claims, but the model can still misread or mix sources, so it does not eliminate them; keep the eval. Line 3 is wrong: temperature 0 makes output more repeatable, not correct; a wrong answer becomes a consistently wrong answer. Line 4 overclaims: 49 of 50 has an exact 95% interval of roughly 89% to 99.9%, so true accuracy could be near 89%, and 50 cases cannot cover five intents; report per intent on a larger set. Line 5 mixes units: 4% to 2% is 2 points, a 50% relative reduction; say both. Line 6 is the wrong tool: fine-tuning on last month's chats teaches past behavior, including past mistakes, and policies change faster than a monthly cycle; policies belong in retrieval from the official policy pages, updated when they change. Line 7 is unsupported: I know of no such public guarantee; I would delete it unless someone can produce the contract. Line 8 conflicts with the data; ask instead for a split launch (Q10).

Rewritten core message: what the agent gets wrong (unsupported facts in billing and policy answers), what we are doing (grounding, refusal when sources are missing, action guards), how we measure (per-intent eval with human-calibrated judge), and the decision needed (launch the intents that meet the bar).

Alternatives for the VP conversation: a live demo of failures (memorable, but anecdotal) or the per-intent table (less vivid, more decision-ready); use the table, with one real failure trace as the example.

Pitfalls: trusting a fluent draft because it sounds confident; keeping a number you cannot source.

Why this is likely: AI PM question banks ask candidates to explain LLM limits, grounding and evaluation to leaders, and Google's PM page scores judging AI output accuracy (medium confidence).

</details>

### AI product integration and system design

<a id="q15"></a>

#### Q15. Set the latency and cost budget for a multi-step diagnosis answer, and show where the time goes.

Round: AI product integration. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Latency and cost budgets for agents.

<details>
<summary>Show answer</summary>

Task: Turn "it feels slow" into a budget per step, then pick the changes that hit the target.

What they grade: targets tied to user expectations, an honest breakdown, alternatives with costs, and protecting quality while cutting time.

Approach: Targets come from the user's task. A diagnosis answer can take a few seconds if progress is visible; a lookup should feel instant. I would set: first useful text within 3 seconds at p95, full diagnosis within 8 seconds at p95, and lookups within 2 seconds.

Numbers, for an assumed sequential flow at p95: planning call 0.8 seconds; three reporting tool calls at 1.5 seconds each, 4.5 seconds; final answer 2.0 seconds. That totals 7.3 seconds before the user sees anything (adding p95s overstates the true p95 slightly, which is fine for a budget).

Changes, in order: run the three tool calls in parallel, since they are independent: 0.8 + 1.5 + 2.0 = 4.3 seconds. Stream the answer and show progress ("checking conversions, budget, policy"), so the first text appears after about 2.3 seconds plus first-token time. Route lookups to a fast path: an intent classifier sends "how did campaign A do" straight to a report query and a template, skipping the planner.

Alternatives: a single large call with all account data in context (one hop, but huge input cost and slower first token), or precomputed daily diagnoses (instant, but stale for an advertiser who just changed something). Parallel tools plus streaming win; precompute only the daily summary.

Cost: the budget per answer follows Q6; the fast path replaces the 4 model calls of a lookup turn with one small classifier call and a report query.

Risks: parallel calls raise peak load on the reporting backends; agree on quotas with those teams. A timeout must produce "I could not check X" rather than a confident partial answer.

Verify: track p50 and p95 per intent and per step, and check that quality on the eval set holds after routing.

Why this is likely: Google's PM page lists AI product integration as balancing performance, reliability and usability, and AI PM question banks ask about latency and cost budgets for multi-step agents (medium confidence).

</details>

<a id="q16"></a>

#### Q16. Design the architecture of a cross-product ads agent: tools over Ads, Analytics and Merchant Center, memory, permissions and policy checks.

Round: AI system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Agent architecture.

<details>
<summary>Show answer</summary>

Task: Draw the system, explain each component's job, and show where safety lives.

What they grade: clear boundaries, permission handling, memory design, write safety, and evals in the loop.

Approach: Components, left to right.
- Surface: the side panel and the search bar in Google Ads, which already serve as entry points.
- Orchestrator: a Gemini model that plans, calls tools, and writes answers; stateless per turn.
- Tools: typed wrappers over Ads reporting and mutate APIs, the Analytics data API and Merchant Center product status. Every tool returns structured data plus freshness and confidence fields (Q13).
- Permissions: every tool call runs with the signed-in user's access, never a service account, so the agent can see and change only what the user can.
- Memory: short-term conversation state, plus long-term saved info the user can view and edit, as the help page's "saved info" does, for example "I only care about ROAS". Memory is scoped per user and account and is never shared across advertisers.
- Write path: the agent proposes a change object; a policy checker validates it against ads policies and the user's autonomy settings (Q4); the user approves; the change is applied and logged with an undo record.
- Observability and evals: every turn is traced (plan, tool calls, answer); a sample feeds the eval pipeline (Q12).

Alternatives: one agent with all tools (simple, but the tool list grows and selection errors rise), or separate agents per product with a router (cleaner ownership, but cross-product questions need hand-offs). I would start with one orchestrator and product-scoped tool groups loaded by intent, and split only if tool-selection errors exceed a threshold on the eval set.

Numbers: if each product team exposes 10 to 15 tools, loading all 40 in every call costs context and accuracy; loading the 10 relevant to the intent keeps prompts smaller.

Risks: prompt injection through data the agent reads, such as product titles in the feed; treat tool output as data, never as instructions, and keep writes behind the policy checker.

Why this is likely: igotanoffer and dataford describe AI system design rounds for Google AI PM loops, and the Google Marketing Live 2026 coverage describes one agent across these products (medium confidence for the round, low for this exact prompt).

</details>

### Prototyping round

<a id="q17"></a>

#### Q17. In 45 minutes, prototype a campaign diagnosis flow with an AI builder and narrate what you check in its output.

Round: Prototyping (specialized loops). AI policy: Provided (AI builder). Type: Build. Difficulty: Medium. Topic: Prototyping with AI tools.

<details>
<summary>Show answer</summary>

Task: Ship a clickable flow that diagnoses a drop from sample data, and show judgment over the builder's output.

What they grade: scoping fast, a clear delegation plan, checking generated code and numbers, and narrating trade-offs.

Approach: Minutes 0 to 5: scope. One user (the small business owner from Q1), one question ("why did my conversions drop"), sample data as a CSV of daily cost, clicks and conversions for two campaigns, plus a budget status row. No real APIs.

Minutes 5 to 25: delegate the build. My prompt: "Build a single-page app that loads this CSV, compares the last 7 complete days with the 7 before, excluding the most recent 4 days because conversions lag, flags the campaign with the largest drop, and shows three possible causes with the evidence row for each. Show cost per conversion and conversion rate. No external APIs." I read the generated code before running it.

What I check: the date windows, the metric formulas, and any API it claims to call. Wrong output I caught: the builder computed cost per conversion as conversions divided by cost, so a worse campaign looked better; I fixed the formula and added a test row with known values (cost 100, conversions 4, expected 25). It also imported a "google-ads-diagnostics" package that does not exist; I removed it, since the round only needs local data.

Minutes 25 to 40: add the approval step: a suggested fix with Apply, which only writes to a local change log with an undo button.

Minutes 40 to 45: narrate trade-offs: sample data versus real API (speed versus realism), rules versus a model for the diagnosis (rules are testable in 45 minutes; a model needs an eval).

Verify: the known-value row, an empty-data case, and a campaign with zero conversions (no division by zero).

Pitfalls: letting the builder choose the scope; demoing without checking a single number.

Why this is likely: Aakash Gupta's 2026 guide quotes a Google AI PM director on hiring-manager loops with vibe coding, and Stellarpeers reports an L7 Gemini case with a prototype step; research rates this low for an Ads team (low confidence).

</details>

### Leadership

<a id="q18"></a>

#### Q18. Tell me about an AI feature you refused to ship, or shipped smaller, and how you brought Engineering and a policy or legal partner along.

Round: Execute with judgment. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: Influence without authority.

<details>
<summary>Show answer</summary>

Task: Tell a true story with a clear conflict, your specific actions, evidence, and a result with numbers.

What they grade: judgment under pressure, influence without authority, use of data rather than opinion, and what you learned.

Approach: Use a situation, task, action, result shape, but spend most of the time on actions. A strong story has a real cost to saying no: a date, a leader who wanted it, or revenue.

Example shape: "We had an AI summary feature ready for launch. Engineering wanted to ship, Sales had promised it to two large customers, and our policy partner was uneasy but had no data. I pulled 300 real outputs and had two specialists label them. 6% contained a number not in the source data, and most of those were in financial fields. I brought the labeled examples, not my opinion, to a meeting with the engineering lead and the policy partner. We agreed to ship the summary without financial figures, add a link to the source table, and gate the full version on a 1% error rate. I told Sales the same day, with the two customers' specific use cases covered by the smaller version. We launched on time; the full version shipped six weeks later at 0.8%."

What makes it land: you name the trade-off you accepted, show the data you gathered yourself, and explain how you kept the relationship with the person who disagreed. Close with what you would do differently, for example setting the error threshold before the build, not at the end.

Alternatives for the story: a time you shipped and it went wrong (shows learning, but make sure you own it), or a time you stopped a launch (strong, but show you offered a path forward).

Pitfalls: a story where everyone agreed; "we" with no "I"; no numbers; blaming the model or the engineers.

Follow-ups: what you would do if the VP overruled you; how you would detect the problem after launch.

Why this is likely: a 2026 Reddit offer report and aced.io describe behavioral stories inside the execution round, and AI PM question banks ask what you would refuse to ship (medium confidence).

</details>

## Prep checklist

- [ ] Read Ask Advisor's public help page and the Google Marketing Live 2026 coverage until you can describe what it does today, its entry points and its limits.
- [ ] Practice one product vision case end to end in 40 minutes: user, jobs, first version, metrics, risks.
- [ ] Learn the ads basics an agent PM is expected to know: conversion lag, budget-limited campaigns, bid strategies, manager accounts, policy disapprovals.
- [ ] Rehearse two estimation chains (usage to tokens to dollars) out loud, stating each assumption.
- [ ] Practice one metric-drop diagnosis: check the metric definition, the funnel, then segments.
- [ ] Be able to size an A/B test and say why you randomize by advertiser.
- [ ] Prepare a per-intent eval design and explain judge calibration with kappa.
- [ ] Draft an autonomy ladder for agent actions with caps, undo and audit.
- [ ] Prepare three stories: influence without authority, a launch you delayed or cut, and an AI output you caught being wrong.
- [ ] Ask the recruiter whether your loop is standard or specialized, and whether any round is in person.
- [ ] Practice every case with only your notes and your voice, since Google's interview page says AI tools are not permitted.

## Sources

- Google careers, PM candidate prep: https://www.google.com/about/careers/applications/candidate-prep/pm
- Google careers, interview tips: https://www.google.com/about/careers/applications/interview-tips/
- Google Ads Help, Ask Advisor in Google Ads (beta): https://support.google.com/google-ads/answer/16574983?hl=en
- Search Engine Land, Ask Advisor across Ads, Analytics and Merchant Center: https://searchengineland.com/google-launches-ask-advisor-across-ads-analytics-and-merchant-center-478114
- PPC Land, Google Marketing Live 2026 recap: https://ppc.land/google-marketing-live-2026-every-announcement-that-actually-matters
- Digiday, trust and Google's AI ad agents: https://digiday.com/marketing/trust-becomes-the-product-marketers-grapple-with-googles-new-suite-of-ai-powered-ad-agents
- Aakash Gupta, Google PM interview guide 2026: https://www.news.aakashg.com/p/google-pm-interview-guide-2026
- aced.io, Google product manager interview guide: https://www.aced.io/guides/google-product-manager-interview
- Reddit r/interviews, Google PM offer report: https://www.reddit.com/r/interviews/comments/1rw7eat/my_google_pm_interview_experience_just_landed_an
- productislife, Google product manager interview: https://productislife.com/google-product-manager-interview
- igotanoffer, Google product manager interview: https://igotanoffer.com/blogs/product-manager/google-product-manager-interview
- igotanoffer, AI product manager interview: https://igotanoffer.com/en/advice/ai-product-manager-interview
- Stellarpeers, Google L7 PM Gemini college assistant case: https://stellarpeers.com/case-collection/google-l7-pm-product-design-gemini-college-assistant
- Aakash Gupta, vibe coding interview: https://www.news.aakashg.com/p/vibe-coding-interview
- dataford, Google AI product manager guide: https://dataford.io/interview-guides/google/ai-product-manager
- calibreos, AI PM interview questions 2026: https://www.calibreos.com/blog/aipm-ai-pm-interview-questions-2026
