# Project Memory

Append-only ledger. Newest entry at the bottom. The sandbox is destroyed
between sessions, so memory that is not committed does not exist.

Procedure: [`../.ecc/skills/project-memory.md`](../.ecc/skills/project-memory.md).

## How to use this file

- Append one entry per working session. Never rewrite or delete an old entry;
  correct it with a new one that says what changed and why.
- Record what was **verified**, with the command and its actual result — not
  what was intended.
- Record surprises and dead ends. A failed approach that is not written down
  gets retried by the next session.
- Durable trade-offs go in [decisions/](decisions/README.md) as ADRs; this file
  points at them rather than duplicating them.

Entry template:

```text
## YYYY-MM-DD — <short title>

**Context:** <issue / branch>
**Did:** <what changed>
**Verified:** <command → actual result>
**Learned:** <surprises, dead ends, constraints discovered>
**Next:** <what the following session should know or do>
```

---

## Template provenance (carried by App-Factory, not project history)

This repository's engineering foundation is App-Factory (see
[`../FOUNDATION_VERSION`](../FOUNDATION_VERSION)). App-Factory v0.1.0 was
derived from the reviewed Ditto Foundation source commit
`anthracite-labs/Ditto@5d9cc349d264f73e8da913da9d2cea664522237d`. That is
factory provenance only: none of the source project's product history,
debugging chronology, issue numbers, or branch names is carried here, and none
of it applies to this repository.

## Operating conventions inherited from the foundation

These are the conventions every session is expected to follow. They are
recorded here because they are the durable context a new session needs before
it has read anything else.

| Convention | Where it is enforced |
| :-- | :-- |
| Read `.ecc/BOOTSTRAP.md` first; load 1–2 skills on demand. | `bootstrap`, `skill_index` checks |
| `scripts/verify.sh` is the only accepted evidence of quality. | CI job `Foundation gate` |
| The gate is proven by negative tests, not by passing. | `scripts/selftest.sh` |
| No stack/product choice without an approved issue and an ADR. | `no_app_stack`, `lifecycle` checks |
| Lifecycle changes are config diffs, never edits to the gate. | `config/project.env` + `lifecycle` check |
| Never commit credentials; findings are reported redacted. | `secrets`, `env_files` checks |
| Work on the session branch; never push to `main`; never self-merge. | `.ecc/rules/git.md`, branch ruleset |
| ECC is adapted, not vendored, and never silently upgraded. | `provenance`, `attribution` checks |

---

## Session entries

<!-- Append below this line. Do not edit entries above it. -->

## 2026-09-06 — App-Factory v0.1.0 foundation created

**Context:** Issue #1, branch `arena/01a076c9-app-factory`
**Did:** Created the generic reusable foundation from the reviewed Ditto
source commit: genericized `.ecc/` adapter, added `FOUNDATION_VERSION`,
`config/project.env` lifecycle state, portable `config/main-ruleset.json`,
`scripts/init-project.sh`, lifecycle-aware no-stack guard, clean
product/domain/roadmap/memory docs, and `docs/FACTORY.md`.
**Verified:** `bash scripts/verify.sh` and `bash scripts/selftest.sh` — see the
PR body for the recorded output of both runs.
**Learned:** The source foundation's permanent `ALLOW_APP_STACK=0` constant
inside `verify.sh` could not survive in a reusable template: a generated
repository must be able to graduate to an application stack without editing the
gate. Moving the state into `config/project.env` and adding a `lifecycle` check
that requires phase + ADR consistency keeps the transition explicit and
reviewable. ECC stays pinned at v2.2.0; upgrading it is a separate version bump.
**Next:** This template is `PROJECT_PHASE=factory`. A generated repository
should run `scripts/init-project.sh` first, then complete the GitHub-admin
checklist in [FACTORY.md](FACTORY.md), which the template cannot do for it.

## 2026-09-10 — Greenfield preflight documentation audit

**Context:** Issue #3, branch `chore/greenfield-preflight-cleanup`.
**Did:** Aligned the README lifecycle summary with the committed lifecycle by
including the `factory` phase, and clarified that GitHub's **Template repository**
setting is administrative state rather than something committed repository files
can prove or enable.
**Verified:** Read-only GitHub repository metadata reported `is_template=false`;
the repository rulesets endpoint returned no live rulesets at the time of this
audit. The repository content itself still carries `config/main-ruleset.json`
as the portable policy definition. No GitHub administrative setting was changed
by this documentation task. CI on the exact PR head is the acceptance evidence
for the repository edits.
**Learned:** Calling App-Factory a template source and GitHub marking it as a
template repository are separate states. The greenfield workflow must verify
both repository contents and live GitHub configuration instead of inferring one
from the other.
**Next:** A maintainer should enable GitHub's **Template repository** setting
before relying on **Use this template**, and separately decide whether to apply
the portable Main ruleset to App-Factory itself. Generated repositories must
still receive their own live governance because GitHub administrative settings
are not inherited.

