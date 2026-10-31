# NVIDIA: Senior Software Engineer, Agentic AI

Likely interview questions based on the public job listing, public hiring guides, and candidate reports. These are estimates, not real or leaked questions.

## Likely interview loop (estimate)

This estimate comes from NVIDIA's public hiring page, third-party guides, and anecdotal candidate reports. Exact rounds vary by team.

1. Recruiter call, about 30 minutes, covering your background, motivation, and level fit.
2. Technical screen, 30 to 60 minutes, with live coding and a deep dive on a past project. NVIDIA says coding exercises usually run on HackerRank, and it expects you to work without unapproved tools such as ChatGPT.
3. Hiring manager conversation about what you have built, why you want to work on agent tooling, and how you work with open-source communities.
4. A final loop of about 3 to 5 interviews, 30 to 60 minutes each, including an in-person onsite that NVIDIA says is required before an offer. Expect a mix of coding (likely Python or Rust, with async work), library and API design, agent performance and evaluation, a systems deep dive, and a behavioral round.
5. Decision. NVIDIA says it takes a few weeks, and candidates report waiting several weeks after the onsite.

## Questions and answers

### Agent architecture

#### Q1. Compare a single ReAct loop, a planner-executor design, and a multi-agent supervisor. When would you pick each one?

I'd compare them on four things: latency, cost, how much control you have, and how each one fails.

A single ReAct loop has the model reason, call a tool, read the result, and repeat. It is the simplest design and the easiest to debug, and it works well for short tasks with a handful of tools. On long tasks it drifts, repeats calls, and burns tokens as the context grows.

A planner-executor splits the work. One call drafts a plan, cheaper calls or plain code run the steps, and the planner revises only when a step fails. That cuts cost and makes long tasks more predictable. A bad plan carries through every step, so you add triggers that send the work back for replanning.

A supervisor with specialist agents makes sense when the tools or domains are very different, or when each agent needs its own permissions. You pay for it with more LLM calls, harder tracing, and coordination bugs such as agents waiting on each other.

I'd pick between them with evals. Run the same task set through each design and compare success rate, tokens, and p95 latency. A library like the one this team builds should make switching between patterns cheap, so users can measure the difference on their own tasks.

#### Q2. How do you make tool calling reliable when the model sends malformed or wrong arguments?

I treat a tool call as untrusted input from a probabilistic component and handle it in layers.

First, constrain the output. Use the provider's structured output or JSON schema mode, or constrained decoding when you host the model yourself, and malformed JSON becomes rare.

Second, validate every call against a typed schema before running it, with Pydantic in Python or serde in Rust. When validation fails, send the model a short, specific error such as "field date must be ISO 8601, got 'next Tuesday'" and allow a bounded number of retries, usually one or two. If the model keeps making the same mistake, the cap ends the loop, and the agent then asks the user or fails cleanly.

Third, make tools safe to retry. Writes take idempotency keys, destructive actions get a dry-run mode, and read tools stay separate from write tools.

Fourth, cut down on arguments that are valid but wrong. Clear tool descriptions, fewer overlapping tools, and examples in the prompt all help here.

Finally, log every validation failure and wrong call with its trace. Those logs become regression cases in the eval suite. I'd also track tool-call accuracy as a metric per model and per tool, so you can see which combinations need work.

#### Q3. The agent ecosystem changes every month. How do you decide which new patterns or protocols a core library should support?

You want to show that you follow the field on purpose and have a filter for what goes into a library people depend on.

My inputs are papers and benchmarks, releases from the major frameworks and model providers, issues and requests from our own users, and what internal NVIDIA teams are building.

Then I ask four questions. Does it have real adoption? Does it fit the core abstractions, so users still have one way to do each job? What will it cost to maintain, counting tests across frameworks and versions? Does it bring measurable gains in speed, cost, or quality?

