# Forge_ Plugin

Get more out of Claude Code without needing to be an expert. Forge audits your setup, surfaces what's silently making Claude less effective on your project, and walks you through the fixes — all without leaving the terminal.

The companion to the Forge_ web app at [forge.lab401.ai](https://forge.lab401.ai). The web app helps you set up; the plugin helps you stay sharp once you're working.

## Why this exists

Claude Code is powerful, but most of that power lives in setup details people don't know to look for. A vague subagent description means the agent never fires. A 2,000-word CLAUDE.md eats your context budget every turn. A hook with a typo silently never runs. Three sessions in a row asking the same question Claude can't remember across sessions. None of these show up as errors. They just quietly make Claude less useful than it could be.

Forge_ closes that gap. It reads your actual project, your actual setup files, and your actual session history, and tells you — in plain language — what's working, what's not, and exactly what to change.

## What's different about this vs. just asking Claude

You can ask Claude to review your CLAUDE.md in a regular session. The plugin gives you four things a one-shot conversation can't:

- **Deterministic structural checks.** A bundled rubric catches missing sections, ghost agents, stale paths, and credential leaks the same way every time — not "what Claude noticed today."
- **Live grounding against current docs.** Claude Code ships features faster than any model's training cutoff. Forge fetches Anthropic's docs at runtime so its recommendations don't drift stale.
- **Cross-session pattern analysis.** `/forge:audit` reads transcripts from past sessions to surface what no live conversation can see — what you've re-asked, what's burning context, where Claude got corrected over and over.
- **A stopping criterion.** Forge tells you when your setup is healthy and to stop running it. No tool earns trust if it always finds something to nag about.

## Commands

### `/forge:lint`

Audits your project's `CLAUDE.md` against best practices, explains every finding in plain language, and tells you exactly what to fix.

**What it checks:**
- Required and recommended sections (commands, conventions, architecture, etc.)
- Ghost agents (mentioned in CLAUDE.md but not on disk) and orphaned agents (on disk but not referenced)
- Stale file paths, placeholder rot, model ID drift, file size, credential exposure
- Subagent description quality — every description scored against a four-signal rubric so you know which agents will actually trigger
- Whether your CLAUDE.md should be split into path-scoped subdirectory files
- **Upgrade opportunities** — Claude Code adds features regularly. Lint pulls live docs and surfaces capabilities your project would benefit from but isn't using yet.

Every finding gets a tier (Critical, Important, Polish) and a verdict. A clean file gets a Pass and is told to stop running lint until something changes.

### `/forge:analyze`

Scans your whole project end-to-end and generates a personalized setup pack — paste-ready content for your CLAUDE.md, subagents tailored to your stack, MCP servers worth connecting, and a recommended default model.

Use it when you're starting fresh on a project, when you've inherited a repo with no `.claude/` setup, or when you want a second opinion on whether your existing setup matches your project's actual shape. Recommendations are tied to specific signals in your code — if Forge can't justify a suggestion from real evidence in the project, it drops it instead of padding.

### `/forge:consult`

Opens an interactive Claude Code expert grounded in your actual project. Ask it anything — "should I use a subagent or a hook for this?", "what does this hook syntax mean?", "is my CLAUDE.md too long?" — and it answers in the context of *your* repo, not a generic answer.

Use it when you have a question that doesn't fit lint or analyze. Use it especially when you're new to Claude Code and want a knowledgeable friend explaining things, not a documentation page.

### `/forge:audit`

Reads your past Claude Code session transcripts for this project (stored at `~/.claude/projects/`) and surfaces patterns no live conversation can see.

**What it surfaces:**
- Questions you've re-asked across sessions — usually a sign something belongs in CLAUDE.md so Claude stops needing to be re-told
- Files Claude reads on every turn — context bloat that's silently slowing every response
- Tools that keep failing or getting denied — usually fixable with a permission rule or an allowlist tweak
- Phrases like "no", "stop", "don't" — places you've corrected Claude repeatedly, where the cause is almost always a missing CLAUDE.md fact or a missing agent
- Stuck loops — sessions where Claude churned on the same file for many turns
- Hook health — whether the hooks you've configured actually fire

Every finding ships with a paste-ready fix (a CLAUDE.md addition, a settings tweak, an agent description). Like lint, it produces a verdict and tells you to stop running it on a Clean result.

**Privacy:** transcript data never leaves your machine. The only outbound call is the optional fetch of public Anthropic docs.

### Which command should I use?

| If you're... | Use |
|---|---|
| Starting a fresh project, or inherited one with no setup | `/forge:analyze` |
| Wondering if your existing CLAUDE.md is healthy | `/forge:lint` |
| Confused about how a Claude Code feature works on your project | `/forge:consult` |
| Asking "why does Claude keep getting this wrong?" | `/forge:audit` |

`/forge:audit` rewards habituated use — its value scales with session history. The first three work from day one.

## Install

### Local development

While iterating on the plugin, from the repo root:

```bash
claude --plugin-dir "$(pwd)/forge-plugin"
```

Loads the plugin for the duration of one session. Useful for testing changes without publishing.

### Permanent

```bash
/plugin marketplace add lab-401-ai/forge
/plugin install forge@lab401
```

Run from inside Claude Code. Updates flow when Lab_401 ships a new version — refresh with `/plugin marketplace update lab401`.

## How Forge knows what "good" looks like

Forge doesn't ask Claude to grade Claude. Every recommendation is grounded in four explicit layers, and every command's output cites which layers fired.

| Layer | Source | When it runs |
|---|---|---|
| **1. Bundled schema** | Deterministic rules in `schema/` (structural checks, severity tiers, agent-description rubric, coverage map, path-scoped-rules) | Every invocation. Free, fast, offline. |
| **2. Live docs** | Anthropic's canonical pages via `WebFetch` | When qualitative or current-spec guidance is needed. Always current. |
| **3. Community corpus** | Curated public repos (`awesome-claude-code-subagents`, etc.) | When suggesting agents, only as examples, never as authority. |
| **4. Project context** | Your CLAUDE.md, package.json, `.claude/` directory, session transcripts | Always — it's what makes the output specific instead of generic. |

This four-layer architecture is the moat. Static linters can't do Layer 2. Generic LLM advice can't do Layer 4. The audit command leans hardest on Layer 4; lint and analyze use all four.

## Verifying each layer is actually firing

Every command's output ends with a "How this was grounded" footer that names which layers ran. Use these tests to confirm each layer in isolation.

### Layer 1 — Bundled schema

```bash
/forge:lint
```

The footer should list `Bundled schema: ${CLAUDE_PLUGIN_ROOT}/schema/claude-md.md`. If you delete or rename a schema file and re-run, the affected step should fail loudly — not silently fall back to vibes.

### Layer 2 — Live docs

```bash
# Run lint in a project with a sparse CLAUDE.md:
/forge:lint
```

The footer should name a specific URL it fetched (e.g., `https://www.anthropic.com/engineering/claude-code-best-practices`). To stress-test, disconnect your network and re-run — the footer should explicitly say `Live docs fetched: none — fetch failed` and the report should fall back to bundled schema only, without inventing claims.

### Layer 3 — Community corpus

```bash
# In a project that needs agent recommendations:
/forge:analyze
```

When subagent suggestions reference real-world examples ("similar to the `code-reviewer` agent in `awesome-claude-code-subagents`"), Layer 3 fired. If recommendations are 100% from the spec with no examples cited, the corpus wasn't used.

### Layer 4 — Project context

```bash
# Run analyze in two completely different projects.
/forge:analyze
```

The two outputs should look almost nothing alike. If they look similar, the project-context layer is being ignored and Forge is generating generic advice — which is the failure mode the whole architecture is built to prevent.

## Plugin structure

```
forge-plugin/
├── .claude-plugin/
│   └── plugin.json                    # Manifest
├── skills/
│   ├── lint/SKILL.md                  # /forge:lint
│   ├── analyze/SKILL.md               # /forge:analyze
│   ├── consult/SKILL.md               # /forge:consult
│   └── audit/SKILL.md                 # /forge:audit
├── schema/
│   ├── claude-md.md                   # Layer 1 — structural rules for CLAUDE.md
│   ├── severity-rubric.md             # Layer 1 — tiers each finding (Critical/Important/Polish/Pass)
│   ├── agent-description-rubric.md    # Layer 1 — four-signal rubric for subagent descriptions
│   ├── agent-coverage-map.md          # Layer 1 — maps project signals to agent types
│   ├── path-scoped-rules.md           # Layer 1 — when to split CLAUDE.md into subdirectory files
│   ├── canonical-urls.md              # Layer 2 — live-fetch targets
│   └── community-corpus.md            # Layer 3 — corpus references
└── README.md
```

## License

MIT
