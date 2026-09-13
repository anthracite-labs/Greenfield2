# Product

**Status:** Product-owner Discovery definition accepted for review in Issue #4.  
**Lifecycle:** `PROJECT_PHASE=discovery` remains authoritative until a separate reviewed phase transition.

Greenfield2 is App 1, treated as a fresh product. This document records the product definition reached through Discovery. It does not select an application stack, provider implementation, protocol, database, auth scheme, hosting target, or UI framework.

## Product definition

Greenfield2 is a **mobile-first universal software-development interface** for technical builders.

It connects to a user's supported software-development provider account, discovers the capabilities and resources that provider officially exposes to that user, and presents those capabilities through one coherent Greenfield2 interface.

The defining authority rule is:

> **Greenfield2 owns no provider development state. The connected provider remains authoritative for every provider-owned resource, capability, runtime, session, file, artifact, action, lifecycle, entitlement, error, approval and result that Greenfield2 exposes.**

Greenfield2 may own only the minimum state that belongs to Greenfield2 itself: Greenfield2 account/session identity, provider-connection metadata, Greenfield2-only UI preferences, security/reconnect metadata, capability-support metadata, and opaque references needed to reopen provider-owned resources. A reference is never ownership.

## Primary user and job to be done

The primary MVP user is a **technical builder**: a developer, indie hacker, founder, or technical product builder who already uses an eligible agentic software-development provider.

The primary job to be done is:

> **Let me build and continue real software wherever I am, through one coherent interface over the development provider I already use.**

The user's provider-native client is the baseline experience Greenfield2 must complement and, for supported workflows, improve through coherent cross-surface access.

## Problem and positioning

Software-development providers increasingly expose capable Agents, cloud workspaces, repositories, files, diffs, approvals, previews, deployments and other development surfaces, but those capabilities are fragmented by provider and constrained by each provider's own client surfaces.

Greenfield2 solves the **interface/access problem**, not the provider-execution problem:

- the provider already owns the capability and state;
- Greenfield2 makes the provider's supported capabilities usable through one coherent, adaptive, mobile-first interface;
- Greenfield2 does not recreate, take ownership of, or independently reinterpret provider development state.

### Category

> **Universal software-development interface**

Short form:

> **Your provider. Its capabilities and state. One Greenfield2 interface.**

Greenfield2 should feel complete enough to support a real software-building loop, but Replit is a completeness/interaction benchmark rather than Greenfield2's category or identity.

Greenfield2 is not:

- a vertically integrated development runtime;
- a master AI;
- an Agent that chooses the user's stack;
- a generic provider dashboard;
- merely a chat wrapper;
- a Replit clone.

## Reference inputs and decision authority

VibeFlow, Replit and Happier are deliberately combined as reference points, but none of them silently defines Greenfield2.

- **VibeFlow** informs the interoperability/provider-neutral product thesis and the importance of capability discovery, supported external-client paths and entitlement legitimacy. Greenfield2 does not inherit VibeFlow's older control-plane, Task/Execution/Verification, policy, durability or independent-verification authority unless a future reviewed Greenfield2 decision explicitly adopts it.
- **Replit research** is clean-room product/behavior evidence for what a coherent mobile software-building experience can expose: Agent interaction, plans/progress, files/diffs, background work, approvals, preview, repository/Git surfaces, logs and deployment-related behavior. It does not define Greenfield2's domain model, implementation or ownership boundary.
- **Happier** is architecture/feasibility reference material for multi-provider capability catalogs, cross-device supervision, reconnect behavior and rich development surfaces. Greenfield2 does not inherit Happier's account, relay, daemon, session, persistence or orchestration authority.

The product owner and reviewed Greenfield2 changes decide what Greenfield2 is. Research challenges and validates decisions; it does not promote itself into requirements.

## Mobile promise

Greenfield2 is **mobile-first, not mobile-only**.

The phone is the priority product surface and must support meaningful software-building work rather than being only a notification or remote-control companion. Greenfield2 should present one adaptive product experience across phone and larger screens.

Discovery does not decide whether those surfaces are implemented as native apps, web, hybrid shells, or another technical approach. That belongs to Architecture.

## MVP product boundary

### Bring-your-own-provider

The MVP requires the user to bring an eligible provider account. The user contracts with and pays the underlying provider directly.

Greenfield2 does not provision, bundle, resell or fund provider subscriptions, compute, credits, model usage, Agent usage or workspace/runtime usage in MVP.

### One provider for the first slice

The first MVP ships one thoroughly validated full-stack provider integration.

MVP behavior is:

> **One Greenfield2 account → one connected provider → one provider-authoritative development experience.**

Multiple providers and multiple accounts for the same provider are post-MVP.

The first provider is an MVP proving slice, not Greenfield2's permanent definition. Discovery does not select that provider.

### Minimum V1 capability and output set

