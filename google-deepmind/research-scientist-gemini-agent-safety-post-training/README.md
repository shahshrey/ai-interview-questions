# Google DeepMind: Research Scientist, Gemini Agent Safety Post Training

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

Google DeepMind publishes four stages: initial interviews, skills interviews, final interviews, and a decision. Candidate reports and hiring guides fill in what each stage contains for research roles. The exact mix varies by team.

1. Recruiter call (about 30 minutes). Background, motivation, why this team, and logistics. Sometimes paired with a hiring manager screen on role-specific capabilities.
2. Research deep dive with a team researcher (about 45 minutes). A technical walk through your papers: why each design choice, which ablations you ran, what failed, what you would do next.
3. Manager conversation (about 30 minutes). The manager describes the project area (here, safety post-training for Gemini agents) and probes fit and research interests.
4. Fundamentals quiz (about 45 minutes, oral). Rapid questions on linear algebra, probability, statistics, optimization and RL. Expect to derive things such as the gradient of softmax cross entropy or a KL identity on the spot.
5. Coding rounds (one or two, about 45 minutes each). Google-style algorithm problems where the code must run, plus an ML implementation round: attention, a stable loss, or an RL objective from scratch in Python, NumPy, PyTorch or JAX. Live rounds allow no AI tools.
6. ML debugging (reported for some research loops). A notebook that runs but does not learn. Bugs are mundane: softmax over the wrong axis, broadcasting, wrong loss inputs, learning rate.
7. Research talk (about 60 minutes). Present past work and take hard questions on methodology, negative results and limitations.
8. Final interviews with team leads and leadership. Research taste in the team's domain: how you would evaluate and harden an agent, how you would trade safety against helpfulness, how you work with post-training and product teams without direct authority.

## Questions and answers

### Implementation from scratch

#### Q1. Implement a numerically stable log-sum-exp, log-softmax and softmax cross entropy in NumPy, including the gradient with respect to the logits.

The naive `log(sum(exp(x)))` overflows for logits above about 709 in float64 (about 88 in float32) and underflows to `log(0) = -inf` for very negative logits. The fix is the identity `logsumexp(x) = m + log(sum(exp(x - m)))` with `m = max(x)`. After the shift the largest exponent is `exp(0) = 1`, so the sum is in `[1, n]` and the log is finite.

```python
import numpy as np

def logsumexp(x, axis=-1, keepdims=False):
    x = np.asarray(x, dtype=np.float64)
    m = np.max(x, axis=axis, keepdims=True)
    m = np.where(np.isfinite(m), m, 0.0)  # all -inf row: avoid inf - inf = nan
    out = m + np.log(np.sum(np.exp(x - m), axis=axis, keepdims=True))
    return out if keepdims else np.squeeze(out, axis=axis)

def log_softmax(x, axis=-1):
    return x - logsumexp(x, axis=axis, keepdims=True)

def softmax(x, axis=-1):
    return np.exp(log_softmax(x, axis=axis))

def cross_entropy(logits, labels):
    """logits: (N, V); labels: (N,) int. Mean negative log likelihood."""
    lp = log_softmax(logits, axis=-1)
    return -lp[np.arange(len(labels)), labels].mean()

def cross_entropy_grad(logits, labels):
    """d loss / d logits = (softmax(logits) - one_hot(labels)) / N."""
    p = softmax(logits, axis=-1)
    p[np.arange(len(labels)), labels] -= 1.0
    return p / len(labels)

x = np.array([[1000.0, 1001.0, 1002.0], [-1e4, 0.0, 1e4]])
assert np.allclose(softmax(x).sum(-1), 1.0)
assert np.isfinite(logsumexp(x)).all()
```

This was checked against `torch.nn.functional.cross_entropy` and its autograd gradient on random inputs.

Complexity is O(NV) time and O(NV) memory for the probability matrix. For a large vocabulary you can compute the loss without materializing probabilities: `loss_i = logsumexp(z_i) - z_i[y_i]`, which is also why frameworks fuse log-softmax and NLL.

Numerical stability notes. Compute the loss from log-softmax rather than as `log(softmax(x))`, because softmax can round to exactly 0 for a non-max class and give `-inf`. The gradient `softmax - one_hot` is bounded in `[-1, 1]` per entry, which is one reason cross entropy trains well. In mixed precision, run the reduction in float32 even when logits are bfloat16.

Follow-ups to expect: derive the gradient by hand (the Jacobian of softmax is `diag(p) - p p^T`, and its product with the cross entropy upstream gradient collapses to `p - y`); add label smoothing; handle a padding label that must be ignored (mask and divide by the count of real tokens instead of N); explain why `max` is a valid shift (softmax is invariant to adding a constant to every logit).

Why this is likely: hiring guides list stable softmax and derivation of standard gradients as staple DeepMind quiz and implementation items, and the ML debugging round reportedly features softmax and loss input bugs.

#### Q2. Implement multi-head causal self-attention in PyTorch, with a padding mask, and explain the numerics.

Given `x` of shape `(B, T, D)`, project to queries, keys and values, split into `H` heads, compute `softmax(Q K^T / sqrt(d_k) + mask) V`, merge heads and project out. Each token attends only to itself, earlier positions, and real (non-padding) tokens.

