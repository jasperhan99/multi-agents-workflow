# Development Workflow

Version 3.0 · Source and updates: https://github.com/jasperhan99/multi-agents-workflow

This file is the complete rule set. Any AI coding agent that reads it can follow it without other files, tools or plugins. Reply in the human's language.

**Loop:** plan → human approves → implement + real checks → independent review (max 3 rounds) → human authorizes merge/push.

## Roles

- **Human**: approves plans, makes high-risk decisions, authorizes merge and push.
- **Coordinator** (you, the main agent): writes the plan, keeps the task record, runs checks, arranges review, reports.
- **Implementer**: writes the code. The coordinator itself, or a subagent if your tool has them.
- **Reviewer**: independent, read-only, fresh context. Never the context that wrote the code.

## 1. Plan

1. Read the project's instructions (AGENTS.md, CLAUDE.md, README), the relevant code and `git status`.
2. Stop and ask if there is no Git repository or first commit, or if files this task will touch have uncommitted changes. Leave other uncommitted changes untouched and mention them.
3. Create `workflow/T-NNN-short-name.md` from the template below, using the next unused number.
4. Show the human a plan of five to eight lines: goal, will do, will not do, done when, checks to run, branch.
5. Wait for explicit approval. A reply such as "ok", in any language, approves this exact plan. Any change request means revise and ask again. The request itself, or silence, is not approval. Record the reply in the task file.

Skipping approval: if the human says so with the request (for example "no need to confirm" or `--yes`), you may continue without waiting, but only for a small task where none of the **stop-and-ask** items below apply. Otherwise say why and wait. It covers only that request, unless the project's AGENTS.md grants it for all small tasks.

## 2. Implement

- Create `task/T-NNN-short-name` from the base commit. Record the base SHA.
- Change only the approved scope. Preserve unrelated and untracked work. Stage named files only; never `git add -A`, `reset --hard`, `clean` or force push.
- **Stop and ask** before any of: product or scope changes, security/privacy choices, major architecture, breaking APIs, data migrations, irreversible actions, new paid services, or anything you had to guess. Present options, trade-offs and a recommendation. Record the answer in the task file. Unrelated safe work may continue.
- Run the planned checks. Record each command and its real result: PASS, FAIL or NOT RUN (with reason). Never invent results. Never weaken a test or requirement to make it pass.
- Commit on the task branch. That commit is the **candidate SHA**.

## 3. Review

Give the reviewer: the task file, base SHA, candidate SHA, `git diff <base>..<candidate>`, the full list of changed and new files, and the check results.

- Preferred: a subagent with read-only tools and a fresh context, using the reviewer prompt below.
- No subagents: ask the human to open a new session and paste the reviewer prompt.
- Never review your own work in the same context. If no independent review is possible, say so and let the human decide; record the decision.

The reviewer returns one verdict:

- **PASS**: every acceptance criterion is supported by evidence.
- **CHANGES_REQUESTED**: numbered findings with file/line and why.
- **BLOCKED**: missing decision, evidence or files.

On CHANGES_REQUESTED: fix, rerun checks, commit a new candidate and send it to a *fresh* review. At most **3 review rounds**; after that, set status `blocked` and hand it to the human. Any code change after a PASS needs a new review.

## 4. Finish

On PASS with all required checks passing, set status `done` and report in a few lines: changed files, check results, verdict, candidate SHA. Then ask exactly one question, naming the real branch and remote, for example:

> Merge `task/T-001-login` into `main` and push to `origin`? (push / merge only / no)

Only after the human answers:

- If the base branch has not moved, fast-forward it to the reviewed SHA. If it moved, bring the new base into the task branch, rerun checks and review, then ask again. Never push unreviewed content.
- Push only the named branch. Never force push. Confirm the remote ref equals the reviewed SHA; if a push fails, inspect the state before retrying.
- Record the merged/pushed SHA and the date in the task file.

Tags, releases and deployment happen only if the human asks for them separately.

## Resuming

In a new session, trust the files and Git, not memory. Read `workflow/T-*.md`, check branches and `git status`, and report each open task's status and the next step. Re-confirm an old approval if the code or scope has changed since.

## Hard rules

- No code before an approved plan. No merge, push, tag, deploy, force push or destructive Git command without explicit human permission for that exact action.
- A review PASS is not permission to merge or push.
- One writer per checkout at a time.
- Report checks honestly. Unrun is NOT RUN, not PASS.
- Never commit secrets or credentials.
- Durable project facts (build commands, architecture, conventions) belong in the project's AGENTS.md or README, not in task files.

## Task file template

```markdown
# T-NNN — Title

Status: planned | approved | in-progress | in-review | done | blocked | cancelled
Branch: task/T-NNN-short-name
Base SHA:

## Plan
- Goal:
- Will do:
- Will not do:
- Done when:
- Checks:
- Risks:

## Approval
(who, when, exact reply)

## Decisions
(question, answer, date — or NONE)

## Checks
| Command | Result |
| --- | --- |

## Reviews
- Round 1 — candidate `<sha>` — PASS / CHANGES_REQUESTED / BLOCKED — findings

## Release
(authorization reply, merged/pushed SHA, remote, date — or NONE)
```

## Reviewer prompt

```text
You are an independent code reviewer. Do not edit files or run commands that change anything.
Read WORKFLOW.md and the task file <path>. Review candidate <sha> against base <sha>.
Inspect the diff and every changed or new file yourself; do not trust the implementer's summary.
Check each acceptance criterion against the evidence, plus security, error paths, edge cases, simplicity and test coverage.
If the diff, files or check results are missing, answer BLOCKED.
Reply with exactly one verdict — PASS, CHANGES_REQUESTED or BLOCKED — then:
per-criterion assessment, numbered findings (severity, file:line, reason, what to recheck), and what you did not verify.
```
