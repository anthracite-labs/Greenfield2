# Project Memory

Append-only ledger. Newest entry at the bottom. The sandbox is destroyed
between sessions, so memory that is not committed does not exist.

Procedure: [`../.ecc/skills/project-memory.md`](../.ecc/skills/project-memory.md).

## How to use this file

- Append one entry per working session. Never rewrite or delete an old entry;
  correct it with a new one that says what changed and why.
- Record what was **verified**, with the command and the value
  that came back. Name the function or path the check actually executed.
- A clean exit code is not a pass if the output is wrong. Read the output.
- A skipped check is reported as skipped, never as passed.
- "I could not run this here, because X" is an acceptable outcome. An
  unverified claim presented as done is not.

## Code shape

Applies as soon as application code exists; harmless before that.

- Functions stay focused (< ~50 lines); prefer early returns over nesting
  deeper than 4 levels.
- Files stay cohesive; treat ~800 lines as a soft ceiling that needs a reason.
- Handle errors explicitly. No silent catches, no swallowed failures.
- No debug output left behind (`console.log`, `print`, stray `set -x`).
- Prefer immutable updates and pure functions over in-place mutation.
- Prefer an existing, maintained dependency over hand-rolled code — but only
  after research, and record the choice.

## Scope discipline

- Implement what the issue asks. Nothing adjacent, nothing speculative.
- No new framework, database, auth scheme, hosting target, or UI without an
  approved issue and an ADR in `docs/decisions/`.
- If the smallest correct change is larger than the issue implies, stop and
  say so rather than quietly expanding scope.

## Arena-specific rules

Derived from `docs/ARENA.md` (concise generic reference; re-verify per project):

- Nothing persists between sessions except what is committed and pushed.
- Shell state (variables, `cd`, env) does not survive between tool calls;
  long-lived processes need the process tools, and servers bind `0.0.0.0`.
- Egress is allowlisted: `github.com`, `api.github.com`, `registry.npmjs.org`,
  `pypi.org`, `files.pythonhosted.org`. Assume anything else is blocked and
  say so instead of retrying.
- No browser binaries and no Playwright download path — E2E is out of scope;
  substitute unit/DOM tests plus the live preview.
- There are no harness hooks, slash commands, or subagents. Enforcement is
  `scripts/verify.sh` plus GitHub Actions, never `.git/hooks/`.
