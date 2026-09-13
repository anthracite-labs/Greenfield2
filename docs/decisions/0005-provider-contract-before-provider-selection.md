# ADR-0005: Define the provider capability contract before selecting the first provider

**Date:** 2026-09-13
**Status:** accepted
**Deciders:** Greenfield2 product owner + Architecture review

## Context

Greenfield2's reviewed product definition requires provider development state to remain provider-authoritative while Greenfield2 presents supported provider capabilities through one coherent interface. Issue #9 initially evaluated concrete providers first, which exposed a sequencing risk: choosing the first provider before defining the provider boundary could cause that vendor's nouns, lifecycle, API shape, transport, or limitations to become Greenfield2 architecture by accident.

The current provider research is intentionally heterogeneous. Jules exposes structured Sessions, Activities and Artifacts; Cursor exposes Agent/Run and richer realtime events; Replit exposes a vertically integrated app/workspace product through a comparatively thin third-party MCP surface; GitHub Copilot provides a different OAuth/entitlement and PR-centric shape. VibeFlow, Replit and Happier remain evidence/reference inputs rather than decision authority.

## Decision

Greenfield2 will define and validate a **provider-neutral capability contract before selecting the first MVP provider**. The contract is an interface/capability boundary, not a Greenfield2 domain model: provider-native resources remain opaque/referenced and provider-authoritative, while capability availability, user entitlement and Greenfield2 UI support remain distinct concerns.

Before first-provider selection, the contract must be stress-tested against at least three materially different provider shapes and revised where that validation reveals vendor leakage, false universality, or lowest-common-denominator loss. Only after that validation may Greenfield2 select the first provider using the accepted Product Fit + Integration Legitimacy gates.

## Alternatives considered

### Alternative: Select the first provider, then design its adapter and generalize later

- **Pros:** Fastest route to a working vertical slice; concrete API constraints are known immediately.
- **Cons:** The first provider's `Session`, `Run`, `Project`, `Workspace`, event model, transport and limitations can silently become Greenfield2's abstractions.
- **Why not:** This conflicts with the product goal of a universal software-development interface and makes later providers pay the cost of undoing first-vendor assumptions.

### Alternative: Define one normalized universal provider resource model up front

- **Pros:** Simple internal types and apparently uniform UI logic.
- **Cons:** Forces unlike provider concepts into Greenfield2-owned semantics, risks collapsing important provider-native distinctions, and can create shadow state or false equivalence.
- **Why not:** Discovery explicitly rejected a fake universal provider domain model and requires provider-native terminology, lifecycle and state to remain authoritative.

### Alternative: Use only the intersection of capabilities every provider shares

- **Pros:** Small contract and easy cross-provider consistency.
- **Cons:** Produces a lowest-common-denominator product and hides richer provider capabilities that Greenfield2 is explicitly allowed to expose faithfully.
- **Why not:** Greenfield2's adaptive UI is capability-driven; provider-specific capabilities may remain provider-specific when Greenfield2 has validated support for them.

## Consequences

### Positive

- No single vendor gets to define Greenfield2's core provider boundary.
- Provider selection becomes a test of an existing Greenfield2 contract rather than the source of that contract.
- Provider-native semantics can remain intact while Greenfield2 still has a coherent capability-oriented interface boundary.
- Transport choices such as REST, MCP, SDK, SSE, WebSocket, webhook or polling stay behind provider adapters.
- The contract can distinguish provider capability, connected-account entitlement and Greenfield2 UI support without conflating them.
- Rich provider-specific capabilities can be preserved without forcing them into a universal resource taxonomy.

### Negative

- Architecture takes longer before the first implementation provider is selected.
- The contract must be tested against multiple live provider surfaces and may require revision before implementation begins.
- Some apparent common abstractions will deliberately remain unresolved until evidence proves they are genuinely provider-neutral.
- Provider adapters may need more translation logic because Greenfield2 will not normalize every provider concept into a shared owned entity.

### Follow-ups

- Define the first Architecture-level provider capability contract under Issue #9.
- Validate that contract against at least three materially different provider shapes; the current stress-test set is Jules, Cursor and Replit, with GitHub Copilot as a useful fourth case.
- Record any contract revisions caused by validation and confirm that no provider noun/lifecycle became universal Greenfield2 semantics.
- Only after validation, open or continue the first-provider selection decision using Product Fit + Integration Legitimacy.
- Keep `ALLOW_APP_STACK=0`; this ADR does not choose an application stack or authorize implementation.
