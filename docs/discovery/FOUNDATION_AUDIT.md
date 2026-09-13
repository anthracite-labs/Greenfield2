# Foundation Audit — Greenfield2 App 1

**Issue:** #1  
**Phase:** Discovery  
**Purpose:** classify the existing repository before product discovery changes it.

This audit treats the App-Factory / ECC-on-Arena system as a technical
foundation that Greenfield2 inherits for engineering discipline. Product work
may extend project-owned documentation, but must not casually rewrite the core
engineering machinery.

## Classification

| Path / surface | Classification | Action during App 1 discovery | Rationale |
| :-- | :-- | :-- | :-- |
| `.ecc/**` | **Foundation core** | **PRESERVE** | ECC rules, skills, roles, bootstrap, provenance and licence are the engineering operating system. |
| `FOUNDATION_VERSION` | **Foundation core** | **PRESERVE** | Identifies the App-Factory release; not the product version. |
| `scripts/verify.sh` | **Foundation core** | **PRESERVE** | Authoritative deterministic gate. Product checks may be added later alongside it, not by weakening it. |
| `scripts/selftest.sh` | **Foundation core** | **PRESERVE** | Negative tests prove the gate can fail. |
| `scripts/bootstrap.sh` | **Foundation core** | **PRESERVE** | Session briefing for the existing ECC-on-Arena protocol. |
| `scripts/init-project.sh` | **Foundation core** | **PRESERVE** | One-time lifecycle/identity helper; its job is generic and already complete. |
| `scripts/sync-ecc.sh` | **Foundation core** | **PRESERVE** | Upstream ECC inspection without overwriting local adaptations. |
| `.github/workflows/verify.yml` | **Foundation core** | **PRESERVE** | Independent execution of the gate. Future app CI is additive. |
| `config/main-ruleset.json` | **Foundation governance** | **PRESERVE** | Portable branch-governance intent. Live application is an administrative action, not a product change. |
| `docs/FACTORY.md` | **Foundation reference** | **PRESERVE** | Documents instantiation and GitHub administration. It should not become a product spec. |
| `docs/ARENA.md` | **Foundation reference** | **PRESERVE** | Harness/environment facts; project-independent unless Arena itself changes. |
| `AGENTS.md` | **Foundation entry point + project overlay** | **AMEND ADDITIVELY ONLY** | Keep all ECC/App-Factory hard rules and operating model; add project-specific read order/boundary only. |
| `README.md` | **Project-owned entry point** | **AMEND** | The repository is no longer the reusable template source; it must identify Greenfield2 while preserving foundation provenance. |
| `config/project.env` | **Project-owned lifecycle state** | **AMEND** | This is the intended machine-checked mechanism for `factory → discovery`. |
| `FOUNDATIONS.md` | **Project constitution** | **CREATE / EVOLVE** | Separates product discovery truth from the protected engineering foundation. |
| `docs/PRODUCT.md` | **Project discovery** | **REWRITE DURING DISCOVERY** | Holds accepted problem/users/scope/non-goals/success after product-owner review. |
| `docs/DOMAIN.md` | **Project discovery** | **REWRITE DURING DISCOVERY** | Holds accepted product vocabulary/entities/states after the product definition emerges. |
| `docs/ARCHITECTURE.md` | **Foundation architecture + later project architecture** | **PRESERVE FOUNDATION; EXTEND LATER** | Existing content correctly documents the engineering system. Application architecture belongs to the Architecture phase. |
| `docs/SECURITY.md` | **Foundation security floor + later application threat model** | **PRESERVE FOUNDATION; EXTEND LATER** | The file explicitly says future application security extends rather than replaces the foundation policy. |
| `docs/ROADMAP.md` | **Lifecycle authority** | **PRESERVE NOW** | The current file correctly maps repository lifecycle. Product feature sequencing should be added only after Discovery defines the product. |
| `docs/MEMORY.md` | **Project memory** | **APPEND ONLY** | Existing foundation/provenance entries remain historical truth; new Greenfield2 sessions append beneath them. |
| `docs/decisions/**` | **Decision framework** | **PRESERVE STRUCTURE; ADD NEW ADRs LATER** | Architecture decisions are not yet authorized in Discovery. |
| `docs/codemaps/**` | **Later implementation docs** | **PRESERVE / UNUSED** | Codemaps become meaningful only when application code exists. |

## Protected-foundation rule

Ordinary Greenfield2 product issues must not modify:

```text
.ecc/**
FOUNDATION_VERSION
scripts/verify.sh
scripts/selftest.sh
scripts/bootstrap.sh
scripts/init-project.sh
scripts/sync-ecc.sh
.github/workflows/verify.yml
config/main-ruleset.json
ECC provenance/licence material
```

If a real defect is later found in this engineering system, open a separate
foundation/ECC issue. Do not mix that work into a feature, discovery or
architecture PR.

## Current repository-state findings

Read-only GitHub inspection during Issue #1 found:

- default branch: `main`;
- `main` currently reports `protected=false`;
- the repository rulesets endpoint currently returns no live rulesets;
- the committed `config/main-ruleset.json` therefore describes intended
  governance but is **not evidence that GitHub is enforcing it**.

No administrative change was made during this audit. `docs/FACTORY.md` correctly
requires an authorized human to apply and then verify the live ruleset. This is
an administrative follow-up, not a reason to weaken or rewrite the committed
foundation.

### 2026-09-13 governance follow-up

The Issue #1 findings above remain historical evidence of the repository state
at that time. The administrative follow-up is now resolved: live GitHub ruleset
`main-protection` is active on `~DEFAULT_BRANCH`, GitHub reports `main` as
protected, and the live policy matches the committed solo-owner governance
intent in `config/main-ruleset.json` for the material controls: pull requests
required, `required_approving_review_count=0`,
`require_last_push_approval=false`, review-thread resolution required, strict
and up-to-date `Foundation gate` and `Independent checks`, no branch deletion,
no non-fast-forward updates, and no bypass actors.

This follow-up records platform state only. It does not change foundation policy
or turn GitHub administrative state into repository-owned configuration.

## App 1 evidence boundary

The following are project research inputs, not foundation machinery:

- VibeFlow target/baseplate research and architecture;
- Replit clean-room behavioral/product research;
- the updated ADOPT/HARVEST/REJECT OSS matrix;
- subsequent product-owner Discovery answers.

They belong in project discovery/reference material and may influence future
requirements and ADRs. They do not belong inside `.ecc/` and must not alter ECC
skills/rules merely to encode App 1 product preferences.

## Audit decision

**Foundation status: KEEP.**

The ECC/App-Factory technical fork is fit to remain Greenfield2's engineering
base. The immediate work is to instantiate the project and populate the
project-owned Discovery layer around it, not to refactor the foundation.
