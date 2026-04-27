# Agent coverage map (Layer 1)

This file maps **project signals** to **agent types** that are typically valuable for projects with that shape. Used by `/forge:analyze` to flag coverage gaps — workflows that exist in the project but have no specialized agent guarding them.

The point isn't to recommend every agent listed. It's to ensure that when a project has a high-frequency or high-risk workflow, *something* is auditing or guarding it.

---

## How `/forge:analyze` uses this file

1. Scan the project's actual signals (file types, scripts, dependencies, directory structure, git activity).
2. For each signal that fires, check whether an agent of the relevant type already exists in `.claude/agents/`.
3. Where a signal fires but no matching agent exists → **gap**. Recommend an agent to fill it.
4. Where the project has agents but no matching signal → **possibly orphaned**. Note but do not auto-recommend removal (the user may know something the analysis doesn't).

---

## Universal signals (apply to nearly every project)

| Signal | Recommended agent type | Why |
|---|---|---|
| Project has any code at all | **Code reviewer / pre-commit reviewer** | Catches bugs before they land. The single highest-value agent in any setup. |
| Project has tests (any framework) | **Test runner / test result interpreter** | Saves context every time a test runs. Reads pass/fail and surfaces only what matters. |
| Project has documentation files | **Docs consistency checker** | Flags drift between code and docs. Cheap, high-leverage. |

---

## Stack-specific signals

### Web frontend (React, Vue, Svelte, Next.js, etc.)
| Signal | Agent type |
|---|---|
| `package.json` has a UI framework dep | Component conventions auditor (BEM, naming, prop patterns) |
| Has a CSS file or styled-components | Style token guardrail (enforces design system) |
| Has a router config | Route convention checker |
| Has accessibility lint config | Accessibility reviewer |

### Backend / API
| Signal | Agent type |
|---|---|
| Has API route handlers | API contract reviewer (input validation, error shape, status codes) |
| Has database migrations | Migration safety reviewer (idempotency, locking, rollback) |
| Has auth middleware | Auth boundary auditor |
| Has environment variable config | Secrets exposure scanner |

### Data / ML
| Signal | Agent type |
|---|---|
| Jupyter notebooks present | Notebook hygiene reviewer (cleared outputs, no committed data) |
| Has model training scripts | Training run sanity checker (hyperparam drift, dataset version) |
| Has prompts in source files | Prompt change reviewer (model ID, structure, JSON parse safety) |

### Mobile (iOS/Android)
| Signal | Agent type |
|---|---|
| Xcode project / Swift files | Build setting reviewer |
| Android Gradle / Kotlin | Gradle dep reviewer |
| App store metadata files | Submission readiness checker |

### Infrastructure
| Signal | Agent type |
|---|---|
| Dockerfile / docker-compose | Image hygiene reviewer (size, layers, secrets) |
| Terraform / Pulumi | IaC change reviewer (state risks, drift) |
| CI/CD config (GitHub Actions, etc.) | Pipeline change reviewer |

### Plugin / extension projects
| Signal | Agent type |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest validator |
| `skills/*/SKILL.md` files | Skill structure validator |
| Schema files referenced in skills | Schema reference checker |

---

## Risk-multiplier signals

These signals don't map to a single agent but raise the bar across the board. When they fire, the universal "code reviewer" should be high-priority and there should be a *specific* guardrail for the risky pattern.

| Risk signal | What to add |
|---|---|
| Direct LLM API calls in source | Model ID / prompt drift reviewer |
| Browser-direct API access (bypasses backend) | Security boundary reviewer |
| Single monolithic file > 500 LOC | Scope sentry (flag drift / unintended growth) |
| Generated/committed lockfiles | Lockfile diff reviewer |

---

## Anti-patterns when recommending agents

When `/forge:analyze` generates agent recommendations from this map, avoid these failure modes:

1. **Don't recommend an agent that overlaps an existing one.** Read existing agent descriptions first. "Code reviewer" already covers basic correctness — don't add a near-duplicate.
2. **Don't recommend more than 4 agents in one pass.** Six new agents create more cognitive load than they solve. Pick the highest-leverage gaps.
3. **Don't recommend an agent without naming the *specific* signal that triggered it.** "You have Next.js, so a routing-conventions agent saves you from..." beats "You should add a routing agent."
4. **Don't recommend the universal agents if the project is tiny.** A 20-line script doesn't need a code reviewer agent — it needs nothing.
