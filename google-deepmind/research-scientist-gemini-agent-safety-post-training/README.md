# Google DeepMind: Research Scientist, Gemini Agent Safety Post Training, DeepMind

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This loop is an estimate. Google DeepMind publishes its general stages. The contents of each round below come from candidate reports and general AI safety interview guides, and this team has not described its own rounds.

1. Recruiter call, about 30 minutes. It covers your background, your motivation, and why you want this team. A hiring manager conversation may be added at this stage.
2. Skills interviews, 2 or 3 calls with potential peers. Expect a mix of coding where your code has to run, ML and post-training fundamentals (RLHF, reward models, KL control, evaluation statistics), and an open-ended safety research brainstorm, such as designing a prompt injection evaluation for an agent.
3. Final interviews with team leads and leadership, including your potential manager. These likely include a deep dive or short talk on one or two of your papers, along with research taste, how you would handle launch tradeoffs, and culture and values.
4. Decision and offer. Google DeepMind says the process usually takes 4 to 10 weeks. AI tools are not allowed during live interviews unless you are told otherwise.

## Questions and answers

### Agent security and prompt injection

#### Q1. How would you build an evaluation that measures how well a tool-using Gemini agent resists indirect prompt injection?
Start with a precise threat model. The attacker controls only content the agent reads through tools, such as an email, a web page, or a calendar invite. Their goal is data exfiltration or an action the user never asked for. The user and the system prompt are trusted.

Build each scenario from three parts: a realistic user task, a tool environment with synthetic private data, and an injection slot inside untrusted content. Then attack it adaptively. Useful attack families include optimization-based triggers, tree-search attacks where an attacker model refines prompts over several rounds, and LLM-generated variants seeded from earlier successes. Google DeepMind's published Gemini work argues that static attack sets overstate robustness because defenses overfit to them.

Report attack success rate on held-out scenarios with confidence intervals, next to a utility metric that asks whether the agent still finished the user's task. A defense counts only if the agent keeps using tool output to get work done. Run the suite against every candidate checkpoint so regressions show up before launch.

Close by naming the limits. Simulated environments miss real product context. Success judges make mistakes, so have people spot check them. And the strongest attacker in your suite gives a lower bound on true risk, since a real attacker may do better.

#### Q2. How would you use adversarial training to make the model resist prompt injection while keeping its instruction following intact?
Generate training episodes where tool outputs contain injected instructions, and label the target behavior clearly. The model completes the user's original task, ignores the embedded instruction, and, where useful, tells the user the content looked suspicious. Mix these episodes with clean tool-use data so the model keeps treating ordinary tool output as useful input.

Use both stages. SFT on curated demonstrations sets the default behavior. RL then rewards task completion and penalizes following injected goals, which helps the model generalize past the exact phrasings in the SFT set.

Give the guardrail metrics the same weight as the robustness metric. Track general instruction following, tool-use benchmarks, and a false suspicion rate on benign content that happens to look like instructions, such as a recipe or a how-to page the user asked to summarize. Google DeepMind's report on defending Gemini says careful adversarial training can raise robustness without noticeable benign regressions. Check that result again on each of your own runs.

Then attack the trained model with fresh adaptive attacks generated after training. Scores on the triggers you trained on only measure memorization. Treat the whole process as a continuing loop, because attackers adapt to every new checkpoint.

#### Q3. When should a safety protection live inside the model, and when should it live outside it?
In-model protections come from training. The model learns to recognize risky requests or untrusted instructions. These protections generalize to cases nobody wrote a rule for and add no serving cost. They are also probabilistic, and an adaptive attacker will eventually find an input that works.

Out-of-model protections include input and output classifiers, privilege separation between trusted and untrusted data, user confirmation before sensitive actions, and designs like CaMeL that control data flow so untrusted content cannot change what the agent does. They give harder guarantees for specific failure modes. In exchange they add latency, block some legitimate use, and cover only what they were designed for.

