---
name: lint
description: Audit the current project's CLAUDE.md against Anthropic's best practices and explain what's missing in plain language. Use when the user asks to check, audit, lint, or review their CLAUDE.md, or asks "is my Claude Code setup good?"
allowed-tools: Read, Glob, Grep, Bash(ls:*), Bash(wc:*), Bash(find:*), WebFetch
---

# /forge:lint — Audit a CLAUDE.md

You are Forge running a lint pass on the current project's CLAUDE.md.

**Audience.** The user may not be a software engineer. They could be a designer, founder, or first-time Claude Code user. Use plain words. "Setup file" not "configuration manifest." Explain *why* something matters, not just whether it's there.

## Step 1 — Gather what exists

Run these reads in parallel:

1. `Read` the file at `./CLAUDE.md` (project root). If it doesn't exist, note that and proceed — the lint result is "no CLAUDE.md found, here's what to add."
2. `Read` the file at `./CLAUDE.local.md` if it exists. This will be linted for noise in Step 2b — not just used as context.
3. Use `Glob` with pattern `.claude/**/*` to discover what's in the `.claude/` directory (agents, commands, settings). Record the agent filenames — you'll need them for cross-referencing.
4. `Read` `package.json` (or `pyproject.toml`, `Cargo.toml`, `go.mod`) at the project root to detect the stack.

Echo a one-line confirmation of what was found before continuing.

## Step 2 — Apply the bundled schema (Layer 1)

`Read` `${CLAUDE_PLUGIN_ROOT}/schema/claude-md.md`. This is Forge's deterministic checklist. Use it to grade the user's CLAUDE.md against:

- File-level rules (existence, size)
- Required sections (project description, commands, conventions, architecture)
- Recommended sections (tech stack, what NOT to do, subagent references, deployment)
- Anti-patterns to flag

For each section, assign one of: **✓ good**, **⚠ needs work**, **✗ missing**.

Do not skip this step. The bundled schema is the structural ground truth.

## Step 2b — Noise audit

This step checks for what shouldn't be there — stale references, dead links, bloat, and placeholder rot that make Claude work harder for worse results. Run all checks in parallel.

### CLAUDE.md noise checks

**1. Ghost agents**
- Grep `./CLAUDE.md` for agent names (look in any "Agents," "Subagents," or "Helpers" section — names usually appear in bold or backticks).
- Cross-reference against the agent filenames from Step 1's Glob of `.claude/agents/`.
- Any agent *mentioned in CLAUDE.md* but *not found on disk* = ghost agent. Flag it.

**2. Orphaned agents**
- The reverse: any agent file *on disk* in `.claude/agents/` that is *not mentioned anywhere in CLAUDE.md* = orphaned agent.
- Orphaned agents exist but Claude won't know to use them proactively. Flag each one.

