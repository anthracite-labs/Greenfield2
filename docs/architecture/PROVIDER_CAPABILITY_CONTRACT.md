# Provider Capability Contract

**Version:** 1.1 (validated; revised after independent review — see [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) §4b findings F11–F15 and §4c consistency corrections C1–C5)
**Status:** **Accepted.** The product owner accepted this contract's direction together with [ADR-0006](../decisions/0006-capability-manifest-and-three-layer-availability.md) (2026-09-13). It is Architecture authority. It is not an interface implementation, not a schema file, and not an SDK.
**Lifecycle:** `PROJECT_PHASE=architecture`, `ALLOW_APP_STACK=0`. This document selects no transport, framework, database, auth implementation, hosting target, UI technology or provider.
**Authority:** [ADR-0005](../decisions/0005-provider-contract-before-provider-selection.md) (accepted) requires this contract to exist and be validated before any first-provider selection. [ADR-0006](../decisions/0006-capability-manifest-and-three-layer-availability.md) (accepted) records its structural shape.
**Validation:** see [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md). v1.1 is the accepted, post-validation form; the revision log there records what earlier drafts got wrong.

---

## 1. What this contract is, and what it is not

Greenfield2 is a mobile-first universal software-development interface. The
connected provider owns the development state. This contract is the boundary
between the two.

**It is** a declaration of the *capabilities and operations* Greenfield2 may
discover, observe and invoke through a provider adapter, plus the rules that
keep provider semantics provider-owned while doing so.

**It is not:**

| Not | Why |
| :-- | :-- |
| A Greenfield2 domain model | [DOMAIN.md](../DOMAIN.md) fixes Greenfield2-owned vocabulary at five concepts. This contract adds none. |
| A provider resource model | Provider resources stay provider-native and provider-authoritative. |
| A transport specification | REST, MCP, SDK, SSE, WebSocket, webhook and polling are adapter-local implementation details. |
| A normalized state machine | Provider lifecycle stays provider-owned. |
| A permission model | Authentication, authorization and entitlement are provider truths this contract reads, never issues. |
| An interface definition in code | No stack exists yet; `ALLOW_APP_STACK=0`. Notation below is deliberately language-neutral. |

### The load-bearing sentence

> A provider capability is something the provider can do. A Greenfield2 capability
> identifier is a *question Greenfield2 knows how to ask*. This contract defines the
> questions, never the answers' shape.

An adapter answers a question in provider-native terms, or reports that the
question does not apply to its provider. Both are valid. A fabricated answer is
not.

---

## 2. Notation

Language-neutral, because no stack is selected.

```text
capability.operation(input, …) -> Result | Absent | ProviderRefusal
```

- `Result` — a provider-native payload inside the envelope defined in §4.4.
- `Absent` — the provider does not expose this capability. Distinguished from a
  failure. `Absent` is a legal, expected, permanent-until-the-provider-changes
  outcome, and must never be retried as if it were transient.
- `ProviderRefusal` — the provider declined: not entitled, out of quota, policy,
  invalid input, or a provider-side error. Carries the provider's error
  **semantics unaltered** — code, category, retryability and provider-stated
  remedy — in a **sanitized presentation**. It does not carry provider text
  verbatim; see §8.1 and §8.2.

`required` / `optional` on a capability describes the **MVP loop role**, not a
demand that every provider supply it. See §5.

---

## 3. The three-layer availability model

Three independent predicates decide whether a capability is usable. They have
different owners, different sources and different rates of change, and the UI
must be able to tell them apart.

| Layer | Question | Owner | Source of truth | Volatility |
| :-- | :-- | :-- | :-- | :-- |
| **L1 — Provider capability** | Does this provider expose the capability through a *supported external-client path*? | Provider | Provider documentation + adapter probe | Low–medium: changes with provider API versions |
| **L2 — Account entitlement** | Is *this* connected account legitimately authorized for it *right now*? | Provider | Provider entitlement probe at connection and on demand | High: plan change, quota exhaustion, revocation, repository or workspace policy |
| **L3 — Greenfield2 UI support** | Does Greenfield2 have a validated surface for it? | Greenfield2 | Greenfield2 build metadata ([DOMAIN.md](../DOMAIN.md) *Capability Support*) | Release-driven |

```text
L2 is three-valued, so availability is NOT a boolean conjunction. L2 = unknown
is neither true nor false, and must never be collapsed into either.

resolution(capability, account) =
    not-offered              if  ¬L1
    unavailable-for-account  if  L1 ∧  L2 = not-entitled
    unconfirmed              if  L1 ∧  L2 = unknown     ∧  L3   # behaviour per §3.2.1
    handoff                  if  L1 ∧  L2 = entitled    ∧ ¬L3
    handoff (unconfirmed)    if  L1 ∧  L2 = unknown     ∧ ¬L3
    operable                 if  L1 ∧  L2 = entitled    ∧  L3
```

Rows are evaluated in order; the function is total over L1 × L2 × L3 and matches
the disposition table in §3.1 exactly.

Only `operable` means the capability is straightforwardly usable in Greenfield2.
`unconfirmed` is a distinct outcome, not a degraded `operable`: under §3.2.1 a
read-only capability may proceed, while a mutating or spend-bearing capability
may not proceed on the strength of an `unknown` L2 without an explicit,
warned user invocation.

### 3.1 Why the layers must not be collapsed

Each failure mode has a **different remedy for the user**, so a single
"unavailable" flag would make the interface lie by omission:

