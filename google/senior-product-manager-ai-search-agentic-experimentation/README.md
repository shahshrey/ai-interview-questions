# Google: Senior Product Manager, AI Search Experience and Agentic Experimentation

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

No first-hand report exists for this exact team. The loop below combines Google's own PM candidate-prep and interview-tips pages with recent Senior PM reports on Glassdoor, Blind and aced.io. An L5 PM who passed wrote on Blind that round names and content are "not 1:1", so any round can include estimation, strategy or product sense. The team owns an agentic system that keeps changing the Search experience and raises the rate of UX experiments, so expect generative UI, experiment design and evaluation to run through every round.

1. Google Hiring Assessment (some candidates, about 30 minutes, online). Work-style statements and situational-judgment scenarios. AI: not allowed.
2. Recruiter screen (30 to 45 minutes). Background, product experience, why Google, level fit; one Glassdoor candidate was asked which AI tools they use and for what. AI: not relevant.
3. Hiring manager or PM screen (45 to 60 minutes, video). Behavioral questions plus a product case, sometimes with an estimate. AI: not allowed. Google's interview-tips page says Search and AI "are not permitted to be used during your interviews".
4. Analytical screen (some loops, 45 to 60 minutes). Metrics, A/B testing, diagnosing a drop. A Senior PM reported "Google Search is seeing a decline in engagement among teenagers" as the prompt. AI: not allowed.
5. Onsite: Product vision (45 minutes). Design or improve a product; Google's own example is the next great feature for Google Search. AI: not allowed.
6. Onsite: Product analysis (45 minutes). Estimation, metric choice, root cause and experiment design with counter-metrics. This is the core round for an experimentation PM. AI: not allowed.
7. Onsite: Strategic insights (45 minutes). Market, competition and monetization, often with heavy pushback. AI: not allowed.
8. Onsite: Execute with judgment and leadership (45 minutes each or combined). Launch or roll back, prioritization, influence without authority. AI: not allowed.
9. Problem space understanding (45 minutes or inside other rounds). Frame competing constraints before choosing. AI: not allowed.
10. AI fluency and AI product integration. Google's PM page lists both as scored components: judging AI output for accuracy, and embedding AI while balancing performance, reliability and usability. Expect them as follow-ups inside rounds 5 to 9 or as an AI system design round. AI: not allowed.
11. Possible AI prototyping round (about 45 minutes, some AI PM loops chosen by the hiring manager). Build a rough prototype with an AI builder while narrating scope and trade-offs. This req asks for an AI-native builder, so ask your recruiter. AI: provided, the builder is the point of the round.
12. Team matching, then hiring committee. AI: not relevant.

No current Google PM report includes a coding or classic algorithm round, so this set has no LeetCode questions. Most questions are design, diagnosis and worked math; the short Python blocks are the checks you would do on a whiteboard.

## Questions and answers

### Question index

