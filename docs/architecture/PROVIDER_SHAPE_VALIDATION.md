# Provider Shape Validation

**Purpose:** Evidence that [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) is provider-neutral rather than first-vendor-shaped, and the record of what earlier drafts got wrong.
**Issue:** #9 — [ADR-0005](../decisions/0005-provider-contract-before-provider-selection.md) requires validation against at least three materially different provider shapes before first-provider selection.
**Result:** four shapes validated; **fifteen contract revisions** (F1–F10 from the provider-shape comparison, F11–F15 from independent review). Contract promoted to v1.1.
**Selects no provider.** Every provider below is validation evidence. Nothing here ranks, scores, recommends or selects one.

---

## 1. Evidence basis and its limits

### 1.1 How the evidence was gathered

`.ecc/skills/research.md` requires a channel preflight before relying on a
channel. Preflight result, this session:

```text
$ curl -sS -o /dev/null -m 8 https://jules.google/
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to jules.google:443
$ curl -sS -o /dev/null -m 8 https://docs.cursor.com/
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL
$ curl -sS -o /dev/null -m 8 https://replit.com/
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL
$ curl -sS -o /dev/null -m 8 https://docs.github.com/
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL
```

Sandbox egress is allowlisted and excludes every vendor documentation host, so
**no provider API was called and no provider surface was exercised live**.
Evidence comes from primary provider documentation read through the
platform-side page-fetch and search tools. This is documentation-level
validation, not integration testing, and is recorded as such.

### 1.2 Sources

All accessed **2026-09-13**.

| Provider | Primary source | Method |
| :-- | :-- | :-- |
| Google Jules | `jules.google/docs/api/reference/overview/`, `/sessions/`, `/activities/`, `/types/` | `/sessions/` fetched directly; others read from primary-page extracts |
| Cursor Cloud Agents | `cursor.com/docs/cloud-agent/api/endpoints` (v1); `docs.cursor.com/en/background-agent/api/*` (legacy v0) | v1 endpoints page fetched directly (3 sections); v0 read from primary-page extracts |
| Replit | `docs.replit.com/platforms/mcp-server` | Fetched directly |
| GitHub Copilot | `docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api` | Fetched directly |

Claims marked **[SECONDARY]** rest on third-party write-ups rather than provider
documentation and are labelled where used.

### 1.3 Point-in-time, not permanent

Every surface below is explicitly pre-stable:

- Jules is `v1alpha`.
- Cursor states: *"The Cloud Agents API v1 is in public beta. APIs may change
  before general availability"*, and ships `v0` and `v1` concurrently.
- The Copilot agent-tasks API is *"in public preview and subject to change"*, and
  its GraphQL assignment path requires a `GraphQL-Features` header.
- Replit's MCP tool set is unversioned; MCP tool schemas do not carry REST-style
  version guarantees.

**This is the single most important finding for the contract's design:** the
instability is not an accident of these four providers, it is the normal
condition of the category. The contract is therefore written so that provider
drift is an expected input, not a breaking surprise (§4.2, §8, §10).

---

## 2. Shape profiles

Materially different on the axes that matter, not cosmetically different.

### 2.1 Google Jules — hierarchical REST resources with an explicit plan gate

| Axis | Observed |
| :-- | :-- |
| Primary unit | `Session` — *"a contiguous amount of work within the same context"* |
| Topology | Hierarchical resource names: `sessions/{id}`, `sessions/{id}/activities/{id}`, `sources/{id}`. Session also carries a separate flat `id`. |
| Auth | API key in the `x-goog-api-key` header, from `jules.google.com/settings` |
| Operations | create / list / get / delete session; `:sendMessage`; `:approvePlan`; list / get activities; list / get sources |
| Lifecycle | Enumerated `SessionState`: `STATE_UNSPECIFIED`, `QUEUED`, `PLANNING`, `AWAITING_PLAN_APPROVAL`, `AWAITING_USER_FEEDBACK`, `IN_PROGRESS`, `PAUSED`, `FAILED`, `COMPLETED` |
| Approval | **Explicit, opt-in**: `requirePlanApproval: true` gates execution; *"If not set, plans are auto-approved."* Dedicated `:approvePlan` endpoint |
| Progress | Structured `Activity` stream. Typed events: `planGenerated`, `planApproved`, `userMessaged`, `agentMessaged`, `progressUpdated`, `sessionCompleted`, `sessionFailed` |
| Changes | `Artifact.changeSet.gitPatch.unidiffPatch` — **inline patch content**, with `baseCommitId` and `suggestedCommitMessage`. Also `bashOutput` and `media` artifacts |
| Result | `outputs[].pullRequest` (`url`, `title`, `description`); `automationMode: AUTO_CREATE_PR` |
| Delivery | **Poll**, token pagination (`pageSize` / `pageToken`) |
| Discovery | `sources` list. No capability endpoint; no entitlement endpoint |
| Continuity | `AWAITING_USER_FEEDBACK` resumes via `:sendMessage` |
| Handoff | `Session.url` to the Jules web app |
| Notable | Repoless sessions supported (`sourceContext` optional). State vocabulary is documented as drifting: both `CANCELLED` and `CANCELED` spellings occur, and `COMPLETED_UNKNOWN` appears in the field **[SECONDARY]** |

