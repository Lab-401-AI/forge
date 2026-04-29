# Forge_

Get more out of Claude Code without needing to be an expert. Forge_ guides you through the setup, analyzes your project, and tells you what to add or change so your AI actually works the way you intended.

The core promise: **you have the vision, Forge_ makes sure your AI delivers it.**

## Two products, one brand

Forge_ is two complementary tools that serve the same user at different moments.

### Web app — [forge.lab401.ai](https://forge.lab401.ai)

Open it in a browser, describe what you're building, and Forge analyzes your project and gives you personalized recommendations. It's where you go to get started, understand your options, and pick a starting point. Visual, guided, approachable.

Source lives in [`src/`](./src). See [`src/CLAUDE.md`](./src/CLAUDE.md) for component conventions.

### CLI plugin — `forge-plugin/`

Lives inside Claude Code itself. Once you're working on a project, you call `/forge:lint`, `/forge:analyze`, `/forge:consult`, or `/forge:audit` without leaving the terminal. It reads your actual project files and tells you what's off, what's missing, and what to do next.

Full plugin docs (commands, install, architecture) in [`forge-plugin/README.md`](./forge-plugin/README.md).

**Install:**

```
/plugin marketplace add lab-401-ai/forge
/plugin install forge@lab401
```

These ship independently — the web app deploys to S3 + CloudFront, and the plugin publishes through the Lab_401 marketplace.

## Local development

```bash
npm install
npm run dev        # web app at http://localhost:5173
npm run build      # production build → dist/
```

The web app expects `VITE_ANTHROPIC_API_KEY` in a local `.env` file (gitignored).

For the plugin, load it locally with:

```bash
claude --plugin-dir "$(pwd)/forge-plugin"
```

## Tech stack

- React 18 + Vite 6 (web app)
- Plain JavaScript (no TypeScript)
- Claude Code skills as Markdown (plugin)
- Anthropic API via `claude-sonnet-4-6`

## Repo layout

```
src/                # Web app — React SPA
forge-plugin/       # CLI plugin — skills, schema, manifest
.claude/agents/     # Subagents used during development of this repo
public/             # Static assets for the web app
```

## License

[MIT](./LICENSE)

## Lab_401

Forge_ is a product of [Lab_401](https://lab401.ai), an AI software development company.
