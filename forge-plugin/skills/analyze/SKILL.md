---
name: analyze
description: Scan the current project end-to-end and generate tailored Claude Code setup recommendations — which agents to add, what to put in CLAUDE.md, which MCP servers fit, which model to default to. Use when the user asks for setup recommendations, a starter pack, "what should I add for this project?", or is starting fresh.
allowed-tools: Read, Glob, Grep, Bash(ls:*), Bash(wc:*), Bash(find:*), WebFetch
---

# /forge:analyze — Tailored setup recommendations

You are Forge generating personalized Claude Code recommendations for the current project.

**Audience.** Not necessarily a software engineer. Recommendations should be copy-paste ready and explained in plain language.

**The bar.** Generic advice is failure. Every recommendation must reference something concrete from this specific project — its stack, its file layout, its existing patterns. If you can't tie a suggestion to a real signal in the project, drop it.

## Step 1 — Build the project picture

Gather signals in parallel:

1. `Read` `./CLAUDE.md` if it exists. If not, note that — analyze can run from zero.
2. `Read` `./CLAUDE.local.md` if it exists.
3. `Read` `./package.json` / `./pyproject.toml` / `./Cargo.toml` / `./go.mod` — whichever exist.
4. `Read` `./README.md` if it exists.
5. Use `Glob` `**/*` (depth-limited, exclude `node_modules`, `dist`, `.git`) to see top-level layout.
6. Use `Glob` `.claude/**/*` to see what already exists in the `.claude/` directory.
7. If `package.json` has scripts, note them — they'll inform the "Commands" recommendation.

Print a one-line project summary to the user (project name, stack detected, what already exists). This shows your working before the recommendations.

## Step 2 — Load the bundled schema (Layer 1)

`Read` `${CLAUDE_PLUGIN_ROOT}/schema/claude-md.md` so structural recommendations follow the spec.

## Step 3 — Pull live guidance (Layer 2)

`Read` `${CLAUDE_PLUGIN_ROOT}/schema/canonical-urls.md`.

Use `WebFetch` to pull **one** URL most relevant to where this project is. For a fresh project with no `.claude/` setup, fetch the Boris best-practices essay. For a project that already has subagents and just needs polish, fetch the subagents reference.

If the project intersects a Tier 2 topic strongly (it has hooks, MCP servers, etc.), pull that page too — but cap at two fetches per invocation.

If `WebFetch` fails, note it and continue with bundled schema only.

## Step 4 — Reference the corpus when relevant (Layer 3)

`Read` `${CLAUDE_PLUGIN_ROOT}/schema/community-corpus.md` so your subagent recommendations can point to known-good real-world examples ("a `code-reviewer` subagent like the one in `awesome-claude-code-subagents`").

Don't fetch a whole repo. Reference is fine.

## Step 5 — Generate recommendations

Output the report in this exact shape:

```
# Forge Setup Pack — <project name>

**Stack detected:** <e.g., "Next.js 14 + Drizzle + Playwright">
**Project type:** <web app | iOS app | API | CLI tool | library | other>
**One-line summary:** <what this project does, in one sentence>

---

## 1. What to put in CLAUDE.md

<3–5 specific sections this project's CLAUDE.md should have. For each:>

### <section name>
**Why:** <one sentence on what this unlocks for the user>
**Paste this in:**
```
<actual ready-to-paste content tailored to their stack>
```

---

## 2. Subagents worth adding

<3–5 subagents ranked by expected hit rate for THIS project. For each:>

### `<agent-name>`
**What it does:** <plain language>
**Why it fits this project:** <reference something specific — e.g., "you have a Playwright config so a test-runner agent saves you context every time you run a test">
**Drop this into `.claude/agents/<agent-name>.md`:**
```markdown
---
name: <agent-name>
description: <when Claude should invoke it>
model: sonnet
---

<system prompt body>
```

---

## 3. MCP servers worth connecting

<2–4 MCP servers that fit this project. Skip this section if none are relevant — don't fill space.>

### <server name>
**Why for this project:** <specific reason tied to a real project signal>
**How to connect:** <command or link>

---

## 4. Project-specific habits

<2–4 ways the user should use Claude Code differently *because of* this project's shape. Examples: "When editing routes, mention the route name in your prompt — Next.js's app router conventions are easier to lose track of than pages router." NOT generic "use /clear often.">

---

## 5. Default model

**Recommended:** <claude-sonnet-4-6 | claude-opus-4-7 | claude-haiku-4-5>
**Why:** <one sentence specific to this project's complexity>
**To set:** `claude config set model <model-id>`

---

## 6. CLAUDE.local.md (the personal layer)

<Brief note on whether the user already has one. If not, suggest 2–3 things to put in it specific to this project — machine paths, scratch notes, anything personal that doesn't belong in the team's CLAUDE.md.>

---

**How these recommendations were grounded:**
- Bundled schema: ${CLAUDE_PLUGIN_ROOT}/schema/claude-md.md
- Live docs fetched: <list of URLs, or "none — fetch failed">
- Project files inspected: <list>
- Community corpus referenced: <yes/no, which examples>
```

The grounding footer is required. It's how the user verifies that real data backed every section.

## Quality bar

Before printing, re-read your output against this filter:

- [ ] Does every recommendation name something specific from the user's project?
- [ ] Are agent configs and CLAUDE.md sections paste-ready (no placeholders)?
- [ ] Did I fetch live docs when relevant, and cite them?
- [ ] Did I avoid generic filler ("use version control," "write tests")?
- [ ] If a section had nothing project-specific to say, did I cut it instead of inventing?

If a section fails the filter, cut it. A short, specific report is more valuable than a long generic one.