If something passes, I prototype it quickly behind an experimental flag or in a separate package, publish it clearly labeled "unstable", and watch usage and issue volume. Once the interface settles and users rely on it, it moves to stable. If usage stays low, it comes out with a deprecation notice.

Bring a concrete example, such as adding support for a new tool protocol through an adapter that leaves the core as it is. Also bring a time you bet wrong, and explain how you unwound it while keeping users' code working.

### Performance and cost

#### Q4. An agent workflow takes 40 seconds end to end. How do you find the latency and cut it while keeping quality?

Measure before changing anything. Add a span for every LLM call, tool call, retrieval, and framework step. Then look at the critical path of one slow run, and at the aggregate over many runs.

A few things usually dominate: sequential LLM calls, long prompts, slow tools, and retries. Each has its own fix. Independent tool calls and sub-agents can run in parallel. Streaming tokens lets downstream steps start early. Caching tool results and reusing prompt prefixes lets the server reuse its KV cache. You can trim context with fewer retrieved chunks, summarized history, and shorter system prompts. Simple steps such as classification or formatting can go to a smaller, faster model. Better stopping conditions cut wasted loops.

On the serving side, compare time to first token with tokens per second. That tells you whether you are waiting in a queue or paying for generation.

Check every change against an eval set. Cutting context or switching models can lower accuracy with no visible error, and the eval set is how you catch it.

When you report results, give p50 and p95 latency and separate wall-clock time from compute time. Then say which wins are general enough to build into a shared library and which were fixes for this one workflow.

#### Q5. How would you cut the cost of a production agent in half while keeping quality the same?

Start with an accounting of tokens in and out per step, per model, and per tool, plus non-LLM costs like retrieval and sandboxes. Most of the spend often sits in a few steps, usually ones that resend a large context every turn.

Then work through the levers, roughly in order of effort. Prompt and prefix caching makes repeated context cheaper. Retrieval can return fewer, better chunks. Conversation history can be summarized or windowed. A smaller model can handle routing, extraction, and formatting while the large model keeps the hard reasoning. Loop iterations get a cap, with an early exit once the answer is already good. Where latency allows, calls can be batched.

Every lever needs a guardrail. Set up an eval suite with task success, tool-call accuracy, and a few hard cases, and accept a change only when quality stays inside a set tolerance. Track cost per successful task, because a cheaper model that fails more often can raise total cost through retries.

In the interview, name the tradeoff between cost and latency. Give a rollout plan with an A/B or shadow test, and add dashboards that catch cost regressions after future changes.

#### Q6. You have a Rust core with Python bindings on an event-emission hot path. Where does the overhead come from, and how do you reduce it?

The overhead comes from five places. Every call across the Python and Rust boundary has a fixed cost. Converting Python objects to Rust types allocates and copies. Acquiring and releasing the GIL takes time and can serialize threads. Serializing every event to JSON or another format is expensive. Per-event allocation also adds up at high rates.

I'd measure before fixing. Use criterion for microbenchmarks on the Rust side and pyperf or pytest-benchmark on the Python side, then run py-spy or perf on a realistic workload to see the whole path.

The fixes follow from the sources. Batch events so the code crosses the boundary once per batch. Keep data in Rust-owned structures and give Python lightweight handles. Release the GIL with allow_threads while Rust does heavy work. Use zero-copy buffers where you can. Serialize lazily, only when an exporter needs the data. Move export to a background Rust thread or a Tokio task with a bounded channel.

Batching has its own cost. It adds latency and makes ordering and flushing on shutdown harder, so say how you would handle both.

Mention PyO3 and maturin by name. Set an overhead budget, such as a few microseconds per event, and test that the program behaves identically with tracing turned on.

### Evaluation and observability

#### Q7. Design a benchmarking and regression system for agent workflows.

Start with what to measure: task success, trajectory quality (the right tools, called in a sensible order, without looping), tool-call accuracy, groundedness for RAG steps, cost, and latency.

