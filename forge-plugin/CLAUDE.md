# forge-plugin/ — CLI plugin half of Forge_

The CLI plugin ships four skills the user can invoke from inside Claude Code:

- `skills/lint/` — audits a project's CLAUDE.md (`/forge:lint`)
- `skills/analyze/` — generates tailored Claude Code setup recommendations (`/forge:analyze`)
- `skills/consult/` — interactive Q&A grounded in the current project (`/forge:consult`)
- `skills/audit/` — analyzes past session transcripts at `~/.claude/projects/` to surface wasted tokens, repeated questions, and patterns Claude got wrong (`/forge:audit`)

When working on plugin behavior, edit the relevant `SKILL.md`. The plugin manifest is at `.claude-plugin/plugin.json`.

## Bundled grounding schema (`schema/`)

- `claude-md.md` — Layer 1 deterministic rules (required sections, file size, anti-patterns). Used by `/forge:lint` offline, no credits.
- `canonical-urls.md` — Layer 2 fetch targets (Anthropic docs pages). Skills fetch these at runtime for always-current guidance.
- `community-corpus.md` — Layer 3 real-world examples (public agent repos). Used for suggestion quality, cited by name, never as authority.
- `agent-description-rubric.md` — four-signal scoring rubric for agent descriptions. Used by both `/forge:lint` (audit existing agents) and `/forge:analyze` (generate new agents that will actually trigger).
- `agent-coverage-map.md` — maps project signals to agent types that typically guard them. Used by `/forge:analyze` for coverage gap analysis.
- `path-scoped-rules.md` — defines when CLAUDE.md should be split into subdirectory CLAUDE.md or `.claude/rules/`. Used by `/forge:lint` for restructure recommendations.

## Conventions for editing skills

- After editing any file under `skills/`, run the `plugin-skill-validator` subagent.
- Every SKILL.md must include `${CLAUDE_PLUGIN_ROOT}/schema/<filename>` references via the bundled-schema path — never relative paths.
- `Bash` in `allowed-tools` must always be scoped (`Bash(ls:*)`, `Bash(find:*)`, etc.). Unscoped `Bash` is a red flag.
- Every skill output must end with a "How this was grounded" footer naming which schemas and URLs fired. The four-layer architecture is the moat — the footer is how it's verifiable.

## What NOT to do

- Don't add a fifth skill without revisiting whether the existing four already cover it. The four-skill set (lint, analyze, consult, audit) was deliberate; expanding it past four needs the same kind of justification as adding the fourth.
- Don't hardcode model IDs in skill bodies — skills should defer to whatever model Claude Code is running.
- Don't fetch every canonical URL on every invocation. Cap at two fetches per skill run. `/forge:lint` may use both slots (one for grounding, one for the upgrade-opportunities scan); `/forge:audit` uses at most one.
