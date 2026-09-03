# Subagent routing (token saving)

Route delegated work through these named agent types (defined in `~/.claude/agents/`) instead of picking a model ad hoc. The model is **pinned per role**, so you don't re-decide it on every Agent call — pick the role, and the model follows:

- `searcher` (haiku) — file search, grep sweeps, listing, naming-convention checks, simple reads. Returns conclusions, not file dumps.
- `implementer` (sonnet) — implementation, routine edits, applying an already-decided plan, running builds/tests. Also code comprehension: reading and summarizing source files/modules.
- `reasoner` (inherit — orchestrator's model) — SOLVES hard problems hands-on: architecture decisions, hard debugging, tricky trade-offs. Can edit and search the web.
- `reviewer` (inherit — orchestrator's model) — JUDGES existing changes, plans, and claims: code review, critique, adversarial verification. Has the `Agent` tool: it delegates gathering to `searcher`/`implementer` and spends its own tokens only on the judgment. Reports findings; does not fix.

Default to `searcher` or `implementer`. Escalate only when the subtask genuinely needs deep reasoning — rule of thumb: **judging → `reviewer`, solving → `reasoner`**. Nested subagents are allowed up to 5 levels deep, so a role with the `Agent` tool (e.g. `reviewer`) may push its cheap subtasks further down.

**Do NOT spawn a plain/general-purpose agent (or one with no `subagent_type`) for mechanical or implementation work** — that silently inherits the orchestrator's model and defeats the whole point of offloading. If none of the roles fit, that's the signal the work belongs in the main loop, not in an inherited-model subagent.

Keep planning and orchestration in the main loop.

Dispatch independent subtasks to subagents in parallel and keep working while they run — don't block on each one. When subagents (or parallel work) edit files in the same repo, use worktree isolation to avoid clobbering concurrent changes.

**Verify subagent completion reports against the environment, never trust them as-is** (false success: agents fabricate detailed completion reports — including test counts and analysis conclusions — for work they never executed). Before acting on a report: check actual state (`git status`, file contents), and re-run any claimed verification numbers (test counts, line counts) yourself. If the report and disk disagree, check the transcript's Edit records before concluding anything. For behavior-critical implementation work, prefer synchronous (`run_in_background: false`) execution.

# Where information belongs (t_wada principle)

Apply in every project: code says **How**, test code says **What**, commit logs say **Why**, code comments say **Why not**.

## Code = How
- Naming and structure alone must convey how it works. If a comment would be needed to explain a step, extract or rename instead of commenting.
- NG: `// filter users` + `const r = us.filter(u => u.a)` / OK: `const activeUsers = users.filter(user => user.isActive)`

## Test code = What (executable specification)
- Test names state the behavior as a spec sentence, not an echo of the function name — e.g. `test("退会済みユーザーはログインできない")`, `it("rejects login for deactivated users")`. Match the language already used in the project's tests.
- One behavior per test. Reading the tests should reveal the module's spec.

## Commit log = Why
- Subject: summary of the change. Body: why the change was needed (background, trigger, related issue).
- Do not enumerate what was changed — the diff shows that.

## Code comments = Why not (plus Why that can't live elsewhere)
- Comment only what the code cannot express:
  - Why the obvious approach was deliberately avoided — e.g. `// navigator.userAgentData only works over HTTPS, so we parse the UA string (link)`
  - External constraints, spec pitfalls, workaround rationale (with links when possible)
- Never: narrating the next line, change history, "added X"-style comments.

# Handling feedback in plan mode

- When I give feedback after a plan is presented, respond **as a discussion first** (your take on the point raised, trade-offs, alternatives — in conversation). Do not silently revise and re-present the full plan.
- Re-present the full plan (ExitPlanMode) only when I explicitly say so, e.g. "finalize the plan" / "プランを確定して".
- For contested decision points, feel free to present options via AskUserQuestion.