Build versioned datasets of tasks with expected outcomes or rubrics. Draw them from real traces, hand-written hard cases, and past failures.

For scoring, use exact checks wherever you can, such as whether the API got the right fields or whether the output parses. Use LLM-as-judge for open-ended answers, and calibrate the judge against a set of human labels while tracking how often the two agree.

Agents are nondeterministic, so run each case several times and report means with confidence intervals. That way the CI gates fire on real regressions and stay quiet on noise.

The runner should replay recorded tool responses so runs are fast and repeatable, with a slower live mode for end-to-end checks. Store results by commit, model, and config so you can diff two runs and see which cases flipped. Every production failure becomes a new eval case.

Then talk about keeping eval runs affordable, the risk of overfitting to your own benchmark, and how the same system lets you compare models, prompts, and framework versions.

#### Q8. How do you instrument a third-party agent framework with OpenTelemetry while keeping its behavior the same for users?

The user's program has to behave exactly the same with instrumentation on, and every design choice follows from that.

Hook in through public extension points first: callback handlers, middleware, or documented events. Wrap functions only when no hook exists, and keep those wrappers thin and enabled only for framework versions you have tested. Inside every hook, catch all exceptions and log them, pass return values and exception types through exactly, and keep side effects in their original order.

Context propagation is the hard part in async code. Use contextvars in Python or the tracing context in Rust so spans nest correctly across tasks, threads, and streaming generators. Map events to OpenTelemetry spans and attributes with the emerging generative AI semantic conventions, so the traces work in any backend.

Bound the overhead with sampling, batch span processors, and async exporters whose bounded queues drop spans when full, so the application thread moves on immediately. Capture prompts and outputs only when the user opts in, since they can hold sensitive data.

To test it, run the framework's own test suite with instrumentation on and off and compare the results, and add overhead benchmarks. Also explain how you support several framework versions and how spans get flushed at shutdown.

### Library and API design

#### Q9. Design a framework-agnostic library that adds profiling and tracing to agents built on several popular frameworks.

I'd split it into three layers.

At the bottom sits a small, stable event model. It covers workflow start and end, LLM calls, tool calls, and retrieval, and each event carries ids, parent ids, timestamps, token counts, and a free-form attributes map.

In the middle, an event bus receives events and fans them out asynchronously to subscribers.

At the edges are adapters and exporters. Each adapter translates one framework's callbacks into the shared events, and exporters send events to OpenTelemetry, files, or analytics backends. Both load as plugins, so adding a framework leaves the core unchanged.

Profiling is built on top of the event stream, covering per-step latency, token usage, concurrency, and critical-path analysis. Configuration should be declarative so users can turn it on through config alone.

There are two tradeoffs to name. A lowest-common-denominator schema loses framework-specific detail, so keep an extension field for it. Deep adapters break when frameworks change, so pin them and test against version ranges.

This code runs on every call, so batch events and export them off the hot path. Round out the design with schema versioning, a test matrix across frameworks, and a clear plan for users who mix frameworks in one workflow.

#### Q10. Design a middleware or plugin system that intercepts agent requests and responses, including guardrails.

I'd model it as an ordered chain. Each middleware receives a typed context holding the request, metadata, and a per-request state bag, and it can act before the call, after the call, or both.

A before-hook can modify the request or short-circuit with its own response. That is how a policy engine blocks a disallowed tool call or a guardrail rejects unsafe input. An after-hook can inspect or rewrite the output, for example to redact secrets or check a response against a policy.

Make the order explicit, by registration order or by priority, and document it, because guardrails usually have to run before logging and caching. Support both sync and async middleware, plus streaming responses, where after-hooks may need to run on each chunk.

Decide per middleware what happens when it errors. Security checks fail closed, and metrics fail open. Give each hook a time budget and measure its overhead.

Keep the interface small and well typed, because a flexible chain is easy to misuse. Then cover how you test the chain in isolation, how plugins are discovered and versioned, and how guardrail decisions get logged for audit and for later evals.

