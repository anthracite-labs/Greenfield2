# First Provider Evaluation — Issue #8

**Status:** Draft evaluation, product-owner decision pending. Selects no provider.
**Lifecycle:** `PROJECT_PHASE=architecture`, `ALLOW_APP_STACK=0`. No stack, transport, framework, database, auth implementation, hosting, UI technology, or provider selected.
**Issue:** [#8 — Architecture: evaluate first MVP provider](https://github.com/anthracite-labs/Greenfield2/issues/8)
**Contract:** [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) v1.1 (accepted), [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) four shapes, fifteen revisions.
**Product gates:** [PRODUCT.md](../PRODUCT.md) minimum V1 loop (7 items) and Integration Legitimacy (supported external-client path + legitimate entitlement).
**Verification date:** 2026-09-13 (UTC) — all first-party URLs fetched via platform fetch/search on this date; re-verified 2026-09-13 (UTC) for first corrective pass, re-verified 2026-09-13 (UTC) for second corrective pass (Replit MCP 3-tool surface), re-verified 2026-09-13 (UTC) for third corrective pass (Copilot entitlement 3-way: REST reference Business/Enterprise only vs June 4 changelog Pro/Pro+/Max vs how-to all paid). Sandbox egress to vendor doc hosts fails TLS handshake (`curl https://jules.google/ → SSL_ERROR_SYSCALL`), so no provider API was called. Documentation-level validation only, no live MCP tools/list introspection exercised. **Discrepancies recorded:** 1) Replit: retrieval channel via platform fetch on 2026-09-13 returned 8-tool table (create_app_from_prompt, search_apps, resolve_app_by_name, list_apps, ask_question, update_app_using_prompt, publish_app, get_publish_status) while independent direct inspection of https://docs.replit.com/platforms/mcp-server on same date reports page text “The server exposes three public tools” with Tools nav containing only create_app_from_prompt, update_app_using_prompt, ask_question — freshest directly inspected page (3 tools) authoritative, stale 8-tool extract conflicting evidence not asserted. 2) Copilot: three GitHub-owned first-party sources disagree on eligible plan classes — REST reference https://docs.github.com/en/rest/agent-tasks/agent-tasks?apiVersion=2026-03-10 says Business/Enterprise only for Start task, June 4 changelog https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ says Pro/Pro+/Max REST support added, how-to https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api says all paid plans — all first-party, conflict preserved, provider truth governs.
**Decision rule:** Do not silently choose a winner. This document produces shortlist and recommendation; product owner and Greenfield2 review explicitly decide.

---

## Summary

| Candidate | Gate 1 — Product Fit | Gate 2 — Integration Legitimacy | Current disposition |
| :--- | :--- | :--- | :--- |
| **Google Jules** | **PASS** — all 7 V1 items via REST API: Session lifecycle, Activities (plan/progress/messages), inline-patch ChangeSet, opt-in plan approval, PR result, list/continue/reconnect | **PASS** — official REST API v1alpha, user-generated API key, provider-direct billing via Google AI Pro/Ultra, GitHub source connection via Jules GitHub App | **Finalist — best documented semantic match to V1 UI** |
| **Cursor Cloud Agents** | **PASS, with one diff-surface caveat to validate in spike** — durable Agent + Runs, rich SSE tool stream, branch/PR result, follow-ups, resume via Last-Event-ID; change inspection is remote-reference (branch) not inline-patch; approval none exposed (required-if-exposed) | **PASS** — official Cloud Agents API v1 public beta, Basic + Bearer auth, user API key + service-account keys + sub-tokens, paid Cursor plan required, usage-priced | **Finalist — best provider-hosted workspace/runtime** |
| **GitHub Copilot cloud agent** | **PASS** — start/list/get tasks + issue assignment + @copilot PR-comment continuation, task state queued/in_progress/waiting_for_user/completed, draft PR result, remote-reference diffs via PR APIs per contract §5.5.1, post-hoc PR review approval where exposed, list/get history | **PASS** — official Agent Tasks REST API public preview + official GitHub issue/PR surfaces, user-to-server tokens (PAT, OAuth, GitHub App user token, no installation tokens), **entitlement conflict preserved: REST reference Business/Enterprise only vs June 4 changelog Pro/Pro+/Max REST support vs how-to all paid plans**, provider truth governs | **Finalist — best GitHub-native legitimacy + firewall/security scanning, with entitlement conflict to revalidate** |
| **Replit MCP** | **PARTIAL/FAIL — App-shaped (no work item), 3-tool surface create/update/ask, replUrl result, no files/diffs/patch/remote-reference, no structured progress/activity event stream (but ask_question can return build-status/progress via discussion mode), no approvals** | **PASS** — official MCP server https://replit-mcp.com/server/mcp Streamable HTTP, OAuth protected-resource discovery, OAuth 2.1 + PKCE per auth section, Free/Core/Pro/Enterprise accounts | **Watch / fails PRODUCT.md item 4 (changed files/diffs) — Gate 1 PARTIAL/FAIL** |
| **OpenAI Codex App Server + hosted Codex** | **TECHNICALLY STRONG for local execution** — threads/turns/items, streaming, diffs, approvals, file changes; hosted third-party path not documented | **UNVERIFIED for third-party use of OpenAI-hosted runtime** — App Server runs on your own infra; running it in Greenfield2-owned compute violates MVP non-goal of no Greenfield2 runtime | **Validation/partner track** |

No additional candidate was found that materially passes both gates better than the three finalists (Jules, Cursor, Copilot) on current first-party evidence.

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
- **Maturity:** API v1alpha, experimental, may change specs, keys, definitions. At least one stable + one experimental version planned. Pre-stable — re-verify before implementation. SessionState 9 values, activities 7 types, ChangeSet/patch inline, plan approval opt-in — all documented but subject to drift.
- API key security: do not embed in public code; exposed keys auto-disabled. Max 3 keys, auth via x-goog-api-key header.
- **Account/plan restrictions:** Paid plans only for @gmail.com individual accounts via Google AI Pro/Ultra; enterprise Workspace path not yet GA, interest form. Free 15 daily tasks (rolling 24h) 3 concurrent, Pro 100/15, Ultra 300/60. Task limits not shared in family. Age 18+ stricter than Google One.
- **Auth:** User-generated API key, not OAuth; GitHub Sources must first be connected via Jules web app and GitHub App installation.
- Repository/task oriented, not generic workspace; no always-on VM.
- **Session lifecycle re-checked:** QUEUED → PLANNING → AWAITING_PLAN_APPROVAL (if requirePlanApproval) → IN_PROGRESS → AWAITING_USER_FEEDBACK (continuation via sendMessage) → PAUSED/FAILED/COMPLETED; activities cover planGenerated/planApproved/userMessaged/agentMessaged/progressUpdated/sessionCompleted/sessionFailed; messages via activities; plan generation/approval explicit; ChangeSet GitPatch baseCommitId/unidiffPatch inline-patch; output pullRequest; continuation via list/get/sendMessage; reconnect via polling snapshot.
- **Not promoted merely because review favors it:** Finalist status based on semantic match to V1 UI (inline-patch + plan approval + activity stream), not review preference; still requires spike for GitHub source connection + API-key UX + poll latency.