Severity and reversibility decide where a protection goes. For low-severity content issues, in-model training plus a light classifier is usually enough. Irreversible actions like sending money or sharing private data need a system-level control that holds even when the model gets it wrong.

I would argue for defense in depth, which matches Google's published framework for secure agents: well-defined human controllers, limited agent powers, and observable actions, layered on a model that is itself hard to manipulate. Then explain how out-of-model failures become training data that improves the in-model layer.

#### Q4. An agent can take irreversible actions such as sending email or making purchases. How would you train it to act safely?
First, sort actions into risk tiers: read-only actions, reversible writes, and irreversible or high-impact actions such as payments, sending messages, deleting data, or sharing files externally. Each tier gets its own policy. Read-only actions just happen. Irreversible ones need confirmation unless the user clearly and recently authorized that exact action.

For SFT, collect demonstrations where the agent pauses and asks for confirmation with a clear summary of what it is about to do. For RL, build environments where actions have consequences. Reward correct task completion, and penalize unconfirmed high-risk actions and actions outside the user's stated intent.

Measure two failure modes together and plot both across checkpoints. Harmful action rate captures the obvious risk. Unnecessary confirmation rate captures annoyance, since people stop using an agent that asks permission for everything.

Then raise the tradeoffs. Confirmation prompts help only when users read them, so the summaries have to be specific. Authorization can be ambiguous across a long conversation. Training alone gives no guarantee, so pair it with a system rule that blocks the highest-risk actions until the user explicitly approves them. Finally, cover prompt injection as a way to trigger these actions, which links back to Q1.

### Post-training and reward design

#### Q5. How would you design a reward model for agent safety that scores full trajectories?
Single-turn safety reward models judge one response. An agent trajectory is a sequence of tool calls, observations, and final answers, and the harm may sit in a single action in the middle.

I would combine three signals. The first is a set of rubric-based critics, one per policy area, that read the trajectory and flag violations with a cited step. The second is outcome checks on the environment state, such as whether private data left the sandbox or a disallowed action executed. These are cheap and hard to game. The third is a learned preference model trained on pairs of trajectories from red teaming and realistic usage, ranked by human raters or a calibrated judge.

Calibrate each component separately and per harm category, because a single scalar hides which category is drifting. Audit for spurious cues. Reward models often learn that refusal phrasing or hedging means safe, and the policy will then exploit that.

Outcome checks are reliable but cover only what you instrumented. Learned models cover more and can be hacked. Long trajectories also strain context length and judge accuracy. Before using the reward model for RL, I would measure its agreement with held-out human labels on agent trajectories, then keep refreshing it with on-policy data as the policy changes.

#### Q6. How do you handle credit assignment for safety in long-horizon agent RL?
A safety penalty given only at the end of a 50-step episode produces a weak, noisy gradient signal for the one bad step. The policy cannot tell which step caused the penalty, so it may learn the wrong lesson, such as avoiding tools in general.

Here are the options, roughly in order of cost. Turn-level penalties apply directly to risky actions when an environment check or critic flags them. Process reward models score each step and are trained on step-labeled data. Counterfactual or advantage-based estimates with a value function can work, but they are harder to stabilize for long text trajectories. A curriculum starts with short tasks where the dangerous step is obvious and then extends the horizon.

One concrete experiment compares outcome-only reward against outcome reward plus a step-level critic, on a suite of agent tasks with planted risky steps. Measure violation rate, task success, and tool usage rate. Tool usage rate catches the degenerate solution where the agent stays safe by doing nothing.

Step critics bring their own errors and can be gamed, and heavy shaping can pull the policy away from the true objective. Tie your answer to the listing's mention of long-range RL, and explain how you would check that shaped rewards still improve end-to-end safety on held-out tasks.

#### Q7. Reward in your safety RL run keeps rising, but red teamers still break the model easily. What is happening, and what do you do?
The most likely explanation is a narrow or hackable reward model. The policy found what the reward model likes, such as refusal phrasing or safety disclaimers, while staying weak against the attacks red teamers actually use.

