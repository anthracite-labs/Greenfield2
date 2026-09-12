# Greenfield2 Foundations

**Status:** Discovery constitution for App 1.  
**Lifecycle:** `PROJECT_PHASE=discovery`.  
**Implementation authority:** none yet; `ALLOW_APP_STACK=0` remains in force.

This file defines the project-level frame for discovery. It does **not** replace
or weaken the App-Factory / ECC-on-Arena engineering foundation. The engineering
foundation defines how work is performed; this file records what Greenfield2 is
trying to discover about the product.

## 1. Fresh-product rule

Greenfield2 is **App 1**, treated as a fresh product.

Prior work is evidence, not inheritance:

- **VibeFlow** is the target/baseplate: its product thesis, authority model,
  provider-neutral design, durability, recovery, verification and broader
  software-development lifecycle are the strongest starting point to challenge,
  refine and improve.
- **Replit research** is clean-room behavioral/reference evidence, especially
  for mobile and Workspace interaction patterns, Agent/task UX and capability
  coverage. Replit does not define Greenfield2 and is not a clone target.
- **OSS research** is evaluated under the existing engineering philosophy:
  ADOPT maintained standards/dependencies where they fit, HARVEST useful
  patterns/components behind Greenfield2-owned boundaries, and REJECT code or
  dependencies that fail product, security, maintenance or licensing review.

Nothing becomes a Greenfield2 requirement merely because VibeFlow specified it,
Replit demonstrates it, or an OSS project implements it.

## 2. Starting product hypothesis — not yet an accepted definition

Discovery begins from this hypothesis:

> Greenfield2 may become a mobile-first, provider-neutral software-development
> control plane that presents one coherent product while allowing users to
> bring or replace important execution rails such as coding Agents, models/API
> keys and workspaces.

The VibeFlow shorthand **BYOA / BYOK / BYOW** is therefore a high-value starting
hypothesis, not an architecture decision and not yet a complete V1 promise.
Discovery must decide the exact user value, scope, terminology and product
boundary.

## 3. Foundation/core engineering system is protected

Greenfield2 is built on App-Factory, a repository-owned ECC-on-Arena adaptation.
The operating model remains:

> **ECC is repository-owned, Arena-executed, ChatGPT-supervised.**

GitHub is the durable source of truth. The committed verification gate and
negative tests remain the evidence standard.

Product discovery must not casually rewrite the engineering foundation. The
following are treated as protected foundation surfaces for ordinary project
work:

- `.ecc/**` — ECC rules, skills, roles, provenance and licence material;
- `FOUNDATION_VERSION`;
- `scripts/verify.sh`, `scripts/selftest.sh`, `scripts/bootstrap.sh`,
  `scripts/init-project.sh`, `scripts/sync-ecc.sh`;
- `.github/workflows/verify.yml`;
- `config/main-ruleset.json`;
- the foundation lifecycle/no-stack mechanism itself.

A future change to those surfaces requires a separate foundation/ECC issue and
must be justified as an engineering-system change, not smuggled into product
work.

## 4. Project-owned discovery surface

The following are expected to evolve as Greenfield2 is discovered:

- `README.md` — project identity and developer entry point;
- this `FOUNDATIONS.md` — project constitution and evidence hierarchy;
- `config/project.env` — legitimate lifecycle state;
- `docs/PRODUCT.md` — problem, users, jobs, scope, non-goals and success;
- `docs/DOMAIN.md` — accepted product vocabulary, entities and state meanings;
- project-specific additions to `docs/SECURITY.md` and
  `docs/ARCHITECTURE.md` when the lifecycle permits them;
- new ADRs under `docs/decisions/` once durable technical choices are actually
  being made.

Foundation text may be extended for the project, but foundation controls and
provenance are preserved rather than replaced.

## 5. Evidence and decision hierarchy

Use the following order when sources disagree:

1. **Reviewed Greenfield2 decisions** — accepted project docs and ADRs.
2. **Explicit product-owner decisions** recorded through Greenfield2 issues and
   reviewed changes.
3. **Verified VibeFlow evidence** — target/baseplate material to reuse only when
   it survives fresh scrutiny.
4. **Verified Replit behavioral evidence** — reference/benchmark input, subject
   to the clean-room boundary.
5. **Verified OSS/upstream evidence** — standards, maintained dependencies and
   harvest candidates at pinned revisions when promoted.
6. **Inference** — useful for forming questions, never silently promoted to
   fact or requirement.

A later source does not automatically outrank an accepted Greenfield2 decision;
changing durable truth requires an explicit reviewed amendment.

## 6. Discovery questions that remain open

Discovery must establish, at minimum:

- the primary user and the problem they are hiring Greenfield2 to solve;
- the role of mobile versus web/desktop surfaces;
- what provider neutrality means to the user and what Greenfield2 itself owns;
- the exact meaning and V1 scope of BYOA, BYOK and BYOW;
- the relationship among Project, Agent, model, workspace, repository and
  deployment/data providers;
- the degree of Agent autonomy, approvals and background/durable work;
- the required recovery, resumability, evidence and independent verification
  promise;
- the artifact/output types the product must support;
- the first coherent, shippable product slice and explicit non-goals;
- measurable success criteria;
- privacy, security, data and commercial constraints.

Until those are answered and reviewed, they remain questions rather than
requirements.

## 7. Clean-room boundary

Behavioral observations may inform independently written requirements,
state-machines, UX expectations, acceptance tests and architecture decisions.
Greenfield2 must not copy proprietary implementation code, private protocols,
prompts, credentials, protected assets, branding or trademarks from Replit or
any other closed product.

Open-source code may be used only under its actual licence and the repository's
normal dependency/harvest review process. Visibility on GitHub is not permission
by itself.

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

The current repository is at the **Discovery hypothesis** stage. No protocol,
framework, database, auth system, workflow engine, workspace provider, Agent,
model provider, deployment provider or harvested codebase is approved by this
file.

## 9. Discovery exit condition

Greenfield2 may move to `PROJECT_PHASE=architecture` only when the reviewed
product definition is sufficiently complete that Architecture can answer
**how** without reopening basic questions about **what**, **for whom**, and
**why**.

The App-Factory lifecycle and no-stack guard remain authoritative throughout
that transition.
