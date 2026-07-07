# Funding the Shift to Autonomous AI: A 100-Agent Fleet Cost Model and Architecture Playbook (2027–2028)

*Part 2 of 2 — builds on the Part 1 disclosure analysis of 15 non-hyperscaler enterprises*

---

## Critical framing — read this before the numbers

Everything in this document is a **modeled, illustrative estimate**, built bottom-up from real, currently published 2026 inference pricing, tooling pricing, and documented case studies of agent cost behavior. **None of it is company-disclosed data.** No company in the Part 1 list — or anywhere else — publicly discloses a "100-agent fleet" cost line. Where Part 1 said "not disclosed separately," this document does not fill that gap with a disguised estimate; it builds a separate, clearly-labeled model and shows its assumptions and sources so you can stress-test or replace any input.

Two things make this modeling exercise unusually volatile compared to normal IT cost forecasting:
1. **Per-token prices are falling** — Claude Opus dropped 67% ($15/$75 → $5/$25 per million input/output tokens) between Opus 4.1 (mid-2025) and Opus 4.6 (Feb 2026); OpenAI's GPT-5.4 sits at $2.50/$15.
2. **Token *consumption* is rising much faster than prices are falling.** Goldman Sachs Research projects enterprise/consumer token consumption will multiply **24x between 2026 and 2030** (to 120 quadrillion tokens/month), driven specifically by the shift from single-turn queries to "always-on" background agents.

