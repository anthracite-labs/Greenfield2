# Provider Capability Contract

**Version:** 1.0 (validated)
**Status:** Architecture-level contract. Not an interface implementation, not a schema file, not an SDK.
**Lifecycle:** `PROJECT_PHASE=architecture`, `ALLOW_APP_STACK=0`. This document selects no transport, framework, database, auth implementation, hosting target, UI technology or provider.
**Authority:** [ADR-0005](../decisions/0005-provider-contract-before-provider-selection.md) requires this contract to exist and be validated before any first-provider selection. [ADR-0006](../decisions/0006-capability-manifest-and-three-layer-availability.md) records its structural shape.
**Validation:** see [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md). Version 1.0 is the post-validation form; the revision log there records what earlier drafts got wrong.

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

- `Result` — a provider-native payload inside the envelope defined in §4.
- `Absent` — the provider does not expose this capability. Distinguished from a
  failure. `Absent` is a legal, expected, permanent-until-the-provider-changes
  outcome, and must never be retried as if it were transient.
- `ProviderRefusal` — the provider declined: not entitled, out of quota, policy,
  invalid input, or a provider-side error. Carries the provider's own error,
  verbatim (§8).

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
effective(capability, account) = L1 ∧ L2 ∧ L3
```

### 3.1 Why the layers must not be collapsed

Each failure mode has a **different remedy for the user**, so a single
"unavailable" flag would make the interface lie by omission:

| L1 | L2 | L3 | Disposition | Rationale |
| :-- | :-- | :-- | :-- | :-- |
| ✗ | – | – | **Not offered.** Hidden or shown as "not available on this provider". | [PRODUCT.md](../PRODUCT.md): Greenfield2 must not invent missing concepts for visual completeness. |
| ✓ | ✗ | – | **Explained as not available for this account**, with the provider's own path forward. | The capability exists; the user's remedy is with the provider, not with Greenfield2. |
| ✓ | ✓ | ✗ | **Native-provider handoff.** "Open in provider". | [PRODUCT.md](../PRODUCT.md): native-provider handoff is preferred over guessed behavior. |
| ✓ | ✓ | ✓ | **Operable in Greenfield2**, with native handoff still offered. | The full case. |

### 3.2 L2 is tri-state, not boolean

```text
entitlement ∈ { entitled, not-entitled, unknown }
```

`unknown` is a first-class value, not a default for `false`. Validation found
providers with **no entitlement endpoint at all**, where the only probe is an
attempt that either succeeds or is refused (§ Validation F6). Rendering
`unknown` as `not-entitled` would tell users they lack something they may have.

Rules:

- `unknown` must be surfaced as uncertainty, never as denial.
- L2 is **never cached as permanent**. [PRODUCT.md](../PRODUCT.md): capabilities
  and entitlements may change; Greenfield2 must reflect current provider truth.
  A cached L2 carries an explicit observation time and a bounded lifetime.
- Authentication success is **not** evidence for L2. ([DOMAIN.md](../DOMAIN.md):
  *Authentication != authorization*, *Authorization != entitlement*.)
- Greenfield2 requests the provider's recommended minimum authority first and
  escalates only when a user invokes a capability that requires more
  (`connection.scope.escalate`).

### 3.3 L3 is Greenfield2's own claim, and it is falsifiable

L3 is the only layer Greenfield2 owns. It must be recorded per provider
capability, not globally: a capability Greenfield2 has a validated surface for on
one provider is not thereby supported on another, because the fulfilment differs
(§6). L3 must never be inferred from L1 or L2.

---

## 4. Provider-native handles

Greenfield2 carries references to provider resources. It never owns them.

```text
ProviderHandle {
  providerId     : string   # Greenfield2's identifier for the provider. Greenfield2-owned.
  kind           : string   # The provider's OWN noun, verbatim ("Session", "Agent", "Run",
                            # "App", "Task", …). Display vocabulary, never a discriminator.
  ref            : string   # Opaque, provider-chosen. Greenfield2 MUST NOT parse, split,
                            # compose, prefix-match or validate its structure.
  nativeUrl      : string?  # Provider-native deep link, for handoff.
  providerState  : string?  # The provider's own state token, verbatim, uninterpreted.
  observedAt     : instant  # When Greenfield2 last saw it. Staleness tracking, NOT provider truth.
}
```

### 4.1 Non-negotiables

1. **`ref` is opaque.** Validation found the same provider change its own
   identifier format between API versions (Validation F8). Anything Greenfield2
   derives from `ref`'s structure breaks silently.
2. **`kind` is never a branch condition in Greenfield2 core.** Adapters may
   branch on their own provider's kinds; core logic may not. A `switch` on
   `kind` in core is vendor leakage by construction.
3. **`providerState` passes through verbatim.** Greenfield2 does not define a
   provider state enum, does not map provider states onto each other, and must
   tolerate state values it has never seen (§4.3).
4. **A reference is not ownership.** ([DOMAIN.md](../DOMAIN.md).) A stale handle
   must never be presented as current provider truth.

### 4.2 The envelope

Every capability result is provider-native content inside a Greenfield2-owned
envelope. The envelope carries *provenance and freshness*; the payload carries
*meaning*, and meaning stays with the provider.

```text
Envelope {
  handle      : ProviderHandle
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

### 4.3 Presentation liveness hint — a controlled exception

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
[PRODUCT.md](../PRODUCT.md). A first provider must expose enough of the `loop`
group to deliver a workspace-level development loop; `optional` capabilities are
richness Greenfield2 may expose where validated.

### 5.1 Connection and authorization

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `connection.authorize` | loop prerequisite | `(user) -> ProviderHandle \| ProviderRefusal` | Must use a provider-supported authorization path. Greenfield2 begins with the provider's recommended minimum authority. |
| `connection.revoke` | loop prerequisite | `(connection) -> Result` | Triggers bounded cache cleanup per the minimum-data rule. |
| `connection.identity` | loop prerequisite | `(connection) -> Envelope` | Provider account identity as the provider states it. |
| `connection.scope.escalate` | optional | `(connection, requiredFor) -> Result \| ProviderRefusal` | Invoked only when a user action genuinely requires more authority. |

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
| `work.start` | loop | `(connection, intent) -> Envelope \| Absent` | Begin new provider work. `intent` is provider-neutral (a user's instruction plus optional attachments); how it maps is adapter-local. |
| `work.continue` | loop | `(handle, intent) -> Envelope \| Absent` | Continue existing provider work where the provider exposes continuity. |
| `work.list` | loop | `(connection, page?) -> Envelope[] \| Absent` | `Absent` is legal: a provider whose unit of work is a long-lived resource rather than a run has nothing to enumerate. |
| `work.inspect` | loop | `(handle) -> Envelope` | |
| `work.control` | optional | `(handle, verb) -> Envelope \| Absent \| ProviderRefusal` | `verb` is provider-native (`pause`, `cancel`, `resume`, `archive`, …). Greenfield2 defines no universal verb set. |

### 5.4 Agent interaction

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `agent.interact` | loop | `(handle, input) -> Envelope \| Absent` | Forward user input to the provider's Agent. A pass-through: Greenfield2 adds no interpretation, no approval, no policy. |

### 5.5 Observation

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `progress.observe` | loop | `(handle, cursor?) -> ProgressUpdate[] \| Absent` | Progress, plan, tool and activity telemetry as the provider reports it. Delivery mode declared per §6. |
| `changes.inspect` | loop | `(handle) -> ChangeView \| Absent` | See §5.5.1 — fulfilment varies categorically. |
| `result.observe` | loop | `(handle) -> Envelope[] \| Absent` | The provider's own meaningful outcome. Typed provider-natively (§5.5.2). |
| `usage.observe` | optional | `(connection \| handle) -> Envelope \| Absent` | Quota, limits, token or credit usage where the provider exposes it. |

#### 5.5.1 `changes.inspect` fulfilment kinds

Validation established that change inspection is **not** one thing
(Validation F2). The contract therefore declares a fulfilment kind rather than
assuming a diff:

```text
ChangeView.kind ∈ { inline-patch, remote-reference, live-artifact, none }
```

| Kind | Meaning | Handoff |
| :-- | :-- | :-- |
| `inline-patch` | The provider hands Greenfield2 the change content directly. | Optional |
| `remote-reference` | The provider exposes the change only as a reference to somewhere it owns. | Expected |
| `live-artifact` | The change is observable only as running/published output. | Expected |
| `none` | The provider exposes no change-inspection surface. | Required |

`none` is a legitimate, honest answer. A first provider may still qualify for
the MVP loop if the rest of the loop is strong and Greenfield2 hands off for
change review — that is a provider-selection judgement, explicitly **not** made
here.

#### 5.5.2 `result.observe` is typed provider-natively

There is no universal result. The contract carries the provider's own result
kind and an opaque reference plus native URL:

```text
ResultView { providerResultKind : string, handle : ProviderHandle, summary? : string }
```

Greenfield2 must not define a canonical result enum. Doing so would rank
providers by how closely they match whichever provider suggested the list.

### 5.6 Approvals

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `approval.discover` | required-if-exposed | `(handle) -> ApprovalRequest[] \| Absent` | |
| `approval.respond` | required-if-exposed | `(approvalHandle, decision) -> Envelope \| Absent \| ProviderRefusal` | Pass-through only. Greenfield2 adds no independent approval or policy authority. |

```text
ApprovalRequest {
  handle           : ProviderHandle
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
| `continuity.reconnect` | loop | `(handle) -> Envelope \| Absent` | Reopen provider-owned work. Provider-native resume semantics; Greenfield2 promises only what the provider supports. |
| `continuity.replay` | optional | `(handle, from?) -> ProgressUpdate[] \| Absent` | Recover observation history. Reconnect/replay is **not** Greenfield2 execution recovery ([DOMAIN.md](../DOMAIN.md)). |
| `handoff.native` | loop prerequisite | `(handle) -> nativeUrl \| Absent` | The escape hatch. Must be reachable for every capability where L3 is false. |

### 5.8 Errors

| Capability | Role | Shape | Notes |
| :-- | :-- | :-- | :-- |
| `error.surface` | loop prerequisite | — (cross-cutting) | Provider errors, limits, quota messages, entitlement failures and degraded states pass through with the provider's own text and code, formatted for readability only. |

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

- Provider errors, quota messages, entitlement refusals and degraded states are
  **provider truth**. Greenfield2 formats them for safe, readable presentation
  and does not reinterpret them into a Greenfield2 state machine.
- `ProviderRefusal` must carry the provider's own code and message. A
  Greenfield2-invented error taxonomy that hides the provider's is a fidelity
  failure.
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
3. Answer each §5 capability in provider-native terms, or report `Absent`.
4. Map provider states to the §4.3 liveness hint **only** under those
   constraints, with `unknown` as the fallback.
5. Keep **transport, retries, pagination, rate limits, pagination cursors and
   auth mechanics** entirely inside the adapter.
6. Surface provider errors verbatim (§8).
7. Own **staleness**: declare maximum staleness per capability and report
   `staleness` on envelopes.
8. Hold **no provider development state** beyond the minimum permitted
   cache/reconnect state, bounded and cleaned on disconnect.
9. Distinguish `Absent` from `ProviderRefusal` from transient failure. Conflating
   them causes Greenfield2 to retry things that will never succeed, or to hide
   things that are merely rate-limited.

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
| P8 | Greenfield2 core never branches on `ProviderHandle.kind`. |
| P9 | No Greenfield2-owned approval, execution, recovery, completion or verification authority. |
| P10 | No identifier structure is parsed. `ref` is opaque. |

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
  until this contract is validated. Validation is now complete
  ([PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md)); selection is a
  separate decision using the Product Fit and Integration Legitimacy gates in
  [PRODUCT.md](../PRODUCT.md). **This document selects no provider.**
- **Transport, framework, database, hosting, UI technology.** Locked by
  `ALLOW_APP_STACK=0`.
- **Adapter interface in code.** Requires the application-stack ADR.
- **Whether a common work-item abstraction is genuine or accidental.** Validation
  found three of four shapes with some run-like unit and one with none (F1), so
  `work.*` stays optional-tolerant rather than structural. Revisit only with
  more providers, not by assumption.
- **Cross-provider composition (BYOK/BYOA/BYOW).** Post-MVP direction in
  [PRODUCT.md](../PRODUCT.md); deliberately not modelled here.

---

## Related

- [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) — validation record and revision log
- [../ARCHITECTURE.md](../ARCHITECTURE.md) — Architecture mandate, invariants, open decisions
- [../PRODUCT.md](../PRODUCT.md) — reviewed product definition and MVP capability loop
- [../DOMAIN.md](../DOMAIN.md) — Greenfield2-owned vocabulary and ownership semantics
- [../decisions/0005-provider-contract-before-provider-selection.md](../decisions/0005-provider-contract-before-provider-selection.md) — sequencing decision
- [../decisions/0006-capability-manifest-and-three-layer-availability.md](../decisions/0006-capability-manifest-and-three-layer-availability.md) — structural decision
