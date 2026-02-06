---
author: Sources keyed by tag; see "Citation Key" slide.
date: YYYY-MM-DD
paging: Slide %d / %d
---

# Multi-Agent Coding: How Nuts is Steve Yegge?


---

# What Yegge Meant By "Gas Town"

- Oh you didn't actually read the blog post?
- Polecats?


---

# Gas Town

- "Gas Town" is Yegge's OSS product for a high-throughput coding environment with many AI agents working in parallel
- The model is orchestration-first: specialized roles, explicit handoffs, and durable workflow state instead of one long chat
- The objective is to turn model parallelism into sustained software delivery, not isolated one-off completions
- The implied risk is governance: review, integration, and ownership can become the bottleneck

###### Sources: [Y1] Gas Town concept + workflow framing; [Y2] broader orchestration trajectory.


---

# Cool Story, Now What?

- This shall be neither a Gas Town Fan Club, nor a Playa Haters' Ball
- We will stress-test Yegge's swarm pitch from both sides
- We separate "zomg, so many PRs" from "did anything good actually ship?"
- Then we pick a strategy that improves outcomes without lighting quality or budget on fire


---

# Ground Rules

- PR count is not a success metric
- LOC is not a success metric
- Cost without quality is failure at scale
- We must focus on outcomes, not vibes


---

# Big Bet One

- By 2028, top engineering teams will probably treat 10+ concurrent coding agents as normal


---

# Big Bet Dos

- Most teams trying multi-agent orchestration in 2026 will likely make their systems worse before they make them better


---

# Hot Take Número Uno

- Multi-agent coding could be either the next big dev platform or another microservices mess: huge upside, huge accidental complexity


---

# The Affirmative Case: Why Swarms Win

- Throughput jumps on decomposable workloads
- Externalized workflow state beats ephemeral chat memory
- Role specialization reduces local context overload
- Teams that learn orchestration early may compound faster than competitors


---

# The Negative Case: Why Swarms Fail

- Review and merge become the new wall immediately
- Coordination bugs multiply faster than code bugs
- Parallel low-quality output increases hidden rework
- Teams confuse activity for delivery and velocity for value


---

# Hard Evidence That Helps The Affirmative

- Anthropic reports multi-agent outperforming single-agent on its research eval (90.2%)
- Anthropic shows clear gains from parallel specialization in complex workflows
- Gas Town-style systems show that durable orchestration can survive session churn
- Inference: agent orchestration is not fake; it can produce real performance gains

###### Sources: [A1] eval + specialization claims; [Y1] durable orchestration behavior; [C1] role-specialization model.


---

# Hard Evidence That Helps The Negative

- Anthropic reports roughly 15x token usage vs chat interactions
- Anthropic also notes many coding tasks are poor candidates for multi-agent decomposition
- Field reports show weak first-pass PR quality and unsafe merge behavior under speed pressure
- Inference: strong upside exists, but naive rollout is expensive and failure-prone

###### Sources: [A1] token and decomposition constraints; [D1] first-pass PR quality and merge risk anecdote.


---

# A Throughput Trap, No Cap

- "36 PRs in 4 hours" sounds like a breakthrough
- "Running three agents at a time" sounds like a breakthrough
- But who reviews, integrates, and owns those changes?
- If review/merge/rollback capacity does not scale too, throughput is theater
- True metric: quality-adjusted throughput per engineer-hour and per dollar

###### Sources: [H1] anecdotal throughput numbers; [D1] review/integration bottleneck examples; [O2] outcome-over-activity framing.


---

# The Merge Queue Reality

- Parallel workers targeting one codebase create constant moving-head conflicts
- Gas Town needed a dedicated Refinery role to serialize merge decisions
- This is distributed systems coordination in disguise
- If your interfaces are weak, more agents amplify entropy

###### Sources: [D1] merge/review friction in practice; [Y1] Refinery role concept; [Y2] orchestration coordination framing.


---

# The Economics Fight

- Multi-agent throughput can burn budget rapidly
- Token costs are not the full cost; rework and incident risk dominate
- Cheap models reduce unit cost but may increase supervision cost
- The winning system minimizes total cost of correct change