### First-party evidence URLs and verification date
- https://jules.google/docs/api/reference/ — Quickstart, auth x-goog-api-key header max 3 keys, concepts Source/Session/Activity, alpha experimental may change specs/keys/definitions, at least one stable + one experimental planned — fetched 2026-09-13, re-verified 2026-09-13
- https://jules.google/docs/api/reference/sessions/ — Create/List/Get/Delete/SendMessage/ApprovePlan, SessionState 9 values QUEUED PLANNING AWAITING_PLAN_APPROVAL AWAITING_USER_FEEDBACK IN_PROGRESS PAUSED FAILED COMPLETED STATE_UNSPECIFIED, requirePlanApproval opt-in, automationMode AUTO_CREATE_PR, repoless supported — fetched 2026-09-13, re-verified 2026-09-13
- https://jules.google/docs/api/reference/activities/ — List/Get Activities, Activity types planGenerated (plan id + steps[]), planApproved, userMessaged, agentMessaged, progressUpdated, sessionCompleted, sessionFailed, artifacts ChangeSet GitPatch baseCommitId/unidiffPatch/suggestedCommitMessage, bashOutput, media — fetched 2026-09-13, re-verified 2026-09-13
- https://jules.google/docs/api/reference/types/ — Session, SessionState, AutomationMode, Activity, Artifact, ChangeSet, GitPatch, Source — fetched 2026-09-13, re-verified 2026-09-13
- https://jules.google/docs/usage-limits — Plans Free 15 daily 3 concurrent, Pro 100/15, Ultra 300/60, model access Gemini 2.5 Pro / 3 Pro, paid via Google AI Pro/Ultra, @gmail.com only, family limits not pooled, age 18+ — fetched 2026-09-13, re-verified 2026-09-13
- https://developers.google.com/jules/api — API concepts mirror — fetched 2026-09-13
- Verification date: 2026-09-13 initial, re-verified 2026-09-13 corrective pass — doc-level only, no live API exercised, marked v1alpha pre-stable experimental, maturity low

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
- **Caveat preserved: branch existence != inspectable diffs via Cursor API alone.** Official docs https://cursor.com/docs/cloud-agent/api/endpoints (fetched 2026-09-13, re-verified 2026-09-13) show `Run.git.branches[]` contains `{ repoUrl, branch?, prUrl? }` — a remote-reference to a branch Cursor owns, not diff content. `git` is described as "Per-agent state, not per-run. Every run on the same agent returns the same git snapshot." No `unidiffPatch` or file-content diff field is documented in this API.
- **Supported path for inspection (requires separate GitHub surface):** Use `prUrl` from `git.branches[]` to fetch PR diff via GitHub's own pull-request APIs, or list/download artifacts (agent-scoped, workspace persists, v1 paths relative). This is a second provider surface (GitHub), not Cursor's own diff surface.
- **Do not silently combine Cursor + GitHub APIs for completeness:** Contract §4 prohibits assuming transport above adapter and P6 prohibits approximating absent capability. Claiming Cursor alone provides inline diffs would be false completeness. If Greenfield2 chooses Cursor, it must accept coupling to GitHub PR APIs for diff projection and document validation work: spike must prove branch → PR → diff rendering, artifact listing, and that remote-reference satisfies PRODUCT.md item 4 per contract §5.5.1 (it does) without implying Cursor itself returns diffs.
- Artifacts: agent-scoped files (e.g., artifacts/screenshot.png), path relative to workspace artifacts/ directory. List artifacts, download artifact.
- RepoUrl returned without scheme (github.com/...), different from request which keeps https://.
- Verification: https://cursor.com/docs/cloud-agent/api/endpoints chunks 4-5 show git.branches[] schema and artifact semantics; re-verified 2026-09-13.

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
- https://cursor.com/docs/cloud-agent/api/endpoints — Create/List/Get Agent, Create/List/Get/Stream/Cancel Run, Artifacts, Usage, Models, Repositories, Workers/Pools — fetched 2026-09-13 (chunks 0-5), re-verified 2026-09-13 chunks 4-5 show git.branches[] {repoUrl, branch, prUrl} remote-reference only, no inline diff, per-agent not per-run, retention via X-Cursor-Stream-Retention-Seconds, Last-Event-ID opaque, 410 stream_expired remedy read terminal state
- https://cursor.com/docs/cloud-agent — capabilities overview — fetched 2026-09-13
- https://cursor.com/docs/api — auth Basic + Bearer, rate limits, best practices — fetched 2026-09-13
- https://cursor.com/docs-static/cloud-agents-openapi.yaml — OpenAPI spec — fetched 2026-09-13
- Verification date: 2026-09-13 initial, re-verified 2026-09-13 corrective pass — doc-level only, no live API exercised, marked v1 public beta pre-stable, concurrent v0, maturity medium-low