For a provider to qualify for the first Greenfield2 MVP, it must expose enough supported capability for Greenfield2 to present a workspace-level development loop.

The user must be able to:

1. start new provider work or continue existing provider work where the provider exposes it;
2. interact with the provider's Agent;
3. observe provider-reported progress, plan/tool/activity information where exposed;
4. inspect meaningful provider-exposed changed files and/or diffs;
5. respond to provider approval requests where exposed;
6. see a meaningful provider result such as a preview, build, deployment, pull request, or provider-equivalent outcome;
7. reconnect to or continue the provider-owned work where the provider supports continuity.

These are capability requirements on the first provider, not declarations that Greenfield2 owns Agents, Workspaces, files, diffs, approvals, previews, builds, deployments or pull requests.

A provider may expose more. Greenfield2 may expose additional provider-specific capabilities when it has a validated UI for them.

## North-star journey

The first coherent journey is:

1. user creates or signs into a Greenfield2 account;
2. Greenfield2 briefly explains the Greenfield2/provider authority boundary;
3. user connects one supported provider account through an official/supported authorization path;
4. Greenfield2 begins with provider-recommended minimal permissions and requests additional authority only when a user-invoked capability requires it;
5. Greenfield2 discovers the account, entitlements, capabilities and resources actually exposed by the provider;
6. Greenfield2 renders provider-native resources, terminology, states, errors and actions through one coherent adaptive UI;
7. user starts new work or continues existing work where the provider exposes it;
8. user interacts with provider-owned Agent/workspace behavior and causes at least one meaningful real change;
9. Greenfield2 shows the resulting provider state and meaningful output;
10. if a capability exists but Greenfield2 has no validated UI for it, Greenfield2 hands off to the provider-native interface rather than fabricating behavior.

MVP success requires this real end-to-end loop; merely connecting an account is insufficient.

## Adaptive interface principles

### Provider capabilities determine what exists

The connected provider determines which resources and capabilities exist. Greenfield2 must not invent missing concepts for visual completeness.

A provider `App` remains an App. A provider `Session` remains a Session. A provider `Cloud Agent` remains a Cloud Agent. If a provider has no Project concept, Greenfield2 does not fabricate one.

### Greenfield2 determines presentation

Greenfield2 may unify navigation, typography, layout, interaction patterns, responsive behavior, loading/error presentation, progressive disclosure, accessibility and contextual provider attribution.

Greenfield2 does not achieve coherence by semantically renaming provider concepts into a Greenfield2-owned universal model.

### Native-provider escape hatch

Where Greenfield2 has a validated UI for a provider capability, the user should be able to operate it inside Greenfield2. Where useful, Greenfield2 should also provide an **Open in provider** path.

If Greenfield2 has no validated UI for a capability, native-provider handoff is preferred over guessed behavior.

## Authority, actions and trust

### Provider-authoritative state

Depending on what a provider exposes, provider-authoritative state may include Apps/Projects, repositories, Agent sessions, conversations/messages, Workspaces/runtimes, Tasks/runs, execution state, files, diffs, tool/terminal/log activity, previews, artifacts, deployments/releases, approvals, provider-native settings, provider history/lifecycle, quotas/usage, capabilities, entitlements, errors/degraded state, background work and provider completion/verification information.

Greenfield2 displays and operates these only through supported provider interfaces.

### Mutating actions are pass-through

For actions such as modifying files, deploying/publishing, pushing repository changes, creating/deleting provider resources, spending provider credits, approving/denying Agent/tool requests, or pausing/cancelling/resuming provider work, Greenfield2 forwards the user's action through the provider's supported interface.

Greenfield2 does not create an independent approval/policy authority layered above the provider.

### Agent autonomy, durability, recovery and verification

The MVP promise is **provider truth only**.

Greenfield2 exposes the provider's own Agent behavior, approval model, background execution, reconnect/resume, recovery, completion state and provider-generated evidence/verification information where supported.

Greenfield2 does not create its own execution authority, background-worker truth, recovery engine, independent completion authority or independent verification layer in MVP.

If the provider does not support durability, reconnect, recovery or verification, Greenfield2 does not fabricate it.

## Authentication, permissions, entitlement and integration legitimacy

Authentication is not capability authority. A successful provider sign-in does not automatically prove that Greenfield2 may access provider Agent sessions, compute, model usage, subscription credits, files, history, billing or other provider resources.

Greenfield2 must discover and respect the actual authorized capabilities and entitlements.

Start with the provider's recommended minimal/default authorization scope. Request additional authority only when a user invokes a capability that genuinely requires it, through the provider's supported authorization flow.

A provider capability qualifies for production Greenfield2 support only when both are true:

1. **Supported external-client path** — the provider officially supports an interface suitable for the capability Greenfield2 needs.
2. **Entitlement legitimacy** — the user's account/plan/API entitlement is legitimately usable through that path.

If either is missing, the capability is not production-supported.

