# Domain

**Status:** Minimal Greenfield2-owned product vocabulary accepted during Discovery review in Issue #4.  
**Lifecycle:** `PROJECT_PHASE=architecture`; the accepted vocabulary remains a product guardrail while Architecture decides implementation details.

Greenfield2 intentionally owns a very small product domain in MVP. Provider-native development resources remain provider-owned and provider-authoritative.

This file defines only the vocabulary Greenfield2 needs to describe its own interface boundary without silently promoting provider concepts into Greenfield2 authority.

## Core domain rule

> **A provider-owned resource does not become a Greenfield2-owned entity merely because Greenfield2 can display, reference, search, navigate to, or operate it through a supported provider interface.**

Greenfield2 may hold opaque references and interface metadata. The provider remains authoritative for the referenced resource's identity, lifecycle, content, capability, entitlement and state.

## Accepted Greenfield2-owned entities / concepts

| Term | Definition | Identity / authority | Invariants |
| :-- | :-- | :-- | :-- |
| **Greenfield2 Account** | The user's identity for the Greenfield2 interface. | Greenfield2-owned identity. | Does not mean provider account; does not own provider resources. |
| **Provider Connection** | The authorized relationship/reference between a Greenfield2 Account and one supported provider account. | Greenfield2 owns connection metadata; provider owns the external account and its resources. | Authentication does not imply every capability, entitlement or permission; connection must use a supported integration path. |
| **Capability Support** | Greenfield2 metadata stating whether a provider-exposed capability has a validated Greenfield2 UI/surface. | Greenfield2-owned support metadata about a provider capability. | Does not redefine or create the provider capability; provider capability/entitlement truth remains provider-side. |
| **Greenfield2 UI Preference** | Presentation state that belongs purely to Greenfield2, such as Greenfield2-level theme, accessibility or layout preferences. | Greenfield2-owned presentation state. | Must not be used to shadow provider-native persistent settings or modes. |
| **Provider Resource Reference** | An opaque reference Greenfield2 may retain to reopen or navigate to a provider-owned resource. | Greenfield2 owns only the reference; provider owns the referenced resource. | Reference != ownership; stale references must never be presented as current provider truth. |

## Provider-native terms are not Greenfield2-owned entities

Terms such as the following may appear in provider UIs, research, adapters, events or documentation:

- App / Project;
- Repository;
- Agent / Agent Session;
- Model;
- Workspace / runtime;
- Session / conversation;
- Task / run / execution;
- file / diff;
- approval / policy;
- preview;
- artifact;
- deployment / release;
- evidence / verification;
- quota / entitlement / usage.

The accepted Discovery vocabulary does **not** establish Greenfield2-owned canonical entities for those concepts.

When a provider exposes one of them, Greenfield2 preserves the provider's own terminology and semantics unless a future reviewed Greenfield2 decision establishes a genuinely distinct Greenfield2 concept.

Examples:

- a provider `App` remains that provider's App;
- a provider `Session` remains that provider's Session;
- a provider `Cloud Agent` remains that provider's Cloud Agent;
- a provider `Workspace` remains provider-owned runtime/execution state;
- a provider deployment remains provider deployment state;
- provider completion or verification remains provider truth in MVP.

## Ubiquitous language