### Gate 1 verdict
**PASS, with one diff-surface caveat to validate in spike — remote-reference only, not inline-patch, branch existence != inspectable diffs**:
1. start/continue where exposed: create agent, create run follow-up, list agents — yes.
2. interact with Agent: assistant SSE, prompt follow-up — yes.
3. observe progress/plan/tool activity where exposed: SSE tool_call events with args/result, status, thinking, interaction_update — yes, richest of candidates.
4. inspect changed files/diffs: remote-reference via git.branches[] (repoUrl/branch/prUrl) + artifacts, not inline-patch — per contract §5.5.1 remote-reference satisfies PRODUCT.md item 4, but **Cursor API alone does not return diff content**; supported inspection path is via GitHub PR APIs using prUrl (separate surface). Spike must prove projection and coupling; do not combine Cursor+GitHub silently for completeness.
5. respond to approval where exposed: none exposed in this API — required-if-exposed, so not disqualifier.
6. meaningful result: branch/PR + result text + artifacts — yes.
7. reconnect/continue where supported: durable agent, new run on same agent, resumable SSE via Last-Event-ID with X-Cursor-Stream-Retention-Seconds — yes.

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
- **First-party conflict documented — three official GitHub sources disagree on plan eligibility (all fetched 2026-09-13, re-verified 2026-09-13, plus changelog fetched 2026-09-13):**
  - How-to guide https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api states: "Copilot cloud agent is available for all paid Copilot plans." — describes overall feature availability (including UI, CLI, issue assignment paths).
  - REST API reference https://docs.github.com/en/rest/agent-tasks/agent-tasks?apiVersion=2026-03-10 states for Start a task: "This endpoint is only available to users with a Copilot Business or Copilot Enterprise subscription." — restricts the programmatic agent-tasks API path that Greenfield2 would use per that page.
  - GitHub Changelog (first-party release announcement, not secondary) https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ states: "Copilot Pro, Pro+, and Max users can now programmatically start and track Copilot cloud agent tasks with the Agent tasks REST API, available in public preview." — announces Pro/Pro+/Max REST API support added.
  - **Resolution for Greenfield2 per evidence rule (preserve conflict when current first-party evidence cannot be reconciled confidently):** Agent Tasks REST integration path is unquestionably official/supported (public preview, versioned API, PAT/OAuth/GitHub App user-to-server tokens). Business/Enterprise eligibility is explicitly documented by REST reference. GitHub subsequently announced Pro/Pro+/Max REST access via June 4 first-party changelog. Current how-to says all paid plans for overall feature. Current documentation therefore disagrees on exact eligible plan classes. Actual per-account entitlement should be treated as provider truth and revalidated before implementation. Do not silently choose whichever source preserves ranking.
- **Gate 2 derivation per Issue #8:** Issue #8 requires 1) supported external-client/third-party integration surface and 2) legitimate way for user's own account/entitlement to authorize and use capability. No private scraping, etc. The Agent Tasks REST API satisfies 1) officially. For 2), user-to-server tokens with provider-direct billing exist; exact plan class eligibility is provider truth with conflicting first-party docs, but legitimate entitlement path exists. Therefore Gate 2 PASS with conflict preserved, not narrowed to Business/Enterprise only.
- Copilot plans (as of 2026-09-13 primary docs, secondary pricing corroboration):
  - Free: allowance of AI credits, limited.
  - Pro $10/mo: Base 1000 credits + 500 flex = 1500 total.
  - Pro+ $39/mo: Base 3900 + 3100 flex = 7000 total.
  - Max $100/mo: Base 10000 + 10000 flex = 20000 total.
  - Business $19/seat/mo: 1900 credits/seat pooled.
  - Enterprise $39/seat/mo + $21/seat GHE Cloud = ~$60 real floor: 3900 credits/seat pooled.
- Usage measured in GitHub AI Credits (1 credit = $0.01). Chat, agent mode, code review, CLI draw from pool; completions unlimited unmetered on paid plans.
- Enablement: available in all repos stored on GitHub except managed user accounts or explicitly disabled. Discoverable via GraphQL `suggestedActors(capabilities: [CAN_BE_ASSIGNED])` — if enabled, first node login `copilot-swe-agent`.
- Provider-direct billing. Greenfield2 does not resell.

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
- New: POST with prompt, base_ref, model, create_pull_request bool, head_ref for existing branch/PR context via Agent Tasks REST API `POST /agents/repos/{owner}/{repo}/tasks`.
- Continue: via commenting on PR with follow-up instructions, including mentioning "@copilot" in a PR comment per current GitHub docs (continue work on PR by mentioning @copilot + instructions), or creating new task, or PR iteration. waiting_for_user state allows clarification mid-task via PR comments.
- List tasks to continue existing via `GET /agents/repos/{owner}/{repo}/tasks` and `GET /agents/tasks` across repos, plus task history via get.
- Issue assignment path: assign issue to Copilot via GraphQL mutations (createIssue/updateIssue/addAssigneesToAssignable/replaceActorsForAssignable with agentAssignment) + required header `GraphQL-Features: issues_copilot_assignment_api_support,coding_agent_model_selection`, or REST `POST /repos/{owner}/{repo}/issues/{number}/assignees` with `copilot-swe-agent[bot]`.

### Observable activity/messages/tool events
- Task API itself exposes state (queued, in_progress, completed, failed, idle, waiting_for_user, timed_out, cancelled), session_count, artifacts, created_at/updated_at. This is provider-reported progress where exposed — task state telemetry.
- Issue/PR surfaces expose additional progress: PR comments, issue comments, PR review comments, draft PR status. Continuing work via "@copilot" mention in PR comment is documented as supported continuation path.
- Provider-shape validation docs/architecture/PROVIDER_SHAPE_VALIDATION.md already treats Copilot as mixed GitHub task/issue/PR-shaped provider and explicitly records task state + logs, PR-reference change inspection, post-hoc PR review, PR-comment continuity — not a single monolithic endpoint.
- Separate Copilot SDK exposes rich hooks, permissions, steerable sessions, on_permission_request callback (in-flight tool permission) — but SDK is different surface from cloud-agent task API + GitHub issue/PR APIs. Must not combine SDK to manufacture capabilities, but also do not call ordinary supported GitHub issue/PR APIs "false completeness" merely because provider exposes workflow across several official GitHub surfaces, unless accepted contract prohibits that composition. Contract P4 bans transport above adapter, not composition of official GitHub REST/GraphQL/PR surfaces that are all part of same provider's supported external-client path.

### Files/diffs
- PR diff via GitHub's own pull-request APIs — remote-reference fulfilment per PROVIDER_CAPABILITY_CONTRACT.md §5.5.1 which explicitly says remote-reference satisfies item 4 (inspect meaningful changed files and/or diffs).
- Artifacts array: provider github, type pull, data id. So result is PR reference (remote-reference).
- Issue/PR-shaped provider: change inspection via PR diff is provider-native, not fabricated.

### Approvals
- No pre-execution plan gate in task API. Provider's gate is post-hoc human code review on draft PR, with iteration via PR comments — this is GitHub's approval model where exposed.
- SDK has tool permission prompt, but not task API.
- Where exposed qualifier per PRODUCT.md item 5 and contract §5 table: respond to provider approval requests where exposed — if provider's exposed approval is PR review, then PR review satisfies where-exposed. Do not add unstated rule requiring Jules-style pre-execution approval.
- Provider-shape validation explicitly records post-hoc PR review as Copilot's approval model.

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
- Do not combine the separate Copilot SDK with the Agent Tasks / issue / PR surfaces merely to manufacture capabilities the supported cloud-agent workflow does not expose. Ordinary supported GitHub Agent Tasks, issue/GraphQL, and PR APIs are part of the provider's documented task/issue/PR workflow and are not false completeness merely because the workflow spans multiple official GitHub surfaces.

