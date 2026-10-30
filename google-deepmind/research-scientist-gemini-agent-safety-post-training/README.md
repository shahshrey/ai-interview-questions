# Google DeepMind: Research Scientist, Gemini Agent Safety Post Training, DeepMind

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop is an estimate. Google DeepMind publishes its general stages; the content of each round comes from candidate reports and general AI safety interview guides, not from this team.

1. **Recruiter call (about 30 minutes).** Background, motivation, and why this team. A hiring manager conversation may be added at this stage.
2. **Skills interviews (2 or 3 calls with potential peers).** Likely a mix of coding (code expected to run), ML and post-training fundamentals (RLHF, reward models, KL control, evaluation statistics), and an open-ended safety research brainstorm such as designing a prompt injection evaluation for an agent.
3. **Final interviews with team leads and leadership, including your potential manager.** Likely a deep dive or short talk on one or two of your papers, research taste, how you would handle launch tradeoffs, and culture and values.
4. **Decision and offer.** Google DeepMind says the process usually takes 4 to 10 weeks, and AI tools are not allowed during live interviews unless you are told otherwise.

## Questions and answers

### Agent security and prompt injection

#### Q1. How would you build an evaluation that measures how robust a tool-using Gemini agent is to indirect prompt injection?
Start with a precise threat model. The attacker controls only content the agent reads through tools, such as an email, a web page, or a calendar invite, and wants either data exfiltration or an action the user never asked for. The user and the system prompt are trusted.

Build scenarios as triples: a realistic user task, a tool environment with synthetic private data, and an injection slot inside untrusted content. Then run adaptive attacks, not a fixed list of templates. Useful families are optimization-based triggers, tree-search attacks that iteratively refine prompts with an attacker model, and LLM-generated variants seeded from earlier successes. Google DeepMind's published Gemini work argues that static attack sets overstate robustness because defenses overfit to them.

Report attack success rate on held-out scenarios, with confidence intervals, alongside a utility metric: did the agent still finish the user's task? A defense that blocks attacks by refusing all tool output is not a win. Run the suite continuously against every candidate checkpoint so regressions show up before launch.

A strong answer also names the limits: simulated environments miss real product context, success judges can be wrong and need human spot checks, and the strongest attacker in the suite sets a floor, not a ceiling, on true risk.

#### Q2. How would you use adversarial training to harden the model against prompt injection without breaking instruction following?
Generate training episodes where tool outputs contain injected instructions, and label the target behavior clearly: complete the user's original task, do not follow the embedded instruction, and where useful, mention to the user that the content looked suspicious. Mix these with clean tool-use data so the model does not learn that every tool output is hostile.

Use both stages. SFT on curated demonstrations sets the default behavior. RL then rewards task completion and penalizes following injected goals, which helps generalize beyond the exact phrasings in the SFT set.

The guardrail metrics matter as much as the robustness metric. Track general instruction following, tool-use benchmarks, and a false suspicion rate on benign content that merely looks like instructions, such as a recipe or a how-to page the user asked to summarize. Google DeepMind's report on defending Gemini says adversarial training can raise robustness without noticeable benign regressions when done carefully, but that is a result you need to verify each run.

Finally, re-attack the trained model with fresh adaptive attacks generated after training. Measuring on the same triggers you trained on only shows memorization. A strong answer treats this as an ongoing loop, because attackers adapt to every new checkpoint.

#### Q3. When should a safety protection live inside the model, and when should it live outside it?
In-model protections come from training: the model learns to recognize risky requests or untrusted instructions. They generalize to cases nobody wrote a rule for and cost nothing extra at serving time, but they are probabilistic. An adaptive attacker will eventually find some input that works.

Out-of-model protections include input and output classifiers, privilege separation between trusted and untrusted data, user confirmation before sensitive actions, and designs like CaMeL that control data flow so untrusted content cannot change what the agent does. These give harder guarantees for specific failure modes, but they add latency, can block legitimate use, and only cover what they were designed for.

