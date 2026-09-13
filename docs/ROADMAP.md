# Roadmap

Lifecycle sequencing only. Product commitments, scope and non-goals live in
[PRODUCT.md](PRODUCT.md); this file records how the repository progresses from
product definition to reviewed Architecture and then implementation.

The stages below map one-to-one onto `PROJECT_PHASE` in
[`../config/project.env`](../config/project.env), so the roadmap and the
machine-checked repository state cannot drift apart.

```text
factory  →  discovery  →  architecture  →  implementation
   │            │              │                 │
 template   product not     product defined,  stack recorded in an ADR;
 itself     yet defined     stack being       application code allowed
                            decided (ADR)     (ALLOW_APP_STACK=1)
                              ▲
                        Greenfield2 now
```

## Stage: factory (`PROJECT_PHASE=factory`)

Only the App-Factory template repository itself sits here. Work in scope:
maintaining the generic foundation — rules, workflows, verification, negative
tests, CI, provenance. No product work of any kind.

Exit condition: a new repository is generated from the template and
`scripts/init-project.sh` moves it to `discovery`.

## Stage: discovery (`PROJECT_PHASE=discovery`)

The default state of a newly generated application repository.

- [x] Run `scripts/init-project.sh`, then `scripts/verify.sh` and
      `scripts/selftest.sh` — both must pass before any other work.
- [ ] Complete the repository-admin checklist in [FACTORY.md](FACTORY.md);
      template copies files, not GitHub configuration. This remains a governance
      follow-up and is not silently treated as complete.
- [x] Open a product-discovery issue and answer the questions in
      [PRODUCT.md](PRODUCT.md).
- [x] Record the domain vocabulary in [DOMAIN.md](DOMAIN.md) as it emerges.

The no-stack guard is active. Application-stack artifacts are rejected.

Exit condition: [PRODUCT.md](PRODUCT.md) contains a reviewed product definition.
This condition was satisfied by PR #5; Issue #6 performs the separate lifecycle
transition into Architecture.

## Stage: architecture (`PROJECT_PHASE=architecture`)

Architecture is the active Greenfield2 lifecycle stage.

- [x] Open Architecture issues for the decisions that actually need to be made.
- [ ] Define Greenfield2's provider-neutral capability contract before selecting
      the first provider; this is required by
      [ADR-0005](decisions/0005-provider-contract-before-provider-selection.md).
- [ ] Validate that contract against at least three materially different provider
      shapes and revise it where the comparison exposes vendor leakage, false
      universality or lowest-common-denominator loss.
- [ ] Only after provider-contract validation, evaluate and select the first
      provider against the accepted product-fit and integration-legitimacy gates;
      do not inherit a provider choice from Discovery references or Architecture
      research candidates.
- [ ] Evaluate real implementation alternatives; record durable choices as ADRs
      in [decisions/](decisions/README.md) with costs and rejected alternatives
      stated.
- [ ] Define application boundaries, provider-adapter responsibilities and
      trust/data boundaries without changing the provider-authoritative product
      rule.
- [ ] Decide the testing strategy and what the future stack-specific CI gate
      must run.
- [ ] Select and accept the application stack through an ADR only after the
      relevant alternatives have been researched and reviewed.

The provider contract is a capability/interface boundary, not a universal
Greenfield2 resource model. Provider-native resources, names, lifecycle and state
remain provider-authoritative; capability availability, connected-account
entitlement and Greenfield2 UI support remain separate concerns.

The no-stack guard remains active throughout Architecture. `ALLOW_APP_STACK=0`
and an empty `STACK_DECISION_ADR` mean Architecture may research and decide but
may not introduce application-stack artifacts simply to "try something out".

Exit condition: an accepted application-stack ADR that satisfies the lifecycle
guard's required markers.

## Stage: implementation (`PROJECT_PHASE=implementation`)

- [ ] Mark the stack ADR with `**Decision Type:** application-stack` and
      `**Status:** accepted`.
- [ ] Set `PROJECT_PHASE=implementation`, `ALLOW_APP_STACK=1` and
      `STACK_DECISION_ADR=docs/decisions/NNNN-<title>.md` in
      `config/project.env`, in one reviewed PR that changes nothing else.
      `scripts/verify.sh` rejects the transition unless all three agree, the
      ADR exists, is not the template, carries the stack marker, and is
      accepted. Both `lifecycle` and `no_app_stack` validate this
      independently, so neither can be bypassed by running one check alone.
- [ ] Add stack-specific lint/test/build jobs to CI. The foundation gate keeps
      running alongside them; it is never replaced.
- [ ] Add codemaps under [codemaps/](codemaps/README.md) as code areas appear.

## Explicitly not pre-decided by the foundation or Discovery

- Any framework, database, auth implementation, hosting target, protocol, UI
  technology or first provider. Those are per-project decisions made through
  Architecture issues and ADRs.
- Native ECC plugin compatibility — Arena has no plugin runtime.
- Browser E2E in the foundation gate — no browser binaries in the sandbox.
- `.git/hooks/` enforcement — hooks do not survive a fresh clone.
