# First Provider Evaluation — Issue #8

**Status:** Draft evaluation, product-owner decision pending. Selects no provider.
**Lifecycle:** `PROJECT_PHASE=architecture`, `ALLOW_APP_STACK=0`. No stack, transport, framework, database, auth implementation, hosting, UI technology, or provider selected.
**Issue:** [#8 — Architecture: evaluate first MVP provider](https://github.com/anthracite-labs/Greenfield2/issues/8)
**Contract:** [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) v1.1 (accepted), [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) four shapes, fifteen revisions.
**Product gates:** [PRODUCT.md](../PRODUCT.md) minimum V1 loop (7 items) and Integration Legitimacy (supported external-client path + legitimate entitlement).
**Verification date:** 2026-09-13 (UTC) — all first-party URLs fetched via platform fetch/search on this date; sandbox egress to vendor doc hosts fails TLS handshake (`curl https://jules.google/ → SSL_ERROR_SYSCALL`), so no provider API was called. Documentation-level validation only.
**Decision rule:** Do not silently choose a winner. This document produces shortlist and recommendation; product owner and Greenfield2 review explicitly decide.

---

## Summary

| Candidate | Gate 1 — Product Fit | Gate 2 — Integration Legitimacy | Current disposition |
| :--- | :--- | :--- | :--- |
| **Google Jules** | **PASS** — all 7 V1 items via REST API: Session lifecycle, Activities (plan/progress/messages), inline-patch ChangeSet, opt-in plan approval, PR result, list/continue/reconnect | **PASS** — official REST API v1alpha, user-generated API key, provider-direct billing via Google AI Pro/Ultra, GitHub source connection via Jules GitHub App | **Finalist — best documented semantic match to V1 UI** |
| **Cursor Cloud Agents** | **PASS, with one diff-surface caveat to validate in spike** — durable Agent + Runs, rich SSE tool stream, branch/PR result, follow-ups, resume via Last-Event-ID; change inspection is remote-reference (branch) not inline-patch; approval none exposed (required-if-exposed) | **PASS** — official Cloud Agents API v1 public beta, Basic + Bearer auth, user API key + service-account keys + sub-tokens, paid Cursor plan required, usage-priced | **Finalist — best provider-hosted workspace/runtime** |
| **GitHub Copilot cloud agent** | **PARTIAL** — start/list/get tasks, PR result, remote-reference diffs via PR APIs, waiting_for_user; lacks rich transcript/tool/diff/approval surface in task API itself | **PASS** — official Agent Tasks REST API public preview, user-to-server tokens (PAT, OAuth, GitHub App user token), paid Copilot plans, explicit legitimacy | **Watch / not first-provider choice yet** |
| **Replit MCP** | **PARTIAL** — App-shaped (no work item), create/update/publish, live public URL result, but no files/diffs, no progress/activity, no approvals exposed | **PASS** — official MCP server over Streamable HTTP, OAuth 2.1 + PKCE with protected-resource discovery, Free/Core/Pro/Enterprise accounts | **Watch / fails PRODUCT.md item 4 (changed files/diffs)** |
| **OpenAI Codex App Server + hosted Codex** | **TECHNICALLY STRONG for local execution** — threads/turns/items, streaming, diffs, approvals, file changes; hosted third-party path not documented | **UNVERIFIED for third-party use of OpenAI-hosted runtime** — App Server runs on your own infra; running it in Greenfield2-owned compute violates MVP non-goal of no Greenfield2 runtime | **Validation/partner track** |

No additional candidate was found that materially passes both gates better than the two finalists on current first-party evidence.

---

## 1. Google Jules — FINALIST

### Supported auth path
- Official Jules REST API, base `https://jules.googleapis.com/v1alpha/`.
- Auth: API key in `x-goog-api-key` header. Generated in Jules web app Settings → API, max 3 keys at a time. Keep secure; publicly exposed keys auto-disabled per Cloud docs.
- Source connection: Before using a source via API, install Jules GitHub App through Jules web app. List sources via `GET /v1alpha/sources`.

### Entitlement/billing relationship
- User contracts directly with Google/Jules. Greenfield2 does not provision, bundle, resell, or fund provider subscriptions.
- Jules Plans accessed via Google AI Plans subscription, currently available only for individual Google Accounts ending in @gmail.com. Business/Workspace users directed to interest form.
- Plan comparison (from https://jules.google/docs/usage-limits):
  - Free Jules: 15 daily tasks (rolling 24h), 3 concurrent, Gemini 2.5 Pro.
  - Jules in Pro (Google AI Pro): 100 daily, 15 concurrent, higher access to latest model (Gemini 3 Pro).
  - Jules in Ultra (Google AI Ultra): 300 daily, 60 concurrent, priority access to latest model.
- Paid access via Google One AI plans. Task limits not shared in family plans. Age 18+ required.

### Agent/session lifecycle surface
- Core resources: Source, Session, Activity.
- Session: contiguous unit of work. Fields: name `sessions/{id}`, id, prompt (required), title, state, url (Jules web app), sourceContext (required SourceContext: source `sources/{id}` + githubRepoContext startingBranch), requirePlanApproval, automationMode, outputs, createTime, updateTime.
- SessionState enum: STATE_UNSPECIFIED, QUEUED, PLANNING, AWAITING_PLAN_APPROVAL, AWAITING_USER_FEEDBACK, IN_PROGRESS, PAUSED, FAILED, COMPLETED.
- Operations: Create Session `POST /v1alpha/sessions`, List `GET /v1alpha/sessions`, Get `GET /v1alpha/sessions/{id}`, Delete `DELETE`, Send Message `POST :sendMessage`, Approve Plan `POST :approvePlan`.
- Repoless sessions supported (sourceContext optional).

### Provider-hosted execution/workspace behavior
- Jules executes coding task in Google-managed environment against connected GitHub repository. No explicit VM details exposed; execution is task-oriented rather than generic always-on workspace shell.
- AutomationMode: AUTO_CREATE_PR automatically creates pull requests when changes ready; otherwise no PR auto-created.

### New/continue workflow support
- New: Create session with prompt + sourceContext + optional title, requirePlanApproval, automationMode.
- Continue: When session in AWAITING_USER_FEEDBACK, send follow-up via `:sendMessage`. List/get sessions to review existing work. Delete to clean.
- Pagination: pageSize 1-100, pageToken.

### Observable activity/messages/tool events
- Activities API: `GET /v1alpha/sessions/{sessionId}/activities`, `GET /v1alpha/sessions/{sessionId}/activities/{activityId}`.
- Activity: name `sessions/{id}/activities/{id}`, id, originator `user|agent|system`, description, createTime, artifacts[], plus exactly one event field: planGenerated (plan id + steps[] id/index/title/description/createTime), planApproved (planId), userMessaged (userMessage), agentMessaged (agentMessage), progressUpdated (title/description), sessionCompleted, sessionFailed (reason).
- Progress telemetry via progressUpdated activities, plan steps, messages.

### Files/diffs
- Artifact types: changeSet (source + gitPatch baseCommitId/unidiffPatch/suggestedCommitMessage) — inline-patch fulfilment; bashOutput (command/output/exitCode); media (mimeType/data base64).
- Contract mapping: `changes.inspect` → `inline-patch`. Satisfies PRODUCT.md item 4 (changed files/diffs) without qualification. Direct evidence of meaningful change.

### Approvals
- Explicit opt-in plan gate: `requirePlanApproval: true` makes plans require explicit approval; if not set, auto-approved.
- Endpoint `:approvePlan` approves latest plan. Plan generation observable via planGenerated activity.
- This is pre-execution plan approval, distinct from PR review. Where provider has no other approval mechanism, this satisfies approval where exposed (PRODUCT.md item 5 carries "where exposed" qualifier).

### Meaningful output
- SessionOutput: pullRequest url/title/description. When automationMode AUTO_CREATE_PR, PR auto-created.
- PR URL is provider-native result, typed as pull request. Also url field to view session in Jules web app for handoff.

### Reconnect/resume/background behavior
- Sessions retained, listable, gettable. History preserved. Can review/manage existing tasks.
- Background execution in provider; polling via GetSession/ListSessions. No SSE stream; snapshot/poll delivery.
- Continue via sendMessage when AWAITING_USER_FEEDBACK; otherwise new session.

### Known policy/terms constraints
- API in alpha, experimental, may change specs, keys, definitions. At least one stable + one experimental version planned.
- API key security: do not embed in public code; exposed keys auto-disabled.
- Paid plans only for @gmail.com individual accounts; enterprise path not yet GA.
- GitHub Sources must first be connected via Jules web app and GitHub App installation.
- Repository/task oriented, not generic workspace; no always-on VM.
- Age 18+ requirement; stricter than some Google One plans.

### First-party evidence URLs and verification date
- https://jules.google/docs/api/reference/ — Quickstart, auth, concepts — fetched 2026-09-13
- https://jules.google/docs/api/reference/sessions/ — Create/List/Get/Delete/SendMessage/ApprovePlan — fetched 2026-09-13
- https://jules.google/docs/api/reference/activities/ — List/Get Activities, types, artifacts — fetched 2026-09-13
- https://jules.google/docs/api/reference/types/ — Session, SessionState, Activity, Artifact, ChangeSet, GitPatch — fetched 2026-09-13
- https://jules.google/docs/usage-limits — Plans, daily tasks, concurrent tasks, upgrade path — fetched 2026-09-13
- https://developers.google.com/jules/api — API concepts mirror — fetched 2026-09-13
- Verification date: 2026-09-13

### Gate 1 verdict
**PASS** — Maps directly to V1 loop:
1. start new / continue existing where exposed: create session, list/get existing, sendMessage continue — yes.
2. interact with provider's Agent: agentMessaged, userMessaged via sendMessage — yes, unqualified required.
3. observe progress/plan/tool activity where exposed: planGenerated with steps, progressUpdated, agent messages — yes.
4. inspect meaningful changed files/diffs: ChangeSet.gitPatch.unidiffPatch inline — yes, unqualified required, fulfilment inline-patch.
5. respond to approval requests where exposed: requirePlanApproval + approvePlan — yes, where exposed.
6. meaningful provider result: pullRequest output + AUTO_CREATE_PR — yes, unqualified required.
7. reconnect/continue where supported: list/get/sendMessage — yes, where supported.

### Gate 2 verdict
**PASS** — Supported external-client path: official Jules REST API v1alpha documented. Legitimate entitlement: user-generated API key tied to user's Google/Jules account; paid access via Google AI Pro/Ultra with documented limits; provider truth governs.

### Major architectural coupling/risks
- Alpha API may change (resource names, states, auth).
- API-key onboarding less polished than OAuth; max 3 keys.
- GitHub-only sources currently; no GitLab/Azure/Bitbucket.
- Paid plan Gmail-only restricts enterprise users.
- Task-oriented not workspace-shell; no generic terminal/browser VM.
- No entitlement endpoint; L2 unknown until attempt (per contract §3.2).
- Poll-only delivery, no resumable stream.

---

## 2. Cursor Cloud Agents — FINALIST

### Supported auth path
- Official Cloud Agents API v1, base `https://api.cursor.com/v1/`, also legacy v0.
- Auth: accepts both Basic and Bearer. Basic: `-u YOUR_API_KEY:` (API key as username). Bearer: `Authorization: Bearer YOUR_API_KEY`.
- User API key from Cursor Dashboard → API Keys. Service-account API keys for team scenarios (Enterprise). Worker sub-tokens via `POST /v1/sub-tokens` mint one-hour user-scoped worker tokens.
- OpenAPI spec at https://cursor.com/docs-static/cloud-agents-openapi.yaml.

### Entitlement/billing relationship
- Requires paid Cursor plan. Cloud Agents require paid plan; Hobby free does not satisfy.
- Pricing (reviewed 2026-08-31 via secondary but consistent with primary docs stating paid plan required):
  - Pro $20/mo includes $20 guaranteed API usage + bonus.
  - Pro+ $60/mo includes $70 guaranteed + 3x multiplier on major frontier models.
  - Ultra $200/mo includes $400 guaranteed + 20x on supported models.
  - Teams Standard $40/user/mo, Premium $120/user/mo (5x limits), Enterprise custom.
- Billing: selected-model API rates consume agent usage; users set spend limit before starting. On-demand overage at same per-token rates when pool exhausted. Two pools: Cursor's own models and third-party models (Claude, GPT, Gemini). Max Mode adds 20% surcharge for some runs. Image inputs max 5, 15MB each counted.
- Provider-direct: Greenfield2 does not resell usage; user contracts with Cursor.

### Agent/session lifecycle surface
- Two-level: durable Agent + per-prompt Run. First-party: "This API splits work into a durable agent plus per-prompt runs, replacing the flatter v0 surface."
- Agent: id `bc-<uuid>` (v1) vs `bc_abc123` (v0) — format changed across versions; opaque. Fields: id, name (max 100 chars, auto-derived from prompt if omitted), status ACTIVE/IDLE/ARCHIVED, env type cloud/pool/machine + name, repos[] url (required) + startingRef + prUrl + workOnCurrentBranch + autoCreatePR + skipReviewerRequest + envVars (max 50, encrypted at rest, session-scoped, cannot combine with client-supplied agentId), mcpServers[] (max 50, remote http/sse or stdio with headers/auth/env), customSubagents[] (max 20, name/description/prompt/model), mode agent/plan, agentId client-supplied for idempotent create (returns 409 agent_id_conflict if duplicate), url (cursor.com/agents/...), createdAt/updatedAt/latestRunId.
- Run: id `run-<uuid>`, agentId, status CREATING/RUNNING/FINISHED/CANCELLED/ERROR/EXPIRED, createdAt/updatedAt/durationMs/result text/git branches[] (repoUrl without scheme, branch, prUrl).
- Operations: Create Agent `POST /v1/agents` returns agent+run, List Agents `GET /v1/agents` (limit default 20 max 100, cursor pagination, prUrl filter, includeArchived), Get Agent `GET /v1/agents/{id}`, Archive/Unarchive, Delete permanently, Create Run `POST /v1/agents/{id}/runs` (follow-up prompt, mcpServers override, mode override), List Runs, Get Run, Stream Run `GET /v1/agents/{id}/runs/{runId}/stream` SSE, Cancel Run `POST .../cancel` (terminal, cannot resume, 409 if not cancellable), List/Download Artifacts (agent-scoped, workspace persists across runs, v1 paths relative), Get Agent Usage `GET /v1/agents/{id}/usage` (totalUsage inputTokens/outputTokens/cacheWriteTokens/cacheReadTokens/totalTokens, runs[] with usageUuid), List Models `GET /v1/models`, List GitHub Repositories `GET /v1/repositories` (very strict rate limits: 1/user/min, 30/user/hour, tens of seconds for many repos), Worker/Pool management (List Workers, Get Worker Summary, List Pools, Register/Deregister Pool, List Pending Pool Requests, Watch Pending via SSE, Claim/Release).
- Cancellation terminal: "cannot be resumed. To continue the conversation, create a new run on the same agent."

### Provider-hosted execution/workspace behavior
- Dedicated Firecracker-based cloud VM per agent. Workspace persists across runs.
- Can build/test/interact with software, use browser/computer controls, run terminal commands, read files.
- MCP servers inline available to agent.
- Session-scoped env vars encrypted at rest, injected into agent shell, deleted with agent.
- Self-hosted pools/machines optional; any-repo pools supported.
- Internet access on by default, commands auto-run.

### New/continue workflow support
- New: Create agent with prompt text + optional images (base64 or URL), model id + params (discover via GET /v1/models), repos[] (url required, startingRef optional, prUrl optional, workOnCurrentBranch bool, autoCreatePR bool), env (cloud/pool/machine), name, envVars, mcpServers, customSubagents, mode, agentId for idempotency.
- Continue: Create run (follow-up) on same agent with new prompt, using current conversation/workspace state. Only one run active per agent; 409 agent_busy if CREATING/RUNNING. Wait or cancel.
- List agents newest first to continue existing.

### Observable activity/messages/tool events
- Stream A Run SSE: `Accept: text/event-stream`. Events:
  - status: {runId, status}
  - assistant: {text} delta
  - thinking: {text} delta
  - tool_call: {callId, name (read_file, run_terminal_cmd, mcp, ...), status running/completed, args JsonValue, result JsonValue, truncated {args?, result?}}
  - interaction_update: richer SDK-shape event (text-delta, tool-call-started/completed, step-started/completed, turn-ended) — matches TypeScript SDK.
  - heartbeat: {}
  - result: {runId, status, text?, durationMs?, git?} terminal
  - error: {code, message}
  - done: {}
- Most events include id line opaque (e.g., 1713033006000-0) — treat as opaque, do not parse.
- Resumable via Last-Event-ID header set to most recent id; must belong to requested run else 400 invalid_last_event_id. After resume, expect another status event before resumed range.
- Retention: header X-Cursor-Stream-Retention-Seconds; after window, 410 stream_expired, remedy is read terminal state via Get A Run, not retry stream.
- Get A Run returns final result text, duration, git branches.

### Files/diffs
- No explicit diff surface in public API docs fetched. Result carries git.branches[] reference (repoUrl/branch/prUrl) — remote-reference fulfilment.
- Artifacts: agent-scoped files (e.g., artifacts/screenshot.png), path relative to workspace artifacts/ directory, because workspace persists across runs. List artifacts, download artifact.
- Change inspection is not inline-patch; it is branch reference + artifacts. Contract says remote-reference satisfies PRODUCT.md item 4, but Greenfield2 would need to render diff via GitHub PR APIs or show branch. Need validation spike to prove exact external projection for user-facing changed-files/diff view.
- RepoUrl returned without scheme (github.com/...), different from request which keeps https://.

### Approvals
- None observed in Cloud Agents API surface (validated 2026-09-13). No plan gate, no tool permission prompt in this API.
- Where exposed qualifier: if provider has no approval mechanism, Greenfield2 must not invent one. So not a disqualifier, but reduces richness vs Jules.

### Meaningful output
- Git branch pushed: git.branches[] with branch name and prUrl.
- PR auto-creation via autoCreatePR true, or via agent target configuration.
- Screenshots and other artifacts listable/downloadable.
- Result text final assistant reply.

### Reconnect/resume/background behavior
- Durable agent, workspace persists across runs, conversation snapshots retained, supports later resume via new run.
- Stream resumable via Last-Event-ID while retained; retention window limited.
- Background work in cloud VM; agent ACTIVE status means keep machine up, IDLE means may hibernate/snapshot.
- Cancel terminal, need new run to continue.
- Poll via Get Agent, List Runs, Get Run.

### Known policy/terms constraints
- v1 API public beta: "APIs may change before general availability". v0 and v1 concurrent.
- Billing usage-priced; long context, retries, premium models expensive; single run can consume ~22.5% of $20 Pro credit pool per secondary analysis.
- List GitHub Repositories very strict rate limits (1/min, 30/hour), can take tens of seconds.
- Image inputs max 5, 15MB each; web attachments separate limits.
- Internet access on by default, auto-run commands → prompt-injection and data-exfiltration risk unless egress and secrets tightly scoped.
- Full repo and execution environment placed in Cursor-managed cloud infra — may be unacceptable for some code/contractual boundaries.
- envVars rolling out, silently ignored if not enabled for account — verify before relying in production.
- Unknown pool name returns 400 not queuing forever.
- Identifier format changed between versions (bc_abc123 → bc-<uuid>) and artifact path semantics changed (relative vs absolute).

### First-party evidence URLs and verification date
- https://cursor.com/docs/cloud-agent/api/endpoints — Create/List/Get Agent, Create/List/Get/Stream/Cancel Run, Artifacts, Usage, Models, Repositories, Workers/Pools — fetched 2026-09-13 (chunks 0-5)
- https://cursor.com/docs/cloud-agent — capabilities overview
- https://cursor.com/docs/api — auth (Basic + Bearer), rate limits, best practices
- https://cursor.com/docs-static/cloud-agents-openapi.yaml — OpenAPI spec
- Verification date: 2026-09-13

### Gate 1 verdict
**PASS, with one diff-surface caveat to validate in spike**:
1. start/continue where exposed: create agent, create run follow-up, list agents — yes.
2. interact with Agent: assistant SSE, prompt follow-up — yes.
3. observe progress/plan/tool activity where exposed: SSE tool_call events with args/result, status, thinking, interaction_update — yes, richest of candidates.
4. inspect changed files/diffs: remote-reference via git.branches[] + artifacts, not inline-patch — satisfies PRODUCT.md item 4 per contract (remote-reference qualifies as change inspection), but exact user-facing diff projection needs spike.
5. respond to approval where exposed: none exposed in this API — required-if-exposed, so not disqualifier.
6. meaningful result: branch/PR + result text + artifacts — yes.
7. reconnect/continue where supported: durable agent, new run on same agent, resumable SSE — yes.

### Gate 2 verdict
**PASS** — Official Cloud Agents API v1 public beta, user-scoped API keys officially supported, service-account keys for team, sub-tokens for workers. Paid Cursor plan required, provider-direct billing with spend limit, legitimate entitlement. Supported external-client path.

### Major architectural coupling/risks
- Beta instability, concurrent v0/v1.
- Usage-based billing at model API rates, cost variable, spend limit required.
- No inline diff surface; remote-reference only.
- No approval mechanism exposed.
- Rate limits on repository listing.
- Security: internet on by default, auto-run, need egress/secrets scoping.
- Workspace in Cursor-managed infra.

---

## 3. GitHub Copilot cloud agent — GATE 1 PARTIAL / GATE 2 PASS

### Supported auth path
- Agent Tasks REST API: `POST /agents/repos/{owner}/{repo}/tasks`, `GET /agents/repos/{owner}/{repo}/tasks`, `GET /agents/tasks` (across repos), `GET /agents/repos/{owner}/{repo}/tasks/{id}`.
- Auth: user-to-server tokens only — PAT classic, fine-grained PAT, OAuth app token, GitHub App user-to-server token. Server-to-server GitHub App installation access tokens not supported (intentional human credential in chain while security model matures).
- Fine-grained PAT permission: "Agent tasks" repository permissions read (list) and read+write (start). For issue assignment path: read metadata + read/write actions, contents, issues, pull requests.
- Issue assignment via GraphQL mutations: createIssue, updateIssue, addAssigneesToAssignable, replaceActorsForAssignable with agentAssignment input (targetRepositoryId/baseRef/customInstructions/customAgent/model) and required header `GraphQL-Features: issues_copilot_assignment_api_support,coding_agent_model_selection`. REST issue assignees: `POST /repos/{owner}/{repo}/issues/{number}/assignees` with assignee `copilot-swe-agent[bot]` + agent_assignment.

### Entitlement/billing relationship
- Available for all paid Copilot plans per docs (note: changelog May 13 says Business/Enterprise only, docs now says all paid — discrepancy noted, re-verify at implementation).
- Copilot plans (as of 2026-09-13 primary docs):
  - Free: allowance of AI credits, limited.
  - Pro $10/mo: Base 1000 credits + 500 flex = 1500 total.
  - Pro+ $39/mo: Base 3900 + 3100 flex = 7000 total.
  - Max $100/mo: Base 10000 + 10000 flex = 20000 total.
  - Business $19/seat/mo: 1900 credits/seat pooled.
  - Enterprise $39/seat/mo + $21/seat GHE Cloud = ~$60 real floor: 3900 credits/seat pooled.
- Usage measured in GitHub AI Credits (1 credit = $0.01). Chat, agent mode, code review, CLI draw from pool; completions unlimited unmetered on paid plans.
- Enablement: available in all repos stored on GitHub except managed user accounts or explicitly disabled. Discoverable via GraphQL `suggestedActors(capabilities: [CAN_BE_ASSIGNED])` — if enabled, first node login `copilot-swe-agent`.
- Provider-direct billing.

### Agent/session lifecycle surface
- Task as unit: create task returns id, url, html_url, name, creator, owner, repository, state, session_count, artifacts[], archived_at, created_at, updated_at.
- Task states: queued, in_progress, completed, failed, idle, waiting_for_user, timed_out, cancelled.
- Operations: Start task (prompt required, base_ref, model, custom_agent, create_pull_request, head_ref), List tasks for repo (params per_page, page, sort updated_at/created_at, direction, state filter, is_archived, since, creator_id), List tasks across all repos, Get task, Check status (state).
- Issue path: assign issue to Copilot bot, optional custom agent via `.github/agents/*.agent.md` filename without extension.
- Model selection: auto if omitted; supported values evolve: claude-sonnet-4.6, claude-opus-4.6, gpt-5.2-codex, gpt-5.3-codex, gpt-5.4, claude-sonnet-4.5, claude-opus-4.5 per docs.

### Provider-hosted execution/workspace behavior
- Ephemeral, firewalled environment powered by GitHub Actions, where agent explores code, makes changes, executes tests/linters, runs automated security scanning (CodeQL, secret scanning, dependency analysis).
- Firewall enabled by default to prevent data exfiltration.
- External integrations: MCP like workIQ and Microsoft 365, external apps Teams/Linear/Slack/Jira can assign tasks and track progress.

### New/continue workflow support
- New: POST with prompt, base_ref, model, create_pull_request bool, head_ref for existing branch/PR context.
- Continue: No sendMessage equivalent on task API. Continue via commenting on PR, or creating new task, or PR iteration. waiting_for_user state allows clarification mid-task.
- List tasks to continue existing.

### Observable activity/messages/tool events
- Task API itself exposes state, session_count, artifacts. Detailed transcript/tool events not in task API docs fetched.
- Separate Copilot SDK exposes rich hooks, permissions, steerable sessions, on_permission_request callback (in-flight tool permission) — but SDK is different surface from cloud-agent task API. Must not combine two surfaces on paper to pretend one supported session contract (per contract anti-leakage).
- Session logs available via other GitHub surfaces, but not documented as part of task API.

### Files/diffs
- PR diff via GitHub's own pull-request APIs — remote-reference fulfilment.
- Artifacts array: provider github, type pull, data id. So result is PR reference.

### Approvals
- No pre-execution plan gate in task API. Provider's gate is post-hoc human code review on draft PR, with iteration via PR comments.
- SDK has tool permission prompt, but not task API.
- Where exposed: approval is PR review, which is human review outside task API.

### Meaningful output
- Draft pull request (artifacts[].data.id, html_url). Branch created.
- Optionally auto-create PR via create_pull_request true.
- Also supports custom agents.

### Reconnect/resume/background behavior
- Task history via list, get. Continue via PR comments or new task. Task state polling.
- Background execution in GitHub Actions environment.

### Known policy/terms constraints
- Public preview, subject to change. API versioned (2026-03-10).
- Authentication user-to-server only; no installation tokens yet; human credential required.
- Available only in repos on GitHub, not other git hosts.
- Excludes managed user accounts and where explicitly disabled.
- Requires GraphQL-Features header for issue assignment.
- Model list evolves; cost control via model parameter.
- Audit endpoint GET returns full agent config (MCP servers, tools, Actions workflow policy, firewall rules) — useful for security/platform teams.
- Combining task API + SDK + PR APIs as one contract would be false completeness.

### First-party evidence URLs and verification date
- https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api — starting, listing, checking status, issues API — fetched 2026-09-13
- https://docs.github.com/en/rest/agent-tasks/agent-tasks — List tasks, Start task, params, auth, states — fetched 2026-09-13 (version 2026-03-10)
- https://github.blog/changelog/2026-05-13-start-copilot-cloud-agent-tasks-via-the-rest-api — Business/Enterprise public preview announcement — fetched via search 2026-09-13
- https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ — Pro/Pro+/Max availability — fetched via search 2026-09-13
- https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent — overview, benefits, integrations — fetched via search 2026-09-13
- Verification date: 2026-09-13

### Gate 1 verdict
**PARTIAL** — Strong on 1,2,6,7 but weak on 3,4,5 for task API alone:
1. start/continue where exposed: start task, list tasks, comment on PR — yes.
2. interact with Agent: via PR comments and follow-up prompts during session — partial, but supported.
3. observe progress/plan/tool activity where exposed: task state only (queued/in_progress/completed etc.) + session_count, no typed activity stream like Jules/Cursor in task API — partial, does not satisfy rich observation.
4. inspect changed files/diffs: PR diff via GitHub PR APIs remote-reference — yes, but not inline.
5. respond to approval where exposed: post-hoc PR review, not explicit API approval — partial.
6. meaningful result: draft PR artifact — yes.
7. reconnect/continue where supported: via PR comments/new task — yes.

Overall does not yet present workspace-level development loop with observable progress/plan/tool activity through single supported external surface. SDK would fill gap but is separate surface.

### Gate 2 verdict
**PASS** — Official REST API public preview, explicit legitimacy: user-to-server tokens, Copilot subscription required, paid plans, per-repo enablement queryable, firewall, security scanning. Excellent integration legitimacy, best of candidates on auth legitimacy documentation.

### Major architectural coupling/risks
- Public preview may change, versioned API.
- Task API alone insufficient for rich observation; need to avoid combining SDK + task API as one contract.
- No installation token support yet — human token required, complicates headless automation.
- Repo must be on GitHub, excludes other hosts.
- Model selection cost lever but list evolves.

---

## 4. Replit MCP — GATE 1 PARTIAL / GATE 2 PASS

### Supported auth path
- Official Replit MCP Server, remote server URL `https://replit-mcp.com/server/mcp`.
- Transport: Streamable HTTP (MCP).
- Auth: OAuth using protected-resource discovery (OAuth 2.1 + PKCE). Client reads Replit's protected-resource metadata, prompts sign-in to Replit. Do not create custom OAuth server; Replit provides OAuth metadata.
- Supports ChatGPT, Claude, Slack native integrations plus any MCP client supporting Streamable HTTP + OAuth.
- Access scoped to apps user can edit, including shared.

### Entitlement/billing relationship
- User's Replit account: Free, Core, Pro, Enterprise. User contracts and pays Replit directly.
- Greenfield2 does not resell.
- Billing relationship provider-direct.

### Agent/session lifecycle surface
- Primary unit: App — long-lived resource. No run/session/task concept exposed at all.
- Tools (8 total):
  - create_app_from_prompt: required appDescription + app_stack (choice list), optional userSpecifiedAppName, userQuotes, attachmentSummary, sourceReplId (private copy).
  - update_app_using_prompt: required replId + changeDescription, optional userQuotes, attachmentSummary.
  - publish_app: replId.
  - get_publish_status: replId — check publish status + public URL.
  - list_apps: optional query, limit (default 25 max 50) — list apps you can edit, ordered recent activity.
  - search_apps: optional query, url, updatedAfter, updatedBefore, limit 1-50.
  - resolve_app_by_name: name exact title.
  - ask_question: replId + question — ask Agent about app without changing.
- No state enum. Work async, progress observable only indirectly: "A create, update, or publish request is still running — Wait before retrying."

### Provider-hosted execution/workspace behavior
- Replit Agent builds app in Replit cloud-hosted project (code, data, assets), secure isolated environment, auto-save, version control, collaboration, publishing to cloud with single click.
- Project Editor tools: AI-powered, collaboration, publishing.
- Zero-setup, pre-configured environments.

### New/continue workflow support
- New: create_app_from_prompt with appDescription + app_stack required. app_stack must be one of fixed provider list: react_website, mobile_app, design, slides, animation, data_visualization, 3d_game, document, spreadsheet — provider-owned taxonomy.
- Continue: update_app_using_prompt mutates existing app (replId). search/list/resolve to find existing. Cross-client continuation using same replId.
- No work item to reopen; App is long-lived.

### Observable activity/messages/tool events
- None exposed in current MCP surface. Closest is get_publish_status reports publish state + public URL, not agent activity.
- ask_question asks agent about app without changing, but no structured activity stream.

### Files/diffs
- None exposed. No file, diff, or patch surface in MCP tools.
- Published app is observable output, not diff.

### Approvals
- None exposed.

### Meaningful output
- Published App and public URL — live-artifact fulfilment, categorically different from pull request.
- replId + replUrl (native Replit URL for reviewing progress).
- This satisfies result.observe (item 6) as meaningful provider result (preview/published app), but does not satisfy item 4.

### Reconnect/resume/background behavior
- update_app_using_prompt mutates existing app, continuation via same replId.
- Native Replit URL for reviewing progress.
- No work item to reopen; App persists.
- Async building, polling get_publish_status.

### Known policy/terms constraints
- MCP tool set unversioned; schemas do not carry REST-style version guarantees.
- No public REST endpoint that builds app; internal GraphQL undocumented/unsupported for third-party use. Admin API is Enterprise account-governance surface read-mostly, not build lifecycle (per scalekit analysis).
- Requires OAuth, not API key.
- app_stack required from fixed list — provider-owned taxonomy with no equivalent in other shapes.
- List default 25 max 50.

### First-party evidence URLs and verification date
- https://docs.replit.com/platforms/mcp-server — tools table, URL, transport, auth, setup — fetched 2026-09-13
- https://docs.replit.com/updates/2026/08/14/changelog — Use Replit through MCP native — fetched via search
- https://www.scalekit.com/blog/replit-mcp-vs-api — MCP vs Admin API decision framework — secondary, accessed 2026-09-13
- Verification date: 2026-09-13

### Gate 1 verdict
**PARTIAL — fails PRODUCT.md item 4 and thus Gate 1 per contract §5.5.1**:
1. start/continue where exposed: create_app_from_prompt, update_app_using_prompt, list/search — yes, but App-shaped not task-shaped.
2. interact with Agent: via prompts in create/update + ask_question — partial.
3. observe progress/plan/tool activity where exposed: none exposed in external surface — no.
4. inspect changed files/diffs: none exposed — fails unqualified required item 4. Per contract §5.5.1, fulfilment none does not qualify for MVP loop; live-artifact alone does not satisfy item 4 (needs inline-patch or remote-reference). So Gate 1 fails.
5. respond to approval where exposed: none exposed — required-if-exposed, not disqualifier alone, but combined with 3,4 fails richness.
6. meaningful result: published app URL live-artifact — yes, satisfies item 6 via result.observe.
7. reconnect/continue where supported: update same replId — yes.

Overall not enough supported capability for workspace-level dev loop as defined.

### Gate 2 verdict
**PASS** — Official MCP server, supported external-client path (Streamable HTTP + OAuth 2.1 + PKCE), legitimate entitlement via user's Replit account (Free/Core/Pro/Enterprise), per-user scope.

### Major architectural coupling/risks
- App-centric contract with no work item at all; no progress/approval concept.
- No change inspection surface; would require private interfaces which are prohibited.
- Fixed app_stack taxonomy.
- Unversioned tools.
- No file/diff/activity surface.

---

## 5. OpenAI Codex App Server / hosted Codex — VALIDATION TRACK

### Supported auth path
- Codex App Server auth modes (from primary docs):
  - apikey: caller supplies OpenAI API key.
  - chatgpt: Codex owns ChatGPT OAuth flow, persists tokens, refreshes automatically. Browser flow (account/login/start type chatgpt) or device-code flow.
  - chatgptAuthTokens (experimental): host app supplies externally managed ChatGPT tokens (accessToken, accountId, planType) directly, must refresh when asked.
  - personalAccessToken, agentIdentity, amazonBedrock (Bedrock API key managed by Codex or external AWS chain).
- Protocol: JSON-RPC 2.0 over stdio (JSONL default), websocket experimental, Unix socket. Not REST.
- Client must send initialize request before any other method, with capabilities.experimentalApi opt-in for experimental methods.
- Sign in with ChatGPT supported.
- Third-party client bindings: Go, Python, TypeScript, Swift, Kotlin. TypeScript definitions generable from Rust protocol.
- Codex SDK: TypeScript/Python libraries that run on your infrastructure, control local app-server over JSON-RPC: start/continue/resume threads, drive from CI/CD.

### Entitlement/billing relationship
- User's OpenAI account / ChatGPT subscription. Provider-direct.
- App Server runs on your own infrastructure; you operate host. So Greenfield2 would need to host runtime if using App Server directly.
- No documented hosted REST endpoint that accepts Codex task over HTTP and returns completed run (per harnessrouter.ai analysis Aug 2026: "As of August 2026, OpenAI ships Codex across seven surfaces, including a Codex SDK that runs on your own infrastructure, but its documentation describes no hosted REST API for running Codex tasks").

### Agent/session lifecycle surface
- Core primitives: Thread (conversation between user and Codex agent, contains turns), Turn (single user request + agent work, contains items, streams incremental updates), Item (unit of input/output: user message, agent message, command runs, file change, tool call, etc.).
- Thread APIs: thread/start (model, cwd, approvalPolicy, sandbox, personality, serviceName), thread/resume, thread/read (without resuming, includeTurns), thread/list (pagination, filters: modelProviders, sourceKinds, archived, isPinned, cwd, searchTerm, parentThreadId), thread/turns/list, thread/items/list, thread/loaded/list, thread/name/set, thread/archive, thread/delete, thread/unarchive, thread/compact/start, thread/status/changed notification, thread/shellCommand (user-initiated shell outside sandbox), thread/inject_items.
- Turn APIs: turn/start (add user input, begin generation, streams events), turn/steer (append user input to in-flight turn), turn/interrupt (cancellation), review/start, command/exec (single command under sandbox), process/spawn experimental (outside sandbox), etc.
- Rich lifecycle: archive, delete, unarchive, compact, rollback recent turns, clean background terminals.

### Provider-hosted execution/workspace behavior
- Codex Web architecture demonstrates provider-hosted container execution, server-side long-running state, reconnect.
- App Server default: starts local Code Mode host; can use remote host via --code-mode-host wss:// URL — every thread shares selected host connection.
- By default local execution; remote host still requires you operate host.
- No explicit Firecracker VM like Cursor, but container-based.

### New/continue workflow support
- New: thread/start with model (e.g., gpt-5.6-terra), cwd, approvalPolicy never/onRequest/unlessTrusted, sandbox readOnly/workspaceWrite, etc.
- Continue: thread/resume, turn/start follow-up, steer active turn.
- List threads with pagination and filters.

### Observable activity/messages/tool events
- Rich streaming: turn/* and item/* notifications, item deltas, tool calls, file changes, command execution, approvals, fuzzy file search events, warning events, etc.
- Items: config, skills, AGENTS.md, plugins, MCP server config, subagents, hooks, commands, sessions, etc.
- Process execution via command/exec/outputDelta.
- Excellent observability.

### Files/diffs
- File change approvals, command execution, fs/readFile/writeFile/createDirectory/getMetadata/readDirectory/remove/copy/watch/changed.
- Diffs observable via items.

### Approvals
- Command execution approvals, file change approvals, tool/requestUserInput (1-3 short questions), MCP server elicitation requests, dynamic tool calls, MCP tool-call approvals.
- Groups concurrent network approval prompts by destination (host/protocol/port).

### Meaningful output
- Depends on workspace: files, branches, PRs, etc. Provider-equivalent outcome.
- Result is whatever agent produces in workspace.

### Reconnect/resume/background behavior
- Thread resume, compaction, status tracking, loaded threads list, unsubscribe, no-subscriber inactivity grace period, thread/closed.
- Background terminals cleanable.
- Rollback recent turns.

### Known policy/terms constraints
- App Server implementation open source in openai/codex repo (codex-rs/app-server).
- Experimental API opt-in required for some methods; without opt-in, server rejects with requires experimentalApi capability.
- No hosted REST API for third-party use documented; to call Codex over HTTP today you must wrap self-hosted SDK in service you build and operate, or use harness platform.
- Running App Server in Greenfield2-owned compute violates MVP non-goal that Greenfield2 does not provide its own development runtime (PRODUCT.md explicit non-goals 1-3).
- Hosted Codex Web runtime third-party entitlement/runtime path remains unverified.

### First-party evidence URLs and verification date
- https://developers.openai.com/codex/app-server — protocol, stdio/websocket/Unix socket, initialization, API overview, threads, turns, approvals — fetched via search 2026-09-13
- https://openai.com/index/unlocking-the-codex-harness/ — origin of App Server, JSON-RPC, client integration, MCP vs App Server — fetched 2026-09-13
- https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md — hosted Codex Apps MCP protocol, thread removal, user verification — fetched via search
- https://github.com/openai/codex/tree/main/codex-rs/app-server — source
- https://www.codex-docs.com/en/docs/app-server — App Server interface, deep integration, Code Mode host — fetched via search
- https://learn.chatgpt.com/docs/app-server — embed Codex, core primitives — fetched via search
- https://harnessrouter.ai/guides/codex-api-for-apps — analysis: no hosted REST API as of Aug 2026 — secondary but consistent with primary docs absence — accessed 2026-09-13
- Verification date: 2026-09-13

### Gate 1 verdict
**TECHNICALLY STRONG for local execution, but fails MVP hosting model**:
1. start/continue where exposed: thread/start, resume, list — yes.
2. interact with Agent: turn/start, steer — yes.
3. observe progress/plan/tool activity where exposed: rich streaming items — yes.
4. inspect changed files/diffs: file changes, diffs — yes.
5. respond to approval where exposed: command/file change approvals — yes.
6. meaningful result: workspace result — yes.
7. reconnect/continue where supported: resume, compact — yes.

Technically would pass Gate 1 if self-hosted, but MVP requires provider-hosted execution, not Greenfield2-owned runtime.

### Gate 2 verdict
**UNVERIFIED for third-party use of OpenAI-hosted Codex Web runtime**:
- App Server is self-hosted SDK, runs on your infrastructure. No documented public third-party path for Greenfield2 to provision/attach to user's OpenAI-hosted Codex Web runtime while keeping OpenAI as execution provider.
- Therefore hosted third-party entitlement/runtime path remains unverified for Greenfield2 MVP shape. Running App Server in Greenfield2-owned compute violates PRODUCT.md non-goals 1-3.

### Major architectural coupling/risks
- No public hosted task API; would require Greenfield2 to host runtime — prohibited.
- Experimental APIs, JSON-RPC not REST, integration work to build client binding.
- Auth modes include experimental external tokens.
- Hosted path unverified.

---

## Additional candidates considered

- **OpenAI Codex CLI, VS Code extension, JetBrains, Xcode integrations** — same hosting issue as App Server; no hosted third-party REST API.
- **Any other provider** — No additional candidate found on current evidence that materially passes both gates better than Jules/Cursor finalists. Search for "hosted coding agent API third party" returns same set (Jules, Cursor, Copilot, Replit, Codex). No new entrant with richer external surface and legitimate entitlement documented.

---

## Recommendation for product-owner decision

### Shortlist (two-provider final)

**A — Google Jules — best documented semantic match to Greenfield2's required V1 UI**
- Explicit Session → Activity (planGenerated/planApproved/userMessaged/agentMessaged/progressUpdated/sessionCompleted/sessionFailed) → progress/messages → plan approval → git patch/bash/media artifacts → PR.
- Inline-patch change inspection satisfies PRODUCT.md item 4 unambiguously.
- Supported external-client path: official REST API v1alpha, user-scoped API key, GitHub App source connection.
- Provider-direct billing via Google AI Pro/Ultra, daily/concurrent limits documented.
- Trade-offs: v1alpha may change, API-key onboarding not OAuth, GitHub-only sources, Gmail-only paid plans currently, poll-only delivery, task-oriented not generic workspace.

**B — Cursor Cloud Agents — best documented provider-hosted workspace/runtime experience**
- Durable Agent + Runs, rich SSE tool stream (status/assistant/thinking/tool_call with callId/name/status/args/result/truncated, interaction_update, result, done), follow-ups, cloud VM (Firecracker), artifacts, PRs, resume via Last-Event-ID with retention window, spend limit.
- Remote-reference change inspection (branch/PR) + artifacts; contract says remote-reference satisfies item 4, but exact user-facing diff projection should be proven in small non-production validation spike before ratification.
- Supported external-client path: official v1 public beta, Basic+Bearer, user API keys, service-account keys, sub-tokens.
- Provider-direct billing at model API rates, usage pool + overage, paid plan required.
- Trade-offs: public-beta API, usage-priced may be expensive, no inline diff, no approval mechanism in API, strict repo listing rate limits, internet auto-run security considerations, workspace in Cursor-managed infra.

### Why not others now

- **GitHub Copilot cloud agent**: Excellent legitimacy (user-to-server tokens, explicit enablement query, firewall, security scanning) but task API alone lacks rich transcript/tool/diff/approval observation. Would require combining SDK + task API + PR APIs as one contract, which is false completeness per anti-leakage. Watch for richer task API surface; currently Gate 1 PARTIAL.
- **Replit MCP**: Excellent legitimacy (OAuth 2.1 + PKCE, Streamable HTTP, official server) but external surface exposes only App create/update/publish + public URL, no files/diffs/activity/approvals. Fails PRODUCT.md item 4 (changed files/diffs) which has no "where exposed" qualifier. Per contract §5.5.1, none + live-artifact alone does not satisfy item 4. Watch if Replit adds file/diff/activity tools.
- **OpenAI Codex**: Technically excellent for local execution, but no documented public third-party path to use OpenAI-hosted runtime while keeping OpenAI as execution provider. Self-hosting App Server would violate MVP non-goal (no Greenfield2-owned runtime). Keep on validation/partner track; re-evaluate if OpenAI publishes hosted third-party task API.

### Next step

Product owner to choose between **Jules and Cursor**, or request additional validation spike before choosing:

- For **Cursor**: spike should prove changed-files/diff external projection (branch reference → PR diff rendering, artifact listing) and confirm approval absence is acceptable (required-if-exposed).
- For **Jules**: spike should validate GitHub source connection flow, API-key auth UX, and poll-based activity observation meets mobile-first latency expectations.

No provider is selected by this document. The next Architecture decision should be an explicit product-owner choice between Jules and Cursor, recorded as ADR if choice creates durable coupling, using Product Fit + Integration Legitimacy gates only.

---

## Contract mapping appendix

Per PROVIDER_CAPABILITY_CONTRACT.md §5 table (PRODUCT.md qualifiers precise):

| PRODUCT.md minimum V1 item | Qualifier | Contract capability | Jules | Cursor | Copilot | Replit | Codex (self-hosted) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1. start new / continue existing | where exposed | work.start, work.continue | Session create + sendMessage | Agent create + Run create follow-up | Task start + PR comment new task | create_app + update_app | thread/start + resume |
| 2. interact with Agent | none (required) | agent.interact | sendMessage + agentMessaged | SSE assistant + Run follow-up | PR comments | prompt in create/update + ask_question | turn/start + steer |
| 3. observe progress/plan/tool activity | where exposed | progress.observe | Activities planGenerated/progressUpdated | SSE tool_call + interaction_update | task state only | none | rich items |
| 4. inspect changed files/diffs | none (required) | changes.inspect | inline-patch ChangeSet.unidiffPatch | remote-reference branch | remote-reference PR | none (fails) | inline changes |
| 5. respond to approval | where exposed | approval.discover/respond | requirePlanApproval + approvePlan | none (required-if-exposed) | post-hoc PR review | none | command/file approvals |
| 6. meaningful result | none (required) | result.observe | SessionOutput.pullRequest | git.branches + prUrl + result text | PR artifact | published URL live-artifact | workspace result |
| 7. reconnect/continue where supported | where supported | continuity.reconnect | list/get/sendMessage | durable Agent + new Run + Last-Event-ID resume | list tasks + PR comment | update same replId | resume + compact |

Fulfilment kinds per §5.5.1:
- Jules: inline-patch — satisfies item 4 yes.
- Cursor: remote-reference — satisfies item 4 yes per contract, but needs spike.
- Copilot: remote-reference — satisfies item 4 yes, but other items partial.
- Replit: live-artifact for result, none for changes — fails item 4 per §5.5.1 (live-artifact alone does not satisfy item 4; none is disqualifier for first provider).
- Codex: inline-patch equivalent — satisfies item 4, but hosting fails Gate 2.

---

## Evidence collection method

- Channel preflight per research.md: repo first (PRODUCT.md, ARCHITECTURE.md, contract v1.1, validation record), then gh api (issue #8 body), then web search/fetch for primary provider docs. Sandbox curl to vendor hosts fails TLS (SSL_ERROR_SYSCALL) — reported as unreachable, not retried.
- Sources are primary provider documentation (jules.google, cursor.com, docs.github.com, docs.replit.com, developers.openai.com, openai.com) accessed 2026-09-13.
- No provider API called, no private scraping, no reverse-engineered first-party session endpoints.
- Secondary sources (scalekit, harnessrouter, blog posts) used only to corroborate absence of surface (e.g., Replit Admin API is governance not build, Codex has no hosted REST API) and labelled where used.

---

## Related

- [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) — capability catalogue, three-layer availability, required inputs, error handling
- [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) — four shapes, fifteen revisions, anti-leakage audits
- [../PRODUCT.md](../PRODUCT.md) — minimum V1 capability loop and gates
- [../DOMAIN.md](../DOMAIN.md) — Provider Connection vs Provider Resource Reference, no Greenfield2-owned provider nouns
- [../ARCHITECTURE.md](../ARCHITECTURE.md) — Architecture mandate, open decisions, first provider now open as consequence of ADR-0006 acceptance
- [../ROADMAP.md](../ROADMAP.md) — Architecture stage, provider selection open
- Issue #8 — evaluation purpose, gates, candidates, output requirements, decision rule, out-of-scope
