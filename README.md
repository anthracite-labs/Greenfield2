# Greenfield2

**Status:** App 1 Architecture; the reviewed Discovery product definition is canonical.  
**Engineering foundation:** App-Factory `0.1.0`, using the repository-owned ECC-on-Arena adapter derived from ECC v2.2.0 (MIT).

Greenfield2 is an instantiated application repository built on the App-Factory engineering foundation. Discovery is complete and the authoritative lifecycle is now `PROJECT_PHASE=architecture`; application implementation remains locked while Architecture evaluates and records technical choices through ADRs.

## Product in one paragraph

Greenfield2 is a **mobile-first universal software-development interface** for technical builders. It connects to a user's supported software-development provider account, discovers the capabilities and resources that provider officially exposes, and presents those capabilities through one coherent Greenfield2 UI.

The defining authority rule is:

> **Greenfield2 owns no provider development state. The connected provider remains authoritative for provider-owned resources, capabilities, runtime, sessions, files, artifacts, actions, lifecycle, entitlements, errors, approvals and results that Greenfield2 exposes.**

For MVP, Greenfield2 proves the model with one user-supplied full-stack provider. Architecture must first define and validate Greenfield2's provider-neutral capability boundary across materially different provider shapes; only then may it select the first proving provider. No provider is selected merely by being a research or validation candidate.

Read in this order when orienting to the repository:

