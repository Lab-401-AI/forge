# CLAUDE.md — Forge_

## What Forge_ is

Forge_ is a developer tool that ensures your AI framework is optimized to deliver your vision. It's both a setup guide and an ongoing reference — not something you use once and forget, but a workspace you return to as your project evolves.

The core promise: **you have the vision for your build, Forge_ ensures your AI delivers it.**

Target audience is any developer using Claude Code — from first-timers who need clear setup steps to experienced devs fine-tuning their AI workflow.

## Current state

The app works but needs a structural overhaul. The current build mixes best practices with project-specific customizations in a way that feels unorganized. The next phase should focus on clean separation and intuitive navigation.

### What exists now
- Single-page React app with three-panel layout: sidebar nav, main content, AI chat
- 10 sidebar sections mixing setup steps, reference material, and project analysis
- Interactive 9-step initialization checklist with localStorage persistence
- Project analysis: users describe their project or upload files, Claude analyzes and generates tailored recommendations per section
- AI chat panel powered by Anthropic API (Sonnet) for real-time Q&A
- Ember orange accent color (#e0663c), dark theme, monospace typography

### What needs to change
- **Clean separation between guide content and project customization.** Best practices / reference material should feel distinct from personalized recommendations. Right now they're interleaved.
- **Clearer navigation hierarchy.** 10 flat sidebar items is too many to scan quickly. Group or layer them so the user always knows where they are and what to do next.
- **Progressive flow.** A first-time user should feel guided through a logical sequence. A returning user should be able to jump straight to any section.

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
- Model: `claude-sonnet-4-20250514` for both

## Design principles

- **Clarity over cleverness.** Every screen should be immediately scannable. No hidden menus, no mystery icons.
- **Guide first, reference second.** New users follow a path. Experienced users browse freely. Both work without friction.
- **Personalization earns trust.** Recommendations should feel specific and useful, never generic. If the analysis can't add value to a section, show nothing rather than filler.
- **Respect the developer.** Terse, practical language. No marketing fluff. Code examples should be copy-paste ready.

## Deployment

- **Parent site:** lab401.ai (separate repo: lab401-web)
- **Apps page tile** on lab401.ai links to Forge_ at `https://forge.lab401.ai`
- Deployment pipeline TBD — will likely mirror lab401-web (S3 + CloudFront)

## Related

- **lab401-web repo:** Contains the Apps page tile and GuideCover animation for Forge_
- **Brand:** Lab_401 — AI software development company