| # | Question | Type | Round | AI policy | Difficulty |
|---|---|---|---|---|---|
| [Q1](#q1) | A user types "plan a 3 day trip to Kyoto with two kids under 8, mostly rainy days" in... | Design | Product vision | Not allowed | Hard |
| [Q2](#q2) | Leadership wants Search to run ten times more UX experiments with the same number of... | Design | Product vision | Not allowed | Hard |
| [Q3](#q3) | UX research wants the agent to test bold layouts, Search Quality wants no change that... | Design | Problem space understanding | Not allowed | Hard |
| [Q4](#q4) | Google Search is seeing a decline in engagement among teenagers. Here is the dashboar... | Trace debug | Analytical screen or Product analysis | Not allowed | Medium |
| [Q5](#q5) | An agent-generated layout for AI Mode answers shows a 3.1% lift in clicks after 14 da... | Trace debug | Product analysis | Not allowed | Hard |
| [Q6](#q6) | Pick the north-star and guardrail metrics that the experimentation agent is allowed t... | Concept plus code | Product analysis | Not allowed | Medium |
| [Q7](#q7) | Estimate how many UX experiments per week AI Mode can run on its traffic, and what it... | Concept plus code | Product analysis | Not allowed | Medium |
| [Q8](#q8) | AI Mode answers more questions directly, publishers say their traffic is falling, and... | Design | Strategic insights | Not allowed | Hard |
| [Q9](#q9) | ChatGPT search and Perplexity are adding agents that book, buy and fill forms for use... | Design | Strategic insights | Not allowed | Hard |
| [Q10](#q10) | A variant the agent built is live at 5% of traffic. Satisfied sessions are up 0.4 poi... | Design | Execute with judgment | Not allowed | Medium |
| [Q11](#q11) | The agent now proposes 400 variants a week, but Search Quality and accessibility revi... | Design | Execute with judgment | Not allowed | Medium |
| [Q12](#q12) | Design the evaluation for generated AI Mode interfaces before they reach users, and d... | Concept plus code | AI fluency | Not allowed | Hard |
| [Q13](#q13) | Screen reader users report that a new AI Mode layout skips the source links. The vari... | Trace debug | AI fluency | Not allowed | Medium |
| [Q14](#q14) | An AI assistant drafted this launch readout for a director review. Find the errors be... | Review | AI fluency | Not allowed | Medium |
| [Q15](#q15) | Generating a full interactive page with a large model can take a minute. Choose the m... | Design | AI product integration | Not allowed | Hard |
| [Q16](#q16) | Design the architecture of the agentic experimentation system end to end: from a hypo... | Design | AI system design | Not allowed | Hard |
| [Q17](#q17) | In 45 minutes, use an AI builder to prototype an AI Mode result card that compares th... | Build | AI prototyping (some AI PM loops) | Provided (AI builder) | Medium |
| [Q18](#q18) | Tell me about an agentic workflow you prototyped or shipped yourself. Which AI tools... | Behavioral | Hiring manager screen or leadership | Not allowed | Medium |

### Product vision

<a id="q1"></a>

#### Q1. A user types "plan a 3 day trip to Kyoto with two kids under 8, mostly rainy days" into AI Mode. Design the generative UI experience for this query: who it serves, what the model builds, and how you would know it works better than a ranked page of links.

Round: Product vision. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Generative UI for complex Search tasks.

<details>
<summary>Show answer</summary>

Task: pick the user and job, define what the generated interface contains, set its limits, and say how you would test it against today's results.

What they grade: a sharp user need, a scoped first version, trust and accuracy controls for generated content, and a measurement plan that goes beyond clicks.

Approach: the user is a parent planning on a phone in the evening, who will come back to the plan several times before the trip. The jobs are to see a feasible day by day plan, swap pieces when the weather or the kids change, and get to bookings or official pages. A ranked list makes them open eight tabs and assemble the plan themselves.

First version: the model generates an interactive itinerary with one card per half day. Each card holds an indoor option, travel time from the previous stop, and a source link for opening hours and prices. A rain toggle swaps outdoor stops for indoor ones, and a "kids under 8" filter is applied from the query. Every factual field (hours, prices, addresses) comes from a tool call to Search or Maps and carries a source; the model writes layout and connecting text only. That split follows Google Research's generative UI design, which gives Gemini tool access and runs post-processors on the output.

Alternatives: a fixed itinerary template filled by the model is fast and consistent but cannot handle "rainy plus small kids" combinations; free-form generated pages are the most flexible but slow and harder to check. A constrained component library that the model arranges wins: layouts vary per query while every component has tested accessibility and fact slots.

Verify: run it as an experiment on travel-planning queries. Primary metric: the share of sessions where the user saves, shares or returns to the plan within 7 days. Guardrails: clicks to source sites, reported errors per 1,000 plans, and time to first useful content. Rate 300 generated plans with human raters for feasibility (can a family actually do this day) and factual accuracy before launch.

Risks: generation latency, since Google Research reports its generative UI can take a minute or more; stream the first card within a few seconds. Stale facts such as closed attractions, which source links and refresh-on-open reduce.

Follow-ups: how would you decide which query classes get generated UI at all? What do publishers lose, and how do you measure it?

Why this is likely: Google's PM prep page uses "the next great feature for Google Search" as a product vision example, and Google Research says generative UI is rolling out in Search starting with AI Mode, which is this team's surface. Medium confidence on this exact prompt, high on the round.

</details>

<a id="q2"></a>

#### Q2. Leadership wants Search to run ten times more UX experiments with the same number of people. Design a product where agents propose, build and test interface variants. What does version one do, and what stays human?

Round: Product vision. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Agentic UX experimentation platform.

<details>
<summary>Show answer</summary>

Task: define the users of the system, the loop the agents run, the human checkpoints, and a version one that is useful within two quarters.

What they grade: understanding where experiment velocity is lost, sensible autonomy limits for a product with billions of users, and a vision where each test teaches the next one, beyond generating more variants.

Approach: the users are Search PMs, UX designers and analysts, and the end users are searchers. In most experiment programs, building a variant is only part of the delay; the larger costs are writing the hypothesis, waiting for traffic, reviews, and reading results. So the product has to speed up the whole loop: hypothesis, variant, review, launch, readout, learning.

Version one: an agent reads past experiment results and user research notes, proposes ranked hypotheses with the expected effect and the evidence behind each, and generates variants from the approved component library only. Automated checks (accessibility, latency, policy, visual diffs) run on every variant. A human approves each hypothesis before traffic, and analysts sign off on readouts. The agent then writes a draft readout and adds the result to a searchable memory of what worked, so the next proposal starts from evidence.

What stays human in version one: launching to more than 1% of traffic, any change to ads, any new component, and every final launch decision. Version two raises autonomy for low-risk surfaces once the agent's proposals have a track record, for example when agent hypotheses win at least as often as human ones over 100 experiments.

Alternatives: a tool that only generates variants faster speeds up the cheap part and floods reviewers; a fully autonomous optimizer launches on clicks and drifts toward clickbait layouts. A loop with human gates at hypothesis and launch wins, because it speeds up every step and keeps accountability with people.

Numbers: if Search runs 100 UX experiments a quarter with a median of 6 weeks from idea to readout, 10x means about 1,000 a quarter; traffic then becomes the limit (see Q7), so the agent must also choose smaller, better powered tests.

Risks: many small wins that add up to a cluttered page; long-term holdouts on 1% of traffic catch that.

Why this is likely: the job description says the team owns the strategy for an agentic system that keeps evolving Search's UX and increases the rate of experimentation, and Google's PM prep page lists product vision as a scored round. Medium confidence.

</details>


### Problem space understanding

<a id="q3"></a>

#### Q3. UX research wants the agent to test bold layouts, Search Quality wants no change that can move ranking signals, and the Ads team wants ad slots untouched. Frame the problem and define what the agent is allowed to change.

Round: Problem space understanding. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Change envelope across competing teams.

<details>
<summary>Show answer</summary>

Task: state each team's real concern, find the constraint behind it, and turn that into an explicit change policy the agent can follow and the teams can audit.

What they grade: framing before solving, separating stated positions from underlying needs, a policy concrete enough to implement, and a way to revisit it with data.

Approach: start with the needs. UX research needs room to learn, which means variants large enough to move behavior. Search Quality's concern is that layout changes alter click patterns, and clicks feed evaluation and possibly ranking, so a layout test can corrupt quality measurement. The Ads team needs ad position, labeling and density to stay within policy and revenue forecasts.

Then define a change envelope in three tiers. Tier 1, the agent may change freely within a test: spacing, typography within the design system, component order inside the AI response, and copy of non-ad labels. Tier 2, needs a named owner's approval: adding or removing a component type, changing how many organic results show above the fold, or any interactive element. Tier 3, outside the agent's scope: ad slots, ad labels, ranking order, and anything in the legal or policy areas. The policy lives as machine-readable rules that the variant checker enforces, and every rejected variant is logged so teams can see what the envelope blocks.

For Search Quality, add a measurement fix rather than a ban: tag sessions in layout experiments so click-based quality metrics exclude them or adjust for position.

Alternatives: one review board that approves every variant is safe but becomes the bottleneck the team exists to remove; giving each team a veto per test has the same problem with three queues. An explicit envelope with tiered approval wins because most variants pass automatically and the teams only see the ones that touch their areas.

Numbers: if 80% of agent variants land in tier 1, a program of 1,000 variants a quarter sends about 200 to human review, roughly 15 a week spread over three teams.

Risks: the envelope can freeze in place, so review it every quarter with data on what was blocked and what won.

Follow-ups: who owns the rule file? What happens when a tier 1 change moves ad revenue by 0.5%?

Why this is likely: Google's PM prep page lists problem space understanding, framing competing constraints such as technical versus business, and Blind Senior PM reports describe these trade-off cases inside the hiring manager screen. Medium confidence.

</details>


### Product analysis and metrics

<a id="q4"></a>

#### Q4. Google Search is seeing a decline in engagement among teenagers. Here is the dashboard the analyst sent. Find the root cause and propose what to do.

Round: Analytical screen or Product analysis. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Root-cause diagnosis of an engagement drop.

The dashboard (8 weeks, users the age model classifies as 13 to 17):

```text
metric                                   week 1     week 8     change
teen weekly active users                 20.0M      23.5M      +17.5%
teen queries                             300M       320M       +6.7%
queries per teen user                    15.0       13.6       -9.3%
same weeks last year, queries per user                         -1.0%
all-ages weekly active users                                   +1.0%

fixed cohort (classified teen under both age model v3 and v4)
queries per user                         15.0       14.4       -4.0%
  homework and education queries         60.0M      51.6M      -14.0%
  entertainment queries                  45.0M      45.5M      +1.1%
  AI Mode turns on homework topics       6.0M       12.0M      +100%

release log
week 5   age model v4 rolled out to 100% (recall on 13 to 17 up)
week 5   AI Mode follow-up suggestions launched for education queries
```

<details>
<summary>Show answer</summary>

Task: separate measurement changes from real behavior change, size each, and recommend actions for the real part.

What they grade: checking the metric definition and segment before blaming users, using the fixed cohort, seasonality, and a recommendation tied to the cause.

Approach: first, the segment changed. Age model v4 raised recall in week 5, and teen weekly users jumped 17.5% while all users grew 1%. The new users it pulls in are lighter searchers, so queries per user falls even if nobody changed behavior. The fixed cohort removes that: -4.0% instead of -9.3%, so about 5 of the 9 points are a mix shift in who counts as a teen.

Second, seasonality. Last year the same weeks fell 1%, so about 3 points of the drop is a change in behavior.

Third, where those 3 points went. Entertainment is flat; homework queries fell 14% in the cohort, 8.4M queries. In the same weeks, AI Mode turns on homework topics doubled, up 6M, after follow-up suggestions launched in week 5. Teens are asking follow-ups inside one AI Mode conversation instead of typing new queries. Total homework interactions went from 66M to 63.6M, down 3.6%, so most of the drop is consolidation and a small part may be real loss.

Recommendation: report engagement on tasks or satisfied sessions instead of raw queries for AI Mode users, and restate the teen trend on the fixed cohort. For the remaining loss, check external panel data for teen use of other AI assistants for homework, and interview 20 teens on why they chose each tool.

Verify: re-cut the dashboard with age model v3 on all weeks; the drop should shrink to about 4%. Pitfalls: proposing a teen marketing campaign before checking the definition; treating fewer queries as worse when tasks moved to AI Mode.

Follow-ups: what single metric would you put on the leadership dashboard for teen engagement now?

Why this is likely: a Senior PM candidate reported this exact prompt on Glassdoor for Google's analytical screen, and Google's PM prep page lists diagnosing usage changes under product analysis. High confidence on the prompt, medium on the data shown.

</details>

<a id="q5"></a>

#### Q5. An agent-generated layout for AI Mode answers shows a 3.1% lift in clicks after 14 days, and the agent's readout recommends launch. Here is the readout. Is the result trustworthy, and what do you do next?

Round: Product analysis. AI policy: Not allowed. Type: Trace debug. Difficulty: Hard. Topic: Experiment validity, SRM and novelty.

The readout (50/50 split by user, 14 days):

```text
arm         users assigned (logged)   clicks per user   p50 time to first render
control     1,000,000                 2.90              0.9 s
treatment     985,000                 2.99              2.4 s
lift in clicks per user: +3.1%   p < 0.001

daily lift in clicks per user, treatment vs control
day   1     2     3     4     5     6     7     8     9    10    11    12    13    14
lift +6.0  +4.1  +3.0  +2.4  +2.0  +1.6  +1.4  +1.2  +1.0  +0.9  +0.8  +0.7  +0.6  +0.6

exposure is logged when the AI Mode response finishes rendering
```

<details>
<summary>Show answer</summary>

Task: check whether the experiment is valid before reading the lift, explain each problem, and decide the next step.

What they grade: checking sample ratio before results, connecting the logging point to the bias, spotting novelty, and a clear decision.

Approach: first check the split. A 50/50 test should log about equal users; 1,000,000 against 985,000 is a sample ratio mismatch, and the check below gives a p-value far below 0.001. Some treatment users are missing from the logs.

The logging rule explains which ones. Exposure is logged when the response finishes rendering, and the treatment renders in 2.4 seconds against 0.9. Users who give up during the wait never get logged in treatment, and they are the least engaged users. The treatment arm is therefore filtered to more patient users, which inflates clicks per user. The 3.1% lift is biased upward by an unknown amount.

Second, novelty. The daily lift falls from 6.0% on day 1 to 0.6% by day 13 and 14. The 14-day average mixes a curiosity spike with a small steady state, so the long-run effect is closer to 0.6%, before fixing the bias.

Decision: do not launch on this readout. Fix exposure logging to fire at query time, rerun for at least 3 weeks, and add time to first render and abandonment as guardrails. Then also report the effect on users who are in their second week of exposure.

Code:

```python
from math import erfc, sqrt

def srm_p(n_a, n_b, share_a=0.5):
    total = n_a + n_b
    exp_a, exp_b = total * share_a, total * (1 - share_a)
    chi2 = (n_a - exp_a) ** 2 / exp_a + (n_b - exp_b) ** 2 / exp_b
    return chi2, erfc(sqrt(chi2 / 2))

chi2, p = srm_p(1_000_000, 985_000)
assert chi2 > 100 and p < 1e-20, (chi2, p)
assert srm_p(1_000_000, 999_200)[1] > 0.05

daily = [6.0, 4.1, 3.0, 2.4, 2.0, 1.6, 1.4, 1.2, 1.0, 0.9, 0.8, 0.7, 0.6, 0.6]
first_week, second_week = sum(daily[:7]) / 7, sum(daily[7:]) / 7
assert second_week < first_week / 3
print(f"chi2={chi2:.1f} p={p:.1e} week1={first_week:.2f}% week2={second_week:.2f}%")
```

Pitfalls: trusting p < 0.001 on a broken split; averaging over a novelty curve; letting the agent that built the variant also judge it.

Why this is likely: Senior PM reports include A/B testing questions, and Google's PM prep page lists diagnosing metric and usage changes under product analysis; this team's system generates variants automatically, so judging readouts is daily work. Medium confidence.

</details>

<a id="q6"></a>

#### Q6. Pick the north-star and guardrail metrics that the experimentation agent is allowed to optimize for AI Mode layouts, then size a test that can detect a 0.2 point change in your primary metric.

Round: Product analysis. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Metrics and experiment sizing for agent-run tests.

<details>
<summary>Show answer</summary>

Task: choose a primary metric an agent cannot game easily, guardrails that catch harm, the randomization unit, and a correct sample size.

What they grade: metrics tied to user value rather than raw clicks, guardrails for the open web and ads, awareness that an optimizer exploits whatever it is given, and correct power math.

Approach: an agent will push hard on whatever metric it gets, so clicks per user is a bad target: a layout that hides the answer behind more clicks wins on clicks. Primary metric: satisfied session rate, the share of sessions that end in a long click (30 seconds or more on the destination) or an answer the user did not follow with a reformulation within 2 minutes. Baseline: 60%.

Guardrails, each with a fixed threshold the agent cannot change: query reformulation rate, time to first useful content, clicks to external sites (the open web must not lose traffic silently), ads revenue per thousand queries, user feedback reports per 10,000 sessions, and accessibility checks passing at 100%. A variant that wins the primary metric and breaks any guardrail is rejected automatically.

Randomize by signed-in user, not by query: the same user sees the layout many times and learns it, and query-level randomization mixes both layouts in one user's experience.

Sample size for 60% to 60.2%, two-sided alpha 0.05, power 0.8: about 941,000 users per arm. Search has the traffic, so the limit is how many tests can share it at once (Q7). Sessions from the same user are correlated, so compute the metric per user and analyze per user, or apply a design effect.

Code:

```python
from math import ceil

def n_per_arm(p1, p2, z_a=1.959964, z_b=0.841621):
    var = p1 * (1 - p1) + p2 * (1 - p2)
    return ceil((z_a + z_b) ** 2 * var / (p2 - p1) ** 2)

n = n_per_arm(0.600, 0.602)
assert 940_000 < n < 945_000, n
assert 3.9 < n / n_per_arm(0.600, 0.604) < 4.1
print("users per arm:", n)
```

Verify: halving the detectable effect quadruples the sample, as the second assert shows, so each test should state its minimum effect worth shipping before it starts.

Pitfalls: letting the agent pick or edit its own success metric; a primary metric with no open web guardrail; reading results daily without a sequential correction.

Follow-ups: how would you validate that satisfied session rate predicts long-term retention? What holdout would catch slow harm from hundreds of small wins?

Why this is likely: Google's PM prep page asks candidates to describe success metrics for their solutions and lists metric-change diagnosis under product analysis, and the req asks for evaluation frameworks for generative outputs. Medium confidence on the format.

</details>

<a id="q7"></a>

#### Q7. Estimate how many UX experiments per week AI Mode can run on its traffic, and what it costs the agent to generate and screen the variants. State every assumption.

Round: Product analysis. AI policy: Not allowed. Type: Concept plus code. Difficulty: Medium. Topic: Estimation of experiment capacity and cost.

<details>
<summary>Show answer</summary>

Task: build an estimate from traffic to concurrent tests to weekly throughput, then a cost estimate for variant generation, and name the levers.

What they grade: a clear chain of assumptions, the right bottleneck, sanity checks, and levers that follow from the math.

Approach: assumptions, stated out loud: 100 million signed-in AI Mode users a week (a round number for the exercise, not a Google figure), 10% kept out of all tests as a long-term holdout, tests need about 941,000 users per arm (Q6) for a 0.2 point effect, and each test runs 14 days with two arms.

Capacity: a user can be in one test per layer. Each test occupies about 1.9 million users for two weeks, so one layer of 90 million users holds 47 concurrent tests and finishes about 23 a week. Orthogonal layers (layout, copy, components inside the answer) can overlap if the changes do not interact, so 3 layers give about 70 tests a week. If teams accept a 0.4 point minimum effect, each test needs a quarter of the users and capacity goes up 4 times.

Cost: per hypothesis the agent generates 20 candidate variants at about 50,000 tokens each, so 1 million tokens. Autoraters then score each candidate on 30 test queries at about 4,000 tokens per rating, which is another 2.4 million. At an assumed blended price of 5 dollars per million tokens, that is about 17 dollars per hypothesis. Human review of the top 3 candidates at 10 minutes each is 30 minutes of a designer's time, which costs more than the model.

Code:

```python
users_week = 100e6
holdout = 0.10
per_arm, arms, weeks = 941_000, 2, 2
per_test = per_arm * arms
concurrent = int(users_week * (1 - holdout) // per_test)
per_week_one_layer = concurrent / weeks
assert concurrent == 47 and 23 <= per_week_one_layer <= 24
three_layers = 3 * per_week_one_layer
tokens = 20 * 50_000 + 20 * 30 * 4_000
cost = tokens / 1e6 * 5
assert tokens == 3_400_000 and cost == 17.0
print(concurrent, round(three_layers), cost)
```

Verify: sanity check with the opposite direction: 70 tests a week times 1.9 million users times 2 weeks is about 266 million user slots, which is 3 layers of 90 million, so the numbers agree.

Pitfalls: treating traffic as unlimited because Search is big; ignoring layer interactions; costing only tokens and forgetting reviewer time.

Follow-ups: which lever would you pull first, smaller tests or more layers, and why?

Why this is likely: a Blind Senior PM report says estimation can appear in any Google round, and the req asks for intuition on model cost and latency trade-offs. Medium confidence.

</details>


### Strategy

<a id="q8"></a>

#### Q8. AI Mode answers more questions directly, publishers say their traffic is falling, and ads need room on the page. As the PM for AI Search experience, what is your strategy for the next two years?

Round: Strategic insights. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Answer engines, the open web and monetization.

<details>
<summary>Show answer</summary>

Task: lay out the forces, pick a position, and make it measurable, with the trade-offs you accept.

What they grade: understanding the three-sided system (users, publishers, advertisers), a clear position rather than a list, and metrics that would show if the strategy fails.

Approach: the forces. Users increasingly expect a direct answer, and competing AI assistants give one. Publishers create the content the answers depend on; if they lose traffic, they publish less or block crawling, and answer quality falls over time. Advertisers need intent signals and placements; an answer that ends the session removes clicks on both organic results and ads.

Position: AI Mode should answer simple facts directly and, for complex tasks, send users to sources with visits that are more useful to them. For questions with one answer (time zones, unit conversions) the answer is the product. For tasks such as planning, comparing or learning, the generated interface should send users to sources at the moment they need depth: the recipe's full method, a review site's test data, a booking page.

I would make three moves. First, design citations as part of the interface: cards that link to the source for each claim, measured by click-through to sources per AI Mode session. Second, give publishers data: a report on how often their content appears in AI answers and how many visits it drives. Third, place ads where commercial intent is explicit, as clearly labeled options inside task flows such as "compare these strollers", and keep them out of pure informational answers.

Numbers to watch: external clicks per 1,000 AI Mode sessions, by category, against classic Search; the share of top publishers allowing crawling; ads revenue per 1,000 queries in AI Mode against classic Search.

Alternatives: maximize direct answers (wins short-term usage, erodes the content supply), or keep AI Mode small to protect the current model (cedes the shift to competitors). The task-plus-sources position wins because it keeps all three sides growing.

Risks: users may skip sources anyway; if external clicks per session fall more than 20% for two quarters in a category, change the design for that category.

Follow-ups: the CEO asks for a single number that says the strategy is working. Which one?

Why this is likely: aced.io Senior PM reports describe a combative strategic insights round on competition and monetization, and Google's PM prep page lists market and monetization questions. Medium confidence.

</details>

<a id="q9"></a>

#### Q9. ChatGPT search and Perplexity are adding agents that book, buy and fill forms for users. Where should AI Mode compete on agentic tasks first, and where should it stay out for now?

Round: Strategic insights. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Competitive strategy for agentic Search.

<details>
<summary>Show answer</summary>

Task: compare the competitors' strengths with Google's, choose a first segment for agentic tasks, and say what you skip and why.

What they grade: honest assessment of where Google is strong, a segment chosen with criteria, and awareness of trust and partner risks when an agent acts for a user.

Approach: criteria for a task category: how often users search for it, whether Google has structured data and partners for it, how bad a mistake is, and whether the result can be checked by the user before it commits.

Google's strengths: query volume that shows intent early, structured data in Maps, Shopping and Flights, and many existing merchant integrations. Competitors' strengths: a chat-first habit for long tasks and fewer legacy constraints on the page.

First segment: start from what has shipped. AI Mode already books restaurants and, in Labs, event tickets and local appointments through partners like OpenTable and Resy. Those tasks are frequent, Maps already has the listings, a mistake is cheap to undo, and the user confirms the time and party size before the agent books. The next segment to win is shopping comparisons that end at the merchant's checkout, where the agent builds the shortlist and the merchant keeps the transaction.

Stay out for now: financial or medical forms. Mistakes there are costly and hard to undo, and liability is unclear. Revisit when the agent's error rate on reservations is proven below an agreed bar, for example 1 failed booking per 1,000. Flights and hotels: go only with partner-backed booking and a confirm step, since Google has said they are coming; complex fares make errors expensive.

Alternatives: launch broad agentic browsing across any website (matches competitor demos but fails often on unfamiliar sites, and every failure is a public trust story), or wait until the technology matures (cedes the habit to competitors). Narrow, partner-backed categories win because they combine Google's data advantage with a failure rate you can control.

Numbers: tasks completed per 1,000 sessions in the category, booking failure rate, undo rate, and partner-reported fraud or no-show rates.

Risks: partners may not want an agent between them and their customer; give them the booking record and the customer relationship, and measure partner churn.

Follow-ups: a competitor ships flights booking before AI Mode's partner-backed version is ready and press coverage is strong: do you speed up the launch or hold the confirm-step bar?

Why this is likely: aced.io reports describe strategy questions about competing with free alternatives, and igotanoffer's Google PM guide includes AI competition cases; this team's agentic work makes the topic natural. Medium confidence.

</details>


### Execution

<a id="q10"></a>

#### Q10. A variant the agent built is live at 5% of traffic. Satisfied sessions are up 0.4 points, but user feedback reports doubled from 3 to 6 per 10,000 sessions and two publishers complained it hides their attribution. Ramp, hold or roll back?

Round: Execute with judgment. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Launch decision with mixed signals.

<details>
<summary>Show answer</summary>

Task: weigh the evidence, make a decision with a time limit, and say what would change it.

What they grade: a clear call, reading the complaints as data, protecting partners, and a plan that keeps the win if it is real.

Approach: first, what the numbers mean. A 0.4 point gain in satisfied sessions is a real win if the test is valid, so I would confirm the sample ratio and novelty checks from Q5 before anything else. Feedback reports at 6 per 10,000 sessions is still a small number of users, but doubling is a clear signal and I read a sample of 200 reports to see what they say. The publisher complaint is specific and checkable: compare the variant's citation display with the control on the same queries.

Suppose the reports mostly mention that sources are hard to find, and the comparison shows citations moved from inline to a collapsed footer. Then the two problems are one cause, and the gain may partly come from users not leaving to sources.

Decision: hold at 5%, do not ramp, and do not roll back yet. Ask the agent for a variant that keeps the new layout but restores inline citations, and run it against both arms for two weeks. Tell the two publishers what we found and when we will decide. If the new variant keeps at least 0.3 points of the gain with reports back near 3 per 10,000 and external clicks at control level, ramp it. If not, roll back.

Also fix the system: add "citation visibility" to the automated checks so the agent cannot ship a variant that hides attribution again, and add external clicks per session to its guardrails.

Alternatives: ramp now because the primary metric won (ignores a clear harm signal and partner trust), or roll back immediately (throws away a gain that may be separable from the harm). Hold and repair wins because it costs two weeks and answers the question.

Risks: the repaired variant may lose the whole gain; then the gain came from hiding sources, which is a result worth knowing.

Follow-ups: a VP wants the ramp this week for a launch event. What do you say?

Why this is likely: Google's PM prep page lists execute with judgment with launch decisions, and Glassdoor Senior PM reviews mention execution and trade-off rounds. Medium confidence.

</details>

<a id="q11"></a>

#### Q11. The agent now proposes 400 variants a week, but Search Quality and accessibility review can handle 60. Engineering wants to skip review for small changes; the reviewers want a cap. What do you prioritize and how?

Round: Execute with judgment. AI policy: Not allowed. Type: Design. Difficulty: Medium. Topic: Velocity versus review capacity.

<details>
<summary>Show answer</summary>

Task: find where review time goes, decide which variants need a human, and set a rule both sides can live with.

What they grade: using numbers instead of positions, risk-based prioritization, automation of the routine part, and a way to adjust the rule with evidence.

Approach: start with data on the last 4 weeks of reviews. Suppose 70% of variants changed only spacing, type size or order within the answer, and reviewers rejected 2% of those, while variants adding or removing components were rejected 25% of the time. Then review effort is spent mostly where it finds little.

Rule: three lanes. Lane A (automated only): changes inside the tier 1 envelope from Q3 that pass the automated checks for accessibility, latency and visual regressions. Lane B (sampled review): tier 1 changes where a check is borderline, with 1 in 5 reviewed. Lane C (always reviewed): new components, anything touching citations or ads, and variants targeting sensitive query classes such as health or news.

Then cut the volume. 400 proposals a week is more than traffic can test (Q7 estimates about 70 tests a week across three layers), so the agent ranks proposals by expected effect and evidence, and only the top 100 get built.

Numbers: of 100 built variants, about 70 go to lane A, 15 to lane B (3 reviewed), and 15 to lane C. That is 18 reviews a week, well under 60, so reviewers can spend longer on lane C and on auditing lane A samples.

Alternatives: skip review for all small changes (fast, but no check on whether the automated checks catch real problems), or cap the agent at 60 variants (simple, but limits velocity to review capacity and wastes the automation). Lanes win because they put human time on the highest-risk changes.

Verify: audit 5% of lane A variants each month; if auditors find problems in more than 2%, tighten the lane.

Risks: lane boundaries can be gamed by splitting a big change into small ones; so the checker measures the cumulative difference from control across all steps.

Follow-ups: who owns the lane rules, and how do reviewers escalate a problem with a launched lane A variant?

Why this is likely: Google's PM prep page lists prioritization and trade-offs under execute with judgment, and Blind Senior PM reports describe trade-off and diagnosis cases. Medium confidence.

</details>


### AI fluency

<a id="q12"></a>

#### Q12. Design the evaluation for generated AI Mode interfaces before they reach users, and decide whether an autorater can replace human raters for it. Use the agreement data below.

Round: AI fluency. AI policy: Not allowed. Type: Concept plus code. Difficulty: Hard. Topic: Evaluating non-deterministic generative UI.

Agreement on 200 generated pages labeled "acceptable" or "not acceptable":

```text
                         autorater: acceptable   autorater: not acceptable
human: acceptable                 150                        10
human: not acceptable              14                        26
```

<details>
<summary>Show answer</summary>

Task: define what "good" means for a generated interface, build the rating set and rubric, and decide where the autorater is trusted.

What they grade: rubric dimensions that match user value, handling non-determinism, judge calibration with a proper agreement measure, and a decision with conditions.

Approach: rubric with five dimensions scored separately: factual accuracy of every claim against its source, task completion (can the user finish the task on this page), usability and accessibility, latency to first useful content, and attribution (each claim links to a source). A page fails if any of accuracy, accessibility or attribution fails, regardless of the other scores.

Rating set: 1,000 queries sampled by query class (travel, learning, shopping, health, how-to), with Google Research's PAGEN set of expert-made pages as a quality reference for the same kind of prompts. Because generation is non-deterministic, generate each query 3 times and report the share of queries where all 3 pass, not just the average.

Autorater: on the table, raw agreement is 88%, which looks good. But 75% of pages are labeled acceptable by both, so chance agreement is high (69%); Cohen's kappa is about 0.61, the bottom edge of 'substantial' and well below the 0.8 a launch gate needs. The autorater also passed 14 of the 40 pages humans rejected, a 35% miss rate on failures, which are the pages that matter.

Decision: use the autorater to screen candidates and rank variants, where a miss only costs a review. Keep human raters for launch gates and for the accuracy and health dimensions. Retrain or reprompt the autorater on the 14 misses and re-measure; trust it for gates only when kappa reaches 0.8 and its miss rate on human failures is under 10%.

Code:

```python
def kappa(a, b, c, d):
    n = a + b + c + d
    po = (a + d) / n
    pe = ((a + b) * (a + c) + (c + d) * (b + d)) / n ** 2
    return po, (po - pe) / (1 - pe)

po, k = kappa(150, 10, 14, 26)
miss_rate = 14 / (14 + 26)
assert abs(po - 0.88) < 1e-9 and 0.60 < k < 0.62 and miss_rate == 0.35
print(f"agreement={po:.2f} kappa={k:.2f} miss_rate={miss_rate:.0%}")
```

Pitfalls: reporting raw agreement; one generation per query; an autorater that sees the expected answer and agrees with it.

Follow-ups: how do you keep the rating set from going stale as the agent learns what raters like?

Why this is likely: Google's PM prep page defines AI fluency as critically assessing AI outputs for accuracy, and the req asks for systematic evaluation frameworks for non-deterministic generative outputs. High confidence on the topic.

</details>

<a id="q13"></a>

#### Q13. Screen reader users report that a new AI Mode layout skips the source links. The variant came through the agent pipeline with every check marked green. Read the trace, find what went wrong, and give the product fixes.

Round: AI fluency. AI policy: Not allowed. Type: Trace debug. Difficulty: Medium. Topic: Failure analysis in an agentic pipeline.

```text
run 7f3a  hypothesis H-512
  10:02:11 planner      read memory: "exp-2231 collapsed source chips -> satisfied +2.0% (win)"
  10:02:12 planner      hypothesis: collapse source chips into a footer drawer, expect +1.5%
  10:02:40 generator    variant v1 built from components: AnswerBlock, SourceDrawer(new)
  10:03:05 check.visual  pass  (diff 12% vs control, under 15% threshold)
  10:03:09 check.latency pass  (p50 first render 1.1 s)
  10:03:39 check.a11y    timeout after 30 s  -> status=skipped
  10:03:39 gate          all required checks: pass (skipped counted as pass)
  10:03:40 autorater     score 4.6 / 5  "clean, modern, less clutter"
  10:03:41 launch        1% traffic, layer=layout

memory record exp-2231
  change: collapse source chips into footer
  primary metric: satisfied sessions  -0.4%  (ci -0.7 to -0.1)
  secondary: clicks on answer           +2.0%
  decision: not launched
```

<details>
<summary>Show answer</summary>

Task: walk the trace step by step, name each failure, and turn each into a product change, not just a bug ticket.

What they grade: reading evidence in order, separating model errors from system design errors, and fixes that prevent the class of problem.

Approach: there are three failures, and they stack.

1. The planner misread memory. The record says satisfied sessions fell 0.4% and the experiment was not launched; the +2.0% was clicks on the answer, a secondary metric. The planner quoted the wrong number as a win and built a hypothesis on it. Fix: memory returns structured fields (primary metric, effect, interval, decision), and the planner must quote the decision field verbatim in the hypothesis.

2. The accessibility check timed out and the gate counted "skipped" as "pass", so the new SourceDrawer component was never checked and screen reader users were hit. Fix: gates fail closed. Skipped, timed out or errored means blocked, and a new component (marked new in the build) always requires a passing accessibility check plus a human review under the tier rules from Q3.

3. The autorater judged looks only. Its comment praises "less clutter", and hiding sources produces less clutter. Fix: add attribution and accessibility to the autorater rubric (Q12), and give it the accessibility tree, not just a screenshot.

Verify: replay run 7f3a through the fixed pipeline; it should stop at the planner (contradicted memory) and again at the gate. Then audit every launch from the last 30 days that had a skipped check.

Pitfalls: blaming only the model; fixing the timeout without changing the fail-open rule; rolling back this variant without searching for others with the same gap.

Follow-ups: how would you tell the affected users and the accessibility team? What metric shows the pipeline is safer after the fix?

Why this is likely: Google's PM prep page scores AI fluency as judging AI outputs critically and AI product integration as balancing reliability and usability, and this team runs exactly this kind of multi-step agent pipeline. Medium confidence.

</details>

<a id="q14"></a>

#### Q14. An AI assistant drafted this launch readout for a director review. Find the errors before it goes out, and rewrite the recommendation.

Round: AI fluency. AI policy: Not allowed. Type: Review. Difficulty: Medium. Topic: Reviewing AI-written experiment readouts.

```text
Launch readout: Generated comparison tables in AI Mode (exp-3310)

1. Result: satisfied sessions rose from 58.0% to 58.9%, a 0.9% improvement.
2. p = 0.03, so there is a 97% chance the variant is better than control.
3. The effect holds across segments: mobile +0.6 pts, desktop +0.4 pts,
   and the overall gain of 0.9 pts is driven by both.
4. Guardrails: latency and ads revenue were neutral.
5. As required by the Search Generative Layout Standard v3, all tables passed
   the 4.5:1 contrast rule.
6. Recommendation: launch to 100% of traffic in all markets next week.

Appendix: traffic share in test, mobile 70%, desktop 30%.
```

<details>
<summary>Show answer</summary>

Task: find every factual, statistical and process error, explain why each matters, and write a recommendation the director can act on.

What they grade: catching statistical misreadings, checking that the numbers agree with each other, spotting a citation that may not exist, and noticing what is missing.

Approach: errors in order.

1. Units: 58.0% to 58.9% is 0.9 percentage points, or a 1.6% relative improvement. Mixing them confuses every later comparison.
2. p-value: p = 0.03 is the probability of a result this large if there were no true effect. It is not a 97% chance the variant is better. Report the effect with its confidence interval.
3. Segments do not add up: a weighted average of +0.6 (70% of traffic) and +0.4 (30%) is +0.54 points, not +0.9. Either the segment numbers or the total is wrong, and the data has to be pulled again before anything else.
4. Missing guardrails: nothing on external clicks to sources, reformulation rate, or feedback reports, which Q6 makes required. Tables that answer comparisons directly are the kind of change that can reduce clicks to publishers.
5. Citation: I cannot find a "Search Generative Layout Standard v3", and the AI may have invented it. The 4.5:1 contrast ratio is the WCAG AA rule for normal text, so cite WCAG and attach the check results instead.
6. Recommendation skips steps: all markets at once, with no ramp, and the test probably ran only in some markets and languages.

Rewritten recommendation: "Satisfied sessions rose 0.9 points (interval to be added after the data pull). Segment numbers do not match the total, so we are re-pulling the data. Pending that and the missing guardrails, we propose ramping to 10% in tested markets for two weeks, with a launch decision on a set date."

Verify: recompute the weighted average yourself; confirm the experiment's markets from its config; search the internal docs for the cited standard before keeping any reference.

Pitfalls: fixing wording and missing the arithmetic; trusting a confident citation; letting the readout go out under your name unchecked.

Follow-ups: how would you change the readout template so the assistant cannot omit guardrails?

Why this is likely: Google's PM prep page defines AI fluency as critically assessing AI outputs for accuracy, and the hiring manager screen for this team will expect you to review AI-made work. Medium confidence.

</details>


### AI product integration and system design

<a id="q15"></a>

#### Q15. Generating a full interactive page with a large model can take a minute. Choose the model and generation approach for generative UI at Search scale, and set the latency and cost budget.

Round: AI product integration. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Model choice and cost-latency trade-offs.

<details>
<summary>Show answer</summary>

Task: compare generation approaches on quality, latency and cost with explicit assumptions, then pick a routing design and budgets.

What they grade: realistic latency math from output length and decode speed, cost at scale, a design that matches approach to query, and the quality bar that decides the routing.

Approach: assumptions for the exercise, not Google figures: 10 million generated pages a day; a full page is about 6,000 output tokens; a large model decodes about 100 tokens a second and a small model about 400; output prices of 10 dollars and 0.6 dollars per million tokens.

Three approaches. (A) Large model writes the whole page: about 60 seconds, which matches Google Research's note that its generative UI can take a minute or more, and about 6 cents per page. (B) Small model writes the whole page: about 15 seconds and a fraction of a cent, but quality drops on complex layouts. (C) A model writes a 600-token layout spec that picks and fills components from a tested library: about 1.5 seconds on the small model and a tiny cost, with accessibility built into the components.

Design: route by need. C is the default for most queries. A runs only for queries that need something the library cannot express, such as a custom simulation, and it streams: show a C layout in under 2 seconds, then swap in the richer page when it is ready. Cache generated pages by query cluster for popular tasks.

Budgets: first useful content under 2 seconds at p50 and 4 at p95; full page under 20 seconds for route A; average cost per page under half a cent.

Code:

```python
pages_per_day = 10_000_000
options = {"A large full page": (6000, 100, 10.0),
           "B small full page": (6000, 400, 0.6),
           "C small layout spec": (600, 400, 0.6)}
out = {}
for name, (tokens, tps, price_per_m) in options.items():
    out[name] = (tokens / tps, tokens * price_per_m / 1e6)
assert out["A large full page"] == (60.0, 0.06)
assert out["C small layout spec"][0] == 1.5
mix = 0.95 * out["C small layout spec"][1] + 0.05 * out["A large full page"][1]
assert mix < 0.005
print({k: (round(s, 1), round(c, 5)) for k, (s, c) in out.items()}, round(mix * pages_per_day))
```

Numbers: with 95% on C and 5% on A, the average page costs about 0.3 cents, or roughly 33 thousand dollars a day, against 600 thousand if every page used A.

Pitfalls: quoting cost without latency; ignoring input tokens for long system instructions (caching helps); picking one model for every query.

Why this is likely: Google's PM prep page scores AI product integration as balancing performance, reliability and usability, and the req asks for intuition on foundation model cost and latency profiles. High confidence on the topic.

</details>

<a id="q16"></a>

#### Q16. Design the architecture of the agentic experimentation system end to end: from a hypothesis to a launched test to a learning the next run can use. Name the agents, tools, data and control points.

Round: AI system design. AI policy: Not allowed. Type: Design. Difficulty: Hard. Topic: Multi-step agent orchestration for experimentation.

<details>
<summary>Show answer</summary>

Task: draw the pipeline, assign each step to an agent, a tool or a person, define the shared data, and show how failures are contained.

What they grade: a clear decomposition, deterministic checks where possible, explicit human gates, memory that improves future runs, and the observability needed to debug it.

Approach: six stages, each with one owner.

1. Planner agent: reads the experiment memory and user research, proposes hypotheses with expected effect, evidence links and the metric it will move. Output is a structured record, not free text.
2. Human gate 1: a PM approves or edits hypotheses in a queue (lanes from Q11).
3. Generator agent: builds variants as layout specs from the component library (Q15). Tools: the component catalog, a preview renderer, and Search results for test queries.
4. Checks, all deterministic and fail closed: accessibility on the rendered tree, latency on a fixed device profile, visual diff against control, policy rules from the change envelope (Q3), and the autorater for quality (Q12).
5. Experiment service: the existing A/B platform assigns traffic within layers, enforces guardrails, and stops tests that breach them automatically.
6. Analyst agent: drafts the readout with SRM and novelty checks built in (Q5), and a human analyst signs it off. Results are written back to memory as structured fields.

Control points: every stage writes to a run log with inputs, outputs and versions of prompts and models, so any launch can be traced as in Q13. A kill switch per layer stops all agent tests at once.

Numbers: at 100 built variants a week, the checks run about 3,000 rendered previews (30 test queries each); at 5 seconds each that is about 4 hours of compute a week, trivial next to traffic limits.

Alternatives: one large agent that plans, builds and judges in a loop is simpler to start but grades its own work and is hard to debug; a fixed scripted pipeline without agents is reliable but cannot propose hypotheses. Separate agents with deterministic checks between them win because each step can be tested and swapped.

Risks: memory poisoning from a wrong readout (only signed-off readouts enter memory); agents optimizing toward the autorater instead of users (refresh the human rating set every quarter).

Follow-ups: which part would you build first, and what would you fake in version one?

Why this is likely: igotanoffer's Google PM guide describes AI system design questions, and the req prefers experience with agentic workflows, multi-step orchestration and autonomous tool use. Medium confidence.

</details>


### Prototyping round

<a id="q17"></a>

#### Q17. In 45 minutes, use an AI builder to prototype an AI Mode result card that compares three options for a user's query, and narrate what you check in its output.

Round: AI prototyping (some AI PM loops). AI policy: Provided (AI builder). Type: Build. Difficulty: Medium. Topic: Prototyping with AI tools.

<details>
<summary>Show answer</summary>

Task: ship a clickable prototype with sample data, show a clear delegation plan, and catch at least one wrong output from the builder.

What they grade: scoping in the first minutes, prompts that state constraints, reading the generated code and numbers, and trade-offs narrated as you go.

Approach: minutes 0 to 5, scope out loud. One query ("compare three kid-friendly rainy day activities in Kyoto"), one user (the parent from Q1), sample data as a JSON file with name, price, travel time, opening hours, age range and a source URL for each option. No live APIs.

Minutes 5 to 20, delegate. My prompt: "Build a single-page app that reads activities.json and shows a comparison card with three columns. Rows: price, travel time, hours, age range. Every row value links to its source URL. Add a toggle 'under 8 only' that hides options whose minimum age is above 7. Use semantic HTML table markup with headers, and no external libraries." I read the code before running it.

What I check: the filter logic, the links, and the accessibility markup. Wrong output I caught: the filter used `min_age < 7`, which hid an activity for ages 7 and up that should stay visible; I changed it to `min_age <= 7` and added a sample row with min_age 7 to prove it. The builder also built the table from div elements, so a screen reader would not announce headers; I asked for a real table with th cells.

Minutes 20 to 35: add a "why these three" line that names the source for each pick, and an empty state when the filter removes everything.

Minutes 35 to 45: narrate trade-offs. A fixed comparison component (fast, testable, accessible) against fully generated layouts (flexible, slower, harder to check), and why the prototype supports the component approach from Q15.

Verify: the age 7 boundary row, the empty state, every link opening the right source, and keyboard navigation through the table.

Pitfalls: letting the builder choose the scope; demoing without reading the code; using real brand data you cannot check in 45 minutes.

Follow-ups: what would you test with users first, and how many sessions would you run?

Why this is likely: a first-hand Reddit report from a Google AI PM candidate describes a vibe coding round, and Product Alliance and igotanoffer report Google adding prototyping rounds for AI PMs; this req asks for an AI-native builder. Low to medium confidence; ask your recruiter.

</details>


### Leadership

<a id="q18"></a>

#### Q18. Tell me about an agentic workflow you prototyped or shipped yourself. Which AI tools did you use, where did the AI get it wrong, and how did you present the risk to senior leadership?

Round: Hiring manager screen or leadership. AI policy: Not allowed. Type: Behavioral. Difficulty: Medium. Topic: AI-native building and leadership communication.

<details>
<summary>Show answer</summary>

Task: one true story covering what you built, how you used and checked AI, a mistake you caught, and how you brought leadership along.

What they grade: hands-on building rather than only managing, honest verification habits, clear communication of uncertainty to executives, and a result with numbers.

Approach: use a situation, action, result shape, and spend most of the time on the build and the leadership moment. Name the tools plainly and say what each was for.

Example shape: "Our support team spent hours tagging feedback before each planning cycle. I built a prototype agent in two weekends with a coding assistant and a model API: it read feedback, grouped it into themes, and drafted a summary with links to examples. I wrote 50 labeled examples first so I could score it. On that set it matched my themes 84% of the time, but it also invented a theme called 'billing confusion' from two tickets that were actually about login. I added a rule that every theme needs at least five linked tickets, and the next run had no invented themes on the test set.

When I presented it to our VP, I led with the numbers: 84% agreement, what the failures looked like, and the cost of a wrong theme (a misdirected sprint). I proposed using it to draft the summary while a PM checks every theme against its tickets, and measuring time saved. The VP approved a quarter-long pilot. Time to prepare planning input went from about 3 days to 1, and two teams adopted it."

The story works because you built something yourself, you measured it before trusting it, and you told leadership the limits before they found them.

Alternatives for the story: an AI feature you stopped or scoped down after testing (strong if you show the data), or an evaluation you designed that changed a launch decision.

Pitfalls: describing only prompts with no measurement; a story where the AI was perfect; hiding the failure from leadership; claiming you avoid AI tools.

Follow-ups: what would you do differently with a bigger budget? How do you decide when an agent can act without review?

Why this is likely: a Glassdoor candidate reported being asked which AI tools they use and for what in a Google screen, and the req prefers an AI-native builder with experience presenting to senior leadership. Medium confidence.

</details>

## Prep checklist

- [ ] Use AI Mode in Search for a week on real tasks and write down where generated layouts help, slow you down or get facts wrong.
- [ ] Read Google Research's generative UI post until you can explain how it works, how it was evaluated, and its known limits.
- [ ] Practice one product vision case end to end in 40 minutes: user, jobs, first version, metrics, risks.
- [ ] Be able to size an A/B test out loud, and explain sample ratio mismatch, novelty effects and guardrail metrics.
- [ ] Practice one metric-drop diagnosis: check the metric definition and logging first, then mix shifts, then segments.
- [ ] Prepare an eval design for generated UI with a human rating set and an autorater calibrated with kappa.
- [ ] Rehearse one cost and latency chain (tokens to seconds to dollars) with every assumption stated.
- [ ] Sketch an agentic experimentation pipeline with human gates, fail-closed checks and a run log.
- [ ] Build one small prototype with an AI builder and practice narrating what you checked in its output.
- [ ] Prepare three stories: something you built with AI yourself, a launch you held or rolled back, and influencing leadership with data.
- [ ] Ask the recruiter whether your loop includes a prototyping round and whether any round is in person.

## Sources

- Google careers, PM candidate prep: https://www.google.com/about/careers/applications/candidate-prep/pm
- Google careers, interview tips: https://www.google.com/about/careers/applications/interview-tips/
- Google Research, generative UI: https://research.google/blog/generative-ui-a-rich-custom-visual-interactive-user-experience-for-any-prompt
- Glassdoor, Google Senior Product Manager interview questions: https://www.glassdoor.com/Interview/Google-Senior-Product-Manager-Interview-Questions-EI_IE9079.0,6_KO7,29.htm
- Blind, Google L6 PM interview: https://www.teamblind.com/post/google-l6-pm-interview-5jw31uct
- aced.io (formerly Exponent), Google PM interview experiences: https://www.tryexponent.com/experiences/google-product-manager-interview-809eb4
- Reddit r/ProductManagement, Google PM vibe coding interview: https://www.reddit.com/r/ProductManagement/comments/1lw9r9h/i_messed_up_my_google_pm_vibe_coding_interview/
- Product Alliance, Google PM interview video: https://www.youtube.com/watch?v=okHVPTUChYQ
- igotanoffer, PM vibe coding interview: https://igotanoffer.com/en/advice/pm-vibe-coding-interview
- igotanoffer, Google product manager interview: https://igotanoffer.com/blogs/product-manager/google-product-manager-interview
- productislife, Google product manager interview: https://productislife.com/google-product-manager-interview
- Aakash Gupta, Google PM interview guide: https://www.news.aakashg.com/p/google-pm-interview-guide-2026