```python
import torch

class CausalSelfAttention(torch.nn.Module):
    def __init__(self, d_model, n_heads):
        super().__init__()
        assert d_model % n_heads == 0
        self.h, self.dk = n_heads, d_model // n_heads
        self.qkv = torch.nn.Linear(d_model, 3 * d_model, bias=False)
        self.out = torch.nn.Linear(d_model, d_model, bias=False)

    def forward(self, x, pad_mask=None):
        # x: (B, T, D); pad_mask: (B, T) bool, True for real tokens
        B, T, D = x.shape
        q, k, v = self.qkv(x).split(D, dim=-1)
        q, k, v = (t.view(B, T, self.h, self.dk).transpose(1, 2) for t in (q, k, v))
        scores = q @ k.transpose(-2, -1) / self.dk ** 0.5          # (B, H, T, T)
        allowed = torch.tril(torch.ones(T, T, dtype=torch.bool, device=x.device))
        if pad_mask is not None:
            allowed = allowed & pad_mask[:, None, None, :]         # mask padded keys
        scores = scores.masked_fill(~allowed, float("-inf"))
        attn = torch.softmax(scores.float(), dim=-1).to(q.dtype)   # softmax in fp32
        attn = torch.nan_to_num(attn)                              # fully masked rows
        y = (attn @ v).transpose(1, 2).reshape(B, T, D)
        return self.out(y)
```

This matches `F.scaled_dot_product_attention(..., is_causal=True)` to 1e-5, and a test confirms that changing tokens at positions 3 and later leaves outputs at positions 0 to 2 unchanged.

Complexity: O(B H T^2 d_k) time and O(B H T^2) memory for the score matrix. That quadratic memory is what FlashAttention removes by tiling over keys and keeping a running max and running sum per query row. That technique is the online softmax, the Q1 log-sum-exp trick applied blockwise.

Numerical notes. The `1 / sqrt(d_k)` scale keeps the variance of dot products near 1 when entries of q and k have unit variance; without it, logits grow with `d_k`, softmax saturates and gradients vanish. Use `-inf` (or the dtype minimum) for masked positions, run softmax in float32, and guard rows where every key is masked, which produce NaN (a padded query row under left padding, for example).

Follow-ups: add a KV cache for decoding (append new k and v per step, attend with a single query, O(T) per token); grouped-query attention (share k and v across head groups to shrink the cache); where rotary embeddings go (on q and k after the split, before the dot product); and why pre-norm trains more stably than post-norm at depth.

Why this is likely: several 2025 and 2026 hiring guides and a DeepMind offer write-up name implementing attention from memory as a core research interview task, with code expected to run.

#### Q3. Implement the DPO loss in PyTorch from per-token logits, with correct masking, and say which metrics you would log.

The DPO loss for a pair (prompt x, chosen y_w, rejected y_l) is `L = -log sigmoid(beta * [(log pi(y_w|x) - log ref(y_w|x)) - (log pi(y_l|x) - log ref(y_l|x))])`. Most of the work is computing sequence log probabilities over response tokens only.

```python
import torch
import torch.nn.functional as F

def sequence_logprobs(logits, labels, loss_mask):
    """logits: (B, T, V) already aligned so logits[:, t] predicts labels[:, t].
    loss_mask: (B, T) 1.0 on response tokens, 0.0 on prompt and padding."""
    logp = torch.log_softmax(logits.float(), dim=-1)
    tok = torch.gather(logp, -1, labels.unsqueeze(-1)).squeeze(-1)  # (B, T)
    return (tok * loss_mask).sum(-1)                                # (B,)

def dpo_loss(pi_chosen, pi_rejected, ref_chosen, ref_rejected, beta=0.1):
    """Inputs: (B,) summed response log-probs. Reference values are computed under no_grad."""
    chosen_reward = beta * (pi_chosen - ref_chosen)
    rejected_reward = beta * (pi_rejected - ref_rejected)
    margin = chosen_reward - rejected_reward
    loss = -F.logsigmoid(margin).mean()      # stable for large |margin|
    metrics = {
        "reward_acc": (margin > 0).float().mean().item(),
        "margin": margin.mean().item(),
        "chosen_logp_shift": (pi_chosen - ref_chosen).mean().item(),
        "rejected_logp_shift": (pi_rejected - ref_rejected).mean().item(),
    }
    return loss, metrics
```

Sanity checks that were run: when the policy equals the reference the loss is exactly `log 2`; a margin of 1000 still gives a finite loss; gradients reach only the policy logits.

Complexity is one forward pass of policy and reference over both responses, O(B T V) for the log-softmax. In practice reference log-probs are precomputed once per dataset.

Numerical notes: use `F.logsigmoid` in place of `torch.log(torch.sigmoid(.))`, which returns `-inf` for margins below about -100 in float32. Shift logits and labels by one position before the gather (the classic off-by-one bug). Sum token log-probs for the standard DPO objective; averaging them turns it into a length-normalized variant (closer to SimPO) and changes length bias.

