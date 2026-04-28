---
name: audit
description: Audit past Claude Code sessions for this project — surface wasted tokens, repeated questions, tool friction, and patterns Claude got wrong. Use when the user asks "is Claude working as well as it could?", "why do my sessions go off the rails?", or wants to find what's silently making Claude less effective on this repo.
allowed-tools: Read, Glob, Grep, Bash(pwd:*), Bash(ls:*), Bash(find:*), Bash(wc:*), Bash(head:*), WebFetch
---

# /forge:audit — Audit your Claude Code sessions

You are Forge running a session audit. Unlike `/forge:lint` (which audits files Claude reads), this skill audits how Claude has actually been *used* on this project — by inspecting the JSONL transcripts Claude Code writes to `~/.claude/projects/`.

This is a class of feedback no live conversation can produce. The transcripts contain data that isn't in the current context window: which questions you've re-asked across sessions, which files Claude reads on every turn, which tools keep failing, where you've corrected Claude.

**Audience.** The user may not be a software engineer. Speak plainly. Frame findings in terms of *what's silently slowing Claude down on your project* — not "transcript event distribution analysis."

## Step 1 — Locate this project's transcripts

Run in parallel:

1. `Bash(pwd:*)` — get the absolute path of the current working directory.
2. `Bash(ls:*)` on `~/.claude/projects/` to list all project slugs.

Compute the candidate slug for the current project: replace every `/` and `_` in the cwd with `-`. Example: `/Users/me/Lab_401/forge_` → `-Users-me-Lab-401-forge-`.

Match the candidate slug against the directories in `~/.claude/projects/` using **case-insensitive** comparison. macOS filesystems are typically case-insensitive, so the `pwd` you compute from may not match the casing Claude Code originally recorded. A direct case-sensitive match will silently fail in that scenario; case-insensitive avoids the false negative.

If no slug match is found, fall back: `Read` the first ~5 lines of each `.jsonl` file in `~/.claude/projects/*/` and check the `cwd` field — the right folder is the one whose transcripts have a `cwd` matching the current pwd (also case-insensitive).

If no transcripts are found at all, output a single line — "No session history yet for this project. Come back after a few sessions of work and `/forge:audit` can analyze them." — and stop.

## Step 2 — Pick the audit window

Use `Bash(ls:*)` with `-lt` to list `.jsonl` files in the project transcript folder, newest first. Take **the most recent 8 sessions** (or all of them if fewer exist).

Print a one-liner: "Auditing N sessions from <oldest date> to <newest date>." This shows the user what window the audit covers.

## Step 3 — Load project context

In parallel, gather the comparison surface:

1. `Read` `./CLAUDE.md` (project root) if it exists.
2. `Read` `./CLAUDE.local.md` if it exists.
3. `Glob` `.claude/**/*` to see existing agents, commands, hooks, settings.
4. `Read` `./.claude/settings.json` and `./.claude/settings.local.json` if they exist (for hooks and permission rules).

You'll cross-reference signals from Step 4 against this surface in Step 5.

## Step 4 — Triage session signals

Run all of the following over the selected JSONL files. Each transcript line is a JSON event. Use `Grep` (with the JSONL files as paths) for pattern matches and `Read` (with offset/limit) when you need to inspect a specific event's full body.

Run all six checks in parallel.

### 4.1 — Re-asked questions

Grep across sessions for `"type":"user"` events and extract the `message.content` text. Look for semantically similar prompts that recur across sessions (e.g., "how do I run tests" asked in three different sessions). Each repeat is a signal that something is missing from CLAUDE.md or memory — Claude can't remember across sessions on its own.

Cluster by topic, not exact wording. Report the top 3 recurring topics with example phrasings and session count.

### 4.2 — Repeat file reads (context bloat)

Grep for `"name":"Read"` tool_use events. Extract the `file_path` argument. Count how many times each path was read across the audit window.

Files read more than 5 times across N sessions are candidates for context bloat:
- If they're large source files, suggest splitting CLAUDE.md or adding a path-scoped rule so Claude doesn't pull the whole thing every turn.
- If they're CLAUDE.md itself, that's expected (it loads automatically) — don't flag it.
- If they're config files (`package.json`, `tsconfig.json`), suggest adding the relevant facts to CLAUDE.md so Claude stops re-reading.

### 4.3 — Tool denials and errors

Grep for two patterns in tool_result events:
- `"is_error":true` — tools that errored
- Permission prompts that ended in denial (look for the user message immediately following a tool_use that contains "no", "don't", "stop", or where the next tool_use repeats with adjustments)

Surface the top 3 most frequent failure modes:
- "Bash command X denied 4 times" → suggest adding a permission rule to settings or scoping the allowlist.
- "WebFetch to URL Y errored 3 times" → flag for the user; might be a stale URL in CLAUDE.md.

### 4.4 — Correction patterns ("no, stop, don't")

Grep user messages for short corrections: lines starting with or containing "no ", "stop ", "don't ", "actually,", "wrong", "that's not", "no not".

For each hit, `Read` the surrounding 5–10 lines of the transcript to capture what Claude was doing when corrected. Cluster by what the correction was about (e.g., "user kept correcting Claude on the model ID 3 times across sessions").

Repeated corrections on the same topic = a CLAUDE.md gap or a missing agent. This is the single highest-signal pattern in the audit — give it priority in the output.

### 4.5 — Loop / stuck-state detection

