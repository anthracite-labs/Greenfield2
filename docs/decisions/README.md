# Architecture Decision Records

Durable trade-offs, recorded so future sessions can reconstruct *why* instead of
re-litigating it. Procedure:
[`.ecc/skills/decisions.md`](../../.ecc/skills/decisions.md).

## Rules

- One decision per file: `NNNN-<kebab-title>.md`, numbered sequentially.
- Accepted ADRs are never edited to change the decision — supersede them with a
  new ADR and mark the old one `superseded by ADR-NNNN`.
- Alternatives must be real, and the "why not" must be specific.
- Consequences must include the costs.

## Index

The records below begin with the **foundation** decisions inherited from
App-Factory, followed by Greenfield2 project Architecture decisions.

| ADR | Title | Status | Date |
| :-- | :-- | :-- | :-- |
| [0001](0001-ecc-on-arena-adapter.md) | Adopt an ECC-on-Arena adapter instead of native ECC | accepted | 2026-09-06 |
| [0002](0002-verification-gate.md) | `scripts/verify.sh` + GitHub Actions as the sole quality gate | accepted | 2026-09-06 |
| [0003](0003-flat-skill-files.md) | Flat per-task workflow files instead of upstream `SKILL.md` directories | accepted | 2026-09-06 |
| [0004](0004-lifecycle-config-stack-guard.md) | Project lifecycle config replaces the hard-coded no-app-stack guard | accepted | 2026-09-06 |
| [0005](0005-provider-contract-before-provider-selection.md) | Define the provider capability contract before selecting the first provider | accepted | 2026-09-13 |
| [0006](0006-capability-manifest-and-three-layer-availability.md) | Model the provider boundary as a declared capability manifest with three-layer availability | accepted | 2026-09-13 |

The ADR that records a project's **implementation stack** is referenced by
`STACK_DECISION_ADR` in [`../../config/project.env`](../../config/project.env),
and `scripts/verify.sh` fails if that reference does not resolve.

## Template

Copy [`0000-template.md`](0000-template.md) to start a new record.