#### Q11. How do you evolve a public cross-language API and keep it backwards compatible?

Start from what users depend on: names, signatures, defaults, error types, serialized formats, and behavior. Use semantic versioning and treat every one of those as part of the contract.

Make changes additive where you can. Add optional parameters with safe defaults, new methods, and new fields that old clients ignore. When something has to go, deprecate it first with a warning that names the replacement, keep the old path for a published window, and remove it in a major version.

Across several languages, define the API once, in a schema or a core crate, and generate the bindings or mirror them carefully. Naming, error models, and async behavior then stay consistent. Each language can follow its own idioms, such as snake_case in Python, as long as the semantics match.

Version any wire or file formats, and make readers tolerant of unknown fields.

Back all of this with tests. Compatibility suites run old client code against new versions, and API diff checks in CI flag breaking changes.

Strict compatibility slows cleanup, so put new features in experimental namespaces. Have a real example ready of a breaking change that was handled well or badly.

### Async systems and coding

#### Q12. Walk through the lifecycle of an async request in an agent runtime, including timeouts, cancellation, and backpressure.

A request arrives and becomes a task on the event loop, asyncio in Python or Tokio in Rust. The task builds its context, then awaits LLM calls and tool calls, often spawning child tasks to run tools in parallel. Use structured concurrency, such as TaskGroup in Python or JoinSet in Rust, so children finish before the parent does and their errors propagate up.

Timeouts belong at two levels. Each external call gets its own, and the request has an overall deadline that shrinks the remaining budget as time passes.

Cancellation flows down. When the client disconnects or the deadline passes, cancel the children, close streams, and release resources in finally blocks or Drop implementations. Make sure every layer re-raises cancellation errors after its cleanup.

Backpressure keeps the runtime standing under load. Use bounded queues and per-provider semaphores on concurrent LLM calls, and when capacity runs out, reject or queue new work with a clear error so memory stays bounded. Callbacks and events emitted along the way should be non-blocking, so the request keeps moving when an observer is slow.

Round it out with retries that use jitter and limits, idempotency for retried tool calls, and spans that record cancellation and timeout reasons so you can observe all of it.

#### Q13. Coding: implement an async executor that runs up to N tool calls at once, with per-call timeouts, and returns results in input order.

Ask clarifying questions first. Should one failure cancel the rest? What does a timed-out call return? Is N global or per tool?

In Python, a typical solution uses an asyncio.Semaphore of size N. Each call runs inside a wrapper that acquires the semaphore, applies asyncio.wait_for with the timeout, and returns a result object holding either the value or the error, tagged with its index. Launch all the wrappers and gather them. Each result carries its index, so the output list comes back in input order.

Returning errors as values lets the other calls keep running when one fails. If the spec asks for fail-fast behavior, use a TaskGroup and let the first exception cancel the group.

In Rust the shape is the same, with a Tokio Semaphore, tokio::time::timeout, and a JoinSet, writing each result into a pre-sized vector by index.

Talk through complexity and the edge cases: an empty input, N larger than the number of calls, N of zero, calls that ignore cancellation, and exceptions raised before the first await.

Finish with tests. A fake slow tool lets you prove that the concurrency limit holds and that results stay in input order when calls finish out of order.

### Behavioral and collaboration

#### Q14. Tell me about a hard performance problem you traced through several layers of a system.

Pick a story that crosses boundaries, ideally an API call that goes down through a runtime, a binding layer, or serialization. That matches what this team does every day.

Tell it in order. Start with the situation: what was slow, how you knew, and the impact in numbers. Then describe your approach, meaning the hypotheses you formed, the tools you used, such as perf, py-spy, flame graphs, and tracing spans, and how you ruled out the wrong leads. State the root cause precisely, for example per-event serialization across a language boundary, lock contention, or a callback blocking the event loop.

