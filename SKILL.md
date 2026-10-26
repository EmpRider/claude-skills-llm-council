---
name: llm-council
description: "An engineering decision council for architecture, design, implementation strategy, reliability, security, testing, and technical trade-offs. An accountable Engineering Decision Owner leads a structured discussion with five independent engineering advisors, challenges objections, and records a recommendation with evidence and dissent. Triggers: 'council this', 'run the council', 'engineering council', 'architecture council', 'design review', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. Use for consequential software-engineering decisions, not routine coding, simple factual questions, or requests to implement an already-settled choice."
---

# LLM Council — Software Engineering

Run an evidence-based technical design review. **The Engineering Decision Owner (EDO) is the accountable lead, not a neutral summarizer.** The EDO frames the decision, develops an initial position, listens to independent specialists, discusses substantive disagreements with them, and issues a reasoned recommendation. A real human owner retains final approval and any required organizational sign-offs.

This adapts the independent-advisor, anonymous peer-review, and synthesis ideas from [Karpathy's LLM Council](https://github.com/karpathy/llm-council) to software engineering. It does not imply that several distinct AI models were consulted: advisors may be sub-agents of the same model.

## When to use

Use for meaningful decisions with competing trade-offs, such as:

- Monolith vs modular monolith vs microservices; synchronous vs event-driven integration.
- SQL vs NoSQL; database migration, cache design, consistency, and failure handling.
- Choosing an API contract, authentication design, deployment strategy, or library.
- Refactoring a critical subsystem, changing boundaries, or deciding build vs buy.
- Performance, reliability, security, maintainability, and delivery trade-offs.

Do **not** convene for factual lookups, routine bug fixes with an established solution, simple formatting, code generation, or tasks that merely require executing an accepted decision. If the user explicitly asks to implement, work on implementation according to the user's instructions; do not substitute a council report for code.

## Who owns the decision?

Identify **one EDO** before convening:

| Scope | Default decision-owner role |
| --- | --- |
| System architecture or cross-service contracts | Software Architect or Principal/Staff Engineer |
| Design within an owned service or component | Owning Tech Lead or Senior Engineer |
| Delivery and implementation approach | Engineering Lead responsible for that work |
| Security, privacy, compliance, or production risk acceptance | Engineering owner **plus** the required Security/SRE/compliance approver |
| Product scope, budget, or organizational policy | Engineering owner advises; the authorized product/business owner makes the non-technical decision |

If the repository or user identifies a human owner, use that role/name in the record; **never impersonate their approval**. Otherwise state the assumed owner role (usually *Software Architect / Tech Lead*) and label the result **Proposed decision — pending owner approval**. The AI acts as that role for reasoning and facilitation only. The EDO cannot override explicit user constraints or established policies.

The EDO must **participate in the technical discussion**:

1. State provisional decision criteria and an initial preference **before** seeing advisor conclusions.
2. Invite disagreement; question evidence, risks, constraints, and alternatives.
3. Revisit the initial preference after peer review. Accept or reject each major objection with reasons.
4. Make a clear recommendation, recording remaining dissent and who must sign off.

Do not decide by majority vote, title seniority, or a stylistic preference. The strongest evidence and the user's stated priorities determine the recommendation.

## The five independent advisors

Each advisor is a **technical lens**, not a final decision maker. Use all five when sub-agent orchestration is available. Tailor their focus to the actual problem while keeping their independent mandates:

1. **Systems Architect** — boundaries, APIs, coupling, scalability, evolvability, and consistency with the existing architecture.
2. **Implementation Lead** — implementation effort, maintainability, backward compatibility, developer ergonomics, dependencies, and migration complexity.
3. **Reliability & Security Engineer** — threat model, failure modes, resilience, performance, observability, operations, and safe rollback. State when specialist approval is needed.
4. **Quality & Verification Lead** — test strategy, correctness, edge cases, reproducibility, acceptance criteria, and how to falsify claims.
5. **Devil's Advocate / Alternative Designer** — best credible competing design, hidden assumptions, costs, lock-in, and reasons the favored choice could fail. Attack ideas, not people.

If a domain specialist is critical (database, accessibility, frontend, mobile, data, ML, infrastructure), adapt the closest lens to include that expertise. Do not claim a security sign-off or other professional certification merely because a role was simulated.

## Workflow

### 1. Gather evidence and frame the decision

Review the user-provided problem and relevant repository context. When access is available, inspect only task-relevant sources: `AGENTS.md` / `CLAUDE.md`, README, architecture docs and ADRs, dependency/config files, affected code, tests, CI, and recent changes. Avoid broad scans and unrelated or sensitive files. Follow repository instructions. If tools or files cannot be accessed, state the limitation and do not invent findings.

Build a concise **decision brief** containing:

- Decision question; why it matters; EDO role and actual approval authority.
- Options, including *do nothing/status quo* when credible.
- Requirements, constraints, non-goals, expected workload, delivery horizon, and compatibility needs.
- Evaluation criteria prioritized for this case (e.g. correctness, security, performance, operability, cost, developer experience, time-to-deliver).
- Evidence with file paths, benchmarks, incidents, user data, or references; explicitly distinguish **verified facts**, **assumptions**, and **unknowns**.
- Scope and reversibility of the decision; success measures.

Ask **one focused blocking question** when a missing requirement could reverse the outcome (e.g. data-loss tolerance, compliance, SLA, scale). Otherwise use labeled assumptions and proceed. Do not claim to have run benchmarks, tests, or threat modeling unless actually done.

### 2. EDO preliminary position

Before collecting advisor output, the EDO writes a short private working note:

```
Owner role: ...
Preferred option (provisional): ...
Decision criteria in priority order: ...
Strongest reason: ...
What evidence would change my mind: ...
```

This position is **not** given to advisors. It prevents hindsight bias and gives the EDO something concrete to defend or revise. The final report should say whether it changed and why.

### 3. Independent advisor analysis

Launch the five advisor analyses **in parallel when supported**. Give each the same decision brief and their own lens; do not share other advisors' outputs or the EDO's initial preference. Require specific, testable claims and a real recommendation.

Advisor prompt:

```text
You are the [ADVISOR ROLE] in a software-engineering design council.
Your mandate: [ROLE DESCRIPTION].

DECISION BRIEF:
[neutral brief including evidence, constraints, options, and unknowns]

Independently assess the options. Do not follow a presumed consensus.
Return 150–300 words covering:
1. Preferred option and why (using the stated criteria).
2. Relevant evidence vs assumptions; cite available file paths or measured data.
3. Strongest risk, failure mode, and trade-off.
4. What would change your recommendation.
5. A practical validation step, including test/benchmark/rollout when appropriate.
Flag missing information. Never invent tests, metrics, code inspection, or approvals.
```

If genuine sub-agents are unavailable, perform a clearly labeled **single-agent perspective review** and say that independent agents were not used. Never present self-generated sections as independent votes.

### 4. Anonymous peer review

Randomize the advisor-to-letter mapping (A–E). When supported, send all anonymized analyses to five parallel reviewers; do not reveal author roles during assessment.

Reviewer prompt:

```text
Review these anonymous engineering proposals against the decision brief:
[DECISION BRIEF]
[RESPONSES A–E]

In under 200 words:
1. Which proposal best meets the requirements and why? Identify evidence.
2. Which claim, assumption, or trade-off most needs challenging?
3. What important risk or viable alternative did everyone miss?
4. What concrete test or additional fact would settle the disagreement?
Reference proposals only by letter. Avoid popularity or rhetoric-based rankings.
```

Deduplicate peer-review findings. A shared unsupported assumption is **not** high-confidence evidence merely because multiple advisors repeated it.

### 5. Owner-led technical discussion — required

The council must **discuss the decision**; a collection of independent opinions followed by a summary is insufficient.

1. The EDO identifies the **one to three most consequential disagreements** (design, operational failure, migration, verification, or cost).
2. For each disagreement, state the alternatives, the competing advisor arguments, the evidence needed to resolve it, and the EDO's direct challenge/question.
3. Ask the relevant advisors to respond to those objections **using their existing agent conversations if supported**. Each response should either rebut with evidence, concede, or propose a discriminating test. Allow one concise rebuttal round; additional rounds only if the user requests them or a material safety/correctness issue remains.
4. The EDO replies explicitly: **accepted / rejected / unresolved**, with rationale and impact on the recommendation. Change the provisional position when new evidence warrants it.

If conversational follow-up is unavailable, conduct a transparent evidence-based comparison of the disputed claims and label it as an EDO assessment, **not a fabricated live debate**. Never flatten an unresolved disagreement into artificial consensus.

### 6. Engineering Decision Owner recommendation

The EDO synthesizes **after** the discussion and weighs decisions in this order:

1. Hard constraints, correctness, security, policy, and irreversible risks.
2. Evidence relative to explicitly prioritized requirements and quality attributes.
3. Migration feasibility, operability, rollback, and verification.
4. Lower-priority preferences and optimization opportunities.

If evidence is insufficient for a consequential irreversible choice, recommend a **time-boxed experiment or spike** with measurable pass/fail criteria, rather than manufacturing certainty. Escalate decisions requiring security, legal, compliance, or product approval; never silently grant it.

Use this output shape (omit irrelevant detail, but retain accountability and dissent):

```markdown
## Engineering Council Verdict: [decision]

**Decision owner:** [role/name if known] · **Status:** Proposed / Approved by [explicitly verified human] / Needs validation

### Decision and rationale
[Recommended option; why it wins against the stated criteria; confidence and uncertainty.]

### Options and trade-offs
| Option | Benefits | Costs and risks | Verdict |
| --- | --- | --- | --- |

### Technical discussion
- **Dispute:** [competing claims and supporting evidence]
- **Owner challenge:** [specific objection/question]
- **Responses:** [substantive rebuttals or labeled assessment]
- **Owner resolution:** Accepted / rejected / unresolved because ...

### Consensus, dissent, and blind spots
[What converged independently; the strongest remaining objection; what changed during review.]

### Validation and safe delivery
[Tests, benchmarks, migration, observability, rollback, measurable acceptance criteria.]

### Decision record
- Context and scope: ...
- Decision / alternatives rejected: ...
- Consequences and revisit trigger: ...
- Required human approvals: ...

### First action
[One concrete next step, with owner if known.]
```

Do not overproduce a table for a tiny decision. For a substantial architecture choice, the **Decision record** can be copied into an ADR; write a repository ADR only if the user asks for a file/change or authorizes recording the decision.

### 7. Present and save only as authorized

Show the verdict in chat, concise enough to scan with evidence and disagreements visible. Do not automatically write files, scripts, tickets, commits, or HTML reports. If asked to implement an accepted option, treat implementation and tests as a separate phase and follow normal repository constraints.

## Quality gate

Before presenting, check all:

- [ ] One EDO is identified; real approval authority is not fabricated.
- [ ] The decision criteria, constraints, options, facts, and unknowns are explicit.
- [ ] Five independent lenses were considered; orchestration limits are disclosed.
- [ ] Anonymous peer review distinguishes evidence from repetition.
- [ ] The EDO actively challenged and answered substantive disagreements.
- [ ] A recommendation or measurable validation plan is given—not a majority vote.
- [ ] Strongest dissent, risk, rollback, verification, and human sign-off are visible.
- [ ] No invented repo inspection, tests, benchmarks, execution, or approval.

## Example trigger

> Engineering council: Should we switch our existing synchronous order processing to an event-driven architecture? We need idempotency, a gradual migration, and zero lost orders. Have the Software Architect own the decision and debate it with the council.

The EDO might provisionally prefer a transactional outbox, then challenge the Implementation Lead's migration concerns and the Reliability Engineer's duplicate-delivery failure case. The verdict should state **why** the chosen architecture handles replay, ordering, observability, and rollback—not merely count votes.
