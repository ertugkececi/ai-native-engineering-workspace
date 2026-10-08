# Spec 0001 — OpenCode adapter (mini)

- Status: In progress
- Mode: lite (the contributing fork runs the template's own loop; the upstream repo is the template itself)
- Plan: `specs/plans/0001-plan.md`
- Source: maintainer alignment via DM — engindemirog/ai-native-engineering-workspace; public issue to follow
- Design: none — no user interface
- Supersedes:

## Intent
Contributors and teams on OpenCode cannot run the ANEW loop with the commands the other adapters
expose. This change adds a fourth adapter (installed by `./scripts/init opencode`) that keeps the
core untouched, states every rule once (ADR 0001), and — on OpenCode — turns several previously
Documented rules into Enforced ones (role agents, shipped-spec immutability, destructive-command
denies).

## Changed behavior
- [ ] CB-1 — `./scripts/init opencode` installs `.opencode/` (12 commands, 4 agents,
  `opencode.json`); existing files are kept unless `--force`; a second run keeps everything.
- [ ] CB-2 — the 12 commands carry the same names and `$ARGUMENTS` semantics as the other
  adapters; `AGENTS.md` is auto-loaded by the tool (no pointer file).
- [ ] CB-3 — the read-only `reviewer` subagent cannot edit any file; its shell is narrowed to
  `./scripts/check` and read-only git.
- [ ] CB-4 — edits under `specs/done/`, force-push/`-f`, hard reset, rebase and `rm -rf` are
  denied by configuration (no hook, no script).
- [ ] CB-5 — role segment commands pin their agent (`/analyze` → analyst, `/plan` and `/build` →
  developer, `/verify` → qa); path-level edit limits enforce the role cards where expressible
  (analyst → `specs/active/`, qa → `tests/`); content-level clauses stay Documented.

## Preserved behavior
- [ ] PB-1 — `./scripts/init claude-code|github-copilot|cursor|generic` file sets and behavior
  unchanged.
- [ ] PB-2 — `./scripts/doctor` output on an upstream-equivalent tree differs only in the
  adapter-presence message (now listing `opencode`); parity check stays green; `--strict` exit 0.
- [ ] PB-3 — `init`'s keep-unless-`--force` contract intact (second run prints "kept").

## Out of scope
- Core files (`AGENTS.md`, `docs/`, `workflows/`, `prompts/`, `scripts/check`) — untouched.
- Extending `scripts/doctor`'s parity check to a fourth tool — maintainer decision, asked in the
  issue.
- Whether these `specs/` records merge upstream or stay in the fork as PR evidence — kept as a
  drop-able commit.

## Definition of Done
- [ ] `scripts/check` — the template has no `check.conf`; CI skips by design (`check.yml`);
  `./scripts/doctor --strict` exits 0
- [ ] Independent review done; real findings fixed, noise rejected with written rationale
- [ ] Criterion ↔ evidence table complete for CB-* and PB-*
- [ ] Spec moved to `specs/done/` at ship (on upstream merge; recorded in the fork)
