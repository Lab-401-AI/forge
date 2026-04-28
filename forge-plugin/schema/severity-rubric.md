# Severity rubric (Layer 1)

This file defines the four severity tiers Forge_'s lint applies to every finding, the rules that map each check to a tier, and ROI guidance for low-tier findings.

The whole point: **the lint should know when to stop.** Without explicit tiers, every run produces actionable-looking suggestions even when the file is healthy. That trains users to either ignore lint output entirely or burn tokens chasing diminishing returns. Severity tiers give the lint a stopping criterion — a verdict the user can trust.

---

## The four tiers

### Critical 🔴
**Meaning:** Claude will fail or behave wrong because of this finding. Always fix.

**Examples:**
- Ghost agent reference (Claude tries to invoke an agent that doesn't exist on disk)
- Stale file path Claude will trust as real
- Credentials or secrets matched in CLAUDE.local.md
- Model ID drift between two locations (one says `claude-sonnet-4-6`, the other says something else)
- A required section is completely missing (no commands, no project description)

The user should fix Critical findings before doing anything else. These are bugs, not preferences.

### Important ⚠️
**Meaning:** Meaningfully reduces Claude's effectiveness. Worth fixing this session.

**Examples:**
- Agent description scores **Will not trigger** (0–1/4 on the rubric) — agent exists but won't fire
- Orphaned agent on disk that nothing references — Claude won't reach for it proactively
- Placeholder rot in an operational section (Commands, Conventions, API)
- File over 2,000 words (the hard adherence threshold from Anthropic's docs)
- A required section is present but skeletal (e.g., a "Commands" section that just says "TBD")

These erode Claude's effectiveness in real ways. Fix when you have time this session.

### Polish 🪞
**Meaning:** Marginal improvement, diminishing returns. Show for awareness, not as action items.

**Examples:**
- Agent description scores **Acceptable** (3/4) — fires most of the time but missing one signal
- File in the 1,000–2,000 word range (over the soft threshold but under the hard one)
- Positioning/strategy content that could move to a non-loaded file
- Undated "Currently:" notes in CLAUDE.local.md
- Duplicate content between CLAUDE.md and CLAUDE.local.md
- Recommended (not required) sections missing

The lint should show these but never lead with them. Past the first lint run on a file, polish findings should be presented as "here's what's left if you want, but you're done."

### Pass ✅
**Meaning:** Nothing Critical or Important fires. The setup is healthy. Running lint again won't change the verdict.

**Triggered when:** zero Critical findings AND zero Important findings.

(Polish findings may still exist — the bar for Pass is "no real issues," not "zero findings.")

When Pass fires, the lint output must explicitly say so and explicitly tell the user to stop running lint until they change something. This is the missing stopping criterion that prevents endless polish loops.

---

## Mapping every check to a tier

| Check | Tier |
|---|---|
| Required section completely missing | Critical 🔴 |
| Ghost agent (mentioned in CLAUDE.md, not on disk) | Critical 🔴 |
| Stale file path (referenced in CLAUDE.md, not on disk) | Critical 🔴 |
| Credentials/secrets pattern in CLAUDE.local.md | Critical 🔴 |
| Model ID drift across locations | Critical 🔴 |
| Required section is skeletal/placeholder content | Important ⚠️ |
| Agent description: Will not trigger (0–1/4) | Important ⚠️ |
| Orphaned agent (on disk, not referenced anywhere) | Important ⚠️ |
| Placeholder rot in operational section | Important ⚠️ |
| File over 2,000 words | Important ⚠️ |
| Agent description: Weak (2/4) | Important ⚠️ |
| Agent description: Acceptable (3/4) | Polish 🪞 |
| File 1,000–2,000 words | Polish 🪞 |
| Path-routing trigger fires on a file under 2,000 words | Polish 🪞 |
| Path-routing trigger fires on a file over 2,000 words | Important ⚠️ |
| Undated focus note in CLAUDE.local.md | Polish 🪞 |
| Duplicate content between CLAUDE.md and CLAUDE.local.md | Polish 🪞 |
| Positioning/strategy content in CLAUDE.md | Polish 🪞 |
| Placeholder rot in non-operational section (e.g., Deployment) | Polish 🪞 |
| Recommended (not required) section missing | Polish 🪞 |
| Duplicate model ID strings (no drift, just redundant references) | Polish 🪞 |

**Escalation rule:** if the same finding crosses a hard threshold, it escalates one tier. Example: file size of 1,200 words is Polish; at 2,100 words it becomes Important.

---

## ROI guidance for Polish findings

Polish items should be shown with an honest estimate of whether the fix pays for itself, so the user can decide whether to act:

```
- **<finding>** — saves ~<N> tokens per turn. Breaks even on the lint cost in ~<M> turns.
  ROI: <Strong | Moderate | Weak>
```

| ROI | Tokens saved per turn | Break-even |
|---|---|---|
| Strong | >300 | <15 turns |
| Moderate | 100–300 | 15–40 turns |
| Weak | <100 | 40+ turns or never |

**Don't recommend Weak ROI cuts** unless the user asks for an exhaustive list. They cost more attention than they save.

---

## How the lint uses this rubric

1. Run all the existing checks (structural, noise, agent quality, path-routing).
2. For each finding, look up its tier in the mapping table above.
3. Compute the verdict:
   - Any Critical → **Action Needed (Critical)**
   - No Critical, any Important → **Action Needed**
   - No Critical, no Important → **Pass**
4. Output sections in tier order. Skip empty sections entirely.
5. If Pass: the verdict line and the suggested next step must explicitly tell the user not to keep running lint.
6. For Polish findings, include the ROI estimate using the table above.
7. The "Suggested next step" section always leads with the highest-tier finding. If Pass, the step is "no action needed — your setup is healthy."

---

## Failure modes the lint must avoid

1. **Don't manufacture severity to look thorough.** A clean file should produce a Pass verdict, not a list of barely-applicable Polish items dressed up as Important.
2. **Don't gate Pass on perfection.** Polish findings can coexist with Pass — that's the design.
3. **Don't escalate Polish to Important to drive action.** If the temptation is "the user might not act unless I tier this higher," the right call is to leave it Polish and trust them. The rubric is a contract.
4. **Don't suggest Weak-ROI Polish cuts as if they were free.** Every recommendation costs the user attention. The lint should be miserly with its recommendations, not generous.