### 2.2 Cursor Cloud Agents — durable agent plus per-prompt runs, with a resumable stream

| Axis | Observed |
| :-- | :-- |
| Primary unit | **Two**: a durable `Agent` and a per-prompt `Run`. First-party: *"This API splits work into a durable agent plus per-prompt runs, replacing the flatter v0 surface."* |
| Topology | Flat paths, two levels: `/v1/agents/{id}`, `/v1/agents/{id}/runs/{runId}` |
| Auth | *"accepts both Basic and Bearer authentication"*; user API key or Enterprise **service-account** key; `POST /v1/sub-tokens` mints one-hour user-scoped worker tokens |
| Operations | create / list / get / archive / unarchive / delete agent; create / list / get / **stream** / cancel run; list / download artifacts; agent usage; models; repositories |
| Lifecycle | Run statuses include `RUNNING`, `FINISHED`, `CANCELLED`. Cancellation is **terminal**: *"cannot be resumed. To continue the conversation, create a new run on the same agent."* |
| Approval | **None observed** in the Cloud Agents API surface |
| Progress | **SSE stream** (`Accept: text/event-stream`) with typed events `status`, `assistant`, `tool_call`, `result`, `done`. `tool_call` carries `callId`, `name` (`read_file`, `run_terminal_cmd`, `mcp`), `status`, and `args`/`result` with explicit `truncated` flags when payloads are too large |
| Changes | **No diff surface.** The `result` event carries `git.branches[{repoUrl, branch}]` — a **reference** to a branch Cursor owns. Artifacts are separate, agent-scoped files (e.g. `artifacts/screenshot.png`), *"because the workspace persists across runs"* |
| Result | Git branch reference; PR creation via agent `target` configuration |
| Delivery | **Three in one provider**: snapshot/poll, `stream` (SSE) for runs, and legacy `push` (webhooks). *"Webhooks are coming soon. The legacy v0 API still supports them."* |
| Stream durability | Resumable via `Last-Event-ID`; event ids are explicitly opaque (*"an opaque string you should not parse"*); `X-Cursor-Stream-Retention-Seconds` header; after the window the endpoint *"may return `410 stream_expired`"* and the documented remedy is to **read terminal state instead of retrying** |
| Discovery | `GET /v1/models`, `GET /v1/repositories`. No capability manifest; no entitlement endpoint (`GET /v1/me` returns key identity, not entitlements) |
| Usage | `GET /v1/agents/{id}/usage` — per-run token counts (`inputTokens`, `outputTokens`, `cacheWriteTokens`, `cacheReadTokens`) |
| Notable | **Identifier format changed between versions**: `bc_abc123` (v0) → `bc-00000000-0000-0000-0000-000000000001` (v1). Artifact path semantics changed too: *"v1 paths are relative; absolute v0 paths (`/opt/cursor/artifacts/...`) are not accepted."* Repository listing is tightly rate-limited **[SECONDARY]** |

### 2.3 Replit — an app-shaped provider reached through MCP tools

| Axis | Observed |
| :-- | :-- |
| Primary unit | **`App`** — a long-lived resource. **No run/session/task concept is exposed at all.** |
| Topology | Not a resource hierarchy: a flat set of eight MCP tools |
| Transport | MCP over Streamable HTTP at `https://replit-mcp.com/server/mcp`; *"OAuth using protected-resource discovery"*; *"Do not create a custom OAuth server."* |
| Operations | `create_app_from_prompt`, `update_app_using_prompt`, `publish_app`, `get_publish_status`, `list_apps`, `search_apps`, `resolve_app_by_name`, `ask_question` |
| Lifecycle | **Not exposed.** No state enum. Work is asynchronous and its progress is observable only indirectly: *"A create, update, or publish request is still running — Wait before retrying."* |
| Approval | **None exposed.** |
| Progress | **None exposed.** Closest is `get_publish_status`, which reports publish state and public URL, not agent activity |
| Changes | **None exposed.** No file, diff or patch surface. The published app is the observable output |
| Result | A published App and its public URL — categorically different from a pull request |
| Delivery | **Snapshot only**, by polling `get_publish_status` |
| Discovery | MCP `tools/list` is the capability mechanism — structurally different from a REST capability endpoint |
| Entitlement | Implicit and scope-based: *"You can create apps or work with apps that you can edit, including apps shared with you."* `list_apps` returns only apps the caller can edit (default limit 25, max 50). No entitlement endpoint |
| Continuity | `update_app_using_prompt` mutates the existing app. There is no work item to reopen |
| Notable | `create_app_from_prompt` requires an `app_stack` from a provider-defined list (`react_website`, `mobile_app`, `design`, `slides`, `animation`, `data_visualization`, `3d_game`, `document`, `spreadsheet`) — a provider-owned taxonomy with no equivalent in the other three shapes. Separately, Replit's documented HTTP API is an Enterprise **Admin API** for account governance; there is no documented public REST endpoint that builds an app **[SECONDARY]** |