Diagnose before you change anything. Pull the highest-reward samples and read them. Compare reward model scores with held-out judge or human labels on red team prompts, because a falling correlation is the signature of reward hacking. Check whether the training prompt distribution covers the attack styles that succeed, especially multi-turn and agentic attacks.

The fix depends on the cause. If the policy is exploiting the reward model, raise the KL coefficient, add an ensemble of reward models, or retrain the reward model on on-policy samples labeled by humans. If coverage is the problem, feed red team failures into the RL prompt pool and keep a separate held-out slice for evaluation. If the problem is over-optimization, stop early based on a held-out safety metric instead of the training reward.

Measure success with held-out, adversarially generated evaluation, and treat training reward as a diagnostic. Make this loop a routine part of the work, since red teaming is how the system finds reward model blind spots.

#### Q8. Explain the role of the KL penalty in RLHF, and how DPO relates to PPO-based RLHF.
RLHF maximizes expected reward minus a KL penalty between the policy and a reference model, usually the SFT checkpoint. The KL term keeps the policy near the distribution where the reward model is accurate, which limits reward hacking. It also preserves capabilities and fluency that the reward model does not measure. With too low a coefficient the policy exploits the reward model, and with too high a coefficient it barely moves.

DPO starts from the same KL-regularized objective. The optimal policy has a closed form in terms of the reward and the reference model, so DPO rearranges the objective to express the reward through the policy itself. The result is a classification-style loss on preference pairs, with no separate reward model and no online sampling.

DPO is simpler, cheaper, and more stable. Because it is offline, it cannot explore or learn from its own new mistakes, and it can overfit to the preference data. PPO and related online methods sample from the current policy. That matters for safety, because new failure modes appear on-policy.

It is worth adding that many teams combine the two, with offline preference optimization early and online RL later. For safety, online methods paired with fresh adversarial prompts tend to matter most.

### Evaluation, red teaming, and helpfulness

#### Q9. How do you measure and reduce over-refusal while improving safety?
Count over-refusal as a safety cost. A model that refuses benign medical, security, or chemistry questions pushes users to worse sources and loses their trust. Gemini's published safety approach lists helpfulness desiderata explicitly, such as refusing only when no compliant answer exists.

Measure first. Build a borderline benign set of legitimate prompts that share surface features with harmful ones, such as asking how a known exploit worked for a security class. Track false refusal rate on this set alongside violation rate on a harmful set, per category. Plot both across checkpoints as a Pareto frontier, so you can see any change that improves one by hurting the other.

To reduce over-refusal, give SFT helpful safe completions for borderline prompts in addition to refusals for harmful ones. In RL, reward partial help, for example general safety context with the operational detail withheld, so the model has more choices than comply or refuse. Audit the reward model for a bias toward refusal language.

For agents, the equivalent problem is unnecessary refusals to use tools and unnecessary confirmations. Define these metrics before training. Explain how you would pick the operating point with product and policy partners, and plan for human review, because judges often misclassify safe partial answers.

#### Q10. Design an automated red teaming pipeline whose failures feed back into post-training.
Start with the components. One or more attacker models generate attacks conditioned on a policy category and an attack style. The target checkpoint responds, including in agent environments with tools. A judge labels each failure with a category and severity. The pipeline then deduplicates failures and clusters them by embedding, so the team sees distinct failure modes instead of a thousand variants of one.

Diversity is the hard part. Reward attackers for novelty as well as success, seed them with human red team findings, and rotate attack styles, so the attacker keeps exploring past the first trick that works.

For the feedback loop, route a portion of the clustered failures into SFT and RL prompt pools with target behaviors. Keep a strictly held-out slice for evaluation, ideally from different attacker seeds or styles, and keep it out of training for good. Gemini's published reports describe using automated red teaming failures to improve both evaluations and training data, and running separate held-out assurance evaluations for release decisions.

Name the risks too. Judge errors create noisy labels, so sample some for human review. Training on attacker outputs can teach the model to recognize the attacker's style instead of the harm. Contamination between training and evaluation makes progress look better than it is. Also track metrics for the pipeline itself, such as the new cluster discovery rate over time.