| L1 | L2 | L3 | Disposition | Rationale |
| :-- | :-- | :-- | :-- | :-- |
| ✗ | – | – | **Not offered.** Hidden or shown as "not available on this provider". | [PRODUCT.md](../PRODUCT.md): Greenfield2 must not invent missing concepts for visual completeness. |
| ✓ | not-entitled | – | **Explained as not available for this account**, with the provider's own path forward. | The capability exists; the user's remedy is with the provider, not with Greenfield2. |
| ✓ | **unknown** | ✓ | **Offered as unconfirmed.** See §3.2.1 — behaviour depends on whether the capability is read-only or mutating. | Greenfield2 must neither fabricate a denial nor silently assume entitlement. |
| ✓ | entitled | ✗ | **Native-provider handoff.** "Open in provider". | [PRODUCT.md](../PRODUCT.md): native-provider handoff is preferred over guessed behavior. |
| ✓ | entitled | ✓ | **Operable in Greenfield2**, with native handoff still offered. | The full case. |
| ✓ | **unknown** | ✗ | **Native-provider handoff**, labelled unconfirmed. | L3 false decides the surface; L2 uncertainty must still be visible. |

### 3.2 L2 is tri-state, not boolean

```text
entitlement ∈ { entitled, not-entitled, unknown }
```

`unknown` is a first-class value, not a default for `false`. Validation found
providers with **no entitlement endpoint at all**, where the only probe is an
attempt that either succeeds or is refused (§ Validation F6). Rendering
`unknown` as `not-entitled` would tell users they lack something they may have;
rendering it as `entitled` would make Greenfield2 assert a provider fact it does
not hold.

Rules:

- `unknown` must be surfaced as uncertainty, never as denial and never as
  confirmation.
- L2 is **never cached as permanent**. [PRODUCT.md](../PRODUCT.md): capabilities
  and entitlements may change; Greenfield2 must reflect current provider truth.
  A cached L2 carries an explicit observation time and a bounded lifetime.
- A `not-entitled` result is **not permanent either**. Quota refills, plan
  changes and policy changes are provider events; Greenfield2 re-probes rather
  than remembering a refusal.
- Authentication success is **not** evidence for L2. ([DOMAIN.md](../DOMAIN.md):
  *Authentication != authorization*, *Authorization != entitlement*.)
- Greenfield2 requests the provider's recommended minimum authority first and
  escalates only when a user invokes a capability that requires more
  (`connection.scope.escalate`).

#### 3.2.1 What Greenfield2 does when L2 is `unknown`

`unknown` was previously declared but not defined, which left every adapter to
invent its own behaviour. It is defined here, and the definition turns on
whether the capability merely **reads** or can **mutate or spend**.

**Read-only capabilities** (`work.list`, `work.inspect`, `progress.observe`,
`changes.inspect`, `result.observe`, `usage.observe`, `discovery.*`):

- Offered normally, with the entitlement shown as **unconfirmed** rather than
  hidden or greyed out.
- May be invoked optimistically. If the provider refuses, that refusal is
  provider truth and is surfaced under §8 — Greenfield2 does not pre-empt it.
- A refusal updates L2 to `not-entitled` for that capability, with the
  observation time recorded.

**Mutating or spend-bearing capabilities** (`work.start`, `work.continue`,
`agent.interact`, `approval.respond`, `work.control`,
`connection.scope.escalate`):

- Never invoked speculatively, and never auto-retried after a refusal. These
  can create provider resources, consume credits or cause real changes, and
  [PRODUCT.md](../PRODUCT.md) makes such actions user-initiated pass-throughs.
- The unconfirmed entitlement must be **visible before the user commits**, not
  revealed by a failure afterwards.
- Each attempt requires a fresh, explicit user invocation.

In both cases:

- Greenfield2 **must not** synthesize an entitlement verdict it does not have.
  Guessing `entitled` is a fabricated provider fact; guessing `not-entitled`
  hides a capability the user may hold.
- The provider's own refusal text governs what the user is told (§8). Greenfield2
  does not substitute its own guess at the reason.

#### 3.2.2 Entitlement evidence kinds

`discovery.entitlements` returns evidence, not verdicts alone, so the UI can say
how sure it is. Each result carries one of:

| Evidence kind | Meaning | Typical resulting L2 |
| :-- | :-- | :-- |
| `provider-endpoint` | The provider exposes a plan/entitlement/policy query. | `entitled` or `not-entitled` |
| `authorization-scope` | Derived from the scopes actually granted at authorization time. | `entitled` for in-scope capabilities |
| `enumeration-scope` | Inferred from what a list call returns — e.g. a provider that silently lists only resources the caller may edit. | `entitled` for what appeared; **`unknown`** for what did not |
| `attempt-outcome` | Known only by attempting: success implies entitled, refusal implies not. | `entitled` / `not-entitled` after the fact |
| `none` | No probe is available at all. | **`unknown`**, and stays `unknown` until an attempt |

`enumeration-scope` deserves care: absence from a list is **not** evidence of
absence of entitlement, because the provider may simply be filtering. Treating a
filtered list as a denial is the same error as rendering `unknown` as
`not-entitled`.

### 3.3 L3 is Greenfield2's own claim, and it is falsifiable

L3 is the only layer Greenfield2 owns. It must be recorded per provider
capability, not globally: a capability Greenfield2 has a validated surface for on
one provider is not thereby supported on another, because the fulfilment differs
(§6). L3 must never be inferred from L1 or L2.

