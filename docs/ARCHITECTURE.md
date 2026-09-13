# Architecture

**Status:** Architecture phase active; no application Architecture decision has been accepted yet.  
**Lifecycle:** `PROJECT_PHASE=architecture`, with `ALLOW_APP_STACK=0` still enforcing the no-stack guard.

This file serves two purposes during Architecture:

1. preserve the engineering-system architecture inherited from App-Factory; and
2. record Greenfield2 application Architecture as alternatives are researched and durable choices are accepted through ADRs.

Entering Architecture does **not** itself select a provider, framework, database, auth implementation, hosting target, protocol, UI technology, or application stack.

## Architecture mandate

Architecture must determine **how** to realize the reviewed product definition in [PRODUCT.md](PRODUCT.md) without reopening its accepted **what, for whom and why**.

The work now permitted includes:

- evaluating candidate first providers against the accepted product-fit and integration-legitimacy gates;
- defining Greenfield2/application boundaries while preserving provider-authoritative development state;
- defining provider-adapter responsibilities and capability-discovery contracts;
- evaluating authentication/authorization integration patterns without assuming authentication implies entitlement;
- evaluating data/cache/reconnect designs consistent with the minimum-data rule and provider truth;
- evaluating mobile/larger-screen implementation approaches;
- evaluating application stack, hosting and persistence alternatives;
- defining the testing strategy and future stack-specific CI expectations;
- recording durable technical trade-offs as ADRs.

Research from VibeFlow, Replit, Happier and other sources may inform alternatives. It does not become Architecture truth until a Greenfield2 review explicitly promotes it.

## Architecture invariants from Discovery

Architecture must preserve these reviewed product boundaries unless a separate product amendment explicitly changes them:

- Greenfield2 owns no provider development state.
- Provider-native resources, terminology, lifecycle and execution state remain provider-authoritative.
- Greenfield2 may own only its own account/interface/connection/support/reference metadata and the minimum permitted cache/reconnect state a real feature requires.
- MVP is bring-your-own-provider and one connected provider account.
- Production capability support requires both a supported external-client path and legitimate user entitlement.
- Provider actions are pass-through; Greenfield2 adds no independent approval/execution authority in MVP.
- Provider truth governs autonomy, durability, recovery, completion and verification in MVP.
- Greenfield2 does not fabricate a universal provider domain model.
- Mobile is the priority product surface, but the product is not mobile-only.
- `ALLOW_APP_STACK=0` remains active until an accepted application-stack ADR and the separate implementation transition satisfy the lifecycle guard.

## Application Architecture decisions — currently open

No item in this section is selected merely by being listed.

| Area | Architecture question | Decision vehicle |
| :-- | :-- | :-- |
| First provider | Which eligible provider best satisfies the MVP capability loop and legitimate third-party integration requirements? | Provider-evaluation issue; ADR if the choice creates durable coupling |
| Client/application stack | Which approach best supports phone-first and larger-screen surfaces while respecting the repository constraints? | Application-stack ADR |
| Provider boundary | What adapter/capability contract keeps provider-native semantics authoritative without duplicating provider state? | Architecture issue / ADR if durable |
| Identity/auth | How does Greenfield2 Account connect to provider authorization while keeping entitlement distinct? | Security/Architecture review + ADR if durable |
| Persistence/cache | What minimum Greenfield2-owned persistence is actually required for account, connection, preferences and opaque references? | Data Architecture ADR if durable |
| Reconnect/events | Which provider-supported event, polling or refresh patterns are valid for the selected provider? | Provider-specific Architecture decision |
| Testing | What unit/integration/contract/UI tests are required before implementation and what will CI enforce? | Architecture issue, then stack-specific gate work |
| Hosting/deployment | What Greenfield2-owned services, if any, are required and where should they run? | ADR after alternatives are known |

## Operating model

> **ECC is repository-owned, Arena-executed, ChatGPT-supervised.**

```text
Human product owner
        ↓
ChatGPT — planning / architecture / independent PR review
        ↓
GitHub — durable source of truth
        ↓
Arena Agent Mode — developer / executor
        ↓
ECC-on-Arena repository adapter
        ↓
branch → tests → PR → CI → ChatGPT review → merge
```

## The engineering system that exists

```text
                    ┌─────────────────────────────┐
                    │  ChatGPT (independent layer)│
                    │  planning input, PR review  │
                    └──────────────┬──────────────┘
                                   │ reviews the real diff
                                   ▼
┌──────────────────────────────────────────────────────────────────┐
│  GitHub  (durable source of truth)                               │
│  issues · branches · PRs · Actions · default-branch ruleset      │
└───────────────────────────────┬──────────────────────────────────┘
                                │ clone / push / gh api
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  Arena Agent Mode sandbox (ephemeral)                            │
│  reads .ecc/BOOTSTRAP.md → loads 1–2 skills → plans → implements │
│  → reviews → runs scripts/verify.sh → commits → opens PR         │
└───────────────────────────────┬──────────────────────────────────┘
                                │ only committed files survive
                                ▼
                    ┌─────────────────────────────┐
                    │  .ecc/  (repository-owned)  │
                    │  rules · skills · roles ·   │
                    │  VERSION · UPSTREAM.md      │
                    └─────────────────────────────┘
```

## Component responsibilities

