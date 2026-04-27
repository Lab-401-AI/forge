# What a good CLAUDE.md looks like

This is Forge's bundled checklist — Layer 1 of the four-layer grounding architecture. Skills load this file before evaluating a user's CLAUDE.md so the structural verdict is deterministic, not "vibes from training data."

---

## File-level rules

| Rule | Threshold | Why |
|---|---|---|
| File exists | `CLAUDE.md` at project root | Without it, Claude has no project context and re-derives everything every session. |
| File size | Under 500 lines / ~25 KB | Long CLAUDE.md files burn context every turn. If it's longer, sections should be split into `.claude/rules/*.md` or skills. |
| Plain language | Yes | The file is read by Claude *and* by future humans on the team. Jargon-heavy files age badly. |

## Required sections

A CLAUDE.md missing any of these should be flagged as **needs work**.

### 1. Project description
**What it is:** One short paragraph explaining what the project does and who it's for.
**Why Claude needs it:** Without this, Claude treats every task as if it's seeing the codebase for the first time.
**Smell test:** A new contributor should understand what this project is from this section alone.

### 2. Commands
**What it is:** The actual shell commands to run, build, test, deploy.
**Why Claude needs it:** Otherwise Claude guesses. Guessing leads to running the wrong test runner or build script.
**Smell test:** Could a new dev get the project running using only this section?

### 3. Conventions / style
**What it is:** Naming patterns, file organization, what to prefer (e.g., "edit existing files instead of creating new ones," "use TypeScript over JavaScript").
**Why Claude needs it:** Without explicit conventions, Claude defaults to its training data, which is averaged across all projects — not yours.

### 4. Architecture / structure
**What it is:** Where the important code lives. Key directories, how layers talk to each other, any non-obvious decisions.
**Smell test:** Could Claude navigate to the right file on the first try without grepping the whole repo?

## Recommended sections

A CLAUDE.md missing these should be flagged as **suggested**, not failed.

### 5. Tech stack
Framework, language, build tool, key dependencies. Helps Claude pick idiomatic solutions.

### 6. What NOT to do
Things Claude tends to do wrong on this project. ("Don't add Tailwind, we use vanilla CSS." "Don't run migrations without asking.") The most underused section, often the highest-leverage.

### 7. Subagent / skill references
If the project has `.claude/agents/` or `.claude/skills/`, mention what they are and when to use them.

### 8. Deployment notes
How does this ship? Where? Any environment variables or secrets to know about?

## Anti-patterns to flag

| Anti-pattern | Why it's bad |
|---|---|
| Marketing-style prose | "Our cutting-edge platform..." — Claude doesn't care, this wastes context. |
| Repeating what the code already says | "We have a `users` table with id, name, email" — Claude can read the schema. Document the *why*, not the *what*. |
| Stale commands | Commands that no longer work erode trust. If the file says `npm run lint` and there's no lint script, Claude will run it anyway and waste time. |
| No update markers | If CLAUDE.md hasn't been touched in 6 months and the project has shipped 200 commits, it's likely stale. |
| Documentation that belongs in code comments | Function-level details should be near the function, not in the project-level CLAUDE.md. |

## CLAUDE.local.md

The personal/uncommitted layer. Lives next to CLAUDE.md but is gitignored. Almost no project uses this well — it's the highest-leverage gap Forge can address.

**What belongs in CLAUDE.local.md:**
- Personal API tokens or local-only credentials
- Machine-specific paths (`~/work/notes/` etc.)
- Personal scratch notes ("ignore the test suite for now, working on a hotfix")
- Per-developer tool preferences

**What does NOT belong:**
- Anything the team needs to know — that goes in CLAUDE.md
- Secrets that should be in environment variables — those still belong in `.env`, not in Claude's context

**Forge's lint check:** if `CLAUDE.local.md` is missing, that's not a failure. It's a suggestion: "Want a place for personal notes Claude reads but your team doesn't see? Here's what to put in it."

## .claude/ directory

If the project has a `.claude/` directory:
- `.claude/agents/` — subagent definitions (markdown with frontmatter)
- `.claude/skills/` — bundled skills (each is a directory with `SKILL.md`)
- `.claude/commands/` — older flat-file format (still works, but `skills/` is preferred for new work)
- `.claude/settings.json` — project-scoped settings

Forge should note which of these exist and whether they're referenced from CLAUDE.md (so the human reader knows they're there too).
