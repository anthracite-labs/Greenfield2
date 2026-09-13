# Greenfield2 Foundations

**Status:** Discovery constitution for App 1 with a product-owner definition under review in Issue #4.  
**Lifecycle:** `PROJECT_PHASE=discovery`.  
**Implementation authority:** none yet; `ALLOW_APP_STACK=0` remains in force.

This file defines the project-level frame for Greenfield2 Discovery. It does **not** replace or weaken the App-Factory / ECC-on-Arena engineering foundation. The engineering foundation defines how work is performed; this file records the evidence hierarchy, product boundary and promotion rules that Discovery must respect.

## 1. Fresh-product rule

Greenfield2 is **App 1**, treated as a fresh product.

Prior work is evidence, not inheritance:

- **VibeFlow** is the target/baseplate and strongest prior interoperability/product reference. Its provider-neutral thesis, capability discovery, BYOA/BYOK/BYOW direction and provider/client distinctions are high-value input. Its older control-plane, durability, recovery, policy, Task/Execution/Verification and independent-verification authority are not inherited unless Greenfield2 explicitly adopts them through review.
- **Replit research** is clean-room behavioral/reference evidence, especially for coherent mobile software-building interactions, Agent/task UX, workspace behavior and capability coverage. Replit does not define Greenfield2 and is not a clone target.
- **Happier** is an OSS architecture/feasibility reference for cross-device Agent supervision, provider capability catalogs, reconnect behavior and rich development surfaces. Greenfield2 does not inherit Happier's account, relay, daemon, session, persistence or orchestration authority.
- **Other OSS/upstream research** is evaluated under the existing engineering philosophy: ADOPT maintained standards/dependencies where they fit, HARVEST useful patterns/components behind Greenfield2-owned boundaries, and REJECT code or dependencies that fail product, security, maintenance or licensing review.

Nothing becomes a Greenfield2 requirement merely because VibeFlow specified it, Replit demonstrates it, Happier implements it, or another OSS project contains it.

## 2. Discovery product outcome

The product-owner Discovery outcome is:

> **Greenfield2 is a mobile-first universal software-development interface for technical builders. It connects to a user's supported software-development provider account, discovers what that provider officially exposes, and presents those capabilities and resources through one coherent Greenfield2 interface.**

The defining authority rule is:

> **Greenfield2 owns no provider development state. The connected provider remains authoritative for provider-owned resources, capabilities, runtime, sessions, files, artifacts, actions, lifecycle, entitlements, errors, approvals and results that Greenfield2 exposes.**

For MVP, Greenfield2 proves this model with one user-supplied full-stack provider. Post-MVP BYOK/BYOA/BYOW composition remains directional only; it is not an MVP architecture decision.

The detailed product definition, V1 scope, non-goals, success measures and constraints live in [`docs/PRODUCT.md`](docs/PRODUCT.md). The minimal accepted product vocabulary and ownership semantics live in [`docs/DOMAIN.md`](docs/DOMAIN.md).

The repository remains in `PROJECT_PHASE=discovery` until the product definition is independently reviewed and a separate reviewed lifecycle transition is made.

## 3. Foundation/core engineering system is protected

Greenfield2 is built on App-Factory, a repository-owned ECC-on-Arena adaptation. The operating model remains:

> **ECC is repository-owned, Arena-executed, ChatGPT-supervised.**

GitHub is the durable source of truth. The committed verification gate and negative tests remain the evidence standard.

Product work must not casually rewrite the engineering foundation. The following are protected foundation surfaces for ordinary project work:

- `.ecc/**` — ECC rules, skills, roles, provenance and licence material;
- `FOUNDATION_VERSION`;
- `scripts/verify.sh`, `scripts/selftest.sh`, `scripts/bootstrap.sh`, `scripts/init-project.sh`, `scripts/sync-ecc.sh`;
- `.github/workflows/verify.yml`;
- `config/main-ruleset.json`;
- the foundation lifecycle/no-stack mechanism itself.

A future change to those surfaces requires a separate foundation/ECC issue and must be justified as an engineering-system change, not smuggled into product work.

## 4. Project-owned product surfaces

The following are expected to evolve through the Greenfield2 lifecycle:

- `README.md` — project identity and developer entry point;
- this `FOUNDATIONS.md` — project constitution and evidence hierarchy;
- `config/project.env` — legitimate lifecycle state;
- `docs/PRODUCT.md` — product problem, users, jobs, scope, non-goals, trust/data constraints and success;
- `docs/DOMAIN.md` — accepted product vocabulary, ownership boundaries and state meanings;
- project-specific additions to `docs/SECURITY.md` and `docs/ARCHITECTURE.md` when the lifecycle permits them;
- ADRs under `docs/decisions/` once durable technical choices are actually being made.

Foundation text may be extended for the project, but foundation controls and provenance are preserved rather than replaced.

## 5. Evidence and decision hierarchy

Use the following order when sources disagree:

1. **Reviewed Greenfield2 decisions** — accepted project docs and ADRs.
2. **Explicit product-owner decisions** recorded through Greenfield2 issues and reviewed changes.
3. **Verified VibeFlow evidence** — target/baseplate material to reuse only when it survives fresh scrutiny.
4. **Verified Replit behavioral evidence** — clean-room reference/benchmark input.
5. **Verified OSS/upstream evidence**, including Happier — standards, maintained dependencies and harvest/reference candidates at pinned revisions when promoted.
6. **Inference** — useful for forming questions, never silently promoted to fact or requirement.

A later source does not automatically outrank an accepted Greenfield2 decision. Changing durable truth requires an explicit reviewed amendment.

## 6. Discovery questions resolved by the product definition

Discovery has explicitly established, at the product-owner level:

- the primary user: technical builders already using an eligible agentic software-development provider;
- the job to be done: build and continue real software wherever the user is through one coherent interface over the provider they already use;
- the category: universal software-development interface;
- mobile's role: mobile-first, not mobile-only;
- provider neutrality: provider capabilities/state remain provider-authoritative while Greenfield2 provides one adaptive interface;
- MVP ownership: Greenfield2 owns only its account/interface/connection/reference metadata, not provider development state;
- MVP scope: one user-supplied full-stack provider and one end-to-end workspace-level development loop;
- autonomy/approval: provider behavior and provider approval semantics govern; Greenfield2 forwards user controls;
- durability/recovery/verification: provider truth only in MVP; no independent Greenfield2 recovery or verification authority;
- V1 output needs: meaningful provider files/diffs plus a provider result such as preview/build/deployment/PR or equivalent, with Agent/activity/approval context where exposed;
- explicit MVP non-goals;
- product-value and reliability success measures;
- minimum-data, supported-integration, entitlement, commercial, geography/compliance and platform constraints.

These answers are recorded in `docs/PRODUCT.md` and `docs/DOMAIN.md`. Architecture must not reopen them silently; a real product change requires a reviewed amendment.

## 7. Clean-room and source-reuse boundary

Behavioral observations may inform independently written requirements, acceptance tests and later architecture decisions. Greenfield2 must not copy proprietary implementation code, private protocols, prompts, credentials, protected assets, branding or trademarks from Replit or any other closed product.

Open-source code may be used only under its actual licence and the repository's normal dependency/harvest review process. Visibility on GitHub is not permission by itself.

Happier and other OSS projects may donate patterns or, after proper review, reusable code. Their product authority and persistence/session models do not become Greenfield2 architecture by default.

## 8. Promotion path

Research moves toward implementation only through explicit promotion:

```text
observed evidence / prior idea
        ↓
Discovery hypothesis
        ↓
reviewed product requirement or domain rule
        ↓
Architecture alternative analysis
        ↓
accepted ADR where a durable technical choice is required
        ↓
implementation mission / issue
        ↓
verified change + independent review
```

Issue #4 is the review anchor for promoting the current product-owner Discovery synthesis into repository truth.

No protocol, framework, database, auth system, workflow engine, workspace provider, Agent, model provider, deployment provider or harvested codebase is approved by this Discovery definition.

## 9. Discovery exit condition

Greenfield2 may move to `PROJECT_PHASE=architecture` only when the product definition in `docs/PRODUCT.md` and ownership vocabulary in `docs/DOMAIN.md` are independently reviewed and accepted as sufficiently complete for Architecture to answer **how** without reopening basic questions about **what**, **for whom**, and **why**.

The lifecycle transition itself is separate reviewed work. `ALLOW_APP_STACK=0` remains in force throughout Architecture until the normal accepted application-stack ADR and implementation transition satisfy the machine-checked lifecycle rules.