###### Sources: [A1] higher token consumption in multi-agent mode; [D1] supervision/rework overhead in field report; [O2] total-value framing.


---

# OSS vs Frontier Labs: Who Owns The Stack?

- OSS dominates orchestration experimentation
- Frontier labs dominate model capability and managed controls
- Emerging standards (AGENTS.md, MCP) weaken lock-in but do not erase it
- Most likely outcome: hybrid stacks, with leverage shifting as model APIs absorb orchestration primitives

###### Sources: [O1] AGENTS.md + MCP interoperability claim; [O2] stack design guidance; market-share statements are inference.


---

# Organizational Flashpoint

- Adopters may ship faster and set new internal expectations
- Non-adopters may look slow on shallow metrics
- Leadership risk: rewarding visible output over durable outcomes
- Culture question: do we optimize for "more code" or "fewer regerts"?


---

# 2028 Scenario A: Managed Swarms Win

- Most teams run supervisor-plus-worker patterns by default
- Human engineers focus on architecture, policy, and arbitration
- Providers package orchestration natively; custom orchestrators become niche
- Review shifts toward risk triage, invariant checks, and policy gates


---

# 2028 Scenario B: Swarm Winter

- Many teams retreat to stronger single-agent workflows
- Multi-agent stays specialized for research and high-latency planning tasks
- The integration tax and reliability burden kill broad adoption
- "More agents" becomes a cautionary anti-pattern


---

# A Fork in the Road

1. Conservative: stay single-agent, harden evals, revisit in 6 months
2. Moderate: pilot 2-3 workers on one decomposable workflow
3. Aggressive: adopt orchestration now and redesign review/merge pipelines


---

# Before Choosing a Path, Define What Failure Looks Like

- Failure if escaped defects rise
- Failure if lead time drops but rework rises
- Failure if token cost increases faster than throughput


---

# Fin

- "Add more agents" is not a strategy
- Orchestration plus governance might be
- The teams that win will not be the most agentic
- They will be the most disciplined at converting agent output into reliable software


---

# Sources

- Steve Yegge, Welcome to Gas Town: https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04
- Steve Yegge, The Future of Coding Agents: https://steve-yegge.medium.com/the-future-of-coding-agents-e9451a84207c
- Anthropic, multi-agent research system: https://www.anthropic.com/engineering/multi-agent-research-system
- Anthropic, Claude Code subagents docs: https://code.claude.com/docs/en/sub-agents
- OpenAI, practical guide to agents: https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/
- OpenAI, Agentic AI Foundation: https://openai.com/index/agentic-ai-foundation/
- OpenAI, Introducing AgentKit: https://openai.com/index/introducing-agentkit/
- DoltHub, A Day in Gas Town: https://www.dolthub.com/blog/2026-01-15-a-day-in-gas-town/
- Hacker News discussion: https://news.ycombinator.com/item?id=46458936
- GitHub issue #503: https://github.com/steveyegge/gastown/issues/503


---

# Citation Key (For Footer Tags)

- [A1] Anthropic multi-agent research system (90.2% eval win; ~15x token use; decomposition limits): https://www.anthropic.com/engineering/multi-agent-research-system
- [D1] DoltHub field report (PR quality + merge behavior anecdotes): https://www.dolthub.com/blog/2026-01-15-a-day-in-gas-town/
- [Y1] Yegge, Welcome to Gas Town (durable workflow state, Refinery role): https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04
- [Y2] Yegge, The Future of Coding Agents (orchestration framing): https://steve-yegge.medium.com/the-future-of-coding-agents-e9451a84207c
- [C1] Claude Code subagents docs (supervisor-worker specialization pattern): https://code.claude.com/docs/en/sub-agents
- [O1] OpenAI Agentic AI Foundation (AGENTS.md + MCP interoperability): https://openai.com/index/agentic-ai-foundation/
- [O2] OpenAI practical guide to agents (single-agent first, escalate as needed): https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/
- [H1] Hacker News discussion (anecdotal throughput numbers like "36 PRs"): https://news.ycombinator.com/item?id=46458936