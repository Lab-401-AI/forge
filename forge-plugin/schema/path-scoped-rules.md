# Path-scoped rules and lazy context loading (Layer 1)

This file defines when a project's CLAUDE.md should stop being one big always-loaded file and split into path-scoped rules or subdirectory CLAUDE.md files. Used by `/forge:lint` to recommend structural restructure when a CLAUDE.md is bloated with section-specific content that doesn't apply to every task.

The reason this matters is mechanical, not stylistic: **CLAUDE.md is loaded into context on every turn**, so every word costs tokens forever. Path-scoped content loads only when relevant. The bigger the project, the bigger the difference.

---

## Two lazy-loading mechanisms Claude Code supports

### 1. Subdirectory CLAUDE.md files
Drop a `CLAUDE.md` inside a subdirectory (e.g., `src/CLAUDE.md`, `forge-plugin/CLAUDE.md`). Claude Code loads it on demand — only when Claude reads files in that subdirectory. The root CLAUDE.md still loads every session, but the subdirectory file is silent until relevant.

**Best for:** large directories with their own conventions (a frontend folder vs. a backend folder, an SDK vs. an app, a plugin vs. its host project).

### 2. `.claude/rules/` with `paths:` frontmatter
Each rule is a markdown file with YAML frontmatter naming the file patterns it applies to. Claude loads the rule only when it reads files matching the pattern.

```markdown
---
paths:
  - "src/**/*.{ts,tsx}"
---

# Web app conventions
- Component names use PascalCase
- ...
```

**Best for:** convention sets that span multiple directories or apply to a file *type* rather than a directory (e.g., "test files have these patterns" — applies to `**/*.test.ts` regardless of location).

### Important caveat about `@import` syntax
CLAUDE.md supports `@path/to/file` imports, but **imports do not save context**. The imported file is expanded and loaded at session launch alongside the parent file. Use `@import` for organization (splitting one file into multiple sources of truth), not for lazy loading. Recommend the two mechanisms above when the goal is to reduce context.

---

## When `/forge:lint` should recommend a split

Trigger the recommendation if **any two** of the following are true:

1. **CLAUDE.md is over 600 words.** This is roughly the threshold where signal starts diluting (Anthropic targets under 200 lines for the same reason).
2. **CLAUDE.md has clearly separable concern domains.** Example signals:
   - Conventions for both a frontend (`src/`) and a plugin (`forge-plugin/`).
   - Style rules that only apply to one file type but live in a general "Conventions" section.
   - Architecture notes that apply only to one subdirectory.
3. **The project has multiple top-level subdirectories with different stacks.** A monorepo, a project with both an SDK and an app, or a project with both source code and a plugin/extension.
4. **There's a clear "this section is irrelevant when working on X" boundary.** If a developer working on the iOS half of a project never needs to read the web-app conventions, those conventions belong elsewhere.

---

## How to recommend the split

When recommending, output:

1. **Which mechanism fits.** Subdirectory CLAUDE.md for whole-folder concern domains; `.claude/rules/` with `paths:` for file-pattern concerns.
2. **The specific sections to move.** Quote section headings from the user's CLAUDE.md and say where each one belongs.
3. **The expected size reduction.** Estimate the new root CLAUDE.md word count.
4. **A concrete starter file.** Paste-ready content for the new file, including `paths:` frontmatter if applicable.

Example output shape:

```
The "Conventions > Styling" section (lines 105–109) only applies to web-app
work in `src/`. Move it to `.claude/rules/webapp-styling.md` with this
frontmatter so it loads only when Claude works in `src/`:

  ---
  paths:
    - "src/**/*.{css,jsx}"
  ---

This drops your root CLAUDE.md from ~1,400 to ~1,200 words and means
Claude doesn't carry styling rules during plugin-only work.
```

---

## What does NOT belong in path-scoped rules

Some content needs to be in the root CLAUDE.md regardless of size, because it shapes Claude's behavior across the whole project:

- **Project identity** — what the project is and who it's for.
- **What NOT to do** — universal guardrails (no Tailwind, no TypeScript migration, etc.).
- **The agents list** — Claude needs to know what agents exist before knowing when to use them.
- **Top-level commands** — `npm run dev`, `npm run build` — used from any working directory.
- **Cross-cutting risks** — model ID drift, secrets handling.

Recommend keeping these in the root file. Move *implementation detail* down, not *strategy*.

---

## Failure modes when recommending

1. **Don't split a small file.** Under 600 words, splitting adds cognitive load without saving context.
2. **Don't fragment cross-cutting rules.** If a rule applies to "all CSS files everywhere," splitting it into per-directory copies creates drift.
3. **Don't recommend `@import`-based reorganization as a context fix.** It isn't one. If the user wants tokens back, they need the lazy mechanisms.
4. **Don't auto-execute the split.** Recommend it, give the paste-ready file, let the user move sections. Forge_ never silently restructures someone's CLAUDE.md.