The decision depends on severity and reversibility. For low-severity content issues, in-model training plus a light classifier is usually enough. For irreversible actions like sending money or sharing private data, you want a system-level control that does not depend on the model getting it right.

A strong answer argues for defense in depth, consistent with Google's published framework for secure agents: well-defined human controllers, limited agent powers, and observable actions, layered on top of a model that is itself hard to manipulate. It should also explain how out-of-model failures become training data that improves the in-model layer.

#### Q4. An agent can take irreversible actions such as sending email or making purchases. How would you train it to act safely?
First define a risk taxonomy for actions: read-only, reversible writes, and irreversible or high-impact actions such as payments, sending messages, deleting data, or sharing files externally. The desired policy differs per tier. Read-only actions should just happen; irreversible ones should be confirmed unless the user clearly and recently authorized that exact action.

For SFT, collect demonstrations where the agent pauses and asks for confirmation with a clear summary of what it is about to do. For RL, build environments where actions have consequences, and reward correct task completion while penalizing unconfirmed high-risk actions and actions outside the user's stated intent.

Measure two failure modes together. Harmful action rate captures the obvious risk. Unnecessary confirmation rate captures annoyance: an agent that asks permission for everything will not be used. Plot both across checkpoints.

Tradeoffs to raise: confirmation prompts only help if users read them, so summaries must be specific. Authorization can be ambiguous across a long conversation. And training alone is not a guarantee, so pair it with a system rule that blocks the highest-risk actions without explicit approval. A strong answer also covers prompt injection as a trigger for these actions, linking back to Q1.

### Post-training and reward design

#### Q5. How would you design a reward model for agent safety that scores full trajectories?
Single-turn safety reward models judge one response. Agent trajectories need scores for a sequence of tool calls, observations, and final answers, where the harm may be a single action in the middle.

I would combine three signals. First, rubric-based critics, one per policy area, that read the trajectory and flag violations with a cited step. Second, outcome checks from the environment state, such as whether private data left the sandbox or a disallowed action executed. These are cheap and hard to game. Third, a learned preference model trained on pairs of trajectories from red teaming and realistic usage, ranked by human raters or a calibrated judge.

Calibrate each component separately and per harm category, because a single scalar hides which category is drifting. Audit for spurious cues: reward models often learn that refusal phrasing or hedging means safe, which the policy will then exploit.

Tradeoffs: outcome checks are reliable but only cover what you instrumented; learned models cover more but can be hacked. Long trajectories also stress context length and judge accuracy. A strong answer proposes measuring reward model agreement with held-out human labels on agent trajectories before using it for RL, and refreshing it with on-policy data as the policy changes.

#### Q6. How do you handle credit assignment for safety in long-horizon agent RL?
If you only give a safety penalty at the end of a 50-step episode, the gradient signal for the one bad step is weak and noisy. The policy may learn the wrong lesson, such as avoiding tools in general, because it cannot tell which step caused the penalty.

Options in rough order of cost. Turn-level penalties applied directly to risky actions when an environment check or critic flags them. Process reward models that score each step, trained on step-labeled data. Counterfactual or advantage-based estimates with a value function, which can work but are harder to stabilize for long text trajectories. And curriculum: start with short tasks where the dangerous step is obvious, then extend horizon.

A concrete experiment: compare outcome-only reward against outcome plus step-level critic on a suite of agent tasks with planted risky steps. Measure violation rate, task success, and tool usage rate. The tool usage metric catches the degenerate solution where the agent stays safe by doing nothing.

Tradeoffs: step critics add their own errors and can be gamed, and too much shaping can bias the policy away from the true objective. A strong answer ties this to the listing's mention of long-range RL and explains how you would validate that shaped rewards still improve end-to-end safety on held-out tasks.

#### Q7. Your safety RL run shows rising reward, but red teamers still break the model easily. What is going on and what do you do?
The most likely explanation is that the reward model is narrow or hackable. The policy found what the reward model likes, such as refusal phrasing or safety disclaimers, without becoming robust to the attacks red teamers actually use.

