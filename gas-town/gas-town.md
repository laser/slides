# Multi-Agent Coding: The Debate We Keep Avoiding

- Provocative discussion deck for software engineers
- Lens: Gas Town as signal, not hero product
- Date: February 5, 2026


---

# Debate Motion

- This house believes 10+ concurrent coding agents will be standard for top engineering teams by 2028.


---

# A Second Motion (Stronger)

- This house believes most teams trying multi-agent orchestration in 2026 will make their systems worse before they make them better.


---

# Ground Rules

- We argue outcomes, not vibes
- PR count is not a success metric
- "Works on my branch" is not a success metric
- Cost without quality is failure at scale


---

# Provocation #1

- Multi-agent coding is either the next dev platform or the next microservices: huge upside, huge accidental complexity.
- There is no neutral path.


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


---

# Hard Evidence That Helps The Negative

- Anthropic reports roughly 15x token usage vs chat interactions
- Anthropic also notes many coding tasks are poor candidates for multi-agent decomposition
- Field reports show weak first-pass PR quality and unsafe merge behavior under speed pressure
- Inference: strong upside exists, but naive rollout is expensive and failure-prone


---

# The Throughput Trap

- "36 PRs in 4 hours" sounds like a breakthrough
- But who reviews, integrates, and owns those changes?
- If review/merge/rollback capacity does not scale too, throughput is theater
- True metric: quality-adjusted throughput per engineer-hour and per dollar


---

# The Merge Queue Reality

- Parallel workers targeting one codebase create constant moving-head conflicts
- Gas Town needed a dedicated Refinery role to serialize merge decisions
- This is distributed systems coordination in disguise
- If your interfaces are weak, more agents amplify entropy


---

# The Economics Fight

- Multi-agent throughput can burn budget rapidly
- Token costs are not the full cost; rework and incident risk dominate
- Cheap models reduce unit cost but may increase supervision cost
- The winning system minimizes total cost of correct change


---

# OSS vs Frontier Labs: Who Owns The Stack?

- OSS dominates orchestration experimentation
- Frontier labs dominate model capability and managed controls
- Emerging standards (AGENTS.md, MCP) weaken lock-in but do not erase it
- Most likely outcome: hybrid stacks, with leverage shifting as model APIs absorb orchestration primitives


---

# Organizational Flashpoint

- Adopters may ship faster and set new internal expectations
- Non-adopters may look slow on shallow metrics
- Leadership risk: rewarding visible output over durable outcomes
- Culture question: do we optimize for "more code" or "fewer regrets"?


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

# Forced Choice for Our Team

1. Conservative: stay single-agent, harden evals, revisit in 6 months
2. Moderate: pilot 2-3 workers on one decomposable workflow
3. Aggressive: build orchestration now and redesign review/merge pipelines


---

# If We Pilot, Define Failure Up Front

- Failure if escaped defects rise
- Failure if lead time drops but rework rises
- Failure if token cost drops while human review time explodes
- Failure if no measurable gain after 30 days


---

# Debate Prompts (High Friction)

- Are agents best framed as junior engineers or as unsafe build tools?
- Should any system allow autonomous merge without mandatory gate policy?
- What is our maximum safe concurrency today: 1, 3, 5, or 10+?
- Which would you rather optimize: iteration speed or incident rate?


---

# Closing Position

- "Add more agents" is not a strategy.
- Orchestration plus governance might be.
- The teams that win will not be the most agentic.
- They will be the most disciplined at converting agent output into reliable software.


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