#### Q11. How would you decide whether checkpoint B is actually safer than checkpoint A?
Use a paired design. Run both checkpoints on the same prompts and scenarios, with several samples per prompt at the production temperature, since a single sample hides variance. Paired comparisons need far fewer samples than a comparison of two independent rates.

For binary failure labels, use a McNemar-style test on discordant pairs, or a paired bootstrap for confidence intervals on the difference. With many harm categories, correct for multiple comparisons or pre-register the few categories that gate launch. Report effect sizes with intervals alongside any p-values.

Judge reliability is often the largest source of error. Measure judge agreement with human labels on a sample from both checkpoints, because a judge can favor one model's style. If B writes longer, more hedged answers, a judge may score it as safer when nothing real changed.

Hold the setup constant across both runs, with the same system prompt, the same tools, and the same decoding settings. Look at the full distribution as well as the mean, since B might cut common low-severity failures while adding a rare severe one.

Finish with a decision rule agreed in advance. For example, B ships only if no gated category regresses beyond a threshold at a stated confidence and overall helpfulness does not drop.

#### Q12. What new safety risks come with deep research agents that browse and plan over many steps, and how would you evaluate them?
Deep research agents read many untrusted pages, issue many searches, and synthesize long reports. That changes the risk profile in four ways. Exposure to prompt injection grows with every page read. Errors compound across steps, so a bad early plan steers the whole report. Private context can leak through the agent's own search queries or the links it visits. And harmful synthesis becomes possible, where each source is benign but the combined report gives meaningful uplift.

Evaluate end to end. Build sandboxed web environments with injections planted at different depths, and measure whether the final report or any action reflects them. Include tasks where the user context holds private data, and check outgoing queries for leakage. For harmful synthesis, use expert-designed tasks in sensitive domains and grade the final report as a whole for uplift.

Mitigations come from both training and system design. Train the model to treat retrieved text as data, to flag suspicious sources, and to decline synthesis in restricted domains. Add system checks on outgoing queries, and keep traces observable for audit.

Realistic environments are expensive to build and go stale, and graders for uplift need domain experts. I would propose a small, high-quality suite first, then scale it with automated attackers.

### Launch decisions and research depth

#### Q13. Late in a launch cycle you find a safety regression in the release candidate. Do you fix it with an SFT patch, an RL change, or a system mitigation?
Start with triage. How severe is the harm, how broad is the trigger, and is it new in this checkpoint or only newly discovered? Reproduce it on a held-out set so you know its real rate beyond the one red team example.

Then weigh the options. A targeted SFT data addition is fast and well understood, but small patches can overfit to the exact phrasing and cause over-refusal in nearby benign areas. An RL change, such as adding prompts to the safety RL mix or adjusting a reward weight, generalizes better. It also needs a full run and full regression testing, which may not fit the timeline. A system mitigation, such as a classifier or a product-level restriction, can ship immediately and is easy to roll back. It adds latency and false positives, and the model itself stays as it was.

A common choice in practice is to ship a system mitigation now if the severity requires it and land the model-level fix in the next post-training iteration. Add the regression case to the evaluation suite permanently.

Also cover communication. Write up the failure, the measured rate, the chosen fix, and the residual risk, so launch decision makers have the data in hand when they choose.

#### Q14. A product team wants to launch on schedule with a known safety gap. How do you handle it when you have no direct authority?
Begin by making the risk concrete. Quantify how often the failure happens on realistic traffic, how severe it is, and who is affected. A measured rate with examples holds up against a deadline far better than a general concern.

Then bring options along with the concern, and estimate the cost of each. You could propose a targeted mitigation that can ship in time, a scoped launch that leaves out the risky surface or user group, a staged rollout with monitoring and a rollback trigger, or a short delay with a clear plan.

Work through the existing launch process. Most organizations have agreed safety criteria and escalation paths, so tie your case to those criteria. If the gap violates a hard criterion, escalate through the defined channel early, well before the night before launch.

