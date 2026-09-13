# Architecture working documents

Detailed Architecture artefacts that are too large to live inline in
[../ARCHITECTURE.md](../ARCHITECTURE.md). That file remains the entry point and
the index of open decisions; the documents here are the working substance behind
specific decisions.

Nothing in this directory is implementation. `PROJECT_PHASE=architecture` and
`ALLOW_APP_STACK=0` remain in force: these documents define boundaries and
obligations, and select no transport, framework, database, auth implementation,
hosting target, UI technology or provider.

| Document | What it is | Decision vehicle |
| :-- | :-- | :-- |
| [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) | Greenfield2's provider-neutral capability contract: what Greenfield2 may ask, observe and invoke through a provider adapter, and how availability is resolved. | Issue #9, [ADR-0005](../decisions/0005-provider-contract-before-provider-selection.md), [ADR-0006](../decisions/0006-capability-manifest-and-three-layer-availability.md) |
| [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) | Point-in-time validation of that contract against four materially different provider shapes, and the contract revisions that validation forced. | Issue #9 |
| [FIRST_PROVIDER_EVALUATION.md](FIRST_PROVIDER_EVALUATION.md) | Current first-party evaluation of first MVP provider candidates against Product Fit and Integration Legitimacy gates, with shortlist and recommendation. Selects no provider. | Issue #8 |

## How to read these

Read the contract first, then the validation record. The validation record is
the evidence that the contract is provider-neutral rather than
first-vendor-shaped; it is also the record of what the contract got wrong before
it was revised. A contract whose validation section is empty has not been
validated.

## Related

- [../ARCHITECTURE.md](../ARCHITECTURE.md) — Architecture mandate, invariants and open decisions
- [../PRODUCT.md](../PRODUCT.md) — reviewed product definition this contract must realize
- [../DOMAIN.md](../DOMAIN.md) — Greenfield2-owned vocabulary and ownership semantics
- [../decisions/README.md](../decisions/README.md) — decision record index