That combination — cheaper unit price, exploding volume — is exactly why real enterprises are already getting burned: Uber exhausted its entire 2026 AI coding budget in four months after rolling Claude Code out to ~5,000 engineers; one unnamed enterprise reportedly spent **$500 million in a single month** after deploying AI access with no usage caps; a documented 100-agent coding fleet (Peter Steinberger's OpenClaw project, May 2026) generated **603 billion tokens and $1.3 million in spend over 30 days** — annualized, roughly **$15.6M/year for 100 agents** — with 70% of that traceable to a single unmanaged setting (a high-rate "Fast Mode" configuration).

That $15.6M/year figure is the cautionary upper bound this model deliberately does **not** assume. It is real, sourced, and should sit in the back of your mind as "what happens with zero governance." The two archetypes below assume competent FinOps discipline at different levels of model-tier ambition.

---

## Deliverable 3: Illustrative 2027–2028 Cost Model for a 100-Agent, 24/7 Fleet

### Stated assumptions (change any of these and the output changes proportionally)

| Assumption | Value used | Source basis |
|---|---|---|
| Fleet size | 100 agents, running continuously (24/7/365) | Per your stated scope |
| Cycle cadence | 288 reasoning cycles/agent/day (~1 every 5 min) | Illustrative; consistent with "always-on background agent" framing in Goldman Sachs Research commentary |
| Cycle mix | Split into lightweight "heartbeat" cycles vs. heavier "active task" cycles — ratio differs by archetype (below) | Reflects the "agent loop" cost-accumulation pattern documented in LeanOps' 2026 audit of 30 production agentic teams |
| Context growth | Active cycles carry larger accumulated context (10K–30K input tokens) than heartbeat cycles (2K–5K), because agents resend full conversation/tool history each step (LLM APIs are stateless) | LeanOps 2026 audit; TechAhead 2026 analysis of "quadratic" agentic cost curves |
| Model pricing | Per Anthropic and OpenAI's own published rate cards, May–June 2026: Haiku 4.5 $1/$5, Sonnet 4.6 $3/$15, Opus 4.8 $5/$25 per MTok (input/output); GPT-5.4 $2.50/$15, GPT-5.5 $5/$30 | platform.claude.com/docs/en/about-claude/pricing; developers.openai.com/api/docs/pricing |
| Prompt caching | Cached input reads run ~90% cheaper than fresh input on Claude models; caching discipline differs sharply by archetype | Anthropic pricing documentation |
| Batch API | Not modeled here (would add a further 50% discount on tolerant, non-time-sensitive workloads) — flagged as a lever, not baked into the base case | Anthropic/OpenAI Batch API pricing |
| Tooling costs | Priced from publicly listed 2026 vendor rate cards (Langfuse, LangSmith, Datadog LLM Observability, Helicone) | See citations inline below |

### Archetype A — "Pure OpEx Reallocator" (Capital One / Block pattern: existing cloud commitment, aggressive model routing and caching, budget reallocated from existing tech spend rather than net-new)

**Per-agent daily profile (modeled):** 230 heartbeat cycles/day at 2,000 input / 200 output tokens, routed to Haiku 4.5, 80% cache-hit rate on input; 58 active-task cycles/day at 15,000 input / 1,000 output tokens, routed to Sonnet 4.6, 50% cache-hit rate.

| Cost component | 2027 (modeled) | 2028 (modeled) | Basis / caveat |
|---|---|---|---|
| Inference/token cost | ~$97,000/yr | ~$113,000/yr | Bottom-up calc from the profile above; 2028 assumes ~45% volume growth (Goldman's agentic-adoption curve) partly offset by ~20% further per-token price decline |
| FinOps/observability tooling | ~$45,000/yr | ~$50,000/yr | Self-hosted Langfuse (MIT-licensed, free core software) + hosting/maintenance; consistent with vendor guidance that self-hosted OTel-native tools suit teams under ~$200K/mo LLM spend |
| Data infra (context/memory storage) | ~$35,000/yr | ~$40,000/yr | Incremental vector-store/object-storage cost layered onto existing cloud commitment — no dedicated buildout assumed |
| Orchestration/agent-framework overhead | ~$75,000/yr | ~$85,000/yr | Open-source frameworks (LangGraph/CrewAI-class) — cost is mostly engineering labor and modest compute, not licensing |
| **Annual total (modeled)** | **~$252,000** | **~$288,000** | |
| **2-year total (2027–2028, modeled)** | **~$540,000** | | |

### Archetype B — "High-Inference Scale" (JPMorgan / Bank of America pattern: frontier-model tolerance, net-new dedicated compute budget, larger context windows, lower caching discipline than Archetype A)

**Per-agent daily profile (modeled):** 172.8 heartbeat cycles/day at 5,000 input / 500 output tokens, routed to Sonnet 4.6, 20% cache-hit rate; 115.2 active-task cycles/day at 30,000 input / 2,000 output tokens, routed to Opus 4.8, 20% cache-hit rate.

| Cost component | 2027 (modeled) | 2028 (modeled) | Basis / caveat |
|---|---|---|---|
| Inference/token cost | ~$852,000/yr | ~$988,000/yr | Same growth/price-decline logic as Archetype A, applied to a frontier-model, lower-caching profile |
| FinOps/observability tooling | ~$275,000/yr | ~$320,000/yr | Enterprise-tier managed observability (LangSmith/Datadog LLM Observability at scale — Datadog LLM span pricing is reported as the most expensive tier in the category, ~$120/day baseline plus volume) + a dedicated agentic spend-governance layer (e.g., Portal26-class token-budget/circuit-breaker tooling); enterprise contract pricing for that governance layer is not publicly listed and is estimated |
| Data infra (context/memory storage) | ~$175,000/yr | ~$210,000/yr | Dedicated vector DB/context-persistence infrastructure sized for larger context windows and longer state retention |
| Orchestration/agent-framework overhead | ~$225,000/yr | ~$260,000/yr | Managed agent-runtime infrastructure, redundancy/failover, higher engineering overhead for a more complex harness |
| **Annual total (modeled)** | **~$1,527,000** | **~$1,778,000** | |
| **2-year total (2027–2028, modeled)** | **~$3,305,000** | | |

### Sanity-check against real-world data points (important caveat)

- These modeled figures sit **well below** Steinberger's documented $15.6M/year run-rate for 100 coding agents in an unmanaged, Fast-Mode-heavy configuration — intentionally, since that case is the "no governance" outcome, not a target architecture.
- They also sit in a plausible range relative to LeanOps' documented 20-engineer agentic coding team, whose *unoptimized* monthly bill was $87,000 ($1.04M/year) before FinOps intervention brought it to $24,000/month ($288K/year) — a different workload shape (coding-specific, not general business-process agents) but directionally consistent with the ~4x optimization delta this model implies between disciplined and undisciplined operation.
- **Do not treat the dollar figures above as precise forecasts.** Treat the *ratio* between Archetype A and B (roughly 6x), and the *direction* of year-over-year change, as the more defensible takeaways than any single number.

---

## Deliverable 4: Architectural Takeaways for a 100-Agent Fleet

### For the Pure OpEx Reallocator track — design for strict token discipline

**Model cascading/routing.** Route by default to the cheapest model capable of the task; escalate only on demonstrated need. The pattern documented across current guidance is a three-tier cascade — cheap/fast model (Haiku-class) for classification, routing, and simple heartbeat checks → mid-tier model (Sonnet-class) for general reasoning → frontier model (Opus-class) reserved for the hardest agentic or long-context work. This is described as "the single biggest cost lever" available before any other optimization.

**Prompt/context caching as a default, not an afterthought.** Cached input reads run at roughly 10% of fresh-input price on current Claude pricing (and a comparable discount on OpenAI's cached-input rate). Structuring agent prompts so that stable system instructions and repeated context sit in a cacheable prefix is described as "a 10x cost reduction with no engineering work" when done correctly — the highest-leverage, lowest-effort lever in this entire model.

**Batch API for tolerant workloads.** Any part of the fleet's workload that does not need sub-second response — nightly reconciliation, batch classification, non-urgent summarization — should route through Batch APIs for a flat 50% discount on both input and output, stackable with caching.

**Sliding-window context pruning.** One documented remediation (the LeanOps case study) added "context pruning with a sliding window of the last 8 steps" and captured a 12% cost reduction on its own, on top of model-routing savings — a concrete, low-risk technique for agents whose reasoning doesn't require the entire history at every step.

**Hard per-agent and per-fleet budget caps, enforced at the gateway.** The documented remediation pattern across multiple 2026 case studies is consistent: a soft daily cap with an alert, a hard daily cutoff that forces a downgrade to the cheapest model, and a monthly ceiling that requires manual approval to exceed. One practitioner reports this pattern "catches 95% of runaway patterns before they happen."

### For the High-Inference Scale track — design for loop boundaries, state persistence, and enterprise FinOps guardrails

**Loop boundaries with machine-verifiable exit criteria.** The most robust pattern documented for long-running autonomous work (the "Ralph Loop" architecture and similar) starts each iteration with a *fresh* context window rather than one continuously-degrading session, reads current state from persistent storage (filesystem, database, or object store), does bounded work, writes updated state back, and repeats — with an explicit, machine-checkable completion criterion (tests passing, a linter clean run, a rubric score) rather than an open-ended "keep going" instruction. This directly targets two of the four most common agent failure modes documented in current engineering guidance: context-window exhaustion and inability to terminate.

**Spawn budgets and iteration ceilings.** Any agent capable of spawning sub-agents or retrying failed steps needs an explicit cap on total spawns/retries per task, not just a per-call token budget — this is the documented fix for the "recursive loop" failure mode that produced the DN42 incident's $6,531 runaway AWS bill (agent re-spinning duplicate cloud stacks on every error with nobody instructing it to stop) and is architecturally identical to the failure that drove Uber past its full annual AI budget in four months.

**Circuit breakers on token velocity, not just total spend.** A rate-based control (tokens-per-minute, not just dollars-per-month) catches a runaway loop in real time — one documented implementation caught a runaway loop within 60 seconds by tripping at a 10,000 tokens/minute threshold, versus a healthy agent's normal sustained rate of 3,000–4,000 tokens/minute.

**Explicit state persistence, not "hope the context window is big enough."** For 100 agents running continuously for two years, state cannot live only in-context. Current architectural guidance converges on the same principle across multiple independent sources (LangGraph checkpointing, the Ralph Loop's filesystem-as-memory, and dedicated agent harnesses like DeerFlow): externalize state to durable storage, checkpoint at defined boundaries, and design for graceful recovery/resume after interruption — this is what allows a 24-month deployment to survive restarts, model version upgrades, and human-in-the-loop pauses without losing continuity or silently re-accumulating cost by replaying full history from scratch.

**Enterprise FinOps as a control plane, not a dashboard.** The distinction that matters at this scale: observability tools (Datadog LLM Observability, LangSmith, Langfuse) tell you what happened after the fact; only a small, newer category of tooling (e.g., Portal26-class agentic token controls) actively *enforces* budgets — throttling, pausing, or killing an agent in real time when it approaches a defined ceiling. At High-Inference Scale, the FinOps Foundation's own 2026 commentary is blunt: the industry conversation shifted in a matter of months from "move fast" to "we need guardrails, how do we control this" — after multiple enterprises found themselves several multiples over their entire annual token budget by April. Build the enforcement layer before the fleet goes live, not after the first surprise invoice.

---

## Consolidated caveats (Part 2)

- All dollar figures in Deliverable 3 are **modeled outputs of stated assumptions**, not disclosed or audited figures. Treat every number as replaceable — the value of this exercise is the *structure* of the model (which cost components exist, how they scale, what levers move them), not the specific dollars.
- Pricing was current as of the search dates in mid-2026 and is a fast-moving target; Anthropic and OpenAI have both changed per-token pricing multiple times within the past 12 months. Re-verify rate cards before using this model for an actual budget submission.
- The "High-Inference Scale" and "Pure OpEx Reallocator" labels are illustrative archetypes inspired by the spending *patterns* observed in Part 1 (JPMorgan/BofA vs. Capital One/Block), not a claim that those specific companies run agent fleets shaped exactly like this model.
- Real-world outcomes have proven highly bimodal — either well-governed (LeanOps' optimized $24K/month case) or ungoverned (Steinberger's $1.3M/month case) — with limited middle ground reported so far in the public record. Budget for the governance work, not just the inference.