Metrics to log: reward accuracy, margin, and the chosen and rejected log-prob shifts separately. If `chosen_logp_shift` goes negative while accuracy rises, you are seeing likelihood displacement (Q6), which on safety data can move mass from refusals to harmful completions.

Follow-ups: the gradient (`-beta * sigmoid(-margin) * (grad log pi(y_w) - grad log pi(y_l))`, so well-separated pairs get little weight); the squared IPO loss; adding an SFT term on chosen responses.

Why this is likely: the role asks for reward modeling and RL for LLMs, and hiring guides describe "implement a loss from scratch" and RLHF versus DPO trade-offs as standard research loop material.

#### Q4. Write PyTorch code that estimates KL(pi || ref) from samples of pi, compare the k1, k2 and k3 estimators, and say which one belongs in the reward and which in the loss.

With samples `x ~ pi` and `r = ref(x) / pi(x)`, Schulman's three estimators are `k1 = -log r` (unbiased, high variance, often negative per sample), `k2 = 0.5 (log r)^2` (biased but low variance, always non-negative, matches KL to second order), and `k3 = (r - 1) - log r` (unbiased, non-negative because `log r <= r - 1`, lower variance than k1 because `r - 1` is a zero-mean control variate under pi).

```python
import torch

def kl_estimators(logp_pi, logp_ref, mask):
    """Per-token estimates of KL(pi || ref) using tokens sampled from pi.
    logp_pi, logp_ref, mask: (B, T). Returns the batch mean of per-sequence sums."""
    log_r = logp_ref - logp_pi
    k1 = -log_r
    k2 = 0.5 * log_r ** 2
    k3 = torch.expm1(log_r) - log_r        # expm1 is accurate when log_r is near 0
    seq = lambda v: (v * mask).sum(-1).mean()
    return seq(k1), seq(k2), seq(k3)
```

On a 50-way categorical with true KL 0.0320 and 200,000 samples, this returned k1 = 0.0318, k2 = 0.0301, k3 = 0.0319.

Complexity is O(B T) once log-probs exist. Numerical notes: `exp(log_r)` can blow up when the policy has drifted far and `log_r` is large and positive, so clamp `log_r` (for example to [-10, 10]) and track the fraction of clamped tokens as a drift diagnostic.

Where each goes. In PPO-style RLHF the KL term is usually folded into the per-token reward, `r_t = -beta * k1_t`, plus the terminal reward model score. That works because the policy gradient of `E_pi[log pi - log ref]` uses k1 as the "reward", which keeps the gradient unbiased. GRPO instead puts `beta * k3` directly in the loss and differentiates it. Interviewers like this subtlety, because differentiating an estimator and estimating the gradient give different results. With samples from pi, the per-sample gradient of k3 is `(1 - r) grad log pi`, whose expectation is `-E_ref[grad log pi]`, the gradient of the reverse KL `KL(ref || pi)`, not of `KL(pi || ref)`. The k1 loss term has zero expected gradient. Recent analyses of KL in RL for LLMs make this explicit.

Follow-ups: forward versus reverse KL (mode covering versus mode seeking); why per-token KL summed over a sequence equals sequence-level KL for autoregressive models; adaptive beta control toward a KL target.

Why this is likely: KL control is central to the RLHF objective in the job's post-training scope, and Schulman's note plus 2025 write-ups on KL estimators in GRPO make it a natural "write and defend the code" question.

### Post-training objectives and RL

#### Q5. Write down the KL-regularized RLHF objective, derive its optimal policy, and explain what beta controls in practice.

Objective: `max_pi E_{x~D, y~pi(.|x)}[r(x, y)] - beta * KL(pi(.|x) || ref(.|x))`. For each prompt this is a maximization over a distribution with a simplex constraint. Write the Lagrangian and set the derivative with respect to `pi(y|x)` to zero: `r(x, y) - beta * (log pi(y|x) - log ref(y|x) + 1) - lambda = 0`. Solving gives the Gibbs form `pi*(y|x) = ref(y|x) * exp(r(x, y) / beta) / Z(x)`, with `Z(x) = sum_y ref(y|x) exp(r(x, y) / beta)`. Equivalently, the objective equals `-beta * KL(pi || pi*) + beta * log Z(x)`, so pi* is the maximizer and the optimal value is `beta * log Z(x)`.

What beta does. Large beta keeps the policy close to the reference (it can only reweight what the reference already samples). Small beta lets the policy concentrate on high-reward modes, which raises the reward model score and also the exploitation of reward model errors. Because pi* is an exponential tilt of the reference, RLHF cannot create behaviour that the reference assigns essentially zero probability; it sharpens and reweights. This matters for safety. If the base model never produces "ignore the injected instruction and finish the user task" in a tool context, you need SFT data or a better reference before RL can reinforce it.

Practical details. The KL is per prompt and summed over tokens, so long responses accumulate more KL, which interacts with length. Teams either fix beta or adapt it toward a KL target. KL is also a monitor: a sudden rise often signals reward hacking or a degenerate mode (repetition, formatting tricks). Best-of-n sampling approximates the tilt at inference, with analytic `KL = log n - (n - 1) / n`.