## 2026-09-12 — Greenfield2 entered App 1 Discovery

**Context:** Issue #1, branch `discovery/issue-1-instantiate-app1`, draft PR #2.
**Did:** Initialized `PROJECT_NAME=Greenfield2`, `PROJECT_SLUG=greenfield2` and
`PROJECT_PHASE=discovery` while preserving `ALLOW_APP_STACK=0`; created the
project-level `FOUNDATIONS.md`; reframed README/product/domain documents for
fresh Discovery; added only an additive project overlay to `AGENTS.md`; and
recorded `docs/discovery/FOUNDATION_AUDIT.md`. No `.ecc/**`, verification script,
CI workflow, foundation version, provenance or branch-ruleset payload was
modified.
**Verified:** GitHub Actions run `34707025554` on PR #2 completed successfully.
`Foundation gate` ran `bash scripts/verify.sh` and reported `PASS — 17 passed,
0 failed, 1 skipped`; the skip was AgentShield scanning zero Claude-config files,
as expected/advisory. The same job ran `bash scripts/selftest.sh` and reported
`SELFTEST: PASS — 128 cases behaved as asserted`. The separate `Independent
checks` job also completed successfully. Read-only GitHub inspection reported
`main` as `protected=false` and the repository rulesets endpoint returned `[]`.
A separate attempt to clone the branch in the local tool container failed before
verification because that environment could not resolve `github.com`; CI is the
actual execution evidence for this session.
**Learned:** The existing ECC/App-Factory technical foundation can be preserved
without bending it around App 1: project identity, lifecycle and Discovery truth
fit cleanly in the intended project-owned surfaces. The committed ruleset is
policy intent only; live GitHub governance is currently absent and remains an
explicit human-admin follow-up.
**Next:** Complete independent review of PR #2 and do not self-merge. An
authorized maintainer should apply/verify the live `main` ruleset separately.
After the Discovery initialization is accepted, resume the fresh product
Discovery grilling using VibeFlow as target/baseplate, Replit as reference
evidence and the OSS ADOPT/HARVEST/REJECT matrix as advisory input. Keep the
application stack locked until the normal Architecture/ADR transition.

## 2026-09-13 — Product-owner Discovery definition promoted for review

**Context:** Issue #4, branch `discovery/issue-4-promote-product-definition`, draft PR #5.
**Did:** Promoted the product-owner Discovery synthesis into project-owned documentation: `docs/PRODUCT.md` now defines Greenfield2 as a mobile-first universal software-development interface with provider-owned development state; `docs/DOMAIN.md` records only the minimal Greenfield2-owned vocabulary; `FOUNDATIONS.md` and `README.md` were aligned to the same authority boundary. VibeFlow remains the interoperability/baseplate reference, Replit remains clean-room product/behavior evidence, and Happier remains an architecture/feasibility reference. No provider, application stack, protocol, framework, database, hosting target or application code was selected or added; `config/project.env` was not changed.
**Verified:** GitHub Actions run `34745639182` on draft PR #5 completed successfully on the documentation head before this memory append. `Foundation gate` ran `bash scripts/verify.sh` and reported `RESULT: PASS — 17 passed, 0 failed, 1 skipped`; the skip was AgentShield scanning zero Claude-config files, explicitly advisory. The same job ran `bash scripts/selftest.sh` and reported `SELFTEST: PASS — 128 cases behaved as asserted`. `Independent checks` also completed successfully, including the check that no application stack was introduced. A separate local attempt to clone the branch failed before any repository command could run because the tool container could not resolve `github.com`; local verification was therefore not claimed. This memory append changes the PR head and requires the normal CI gate to run again before review readiness is final.
**Learned:** The Discovery synthesis can be promoted without inheriting VibeFlow control-plane authority, Replit's entity model, or Happier's session/persistence authority. The durable product rule is that Greenfield2 is the interface and provider resources remain provider-authoritative; Greenfield2-owned state is limited to its own account/interface/connection/support/reference metadata. The first provider and all application-architecture choices remain intentionally unresolved.
**Next:** Confirm the post-memory PR head passes the GitHub `verify` workflow, update PR #5 with the final verification evidence, then send the draft for independent review. Do not merge or move `PROJECT_PHASE` to `architecture` as part of this Discovery-promotion PR; any lifecycle transition is separate reviewed work.