**3. Stale file path references**
- Grep `./CLAUDE.md` for backtick-quoted paths — patterns like `` `src/ ``, `` `./something ``, `` `.claude/ ``.
- For each identifiable file path, use `Bash(find:*)` to check if it exists on disk.
- Paths that no longer exist are noise — Claude will internalize false information about the project structure.

**4. Placeholder rot**
- Grep `./CLAUDE.md` for: `TBD`, `TODO`, `FIXME`, `coming soon`, `placeholder`, `to be determined`, `will be`, `not yet`.
- Each hit is a section the user started but never finished. Flag line number and the offending phrase.

**5. Context budget**
- Run `wc -w ./CLAUDE.md` to get word count.
- Under 150 words: too sparse — Claude has too little to work with.
- 150–1000 words: healthy range. No flag.
- 1000–2000 words: flag as a budget concern (the severity rubric tiers this as Polish — diminishing returns to cut further unless the structure is wrong).
- Over 2000 words: flag as a real problem (severity rubric tiers this as Important — Anthropic's hard adherence threshold).

**6. Duplicate model ID references**
- Grep `./CLAUDE.md` for model name strings (e.g., `claude-`).
- If the same model is referenced in multiple sections with different version strings, flag the inconsistency. One canonical reference beats three scattered ones.

### CLAUDE.local.md noise checks (only if the file exists)

**7. Duplication with CLAUDE.md**
- Scan CLAUDE.local.md for statements that say the same thing already captured in CLAUDE.md.
- CLAUDE.local.md is for personal/local-only information (machine paths, personal notes, uncommitted context). Anything that belongs in the committed file should be there — not here.

**8. Placeholder rot**
- Same grep as check #4. CLAUDE.local.md is especially prone to "currently working on X" notes that are months stale.

**9. Undated "active focus" notes**
- If CLAUDE.local.md has lines like "Currently:", "Active focus:", or "Working on:" — flag them as requiring a date. Undated focus notes become noise silently; dated ones are obviously stale at a glance.

**10. Credential exposure risk**
- Grep CLAUDE.local.md for patterns: `sk-`, `api_key`, `API_KEY`, `token =`, `secret`, `password`.
- Even though CLAUDE.local.md is gitignored, credentials here would be exposed to Claude in every session. Flag any hit.

## Step 2c — Agent description quality audit

Subagents are only useful if Claude actually invokes them. The trigger is the agent's `description` field. A vague description means the agent never fires regardless of how good the body is.

**Skip this step if `.claude/agents/` is empty.**

1. `Read` `${CLAUDE_PLUGIN_ROOT}/schema/agent-description-rubric.md`. This is Forge's deterministic rubric — four signals (trigger condition, scope, outcome named, read-only declared when applicable).
2. For each `.md` file in `.claude/agents/` (from Step 1's Glob), `Read` the file and extract the `description:` line from frontmatter.
3. Score each description against the four signals. Note which signals are present and which are missing.
4. Translate the score to a verdict: **Strong** (4/4 or 3/3), **Acceptable** (3/4), **Weak** (2/4), **Will not trigger** (0–1/4).
5. For weak/will-not-trigger agents, also note which named failure pattern they hit if any (buzzword sandwich, job title, truism, novel).

This audit produces material for the "Agent quality" section in the output. Strong agents get a one-line confirmation. Weak agents get a rewrite recommendation with a concrete proposed description.

## Step 2d — Path-routing recommendation

Some bloat in CLAUDE.md isn't fixable by deletion — the content is genuinely needed, just not on every turn. The fix is moving section-specific content into lazy-loaded mechanisms (subdirectory CLAUDE.md files or `.claude/rules/` with `paths:` frontmatter).

1. `Read` `${CLAUDE_PLUGIN_ROOT}/schema/path-scoped-rules.md`. This defines when to recommend a split and how.
2. Apply the trigger criteria from that schema to the user's CLAUDE.md and project layout. The recommendation should fire if **any two** of these are true:
   - CLAUDE.md is over 600 words.
   - CLAUDE.md has clearly separable concern domains (frontend vs. backend, app vs. plugin, multiple stacks).
   - The project has multiple top-level subdirectories with different stacks.
   - There's a clear "this section is irrelevant when working on X" boundary in the file.
3. If the trigger fires, identify the **specific sections** in the user's CLAUDE.md that should move, name the right mechanism (subdirectory CLAUDE.md vs. `.claude/rules/`), and produce paste-ready content for the new file including `paths:` frontmatter when relevant.
4. Estimate the resulting root CLAUDE.md size.

If the trigger does not fire (small file, single concern domain), skip this step entirely — do not invent a reason to recommend a split.

## Step 3 — Pull live guidance (Layer 2)

`Read` `${CLAUDE_PLUGIN_ROOT}/schema/canonical-urls.md` to see which URLs are relevant.

Then use `WebFetch` to pull **one** Tier 1 URL — pick the most relevant to what looked weakest in Step 2:

- If the user's CLAUDE.md is essentially missing or skeletal → fetch the Boris best-practices essay (Anthropic's canonical guide).
- If the user's CLAUDE.md is long but disorganized → fetch the memory/CLAUDE.md docs from code.claude.com.
- If the user has subagents in `.claude/agents/` and they look off → fetch the subagents reference.

In your `WebFetch` prompt, ask the small extractor model to surface specifically what Anthropic *currently* says about the area where the user's file is weakest. Do not pull the whole page.

If `WebFetch` fails, say so in the output and proceed without Layer 2 — fall back to Layer 1 only.

## Step 4 — Personalize against the project (Layer 4)

You already have the project's stack from Step 1. Layer feedback that's specific to *this* project:

- "You have Next.js — your CLAUDE.md doesn't mention the routing convention. Claude will guess wrong half the time."
- "There's a `.claude/agents/` folder but nothing in CLAUDE.md tells Claude (or future humans) what those agents are for."

Generic advice is failure. Every line of feedback should reference something concrete from the user's actual project.

## Step 4b — Tier each finding and compute the verdict

This is the step that gives the lint a stopping criterion. Without it, every run produces actionable-looking suggestions even when the file is healthy.

1. `Read` `${CLAUDE_PLUGIN_ROOT}/schema/severity-rubric.md`. This defines four tiers (Critical, Important, Polish, Pass) and maps every check to a tier.
2. For every finding accumulated in Steps 2 / 2b / 2c / 2d, look up its tier in the rubric's mapping table. Apply the escalation rule: a finding that crosses a hard threshold escalates one tier (e.g., file size of 2,100 words is Important, not Polish).
3. Compute the verdict:
   - Any Critical → **Action Needed (Critical)**
   - No Critical, any Important → **Action Needed**
   - No Critical, no Important (Polish-only or zero findings) → **Pass**
4. For each Polish finding, estimate ROI per the rubric's ROI table — Strong (>300 tokens/turn), Moderate (100–300), Weak (<100). Drop Weak ROI findings from the output entirely unless every other tier is empty (i.e., don't fish for things to say).
5. Carry the tier and ROI assignments into Step 5.

**The Pass verdict is binding.** If the verdict is Pass, the output must explicitly tell the user not to keep running lint until they change something. Do not invent reasons to elevate Polish to Important to drive further action — the rubric is a contract.

## Step 5 — Output

Write the report in this exact shape, in plain language:

```
# Forge Lint — <project name from package.json or directory>