### 2.4 GitHub Copilot — issue/PR-shaped, entitlement-gated, preview-flagged

| Axis | Observed |
| :-- | :-- |
| Primary unit | **`Task`** (`/agents/repos/{owner}/{repo}/tasks`), plus a second, different entry point: assigning an **Issue** to `copilot-swe-agent` |
| Topology | Repository-scoped paths, GitHub-shaped: `/agents/repos/{owner}/{repo}/tasks`, plus an account-wide `/agents/tasks` |
| Auth | *"only supports user-to-server tokens"* — PAT, OAuth app token, or GitHub App user-to-server token. **Server-to-server tokens such as GitHub App installation access tokens are not supported.** Fine-grained PATs need read metadata plus read/write on actions, contents, issues and pull requests; classic PATs need `repo` |
| Entitlement | **The most explicit of the four**: *"available for all paid Copilot plans"*, unavailable in repositories owned by managed user accounts or where explicitly disabled. Per-repository enablement is discoverable via `suggestedActors(capabilities: [CAN_BE_ASSIGNED])` |
| Lifecycle | Enumerated `state`: `queued`, `in_progress`, `completed`, `failed`, `idle`, `waiting_for_user`, `timed_out`, `cancelled` — a **different vocabulary from Jules'**, overlapping but not equal |
| Approval | No pre-execution plan gate. The provider's gate is **post-hoc human code review on a draft pull request**, with iteration via PR comments |
| Progress | Task state plus session logs. GraphQL assignment additionally requires a `GraphQL-Features` header (`issues_copilot_assignment_api_support`, `coding_agent_model_selection`) |
| Changes | The PR diff, via GitHub's own pull-request APIs — a **remote reference**, not inline patch content |
| Result | A draft pull request |
| Delivery | Poll for task state; GitHub's issue/PR event surfaces for the rest |
| Discovery | No capability manifest. Capability is inferred from `suggestedActors` — a **feature-flagged GraphQL query**, not a capabilities endpoint |
| Continuity | Comment on the PR, or create a new task. There is no `sendMessage`-equivalent on the task surface |
| Notable | Execution rides on GitHub Actions. The separate Copilot SDK exposes a third interaction model again — client sessions with `sendAndWait` and an `on_permission_request` callback, i.e. an **in-flight tool-permission** prompt that is neither Jules' plan gate nor Copilot's PR review |

---

## 3. Heterogeneity matrix

The axes on which these four genuinely diverge. "✓" means present; "✗" means
absent from the documented surface.

| Axis | Jules | Cursor | Replit | Copilot |
| :-- | :-- | :-- | :-- | :-- |
| Exposed work item | Session | Agent **+** Run | **none** | Task / Issue |
| Work-item count per unit of work | 1 | **2 levels** | 0 | 1 |
| Identifier style | hierarchical + flat | opaque, **format changed across versions** | `replId` | task id / issue number |
| Auth mechanism | API-key header | Basic **or** Bearer, service accounts, sub-tokens | OAuth PRM discovery | user-to-server token only |
| Capability discovery | none | none | MCP `tools/list` | feature-flagged GraphQL query |
| Entitlement endpoint | **none** | **none** | implicit scope | plan + repo policy, partly queryable |
| Approval mechanism | opt-in plan gate | **none** | **none** | post-hoc PR review (+ SDK tool prompts) |
| Progress telemetry | typed activity stream | SSE typed event stream | **none** | task state + logs |
| Change inspection | **inline patch** | **branch reference** | **none** | **PR reference** |
| Result kind | pull request | branch / PR | **published app URL** | draft pull request |
| Update delivery | poll | snapshot + **stream** + push | **snapshot only** | poll |
| Resumable observation | n/a | `Last-Event-ID`, **retention-limited** | n/a | n/a |
| Documented stability | `v1alpha` | v1 public beta **+** concurrent v0 | unversioned tools | public preview **+** feature flags |