Greenfield2 must not rely on scraped private first-party interfaces, reverse-engineered private session endpoints, entitlement bypass, pooled/shared consumer accounts, or impersonation outside authorized scopes.

## Errors and changing capabilities

Provider errors, limits, quota messages, entitlement failures, degraded states and unavailable capabilities remain provider truth. Greenfield2 may format them for readability and safe presentation but does not reinterpret them into a separate Greenfield2 execution state machine.

Capabilities and entitlements may change. Greenfield2 must reflect current provider truth rather than treating an initial capability snapshot as permanent.

## Data, privacy, search and caching

Greenfield2 follows a **minimum-data rule**.

Provider-derived content may be retained only when:

1. the provider permits that retention; and
2. a real Greenfield2 feature requires it.

Any permitted cache must use minimum necessary data, bounded retention, clear staleness handling and predictable cleanup when the provider is disconnected or the data is no longer required.

Greenfield2 does not create a central provider-content mirror by default and should not automatically retain copies of provider conversations, files, diffs, artifacts or history.

Search over provider resources should use provider-supported search/list/filter capabilities where available. Minimal opaque references for recents, navigation or reopening provider resources are allowed and do not make those resources Greenfield2-owned.

## Fixed constraints

- **Platform:** mobile-first adaptive experience across phone and larger screens; implementation technology is deferred.
- **Commercial:** bring-your-own-provider only for MVP; provider costs remain between the user and provider.
- **Geography/compliance:** global product intent, but an integration is enabled only where supported auth, entitlement, data handling, provider terms and applicable legal/compliance requirements permit it.
- **Data:** minimum-data, explicit necessity, provider permission and bounded retention.
- **Integration:** supported external-client path plus entitlement legitimacy are mandatory.
- **Time/budget:** no fixed delivery deadline or hard budget ceiling was imposed during Discovery; Architecture should expose realistic cost/effort before trade-offs are made.
- **Lifecycle:** `ALLOW_APP_STACK=0` remains in force until the normal reviewed Architecture/implementation transitions.

## Explicit MVP non-goals

Greenfield2 MVP does **not**:

1. provide its own development runtime or execution environment;
2. own provider Apps, Sessions, Projects, Workspaces, Tasks, files, artifacts, deployments or history;
3. create an independent execution, recovery, approval or verification state machine;
4. fabricate capabilities the provider does not expose;
5. normalize provider-native concepts into a fake universal domain model;
6. use unsupported/private provider interfaces as production dependencies;
7. support multi-provider composition in MVP;
8. support multiple accounts for the same provider in MVP;
9. provision or resell provider services in MVP;
10. maintain a central mirror/search index of provider content;
11. require replacement of every feature in the provider's native application;
12. choose the application stack, database, framework, hosting target or provider architecture during Discovery.

## Success measures

MVP success is observed using both product value and interface reliability:

- provider connection success;
- successful first meaningful end-to-end software-building workflow;
- time to first meaningful change;
- successful reconnect/continue behavior where supported by the provider;
- repeat Greenfield2 usage;
- reliability of forwarded provider actions;
- reliability and fidelity of provider state/error presentation.

Discovery selects what should be observed; it does not invent numerical targets before implementation and real usage provide a basis for them.

## First-provider qualification

Discovery does not select the first provider.

After this product definition is accepted, candidate providers should be evaluated against two gates:

1. **Product fit:** the provider exposes enough supported capability to deliver the minimum V1 development loop.
2. **Integration legitimacy:** the provider exposes a supported external-client interface and a legitimate entitlement/auth path for the required capabilities.

Commercial attractiveness or popularity may be considered only after those gates.

## Post-MVP direction

Greenfield2 may later support composition of user-selected capabilities such as BYOK (model/API capability), BYOA (Agent capability), BYOW (workspace/execution capability), repository/source-control integration, multiple providers, multiple accounts and federated provider search.

If Greenfield2 later owns composition/wiring configuration, that does not automatically make it authoritative for the provider-owned things being wired.

This is direction, not MVP scope and not Architecture.

## Discovery exit

This product definition answers the Discovery questions required by `FOUNDATIONS.md` and `docs/ROADMAP.md` at the product-owner level. The repository remains in `PROJECT_PHASE=discovery` until this definition is independently reviewed and a separate reviewed lifecycle transition is made.

Architecture must answer **how** without redefining the accepted **what, for whom and why**.

## Related

- [`../FOUNDATIONS.md`](../FOUNDATIONS.md) — Discovery constitution and evidence hierarchy
- [DOMAIN.md](DOMAIN.md) — accepted minimal product vocabulary and ownership semantics
- [ROADMAP.md](ROADMAP.md) — lifecycle sequencing
- [ARCHITECTURE.md](ARCHITECTURE.md) — engineering foundation; application architecture later
- [MEMORY.md](MEMORY.md) — append-only verified project memory
- [decisions/](decisions/README.md) — ADR structure for later durable technical choices