---

## 4. Two distinct reference types — Connection and Resource Reference

Greenfield2 carries two kinds of reference. [DOMAIN.md](../DOMAIN.md) already
defines **both** as separate Greenfield2-owned concepts with different
definitions, different authority and different invariants. They are not one
thing, and this contract must not collapse them.

### 4.1 Provider Connection

[DOMAIN.md](../DOMAIN.md): *"The authorized relationship/reference between a
Greenfield2 Account and one supported provider account."* Greenfield2 owns
**connection metadata**; the provider owns the external account and everything
in it.

```text
ProviderConnection {
  connectionId      : string   # Greenfield2-owned. Identifies the CONNECTION, never a provider resource.
  providerId        : string   # Greenfield2's identifier for the provider.
  providerAccountId : string?  # The provider's own account identifier, opaque, as the provider states it.
  authorizedScopes  : string[]?# The authority actually granted, exactly as the provider reports it.
  authorizedAt      : instant
  state             : connected | revoked | expired | unknown
}
```

Invariants, taken from DOMAIN.md:

- Authentication does not imply every capability, entitlement or permission.
- The connection must use a **supported integration path**.
- `Provider Connection != provider account`, and it is **not** a resource reference.
- MVP is one Greenfield2 account → **one** connected provider.
- Revocation triggers bounded cache cleanup under the minimum-data rule.