Three of four shapes have a run-like unit; one has none. Three of four have no
capability-discovery endpoint. Two of four expose no approval mechanism, and the
two that do expose *different kinds* of approval. All four use different
auth mechanisms. **No axis is common to all four.**

---

## 4. Findings and the revisions they forced

Each finding names the draft assumption it falsified, the evidence, and the
revision now carried in contract v1.1. These are the "contract changes required
by validation" that Issue #9 asks for.

### F1 — There is no universal work-item topology

- **Draft assumption:** the contract's root is a collection of work items;
  `work.list` is structural.
- **Falsified by:** Replit exposes **no** run/session/task concept — its unit is
  a long-lived `App`, and `update_app_using_prompt` mutates it. Meanwhile Cursor
  needs **two** levels, and changed from one to two across API versions.
- **Revision:** `work.*` is a capability group, not the contract's root. The root
  is the capability manifest (§7). `work.list` may lawfully return `Absent`, and
  `Absent` is not a defect. Contract §5.3, §11.

### F2 — Change inspection is not one thing

- **Draft assumption:** `changes.inspect` returns a diff.
- **Falsified by:** four different fulfilments — Jules hands over
  `unidiffPatch` **inline**; Cursor exposes only a **branch reference**;
  Copilot exposes only a **PR reference**; Replit exposes **nothing**.
- **Revision:** `ChangeView.kind ∈ { inline-patch, remote-reference,
  live-artifact, none }`, declared per provider. `none` is a legal, honest
  answer and triggers handoff rather than an approximation. A mandatory diff
  would have disqualified two of four shapes, or forced a
  lowest-common-denominator contract that hides Jules' richer surface.
  Contract §5.5.1.

### F3 — Approval is not one concept

- **Draft assumption:** a single `approve()` with a normalized decision enum.
- **Falsified by:** three categorically different mechanisms — Jules' **opt-in
  pre-execution plan gate** (`requirePlanApproval`, `:approvePlan`); Copilot's
  **post-hoc human review of a draft PR**; the Copilot SDK's **in-flight tool
  permission** callback (`on_permission_request`). Cursor and Replit expose
  **none**.
- **Revision:** `ApprovalRequest.providerKind` and `permittedDecisions` are
  provider vocabularies. Greenfield2 defines no normalized approval type and no
  universal decision set, and must not invent an approval mechanism for a
  provider that has none. `approval.*` is `required-if-exposed`, not
  `required`. Contract §5.6, P9.

### F4 — There is no universal result artifact

- **Draft assumption:** the result envelope has a `pullRequest` field.
- **Falsified by:** Replit's result is a **published app and its public URL**,
  which is not a pull request in any sense. Jules, Cursor and Copilot are
  PR-shaped; Replit is not.
- **Revision:** `ResultView { providerResultKind, handle, summary? }` with a
  provider-declared kind string. No canonical result enum — a canonical list
  would rank providers by resemblance to whichever provider suggested it.
  Contract §5.5.2.

### F5 — Update delivery differs categorically, including within one provider

- **Draft assumption:** the contract offers a subscription primitive.
- **Falsified by:** Cursor alone uses **three** mechanisms (snapshot/poll, SSE
  stream, legacy webhooks). Replit has **snapshot only**. Jules has **poll**.
  Copilot has poll plus GitHub's own event surfaces.
- **Revision:** `delivery ∈ { snapshot, poll, stream, push }` declared **per
  capability**, with `maxStaleness`. `stream` is never required. Transport
  tokens are banned above the adapter (P4). Contract §6.

### F6 — Entitlement discovery differs categorically, and is often absent

- **Draft assumption:** `discovery.entitlements` reads a provider entitlement
  endpoint and returns a boolean per capability.
- **Falsified by:** Jules and Cursor have **no** entitlement endpoint. Replit's
  entitlement is **implicit** in OAuth scope (`list_apps` silently returns only
  editable apps). Copilot's is real but composite (paid plan + repository policy
  + `suggestedActors`).
- **Revision:** entitlement becomes **tri-state** — `entitled | not-entitled |
  unknown` — with `unknown` mandatory and distinct from denial, plus an
  **evidence kind** so the UI can state how sure it is. L2 is never cached as
  permanent. Contract §3.2.

### F7 — Continuity is not one operation

