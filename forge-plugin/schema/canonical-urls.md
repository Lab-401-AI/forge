# Canonical sources (Layer 2 — live fetch targets)

These are the URLs Forge skills fetch at lint/analyze time so feedback is grounded in current Anthropic guidance, not Claude's training-time snapshot.

Skills fetch one or two of these per invocation depending on what's relevant. Don't fetch all of them every time — that wastes credits.

## Tier 1 — must-have, always relevant

| Purpose | URL |
|---|---|
| Boris Cherny's best-practices essay (canonical) | https://www.anthropic.com/engineering/claude-code-best-practices |
| Claude Code memory (CLAUDE.md) reference | https://code.claude.com/docs/en/memory |
| Plugin reference (manifest, components, distribution) | https://code.claude.com/docs/en/plugins-reference |
| Skills reference (frontmatter, lifecycle) | https://code.claude.com/docs/en/skills |
| Subagents reference (frontmatter, scope, isolation) | https://code.claude.com/docs/en/sub-agents |

## Tier 2 — pull when the user's project intersects these topics

| Topic | URL |
|---|---|
| Hooks (event handlers) | https://code.claude.com/docs/en/hooks |
| MCP servers | https://code.claude.com/docs/en/mcp |
| Settings & permissions | https://code.claude.com/docs/en/settings |
| Plugin marketplaces (distribution) | https://code.claude.com/docs/en/plugin-marketplaces |
| Model selection / effort levels | https://code.claude.com/docs/en/model-config |

## How to use this in a skill

```
1. Read the user's CLAUDE.md first.
2. Decide which 1-2 canonical URLs are relevant to the question or audit.
3. Use WebFetch to pull them. Ask for the specific information you need, not the full page.
4. Compare what the user has against what the docs say *today*.
5. In the output, cite which URLs were fetched so the user can verify.
```

## Failure mode

If WebFetch fails (offline, rate-limited, redirect loop), the skill must:
1. Note that live fetch failed in the output
2. Fall back to the bundled schema (Layer 1) for structural checks
3. Skip qualitative claims that depend on current guidance — don't fabricate them from training memory
