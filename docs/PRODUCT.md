# Product

**Status: Discovery in progress. No reviewed product definition has been accepted yet.**

Greenfield2 is App 1, treated as a fresh product. The repository has now moved
from the generic App-Factory `factory` phase into `discovery`, but the product
must still earn its definition through product-owner answers and reviewed
changes.

## Discovery mandate

The purpose of Discovery is to decide what Greenfield2 is, for whom, why it
should exist, what the first coherent version contains, and what it explicitly
does not attempt.

Prior work is input, not inherited truth:

- **VibeFlow** is the target/baseplate to examine with fresh eyes. Its
  mobile-first, provider-neutral software-development control-plane thesis,
  BYOA/BYOK/BYOW direction, authority model, durability, recovery, evidence and
  verification concepts are high-value hypotheses rather than automatically
  accepted App 1 requirements.
- **Replit research** is clean-room behavioral/reference evidence for proven
  product interactions, mobile/Workspace UX and capability coverage. It does
  not define Greenfield2.
- **OSS research** is advisory until a later Architecture decision promotes a
  standard, dependency, wrapper, bridge or harvested component.

See [`../FOUNDATIONS.md`](../FOUNDATIONS.md) for the full evidence hierarchy and
promotion path.

## Questions the product owner must answer

These are the authoritative Discovery questions. Answers should land through
GitHub issues and reviewed changes rather than being inferred from prior code or
research.

| Question | Why it blocks everything else |
| :-- | :-- |
| What problem is being solved, for whom? | Without it, scope is unbounded. |
| What category should Greenfield2 create or occupy? | Prevents us from accidentally reducing it to an IDE, app builder or Agent wrapper. |
| Who is the primary user and what do they use today instead? | Defines the baseline to beat. |
| What role does mobile play in the product promise? | Mobile is a major hypothesis and affects the whole interaction model. |
| What does provider neutrality mean to the user? | BYOA/BYOK/BYOW can be a product promise, an advanced capability, or merely architecture; Discovery must decide. |
| What does Greenfield2 itself own authoritatively versus broker from providers? | Defines the durable product boundary. |
| What work may Agents perform autonomously and what requires approval? | Defines trust, policy and UX. |
| What durability, recovery and verification promise does the product make? | Determines whether remote Agent work is merely convenient or actually dependable. |
| What artifact/output types matter in the first usable version? | Prevents the broad VibeFlow artifact vision from making V1 unbounded. |
| What is explicitly out of scope? | Non-goals prevent drift more than goals do. |
| What does the first coherent, usable version do end-to-end? | Sets the smallest shippable slice. |
| How is success observed? | Determines what has to be measurable. |
| What data does the product touch and retain? | Drives security, privacy and trust requirements. |
| What constraints are fixed (regulatory, budget, deadline, platform, commercial)? | Cannot be discovered late cheaply. |

## Current accepted product facts

Only the following have been explicitly established for Discovery so far:

1. Greenfield2 is a **fresh App 1**, not a continuation that inherits every old
   VibeFlow decision.
2. VibeFlow is the **target/baseplate** and strongest prior product reference;
   it is to be scrutinized and improved rather than copied mechanically.
3. Replit is **reference/benchmark evidence**, not the product target.
4. The existing ECC/App-Factory engineering system is not the product and must
   remain intact while product discovery proceeds.
5. The product is expected to explore the VibeFlow direction of a mobile-first,
   provider-neutral software-development control plane, including the
   BYOA/BYOK/BYOW thesis, but those details still require Discovery validation.

Everything beyond those facts remains open until explicitly decided.

## Discovery exit criteria

This file is ready to support a move to `PROJECT_PHASE=architecture` only when
it contains, in reviewed form:

- primary user(s) and jobs-to-be-done;
- product problem and positioning;
- product promise and north-star journeys;
- accepted V1 scope;
- explicit non-goals;
- major trust/privacy/security requirements;
- success measures;
- enough ownership boundaries that Architecture can evaluate technical
  alternatives without redefining the product.

Technical consequences such as framework, database, hosting, auth, Agent,
workflow engine or workspace provider are recorded later as ADRs, not implied
by this document.

## Guardrails in force during Discovery

- `ALLOW_APP_STACK=0` remains active.
- No application source code or stack-specific artifact is introduced.
- No OSS candidate is vendored/forked merely because it appears promising.
- No VibeFlow or Replit behavior is silently promoted to a requirement.
- Product changes must stay outside the protected ECC/App-Factory core unless a
  separate foundation issue explicitly authorizes that work.

## Related

- [`../FOUNDATIONS.md`](../FOUNDATIONS.md) — project discovery constitution
- [DOMAIN.md](DOMAIN.md) — product vocabulary as it becomes accepted
- [ROADMAP.md](ROADMAP.md) — repository lifecycle sequencing
- [ARCHITECTURE.md](ARCHITECTURE.md) — engineering foundation; application architecture later
- [FACTORY.md](FACTORY.md) — App-Factory instantiation/governance reference
- [decisions/](decisions/README.md) — ADR structure for later durable choices