| Term | Means | Does NOT mean |
| :-- | :-- | :-- |
| **Provider** | An external software-development service whose supported capabilities Greenfield2 may expose to an authorized user. | A Greenfield2-owned runtime or subcontracted execution layer. |
| **Provider-native** | Defined, named and semantically owned by the external provider. | A Greenfield2-normalized entity merely rendered with provider branding. |
| **Provider-authoritative** | The provider is the source of truth for the resource/capability/state. | Greenfield2 may not display or operate it. |
| **Supported external-client path** | A provider-supported interface that permits a third-party client to access the required capability. | Scraping, reverse-engineered private endpoints, or first-party-only UI automation. |
| **Entitlement legitimacy** | The connected user's account/plan/API entitlement is legitimately usable through the selected supported path. | Mere successful authentication. |
| **Validated Greenfield2 surface** | A Greenfield2 UI intentionally built and verified for a real provider capability. | Generic UI that guesses unsupported semantics. |
| **Pass-through action** | A user action Greenfield2 forwards to the provider through the provider's supported interface. | A Greenfield2-owned approval, execution or policy decision. |
| **Provider truth only** | Greenfield2 presents the provider's own autonomy, durability, recovery, completion and verification semantics in MVP. | Independent Greenfield2 execution/recovery/verification authority. |
| **Mobile-first** | Phone is the priority full product surface for meaningful software-building work. | Mobile-only or notification-only. |
| **Universal software-development interface** | One coherent Greenfield2 UI over supported provider-owned development capabilities. | One universal Greenfield2 resource model that erases provider differences. |

## Conceptual distinctions to preserve

Unless a later reviewed Greenfield2 decision explicitly changes them:

- **Project != Repository != Workspace**
- **Agent != Model != Workspace provider**
- **Task != Execution != Verification**
- **Greenfield2 Account != provider account**
- **Provider Connection != provider account**
- **Provider Resource Reference != provider resource**
- **Capability discovery != capability ownership**
- **Capability != entitlement**
- **Authentication != authorization**
- **Authorization != entitlement**
- **Entitlement != Greenfield2 permission**
- **Model API != Agent-session API**
- **Provider-hosted workspace != remote control of a user's local machine**
- **Unified presentation != normalized provider semantics**
- **Provider completion != Greenfield2 independent verification**
- **Reconnect/replay != Greenfield2 execution recovery**

These distinctions are product/architecture guardrails; they do not imply that Greenfield2 owns every named concept.

## Business rules

| Rule | Rationale |
| :-- | :-- |
| Provider capabilities determine what provider development surfaces can exist in Greenfield2. | Prevents fabricated behavior and accidental platform ownership. |
| Greenfield2 may unify presentation but preserves provider-native terminology and semantics. | Keeps one coherent UX without misleading the user. |
| Provider Resource References are opaque and non-authoritative. | Prevents a reference/cache from becoming a shadow source of truth. |
| Provider actions are pass-through in MVP. | Keeps approval and execution authority with the provider. |
| Provider-derived content is retained only when permitted and necessary. | Minimizes privacy/security surface and avoids a shadow content store. |
| Production capability support requires a supported external-client path and legitimate entitlement. | Technical accessibility alone is insufficient. |
| A missing Greenfield2 UI does not justify inventing generic provider behavior. | Use native-provider handoff instead. |

## State semantics

Greenfield2 does not define a Greenfield2 state machine for provider resources.

Provider resource lifecycle and states remain provider-native. Greenfield2 may maintain its own account/connection/UI lifecycle metadata as needed. The accepted provider capability contract defines the interface-level boundary; persistence design, the code-level adapter interface and provider-specific transport choices remain Architecture work.

## Reference-source boundary

- **VibeFlow** supplies prior interoperability/product hypotheses to challenge and reuse only when explicitly accepted.
- **Replit** supplies clean-room behavioral/product evidence and conceptual distinctions; it is not the domain model.
- **Happier** supplies architecture/feasibility reference patterns; its account/session/relay/daemon/persistence semantics are not Greenfield2 domain authority.

Research vocabulary remains evidence until a reviewed Greenfield2 decision promotes a concept.

## Related

- [`../FOUNDATIONS.md`](../FOUNDATIONS.md) — product constitution and evidence hierarchy
- [PRODUCT.md](PRODUCT.md) — accepted product definition
- [ROADMAP.md](ROADMAP.md) — lifecycle sequencing
- [ARCHITECTURE.md](ARCHITECTURE.md) — active application Architecture decisions and open questions
- [decisions/](decisions/README.md) — durable technical decisions