## 2026-09-13 — Entered Architecture lifecycle for review

**Context:** Issue #6, branch `lifecycle/issue-6-enter-architecture`, draft PR #7.
**Did:** Changed only the repository lifecycle from `PROJECT_PHASE=discovery` to `PROJECT_PHASE=architecture`; preserved `ALLOW_APP_STACK=0` and an empty `STACK_DECISION_ADR`; aligned `README.md`, `FOUNDATIONS.md`, `docs/PRODUCT.md`, `docs/ROADMAP.md`, and `docs/ARCHITECTURE.md` with the reviewed Discovery baseline and active Architecture stage. No provider, stack, protocol, framework, database, auth implementation, hosting target, UI technology or application code was selected or introduced. VibeFlow, Replit, Happier and other research remain evidence/reference inputs rather than inherited Architecture decisions.
**Verified:** GitHub Actions run `34746302562` on draft PR #7 completed successfully on the pre-memory head. `Foundation gate` ran `bash scripts/verify.sh` and reported `phase: architecture, allow_app_stack=0`, `RESULT: PASS — 17 passed, 0 failed, 1 skipped`; AgentShield scanned zero Claude-config files and was explicitly advisory. The same job ran `bash scripts/selftest.sh` and reported `SELFTEST: PASS — 128 cases behaved as asserted`, including `no_app_stack/rejected-src-dir-in-architecture exit 1`. `Independent checks` completed successfully, including `Confirm no application stack was introduced`. Security review found no new secret, network, dependency, CI, auth-execution or other runtime attack surface; the lifecycle transition does not weaken the no-stack guard. This memory append changes the PR head, so CI must pass again before review readiness is final.
**Learned:** The repository can enter Architecture as a pure state-and-documentation transition: Architecture is permission to research and decide, not permission to implement. The machine guard remains the enforcement boundary, and the accepted provider-authoritative product definition stays fixed unless separately amended through product review.
**Next:** Confirm CI passes on the post-memory PR head, re-read the final real diff for append-only memory and Issue #6 conformance, then mark PR #7 ready for independent review. Do not self-merge; provider selection and all application-stack choices are separate Architecture work.

## 2026-09-13 — Provider contract before provider selection

**Context:** Issue #9, branch `architecture/issue-9-provider-contract-first`, draft PR #10.
**Did:** Recorded ADR-0005 and aligned `docs/ARCHITECTURE.md`, `docs/ROADMAP.md`, the ADR index and README so Greenfield2 must define and validate a provider-neutral capability contract before selecting the first MVP provider. The contract is explicitly a capability/interface boundary rather than a Greenfield2-owned provider resource model; provider capability, connected-account entitlement and Greenfield2 UI support remain distinct. Validation must cover at least three materially different provider shapes before provider selection. Jules, Cursor, Replit and GitHub Copilot are current stress-test evidence only; VibeFlow, Replit, Happier and all provider research remain reference inputs rather than decision authority.
**Verified:** GitHub Actions run `34749346800` on PR #10 completed successfully on the pre-memory head `e54a46bd7a69854986c493e20a98ba22c377ac47`. `Foundation gate` ran `bash scripts/verify.sh` and reported `phase: architecture, allow_app_stack=0`, `RESULT: PASS — 17 passed, 0 failed, 1 skipped`; AgentShield scanned zero Claude-config files and was explicitly advisory. The same job ran `bash scripts/selftest.sh` and reported `SELFTEST: PASS — 128 cases behaved as asserted`. `Independent checks` completed successfully, including `Confirm no application stack was introduced`. A local clone/verification attempt failed before checkout because the tool container could not resolve `github.com`; local verification is not claimed. Before the branch was created, an attempted content-identical README write accidentally omitted the branch parameter and created direct-main commit `c65745def6a4a61e6bd52622c5501b5f8d9bf9dd`; GitHub reports `diff: null` and `files: null`, and the README blob SHA remained unchanged. No repository content changed in that commit; the history is preserved rather than rewritten. This memory append changes the PR head and therefore requires a fresh CI run before review readiness is final.
**Learned:** Selecting a provider before defining the provider boundary creates vendor-shaping risk even when the provider is described as only an MVP proving slice. Greenfield2's safer order is contract → heterogeneous provider validation → contract revision → provider selection → provider-specific adapter. The process incident also confirms branch parameters must be explicit on every GitHub contents write; a content-identical main commit is still a workflow violation even when it has no diff.
**Next:** Confirm the post-memory PR head passes GitHub Actions, verify the final diff keeps this memory change append-only, update PR #10 with final conformance/verification evidence, and mark it ready for independent review. Keep Issue #9 open for the actual capability-contract definition and multi-provider validation; do not select a provider or enable implementation in this PR.