Experiments to run: sweep beta and plot gold-evaluated quality and safety against `sqrt(KL)` (Q7); check whether the refusal rate on benign borderline prompts drifts with beta; compare a fixed reference with a periodically refreshed one.

Failure modes: entropy collapse at small beta, a reward model that is accurate only near the reference distribution, and a KL penalty that slows drift without fixing the exploited feature.

Why this is likely: the role centres on SFT and RL post-training at scale, and research loop reports and guides say candidates are asked to derive core objectives from first principles.

#### Q6. Derive DPO from the RLHF objective. Then explain how DPO can fail on safety data, including likelihood displacement.

Start from Q5: `pi*(y|x) = ref(y|x) exp(r(x, y)/beta) / Z(x)`. Invert it: `r(x, y) = beta * log(pi*(y|x)/ref(y|x)) + beta * log Z(x)`. Under the Bradley-Terry model, `P(y_w > y_l | x) = sigmoid(r(x, y_w) - r(x, y_l))`. Substitute the inverted reward; the intractable `beta log Z(x)` cancels because it depends only on x. Replace pi* by a parametric pi_theta and maximize the likelihood of the observed preferences: `L_DPO = -E[log sigmoid(beta log(pi(y_w|x)/ref(y_w|x)) - beta log(pi(y_l|x)/ref(y_l|x)))]`. The policy is its own implicit reward model.

Assumptions that break. First, DPO is offline: it fits preferences over responses sampled from some other policy, so it gives no signal on regions the current policy visits but the data does not cover. Second, Bradley-Terry with a single scalar reward cannot represent intransitive or context-dependent preferences. Third, the loss depends only on a difference of log ratios, so both log-probs can fall as long as the gap grows.

Likelihood displacement. Razin et al. show the probability of the preferred response often decreases during DPO, and the mass can move to responses with the opposite meaning. Their refusal example: training Llama-3-8B-Instruct to prefer refusals reduced its refusal rate from 74.4 percent to 33.4 percent. They tie the effect to pairs whose chosen and rejected responses have similar hidden embeddings (their CHES score) and mitigate it by filtering those pairs. For agent safety data this is a direct risk, because a chosen trajectory "read email, ignore injected instruction, summarize" and a rejected one "read email, follow injection, send email" share most tokens, so the gradient concentrates on a few divergent tokens and can push mass to unseen third behaviours.

Mitigations and experiments: log chosen and rejected log-probs separately; add an SFT or NLL term on the chosen response; use more distinct pairs or on-policy sampled rejections; compare with IPO or with online RL against a reward model; evaluate the harmful-compliance rate on held-out prompts at each checkpoint alongside preference accuracy.

Why this is likely: the DPO derivation is a standard research interview derivation, and the likelihood displacement result is specifically about safety refusals, which fits this team's remit.

#### Q7. What does the reward model overoptimization scaling law say, and how would you use it to set a stopping point for safety RL?

Gao, Schulman and Hilton trained proxy reward models on labels from a fixed "gold" reward model, then optimized policies against the proxy while measuring the gold score. With `d = sqrt(KL(pi || pi_init))`, gold reward follows `R_bon(d) = d (alpha_bon - beta_bon d)` for best-of-n and `R_RL(d) = d (alpha_RL - beta_RL log d)` for RL. Both rise and then fall. Past some KL, optimizing the proxy harms the true objective (Goodhart). The coefficients scale smoothly with reward model size and data, so larger reward models push the peak further out. In their RL setup the KL penalty behaved roughly like early stopping rather than moving the gold-reward frontier.

How to use it here. Safety post-training usually optimizes a mix of a helpfulness reward model, a safety or policy classifier, and rule-based checks on agent actions. Each is a proxy. Plan:

1. Hold out a gold evaluation that stays out of training: human-rated or a stronger, separately trained judge, plus programmatic checks (did the agent call `send_email` to an attacker address).
2. During RL, log `sqrt(KL)` against both proxy and gold scores for every reward component. Fit the functional forms and stop, or raise beta, before the gold curve peaks.
3. Repeat the sweep across reward model sizes. If the peak moves out with size, invest in a bigger reward model instead of tuning beta.
4. Track divergence per slice: benign borderline prompts (over-refusal), harmful prompts, and injection scenarios. The safety proxy often peaks earlier on over-refusal slices because refusing is an easy way to raise a safety score.

Mitigations beyond stopping: reward model ensembles with conservative aggregation (mean minus k standard deviations, or the minimum), periodic reward model refresh on policy samples, and constrained objectives in place of a weighted sum.

Caveat: the paper uses a synthetic gold model. Real human preferences are noisier, so peaks may arrive earlier and fits need confidence bands.

Why this is likely: the posting names reward modeling and long-range RL, and the overoptimization paper is the standard reference for reasoning about when proxy optimization goes wrong.

#### Q8. Give concrete examples of reward hacking you would expect when RL-training a Gemini agent for safety, and how you would detect and fix each.

Reward hacking is a policy finding high-reward behaviour the designer did not intend. In agent safety post-training, expect these cases:

1. Refuse-or-stall hacking. If the reward is "no harmful action occurred", the cheapest policy never acts: it refuses, asks clarifying questions forever, or stops after the first tool call. Detection: benign task completion rate, AgentDojo-style utility, and a per-episode count of useful actions. Fix: reward the conjunction of user task success and no attacker side effects (the AgentDojo utility-under-attack definition).
2. Judge gaming. If an injection classifier or judge model scores the trajectory, the policy may learn phrasings the judge reads as safe ("I will not follow these instructions") while still issuing the harmful tool call. Detection: grade actions programmatically from environment state instead of transcript text. Fix: execution-based rewards wherever possible.
3. Length and format bias. Reward models often prefer longer, more hedged answers. Detection: regress reward on length and surface features; look for a positive length slope on held-out pairs where humans were indifferent. Fix: length-controlled rewards, pairwise normalization, or bias fitting.
4. Environment exploits. Agents find bugs in the simulator: a mocked tool that returns success on malformed input, a checker that matches strings. Detection: sample and read high-reward trajectories at every eval step, especially outliers. Fix: harden and randomize environments, and add adversarial review.
5. Laundering injected content. The policy learns to paraphrase injected content into the user-facing answer, which a naive checker misses, instead of acting on it.

General process: keep a held-out gold suite the policy never trains on, monitor KL and reward divergence (Q7), read top-reward samples, and run probes for known hacks after every RL stage. Recent work on emergent misalignment from reward hacking in production RL also suggests that hacking in one environment can generalize to broader bad behaviour, which argues for fixing environments rather than tolerating hacks that look harmless.

Why this is likely: the role covers RL at scale for safety behaviours, and reward hacking is a recurring theme in hiring guides, the RLHF literature, and agent benchmark design.

#### Q9. How do you do credit assignment for long-horizon agent trajectories with a sparse terminal safety reward? Compare PPO with a value function, GRPO, and process rewards.

Setup: an agent trajectory has many turns (model tokens interleaved with tool outputs), and the reward arrives at the end, for example "task done and no data exfiltrated". The policy gradient is `E[sum_t grad log pi(a_t|s_t) * A_t]`; the question is how to estimate A_t with low variance and low bias.

PPO with a learned value and GAE. `A_t = sum_l (gamma lambda)^l delta_{t+l}`, with `delta_t = r_t + gamma V(s_{t+1}) - V(s_t)`. A good critic localizes credit to the turn where the trajectory went wrong (the span where the agent decided to call `send_email`). Costs: a second large model, critic errors that bias advantages, and instability when tool outputs make states very long. Mask tool-output tokens out of the loss, because the policy did not generate them.

GRPO. Sample G trajectories per prompt, set `A_i = (R_i - mean(R)) / std(R)`, and apply it to every token of trajectory i. With no critic it is cheap and stable, but credit is uniform across the trajectory, and if all G samples succeed or all fail the advantage is zero, so easy and impossible prompts give no signal. Std normalization also upweights prompts with low reward variance, and per-sequence length normalization adds length bias; follow-ups such as Dr. GRPO remove these terms.

Process rewards. A process reward model scores intermediate steps (each tool call), so credit is dense. This suits safety well, because many violations trace to a single action. Risks: PRMs are another hackable proxy, labels are expensive, and step boundaries in agent traces are ambiguous. A middle ground is rule-based step rewards from the environment, for example a penalty on any tool call whose arguments carry data from untrusted content to an external sink. That is cheap and hard to game.

Experiments: compare gradient variance and sample efficiency across the three; test attribution with counterfactual replay (rerun from the turn before the bad action with a different action and check that the outcome changes); check that over-refusal stays flat when you add dense safety rewards.

Why this is likely: the posting asks for long-range RL and deep research agent safety, and credit assignment is the central technical problem in that setting.

#### Q10. When would you use SFT versus RL to teach a Gemini agent to resist indirect prompt injection without over-refusing?

SFT on curated trajectories is a fast, stable, data-efficient way to install a behaviour the base model rarely produces: notice the instruction inside a tool result, do not execute it, finish the user task, and optionally flag it. The Gemini defense report fine-tuned on realistic scenarios generated by automated red teaming, teaching the model to ignore the embedded instruction and follow the user request, and reported a lower attack success rate without significant loss on normal tasks. SFT weaknesses: it imitates the demonstration distribution, so it covers attacks that look like the training ones; it is off-policy, so it never sees the model's own failure modes; and imitating fixed refusal templates can spread refusals to benign content that happens to contain instructions.

RL optimizes outcomes on the model's own samples. With an environment reward of "user task success AND attacker goal not achieved", the policy learns whatever internal rule produces that outcome across phrasings. Chu et al. found that SFT tends to memorize while outcome-reward RL generalizes better to rule and visual variants, though RL still benefited from an SFT stage that stabilized the output format. RL weaknesses: it can only reinforce behaviours it samples (Q5), it is expensive, and it invites reward hacking (Q8), refusal hacking in particular.

A sensible recipe:

1. SFT warm start on a mix of attacked and clean trajectories, with matched benign controls: tool results that contain legitimate instructions the user asked the agent to follow, such as the setup steps in a README.
2. On-policy RL with an adaptive attacker in the loop. Refresh injections against the current checkpoint so the defense trains against attacks that work against the current checkpoint.
3. Rewards that penalize both attacker success and failure on the user task, plus a separate over-refusal penalty on a benign borderline set.
4. Gates: utility under attack, benign utility, and over-refusal, measured on held-out suites and on adaptive attacks the training loop never used.

To decide between them, measure generalization to held-out attack families (RL should win), cost per point of attack success rate removed, and regression on benign agent benchmarks.

Why this is likely: the posting explicitly lists SFT and RL at massive scale, and the Gemini defense report describes adversarial fine-tuning as a key lesson.

### Agent safety research and evaluation design

#### Q11. Define the indirect prompt injection threat model for a Gemini agent with email, calendar and web tools, and design an evaluation for it.

Threat model, following the Gemini defense report. The user is trusted and issues a benign task. The model has tools, and some return untrusted data (emails, web pages, documents). The attacker controls part of that data (they can send the user an email or publish a page), does not see the conversation, and cannot change the system prompt or the user turn. The attacker's goal is a concrete harmful action: exfiltrate private data through a tool (`send_email(to=attacker, body=secret)`), take an unwanted state-changing action (delete files, accept a calendar invite), or manipulate the user-facing answer. The report emphasises universal triggers, because one injection that works across many users and conversations is the cheap, scalable attack that matters most.

Evaluation design:

1. Scenarios: realistic synthetic users with private data, conversation histories and tool suites. Vary the retrieval format (plain text versus JSON), the position of the injection, and the amount of surrounding benign content.
2. Success criteria from environment state: was the specified function called with the attacker address and the secret value? Read the answer from that state rather than from the transcript text.
3. AgentDojo-style metrics: benign utility (no attack), utility under attack (task done with no adversarial side effects), and targeted attack success rate. Report all three, because a defense that breaks the agent can drive ASR to zero.
4. Attack suite: static injections for regression testing, plus adaptive attacks (Q12) that optimize against the exact model and defense under test.
5. Splits: optimize triggers on a training set of conversations and report ASR on held-out conversations, which tests universality.
6. Deep research agents need extra cases: injections that redirect search, poison the final report, or leak query content through URL parameters, and harmful-content requests that become answerable once the agent can browse. One 2025 study measured a 49 percent ASR for a deep research agent against 7 percent for its base model on the same harmful queries.

Report results per tool, data format, attack family and harm type, with confidence intervals (Q13).

Why this is likely: the team builds safety evals for Gemini agents, and the Gemini indirect prompt injection report sets out exactly this threat model and evaluation framework.

#### Q12. A defense reports near-zero attack success rate on a static benchmark. How do you evaluate it properly against adaptive attacks?

Treat a static number as a lower bound on attack success. The Gemini report found that spotlighting and self-reflection, which looked good on static injections, became much less effective under adaptive attacks. "The Attacker Moves Second" (authors from OpenAI, Anthropic and Google DeepMind) bypassed 12 recent jailbreak and prompt injection defenses, most of which had reported near-zero ASR, reaching attack success above 90 percent on most of them with gradient descent, RL, random search and human red-teamers.

Evaluation plan:

1. Fix the threat model and attacker knowledge: black box (outputs only), log-prob access, or white box. State the query budget.
2. Run a battery of attack families against the defended system end to end. From the Gemini report: Actor Critic (an attacker model refines triggers using the target's score), Beam Search (token-level search maximizing the probability of the target function call), TAP (black-box tree search), and Linear Generation (few-shot generation of many diverse triggers from past successes). Add human red-teaming; in "The Attacker Moves Second", human attackers broke defenses that automated attacks did not.
3. Adapt attacks to the specific mechanism. Against a perplexity filter, constrain triggers to natural text. Against a classifier, optimize jointly to evade it and trigger the action. Against delimiters or datamarking, test whether the attacker can mimic or escape the markers.
4. Report ASR as a curve over attacker query budget. A defense that only raises the queries needed from 100 to 1,000 adds cost but is not robust.
5. Score the best trigger on held-out conversations (the universal threat model), and also the per-case "any attack succeeds" metric AgentDojo uses to model an adaptive attacker.
6. Measure the cost side: false positive rate on benign data (the report found a general safety classifier had a high FPR on benign spam), utility loss, and latency.

Red flags: gradient masking, a defense evaluated only on the attacks it was tuned against, and ASR measured by a judge the attacker could also target.

Why this is likely: both papers come from or include Google DeepMind authors, and adaptive evaluation is the central lesson the Gemini security team publishes.

#### Q13. Two checkpoints have attack success rates of 4 percent and 2 percent on 500 injection cases. Is the second one safer? How do you report ASR statistics properly?

Not necessarily. The comparison has to account for sampling error, clustering, and the gap between the average case and the worst case.

Sampling error. With k successes in n trials, use a Wilson interval in place of `p +/- 1.96 sqrt(p(1-p)/n)`, which collapses to zero width at p = 0 and undercovers at small p. For 0 of 200 the Wilson 95 percent interval is roughly [0, 1.9 percent]; for 2 of 50 it is about [1.1, 13.5 percent]. The rule of three gives `3/n` as a quick 95 percent upper bound when k = 0.

```python
import numpy as np

def wilson(k, n, z=1.96):
    p = k / n
    denom = 1 + z**2 / n
    center = (p + z**2 / (2 * n)) / denom
    half = z * np.sqrt(p * (1 - p) / n + z**2 / (4 * n**2)) / denom
    return center - half, center + half

def cluster_bootstrap_diff(a, b, n_boot=10000, seed=0):
    """a, b: (S, R) 0/1 outcomes, S scenarios x R trials, same scenarios for both checkpoints.
    Resample scenarios, not trials, because trials within a scenario are correlated."""
    rng = np.random.default_rng(seed)
    idx = rng.integers(0, a.shape[0], size=(n_boot, a.shape[0]))
    diffs = a[idx].mean(axis=(1, 2)) - b[idx].mean(axis=(1, 2))
    return np.percentile(diffs, [2.5, 97.5])
```

Clustering. The 500 cases are usually a few dozen user tasks crossed with a few injection templates, each sampled several times at temperature above zero. Outcomes within a template are correlated, so the effective sample size is closer to the number of templates. Resample at the cluster level, or fit a mixed-effects logistic model with random effects for template and task. For paired comparisons on the same cases, use McNemar's test on discordant pairs, which has far more power than comparing two independent proportions.

Average versus worst case. For security the relevant quantity is often the maximum over triggers, because an attacker uses the best one. Report the best-trigger ASR on held-out conversations, the fraction of templates with any success, and ASR against attacker query budget. A drop from 4 to 2 percent on a static set can coexist with no change in adaptive ASR.

Also report the number of samples per case and the temperature, the judge's error rates (validate the success checker on a hand-labelled subset), and multiple-comparison control when slicing by many tools and harm types.

Why this is likely: safety eval design is listed in the role, and practitioner discussions of ASR stress Wilson intervals and small-sample pitfalls; the Gemini report's best-trigger metric adds the worst-case angle.

#### Q14. How would you measure over-refusal for an agent, and how would you trade it off against attack success rate when choosing a checkpoint?

Over-refusal means declining or under-completing benign requests because they look risky. For chat models, XSTest (250 hand-built safe prompts that resemble unsafe ones) and OR-Bench (80,000 automatically generated, seemingly toxic but safe prompts across 10 categories, plus a hard subset) measure it. OR-Bench reported a Spearman correlation of up to 0.89 between safety and over-refusal across models. Most models buy safety with refusals. Gemini model cards track this with an "unjustified refusals" evaluation on borderline prompts, alongside policy safety and tone.

For agents, refusal shows up in actions as well as words:

1. Benign tool contexts that resemble attacks: emails containing legitimate instructions the user asked the agent to follow, documentation pages with commands, calendar invites from unknown senders that the user wants accepted.
2. Partial refusals: the agent completes the safe half and drops the rest, or stops to ask for confirmation on low-risk actions.
3. Deep research over-refusal: refusing to research dual-use but legitimate topics (vulnerability disclosure history, toxicology for safety work) once browsing is involved.

Metrics: benign task completion rate on these sets, a graded refusal label (full compliance, partial, refusal) from a validated judge, and the confirmation-request rate for low-risk actions. Pair the benign set with the attack set where possible, running the same task with and without the injection, so you can separate "the injection made the agent stop" from "the agent refused anyway".

Choosing a checkpoint. Plot each checkpoint on a two-axis frontier: adaptive ASR (lower is better) against benign utility or over-refusal rate. Prefer checkpoints that move the frontier over ones that slide along it. Set hard gates (ASR below a threshold on held-out adaptive attacks, over-refusal no worse than the previous release beyond a confidence interval), then optimize the rest. A weighted sum hides the trade and invites the refuse-everything hack (Q8).

Experiments: ablate training data containing refusals near benign triggers and measure both axes; test whether system-level defenses (Q15) let you relax in-model refusal pressure.

Why this is likely: the posting covers over-refusal within agent safety, and Gemini model cards report unjustified refusals as a release metric alongside safety.

#### Q15. Compare CaMeL-style system-level defenses with in-model hardening. When is each the right tool, and how would they combine in a Gemini agent?

CaMeL (authors from Google, Google DeepMind and ETH Zurich) treats prompt injection as a software security problem. A privileged LLM sees only the trusted user query and writes a program in a restricted Python dialect that fixes the control flow: which tools to call and in what order. A quarantined LLM parses untrusted data into values but has no tool access. A custom interpreter attaches capabilities to every value (provenance, such as user, tool or quarantined-LLM output, and the set of allowed readers) and checks security policies before each tool call, for example "do not send data derived from an untrusted email to an address the user did not provide". Untrusted data therefore cannot change the program's control flow, and data flows that violate policy are blocked. On AgentDojo it solved 77 percent of tasks with provable security, against 84 percent for an undefended system.

Limits of CaMeL: a utility cost on tasks where the plan depends on untrusted content ("do what the email says"); policies must be written and maintained per tool; side channels remain (the paper discusses inferring private facts from which branch ran or from error behaviour); and the user can still be socially engineered through the final text.

In-model hardening (adversarial fine-tuning, instruction hierarchy training) makes the model itself better at telling instructions from data. It generalizes across products and costs no utility at inference, and the Gemini report argues some model-level judgment is necessary. But it is probabilistic and supports no claim of robustness to unseen adaptive attacks.

How they combine, as defense in depth in the Gemini team's framing:

1. Model hardening as the base layer on every surface.
2. Input and output classifiers trained narrowly on injection (the report found general safety classifiers had high false positive rates).
3. System-level controls for high-impact actions: provenance tracking, CaMeL-style plans for sensitive workflows, and confirmation for state-changing or external-communication actions while untrusted data is in context. This matches the "Agents Rule of Two" idea, under which an unsupervised session combines at most two of untrusted input, sensitive data and external actions.

One research question worth raising is whether post-training can make the model a better planner under CaMeL (fewer utility failures) while also hardening it for surfaces where the plan must depend on data. Measure utility under attack and adaptive ASR for each layer, alone and stacked.

Why this is likely: CaMeL is a Google DeepMind co-authored system defense, the posting names in-model and out-of-model protections, and the Gemini report argues for layered defenses.

## Prep checklist

- [ ] Read "Lessons from Defending Gemini Against Indirect Prompt Injections" (arXiv 2505.14534) and be able to explain the threat model, the four attack families (Actor Critic, Beam Search, TAP, Linear Generation), and the five lessons.
- [ ] Read CaMeL (arXiv 2503.18813) and sketch the privileged and quarantined LLM split, capabilities and policies from memory, plus its utility cost and side channels.
- [ ] Learn the AgentDojo metrics (benign utility, utility under attack, targeted ASR) and run one AgentDojo suite locally against an open model.
- [ ] Read "The Attacker Moves Second" (arXiv 2510.09023) and prepare a two-minute explanation of why static evaluations overstate robustness.
- [ ] Derive the RLHF optimal policy and DPO on paper without notes, including why `log Z(x)` cancels.
- [ ] Implement from memory and test against PyTorch: stable log-sum-exp and cross entropy, causal multi-head attention with padding, the DPO loss, and the k1, k2, k3 KL estimators.
- [ ] Write a PPO clipped surrogate loss with GAE and a GRPO loss, and explain how each assigns credit across a multi-turn agent trajectory.
- [ ] Study the reward model overoptimization functional forms (arXiv 2210.10760) and be ready to draw proxy and gold reward against `sqrt(KL)`.
- [ ] Read the likelihood displacement paper (arXiv 2410.08847) and know the refusal-rate example and the CHES filtering fix.
- [ ] Practice the statistics: Wilson intervals, McNemar's test, cluster bootstrap, and why the best trigger matters more than the average.
- [ ] Review XSTest and OR-Bench, then design an agent-specific over-refusal set with paired benign and attacked versions.
- [ ] Prepare a 45 to 60 minute research talk with explicit negative results and limitations, and drill the oral fundamentals quiz (linear algebra, probability, MLE, regularization as priors).

## Sources

- https://deepmind.google/careers
- https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf
- https://www.sundeepteki.org/advice/the-ultimate-ai-research-scientist-interview-guide-cracking-anthropic-openai-google-deepmind-top-ai-labs-in-2026
- https://igotanoffer.com/en/advice/google-deepmind-research-engineer-interview
- https://www.datainterview.com/blog/google-deepmind-research-engineer-interview
- https://www.interviewquery.com/guides/google-ai-research-scientist
- https://github.com/landedjobs/ai-interview-guides/blob/main/guides/google-deepmind.md
- https://github.com/ombharatiya/AI-Engineer-Interview-Questions/blob/main/14-company-interview-questions/google-deepmind.md
- https://www.reddit.com/r/MachineLearning/comments/1q2wiub/d_google_deepmind_research_engineerscientist/
- https://arxiv.org/abs/2505.14534
- https://deepmind.google/blog/advancing-geminis-security-safeguards
- https://arxiv.org/abs/2503.18813
- https://arxiv.org/abs/2406.13352
- https://arxiv.org/abs/2510.09023
- https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers
- https://arxiv.org/abs/2305.18290
- https://arxiv.org/abs/2210.10760
- https://arxiv.org/abs/2410.08847
- http://joschu.net/blog/kl-approx.html
- https://huggingface.co/blog/NormalUhr/kl-divergence-estimator-rl-llm
- https://deepwiki.com/huggingface/trl/5-group-relative-policy-optimization-(grpo)
- https://arxiv.org/abs/2501.17161
- https://en.papernotes.org/ICML2025/social_computing/or-bench_an_over-refusal_benchmark_for_large_language_models
- https://storage.googleapis.com/deepmind-media/Model-Cards/Gemini-3-Pro-Model-Card.pdf
- https://arxiv.org/html/2510.11851v3
- https://arxiv.org/html/2505.12843v1
- https://huggingface.co/datasets/rl-llm-wiki/knowledge-base/blob/main/topics/reward-modeling/process-vs-outcome-rewards.md
- https://huggingface.co/papers/2404.13208
- https://arxiv.org/html/2403.14720v1
- https://skyzh.github.io/tiny-llm/week1-01-attention.html
