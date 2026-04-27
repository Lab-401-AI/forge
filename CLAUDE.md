# CLAUDE.md — Forge_

## What Forge_ is

Forge_ helps anyone using Claude Code get more out of it — without needing to know what they're doing. It guides you through the setup, analyzes your project, and tells you what to add or change so your AI actually works the way you intended.

The core promise: **you have the vision, Forge_ makes sure your AI delivers it.**

The people using this aren't necessarily engineers. They might be designers, founders, marketers, or hobbyists who picked up Claude Code and are figuring it out as they go. The app should never make them feel like they're missing context.

## Two products, one brand

Forge_ is becoming two complementary tools that serve the same user at different moments:

**The web app** (what's built) — forge.lab401.ai. You open it in a browser, describe what you're building, and Forge analyzes your project and gives you personalized recommendations. It's where you go to set things up, understand your options, and get a starting point. Visual, guided, and approachable.

**The CLI plugin** (what's next) — lives inside Claude Code itself. Once you're working on a project, you call `/forge:lint` or `/forge:consult` without ever leaving the terminal. It reads your actual project files and tells you what's off, what's missing, and what to do next. No browser, no copy-pasting.

These aren't competing products. The web app is for getting started. The plugin is for staying sharp as the project grows.

## Where Forge wins (the actual wedge)

Several tools already lint or scaffold CLAUDE.md files. Forge's defensible ground is the combination of things none of them do:

- **CLAUDE.local.md curation** — the personal, uncommitted layer of Claude Code setup. Nobody handles this. It's where user-specific tokens, notes, and machine paths should live, and there's no guidance for it anywhere.
- **Codebase-aware suggestions** — not templates. Forge scans what you've actually built and says "given your stack, here are the 4 agents that would save you the most time." That's a different class of recommendation.
- **Conversational refinement** — other tools return pass/fail. Forge walks you through it. Explaining why something matters before telling you to change it.
- **Living rules** — Claude Code updates frequently. Forge's recommendations should pull from current documentation, not hardcoded rules that go stale.

## Current state

The web app is roughly 70% complete. The core flow works. The next phase is structural cleanup and the beginning of the plugin work.

### What exists now
- Single-page React app with three-panel layout: sidebar nav, main content, AI chat
- 10 sidebar sections covering setup steps, reference material, and project analysis
- Interactive 9-step initialization checklist (persisted across sessions)
- Project analysis: describe your project or upload files → Claude generates tailored recommendations per section
- AI chat panel powered by Anthropic API for real-time Q&A
- Ember orange accent color (#e0663c), dark theme, monospace typography

### What needs to change
- **Separate guide content from personalized recommendations.** Right now they're mixed together in a way that's hard to scan. Best practices should feel like reference material; your specific recommendations should feel like a personal briefing.
- **Simplify navigation.** 10 flat sidebar items is too many. Group or layer them so the user always knows where they are and what's next.
- **Progressive flow for new users.** Someone opening Forge for the first time should feel guided. A returning user should be able to jump straight to what they need.

## Tone and communication

When Forge talks to users — in the UI, in recommendations, in the chat — it should sound like a knowledgeable friend explaining something over coffee, not a developer writing documentation.

- Use plain words. "Your setup file" not "your CLAUDE.md." "AI assistant" not "LLM." Introduce jargon only when necessary, and explain it when you do.
- Lead with what it means for the user, not what the thing is. "This tells Claude how to behave in your project" before "this is the CLAUDE.md file."
- Keep it short. If something can be said in one sentence, don't use three.
- Never assume the user knows what something is just because it's obvious to a developer.

## Tech stack

- **Framework:** React 18 (no router — single-page app)
- **Build tool:** Vite 6 (ESM-only, `"type": "module"`)
- **Language:** JavaScript (JSX) — no TypeScript
- **Styling:** Plain CSS with custom properties prefixed `--fg-*`
- **AI:** Anthropic API via direct browser access (`anthropic-dangerous-direct-browser-access` header)
- **State:** React hooks + localStorage for persistence (checklist, project description, recommendations)
- **Node:** >=20

## Commands

```bash
npm run dev        # Start dev server
npm run build      # Production build → dist/
npm run preview    # Preview production build
```

## Project structure

```
src/
  main.jsx         # Entry point — renders Forge into #root
  Forge.jsx        # Entire app (~950 lines, single component + helpers)
  Forge.css        # All styles (~980 lines)
public/            # Static assets
.env               # VITE_ANTHROPIC_API_KEY (gitignored)
```

Key prompt locations inside `src/Forge.jsx` (do not move without updating references in plugin skills):
- `ANALYSIS_PROMPT` — lines 29–46. Drives the project analysis API call; defines the JSON shape the app expects back.
- `MODES` object — lines 48–115. Three system prompts: `claude-code` (expert consult), `architect` (design review), `rubber-duck` (thinking partner). The consult skill pulls from `claude-code`.

## What this repo also contains

The `forge-plugin/` directory is the CLI plugin half of Forge_. It ships three skills:

- `forge-plugin/skills/lint/` — audits a project's CLAUDE.md (`/forge:lint`)
- `forge-plugin/skills/analyze/` — generates tailored Claude Code setup recommendations (`/forge:analyze`)
- `forge-plugin/skills/consult/` — interactive Q&A grounded in the current project (`/forge:consult`)

When working on plugin behavior, edit the relevant `SKILL.md`. The plugin manifest is at `forge-plugin/.claude-plugin/plugin.json`.

Bundled grounding schema in `forge-plugin/schema/`:
- `claude-md.md` — Layer 1 deterministic rules (required sections, file size, anti-patterns). Used by `/forge:lint` offline, no credits.
- `canonical-urls.md` — Layer 2 fetch targets (Anthropic docs pages). Skills fetch these at runtime for always-current guidance.
- `community-corpus.md` — Layer 3 real-world examples (public agent repos). Used for suggestion quality, cited by name, never as authority.

## Agents

Four specialized subagents live in `.claude/agents/`. Use them proactively:

- **anthropic-fetch-reviewer** — after any change to API fetch calls or prompts in `src/Forge.jsx`
- **forge-css-token-guardrail** — after any edit to `src/Forge.css`
- **forge-jsx-monolith-sentry** — before adding significant logic to `src/Forge.jsx`
- **plugin-skill-validator** — after editing any file under `forge-plugin/skills/`

## What NOT to do

- Don't add Tailwind, CSS-in-JS, or any styling library — vanilla CSS with `--fg-*` custom properties is the convention.
- Don't migrate to TypeScript without asking. The project is intentionally JS.
- Don't refactor `src/Forge.jsx` into smaller components as part of an unrelated task. It's a known ~950-line monolith and breaking it up is its own scoped piece of work.
- Don't hardcode the Anthropic model ID in new code without checking the current one (currently `claude-sonnet-4-6`).
- Don't conflate the web app and the plugin. They share a repo but ship independently — changes to `src/` don't touch `forge-plugin/` and vice versa.

## Conventions

### Styling
- All CSS custom properties prefixed with `--fg-` to namespace away from any parent site
- Dark theme only: `--fg-bg: #0f0f0f`, accent: `--fg-accent: #e0663c`
- BEM-like class naming: `forge-component`, `forge-component--modifier`, `forge-component-element`
- Two font stacks: `--fg-mono` (SF Mono/Menlo) for code, `--fg-sans` (system) for body

### Component patterns
- Currently a single monolithic component — should be broken into smaller components as the app matures
- `useCallback` for all handlers to prevent unnecessary re-renders
- localStorage keys prefixed with `forge_` (e.g., `forge_completed`, `forge_project_desc`, `forge_recommendations`)

### API calls
- All Anthropic API calls use `import.meta.env.VITE_ANTHROPIC_API_KEY`
- Two API integration points: project analysis (structured JSON response) and chat (conversational)
- Model: `claude-sonnet-4-6` for both
- **Model drift risk:** both call sites must stay in sync. If you update the model string in one place, grep for the other before committing. The `anthropic-fetch-reviewer` subagent checks this automatically.

## Design principles

- **Clarity over cleverness.** Every screen should be immediately scannable. No hidden menus, no mystery icons.
- **Guide first, reference second.** New users follow a path. Experienced users browse freely. Both work without friction.
- **Personalization earns trust.** Recommendations should feel specific and useful, never generic. If the analysis can't add real value to a section, show nothing rather than filler.
- **Meet the user where they are.** The target user may not know what an agent, a hook, or a subagent is. That's fine. Forge explains it in context, without condescending.

## Deployment

- **Parent site:** lab401.ai (separate repo: lab401-web)
- **Apps page tile** on lab401.ai links to Forge_ at `https://forge.lab401.ai`
- Deployment pipeline TBD — will likely mirror lab401-web (S3 + CloudFront)

## Related

- **lab401-web repo:** Contains the Apps page tile and GuideCover animation for Forge_
- **Brand:** Lab_401 — AI software development company
