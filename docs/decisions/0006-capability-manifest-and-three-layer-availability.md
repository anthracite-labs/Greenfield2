# ADR-0006: Model the provider boundary as a declared capability manifest with three-layer availability

**Date:** 2026-09-13
**Status:** proposed
<!-- Becomes `accepted` when PR #11 is merged after independent review.
     .ecc/skills/decisions.md: "Status is honest. `proposed` until it is
     actually in force." An ADR on an unmerged branch is not in force, and
     nothing may rely on it as settled until it is. -->
**Deciders:** Greenfield2 Architecture (Issue #9), under the sequencing rule in ADR-0005

## Context

[ADR-0005](0005-provider-contract-before-provider-selection.md) fixed the
*order* — contract before provider — but deliberately left the contract's
*structural shape* undecided. Issue #9 required that shape to be chosen and then
stress-tested against at least three materially different provider shapes.

That validation was performed against four documented surfaces (Google Jules,
Cursor Cloud Agents, Replit MCP, GitHub Copilot) and is recorded in
[../architecture/PROVIDER_SHAPE_VALIDATION.md](../architecture/PROVIDER_SHAPE_VALIDATION.md).
The measured result drove this decision: on **no** axis — work-item topology,
identifier style, auth mechanism, capability discovery, entitlement discovery,
approval mechanism, progress telemetry, change inspection, result kind, or update
delivery — is any property common to all four. Three of four have a run-like
unit and one has none. Three of four have no capability-discovery endpoint. Two
of four expose no approval mechanism, and the two that do expose different kinds
of approval. All four use different auth mechanisms. Cursor changed its own
identifier format and artifact-path semantics between `v0` and `v1`; Jules is
`v1alpha`; the Copilot agent-tasks API is public preview behind feature headers.

Every apparent universal in the first draft turned out to be a majority case,
The draft required ten revisions (F1–F10) to stop asserting them, and a further five (F11–F15) after independent review checked the contract clause by clause against accepted product and domain truth.

## Decision

Greenfield2's provider boundary is a **declared capability manifest over opaque
provider-native resource references**, resolved through **three
independently-sourced availability layers**, and bounded by — never narrowing —
the accepted product requirements it exists to realize.

Concretely, the contract at
[../architecture/PROVIDER_CAPABILITY_CONTRACT.md](../architecture/PROVIDER_CAPABILITY_CONTRACT.md) v1.1:

1. Names capabilities as **provider-neutral questions** (`work.start`,
   `progress.observe`, `approval.respond`, …), never as provider resources.
2. Makes the **capability manifest** the contract's root: an adapter declares
   what its provider exposes, and absence is declared rather than inferred.
3. Keeps **Provider Connection** and **Provider Resource Reference** as two
   distinct types, per [DOMAIN.md](../DOMAIN.md). Resource references are
   opaque: `ref` is never parsed, `kind` is the provider's own noun and never a
   branch condition in Greenfield2 core, and `providerState` passes through
   verbatim. A connection, by contrast, legitimately carries a
   Greenfield2-owned lifecycle.
4. Separates availability into **L1 provider capability**, **L2 account
   entitlement** and **L3 Greenfield2 UI support**, each with its own owner and
   source, with L2 tri-state (`entitled | not-entitled | unknown`) and L3
   recorded per provider capability rather than globally. `unknown` has defined
   behaviour that differs for read-only and mutating capabilities.
5. Lets adapters **declare the provider's required invocation inputs**, so
   mandatory provider-native context can be expressed instead of being lost
   behind a provider-neutral prompt.
6. Treats **`Absent` as a first-class, non-exceptional answer**, and bans
   transport vocabulary above the adapter boundary.
7. Preserves provider error **semantics** unaltered while **sanitizing** its
   presentation, because provider free text is untrusted input.
8. Relaxes **no** accepted requirement in [PRODUCT.md](../PRODUCT.md); where the
   two disagree, PRODUCT.md wins.

## Alternatives considered

### Alternative: Normalize provider resources into one Greenfield2-owned resource model

- **Pros:** Simple internal types; uniform UI logic; one place to add cross-provider features later.
- **Cons:** Requires a single work-item topology, one state vocabulary, one result type and one change model. Validation shows none exists: Replit exposes no work item at all, Cursor needs two levels, Replit's result is a published app rather than a pull request, and two of four providers expose no change inspection.
- **Why not:** It would force Replit's App and Cursor's Agent/Run into one shape, and would need a canonical result enum that silently ranks providers by resemblance to whichever provider suggested the list. This is also the alternative Discovery already rejected, and ADR-0005 rejected again.

### Alternative: Contract only over the intersection of what all providers share

- **Pros:** Smallest contract; trivially cross-provider consistent; no per-provider branching.
- **Cons:** The measured intersection here is close to empty — no axis is common to all four. What remains is "connect an account and send a prompt", which cannot deliver the MVP development loop in [PRODUCT.md](../PRODUCT.md).
- **Why not:** It produces the lowest-common-denominator product ADR-0005 explicitly rejected, and hides exactly the richer provider capabilities (Jules' inline patches, Cursor's resumable event stream) that Greenfield2 is allowed to expose faithfully.

### Alternative: Capability flags on a fixed provider resource model (hybrid)

- **Pros:** Keeps familiar resource types for the majority case while allowing providers to opt out of individual features; less unfamiliar than a pure manifest.
- **Cons:** The fixed resource model still has to exist, so the hard cases are still forced: Replit must be given a synthetic work item, and Cursor must collapse two levels into one or be special-cased. Capability flags then decorate a shape that already encodes a vendor assumption.
- **Why not:** It relocates the leakage rather than removing it. The special cases end up in core, which is precisely where vendor-specific branching is prohibited (contract P8).

### Alternative: Defer the contract's shape until the first provider is chosen

- **Pros:** Concrete API constraints known immediately; no risk of designing against documentation that later changes.
- **Cons:** This is the ordering ADR-0005 rejected: the first vendor's nouns, lifecycle, transport and limits become Greenfield2's architecture by accident, and later providers pay to undo them.
- **Why not:** Already decided against in ADR-0005. Re-listed here because it remains the path of least resistance and will be proposed again.

## Consequences

### Positive

- No single vendor defines Greenfield2's provider boundary; provider selection becomes a test of an existing contract.
- Provider-native terminology, lifecycle and state stay provider-owned, satisfying the [DOMAIN.md](../DOMAIN.md) ownership rule structurally rather than by convention.
- Capability, entitlement and UI support are separately sourced, so the three distinct user remedies — "not on this provider", "not for your account", "open in provider" — are expressible instead of collapsing into one "unavailable".
- A provider can be added without core changes: it declares a manifest and answers capabilities.
- Transport choices stay behind adapters, so REST, MCP, streaming, webhooks and polling can coexist per provider.
- Provider drift is an expected input: opaque references, pass-through state tokens and a mandatory `unknown` mean a changed enum or identifier format degrades rather than breaks.
- Rich provider-specific capability can be exposed without a universal taxonomy.

### Negative

- Adapters carry more translation logic, because Greenfield2 will not normalize provider concepts for them.
- UI logic must handle `Absent` and `unknown` everywhere, which is more work than a uniform model and is easy to get subtly wrong.
- Cross-provider features are harder: anything spanning providers must be expressed in capability terms, which rules out shortcuts through a shared resource type.
- The presentation liveness hint is a standing leakage risk. It is constrained to four values with a mandatory `unknown`, presentation-only, non-authoritative and never persisted as provider state — but it is the place a shadow state machine would start, and it needs review on every change.
- The contract is validated against documentation, not live integration, so some assumptions will still need revision against real behaviour.
- Capability identifiers are a new Greenfield2-owned vocabulary that must be governed; an identifier added casually is as much a leak as a provider noun.

### Follow-ups

- First-provider selection is now **unblocked** and is already tracked by the existing **[Issue #8 — Architecture: evaluate first MVP provider](https://github.com/anthracite-labs/Greenfield2/issues/8)**, which carries the Product Fit and Integration Legitimacy gates. Selection is deliberately **not** made here or in Issue #9, and no duplicate issue should be opened.
- Re-verify the validation set's provider surfaces before relying on any specific endpoint in that selection; the profiles are dated 2026-09-13 and every surface is pre-stable.
- Define the adapter interface in code once an application-stack ADR exists. Until then `ALLOW_APP_STACK=0` stands.
- Establish governance for adding capability identifiers, so the catalogue grows deliberately rather than per integration.
- Revisit whether a common work-item abstraction is genuine or accidental only when more provider shapes are available — not by assumption from four.
- Consider a fifth validation shape covering multi-repository work or self-hosted execution before the contract is treated as settled.
- **Refer to the product owner:** whether observing a running or published artifact (`live-artifact`) can on its own satisfy minimum V1 item 4, *"inspect meaningful provider-exposed changed files and/or diffs"*. This is a product interpretation, not an Architecture call. Until it is answered the safe reading applies — `live-artifact` alone does not satisfy item 4 (contract §5.5.1, §11).
- Mark this ADR `accepted` and update the index only once PR #11 is merged after independent review.