- **Draft assumption:** a single `resume()`.
- **Falsified by:** Jules resumes by `:sendMessage` into
  `AWAITING_USER_FEEDBACK`; Cursor **cannot** resume a cancelled run (*"create a
  new run on the same agent"*); Replit has no work item to resume and mutates
  the app instead; Copilot resumes by commenting on a PR or creating a new task.
- **Revision:** `continuity.reconnect` is provider-native, `continuity.replay`
  is separate and optional, and Greenfield2 promises only what the provider
  supports. Reconnect/replay is explicitly **not** Greenfield2 execution
  recovery. Contract §5.7.

### F8 — Identifier formats are heterogeneous *and* unstable within a provider

- **Draft assumption:** references can be validated or lightly parsed.
- **Falsified by:** Cursor changed its own id format between versions
  (`bc_abc123` → `bc-<uuid>`) and its artifact path semantics with it; Jules
  carries both a hierarchical `name` and a separate flat `id` for the same
  session; Cursor documents its stream event ids as *"an opaque string you
  should not parse"*.
- **Revision:** `ProviderResourceReference.ref` is opaque and must never be parsed, split,
  composed or prefix-matched (P10). `kind` is display vocabulary and never a
  branch condition in core (P8). Contract §4.2.

### F9 — Capability discovery itself has no universal mechanism

- **Draft assumption:** adapters read a provider capabilities endpoint.
- **Falsified by:** Jules and Cursor have **none**; Replit's is MCP
  `tools/list`, structurally unlike a REST endpoint; Copilot requires a
  **feature-flagged GraphQL query** (`suggestedActors` plus a
  `GraphQL-Features` header).
- **Revision:** the contract defines the **manifest's shape**, not the probe.
  Producing it is an adapter obligation, however the provider allows. Contract
  §7 rule 5, §9.2.

### F10 — Provider vocabularies drift, and unrecognized values are normal

- **Draft assumption:** provider states and kinds can be enumerated in
  Greenfield2 types.
- **Falsified by:** documented Jules state drift (both `CANCELLED` and
  `CANCELED`; `COMPLETED_UNKNOWN` in the field) **[SECONDARY]**; Cursor shipping
  `v0` and `v1` concurrently with different semantics; the Copilot API being
  public preview behind feature headers.
- **Revision:** every provider-supplied token has a defined path for "value
  Greenfield2 has never seen", which is `unknown` — never a crash, never a
  silent substitution. This is what makes the §4.5 liveness hint's mandatory
  `unknown` non-optional. Contract §4.5, §8.

---

## 4b. Findings from independent review of PR #11 (F11–F15)

F1–F10 came from comparing the contract against provider shapes. These five came
from comparing the contract against **Greenfield2's own accepted product and
domain truth**, which the first pass read but did not check the contract
against clause by clause. They are recorded here rather than quietly patched,
because each one is a defect a future session could reintroduce.

### F11 — The contract relaxed an accepted MVP product requirement

- **Defect:** §5.5.1 said a provider with no change-inspection surface *"may
  still qualify for the MVP loop if the rest of the loop is strong and
  Greenfield2 hands off for change review"*, and the §5 preamble said a provider
  needs *"enough of the `loop` group"*.
- **Falsified by:** [PRODUCT.md](../PRODUCT.md) minimum V1 item 4 — *"inspect
  meaningful provider-exposed changed files and/or diffs"* — carries **no**
  "where exposed" qualifier, unlike items 1, 3, 5 and 7. A contract has no
  authority to relax an accepted product requirement.
- **Revision:** §5 now carries an explicit item-by-item mapping that reproduces
  PRODUCT.md's qualifiers exactly and states that PRODUCT.md wins on
  disagreement. §5.5.1 now states that `none` **fails** Product Fit (Gate 1).
  `none` stays representable as honest L1 truth — the correction is about
  qualification, not about hiding the capability. The same reading resolves
  the `live-artifact` question: PRODUCT.md item 4 says *"changed files and/or
  diffs"*, so running or published output is not change inspection and does
  not satisfy item 4 on its own — though it does count under item 6 via
  `result.observe`. That followed from the accepted wording and needed no new
  product decision.

### F12 — `unknown` entitlement was declared but never defined

- **Defect:** §3.2 declared `entitlement ∈ { entitled, not-entitled, unknown }`
  and said `unknown` must be "surfaced as uncertainty", but never said what
  Greenfield2 **does**. The §3.1 disposition table had no `unknown` row, so
  every adapter would have invented its own behaviour.
- **Revision:** §3.1 gains two `unknown` rows. §3.2.1 defines the behaviour and
  splits it by risk: read-only capabilities may be invoked optimistically, while
  **mutating or spend-bearing** capabilities are never invoked speculatively,
  never auto-retried after a refusal, and must show the unconfirmed entitlement
  *before* the user commits. §3.2.2 enumerates the five entitlement evidence
  kinds. The manifest gains a `mutates` flag so adapters must declare this
  truthfully.

### F13 — The contract could not express required provider-native invocation inputs

- **Defect:** `work.start(connection, intent)` carried only a provider-neutral
  instruction. Nothing could express that a provider requires specific
  provider-owned inputs before it will accept the request.
- **Falsified by:** Replit's `create_app_from_prompt` requires **both**
  `appDescription` **and** `app_stack`, from a fixed provider-defined list.
  Jules requires `sourceContext` for non-repoless sessions. Cursor requires
  `repos[].url` on every entry plus a mutually exclusive execution-environment
  choice. Copilot addresses work by `{owner}/{repo}`.
- **Revision:** new §5.9 defines `RequiredInput` and `InvocationContext`; the
  manifest gains `requiredInputs`; `work.start` and `work.continue` take an
  `InvocationContext`. Provider choice lists stay provider vocabulary (not
  renamed), provider defaults are passed through rather than replaced, and a
  missing required input is explicitly **not** `Absent`.

### F14 — Provider Connection and Provider Resource Reference were one type

- **Defect:** a single `ProviderHandle` type was returned by
  `connection.authorize` *and* used for every provider resource reference.
- **Falsified by:** [DOMAIN.md](../DOMAIN.md) defines **two** separate
  Greenfield2-owned concepts — *Provider Connection* ("the authorized
  relationship/reference between a Greenfield2 Account and one supported
  provider account") and *Provider Resource Reference* ("an opaque reference …
  to reopen or navigate to a provider-owned resource") — with different
  authority and different invariants.
- **Revision:** §4 split into §4.1 Provider Connection, §4.2 Provider Resource
  Reference and §4.3 a comparison of why they must stay separate. The connection
  legitimately carries a Greenfield2-owned lifecycle state; the resource
  reference does not. All capability shapes updated, and adapter obligation #9
  forbids returning one where the other is meant. Collapsing them fails in both
  directions: the second failure mode is granting Greenfield2 a lifecycle over
  provider-owned resources, which is the shadow-state risk DOMAIN.md prohibits.

### F15 — "Verbatim" error pass-through contradicted the safe-presentation requirement

- **Defect:** §8 required `ProviderRefusal` to carry the provider's message and
  §5.8 said errors pass through "formatted for readability **only**".
- **Falsified by:** [PRODUCT.md](../PRODUCT.md) requires formatting "for
  readability **and safe presentation**". Provider free text is untrusted input:
  it can carry markup or script, embedded URLs, credential-shaped substrings,
  personal data, and instruction-shaped content aimed at an assistant.
- **Revision:** §8 now separates the two obligations. **§8.1** — error
  *semantics* (code, category, retryability, entitlement vs quota vs policy,
  provider-stated remedy) pass through unaltered and Greenfield2 adds no
  execution state machine. **§8.2** — *presentation* is sanitized against a
  named risk table, and sanitizing is explicitly **not** reinterpretation.
  **§8.3** adds the retry and spend rules. Where a provider exposes no
  structured code, adapters report `unclassified` rather than guessing, because
  guessing is reinterpretation.

### Process correction alongside F11–F15

ADR-0006 was committed with `**Status:** accepted` while sitting on an unmerged
branch. [`.ecc/skills/decisions.md`](../../.ecc/skills/decisions.md) requires
*"Status is honest. `proposed` until it is actually in force."* It was changed
to `proposed` and the ADR index agreed; nothing relied on it as settled. (The
product owner accepted ADR-0006 later the same day, so it is now `accepted` —
see §4c.)

---

## 4c. Consistency corrections from the second review pass

Four internal inconsistencies remained after F11–F15. None changed the
architecture; each was a place where one section had been updated and another
had not.

| # | Inconsistency | Correction |
| :-- | :-- | :-- |
| C1 | §3 still stated `effective = L1 ∧ L2 ∧ L3` — **boolean** notation over a layer §3.2 had just made **three-valued**. Under that formula `unknown` would silently evaluate as `false`, contradicting §3.2.1 and the §3.1 table. | §3 now defines a total `resolution(...)` function over L1 × L2 × L3 whose six outcomes match the §3.1 disposition table row for row, with a note that `unknown` must never be collapsed into either boolean value. |
| C2 | §2 still described `ProviderRefusal` as carrying the provider's error *"verbatim"* — the exact wording F15 removed from §8. | §2 now says it carries provider **semantics unaltered** in a **sanitized presentation**, pointing at §8.1/§8.2. |
| C3 | `live-artifact` was left as an open product question, but PRODUCT.md already answers it. | Resolved from the accepted wording: item 4 says *"changed files and/or diffs"*, so running or published output is **not** change inspection and does not satisfy item 4 alone. It still counts under **item 6** via `result.observe`. No new product decision was needed; contract §5.5.1 and §11 updated. |
| C4 | ARCHITECTURE.md listed ADR-0006 under *"decisions — decided"* and called first-provider selection *"now unblocked"* in three places, while the ADR was `proposed`. | ADR-0006 was moved to a "proposed, not yet in force" section and every consequence was re-worded to blocked. **Superseded by the acceptance recorded below**: the product owner has since accepted ADR-0006, so the section was folded back into "decided" and Issue #8 is genuinely open. The lesson stands independently of the outcome. |
| **C5** | The §3.1 disposition table gave the `unknown`-entitlement row `L3 = –` (any), which **overlapped** the separate `L3 = ✗` row and contradicted `resolution()` in §3, where `unconfirmed` requires `L3` to be true. | Row corrected to **`L1=✓, L2=unknown, L3=✓ → unconfirmed`**. The six rows now map one-to-one onto the six `resolution()` outcomes, covering all twelve L1 × L2 × L3 combinations exactly once, with no overlap and no gap. |

C4 was the one with teeth, and C5 is its mirror image inside a table. Marking
an ADR `proposed` is not enough if the surrounding documents keep describing its
consequences as already in force — the status field becomes decoration. Equally,
a summary table must be re-derived whenever the formal definition beside it
changes: the `unknown` row drifted out of step with `resolution()` the moment
`resolution()` was introduced in C1, and nothing caught it until review.

#### Status of C4 after acceptance

The product owner accepted ADR-0006 on 2026-09-13, so the C4 correction has been
**reversed in the correct direction**: ADR-0006 is `accepted`, the contract row
sits in ARCHITECTURE.md's "decided" table, and Issue #8 is open *as a
consequence of that acceptance* rather than by assumption. C4 stays in this log
because the failure mode it describes does not depend on which way the status
happens to point.

---

## 5. Vendor-leakage audit

Issue #9 requires confirmation that no single provider's nouns or lifecycle
became Greenfield2 universal semantics.

### 5.1 Noun audit

Every provider noun observed in the validation set, and where it lives.

| Provider noun | Provider | Greenfield2-owned entity? | Where it appears |
| :-- | :-- | :-- | :-- |
| `Session` | Jules | **No** | Provider profile only; carried as `ProviderResourceReference.kind = "Session"` |
| `Activity`, `Artifact`, `Source`, `Plan` | Jules | **No** | Provider profile only |
| `Agent`, `Run`, `Worker`, `Pool` | Cursor | **No** | Provider profile only |
| `App`, `Repl` | Replit | **No** | Provider profile only |
| `Task`, `Issue`, `Pull Request` | Copilot | **No** | Provider profile only |

Greenfield2-owned vocabulary is unchanged from
[DOMAIN.md](../DOMAIN.md): **Greenfield2 Account, Provider Connection,
Capability Support, Greenfield2 UI Preference, Provider Resource Reference.**
This contract introduces **no** new Greenfield2-owned entity. The only new
Greenfield2-owned *names* are capability identifiers (`work.start`,
`progress.observe`, …), which are verbs-phrased questions, not nouns, and none
of which is a provider term.

### 5.2 Lifecycle audit

| Provider lifecycle vocabulary | Adopted by Greenfield2? |
| :-- | :-- |
| Jules `SessionState` (9 values) | **No.** Passes through verbatim as `providerState`. |
| Cursor run status (`RUNNING`, `FINISHED`, `CANCELLED`, …) | **No.** Passes through verbatim. |
| Copilot task `state` (8 values) | **No.** Passes through verbatim. |
| Replit (no exposed lifecycle) | **No.** Greenfield2 did not invent one to fill the gap. |

The only Greenfield2-side lifecycle is the §4.5 presentation liveness hint
(`active | awaiting-user | settled | unknown`). It is lossy, presentation-only,
non-authoritative, never persisted as provider state, and carries a mandatory
`unknown`. It exists so a phone screen can choose an affordance; it is not a
state machine and it never decides whether an action is permitted.

### 5.3 Counterfactual check — what a first-vendor-shaped contract would have said

The test ADR-0005 exists to force. Had any single provider been designed first,
the contract would plausibly have inherited:

| If designed from… | The leakage would have been | Actually present in v1.1? |
| :-- | :-- | :-- |
| Jules | a required plan-approval step; `Activity`/`Artifact` as core types; inline diff as *the* change model; polling as the only delivery | **No** — approval is optional and provider-typed (F3); change inspection has four fulfilments (F2); delivery is declared (F5) |
| Cursor | a two-level Agent/Run hierarchy; SSE as the observation primitive; artifact paths as a typed contract | **No** — no hierarchy is assumed (F1); `stream` is optional (F5); `ref` is opaque and unparsed (F8) |
| Replit | an App-centric contract with no work item at all; no progress or approval concept | **No** — `work.*` and `progress.*` exist but tolerate `Absent` (F1); `approval.*` is `required-if-exposed` (F3) |
| Copilot | repository-scoped paths; PR review as *the* approval model; paid-plan entitlement as the universal gate; issue assignment as the entry point | **No** — no repository scoping in the contract; approval is provider-typed (F3); entitlement is tri-state with an evidence kind (F6) |

### 5.4 Reproducible checks

```bash
# A — no Greenfield2-owned provider nouns
grep -rniE 'greenfield2[- ](owned|canonical)[^|]{0,40}\b(session|run|task|agent|workspace|project|app|artifact|activity|repl)\b' docs/

# B — no transport vocabulary above the adapter
grep -rnE '\b(REST|SSE|WebSocket|webhook|GraphQL)\b' docs/architecture/PROVIDER_CAPABILITY_CONTRACT.md
```

Both were executed when this record was written. Check A returned **no matches**
(exit 1). Check B returned **four** matches, all inside the contract's own §1
"not a transport specification" row, §6 delivery rules, the P4 prohibition, and
the audit command itself — i.e. all in the places that *state the prohibition*.
Check B is case-sensitive on purpose: a case-insensitive `rest` also matches
ordinary English prose such as "the rest of the loop".

---

## 6. Conclusion

- The contract survived contact with four materially different shapes, **after
  fifteen revisions**. That revision count is the useful signal: an unrevised
  contract validated against three providers would more likely have been a
  first vendor's shape with the serial numbers filed off.
- The revisions fall into **two distinct clusters**, and the difference matters.
  **F1–F10** came from comparing the contract against provider shapes, and they
  share one theme: **every apparent universal was actually a majority case.**
  Work items, diffs, approvals, results, streaming and entitlements each looked
  universal until a shape arrived that lacked them or meant something different
  by them. **F11–F15** came from comparing the contract against Greenfield2's
  *own* accepted product and domain truth, and they share a different theme:
  **provider-neutrality was being pursued at the cost of product fidelity.** The
  contract relaxed an accepted MVP requirement, left `unknown` undefined, could
  not express mandatory provider inputs, merged two DOMAIN.md entities, and
  preferred verbatim error text over safe presentation. Passing the
  provider-shape test did not catch any of those.
- The contract's spine is therefore *declared capability over assumed
  structure*: a manifest that states what exists, opaque resource references
  that refuse to interpret provider identity, three independently-sourced
  availability layers, and `Absent` as a first-class, non-exceptional answer —
  bounded by the accepted product requirements it exists to realize, not
  narrowed by them.
- **No provider is selected, ranked, scored or recommended by this document.**
  First-provider selection is a separate decision belonging to
  **[Issue #8](https://github.com/anthracite-labs/Greenfield2/issues/8)**, using
  the Product Fit and Integration Legitimacy gates in
  [PRODUCT.md](../PRODUCT.md). With
  [ADR-0006](../decisions/0006-capability-manifest-and-three-layer-availability.md)
  accepted, ADR-0005's precondition — a validated contract — is discharged and
  Issue #8 is open. Nothing in this document selects a provider.

### 6.1 What this validation does *not* establish

Stated plainly, so it is not over-read:

- **No provider API was called.** This is documentation-level validation, not
  integration testing. Real surfaces will disagree with their own docs at the
  edges; ADR-0005 anticipates further revision.
- **Four shapes is a floor, not a ceiling.** Three were required; a fifth shape
  could still falsify a remaining assumption, particularly around
  multi-repository work, self-hosted execution, or provider-to-provider
  composition.
- **Capability claims were not exercised.** Whether a documented endpoint
  actually behaves as documented for a given account is an integration question.
- **Provider terms may have changed since 2026-09-13.** Re-verify before relying
  on any specific endpoint in the Issue #8 provider-selection evaluation.

## Related

- [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) — the contract this validates
- [../ARCHITECTURE.md](../ARCHITECTURE.md) — Architecture mandate and open decisions
- [../PRODUCT.md](../PRODUCT.md) — MVP capability loop and the two selection gates
- [../decisions/0005-provider-contract-before-provider-selection.md](../decisions/0005-provider-contract-before-provider-selection.md) — why validation precedes selection
- [../decisions/0006-capability-manifest-and-three-layer-availability.md](../decisions/0006-capability-manifest-and-three-layer-availability.md) — the structural decision