`state` here is legitimately Greenfield2-owned. It describes Greenfield2's *own*
connection lifecycle, which DOMAIN.md explicitly permits ("Greenfield2 may
maintain its own account/connection/UI lifecycle metadata"). That is a different
thing from provider **resource** state, which is not Greenfield2's to model.

### 4.2 Provider Resource Reference

[DOMAIN.md](../DOMAIN.md): *"An opaque reference Greenfield2 may retain to
reopen or navigate to a provider-owned resource."* Greenfield2 owns **only the
reference**.

```text
ProviderResourceReference {
  connectionId   : string   # Which ProviderConnection this resource was reached through.
  providerId     : string
  kind           : string   # The provider's OWN noun, verbatim ("Session", "Agent", "Run",
                            # "App", "Task", …). Display vocabulary, never a discriminator.
  ref            : string   # Opaque, provider-chosen. Greenfield2 MUST NOT parse, split,
                            # compose, prefix-match or validate its structure.
  nativeUrl      : string?  # Provider-native deep link, for handoff.
  providerState  : string?  # The provider's own state token, verbatim, uninterpreted.
  observedAt     : instant  # When Greenfield2 last saw it. Staleness tracking, NOT provider truth.
}
```

Non-negotiables:

1. **`ref` is opaque.** Validation found the same provider change its own
   identifier format between API versions (Validation F8). Anything Greenfield2
   derives from `ref`'s structure breaks silently.
2. **`kind` is never a branch condition in Greenfield2 core.** Adapters may
   branch on their own provider's kinds; core logic may not. A `switch` on
   `kind` in core is vendor leakage by construction.
3. **`providerState` passes through verbatim.** Greenfield2 does not define a
   provider state enum, does not map provider states onto each other, and must
   tolerate state values it has never seen (§4.5).
4. **Reference != ownership.** A stale reference must never be presented as
   current provider truth.

### 4.3 Why they must stay separate

| | Provider Connection | Provider Resource Reference |
| :-- | :-- | :-- |
| Refers to | the authorization relationship to a provider account | a provider-owned resource |
| Greenfield2 owns | connection metadata | only the reference |
| Lifecycle owner | **Greenfield2** (connect / revoke / expire) | **the provider** |
| State | Greenfield2-owned connection state is legitimate | provider-owned; passes through uninterpreted |
| MVP cardinality | one | many |
| Cleanup | on revoke or disconnect | bounded retention; never a content mirror |

Collapsing them fails in both directions. Treating a connection as a resource
reference would strip it of the Greenfield2-owned lifecycle and revocation
cleanup it needs. Treating a resource reference as a connection would give
Greenfield2 a lifecycle over provider-owned resources — which is precisely the
shadow-state ownership DOMAIN.md prohibits.

### 4.4 The envelope

Every capability result is provider-native content inside a Greenfield2-owned
envelope. The envelope carries *provenance and freshness*; the payload carries
*meaning*, and meaning stays with the provider.

```text
Envelope {
  resource    : ProviderResourceReference
  payload     : provider-native, uninterpreted by Greenfield2 core
  payloadKind : string            # the provider's own name for the payload shape
  observedAt  : instant
  staleness   : fresh | stale | unknown
  attribution : string            # provider display name, for contextual attribution
}
```

Greenfield2 may unify *presentation* of envelopes — layout, typography, loading
and error treatment, progressive disclosure, accessibility, attribution. It may
not achieve coherence by *renaming* what is inside them.

### 4.5 Presentation liveness hint — a controlled exception

Mobile layout needs to know whether to show an in-progress affordance or a
"needs your input" affordance. Deriving that from `providerState` is
presentation, which [PRODUCT.md](../PRODUCT.md) assigns to Greenfield2 — but it
is also exactly where a shadow state machine would start. It is therefore
permitted only under all of these conditions:

- The hint has **four** values: `active | awaiting-user | settled | unknown`.
- `unknown` is **mandatory** and is the required mapping for any provider state
  the adapter does not explicitly recognize. An unrecognized state must never be
  coerced into a recognized one.
- The hint is **lossy and presentation-only**. It is never persisted as provider
  state, never used to decide whether a pass-through action is permitted, never
  shown as the resource's status, and never used to infer completion or
  verification.
- The provider's own state token remains available alongside it, unmodified.
- Provider completion and verification remain provider truth. The hint must
  never imply that Greenfield2 has verified anything.

Anything beyond these four values is a state machine and is out of scope for
this contract.

---

## 5. Capability catalogue

Capability identifiers are **Greenfield2-owned question names**. They are
provider-neutral by construction: none is a provider noun, and none implies a
provider resource topology.

MVP loop role comes from the minimum V1 capability set in
[PRODUCT.md](../PRODUCT.md). That set is an **accepted product requirement**, and
this contract does not relax it. PRODUCT.md states the qualifiers precisely, so
they are carried precisely:

| PRODUCT.md minimum V1 item | Qualifier in PRODUCT.md | Contract capability | Consequence for a first provider |
| :-- | :-- | :-- | :-- |
| 1. start new / continue existing provider work | *where the provider exposes it* | `work.start`, `work.continue` | At least one of the two is required |
| 2. interact with the provider's Agent | **none** | `agent.interact` | **Required** |
| 3. observe progress / plan / tool / activity | *where exposed* | `progress.observe` | Required when exposed |
| 4. inspect meaningful changed files and/or diffs | **none** | `changes.inspect` | **Required** — see §5.5.1 |
| 5. respond to provider approval requests | *where exposed* | `approval.discover`, `approval.respond` | Required when exposed |
| 6. see a meaningful provider result | **none** | `result.observe` | **Required** |
| 7. reconnect / continue provider-owned work | *where the provider supports continuity* | `continuity.reconnect` | Required when supported |

`optional` marks richness beyond that set, which Greenfield2 may expose where it
has a validated surface. An `optional` capability is never a substitute for an
unqualified item above.

This table is a restatement of accepted product requirements, not a new one. If
it and [PRODUCT.md](../PRODUCT.md) ever disagree, PRODUCT.md wins and this table
is wrong.

### 5.1 Connection and authorization

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `connection.authorize` | loop prerequisite | `(user) -> ProviderConnection \| ProviderRefusal` | Returns a **Provider Connection** (§4.1), not a resource reference — it establishes the authorized relationship to a provider account. Must use a provider-supported authorization path. Greenfield2 begins with the provider's recommended minimum authority. |
| `connection.revoke` | loop prerequisite | `(connection) -> Result` | Triggers bounded cache cleanup per the minimum-data rule. |
| `connection.identity` | loop prerequisite | `(connection) -> Envelope` | Provider account identity as the provider states it. The envelope's `resource` points at the provider account, not at a connection. |
| `connection.scope.escalate` | optional | `(connection, requiredFor) -> ProviderConnection \| ProviderRefusal` | Invoked only when a user action genuinely requires more authority. Returns the updated connection so `authorizedScopes` reflects provider truth. |

Every capability below that is scoped to an account takes a **Provider
Connection**; every capability scoped to a piece of provider-owned work takes a
**Provider Resource Reference**. The two are not interchangeable (§4.3).

### 5.2 Discovery

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `discovery.capabilities` | loop prerequisite | `(connection) -> CapabilityManifest` | Adapter-produced (§9). There is no universal provider endpoint for this; how it is derived is adapter-local. |
| `discovery.resource-kinds` | optional | `(connection) -> string[]` | The provider's own nouns, verbatim. |
| `discovery.entitlements` | loop prerequisite | `(connection) -> EntitlementEvidence[]` | Tri-state per §3.2. Carries the *evidence kind* the adapter used, so the UI can say how sure it is. |
| `discovery.resources` | optional | `(connection, kind, page?) -> Envelope[]` | Enumerate provider-owned resources of a provider-native kind. Providers with no enumerable work concept may lawfully return `Absent`. |

### 5.3 Work

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `work.start` | loop | `(context : InvocationContext) -> Envelope \| Absent \| ProviderRefusal` | Begin new provider work. `context` carries the provider's **declared required inputs** (§5.9) — a provider-neutral instruction alone is not sufficient for every provider. |
| `work.continue` | loop | `(context : InvocationContext) -> Envelope \| Absent \| ProviderRefusal` | Continue existing provider work where the provider exposes continuity; `context.target` names it. |
| `work.list` | loop | `(connection, page?) -> Envelope[] \| Absent` | `Absent` is legal: a provider whose unit of work is a long-lived resource rather than a run has nothing to enumerate. |
| `work.inspect` | loop | `(resource) -> Envelope` | |
| `work.control` | optional | `(resource, verb) -> Envelope \| Absent \| ProviderRefusal` | `verb` is provider-native (`pause`, `cancel`, `resume`, `archive`, …). Greenfield2 defines no universal verb set. |

### 5.4 Agent interaction

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `agent.interact` | loop | `(resource, input) -> Envelope \| Absent` | Forward user input to the provider's Agent. A pass-through: Greenfield2 adds no interpretation, no approval, no policy. |

### 5.5 Observation

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `progress.observe` | loop | `(resource, cursor?) -> ProgressUpdate[] \| Absent` | Progress, plan, tool and activity telemetry as the provider reports it. Delivery mode declared per §6. |
| `changes.inspect` | loop | `(resource) -> ChangeView \| Absent` | See §5.5.1 — fulfilment varies categorically. |
| `result.observe` | loop | `(resource) -> Envelope[] \| Absent` | The provider's own meaningful outcome. Typed provider-natively (§5.5.2). |
| `usage.observe` | optional | `(connection \| resource) -> Envelope \| Absent` | Quota, limits, token or credit usage where the provider exposes it. |

#### 5.5.1 `changes.inspect` fulfilment kinds

Validation established that change inspection is **not** one thing
(Validation F2). The contract therefore declares a fulfilment kind rather than
assuming a diff:

```text
ChangeView.kind ∈ { inline-patch, remote-reference, live-artifact, none }
```

| Kind | Meaning | Satisfies minimum V1 item 4? |
| :-- | :-- | :-- |
| `inline-patch` | The provider hands Greenfield2 the change content directly. | **Yes** |
| `remote-reference` | The provider exposes the change only as a reference to somewhere it owns. | **Yes** — the provider exposes changed files/diffs; whether Greenfield2 renders them inline or hands off is an L3 question |
| `live-artifact` | The change is observable only as running/published output. | **Not by itself** — published output is not changed files or diffs. See the note below |
| `none` | The provider exposes no change-inspection surface. | **No** |

**`none` does not qualify a provider for the MVP loop.** PRODUCT.md's minimum V1
item 4 — *"inspect meaningful provider-exposed changed files and/or diffs"* —
carries **no** "where exposed" qualifier, unlike items 1, 3, 5 and 7. A provider
whose `changes.inspect` fulfilment is `none` therefore fails item 4 and fails
Product Fit (Gate 1) for first-provider selection under Issue #8. An earlier draft of this
contract said such a provider "may still qualify … if the rest of the loop is
strong". That was wrong: it relaxed an accepted product requirement, and the
contract has no authority to do so. The declaration `none` remains a legal and
honest *capability statement* — it is L1 truth and must be representable — but
it is a disqualifier for the first provider, not a tolerated gap.

**`live-artifact` does not satisfy item 4 — resolved from PRODUCT.md's own
wording.** Item 4 is *"inspect meaningful provider-exposed **changed files
and/or diffs**"*. A running, previewed or published artifact is neither changed
files nor diffs, so observing one is not change inspection, and `live-artifact`
alone does not satisfy item 4.

That is a scoping statement, not a demotion. The same observable output is
squarely within **item 6** — *"a meaningful provider result such as a preview,
build, deployment, pull request, or provider-equivalent outcome"* — so a
provider that exposes running or published output earns that credit under
`result.observe` (§5.5.2), and `handoff.native` still gives the user a path to
the provider's own surfaces. What it cannot do is stand in for item 4. A first
provider therefore needs a genuine change-inspection surface (`inline-patch` or
`remote-reference`) in addition to any live output it exposes.

#### 5.5.2 `result.observe` is typed provider-natively

There is no universal result. The contract carries the provider's own result
kind and an opaque reference plus native URL:

```text
ResultView { providerResultKind : string, resource : ProviderResourceReference, summary? : string }
```

Greenfield2 must not define a canonical result enum. Doing so would rank
providers by how closely they match whichever provider suggested the list.

### 5.6 Approvals

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `approval.discover` | required-if-exposed | `(resource) -> ApprovalRequest[] \| Absent` | |
| `approval.respond` | required-if-exposed | `(approvalHandle, decision) -> Envelope \| Absent \| ProviderRefusal` | Pass-through only. Greenfield2 adds no independent approval or policy authority. |

```text
ApprovalRequest {
  resource       : ProviderResourceReference
  providerKind     : string      # the provider's OWN word for this request
  prompt           : provider-native, uninterpreted
  permittedDecisions : string[]  # provider-defined; Greenfield2 adds none
  observedAt       : instant
}
```

Rules, forced by validation (F3):

- `providerKind` and `permittedDecisions` are **provider vocabularies**. Greenfield2
  defines no normalized approval type and no universal decision set.
- Approval is **not one concept**. A pre-execution plan gate, an in-flight tool
  permission prompt, and a post-hoc code review are different provider
  mechanisms. Collapsing them would be false universality.
- Where a provider has no approval mechanism, Greenfield2 must not invent one.
  Where a provider's gate is human code review on provider-owned infrastructure,
  that is the provider's approval model and Greenfield2 surfaces it as such.

### 5.7 Continuity and handoff

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `continuity.reconnect` | loop | `(resource) -> Envelope \| Absent` | Reopen provider-owned work. Provider-native resume semantics; Greenfield2 promises only what the provider supports. |
| `continuity.replay` | optional | `(resource, from?) -> ProgressUpdate[] \| Absent` | Recover observation history. Reconnect/replay is **not** Greenfield2 execution recovery ([DOMAIN.md](../DOMAIN.md)). |
| `handoff.native` | loop prerequisite | `(resource) -> nativeUrl \| Absent` | The escape hatch. Must be reachable for every capability where L3 is false. |

### 5.8 Errors

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `error.surface` | loop prerequisite | — (cross-cutting) | Provider errors, limits, quota messages, entitlement failures and degraded states pass through with the provider's own code and category **unaltered**, in a **sanitized** presentation. See §8 — fidelity of meaning and safety of presentation are separate obligations. |

### 5.9 Required invocation inputs and provider-native context

A capability identifier says *what* Greenfield2 can ask. It does not say *what
the provider requires before it will accept the ask*. Validation shows those
requirements are provider-owned, sometimes mandatory, and not guessable:

- Jules requires `sourceContext` — a source plus a starting branch — for any
  session that is not repoless.
- Cursor requires `repos[].url` on every repository entry, and needs an
  execution-environment choice (`cloud`, `pool` or `machine`) that is mutually
  exclusive with an explicit repository list.
- Replit's `create_app_from_prompt` requires **both** `appDescription` **and**
  `app_stack`, where `app_stack` must be one of a fixed provider-defined list.
- Copilot addresses work by `{owner}/{repo}` and optionally `base_ref`.

A contract that carries only a provider-neutral `intent` cannot express any of
this, and Greenfield2 would have no way to render the inputs a provider requires
before work can start. Required inputs are therefore **declared by the adapter**
and carried as provider-native context.

```text
RequiredInput {
  name          : string   # The provider's OWN field name, verbatim.
  label         : string?  # The provider's own user-facing wording, where it has one.
  required      : boolean
  valueKind     : text | choice | reference | attachment | flag
  allowedValues : string[]?  # For `choice`: the provider's own value tokens, verbatim.
                             # Greenfield2 MUST NOT rename, re-interpret or extend this list.
  source        : user | connection-context | provider-default
  constraint    : string?  # Provider-stated limits (max count, max size, …), verbatim where possible.
}

InvocationContext {
  connection : ProviderConnection
  inputs     : map<name, provider-native value>   # only declared names are accepted
  target     : ProviderResourceReference?          # when continuing or acting on existing work
}
```

Rules:

1. **The adapter declares; Greenfield2 renders.** Greenfield2 builds the
   invocation surface from `requiredInputs` in the manifest (§7). It never
   hard-codes a provider's field list.
2. **`allowedValues` is provider vocabulary.** A provider-defined choice list —
   Replit's `app_stack`, Cursor's `env.type` — is displayed using the provider's
   own tokens. Greenfield2 may add presentation labels beside them; it must not
   rename the values, because the values go back to the provider.
3. **`source` separates what the user must supply** from what Greenfield2 can
   already resolve from the connection or leave to the provider. Only
   `source: user` inputs need a visible control.
4. **An unsatisfiable required input makes the capability non-invocable**, and
   the reason shown is the missing provider input — not a Greenfield2 failure.
5. **A missing required input is not `Absent`.** `Absent` means the provider
   does not expose the capability; a missing input means the user has not yet
   supplied what the provider needs. Conflating them would hide a completable
   form behind a permanent "not available".
6. **Provider defaults are respected, not replaced.** Where a provider resolves
   an omitted value itself — Cursor resolves an omitted model as user default →
   team default → system default — Greenfield2 passes the omission through
   rather than inventing a value the provider did not choose.
7. **Attachments carry provider-stated limits.** Cursor caps API image inputs at
   5 images and 15 MB each; those limits are the provider's, are declared in
   `constraint`, and are surfaced before submission rather than discovered as a
   refusal.

---

## 6. Update delivery is a declared property, not a shared mechanism

Validation found four categorically different delivery mechanisms across the
validation set, including two in the same provider (F5). The contract therefore
declares delivery per capability rather than assuming one.

```text
delivery ∈ { snapshot, poll, stream, push }
```

| Mode | Meaning | Greenfield2 obligation |
| :-- | :-- | :-- |
| `snapshot` | Value read on demand only. | State `staleness` honestly. |
| `poll` | Adapter re-reads on an interval. | Declare interval; bound staleness. |
| `stream` | Provider pushes an ordered event sequence. | Declare resumability and the retention window; handle expiry as "read current state", not as an error. |
| `push` | Provider calls back out-of-band. | Declare verification of the callback's authenticity. |

Rules:

- **Transport never appears above the adapter.** No `sse`, `websocket`, `mcp`,
  `rest` or `webhook` token is part of this contract's vocabulary.
- `stream` is **never required**. A provider with `snapshot`-only delivery is
  still contract-conformant.
- Where a provider's push or stream has a **retention window**, expiry is an
  expected condition: fall back to reading current state rather than surfacing a
  failure (F5).
- A provider may deliver different capabilities by different modes. The
  declaration is per capability, not per provider.
- Greenfield2 core sees exactly one thing: *the adapter will report change, or
  Greenfield2 will ask*. Maximum staleness is declared so the UI can label it.

---

## 7. Capability manifest

`discovery.capabilities` returns the adapter's manifest. The manifest is the
contract's machine-facing surface: it is what lets Greenfield2 render a provider
it has never been told about, and what stops a capability from being assumed.

```text
CapabilityManifest {
  providerId      : string
  manifestVersion : string
  capabilities    : CapabilityDeclaration[]
  declaredAt      : instant
}

CapabilityDeclaration {
  id              : string          # a §5 identifier
  l1              : exposed | not-exposed
  fulfilment      : string?         # provider-specific fulfilment kind (§5.5.1 etc.)
  delivery        : delivery[]      # §6
  maxStaleness    : duration?
  nativeKind      : string?         # the provider's own noun for the subject
  requiredInputs  : RequiredInput[]?  # §5.9 — what the provider needs before it accepts the ask
  mutates         : boolean         # true when invocation can change provider state or spend
                                    # provider resources; drives §3.2.1 handling of unknown L2
  notes           : string?         # provider-stated limitations, verbatim where possible
}
```

Manifest rules:

1. **Absence is declared, not implied.** A capability the provider does not
   expose appears with `l1: not-exposed`, or is absent from the manifest
   entirely; both mean the same thing and neither may be read as "not yet
   probed".
2. **The manifest is evidence for L1 only.** It says nothing about L2 or L3.
3. **It is a snapshot with a timestamp**, never a permanent fact.
4. **Unrecognized capability identifiers are ignored, not errors.** A provider or
   adapter may declare identifiers Greenfield2 does not yet have a surface for;
   that is L3-false, and it is the handoff case.
5. **How the manifest is produced is adapter-local.** Validation found providers
   with a first-class discovery mechanism, providers with none, and providers
   where discovery requires a feature-flagged query (F9). The contract defines
   the manifest's shape; it does not define the probe.

---

## 8. Errors, limits and provider truth

[PRODUCT.md](../PRODUCT.md) states the requirement precisely: provider errors
*"remain provider truth. Greenfield2 may format them for readability **and safe
presentation** but does not reinterpret them into a separate Greenfield2
execution state machine."*

Two obligations sit in that sentence and they pull in opposite directions. An
earlier draft of this contract resolved the tension by saying provider messages
pass through *"verbatim"*, which satisfies fidelity and ignores safety. They are
separated here instead.

### 8.1 Semantics pass through unaltered

What the error **means** is the provider's, and Greenfield2 does not touch it:

- the provider's own error code and category;
- whether the provider treats it as terminal, retryable or awaiting-user;
- whether it is an entitlement failure, a quota or rate limit, a policy refusal,
  an invalid-input rejection or a provider-side fault;
- any provider-stated remedy, limit or quota value.

A Greenfield2-invented error taxonomy that replaces or hides the provider's is a
fidelity failure. Greenfield2 adds **no** execution state machine over the top,
and does not decide retryability, completion or verification for itself.

### 8.2 Presentation is sanitized, because provider text is untrusted input

Provider-supplied free text is **data, not instructions and not markup**. It is
rendered safely before it reaches a screen:

| Risk in provider-supplied text | Required handling |
| :-- | :-- |
| Markup or script (`<script>`, HTML, control characters) | Rendered as inert text; never parsed, never executed |
| Embedded URLs | Not auto-navigated and not auto-fetched; shown as text, with provider attribution if linked at all |
| Credential-shaped substrings (tokens, keys, signed URLs, cookies) | Redacted before display and before logging, using the same rule the repository gate applies to its own output |
| Personal or account data not needed to act on the error | Minimised under the minimum-data rule |
| Instruction-shaped content ("ignore previous instructions…", imperative text addressed to an assistant) | Treated as data. Never executed, never forwarded to any model or automation as a directive |
| Unbounded length | Truncated for display with the full text available on demand |

Sanitizing presentation is **not** reinterpretation. Greenfield2 may strip
markup, redact a credential and truncate a stack trace; it may not change which
category the error belongs to, invent a cause the provider did not state, or
soften a refusal into a warning.

### 8.3 Refusals, retry and spend

- `ProviderRefusal` carries the provider's own code and category (§8.1) plus the
  sanitized presentation (§8.2).
- An entitlement refusal updates L2 per §3.2.1; it is not treated as permanent.
- A refusal on a **mutating or spend-bearing** capability is never auto-retried
  (§3.2.1). Rate-limit refusals may be retried only on explicit user action or
  under an adapter-declared, user-visible backoff.
- Where a provider exposes no structured code at all, the adapter reports
  `unclassified` rather than guessing a category. Guessing is reinterpretation.

### 8.4 Other provider truths

- Autonomy, durability, recovery, completion and verification are provider
  truths in MVP. Where a provider does not support one, Greenfield2 does not
  fabricate it.
- **Unrecognized values are a normal condition.** Provider enums drift and
  providers ship concurrent API versions. Every provider-supplied token —
  state, kind, decision, error code — must have a defined path for "value
  Greenfield2 has never seen", which is `unknown`, not a crash and not a
  silent substitution (F10).

---

## 9. Adapter obligations

A provider adapter is the only place provider specifics may live. It must:

1. Use a **supported external-client path** — never scraped, reverse-engineered
   or first-party-only interfaces ([PRODUCT.md](../PRODUCT.md)).
2. Produce a **capability manifest** (§7) and keep it current.
3. Declare the provider's **required invocation inputs** (§5.9), including
   provider-owned choice lists verbatim and provider-stated limits.
4. Declare **`mutates`** truthfully for every capability, so §3.2.1 can treat
   spend-bearing invocations differently from reads.
5. Answer each §5 capability in provider-native terms, or report `Absent`.
6. Map provider states to the §4.5 liveness hint **only** under those
   constraints, with `unknown` as the fallback.
7. Keep **transport, retries, pagination, rate limits, pagination cursors and
   auth mechanics** entirely inside the adapter.
8. Preserve provider error **semantics unaltered** and hand Greenfield2 a
   **sanitized** presentation (§8.1, §8.2). Report `unclassified` rather than
   guessing a category the provider did not state.
9. Keep **Provider Connection** and **Provider Resource Reference** separate
   (§4.3), and never return one where the other is meant.
10. Own **staleness**: declare maximum staleness per capability and report
    `staleness` on envelopes.
11. Hold **no provider development state** beyond the minimum permitted
    cache/reconnect state, bounded and cleaned on disconnect.
12. Distinguish `Absent` from `ProviderRefusal` from a missing required input
    (§5.9 rule 5) from transient failure. Conflating them causes Greenfield2 to
    retry things that will never succeed, to hide things that are merely
    rate-limited, or to present a completable form as permanently unavailable.

---

## 10. Anti-leakage prohibitions

These are the rules [ADR-0005](../decisions/0005-provider-contract-before-provider-selection.md)
exists to enforce. They are stated as prohibitions so a reviewer can check them
mechanically.

| # | Prohibition |
| :-- | :-- |
| P1 | No Greenfield2-owned entity or type named after a provider noun (`Session`, `Run`, `Task`, `Agent`, `Workspace`, `Project`, `App`, `Artifact`, `Activity`, `Repl`, …). |
| P2 | No Greenfield2 state machine for provider resources. |
| P3 | No normalized enum over provider states, approval kinds, result kinds or error codes. |
| P4 | No transport concept (`rest`, `mcp`, `sse`, `websocket`, `webhook`, `poll`-as-protocol) above the adapter boundary. |
| P5 | No capability becomes `required` merely because one provider has it. |
| P6 | No capability is approximated where a provider does not expose it. `Absent` is rendered as absent. |
| P7 | Provider nouns are displayed using the provider's own words, not Greenfield2 synonyms. |
| P8 | Greenfield2 core never branches on `ProviderResourceReference.kind`. |
| P9 | No Greenfield2-owned approval, execution, recovery, completion or verification authority. |
| P10 | No identifier structure is parsed. `ref` is opaque. |
| P11 | This contract does not relax any accepted requirement in [PRODUCT.md](../PRODUCT.md). Where the two disagree, PRODUCT.md wins and the contract is wrong (F11). |
| P12 | A **Provider Connection** is never returned where a **Provider Resource Reference** is meant, or vice versa (F14). |
| P13 | Provider-supplied text is untrusted data. It is never parsed as markup, never executed, never treated as an instruction, and never displayed unsanitized (§8.2). |
| P14 | Provider error *semantics* are never rewritten. Sanitizing presentation is permitted; reclassifying the error is not (§8.1). |

### 10.1 Audit procedure

Two checks a reviewer can run. Both are cheap and both are meant to be re-run
whenever this contract or an adapter changes.

**A — no Greenfield2-owned provider nouns.** The Greenfield2-owned vocabulary is
fixed at five concepts in [DOMAIN.md](../DOMAIN.md). Any *new* Greenfield2-owned
entity named after a provider noun is leakage:

```bash
grep -rniE 'greenfield2[- ](owned|canonical)[^|]{0,40}\b(session|run|task|agent|workspace|project|app|artifact|activity|repl)\b' docs/
```

Expected: no matches asserting such an entity exists. Mentions inside
prohibitions, audits and provider profiles are fine — the check is for
*declarations of ownership*.

**B — no transport above the adapter.** Transport words may appear in this
file only inside the §1 "not a transport specification" row, the §6 delivery
rules, the P4 prohibition, and this audit command itself:

```bash
grep -rnE '\b(REST|SSE|WebSocket|webhook|GraphQL)\b' docs/architecture/PROVIDER_CAPABILITY_CONTRACT.md
```

Expected: matches confined to those four places. The pattern is deliberately
case-sensitive — a case-insensitive `rest` also matches ordinary English prose
such as "the rest of the loop", which would make the check cry wolf.

---

## 11. Deliberately unresolved

Recorded so a future session does not mistake silence for an oversight, or fill
the gap by accident:

- **Which provider is first.** Explicitly deferred by
  [ADR-0005](../decisions/0005-provider-contract-before-provider-selection.md)
  until this contract is validated. The validation is complete
  ([PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md)) and the
  structural decision
  ([ADR-0006](../decisions/0006-capability-manifest-and-three-layer-availability.md))
  is **accepted**, so Issue #9's ordering precondition is discharged and
  selection may proceed. It belongs to the existing
  **[Issue #8 — Architecture: evaluate first MVP provider](https://github.com/anthracite-labs/Greenfield2/issues/8)**
  using the Product Fit and Integration Legitimacy gates in
  [PRODUCT.md](../PRODUCT.md). **This document selects no provider**, and no
  duplicate provider-selection issue should be opened.
- **Transport, framework, database, hosting, UI technology.** Locked by
  `ALLOW_APP_STACK=0`.
- **Adapter interface in code.** Requires the application-stack ADR.
- **Whether a common work-item abstraction is genuine or accidental.** Validation
  found three of four shapes with some run-like unit and one with none (F1), so
  `work.*` stays optional-tolerant rather than structural. Revisit only with
  more providers, not by assumption.
- **Cross-provider composition (BYOK/BYOA/BYOW).** Post-MVP direction in
  [PRODUCT.md](../PRODUCT.md); deliberately not modelled here.
- *Resolved, recorded here so it is not reopened:* **`live-artifact` alone does
  not satisfy minimum V1 item 4** (§5.5.1). PRODUCT.md item 4 says *"changed
  files and/or diffs"*; running or published output is neither. Such output
  still counts under item 6 via `result.observe`. This follows from the accepted
  product wording and needed no new product decision.

---

## Related

- [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) — validation record and revision log
- [../ARCHITECTURE.md](../ARCHITECTURE.md) — Architecture mandate, invariants, open decisions
- [../PRODUCT.md](../PRODUCT.md) — reviewed product definition and MVP capability loop
- [../DOMAIN.md](../DOMAIN.md) — Greenfield2-owned vocabulary and ownership semantics
- [../decisions/0005-provider-contract-before-provider-selection.md](../decisions/0005-provider-contract-before-provider-selection.md) — sequencing decision
- [../decisions/0006-capability-manifest-and-three-layer-availability.md](../decisions/0006-capability-manifest-and-three-layer-availability.md) — structural decision