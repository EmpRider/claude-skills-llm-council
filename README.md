# LLM Council — Software Engineering

An engineering-focused Claude skill for making difficult technical decisions through a structured, evidence-based discussion.

Based on the independent-advisor and anonymous peer-review approach in [Andrej Karpathy's LLM Council](https://github.com/karpathy/llm-council). This fork adapts the workflow to software engineering decisions, with a **Software Architect, Principal/Staff Engineer, or owning Tech Lead** acting as the accountable **Engineering Decision Owner (EDO)**.

## What changed

The original council had five broad business thinking styles and a chairman who summarized the result. This version makes technical decision ownership and real discussion central:

1. **Repository-aware evidence:** read relevant architecture, code, tests, constraints, and existing decisions when available.
2. **Five technical perspectives:** systems architecture, implementation, reliability/security, quality/verification, and a skeptical alternative designer.
3. **An active owner:** the engineering lead takes a provisional position, questions conflicting arguments, hears rebuttals, and explicitly resolves or records dissent.
4. **Anonymous peer review:** proposals are criticized on their evidence, rather than the perceived authority of their author.
5. **Decision-ready result:** a recommended option, trade-offs, tests, rollout/rollback, remaining risks, and an ADR-style record.
6. **Codebase-first issue fixing:** the council inspects affected code and tests, verifies the root cause, and prefers the smallest durable change unless severity and evidence justify a broader one.

The AI provides a **proposed** decision. It never claims to have secured a real engineer's approval or a required Security/SRE/compliance sign-off.

## Who leads?

| Decision type | Default owner |
| --- | --- |
| Cross-service or architectural changes | Software Architect / Principal Engineer |
| Component or service design | Owning Tech Lead / Senior Engineer |
| Implementation and delivery strategy | Engineering Lead |
| Security or production risk exception | Engineering owner, with required specialist approval |
| Product scope and budget | Appropriate product/business owner; engineering provides technical advice |

If your team has a designated human decision owner, include their role in the prompt. The skill can facilitate the discussion but cannot sign off on their behalf.

## When to use

Useful for questions with meaningful trade-offs:

- Should our service use a modular monolith or microservices?
- Should we adopt Kafka or keep synchronous APIs?
- Should we migrate a production database now or incrementally?
- Which caching design meets our consistency and latency requirements?
- Should we refactor a critical module or isolate it behind an adapter?
- Is this authentication architecture safe and maintainable?
- Which permanent fix for this production bug has the smallest justified blast radius?

Skip the council for straightforward bug fixes, routine coding, factual lookups, and decisions already made.

## Install

The skill instructions live in [SKILL.md](./SKILL.md). Copy that file into the `llm-council` skill directory of your supported Claude Code or Claude Cowork setup, following the skill installation instructions for your environment. The skill name and existing `council this` trigger are retained.

## Use

Try:

```text
engineering council: Should we use a transactional outbox or direct event publishing
for order events? We cannot lose events, and migration must be incremental.
Have the Software Architect own the decision and discuss the risks with all advisors.
```

```text
pressure-test this architecture decision: REST vs gRPC between our internal services.
The owning Tech Lead should make a recommendation based on our existing code,
latency requirements, deployability, and test strategy.
```

```text
run the council on this database migration plan. Identify what evidence is missing,
debate rollback and data consistency, then give me an ADR-style proposed decision.
```

```text
bug-fix review: investigate the duplicate processing issue in the current codebase.
Trace the affected code paths, root cause, callers, and tests first. Compare a
minimal-impact permanent fix with a broader structural fix. Account for severity,
regression risk, safe rollout, and rollback. Have the owning Tech Lead challenge
the options and recommend the smallest safe permanent fix.
```

Other triggers: `council this`, `run the council`, `architecture council`, `design review`, `war room this`, `stress-test this`, `debate this`.

## Issue-fix policy

**Review the current codebase before selecting a fix.** The council must inspect the relevant implementation, reproduction or failing behavior, dependencies, consumers, and tests. If it cannot access the codebase, it must say so and request the necessary evidence rather than inventing a permanent fix.

The default is **the smallest change that permanently resolves the verified root cause**—not simply the patch with the fewest lines. If a structural problem requires a high-impact permanent change, the council weighs severity against regression, migration, and operational risk and plans a staged rollout.

| Severity | What takes priority |
| --- | --- |
| P0 Critical | Restore safety quickly; reversible mitigation may come first, with permanent remediation tracked separately. |
| P1 High | Remove serious risk promptly; use rollout safeguards for unavoidable high-impact changes. |
| P2 Medium | Prefer the narrow, verified permanent solution. |
| P3 Low | Avoid disproportionate change risk; plan invasive remediation rather than rushing it. |

Neither change impact nor permanence is silently ignored. Any compromise is documented with its reason, residual risk, responsible owner, and follow-up. A workaround is never labeled a completed permanent fix.

## Decision workflow

```mermaid
flowchart TD
    A["Evidence and decision brief"] --> B["Owner's provisional position"]
    B --> C["Five independent advisors"]
    C --> D["Anonymous peer review"]
    D --> E["Owner-led challenges and rebuttals"]
    E --> F["Recommendation and decision record"]
    F --> G["Human approval and implementation"]
```

The owner must resolve or preserve the **actual disagreements**. A simple vote or generic synthesis does not complete the workflow. When sub-agents are not available, the skill discloses that its perspectives are a single-agent analysis rather than claiming independent reviewers.

## Expected output

- Proposed decision and accountable owner role.
- Option comparison grounded in constraints and available evidence.
- Technical arguments, rebuttals, and how the owner resolved them.
- Remaining dissent and risks.
- Acceptance tests, performance/security validation where relevant, delivery and rollback plan.
- Concise ADR-style record and the first action.

## Origins

Originally shared as [Ole Lehmann's Claude LLM Council skill](https://github.com/aiwithremy/claude-skills-llm-council), inspired by [Karpathy's LLM Council](https://github.com/karpathy/llm-council). This repository maintains an engineering-specific adaptation.