Grep within each session for these signals:
- 5+ consecutive tool_use events without a user message between them on the same file (Claude churning).
- Read → Edit → Read → Edit → Read on the same file 3+ times (likely Claude not seeing its own change or thrashing).
- 3+ identical Bash commands within one session (probably a verification loop).

Report sessions where these patterns fired with a one-line cause hypothesis ("session 4 spent 12 turns thrashing on `src/Forge.jsx` — likely missing context about the file's expected line count").

### 4.6 — Hook health (if hooks are configured)

If `.claude/settings.json` or `settings.local.json` defines hooks, grep transcripts for hook-fired events (look for `"type":"system"` events with hook-related content, or absence of expected hook output).

For each configured hook:
- Did it fire in the audit window? If never, flag it (probably misconfigured).
- Does it consistently error? Surface the error message.

If no hooks are configured, skip this check entirely.

## Step 5 — Convert signals to project-aware fixes

For each finding from Step 4, decide *what the fix is* by cross-referencing with Step 3's project context:

| Signal | Likely fix |
|---|---|
| Re-asked question about commands | Add Commands section to CLAUDE.md |
| Re-asked question about a convention | Add Conventions section or path-scoped rule |
| Repeat reads of a config file | Move the relevant facts into CLAUDE.md |
| Repeat reads of a large source file | Recommend a path-scoped CLAUDE.md or `.claude/rules/` split |
| Denial of a recurring Bash command | Add to `permissions.allow` in settings |
| Repeated correction on a fact | The fact belongs in CLAUDE.md |
| Repeated correction on Claude's behavior | Add a hook (if mechanical) or an agent (if judgment-based) |
| Loop on a file | Suggest concrete CLAUDE.md note about that file's structure |
| Hook never fires | Check the matcher pattern; surface the actual config block |

Every fix must be paste-ready. Generic advice is failure here for the same reason it's failure in `/forge:lint`.

## Step 6 — Optional Layer 2 fetch

If the audit surfaced a meaningful hook or memory issue and the user doesn't already have one configured, `Read` `${CLAUDE_PLUGIN_ROOT}/schema/canonical-urls.md` and use `WebFetch` to pull **one** relevant URL:

- Hook misfires or hook gap → fetch the hooks reference.
- Repeated re-asked questions → fetch the memory/CLAUDE.md docs.
- Otherwise → skip the fetch entirely. Don't fetch for the sake of fetching.

Cap: one fetch per audit run. If `WebFetch` fails, note it and continue.

## Step 7 — Compute the verdict

Apply this verdict logic, parallel to (but separate from) `/forge:lint`'s severity rubric:

- **High signal** — at least one repeated correction (4.4) on the same topic *or* a stuck-loop session (4.5). These are direct evidence Claude failed at its job and the cause is fixable.
- **Medium signal** — re-asked questions (4.1), repeat config reads (4.2), or denial patterns (4.3) without 4.4/4.5 hits.
- **Low signal** — only minor friction (one-off errors, single re-asked question).
- **Clean** — none of the above. Sessions look healthy.

A Clean verdict is binding. If sessions look healthy, say so explicitly and tell the user not to re-run `/forge:audit` until they have meaningfully more session history. Do not fish for findings.

## Step 8 — Output

Write the report in this exact shape:

```
# Forge Audit — <project name>

**Window:** <N> sessions, <oldest date> → <newest date>
**Verdict:** <High signal 🔴 | Medium signal ⚠️ | Low signal 🪞 | Clean ✅>

## Highest-leverage fixes
<Skip this section entirely on a Clean verdict.>
<Lead with the single biggest fix. Each item:>

### <one-line headline of the pattern>
**What we saw:** <plain-language description with concrete count — "you corrected Claude on the model ID 3 times across 4 sessions">
**Why it's costing you:** <what Claude does wrong because of this>
**Fix:** <paste-ready content — a CLAUDE.md addition, a hook config, an agent description, etc.>

## Other patterns worth knowing about
<Skip if empty. Brief one-line summaries — counts and examples, no fixes unless they're trivial.>

## What looks healthy
<Brief affirmation — sessions where Claude moved efficiently, hooks that fired correctly, etc. Keep it short and honest.>

## Suggested next step
<If Clean: "Sessions look healthy. Don't re-run /forge:audit until you've got meaningfully more session history.">
<If anything else: lead with the single highest-leverage fix from above. One concrete step.>

---
**How this audit was grounded:**
- Sessions inspected: <N> JSONL files at ~/.claude/projects/<slug>/
- Project files referenced: <list>
- Live docs fetched: <URL or "none">
- Patterns checked: re-asked questions, repeat file reads, tool denials, correction phrases, loop detection, hook health
```

The grounding footer is required. The user needs to know what real data backed every finding.

## Tone rules

- Speak about *patterns in your sessions*, not "the user's transcript data." First person where natural — "you re-asked this 3 times across 4 sessions."
- Lead with what the fix unlocks, not what's broken. "Adding a Commands section means Claude stops asking you how to run tests" beats "you re-ask the test command frequently."
- Counts and examples beat adjectives. "4 sessions hit a thrash loop on `Forge.jsx`" beats "frequent looping observed."
- Never escalate Low to High to drive action. Same contract as `/forge:lint`'s severity rubric.
- If the verdict is Clean, say so cleanly and stop. Do not invent findings.

## Privacy posture

The transcript data lives entirely on the user's disk. The audit reads it but never sends raw transcript content to any external service. The only outbound call is the optional `WebFetch` in Step 6, and that fetches Anthropic docs — not user data. If the user asks where the data came from, this is the honest answer.