Next comes the fix, with the tradeoffs you accepted, and then the result as before and after numbers for p95 latency, throughput, or CPU. Close with the benchmark, CI check, or dashboard you added to keep the problem fixed.

A systems-heavy team will dig into the details, so be ready to explain why the fix worked at the level of memory, threads, or the event loop. Walk through your debugging calmly and step by step, own any mistakes you made along the way, and show the measurements that prove the fix worked.

#### Q15. Describe a time you worked with an open-source community or outside contributors to land a contested change.

This role works directly with open-source users and contributors, so the story should show you can lead without authority.

Choose a real case where people disagreed. It might be an API change, a new dependency, a performance tradeoff, or a feature some users wanted and the maintainers did not. Explain the context and who had a stake in the outcome.

Then describe how you made the case. Maybe you wrote an issue or a design document, backed it with benchmarks or user data, and built a prototype people could try. Show that you listened by naming what you changed in your proposal because of feedback.

Cover how the decision got made, how you handled the people who still disagreed, and how you kept contributors engaged, for example by crediting their work or giving them ownership of a follow-up.

End with the outcome and what you learned about maintaining a project others depend on, such as writing migration guides or setting clearer contribution rules. Be specific, give credit to others, and show respect for users' time and the stability they count on.

An internal platform or shared library with many consuming teams works as the example if that is where your experience is.

## Prep checklist

- [ ] Read NVIDIA's open-source NeMo Agent Toolkit docs on profiling, observability, and evaluation to see the kind of tooling this area builds.
- [ ] Build a small agent in two frameworks and trace both with OpenTelemetry, so you can explain where the time and tokens go.
- [ ] Practice async coding in Python and Rust with semaphores, timeouts, cancellation, and ordered results.
- [ ] Write a tiny PyO3 extension with maturin and benchmark per-call against batched calls across the boundary.
- [ ] Sketch a framework-agnostic event schema and a plugin design for tracing and middleware.
- [ ] Design an agent eval system on paper, covering datasets, metrics, judges, variance handling, and CI gates.
- [ ] Prepare two performance stories with before and after numbers, plus one collaboration story from open source or a shared library.
- [ ] Review DSA fundamentals for a HackerRank-style coding screen and practice solving problems on your own, since the screen bars outside AI tools.
- [ ] Confirm the role is still open, since the listed application date has passed.

## Sources

- https://www.nvidia.com/en-us/about-nvidia/careers/how-we-hire/ : NVIDIA's public hiring page, covering interview length, HackerRank coding, the onsite requirement, and the rule against outside AI tools.
- https://igotanoffer.com/en/advice/nvidia-software-engineer-interview : third-party guide to the stages and timeline of NVIDIA software engineer interviews.
- https://ophyai.com/blog/company-guides/nvidia-interview-guide : third-party 2026 guide covering screens, onsite rounds, and timelines.
- https://www.reddit.com/r/leetcode/comments/1sa2i2y/systems_software_engineer_interview_process_at : anecdotal candidate report about deep questions on the resume.
- https://www.reddit.com/r/embedded/comments/1rp6dhk/nvidia_interview_on_hold_after_final_onsite : anecdotal report of waits after the final onsite.
- https://github.com/NVIDIA/NeMo-Agent-Toolkit : NVIDIA's open-source agent toolkit, with profiling, observability, and evaluation features.
- https://docs.nvidia.com/nemo/agent-toolkit/latest/run-workflows/observe/observe.html : event-driven, plugin-based observability with OpenTelemetry exporters.
- https://docs.nvidia.com/nemo/agent-toolkit/latest/improve-workflows/profiler.html : callback-based profiler for latency, throughput, and bottleneck analysis.
- https://blogs.nvidia.com/blog/nvidia-agent-toolkit-open-models-tools-skills-secure-runtime-ai-agents : NVIDIA blog post describing its Agent Toolkit of models, tools, skills, and a secure runtime.