## 2026-09-13 — Provider capability contract defined and validated

**Context:** Issue #9, branch `arena/01a09a43-greenfield2`. PR #10 had already
merged ADR-0005 and the sequencing docs, satisfying only acceptance criterion 1
and deliberately leaving Issue #9 open for the contract itself.

**Did:** Defined the provider-neutral capability contract as
`docs/architecture/PROVIDER_CAPABILITY_CONTRACT.md` v1.0 — capability catalogue
phrased as provider-neutral questions, a capability manifest as the contract's
root, opaque provider-native handles, three independently-sourced availability
layers (provider capability / account entitlement / Greenfield2 UI support),
declared update-delivery modes, adapter obligations, and ten anti-leakage
prohibitions with a reproducible audit. Validated it against four materially
different provider shapes in `docs/architecture/PROVIDER_SHAPE_VALIDATION.md`
(Google Jules, Cursor Cloud Agents, Replit MCP, GitHub Copilot), recording ten
revisions F1–F10. Recorded the structural choice as ADR-0006. Added
`docs/architecture/README.md`, indexed ADR-0006, and aligned
`docs/ARCHITECTURE.md` (contract now in a "decided" table; first-provider row
marked unblocked), `docs/ROADMAP.md` (two checkboxes ticked) and `README.md`.
No provider, transport, framework, database, auth implementation, hosting target
or UI technology was selected. `config/project.env` was not touched. No
`.ecc/**`, `scripts/**` or `.github/**` file was modified.

**Verified:** `bash scripts/verify.sh` → `RESULT: PASS — 16 passed, 0 failed,
2 skipped`; the skips were `shell_lint` (shellcheck absent locally; CI runs it)
and `agentshield` (0 Claude-config files, advisory). `bash scripts/selftest.sh`
→ `SELFTEST: PASS — 128 cases behaved as asserted`. `links` resolved 153
relative links across the enlarged doc set. The two anti-leakage audits in the
contract were executed, not just written: audit A returned no matches (exit 1);
audit B returned exactly 4 matches, all inside the sections that state the
prohibition. Sandbox egress to every vendor documentation host failed at the TLS
handshake (`curl https://jules.google/` → `SSL_ERROR_SYSCALL`), so provider
evidence came from the platform-side fetch/search tools against primary provider
documentation; each source is cited with its access date in the validation
record. No provider API was called.

**Learned:** Three environment traps worth recording. (1) `PyYAML` is absent in
the sandbox, so `workflows_yaml` SKIPs and `selftest.sh` genuinely FAILS on
`workflows_yaml/corrupted` — that failure is real, not cosmetic, and is fixed by
`pip install --break-system-packages PyYAML` (plain `pip install` is refused
under PEP 668). (2) `npm install -g shellcheck` is a trap: it installs a shim
that cannot download the real binary here (`unable to verify the first
certificate`), and because `verify.sh` only tests `command -v shellcheck`, the
broken shim turns an honest SKIP into a false FAIL across all five foundation
scripts. Uninstall it rather than trusting that FAIL. (3) The GitHub token
available here can read issues and write contents/PRs but cannot comment on
issues (`POST /issues/9/comments` → 403 "Resource not accessible by integration"),
so the Issue #9 plan went into the PR body instead, as `planning.md` permits.

On the substance: every apparent universal in the first contract draft turned
out to be a majority case. Work items, diffs, approvals, results, streaming and
entitlements each looked provider-neutral until a shape arrived that lacked them
or meant something different — Replit exposes no work item at all, and Cursor
changed its own resource topology and identifier format between `v0` and `v1`.
The durable lesson is that a contract validated without revisions is suspicious,
not successful.

