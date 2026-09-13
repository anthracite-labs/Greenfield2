# Plan — Issue #8 Architecture: evaluate first MVP provider

Requirement: Evaluate candidate first providers for Greenfield2 MVP now that repository is formally in Architecture. Must use current first-party evidence, respect provider-authoritative product rule, apply Product Fit and Integration Legitimacy gates, and produce per-candidate evaluation covering auth, entitlement, lifecycle, execution, workflows, activity, diffs, approvals, output, reconnect, policy, evidence URLs, gate verdicts, and coupling risks. Provide concise shortlist and recommendation without silently selecting winner. No implementation, stack selection, or adapter code.

Acceptance (verbatim from Issue #8):

## Purpose
Evaluate candidate first providers for the Greenfield2 MVP now that the repository is formally in Architecture.

The first provider is an MVP proving slice, not Greenfield2's permanent identity.

## Accepted product gates
A candidate must pass both gates before it can be selected:

### Gate 1 — Product fit
The provider must expose enough supported capability for Greenfield2 to present the reviewed minimum V1 development loop:

1. start new work or continue existing provider work where exposed;
2. interact with the provider's Agent;
3. observe provider-reported progress / plan / tool activity where exposed;
4. inspect meaningful changed files and/or diffs;
5. respond to provider approval requests where exposed;
6. see a meaningful provider output/result such as preview, build, deployment, pull request, or equivalent;
7. reconnect to or continue provider-owned work where the provider supports continuity.

### Gate 2 — Integration legitimacy
The required MVP capability path must provide both:

1. a supported external-client / third-party integration surface; and
2. a legitimate way for the user's own account/entitlement to authorize and use that capability.

No private scraping, reverse-engineered first-party session APIs, entitlement bypass, pooled consumer accounts, or unsupported impersonation qualify.

## Architecture invariants
- Greenfield2 owns no provider development state.
- Provider-native resources, semantics, runtime, lifecycle and errors remain provider-authoritative.
- Authentication != authorization != entitlement.
- Rich provider-specific capability may remain provider-specific.
- No provider is selected by benchmark/model quality alone.
- VibeFlow, Replit and Happier are research/reference inputs only; they do not silently select the provider.
- This issue does not select the client/application stack.

## Initial research candidates
Revalidate against current first-party evidence rather than trusting prior snapshots:
- GitHub Copilot
- Google Jules
- Cursor Cloud Agents
- Replit
- OpenAI Codex
- any additional candidate that currently passes both gates materially better

## Evaluation output
For each serious candidate record:
- supported auth path;
- entitlement/billing relationship;
- Agent/session lifecycle surface;
- provider-hosted execution/workspace behavior;
- new/continue workflow support;
- observable activity/messages/tool events;
- files/diffs;
- approvals;
- meaningful output (preview/build/deploy/PR/equivalent);
- reconnect/resume/background behavior;
- known policy/terms constraints;
- first-party evidence URLs and verification date;
- Gate 1 verdict;
- Gate 2 verdict;
- major architectural coupling/risks.

## Decision rule
Do not silently choose a winner. Research should produce a concise shortlist and recommendation. The product owner and Greenfield2 review explicitly decide the first provider.

## Out of scope
- application stack selection;
- provider adapter implementation;
- provider-specific source code;
- database/auth/hosting choices;
- multi-provider MVP support;
- BYOK/BYOA/BYOW composition implementation;
- implementation lifecycle transition.

Approach:
- Use research skill channel order: repo first, then GitHub via gh api, then package registries, then web search/fetch for current provider docs. Record verification date and URLs.
- Validate each candidate against PROVIDER_CAPABILITY_CONTRACT.md v1.1 (accepted) and PRODUCT.md minimum V1 loop mapping (including §5.5.1 change-inspection fulfilment kinds and approval required-if-exposed).
- Produce evaluation document docs/architecture/FIRST_PROVIDER_EVALUATION.md with per-candidate sections covering all required fields, plus summary table and shortlist recommendation. Do not select provider; leave decision to product owner.
- Update docs/architecture/README.md index and docs/ARCHITECTURE.md open-decisions reference if needed (no provider selection).
- Verify with bash scripts/verify.sh and selftest.
- Open PR referencing Closes #8, leave open for independent review.

Phases:
1. Research — gather current first-party evidence via web_search and fetch_page for Jules, Cursor, Copilot, Replit, Codex; record evidence URLs and 2026-09-13 verification date. Verify: search results contain primary docs.
2. Draft evaluation — write docs/architecture/FIRST_PROVIDER_EVALUATION.md with full per-candidate fields, gate verdicts, risks, summary, recommendation. Verify: file exists, links resolve.
3. Index alignment — update docs/architecture/README.md to list new evaluation doc, ensure no stack selection. Verify: bash scripts/verify.sh --only=links
4. Verification — run bash scripts/verify.sh full and bash scripts/selftest.sh. Verify: PASS.
5. PR — commit, push branch arena/01a09b90-greenfield2, open PR with template, verification output, known limitations. Verify: gh pr view.

Risks/unknowns:
- Sandbox egress allowlist blocks vendor doc hosts at TLS handshake (curl SSL_ERROR_SYSCALL); must rely on platform-side fetch/search tools for primary evidence, not live API calls.
- Provider surfaces pre-stable: Jules v1alpha, Cursor v1 public beta + concurrent v0, Copilot public preview + feature flags, Replit MCP unversioned. Docs may drift from implementation.
- Cursor diff surface: public API exposes branch reference, not inline diff; need spike note.
- Jules auth is API-key header, not OAuth; paid plans limited to @gmail.com individual accounts currently.
- OpenAI Codex hosted path unverified for third-party use; local App Server would violate MVP non-goal of no Greenfield2-owned runtime.
- Replit MCP only 8 tools, no files/diffs/progress/approval surface; would require private interfaces which are prohibited.
- Must not introduce application stack, framework, database, hosting; keep ALLOW_APP_STACK=0.
- No provider API calls; documentation-level validation only.

Out of scope:
- Application stack selection, provider adapter implementation, provider-specific source code, database/auth/hosting choices, multi-provider MVP support, BYOK/BYOA/BYOW composition implementation, implementation lifecycle transition, merging PR.