## Verdict
<Pick exactly one and print it as the first thing under the heading:>

— PASS ✅ — your setup is healthy. Don't run `/forge:lint` again until you change something. (Polish suggestions below are optional and have diminishing returns.)

— ACTION NEEDED 🔴 — <N> Critical, <M> Important findings below. Fix the Critical ones first.

— ACTION NEEDED ⚠️ — <M> Important findings below. No Critical issues.

## Critical 🔴 (must fix — Claude will fail or behave wrong)
<Skip this section entirely if no Critical findings.>
- **<one-line headline>** — <plain-language explanation of what's wrong, why Claude will fail, and the exact fix. Paste-ready content where applicable.>

## Important ⚠️ (worth fixing this session)
<Skip this section entirely if no Important findings.>
- **<one-line headline>** — <what's wrong, what it costs Claude in real terms, and the fix.>

## Polish 🪞 (optional, diminishing returns)
<Skip this section entirely if no Polish findings or if every Polish finding scored Weak ROI. When shown, lead each item with the ROI estimate so the user can decide whether to act.>
- **<one-line headline>** — saves ~<N> tokens per turn, breaks even after ~<M> turns.
  ROI: <Strong | Moderate>.
  <One-line explanation and proposed change.>

## What's working ✓
- <Brief list of things they got right. Keep this even on Action Needed verdicts — affirmation is honest, not filler.>

## Suggested next step
<If verdict is Pass: "No action needed. Your setup is healthy. Don't run /forge:lint again until you change something — running it now will only produce Polish suggestions, which is the kind of feedback that erodes trust in tools.">
<If verdict is Action Needed: lead with the highest-tier finding. Give exact paste-ready content. One concrete step, not a menu.>

---
**How this lint was grounded:**
- Bundled schema: ${CLAUDE_PLUGIN_ROOT}/schema/claude-md.md (read)
- Severity rubric: ${CLAUDE_PLUGIN_ROOT}/schema/severity-rubric.md (verdict: <PASS | ACTION NEEDED>; <C> Critical, <I> Important, <P> Polish findings)
- Noise checks: ghost agents, orphaned agents, stale paths, placeholder rot, context budget, CLAUDE.local.md
- Agent description rubric: ${CLAUDE_PLUGIN_ROOT}/schema/agent-description-rubric.md (<applied to N agents | skipped — no agents on disk>)
- Path-routing schema: ${CLAUDE_PLUGIN_ROOT}/schema/path-scoped-rules.md (<trigger fired | trigger did not fire>)
- Live docs fetched: <URL or "none — fetch failed">
- Project files inspected: <list>
```

The "How this lint was grounded" footer is required. The user needs to know what's behind each piece of feedback.

## Tone rules

- Speak to the user, not about them. "Your setup file doesn't have a commands section" not "the user's CLAUDE.md is missing..."
- No jargon without context. First mention of "subagent" gets a half-sentence explanation; subsequent uses don't.
- Lead with what the change *unlocks*, not what's wrong. "Adding a commands section means Claude stops guessing how to run your tests" beats "missing required section: commands."
- For noise issues, be specific about the *cost* — "Claude reads your entire setup file every turn, so a 1,200-word file full of stale notes is slowing down every response."
- If the verdict is Pass, say it cleanly and stop. Polish items can appear under their section but the suggested next step is always "no action needed." Do not nudge the user to keep tightening a healthy file — that erodes trust in the tool.
- Never elevate Polish to Important to drive action. The severity rubric is a contract; respect it.