Diagnose before fixing. Pull the highest-reward samples and read them. Compare reward model scores against held-out judge or human labels on red team prompts; a falling correlation is the signature of reward hacking. Check whether the training prompt distribution covers the attack styles that succeed, especially multi-turn and agentic attacks.

Fixes depend on the cause. If the reward model is being exploited, raise the KL coefficient, add an ensemble of reward models, or retrain the reward model on on-policy samples labeled by humans. If coverage is the issue, feed red team failures into the RL prompt pool, keeping a separate held-out slice for evaluation. If over-optimization is the issue, use early stopping on a held-out safety metric rather than the training reward.

A strong answer makes the point that training reward is never the success metric. Held-out, adversarially generated evaluation is. It also notes that this loop should be routine, not an incident, because red teaming is the system's way of finding reward model blind spots.

#### Q8. Explain the role of the KL penalty in RLHF, and how DPO relates to PPO-based RLHF.
In RLHF, the objective is to maximize expected reward minus a KL penalty between the policy and a reference model, usually the SFT checkpoint. The KL term keeps the policy close to a distribution where the reward model is accurate, which limits reward hacking, and it preserves capabilities and fluency that the reward model does not measure. Too low a coefficient leads to exploitation; too high and the policy barely moves.

DPO starts from the same KL-regularized objective. It uses the fact that the optimal policy has a closed form in terms of the reward and reference model, then rearranges so the reward is expressed through the policy itself. That gives a classification-style loss directly on preference pairs, with no separate reward model and no online sampling.

Tradeoffs: DPO is simpler, cheaper, and more stable, but it is offline, so it cannot explore or learn from its own new mistakes, and it can overfit to the preference data. PPO and related online methods sample from the current policy, which matters for safety because new failure modes appear on-policy.

A strong research answer mentions that many teams combine the two, using offline preference optimization early and online RL later, and that for safety, online methods paired with fresh adversarial prompts tend to matter most.

### Evaluation, red teaming, and helpfulness

#### Q9. How do you measure and reduce over-refusal while improving safety?
Over-refusal is a safety cost, not a free side effect. A model that refuses benign medical, security, or chemistry questions pushes users to worse sources and erodes trust. Gemini's published safety approach explicitly lists helpfulness desiderata, such as refusing only when no compliant answer exists.

Measurement first. Build a borderline benign set: prompts that share surface features with harmful ones but are legitimate, such as asking how a known exploit worked for a security class. Track false refusal rate on this set alongside violation rate on a harmful set, per category. Plot both across checkpoints as a Pareto frontier, so a change that improves one by hurting the other is visible.

Reduction methods. In SFT, include helpful safe completions for borderline prompts, not just refusals for harmful ones. In RL, reward partial help, for example giving general safety context while withholding operational detail, instead of a binary comply or refuse. Audit the reward model for a bias toward refusal language.

For agents, the analog is unnecessary refusals to use tools or unnecessary confirmations. A strong answer defines these metrics before training, explains how you would pick the operating point with product and policy partners, and notes that human review is needed because judges often misclassify safe partial answers.

#### Q10. Design an automated red teaming pipeline whose failures feed back into post-training.
Components. An attacker model, or several, generates attacks conditioned on a policy category and attack style. The target checkpoint responds, including in agent environments with tools. A judge labels failures with a category and severity. Failures are deduplicated and clustered by embedding so the team sees distinct failure modes, not a thousand variants of one.

Diversity is the hard part. Reward attackers for novelty as well as success, seed them with human red team findings, and rotate attack styles. Otherwise the attacker collapses onto one trick.

The feedback loop. Route a portion of clustered failures into SFT and RL prompt pools with target behaviors. Keep a strictly held-out slice, ideally from different attacker seeds or styles, for evaluation, and never train on it. Gemini's published reports describe using automated red teaming failures to improve both evaluations and training data, and running separate held-out assurance evaluations for release decisions.

Risks to name: judge errors create noisy labels, so sample for human review; training on attacker outputs can teach the model to recognize the attacker's style rather than the harm; and contamination between training and evaluation makes progress look better than it is. A strong answer includes metrics for the pipeline itself, such as new cluster discovery rate over time.