### First-party evidence URLs and verification date
- https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api — states "Copilot cloud agent is available for all paid Copilot plans", starting/listing/checking status, issues API with GraphQL-Features header, auth user-to-server only, continuing work on PR by mentioning "@copilot" in comment + follow-up instructions — fetched 2026-09-13, re-verified 2026-09-13 shows all-paid-plans wording and @copilot continuation
- https://docs.github.com/en/rest/agent-tasks/agent-tasks?apiVersion=2026-03-10 — REST API reference versioned 2026-03-10, public preview subject to change, List tasks / Start a task / Get task / List tasks across repos, states queued/in_progress/completed/failed/idle/waiting_for_user/timed_out/cancelled, fine-grained PAT permission "Agent tasks" read vs read+write, GitHub App installation tokens not supported, **Start a task notes "This endpoint is only available to users with a Copilot Business or Copilot Enterprise subscription"** — fetched 2026-09-13, re-verified 2026-09-13 shows Business/Enterprise restriction, fetched again 2026-09-13 third pass confirms same wording
- https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ — **first-party GitHub-owned release announcement (not secondary)** — states "Copilot Pro, Pro+, and Max users can now programmatically start and track Copilot cloud agent tasks with the Agent tasks REST API, available in public preview." — fetched 2026-09-13 third pass via fetch_page, confirms Pro/Pro+/Max REST support added
- https://github.blog/changelog/2026-05-13-start-copilot-cloud-agent-tasks-via-the-rest-api — Business/Enterprise public preview announcement — fetched via search 2026-09-13
- https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent — overview, benefits, integrations, GitHub Actions powered, firewall, security scanning — fetched via search 2026-09-13
- Verification date: 2026-09-13 initial, re-verified 2026-09-13 first corrective, re-verified 2026-09-13 second corrective, re-verified 2026-09-13 third corrective (Copilot entitlement 3-way) — doc-level only, no live API exercised, marked public preview pre-stable, versioned API, maturity medium. **Genuine first-party 3-way conflict preserved:** how-to says all paid plans (overall feature), REST reference says Business/Enterprise only for Start task, June 4 changelog says Pro/Pro+/Max REST support added. All three are GitHub-owned first-party sources. Resolution: Agent Tasks REST path unquestionably official/supported, Business/Enterprise explicitly documented, Pro/Pro+/Max subsequently announced, documentation disagrees on exact eligible plan classes, actual per-account entitlement is provider truth to revalidate before implementation.