**Next:** First-provider selection is now **unblocked and still open**; it is a
separate decision using the Product Fit + Integration Legitimacy gates in
`docs/PRODUCT.md`, and the validation set must not be inherited as the answer.
Re-verify the dated provider profiles before relying on any specific endpoint in
that decision — all four surfaces are pre-stable. Independent review should look
hardest at the §4.3 presentation liveness hint, which is the one place a shadow
state machine could start, and at whether `Absent` is honoured everywhere rather
than silently coerced. Do not self-merge.

## 2026-09-13 — Provider contract corrected after independent review (v1.0 → v1.1)

**Context:** Issue #9, branch `arena/01a09a43-greenfield2`, PR #11. Independent
review of the v1.0 contract returned five substantive findings plus one process
finding. The overall capability-contract approach was kept; the defects were
corrected.

**Did:** Contract promoted to **v1.1** with five corrections, each recorded as a
new finding F11–F15 in `PROVIDER_SHAPE_VALIDATION.md` §4b rather than patched
silently. **F11** — v1.0 said a provider with no change-inspection surface "may
still qualify" for the MVP loop; PRODUCT.md minimum V1 item 4 carries no "where
exposed" qualifier, so that relaxed an accepted product requirement. §5 now maps
every PRODUCT.md item to a capability with its exact qualifier, and `none` is a
Gate 1 disqualifier. **F12** — `unknown` entitlement was declared but undefined;
§3.2.1 now defines it and splits behaviour by risk (reads may be optimistic,
mutating/spend-bearing invocations may not be speculative or auto-retried and
must warn before commit), §3.2.2 enumerates five evidence kinds, and the manifest
gains a `mutates` flag. **F13** — the contract could not express mandatory
provider-native inputs (Replit requires `app_stack` from a fixed provider list);
new §5.9 adds `RequiredInput`/`InvocationContext`. **F14** — one `ProviderHandle`
type served both `connection.authorize` and resource references, but DOMAIN.md
defines **Provider Connection** and **Provider Resource Reference** as separate
entities; §4 is now split, with the connection legitimately carrying a
Greenfield2-owned lifecycle and the resource reference not. **F15** — "verbatim"
error pass-through contradicted PRODUCT.md's "readability **and safe
presentation**"; §8 now separates unaltered semantics (§8.1) from sanitized
presentation (§8.2) with a named risk table. Process: ADR-0006 was `accepted`
while on an unmerged branch, contrary to `.ecc/skills/decisions.md`; it is now
`proposed`. Added prohibitions P11–P14. No duplicate provider-selection issue
was created — **Issue #8 already exists** and is now referenced from the
contract, validation record, ADR-0006, ARCHITECTURE, ROADMAP and README.

**Verified:** `bash scripts/verify.sh` → `RESULT: PASS — 16 passed, 0 failed,
2 skipped`, with `links` resolving 163 relative links (was 153). `bash
scripts/selftest.sh` → `SELFTEST: PASS — 128 cases behaved as asserted`. Both
anti-leakage audits re-executed after the rewrite: audit A no matches (exit 1);
audit B exactly 4 matches, unchanged and confined to the sections stating the
prohibition. Section numbering re-checked end to end after the §4 split and §4.5
renumber. No provider selected; `config/project.env` untouched; no `.ecc/**`,
`scripts/**` or `.github/**` file modified.

**Learned:** The most useful lesson is that the two review passes found
**different classes** of defect. F1–F10 came from comparing the contract against
provider shapes and all say the same thing — every apparent universal was a
majority case. F11–F15 came from comparing it against Greenfield2's own accepted
product and domain truth, and passing the provider-shape test caught none of
them. A contract can be perfectly provider-neutral and still be wrong about the
product it exists to serve. Concretely: provider-neutrality pulled toward
"everything is optional and provider-defined", which quietly relaxed a product
requirement, merged two domain entities into one convenient type, and preferred
fidelity over safety in error text. Two more traps worth recording: a
`sed`-style blanket rename of `ProviderHandle` also rewrote
`connection.authorize`'s return type, which is exactly the conflation being
fixed — type splits need per-site review, not global replace; and an ADR marked
`accepted` on an unmerged branch is a real workflow violation even though
nothing depended on it yet.

**Next:** PR #11 is updated and left open for a second independent review. The
reviewer should check (a) whether the §5 qualifier table now matches PRODUCT.md
exactly, (b) whether §3.2.1's read/mutating split is the right risk boundary,
and (c) the referred product question: can `live-artifact` alone satisfy minimum
V1 item 4? ADR-0006 must be flipped to `accepted` and re-indexed only after the
PR merges. Provider selection remains Issue #8's job.