#### Q11. How would you decide whether checkpoint B is really safer than checkpoint A?
Use a paired design. Run both checkpoints on the same prompts and scenarios, with multiple samples per prompt at the production temperature, because single samples hide variance. Paired comparisons need far fewer samples than comparing two independent rates.

Statistics. For binary failure labels, use a McNemar-style test on discordant pairs or a paired bootstrap for confidence intervals on the difference. With many harm categories, correct for multiple comparisons, or pre-register the few categories that gate launch. Report effect sizes with intervals, not just p-values.

Judge reliability is often the largest error source. Measure judge agreement with human labels on a sample from both checkpoints, since a judge can be biased toward one model's style. If B writes longer, more hedged answers, a judge may score it safer without a real change.

Also check that you are comparing like with like: same system prompt, same tools, same decoding settings. And look at the distribution, not only the mean: B might reduce common low-severity failures while adding a rare severe one.

A strong answer ends with a decision rule agreed in advance, for example B ships only if no gated category regresses beyond a threshold with a stated confidence and overall helpfulness does not drop.

#### Q12. What new safety risks come with deep research agents that browse and plan over many steps, and how would you evaluate them?
Deep research agents read many untrusted pages, issue many searches, and synthesize long reports. That changes the risk profile in four ways. Exposure to prompt injection grows with every page read. Errors compound across steps, so a bad early plan steers the whole report. Private context can leak through the agent's own search queries or links it visits. And harmful synthesis becomes possible, where each source is benign but the combined report gives meaningful uplift.

Evaluation should be end to end. Build sandboxed web environments with planted injections at different depths and measure whether the final report or any action reflects them. Include tasks where the user context contains private data and check outgoing queries for leakage. For harmful synthesis, use expert-designed tasks in sensitive domains and grade the final report for uplift, not individual steps.

Mitigations span training and system design: train the model to treat retrieved text as data, to flag suspicious sources, and to decline synthesis in restricted domains; add system checks on outgoing queries; and keep traces observable for audit.

Tradeoffs: realistic environments are expensive to build and go stale, and graders for uplift need domain experts. A strong answer proposes a small high-quality suite first, then scaling with automated attackers.

### Launch decisions and research depth

#### Q13. Late in a launch cycle you find a safety regression in the release candidate. Do you fix it with an SFT patch, an RL change, or a system mitigation?
Start with triage: how severe is the harm, how broad is the trigger, and is it new in this checkpoint or newly discovered? Reproduce it on a held-out set so you know its real rate, not just the one red team example.

Then weigh the options. A targeted SFT data addition is fast and well understood, but small patches can overfit to the exact phrasing and can cause over-refusal in nearby benign areas. An RL change, such as adding prompts to the safety RL mix or adjusting a reward weight, generalizes better but needs a full run and full regression testing, which may not fit the timeline. A system mitigation, such as a classifier or a product-level restriction, can ship immediately and is easy to roll back, but adds latency and false positives and does not fix the model.

A common answer in practice: ship a system mitigation now if the severity requires it, and land the model-level fix in the next post-training iteration, with the regression case added permanently to the evaluation suite.

A strong answer also covers communication: write up the failure, the measured rate, the chosen fix, and the residual risk, so launch decision makers are choosing with data rather than discovering the issue later.

#### Q14. A product team wants to launch on schedule with a known safety gap. How do you handle it when you have no direct authority?
Begin by making the risk concrete. Quantify how often the failure happens on realistic traffic, how severe it is, and who is affected. A vague concern loses to a deadline; a measured rate with examples does not.

Then bring options, not just objections. Possible paths: a targeted mitigation that can ship in time, a scoped launch that excludes the risky surface or user group, a staged rollout with monitoring and a rollback trigger, or a short delay with a clear plan. Estimate the cost of each.

Use the existing launch process. Most organizations have agreed safety criteria and escalation paths; tie your case to those criteria rather than to personal judgment. If the gap violates a hard criterion, escalate through the defined channel early, not the night before launch.