| Component | Responsibility | Deliberately does not |
| :-- | :-- | :-- |
| `FOUNDATION_VERSION` | The App-Factory template version of this repository | Track the product or the ECC version |
| `.ecc/BOOTSTRAP.md` | Small, complete session protocol | Duplicate rules; load every workflow |
| `.ecc/rules/` | Standing, always-in-force rules | Describe task procedure (that is skills) |
| `.ecc/skills/` | On-demand task workflows, routed by `INDEX.md` | Auto-load; run without being read |
| `.ecc/roles/` | Sequential review personas for one agent | Pretend to be parallel subagents |
| `.ecc/VERSION`, `.ecc/UPSTREAM.md` | Upstream ECC provenance and licence record | Vendor upstream code |
| `config/project.env` | Committed lifecycle state (phase, stack guard) | Hold secrets or be sourced by a shell |
| `config/main-ruleset.json` | Portable branch-protection intent | Apply itself; contain instance ids |
| `docs/PRODUCT.md` | Reviewed product definition and Architecture constraints | Choose technical implementation |
| `docs/DOMAIN.md` | Minimal Greenfield2-owned product vocabulary | Normalize provider-native resources into Greenfield2 ownership |
| `docs/MEMORY.md` | Durable cross-session memory | Replace ADRs or PR descriptions |
| `docs/decisions/` | Durable trade-off records | Track task state |
| `docs/FACTORY.md` | Instantiation and admin checklist | Automate GitHub administration |
| `scripts/verify.sh` | Deterministic quality gate | Test application behaviour before an application exists |
| `scripts/selftest.sh` | Negative tests that prove the gate can fail | Modify the real working tree |
| `scripts/init-project.sh` | One-time, non-destructive project identity setup | Commit, push, or change GitHub settings |
| `scripts/bootstrap.sh` | Session briefing from repository state | Mutate anything |
| `scripts/sync-ecc.sh` | Upstream inspection and diff preparation | Overwrite local adaptations |
| `.github/workflows/verify.yml` | Independent execution of the same gate | Trust an agent's self-report |

## Lifecycle as repository state

The foundation's no-stack guard is not a permanent property of the gate; it is a function of committed configuration.

```text
config/project.env                 scripts/verify.sh
──────────────────                 ─────────────────
PROJECT_PHASE=architecture   ──▶   check_lifecycle   validates the phase
ALLOW_APP_STACK=0            ──▶   check_no_app_stack rejects stack artifacts
STACK_DECISION_ADR=

           ── accepted application-stack ADR + reviewed transition ──▶

PROJECT_PHASE=implementation ──▶   check_lifecycle   accepts: phase + ADR agree
ALLOW_APP_STACK=1            ──▶   check_no_app_stack stands down (SKIP)
STACK_DECISION_ADR=docs/decisions/NNNN-....md
```

A single shared helper, `validate_stack_transition`, decides whether the guard may stand down, and **both** `check_lifecycle` and `check_no_app_stack` call it. Neither check trusts the other to have run, so `verify.sh --only=no_app_stack` reaches the same verdict as a full run — a check that stands down because it assumed another check validated the state is not a guard.

The transition to implementation is rejected unless `ALLOW_APP_STACK=1`, `PROJECT_PHASE=implementation`, and `STACK_DECISION_ADR` all agree **and** the referenced ADR exists, is not the `0000-template.md` skeleton, carries `**Decision Type:** application-stack`, and is marked `**Status:** accepted`. Existing on disk is not approval. Both directions, and every rejection path, are covered by negative tests in `scripts/selftest.sh`.

## Design principles

1. **Small startup context.** Bootstrap plus the always-read set is a few pages. Everything else is loaded only when the task needs it.
2. **Everything durable is in Git.** The sandbox dies between sessions; uncommitted knowledge does not exist.
3. **Verification is a program, not a promise.** A committed script with a non-zero exit path, re-run by CI, is the only accepted evidence of quality.
4. **A gate that never fails proves nothing.** `scripts/selftest.sh` injects faults into a throwaway copy and asserts the gate rejects each one.
5. **Adapt, do not import.** Upstream ECC is large; this adapter carries 10 workflows, 4 rules, and 3 personas, and never claims to be native ECC.
6. **State, not surgery.** Lifecycle changes are config diffs reviewed in a PR, never edits to the script that enforces them.
7. **Reviewed product first.** Architecture starts from `docs/PRODUCT.md` and `docs/DOMAIN.md`; technical convenience does not silently redefine product authority or scope.
8. **Alternatives before commitment.** Research real options and costs before accepting a durable Architecture choice.
9. **Implementation stays locked.** Architecture may decide; application-stack artifacts wait for the accepted stack ADR and implementation transition.

## Execution environment

See [ARENA.md](ARENA.md) for the concise, generic harness reference and the re-verification instruction. Environment facts are project-local and must be re-checked rather than inherited as guarantees.

## Related

- [PRODUCT.md](PRODUCT.md) — reviewed product definition and Architecture constraints
- [DOMAIN.md](DOMAIN.md) — accepted product vocabulary and authority semantics
- [ROADMAP.md](ROADMAP.md) — lifecycle stages
- [SECURITY.md](SECURITY.md) — repository security floor; application threat model evolves with Architecture
- [decisions/](decisions/README.md) — decision record index
- [FACTORY.md](FACTORY.md) — instantiation and repository-admin reference
- [../.ecc/UPSTREAM.md](../.ecc/UPSTREAM.md) — ECC provenance and omissions
