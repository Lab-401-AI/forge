# Community corpus (Layer 3 — examples in the wild)

Real-world CLAUDE.md files and subagent definitions Forge can reference when suggesting what "good" looks like for a given stack. These are validation datasets, not authority. They drift from the spec.

## Subagent collections

| Repo | Why useful |
|---|---|
| `github.com/VoltAgent/awesome-claude-code-subagents` | 100+ subagent definitions across many domains. Best source for "what subagents have other people built for X stack?" |
| `github.com/wshobson/agents` | Large agent collection, often more opinionated. Good for cross-checking patterns. |

## Plugin / setup examples

| Repo | Why useful |
|---|---|
| `github.com/ComposioHQ/awesome-claude-plugins` | Curated plugin index — useful when suggesting existing plugins instead of writing one. |
| `github.com/anthropics/anthropic-cookbook` | Reference implementations from Anthropic. |

## How to use this in a skill

When the user asks "what agents should I add for a Next.js + Drizzle project?", Forge can:

1. Detect the stack from the user's `package.json` and project files.
2. Reference (don't fetch wholesale) one of these repos as the example dataset.
3. Use `WebFetch` to pull a specific known-good example only when needed for direct comparison.
4. Always be explicit in the output: "this pattern is common in `awesome-claude-code-subagents`" — the user should know whether a suggestion comes from canonical docs or community examples.

## What NOT to do

- Do not present community examples as authoritative. They drift, they're sometimes wrong, and they're sometimes outdated relative to current Claude Code spec.
- Do not fetch a full repo index unprompted. The corpus is a reference of where to look, not a thing to download every invocation.
