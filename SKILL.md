---
name: llm-council
description: "An engineering decision council for architecture, issue fixing, design, reliability, security, testing, and technical trade-offs. An accountable Engineering Decision Owner leads five independent advisors, challenges objections, and records an evidence-based recommendation. Triggers: 'council this', 'run the council', 'engineering council', 'architecture council', 'design review', 'bug-fix review', 'war room this', 'pressure-test this', 'stress-test this', 'debate this'. Use for consequential engineering choices, not simple factual questions or implementing an already-settled choice."
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
- Choosing between low-impact and high-impact permanent fixes for a reported defect.

Do **not** convene for factual lookups, routine bug fixes with an established solution (unless the user explicitly requests council review), simple formatting, code generation, or tasks that merely require executing an accepted decision. **If invoked for issue fixing, use the issue-fixing protocol below even if the likely fix appears obvious.** If the user explicitly asks to implement, work on implementation according to the user's instructions; do not substitute a council report for code.

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

## Issue-fixing protocol — codebase first

When the council evaluates a defect, regression, outage, data-integrity issue, or security issue, **do not choose a fix from the issue report alone**. The codebase and observed behavior are the starting point. The EDO must enforce this sequence **before** advisor recommendations:

1. **Inspect the current implementation.** Trace the failing entry point through relevant callers, dependencies, configs, data contracts, error handling, and tests. Check relevant recent changes or prior fixes when available. Respect repository instructions and access boundaries. Keep investigation scoped to affected code and consumers; do not read unrelated secrets.
2. **Establish the failure and root cause.** Gather reproduction steps, expected vs actual behavior, logs, failing tests, and code-path evidence. Distinguish a proven root cause from a hypothesis or symptom. If the codebase is unavailable, explicitly mark the assessment **unverified; no permanent fix selected** and request access or the minimum necessary files. For an active incident, a clearly provisional, reversible safety mitigation may still be recommended while evidence is collected. If the issue cannot be reproduced, define the diagnostic evidence needed before confidently choosing a permanent fix.
3. **Map the change surface.** Identify affected modules, public APIs, callers, persisted data, compatibility, existing tests, operational dependencies, and rollback constraints. Consider whether the defect is local or caused by a shared architectural invariant.
4. **Develop alternatives.** Compare at least (a) the smallest plausible permanent root-cause fix and (b) a broader permanent fix, if credible. For active incidents also assess a reversible mitigation. Do not inflate the option count when only one credible permanent design exists.
5. **Require verification.** For each candidate state how a regression test or reproduction proves the issue is fixed, what adjacent behavior could break, and what monitoring or rollback would detect and contain failure.

### Severity vs impact decision rule

**Default:** choose the **minimal-impact permanent solution that fixes the verified root cause**. "Minimal impact" means least necessary blast radius and regression/migration/operational risk, not simply fewest changed lines. A tiny patch that hides the symptom, breaks an invariant, or leaves the root cause is **not permanent**.

Never silently ignore either permanence or change impact. When they conflict, the EDO must make the trade-off explicit and record the deferred concern, justification, risk, owner, and follow-up. Use severity as a decision input, **not permission to skip evidence, tests, or safety checks**:

| Severity | Priority | Council behavior |
| --- | --- | --- |
| **P0 / Critical** — severe outage, data loss, or exploitable critical security risk | Protect users/data and restore safe service fast | Prefer reversible containment when a broad change is too risky during the incident. Require an owned, tracked permanent root-cause fix after stabilization. A low-risk permanent fix can be deployed immediately when evidence supports it. |
| **P1 / High** — major degradation or serious risk | Remove serious risk promptly without causing a larger incident | Prefer a permanent fix; if high impact is unavoidable, add staged rollout, compatibility safeguards, focused tests, monitoring, and rollback. Temporary mitigation is allowed only with an explicit permanent-fix commitment. |
| **P2 / Medium** — bounded defect with a viable workaround | Smallest proven durable correction | Choose the narrow permanent fix when available. Defer broad refactoring unless the shared root cause makes it necessary; validate before escalating the blast radius. |
| **P3 / Low** — minor or cosmetic issue | Avoid disproportionate disruption | Prefer a small permanent fix. If only an invasive permanent fix exists, recommend scheduling it with appropriate tests rather than rushing it. |

Use the team's own incident severity definitions if available; these labels are defaults, not an invented organizational SLA. Never use severity alone to overrule correctness, data integrity, security controls, or required approval. Production mitigation and permanent remediation are **separate statuses**.

### Issue-fix decision checklist

Compare credible candidates on:

- **Root-cause coverage:** eliminate the underlying failure and related paths, or just conceal the symptom?
- **Change impact:** affected code, contracts, persisted data, consumers, and blast radius.
- **Regression risk:** invariants, existing features, edge cases, and possible unintended behavior.
- **Urgency and reversibility:** time to deliver safely; staging, rollback, and mitigations.
- **Verification:** reproduction, automated regression test, benchmark, security check, or rollout signal.
- **Residual risk:** what remains unfixed, and who owns the follow-up?

Use qualitative impact ratings (**low / medium / high**) with reasons unless actual measurements exist. Do not invent probabilities, effort estimates, test results, or evidence. The EDO must challenge an advisor recommending a broader change without showing why a narrow permanent fix is insufficient, **and** challenge a narrow fix leaving systemic defects in place. When a high-impact permanent change is necessary, propose a risk-managed migration rather than rejecting it solely for its size.

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

For issue-fix councils, include the **verified or hypothesized root cause, code paths examined, severity basis, affected consumers, reproduction, and tests** from the issue-fixing protocol. Do not select a **permanent** fix until this evidence gate is satisfied; explicitly pause that decision if codebase access is missing. An emergency containment discussion may proceed with limitations disclosed.

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

### Issue-fix assessment (when applicable)
- Root cause and codebase evidence: ...
- Severity and affected behavior: ...
- Smallest permanent fix vs broader permanent fix: ...
- Blast radius and regression risk: ...
- Mitigation status vs permanent fix status: ...
- Trade-off deliberately deprioritized, reason, owner, and follow-up: ...

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
- [ ] For issue fixes: current code paths were reviewed, root-cause confidence is explicit, and impact was assessed before selecting a fix.
- [ ] For issue fixes: severity justifies the trade-off; temporary mitigation is not mislabeled as a permanent resolution.

## Example trigger

> Engineering council: Should we switch our existing synchronous order processing to an event-driven architecture? We need idempotency, a gradual migration, and zero lost orders. Have the Software Architect own the decision and debate it with the council.

The EDO might provisionally prefer a transactional outbox, then challenge the Implementation Lead's migration concerns and the Reliability Engineer's duplicate-delivery failure case. The verdict should state **why** the chosen architecture handles replay, ordering, observability, and rollback—not merely count votes.

> Bug-fix review: Our API sometimes returns duplicate payment records after retries. Inspect the current handlers, idempotency logic, persistence constraints, and tests. Confirm the root cause, compare the smallest permanent fix against any schema-level correction, estimate blast radius, and have the Tech Lead discuss severity versus migration risk before choosing.
