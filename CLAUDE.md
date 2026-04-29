# CLAUDE.md — Forge_ (CLI plugin repo)

## What this repo is

This is the public repository for the Forge_ Claude Code plugin. It ships through the Lab_401 marketplace and adds four commands to Claude Code: `/forge:lint`, `/forge:analyze`, `/forge:consult`, `/forge:audit`.

User-facing docs are in [`README.md`](./README.md). Path-scoped editing conventions for the plugin live in [`forge-plugin/CLAUDE.md`](./forge-plugin/CLAUDE.md) and load only when Claude works inside `forge-plugin/`.

The companion web app (`lab-401-ai/forge-web`) is in a separate repo. They share a brand but ship independently — don't conflate the two.

## Audience

The people using the plugin aren't necessarily seasoned engineers. They might be designers, founders, marketers, or hobbyists who picked up Claude Code and are figuring it out as they go. Plugin output should never make them feel like they're missing context.

## Tone for plugin output

When Forge talks to users — in skill output, recommendations, footers — it should sound like a knowledgeable friend explaining something over coffee, not a developer writing documentation.

- Use plain words. "Your setup file" not "your CLAUDE.md." "AI assistant" not "LLM." Introduce jargon only when necessary, and explain it when you do.
- Lead with what something means for the user before naming the thing.
- Keep it short. If something can be said in one sentence, don't use three.

## Repo layout

```
.claude-plugin/
  marketplace.json         # Lab_401 marketplace manifest
forge-plugin/              # the plugin itself
  .claude-plugin/plugin.json
  skills/                  # /forge:lint, /forge:analyze, /forge:consult, /forge:audit
  schema/                  # bundled grounding rules
  CLAUDE.md                # path-scoped editing conventions
.claude/
  agents/plugin-skill-validator.md
README.md                  # user-facing storefront
LICENSE                    # MIT
```

## Commands (for working on this repo)

There's no build step. The plugin is loaded directly from disk:

```bash
claude --plugin-dir "$(pwd)/forge-plugin"
```

## Agents

One subagent in `.claude/agents/`:

- **plugin-skill-validator** — run after editing any file under `forge-plugin/skills/`. Checks SKILL.md frontmatter, body structure, and tool references.

## What NOT to do

- Don't hardcode model IDs in skills — defer to whatever model Claude Code is running.
- Don't add a fifth skill without revisiting whether the existing four already cover it. The four-skill set was deliberate.
- Don't add files to this repo that aren't plugin-essential. Web-app material belongs in `lab-401-ai/forge-web`.

## Deployment

The plugin publishes through the Lab_401 marketplace declared in `.claude-plugin/marketplace.json`. End users install with:

```
/plugin marketplace add lab-401-ai/forge
/plugin install forge@lab401
```

When `forge-plugin/.claude-plugin/plugin.json` version bumps and the change lands on `main`, marketplace consumers pick it up via `/plugin marketplace update lab401`.

## Related

- **`lab-401-ai/forge-web`** — the companion web app (private repo, deploys to forge.lab401.ai)
- **`lab401-web`** — parent site repo
- **Brand:** Lab_401 — AI software development company