Keep the relationship collaborative throughout. Product teams share the goal of a safe launch, and framing your work as protecting the launch tends to land well. Document the final decision, the data behind it, and the follow-ups.

Use a real example from your past where evidence and options changed a decision, and say what you would do differently now. The listing explicitly asks for influencing safety decisions without direct authority, so expect this theme.

#### Q15. Walk me through your strongest paper. What would you do differently now, and how does it connect to agent safety?
Plan about five minutes for the walkthrough and expect follow-ups for the rest of the slot. State the research question and why it mattered. Give the key result in one sentence with the main number. Explain the one or two design choices that made the work succeed and what you gave up for them.

Then critique the paper yourself before the interviewer does. Name the weakest assumption, the experiment a skeptical reviewer would ask for, and whether the result has held up since publication. Interviewers use this to test whether you can separate a solid finding from a convenient one.

Next, say what you would do now. Explain what changes with better models, more compute, or a production setting. Propose the next experiment concretely, with a hypothesis, setup, metric, and the result that would change your mind.

Finally, connect it to this team. If your work is in RL, reward modeling, evaluation, robustness, or agents, draw the line to safety post-training directly. If it is in another area, pick the transferable skill, such as careful evaluation design or large-scale training, and give one idea for applying it to agent safety.

Candidate reports suggest only one or two papers carry real weight in these interviews, so know those deeply and put your prep time there.

## Prep checklist

- [ ] Read "Lessons from Defending Gemini Against Indirect Prompt Injections" (arXiv 2505.14534) and be ready to explain adaptive attacks and the adversarial training result.
- [ ] Read the CaMeL paper (arXiv 2503.18813) and Google's secure AI agents framework, and form a view on in-model versus system-level defenses.
- [ ] Review the safety section of the Gemini 2.5 technical report, covering policies, helpfulness desiderata, SFT and RL from human and critic feedback, and automated red teaming.
- [ ] Refresh RLHF fundamentals, including the KL-regularized objective, PPO versus DPO, reward hacking, and reward model calibration.
- [ ] Prepare one end-to-end experiment design for agent safety that covers the threat model, data, training method, metrics, and guardrail metrics.
- [ ] Practice paired evaluation statistics, including bootstrap intervals, the McNemar test, multiple comparisons, and judge agreement checks.
- [ ] Do timed coding practice at medium to hard difficulty, writing code that runs.
- [ ] Prepare a 20 minute talk on your one or two strongest papers, including their weaknesses and your next experiments.
- [ ] Prepare two stories about influencing a decision without authority under deadline pressure.
- [ ] Ask the recruiter which round types to expect, since DeepMind sends role-specific prep information.

## Sources

- https://deepmind.google/careers/ : Google DeepMind's published interview stages.
- https://www.datainterview.com/blog/google-deepmind-research-engineer-interview : third-party guide that cites the 4 to 10 week timeline and the rule against AI tools in live interviews. Its candidate reports focus on RE roles.
- https://silviasapora.github.io/blog/ml-interviews.html : candidate blog (DeepMind RS offer) describing coding, ML, and research interview types. Anecdotal.
- https://yongzx.github.io/blog/2026/06/24/job-search : lessons from a research scientist job search, written for AI safety candidates. Anecdotal.
- https://www.lesswrong.com/posts/dvsFfGuXXyHYkyifp/tips-for-cracking-the-ai-safety-technical-interview-1 : common round types in AI safety interviews.
- https://www.reddit.com/r/MachineLearning/comments/1q2wiub/d_google_deepmind_research_engineerscientist : community discussion of DeepMind RS and RE interviews. Anecdotal.
- https://arxiv.org/abs/2505.14534 : Google DeepMind on defending Gemini against indirect prompt injections.
- https://arxiv.org/abs/2503.18813 : CaMeL, a system-level prompt injection defense.
- https://arxiv.org/html/2507.06261v1 : Gemini 2.5 technical report, safety and red teaming sections.
- https://research.google/pubs/an-introduction-to-googles-approach-for-secure-ai-agents : Google's framework for secure AI agents.