1. [`AGENTS.md`](AGENTS.md) — engineering operating rules and ECC-on-Arena entry point.
2. [`README.md`](README.md) — repository identity and current lifecycle.
3. [`FOUNDATIONS.md`](FOUNDATIONS.md) — product constitution, evidence hierarchy and protected-foundation boundary.
4. [`config/project.env`](config/project.env) — machine-checked lifecycle state.
5. [`docs/MEMORY.md`](docs/MEMORY.md) — append-only verified project memory.
6. [`docs/PRODUCT.md`](docs/PRODUCT.md) — reviewed product definition and V1 boundary.
7. [`docs/DOMAIN.md`](docs/DOMAIN.md) — minimal Greenfield2-owned vocabulary and provider-authority rules.
8. [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — Architecture-stage scope, alternatives and engineering foundation.

## Reference model

Greenfield2 was defined using several references deliberately, without letting any one reference silently become the product:

- **VibeFlow** informs the interoperability/provider-neutral thesis and supported provider/client boundary.
- **Replit research** is clean-room evidence for coherent mobile software-building interactions and capability completeness; it is not a clone specification.
- **Happier** is an OSS architecture/feasibility reference for provider catalogs, cross-device supervision and reconnect patterns; its product/session authority is not inherited.
- **Other OSS/upstream research** remains evidence or an ADOPT/HARVEST/REJECT candidate until Architecture explicitly promotes it through review.

See [`FOUNDATIONS.md`](FOUNDATIONS.md) for the evidence and promotion rules.

## MVP boundary

The first MVP is deliberately narrow:

- primary user: technical builder already using an eligible agentic software-development provider;
- mobile-first, not mobile-only;
- bring-your-own-provider only;
- one Greenfield2 account → one connected provider → one provider-authoritative development experience;
- one complete workspace-level development loop: start/continue work, interact with the provider Agent, observe progress/activity, inspect meaningful files/diffs, handle provider approvals where exposed, see a meaningful provider result, and reconnect/continue where supported;
- provider-native terminology and semantics remain intact;
- Greenfield2 exposes only capabilities available through a supported external-client path with legitimate entitlement;
- provider truth only for autonomy, durability, recovery, completion and verification in MVP;
- minimum Greenfield2 data retention and no central shadow copy of provider content.

The full scope, constraints, non-goals and success measures are in [`docs/PRODUCT.md`](docs/PRODUCT.md).

## Lifecycle

The authoritative state is [`config/project.env`](config/project.env):

```text
factory  →  discovery  →  architecture  →  implementation
                              ▲
                        Greenfield2 now
```

Discovery review is complete. Architecture may now evaluate real alternatives and record durable technical choices as ADRs, but `ALLOW_APP_STACK=0` remains active.

No application framework, database, auth implementation, hosting target, provider implementation, protocol, or UI stack may be introduced merely because research suggests one. Those choices belong to the Architecture/ADR process, and application code remains locked until the later implementation transition satisfies the repository gate.

## Engineering operating model — preserved from App-Factory

> **ECC is repository-owned, Arena-executed, ChatGPT-supervised.**

```text
Human product owner
        ↓
ChatGPT — discovery / planning / architecture / independent PR review
        ↓
GitHub — durable source of truth
        ↓
Arena Agent Mode — developer / executor
        ↓
ECC-on-Arena repository adapter
        ↓
branch → tests → PR → CI → ChatGPT review → merge
```

The App-Factory foundation defines **how** work is done. Product and Architecture work do not rewrite the ECC skills, rules, roles, provenance or verification philosophy. Foundation changes require their own issue and review.

## What the foundation provides

| | |
| :-- | :-- |
| **ECC-on-Arena adapter** | Engineering rules, 10 on-demand workflows, 3 review personas, adapted from ECC v2.2.0 (MIT), fully attributed. Not native ECC. |
| **Deterministic gate** | `scripts/verify.sh` — committed checks, non-zero on failure, re-run independently in CI. |
| **Negative tests** | `scripts/selftest.sh` — injects faults into a throwaway copy and asserts the gate rejects them. |
| **Lifecycle state** | `config/project.env` — factory → discovery → architecture → implementation, with a no-stack guard that stands down only via a reviewed, ADR-backed implementation transition. |
| **Portable governance** | `config/main-ruleset.json` — branch-protection policy intent that must be applied to the actual GitHub repository by an authorized human. |
| **Project documentation system** | Product, domain, roadmap, architecture, security, memory, factory and ADR documents. |

## Starting an Arena session

Arena auto-loads nothing. Start with:

> Read `.ecc/BOOTSTRAP.md`, initialize the project engineering protocol, inspect project memory and the skill index, then work GitHub Issue #X. Load only skills relevant to that issue.

Or print the same briefing:

```bash
bash scripts/bootstrap.sh
```

Startup context is intentionally small. Workflows are loaded one or two at a time only when the task calls for them.

## Repository map

```text
AGENTS.md                  engineering entry point
FOUNDATIONS.md             App 1 product constitution and evidence hierarchy
FOUNDATION_VERSION         App-Factory foundation version
.ecc/                      protected ECC-on-Arena adapter
config/project.env         project identity, lifecycle and no-stack guard
config/main-ruleset.json   portable branch-governance intent
docs/PRODUCT.md            reviewed product definition and V1 boundary
docs/DOMAIN.md             minimal product vocabulary and authority semantics
docs/ARCHITECTURE.md       Architecture-stage alternatives + engineering foundation
docs/SECURITY.md           foundation security floor; application extensions later
docs/ROADMAP.md            lifecycle sequencing
docs/MEMORY.md             append-only verified project memory
docs/FACTORY.md            App-Factory instantiation/admin reference
docs/decisions/            architecture decision records
scripts/                   foundation verification/bootstrap/init tooling
.github/workflows/verify.yml   independent CI execution of the gate
```

## Quality gate

```bash
bash scripts/verify.sh
bash scripts/selftest.sh
```

The foundation gate remains authoritative. A future application stack adds its own lint/test/build gates alongside it; it does not replace the foundation gate.

## Foundation provenance and licensing

- `.ecc/` is derived from [Everything Claude Code (ECC)](https://github.com/affaan-m/ECC) v2.2.0, MIT licensed. The upstream notice is committed at [`.ecc/LICENSE-ECC`](.ecc/LICENSE-ECC); details live in [`.ecc/UPSTREAM.md`](.ecc/UPSTREAM.md).
- This is an **adaptation, not native ECC**. Arena has no plugin runtime, slash commands, lifecycle hooks or subagent API, and the repository does not claim otherwise.
- Greenfield2 was instantiated from App-Factory `0.1.0`, whose factory provenance is recorded in [`docs/FACTORY.md`](docs/FACTORY.md) and [`docs/MEMORY.md`](docs/MEMORY.md). Product history from the source foundation is not inherited as App 1 product truth.

## Current work

Architecture work is tracked in **Issue #9**. [ADR-0005](docs/decisions/0005-provider-contract-before-provider-selection.md) required the provider-neutral capability boundary to be defined and validated across at least three materially different provider shapes before any first-provider selection. That is now done: the contract is recorded as [v1.0](docs/architecture/PROVIDER_CAPABILITY_CONTRACT.md), its structure is decided by [ADR-0006](docs/decisions/0006-capability-manifest-and-three-layer-availability.md), and [four provider shapes were validated](docs/architecture/PROVIDER_SHAPE_VALIDATION.md), forcing ten revisions. First-provider selection is now unblocked but remains a separate decision — Jules, Cursor, Replit, GitHub Copilot, VibeFlow and Happier are evidence inputs only, and none is selected or privileged by having been tested. `ALLOW_APP_STACK=0` continues to block application implementation.
