# Forge_ Plugin

Get more out of Claude Code without needing to be an expert. Forge audits your CLAUDE.md, suggests agents tailored to your project, and walks you through setup conversationally — all without leaving the terminal.

The companion to the Forge_ web app at [forge.lab401.ai](https://forge.lab401.ai). The web app helps you set up; the plugin helps you stay sharp once you're working.

## Commands

| Command | What it does |
|---|---|
| `/forge:lint` | Audits your current project's CLAUDE.md against best practices and explains what to fix in plain language. |
| `/forge:analyze` | Scans your whole project and generates a personalized setup pack — what to put in CLAUDE.md, which agents to add, which model to default to. |
| `/forge:consult` | Opens an interactive Claude Code expert consultation grounded in your actual project. |

## Install (local development)

While iterating on the plugin:

```bash
# From the directory containing this README:
claude --plugin-dir "$(pwd)"
```

This loads the plugin for the duration of one session. Useful for testing changes without publishing.

## Install (permanent)

Once published to a marketplace:

```bash
claude plugin install forge@<marketplace-name>
```

## How Forge knows what "good" looks like

Forge doesn't ask Claude to grade Claude. Every recommendation is grounded in four explicit layers, and every command's output cites which layers fired.

| Layer | Source | When it runs |
|---|---|---|
| **1. Bundled schema** | `schema/claude-md.md` (deterministic structural rules) | Every invocation. Free, fast, offline. |
| **2. Live docs** | Anthropic's canonical pages via `WebFetch` | When qualitative guidance is needed. Always current. |
| **3. Community corpus** | Curated public repos (`awesome-claude-code-subagents`, etc.) | When suggesting agents, only as examples, never as authority. |
| **4. Project context** | Your CLAUDE.md, package.json, `.claude/` directory | Always — it's what makes the output specific instead of generic. |

This four-layer architecture is the moat. Static linters can't do Layer 2. Generic LLM advice can't do Layer 4.

## Verifying each layer is actually firing

Every command's output ends with a "How this was grounded" footer that names which layers ran. Use these tests to confirm each layer in isolation.

### Layer 1 — Bundled schema

```bash
# In any project, run:
/forge:lint
```

The footer should list `Bundled schema: ${CLAUDE_PLUGIN_ROOT}/schema/claude-md.md`. If you delete or rename the schema file and re-run, lint should fail loudly — not silently fall back to vibes.

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
│   └── plugin.json              # Manifest (only `name` is required)
├── skills/
│   ├── lint/SKILL.md            # /forge:lint
│   ├── analyze/SKILL.md         # /forge:analyze
│   └── consult/SKILL.md         # /forge:consult
├── schema/
│   ├── claude-md.md             # Layer 1 — structural rules
│   ├── canonical-urls.md        # Layer 2 — fetch targets
│   └── community-corpus.md      # Layer 3 — corpus references
└── README.md
```

## License

MIT