### Gate 1 verdict
**PASS — re-derived strictly from accepted Greenfield2 requirements per task (PRODUCT.md, Issue #8, PROVIDER_CAPABILITY_CONTRACT.md §5, PROVIDER_SHAPE_VALIDATION.md), not from richness preference:**

Per contract §5 table:
1. start new / continue existing where exposed — at least one required where exposed: **yes**. Start via Agent Tasks REST API `POST /agents/repos/{owner}/{repo}/tasks` with prompt/base_ref/model/create_pull_request/head_ref. Continue via PR comments with "@copilot" mention + follow-up instructions per how-to docs, plus new task, plus issue assignment via GraphQL mutations. List tasks via `GET /agents/repos/{owner}/{repo}/tasks` and `GET /agents/tasks`. Documented in https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api and https://docs.github.com/en/rest/agent-tasks/agent-tasks?apiVersion=2026-03-10.
2. interact with provider's Agent — required (no qualifier): **yes**. Interact via PR comments, issue comments, @copilot mention continuation, task creation with prompt. Provider-shape validation explicitly treats Copilot as task/issue/PR-shaped with PR-comment continuity.
3. observe progress/plan/tool activity where exposed — required when exposed: **yes where exposed**. Provider exposes task state queued/in_progress/completed/failed/idle/waiting_for_user/timed_out/cancelled + session_count + artifacts + created_at/updated_at via task API. That is provider-reported progress where exposed. Do not add unstated rule requiring Jules-style typed activity stream or Cursor-style SSE; those are richness differences, not automatically Product Fit failures per task instruction. Where provider exposes only task state, observing task state satisfies where-exposed.
4. inspect meaningful changed files/diffs — required (no qualifier): **yes**. PR diff via GitHub's own pull-request APIs — remote-reference fulfilment. Contract §5.5.1 explicitly says remote-reference satisfies item 4. Provider-shape validation records PR-reference change inspection for Copilot. Do not require inline patches.
5. respond to provider approval requests where exposed — required when exposed: **yes where exposed**. Provider's exposed approval model is post-hoc human code review on draft PR with iteration via PR comments. Contract §5.6 and validation F3: approval is not one concept; provider vocabularies differ; where provider has no pre-execution gate, Greenfield2 must not invent one; where provider's gate is PR review, that is the model. So post-hoc PR review satisfies where-exposed. Do not require pre-execution approvals.
6. meaningful result — required (no qualifier): **yes**. Draft pull request artifact (provider github, type pull, data id), branch created, html_url, optionally auto-create PR via create_pull_request true. Satisfies preview/build/deployment/PR/equivalent per PRODUCT.md item 6.
7. reconnect/continue where supported — required where supported: **yes where supported**. Task history via list/get, continue via PR comments/@copilot mention/new task, issue assignment continuity, provider-hosted GitHub Actions execution with background work.

All 7 accepted V1 items satisfied when evaluated against accepted contract qualifiers (where exposed / where supported) and fulfilment kinds (remote-reference satisfies item 4). Previous PARTIAL verdict was based on task-API-alone lacking rich transcript and on treating ordinary supported GitHub issue/PR APIs as false completeness. Task instruction clarifies: do not call ordinary supported GitHub issue/PR APIs false completeness merely because provider exposes workflow across several official GitHub surfaces, unless accepted contract prohibits that composition — it does not (P4 bans transport above adapter, not composition of official GitHub REST/GraphQL/PR surfaces). SDK remains separate surface and is not combined.

Therefore Gate 1 PASS. Richness vs Jules/Cursor (typed activity stream, SSE tool_call, inline-patch) remains as trade-off for recommendation, not as gate.

### Gate 2 verdict
**PASS — with first-party entitlement conflict preserved, not narrowed:**

Official Agent Tasks REST API public preview is unquestionably supported external-client path: versioned API 2026-03-10, user-to-server tokens only (PAT classic/fine-grained, OAuth app token, GitHub App user-to-server token, no installation tokens), fine-grained permission "Agent tasks" read vs read+write, per-repo enablement queryable via GraphQL `suggestedActors(capabilities: [CAN_BE_ASSIGNED])` with login `copilot-swe-agent`, firewall enabled by default, security scanning (CodeQL, secret scanning, dependency analysis), execution via GitHub Actions.

Legitimate entitlement: user contracts directly with GitHub, provider-direct billing via GitHub AI Credits (Pro $10/mo 1500 credits, Pro+ $39/mo 7000, Max $100/mo 20000, Business $19/seat 1900 pooled, Enterprise $39/seat + $21 GHE Cloud ~$60 floor 3900 pooled per docs), provider truth governs.

**Entitlement conflict preserved per evidence rule:**
- REST reference https://docs.github.com/en/rest/agent-tasks/agent-tasks?apiVersion=2026-03-10 says Start task "only available to users with a Copilot Business or Copilot Enterprise subscription" — explicitly documents Business/Enterprise eligibility.
- First-party GitHub changelog https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ says "Copilot Pro, Pro+, and Max users can now programmatically start and track Copilot cloud agent tasks with the Agent tasks REST API" — announces Pro/Pro+/Max REST support added.
- How-to https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api says "Copilot cloud agent is available for all paid Copilot plans" — overall feature availability.

Current first-party GitHub evidence is internally inconsistent on exact eligible plan classes. Do not silently choose whichever preserves ranking. Evaluation preserves conflict, records all three sources with verification date 2026-09-13, and treats actual per-account entitlement as provider truth to revalidate before implementation.

Derive Gate 2 from Issue #8 actual requirement (supported external-client surface + legitimate entitlement path), not from desire to narrow or widen candidate. Both requirements satisfied: official REST API exists, legitimate user-to-server token path exists, provider-direct billing exists. Gate 2 PASS with conflict documented.

### Major architectural coupling/risks
- Public preview may change, versioned API, docs internally inconsistent on plan eligibility (Business/Enterprise vs Pro/Pro+/Max vs all paid).
- Richness less than Jules/Cursor: task state only (queued/in_progress/waiting_for_user etc.) vs typed activity stream / SSE tool_call; remote-reference PR vs inline-patch; post-hoc PR review vs pre-execution gate — richness is trade-off, not gate, but affects mobile UX.
- Do not combine Copilot SDK (separate surface with rich hooks) to manufacture capabilities — SDK remains separate. However, ordinary GitHub task/issue/PR/GraphQL surfaces (Agent Tasks REST + issue assignment + PR APIs + @copilot continuation) are all official GitHub-supported and part of same provider's workflow per PROVIDER_SHAPE_VALIDATION.md — not false completeness per task instruction, unless contract prohibits composition (it does not).
- No installation token support yet — human token required, complicates headless automation.
- Repo must be on GitHub, excludes other hosts.
- Model selection cost lever but list evolves.
- Entitlement conflict requires revalidation per account before implementation — provider truth governs.

---

## 4. Replit MCP — GATE 1 PARTIAL/FAIL / GATE 2 PASS (3-tool surface)

### Supported auth path
- **First-party source inspected:** https://docs.replit.com/platforms/mcp-server — fetched 2026-09-13, re-fetched 2026-09-13 for second corrective pass. Current page states: “The server exposes three public tools.” Tools navigation contains only three entries.
- **Exact public MCP tools now documented (3):**
  - `create_app_from_prompt`
  - `update_app_using_prompt`
  - `ask_question`
- **Removed previous eight-tool claims:** `publish_app`, `get_publish_status`, `list_apps`, `search_apps`, `resolve_app_by_name` are no longer documented as part of same supported public MCP server surface per current first-party page. They were present in stale retrieval channel extract that returned 8-tool table on 2026-09-13 earlier fetch; that stale extract is now recorded as conflicting evidence and not asserted. No current first-party Replit source clearly documents those five operations as part of same public MCP server surface — do not infer from Replit native UI, historical MCP docs, Admin API, product capabilities, or secondary connector catalogs.
- Official Replit MCP Server, remote server URL `https://replit-mcp.com/server/mcp`.
- Transport: Streamable HTTP (MCP).
- Auth: OAuth using protected-resource discovery. Authentication section explicitly mentions OAuth 2.1 with PKCE. Client reads Replit's protected-resource metadata, prompts sign-in to Replit. Do not create custom OAuth server; Replit provides OAuth metadata.
- Supports ChatGPT, Claude, Slack native integrations plus any MCP client supporting Streamable HTTP + OAuth.
- Access scoped to apps user can edit, including shared — per page statement “You can create apps or work with apps that you can edit, including apps shared with you.”
- Entitlement: Replit Free/Core/Pro/Enterprise accounts per docs.

### Entitlement/billing relationship
- User's Replit account: Free, Core, Pro, Enterprise. User contracts and pays Replit directly.
- Greenfield2 does not resell.
- Billing relationship provider-direct.
- No entitlement endpoint; L2 unknown until attempt per contract.

### Agent/session lifecycle surface
- Primary unit: **App** — long-lived provider-owned resource, not a task/session/run. **Product vs MCP distinction preserved:** Replit product has Apps, Repls, Deployments, but MCP surface exposes only App-shaped operations; we do not infer additional lifecycle from Replit UI, Admin API, or historical docs.
- **Current 3-tool surface per first-party page:**
  - `create_app_from_prompt`: required `appDescription`, `app_stack` (choice list `react_website`, `mobile_app`, `design`, `slides`, `animation`, `data_visualization`, `3d_game`, `document`, `spreadsheet`), optional `userSpecifiedAppName`, `userQuotes`, `attachmentSummary`, `sourceReplId` (private copy). Response includes at least `phase`, `replId`, `turnId`, `replUrl` per current docs. Replit Agent then builds asynchronously.
  - `update_app_using_prompt`: required `replId`, `changeDescription`, optional `userQuotes`, `attachmentSummary`. Used for continuing iteration against same app.
  - `ask_question`: required `replId`, `question`. Discussion mode, does not modify app. Current docs say can be used to check build status, ask about tech stack, relay questions, report build progress.
- No state enum exposed via MCP (no SessionState-like vocabulary). Work is asynchronous. Previous claim “A create, update, or publish request is still running — Wait before retrying” was from 8-tool page troubleshooting section; with 3-tool surface, async nature still documented via `phase` + `replUrl` tracking, but `get_publish_status` polling no longer documented — do not claim it.
- **Discrepancy note:** Earlier fetch channel returned 8-tool table including `publish_app`/`get_publish_status`/`list_apps`/`search_apps`/`resolve_app_by_name`; current directly inspected page shows 3 tools. Freshest directly inspected first-party page (3 tools) is authoritative; stale 8-tool extract recorded as conflicting evidence.

### Provider-hosted execution/workspace behavior
- Replit Agent builds app in Replit cloud-hosted project (code, data, assets), secure isolated environment, auto-save, version control, collaboration, publishing to cloud.
- Project Editor tools: AI-powered, collaboration, publishing.
- Zero-setup, pre-configured environments.
- Execution is task-oriented App build, not generic always-on workspace shell with terminal/browser VM like Cursor.

### New/continue workflow support
- New: `create_app_from_prompt` with `appDescription` + `app_stack` required. Returns `phase`, `replId`, `turnId`, `replUrl`. `phase` indicates build stage, `replUrl` is native Replit URL to track progress.
- Continue: `update_app_using_prompt` using same `replId`. Documentation explicitly describes continuing iteration against same app and handing off between MCP clients using same `replId` — cross-client continuation supported.
- No work item to reopen; App is long-lived. Do not invent run/session abstraction — App is the unit.

### Observable activity/messages/tool events
- **No structured progress/activity event stream is exposed** via supported MCP surface (no Activity stream like Jules, no SSE tool_call stream like Cursor).
- **But supported Agent interaction can return build-status/progress information via `ask_question`:** Current documentation says `ask_question` can be used to check build status, ask about tech stack, relay questions, report build progress. This is discussion mode returning provider text, not typed events. Distinguish: no structured telemetry, but build-status/progress obtainable via discussion.
- No tool execution observability like `tool_call` with args/result.

### Files/diffs
- **None exposed.** Current 3-tool documentation still appears to expose no changed-file, diff, patch, or equivalent remote change-inspection surface (no file list, no diff content, no branch reference, no PR reference).
- Published app is observable output via `replUrl`, not diff. No file/diff/activity surface.
- Contract mapping: `changes.inspect` → `none`. Per PROVIDER_CAPABILITY_CONTRACT.md §5.5.1, `none` is disqualifier for first provider, `live-artifact` alone does not satisfy item 4. PRODUCT.md minimum V1 item 4 says “inspect meaningful provider-exposed changed files and/or diffs” with no where-exposed qualifier, so fails.
- Re-evaluated against accepted contract §5.5.1: still fails.

### Approvals
- None exposed in current 3-tool MCP surface.

### Meaningful output
- **Precise accounting from current first-party page:** `create_app_from_prompt` returns `replUrl` (native Replit URL) and `replId` + `phase` + `turnId`. Docs say Replit Agent builds asynchronously, returns `replUrl`, directs builder to that URL to track progress. Product description level says Replit Agent transforms prompts into live/published apps.
- **Do not claim dedicated public `publish_app` / `get_publish_status` MCP capability** if those tools are no longer documented — they are removed from claims.
- For V1 item 6 (meaningful provider output/result such as preview, build, deployment, PR, or equivalent): `replUrl` qualifies as live-artifact result (published app URL / preview URL) obtainable from create response and trackable via that URL. This satisfies item 6 via `result.observe` as live-artifact, but uncertainty remains: current page does not explicitly state whether `replUrl` is already live/published or requires separate publish step via UI; docs describe Agent building asynchronously and transforming into live/published apps at product level, but no `publish_app` tool documented. State uncertainty explicitly: we credit live-artifact via `replUrl`, but cannot assert dedicated publish/status MCP tools exist.
- `replId` + `replUrl` native Replit URL for reviewing progress.

### Reconnect/resume/background behavior
- `update_app_using_prompt` mutates existing app, continuation via same `replId`.
- Documented same-`replId` continuation and cross-client handoff behavior per current docs (hand off between MCP clients using same `replId`).
- Native `replUrl` for reviewing progress / tracking build.
- No work item to reopen; App persists. Background building in provider, observable via `replUrl` + `ask_question` for status, not via `get_publish_status` polling (removed).

### Known policy/terms constraints
- **Maturity:** MCP tool set unversioned; schemas do not carry REST-style version guarantees. Pre-stable — re-verify before implementation. Current surface reduced from 8 to 3 tools, indicating drift.
- No public REST endpoint that builds app; internal GraphQL undocumented/unsupported for third-party use. Admin API is Enterprise account-governance surface read-mostly, not build lifecycle — do not infer MCP tools from Admin API.
- Requires OAuth (OAuth 2.1 + PKCE), not API key.
- `app_stack` required from fixed provider list — provider-owned taxonomy with no equivalent in other shapes.
- No file/diff/activity/approval surface.
- **Discrepancy recorded:** Earlier platform fetch returned 8-tool table; current direct inspection shows 3 tools with statement “The server exposes three public tools.” Freshest page treated as authoritative.

### First-party evidence URLs and verification date
- https://docs.replit.com/platforms/mcp-server — **current official page directly inspected 2026-09-13 second corrective pass** — states “The server exposes three public tools”, Tools nav only three tools, lists `create_app_from_prompt` (required appDescription, app_stack choice list react_website/mobile_app/design/slides/animation/data_visualization/3d_game/document/spreadsheet, optional userSpecifiedAppName/userQuotes/attachmentSummary/sourceReplId, response includes phase/replId/turnId/replUrl, Agent builds async), `update_app_using_prompt` (replId/changeDescription, same replId continuation and cross-client handoff), `ask_question` (replId/question, discussion mode, can check build status/tech stack/relay questions/report build progress, no structured progress stream). Also documents URL https://replit-mcp.com/server/mcp, Transport Streamable HTTP, Auth OAuth using protected-resource discovery, OAuth 2.1 with PKCE in auth section, Free/Core/Pro/Enterprise accounts, replUrl tracking, async build. No publish_app/get_publish_status/list_apps/search_apps/resolve_app_by_name in current page.
- https://docs.replit.com/platforms/mcp-server — **stale retrieval channel extract 2026-09-13 earlier fetch** returned 8-tool table (create_app_from_prompt, search_apps, resolve_app_by_name, list_apps, ask_question, update_app_using_prompt, publish_app, get_publish_status) — recorded as conflicting evidence, not asserted as current, prefer freshest page.
- https://docs.replit.com/updates/2026/08/14/changelog — Use Replit through MCP native — fetched via search 2026-09-13 (secondary corroboration of MCP existence, not tool list)
- https://www.scalekit.com/blog/replit-mcp-vs-api — MCP vs Admin API decision framework, previously stated 8 tools (secondary, stale relative to current first-party page) — accessed 2026-09-13, now considered stale secondary vs current first-party 3-tool page.
- https://www.promptarmor.com/connectors/replit — shows 4 tools for Claude connector (secondary filtered view, not authoritative) — accessed 2026-09-13
- Verification date: 2026-09-13 initial, re-verified 2026-09-13 first corrective (8-tool per stale fetch), re-verified 2026-09-13 second corrective (3-tool per freshest directly inspected first-party page) — doc-level only, no live tools/list exercised, marked unversioned/pre-stable, discrepancy explicitly recorded.

### Gate 1 verdict
**PARTIAL/FAIL — fails PRODUCT.md item 4 and thus Gate 1 per contract §5.5.1, with revised analysis from 3-tool surface**:
1. start new work: `create_app_from_prompt` returns phase/replId/turnId/replUrl, Agent builds async — yes, but App-shaped not task-shaped.
2. interact with Agent: via prompts in create/update + `ask_question` discussion mode — yes, partial, discussion does not modify app, can relay build-status/progress.
3. observe progress/plan/tool activity where exposed: **no structured progress/activity event stream** exposed; but **build-status/progress information obtainable via `ask_question`** discussion mode per docs — partial, not typed stream like Jules/Cursor. Distinguish no structured stream vs discussion-mode status.
4. inspect changed files/diffs: **none exposed** — no file/diff/patch/remote-reference surface in current 3-tool docs — fails unqualified required item 4. Per contract §5.5.1, fulfilment `none` does not qualify for MVP loop; live-artifact alone does not satisfy item 4. So Gate 1 fails.
5. respond to approval where exposed: none exposed — required-if-exposed, not disqualifier alone, but combined with 3,4 fails richness.
6. meaningful result: `replUrl` live-artifact (published app URL / native URL to track progress) — yes, satisfies item 6 via result.observe, but uncertainty about publish vs live state since no publish_app tool documented; credit live-artifact via replUrl with uncertainty stated.
7. reconnect/continue where supported: same `replId` via `update_app_using_prompt`, documented cross-client handoff — yes.

Overall not enough supported capability for workspace-level dev loop as defined — fails item 4, no structured progress stream, no file/diff surface.

### Gate 2 verdict
**PASS** — Official MCP server https://replit-mcp.com/server/mcp, supported external-client path Streamable HTTP + OAuth using protected-resource discovery + OAuth 2.1 with PKCE, legitimate entitlement via user's Replit account Free/Core/Pro/Enterprise, per-user scope. Integration path legitimacy preserved even with reduced 3-tool surface.

### Major architectural coupling/risks
- App-centric contract with no work item at all; no structured progress/approval/file/diff concept in current 3-tool surface — only discussion-mode status via ask_question.
- No change inspection surface; would require private interfaces which are prohibited — fails PRODUCT.md item 4.
- Fixed `app_stack` taxonomy provider-owned.
- Unversioned tools, surface drift from 8 to 3 tools observed 2026-09-13 — pre-stable, re-verify.
- No discovery/listing/publish/status tools in current surface — continuity only via same replId, not via list/search/resolve.

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
- **Any other provider** — No additional candidate found on current evidence that materially passes both gates better than Jules/Cursor/Copilot finalists. Search for "hosted coding agent API third party" returns same set (Jules, Cursor, Copilot, Replit, Codex). No new entrant with richer external surface and legitimate entitlement documented beyond the three finalists.

---

## Recommendation for product-owner decision

### Shortlist (three-provider final — re-derived after Gate verdicts corrected)

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

**C — GitHub Copilot cloud agent — best GitHub-native legitimacy, now Gate 1 PASS after re-derivation**
- Task/issue/PR-shaped provider per PROVIDER_SHAPE_VALIDATION.md: Agent Tasks REST API `POST /agents/repos/{owner}/{repo}/tasks` + list/get across repos, issue assignment via GraphQL mutations + `GraphQL-Features` header, PR-comment continuation via "@copilot" mention + follow-up instructions, task states queued/in_progress/waiting_for_user/completed etc., draft PR result, remote-reference diffs via PR APIs (satisfies item 4 per §5.5.1), post-hoc PR review as approval where exposed, task history/list/get continuity.
- Supported external-client path: official Agent Tasks REST API public preview (versioned 2026-03-10) + official GitHub issue/PR/GraphQL surfaces; user-to-server tokens (PAT, OAuth, GitHub App user-to-server, no installation tokens); per-repo enablement via `suggestedActors`; firewall + security scanning; GitHub Actions execution.
- Entitlement conflict preserved: REST reference says Business/Enterprise only for Start task, June 4 first-party changelog https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ says Pro/Pro+/Max REST support added, how-to says all paid plans — all three GitHub-owned first-party sources, documentation disagrees on exact eligible plan classes, provider truth governs, revalidate before implementation.
- Trade-offs: less rich progress/activity than Jules/Cursor (task state + session_count vs typed activity stream / SSE tool_call), no inline-patch (remote-reference PR), approval is post-hoc PR review not pre-execution gate, entitlement docs internally inconsistent, public preview may change, human token required (no installation tokens yet). Richness is trade-off for recommendation, not gate.

### Why not others now

- **Replit MCP**: Excellent legitimacy (OAuth 2.1 + PKCE per auth section explicitly OAuth 2.1 with PKCE, Streamable HTTP, official server https://replit-mcp.com/server/mcp) but **current first-party page https://docs.replit.com/platforms/mcp-server explicitly states “The server exposes three public tools” (create_app_from_prompt returns phase/replId/turnId/replUrl async, update_app_using_prompt same replId continuation + cross-client handoff, ask_question discussion mode can check build status/tech stack/relay questions/report progress)** — no files/diffs/patch/remote-reference, no structured progress/activity event stream (only discussion-mode status), no approvals, no publish_app/get_publish_status/list_apps/search_apps/resolve_app_by_name (previous 8-tool claims removed as stale, discrepancy recorded). Fails PRODUCT.md item 4 (changed files/diffs) which has no “where exposed” qualifier. Per contract §5.5.1, none + live-artifact replUrl alone does not satisfy item 4. Watch if Replit adds file/diff/activity tools.
- **OpenAI Codex**: Technically excellent for local execution, but no documented public third-party path to use OpenAI-hosted runtime while keeping OpenAI as execution provider. Self-hosting App Server would violate MVP non-goal (no Greenfield2-owned runtime). Keep on validation/partner track; re-evaluate if OpenAI publishes hosted third-party task API.

### Next step

Product owner to choose between **Jules, Cursor, and Copilot** (three-provider shortlist after Gate corrections), or request additional validation spike before choosing:

- For **Cursor**: spike should prove changed-files/diff external projection (branch reference → PR diff rendering, artifact listing) and confirm approval absence is acceptable (required-if-exposed).
- For **Jules**: spike should validate GitHub source connection flow, API-key auth UX, and poll-based activity observation meets mobile-first latency expectations.
- For **Copilot**: spike should validate per-account entitlement for Agent Tasks REST API (Business/Enterprise vs Pro/Pro+/Max conflict), @copilot PR-comment continuation UX, and task state + PR diff projection meets mobile-first expectations. Confirm that task/issue/PR composition is acceptable (not false completeness per contract, as all surfaces are official GitHub).

No provider is selected by this document. The next Architecture decision should be an explicit product-owner choice between Jules, Cursor, and Copilot, recorded as ADR if choice creates durable coupling, using Product Fit + Integration Legitimacy gates only. Richness, mobile UX, maturity, coupling, account restrictions, auth UX, polling/streaming behavior and implementation risk may inform recommendation after gates, not rewrite gates.

---

## Contract mapping appendix

Per PROVIDER_CAPABILITY_CONTRACT.md §5 table (PRODUCT.md qualifiers precise):

| PRODUCT.md minimum V1 item | Qualifier | Contract capability | Jules | Cursor | Copilot | Replit | Codex (self-hosted) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1. start new / continue existing | where exposed | work.start, work.continue | Session create + sendMessage | Agent create + Run create follow-up | Task start via REST + issue assignment via GraphQL + PR comment via @copilot mention | create_app (returns phase/replId/turnId/replUrl) + update_app same replId | thread/start + resume |
| 2. interact with Agent | none (required) | agent.interact | sendMessage + agentMessaged | SSE assistant + Run follow-up | PR comments + @copilot mention + issue comments (task/issue/PR-shaped) | prompt in create/update + ask_question (discussion mode, can check build status/tech stack/relay questions/report progress) | turn/start + steer |
| 3. observe progress/plan/tool activity | where exposed | progress.observe | Activities planGenerated/progressUpdated | SSE tool_call + interaction_update | task state queued/in_progress/waiting_for_user/completed + session_count + artifacts (where exposed) + PR/issue comment progress | no structured stream, but build-status via ask_question discussion mode | rich items |
| 4. inspect changed files/diffs | none (required) | changes.inspect | inline-patch ChangeSet.unidiffPatch | remote-reference branch (branch != diffs) | remote-reference PR (satisfies per §5.5.1) | none (fails) — no file/diff/patch/remote-reference in current 3-tool surface | inline changes |
| 5. respond to approval | where exposed | approval.discover/respond | requirePlanApproval + approvePlan | none (required-if-exposed) | post-hoc PR review (provider's approval model where exposed) | none (required-if-exposed) | command/file approvals |
| 6. meaningful result | none (required) | result.observe | SessionOutput.pullRequest | git.branches + prUrl + result text | PR artifact (draft PR + branch + html_url) | replUrl live-artifact (uncertainty about publish vs live, no publish_app tool) | workspace result |
| 7. reconnect/continue where supported | where supported | continuity.reconnect | list/get/sendMessage | durable Agent + new Run + Last-Event-ID resume | list tasks + get + PR comment @copilot + new task + issue assignment (where supported) | same replId via update_app, cross-client handoff documented | resume + compact |

Fulfilment kinds per §5.5.1:
- Jules: inline-patch — satisfies item 4 yes.
- Cursor: remote-reference — satisfies item 4 yes per contract, but needs spike, branch != diffs.
- Copilot: remote-reference — satisfies item 4 yes per contract §5.5.1 (PR diff via GitHub PR APIs), all 7 V1 items satisfied per re-derived Gate1 PASS.
- Replit: **current 3-tool surface** — live-artifact via replUrl for result (with uncertainty, no publish_app/get_publish_status), none for changes — fails item 4 per §5.5.1 (live-artifact alone does not satisfy item 4; none is disqualifier for first provider). Previous 8-tool claims (publish_app/get_publish_status/list_apps/search_apps/resolve_app_by_name) removed as no longer documented in freshest first-party page https://docs.replit.com/platforms/mcp-server stating “The server exposes three public tools.”
- Codex: inline-patch equivalent — satisfies item 4, but hosting fails Gate 2.

---

## Evidence collection method

- Channel preflight per research.md: repo first (PRODUCT.md, ARCHITECTURE.md, contract v1.1, validation record), then gh api (issue #8 body), then web search/fetch for primary provider docs. Sandbox curl to vendor hosts fails TLS (SSL_ERROR_SYSCALL) — reported as unreachable, not retried.
- Sources are primary provider documentation (jules.google, cursor.com, docs.github.com, docs.replit.com, developers.openai.com, openai.com, github.blog changelog as first-party GitHub-owned release announcement) accessed 2026-09-13, re-verified 2026-09-13 for first corrective pass, re-verified 2026-09-13 for second corrective pass (Replit 3-tool surface https://docs.replit.com/platforms/mcp-server stating “The server exposes three public tools” — create_app_from_prompt/update_app_using_prompt/ask_question, with stale 8-tool extract recorded as conflicting evidence), re-verified 2026-09-13 for third corrective pass (Copilot entitlement 3-way: REST reference https://docs.github.com/en/rest/agent-tasks/agent-tasks?apiVersion=2026-03-10 Business/Enterprise only vs June 4 first-party changelog https://github.blog/changelog/2026-06-04-agent-tasks-rest-api-now-available-for-copilot-pro-pro-and-max/ Pro/Pro+/Max REST support added vs how-to https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-cloud-agent-via-the-api all paid plans; Cursor git.branches[] remote-reference https://cursor.com/docs/cloud-agent/api/endpoints; Jules lifecycle/activities/types/usage-limits https://jules.google/docs/api/reference/ etc.).
- No provider API called, no private scraping, no reverse-engineered first-party session endpoints. No live MCP tools/list introspection — doc-level validation only, marked pre-stable alpha/beta/preview/unversioned where applicable. Ambiguity recorded where first-party sources conflict (Copilot entitlement 3-way: all-paid vs Business/Enterprise vs Pro/Pro+/Max per REST reference vs June 4 changelog vs how-to, Replit stale 8-tool vs current 3-tool).
- Secondary sources (scalekit, harnessrouter, promptarmor, blog posts) used only to corroborate absence or filtered views and labelled where used, separated from first-party, not used to infer MCP surface. github.blog changelog is first-party GitHub-owned, not secondary.

---

## Related

- [PROVIDER_CAPABILITY_CONTRACT.md](PROVIDER_CAPABILITY_CONTRACT.md) — capability catalogue, three-layer availability, required inputs, error handling
- [PROVIDER_SHAPE_VALIDATION.md](PROVIDER_SHAPE_VALIDATION.md) — four shapes, fifteen revisions, anti-leakage audits
- [../PRODUCT.md](../PRODUCT.md) — minimum V1 capability loop and gates
- [../DOMAIN.md](../DOMAIN.md) — Provider Connection vs Provider Resource Reference, no Greenfield2-owned provider nouns
- [../ARCHITECTURE.md](../ARCHITECTURE.md) — Architecture mandate, open decisions, first provider now open as consequence of ADR-0006 acceptance
- [../ROADMAP.md](../ROADMAP.md) — Architecture stage, provider selection open
- Issue #8 — evaluation purpose, gates, candidates, output requirements, decision rule, out-of-scope
