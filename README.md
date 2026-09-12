# Greenfield2

**Status:** App 1 discovery.  
**Engineering foundation:** App-Factory `0.1.0`, using the repository-owned
ECC-on-Arena adapter derived from ECC v2.2.0 (MIT).

Greenfield2 is now an instantiated application repository, not the reusable
App-Factory template source. The product itself is deliberately being defined
through Discovery before any application stack is allowed.

Read in this order when orienting to the repository:

1. [`AGENTS.md`](AGENTS.md) — engineering operating rules and ECC-on-Arena entry point.
2. [`README.md`](README.md) — repository identity and current lifecycle.
3. [`FOUNDATIONS.md`](FOUNDATIONS.md) — App 1 discovery constitution and evidence hierarchy.
4. [`config/project.env`](config/project.env) — machine-checked lifecycle state.
5. [`docs/MEMORY.md`](docs/MEMORY.md) — append-only verified project memory.
6. [`docs/PRODUCT.md`](docs/PRODUCT.md) / [`docs/DOMAIN.md`](docs/DOMAIN.md) — discovery outputs as they become accepted.

## Discovery starting point

App 1 is being treated with a fresh pair of eyes.

- **VibeFlow** is the target/baseplate: the strongest prior expression of the
  intended provider-neutral, mobile-first software-development control-plane
  direction. Its ideas must survive fresh Discovery before becoming App 1 truth.
- **Replit research** is benchmark/reference evidence, especially for proven
  mobile/Workspace interactions, Agent/task behavior and capability coverage.
  It is not a clone specification.
- **Open-source research** follows the existing ADOPT / WRAP / BRIDGE / EXTEND /
  BUILD discipline. Candidates such as AHP, ACP, MCP, T3 Code, Lody, CC Pocket,
  OpenHands/OpenCode and worktree/verification projects are evidence and
  harvest candidates until formally promoted.

See [`FOUNDATIONS.md`](FOUNDATIONS.md) for the rules that govern this evidence.

## Lifecycle

The authoritative state is [`config/project.env`](config/project.env):

```text
factory  →  discovery  →  architecture  →  implementation
                ▲
          Greenfield2 now
```

During Discovery, `ALLOW_APP_STACK=0`. No application framework, database,
auth scheme, hosting target, provider implementation or UI stack may be
introduced merely because prior research suggests one. Those choices belong to
the later Architecture/ADR process.

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

The App-Factory foundation defines **how** work is done. Product discovery does
not rewrite the ECC skills, rules, roles, provenance or verification philosophy.
Foundation changes require their own issue and review.

## What the foundation provides

| | |
| :-- | :-- |
| **ECC-on-Arena adapter** | Engineering rules, 10 on-demand workflows, 3 review personas, adapted from ECC v2.2.0 (MIT), fully attributed. Not native ECC. |
| **Deterministic gate** | `scripts/verify.sh` — committed checks, non-zero on failure, re-run independently in CI. |
| **Negative tests** | `scripts/selftest.sh` — injects faults into a throwaway copy and asserts the gate rejects them. |
| **Lifecycle state** | `config/project.env` — factory → discovery → architecture → implementation, with a no-stack guard that stands down only via a reviewed, ADR-backed transition. |
| **Portable governance** | `config/main-ruleset.json` — a branch-protection payload that must be applied to the actual GitHub repository by an authorized human. |
| **Project documentation system** | Product, domain, roadmap, architecture, security, memory, factory and ADR documents. |

## Starting an Arena session

Arena auto-loads nothing. The engineering bootstrap remains unchanged:

> Read `.ecc/BOOTSTRAP.md`, initialize the project engineering protocol,
> inspect project memory and the skill index, then work GitHub Issue #X. Load
> only skills relevant to that issue.

Or print the same briefing:

```bash
bash scripts/bootstrap.sh
```

Startup context is intentionally small. Workflows are loaded one or two at a
time only when the task calls for them.

## Repository map

```text
AGENTS.md                  engineering entry point
FOUNDATIONS.md             App 1 discovery constitution
FOUNDATION_VERSION         App-Factory foundation version
.ecc/                      protected ECC-on-Arena adapter
config/project.env         project identity, lifecycle and no-stack guard
config/main-ruleset.json   portable branch-governance intent
docs/PRODUCT.md            product discovery and accepted product definition
docs/DOMAIN.md             product vocabulary and state meanings
docs/ARCHITECTURE.md       engineering foundation; application architecture later
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

The foundation gate remains authoritative. A future application stack adds its
own lint/test/build gates alongside it; it does not replace the foundation gate.

## Foundation provenance and licensing

- `.ecc/` is derived from
  [Everything Claude Code (ECC)](https://github.com/affaan-m/ECC) v2.2.0, MIT
  licensed. The upstream notice is committed at
  [`.ecc/LICENSE-ECC`](.ecc/LICENSE-ECC); details live in
  [`.ecc/UPSTREAM.md`](.ecc/UPSTREAM.md).
- This is an **adaptation, not native ECC**. Arena has no plugin runtime, slash
  commands, lifecycle hooks or subagent API, and the repository does not claim
  otherwise.
- Greenfield2 was instantiated from App-Factory `0.1.0`, whose factory
  provenance is recorded in [`docs/FACTORY.md`](docs/FACTORY.md) and
  [`docs/MEMORY.md`](docs/MEMORY.md). Product history from the source foundation
  is not inherited as App 1 product truth.

## Current work

Discovery is tracked through GitHub issues and reviewed pull requests. Issue #1
establishes the fresh-product boundary and protects the ECC/App-Factory core
while Greenfield2 is instantiated as App 1.
