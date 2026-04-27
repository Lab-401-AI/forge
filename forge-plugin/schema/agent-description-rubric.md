# Agent description rubric (Layer 1)

This is Forge's deterministic rubric for scoring whether a subagent's `description` field will reliably trigger Claude to invoke it at the right moment.

The whole point of a subagent is delegation — Claude reads the descriptions and routes the right work to the right agent. A vague description means the agent never fires, no matter how well-written the body is. So the description is the load-bearing field.

Both `/forge:lint` (audit existing agents) and `/forge:analyze` (generate new agents) load this file.

---

## The four signals of a strong description

Score each agent's description against these four signals. Each is binary — either the description carries the signal or it doesn't.

### 1. Trigger condition

Does the description name a specific moment when Claude should invoke this agent? "When," "after," "before," "if" are the load-bearing words.

| Pass | Fail |
|---|---|
| "Use **after any edit to** `src/Forge.css`..." | "Helps with CSS." |
| "Run **before committing** to validate..." | "Validates commits." |
| "Use **when** the user mentions deployment..." | "Knows about deployment." |

### 2. Scope or file pattern

Does the description name *what* triggers it concretely — a file path, a file type, a topic, a command? "Anything CSS-related" is too fuzzy. "`src/**/*.css`" or "`.claude/agents/*.md`" is concrete enough for Claude to pattern-match.

| Pass | Fail |
|---|---|
| "...edits to `forge-plugin/skills/**/*`..." | "...changes to plugin files..." |
| "...questions about the API integration..." | "...help with the backend..." |

### 3. Outcome or check named

Does the description say *what the agent actually does* — verifies, reviews, generates, audits? Vague verbs ("helps," "handles," "deals with") fail. Specific verbs that name an outcome pass.

| Pass | Fail |
|---|---|
| "...**verifies** new rules use `--fg-*` tokens..." | "...handles styling concerns..." |
| "...**reviews** API fetch calls for model ID and headers..." | "...helps with API stuff..." |

### 4. Read-only declared (when applicable)

For audit and review agents, does the description say "Read-only" or equivalent? Without it, Claude may hesitate to invoke the agent during an active edit because it's unclear whether the agent will write back. Read-only agents are safe at any time and Claude will reach for them more readily when this is explicit.

| Pass | Fail |
|---|---|
| "...validates SKILL.md frontmatter and tool refs. **Read-only.**" | "...validates SKILL.md frontmatter." |

This signal is N/A for agents that *do* write or run commands. Don't penalize an editor agent for not declaring read-only.

---

## Scoring

For each agent description, count signals present (0–4, or 0–3 if read-only is N/A).

| Score | Verdict |
|---|---|
| 4 / 4 (or 3/3) | **Strong** — Claude will route to this agent reliably. |
| 3 / 4 | **Acceptable** — will trigger most of the time. Note the missing signal. |
| 2 / 4 | **Weak** — will fire occasionally. Recommend rewriting. |
| 0–1 / 4 | **Will not trigger** — agent exists but is dead weight. Strong recommendation to rewrite. |

---

## Common failure patterns to flag

These are specific shapes of bad descriptions worth calling out by name when you find them:

1. **The buzzword sandwich** — full of nouns, no verbs, no triggers. ("CSS, styling, design, frontend.")
2. **The job title** — names the *role* not the *action*. ("Frontend specialist." "API expert.")
3. **The truism** — describes what it does in a way that's true of any agent. ("Helps with code.")
4. **The novel** — three paragraphs of body text crammed into the description field. The description is what Claude scans; the body is what Claude reads after invocation. Don't merge them.

---

## How skills use this file

**`/forge:lint`** — for each agent in `.claude/agents/`, score the description, attach a verdict. Surface weak/dead-weight agents in the output.

**`/forge:analyze`** — when generating new agent recommendations, write descriptions that hit all four signals. Re-read each draft against this rubric before printing.
