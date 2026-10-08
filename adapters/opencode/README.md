# Adapter: OpenCode

Install: `./scripts/init opencode` — copies `.opencode/` (commands, agents, opencode.json) to the
repo root. No pointer file is needed: OpenCode auto-loads **AGENTS.md** natively, so the single
source of truth is already its context file.

What it adds on top of the core:

- **Segment commands** (`.opencode/commands/`): `/analyze`, `/plan`, `/build`, `/review`,
  `/verify` — one per segment of `workflows/segments.md`. Each checks its entry condition (spec
  `Status: Approved`, plan `Approved by / on`, build evidence, triage), does its segment, and
  **always stops** with a handoff line naming the next command. Handoffs travel through files.
- **Chainer**: `/new-feature` reads the Mode from `AGENTS.md`. In `lite` it runs the segments in
  order and asks at each gate (spec, plan, triage, ship); in `strict` it refuses and redirects to
  the segment commands, one role per session. The rule:
  *Segments always stop; /new-feature flows only as far as the mode allows.*
- **Change requests**: `/change` — runs the triage rubric of `workflows/change-request.md`
  (bug / trivial / change) with a reasoned verdict, then the lane the behavior demands; in strict
  mode it stops after the mini-spec, in lite it chains with gate approvals.
- **Other commands**: `/bootstrap`, `/fix-bug`, `/refactor`, `/adr`, `/recover` — each loads the
  matching workflow and honors the operating mode. All commands are thin by design: they point to
  `workflows/` and `prompts/`, they don't restate them.
- **Read-only reviewer subagent** (`.opencode/agents/reviewer.md`): the "producer never verifies
  its own work" rule at the tool level. `edit` is denied outright; `shell` is narrowed to
  `./scripts/check` plus read-only git inspection (`git diff`, `git log`, `git status`). Write
  flags (`git diff --output`), web tools and the subagent tool are denied too — the reviewer
  cannot fetch, spawn, or write. Projects that add MCP tools should deny their `server_*`
  actions here as well. This is *Enforced* by permissions, not by instruction — narrower than
  the Claude Code reviewer, which keeps full Bash with an instruction never to write.
- **Role agents** (`.opencode/agents/`): `analyst`, `developer` and `qa`, alongside the read-only
  `reviewer`. The single-role segment commands pin their agent (`/analyze` → analyst, `/plan` and
  `/build` → developer, `/verify` → qa), so the path-level "MAY NOT" clauses are enforced by
  permissions: the Analyst cannot edit outside `specs/active/`, QA cannot edit outside `tests/`
  (adjust the path for your stack). Content-level clauses (no weakened tests, no softened
  criteria) stay Documented — no path rule can express them. The chainers (`/new-feature`,
  `/change`) intentionally run unpinned — they cross roles and must not inherit one role's limits.
- **Ask profile** (`.opencode/opencode.json`): any shell command outside a narrow allowlist
  (read-only git, `./scripts/check`, `./scripts/doctor`) asks the human first; approvals can be
  saved per project. Too loud for solo lite work? Delete the `ask` rule and the allowlist —
  the denies above stay.
- **Permission denies** (`.opencode/opencode.json`): `git push --force` (including `-f` and
  non-leading spellings such as `git push origin main --force`), `git reset --hard`,
  `git rebase`, and `rm -rf` (plus `-fr` / `-r -f` / `/bin/rm` variants) are blocked by the
  tool, not by politeness — an unusual spelling falls to the ask profile, a human gate, not a
  silent pass.
- **Immutability without a hook**: edits to `specs/done/` are rejected by a permission rule
  (`edit` on `specs/done/*` → deny) — the same guarantee as Claude Code's hook, with no script
  and no `chmod`. Shell writes through an unrecognized command are not caught — the CI gate
  (`scripts/doctor --strict`) and review are the backstop, same as the other adapters.

All defaults are adjustable — see "Adapting it" in the root README.