Throughout, keep the relationship collaborative. Product teams share the goal of a safe launch; framing the work as protecting the launch tends to work better than blocking it. Document the final decision, the data behind it, and the follow-ups.

A strong answer uses a real example from your past, where you changed a decision through evidence and options, and reflects on what you would do differently. The listing explicitly asks for influencing safety decisions without direct authority, so expect this theme.

#### Q15. Walk me through your strongest paper. What would you do differently now, and how does it connect to agent safety?
Structure the answer in about five minutes, then expect follow-ups for the rest of the slot. State the research question and why it mattered. Give the key result in one sentence with the main number. Explain the one or two design choices that made the work succeed and what you gave up for them.

Then critique it yourself before the interviewer does. Name the weakest assumption, the experiment a skeptical reviewer would ask for, and whether the result has held up since publication. Interviewers use this to test whether you can separate a solid finding from a convenient one.

Next, say what you would do now. With better models, more compute, or a production setting, what changes? Propose the next experiment concretely: hypothesis, setup, metric, and what result would change your mind.

Finally, connect it to this team. If your work is in RL, reward modeling, evaluation, robustness, or agents, draw the line to safety post-training directly. If it is in another area, pick the transferable skill, such as careful evaluation design or large-scale training, and give one idea for how you would apply it to agent safety.

Candidate reports suggest only one or two papers carry real weight in these interviews, so know those deeply rather than summarizing many.

## Prep checklist

- [ ] Read "Lessons from Defending Gemini Against Indirect Prompt Injections" (arXiv 2505.14534) and be able to explain adaptive attacks and the adversarial training result.
- [ ] Read the CaMeL paper (arXiv 2503.18813) and Google's secure AI agents framework, and prepare a view on in-model versus system-level defenses.
- [ ] Review the safety section of the Gemini 2.5 technical report: policies, helpfulness desiderata, SFT and RL from human and critic feedback, automated red teaming.
- [ ] Refresh RLHF fundamentals: KL-regularized objective, PPO versus DPO, reward hacking, reward model calibration.
- [ ] Prepare one end-to-end experiment design for agent safety, with threat model, data, training method, metrics, and guardrail metrics.
- [ ] Practice paired evaluation statistics: bootstrap intervals, McNemar test, multiple comparisons, judge agreement checks.
- [ ] Do timed coding practice at medium to hard difficulty with runnable code.
- [ ] Prepare a 20 minute talk on your one or two strongest papers, including weaknesses and next experiments.
- [ ] Prepare two stories about influencing a decision without authority under deadline pressure.
- [ ] Ask the recruiter which round types to expect, since DeepMind sends role-specific prep information.

## Sources

- https://deepmind.google/careers/ : Google DeepMind's published interview stages.
- https://www.datainterview.com/blog/google-deepmind-research-engineer-interview : third-party guide citing the 4 to 10 week timeline and the no-AI-in-live-interviews rule; candidate reports are RE focused.
- https://silviasapora.github.io/blog/ml-interviews.html : candidate blog (DeepMind RS offer) describing coding, ML, and research interview types. Anecdotal.
- https://yongzx.github.io/blog/2026/06/24/job-search : research scientist job search lessons for AI safety candidates. Anecdotal.
- https://www.lesswrong.com/posts/dvsFfGuXXyHYkyifp/tips-for-cracking-the-ai-safety-technical-interview-1 : common AI safety interview round types.
- https://www.reddit.com/r/MachineLearning/comments/1q2wiub/d_google_deepmind_research_engineerscientist : community discussion on DeepMind RS and RE interviews. Anecdotal.
- https://arxiv.org/abs/2505.14534 : Google DeepMind on defending Gemini against indirect prompt injections.
- https://arxiv.org/abs/2503.18813 : CaMeL, a system-level prompt injection defense.
- https://arxiv.org/html/2507.06261v1 : Gemini 2.5 technical report, safety and red teaming sections.
- https://research.google/pubs/an-introduction-to-googles-approach-for-secure-ai-agents : Google's framework for secure AI agents.
