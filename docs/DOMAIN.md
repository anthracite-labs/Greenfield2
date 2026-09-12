# Domain

**Status: Discovery in progress. Product domain not yet accepted.**

Greenfield2 has entered product Discovery, but no product-domain model is yet
authoritative. VibeFlow terminology is useful starting material; Replit and OSS
research add behavioral and implementation evidence. None of those vocabularies
becomes Greenfield2 truth until it survives fresh product discovery.

This file will hold the accepted ubiquitous language, entities, invariants,
state transitions and business rules once they are agreed.

## Discovery rule for vocabulary

A term may appear in research without being a Greenfield2 domain entity.
For example, prior work discusses concepts such as `Project`, `Conversation`,
`Agent`, `Model`, `Workspace`, `Repository`, `Task`, `Execution`, `Approval`,
`Policy`, `Evidence`, `Verification`, `Artifact`, `Release` and provider
bindings. During Discovery these are **candidate terms only**.

Before a term becomes canonical, define:

1. what it means to the user/product;
2. what it explicitly does **not** mean;
3. who owns its identity and authoritative state;
4. which lifecycle/state transitions matter;
5. whether the term belongs to Greenfield2 or to an external provider adapter.

This is especially important for the provider-neutral thesis: provider
vocabulary must not accidentally become Greenfield2 authority merely because an
upstream Agent, workspace, Git host or deployment system uses the same word.

## Candidate questions to resolve

| Area | Discovery question |
| :-- | :-- |
| Project | Is Project the stable product identity across repositories, workspaces, Agents and deployments? |
| Agent vs model | Are they separate user-visible concepts, separate internal bindings, or both? |
| Workspace | Is Workspace a replaceable execution/development environment rather than the Project itself? |
| Repository | How does source-control identity relate to a Project and to ephemeral task worktrees? |
| Task vs Execution | What does the user-facing work item mean versus the durable run/attempt that performs it? |
| Verification | Is trusted completion a separate lifecycle from task/execution completion, and how visible is it? |
| Approval / Policy | What decisions require human or organizational authority, and how are recurring decisions represented? |
| Artifact / Asset | What outputs can a Project own beyond source files and one preview? |
| Release / Deployment | What is Greenfield2 authority versus external deployment-provider state? |
| Provider bindings | Which provider categories are first-class product concepts under BYOA/BYOK/BYOW? |

The table is a question set, not an accepted entity list.

## Skeleton to fill as Discovery accepts terms

### Entities

| Entity | Definition | Identity | Invariants |
| :-- | :-- | :-- | :-- |
|  |  |  |  |

### Ubiquitous language

| Term | Means | Does NOT mean |
| :-- | :-- | :-- |
|  |  |  |

### State transitions

| From | Event | To | Guard |
| :-- | :-- | :-- | :-- |
|  |  |  |  |

### Business rules

| Rule | Rationale | Enforced where |
| :-- | :-- | :-- |
|  |  |  |

## Engineering terms that are already authoritative

These describe the repository foundation, not the product domain.

| Term | Meaning here |
| :-- | :-- |
| **Foundation** | The generic engineering system shipped by App-Factory. |
| **Adapter** | `.ecc/` — the ECC-on-Arena adaptation. Not native ECC. |
| **Skill / workflow** | An on-demand Markdown procedure under `.ecc/skills/`. |
| **Rule** | A standing, always-in-force constraint under `.ecc/rules/`. |
| **Role** | A sequential review persona under `.ecc/roles/` (not a subagent). |
| **Gate** | `scripts/verify.sh` — deterministic, non-zero on failure. |
| **Lifecycle phase** | `PROJECT_PHASE` in `config/project.env`. |
| **No-stack guard** | The `no_app_stack` check, driven by `ALLOW_APP_STACK`. |
| **Project memory** | `docs/MEMORY.md` — append-only, Git-tracked. |
| **ADR** | A record under `docs/decisions/` for a durable trade-off. |

## Related

- [`../FOUNDATIONS.md`](../FOUNDATIONS.md) — evidence hierarchy and Discovery constitution
- [PRODUCT.md](PRODUCT.md) — product definition in progress
- [ARCHITECTURE.md](ARCHITECTURE.md) — engineering foundation; application architecture later
- [decisions/](decisions/README.md) — decision record index
