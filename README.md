# brainstorm-party

A Claude Code skill that turns decisions and open-ended problems into structured multi-perspective debates. Instead of generating a single answer, it assembles 2-3 expert personas with explicit biases and stakes, forces genuine disagreement, and synthesizes a recommendation only after real tension has been explored.

Pure prompt — no code, no dependencies, no API keys. Drop it in your skills directory and go.

## Why This Exists

LLMs default to giving you one confident answer. That's great for factual questions, but terrible for decisions with real tradeoffs. brainstorm-party fixes this by simulating the experience of having smart people in a room who disagree with each other — so you get the tension, the edge cases, and the "yeah but what about..." before you commit.

## Features

- **Expert panel assembly** — 2-3 personas selected for the domain, each with a distinct lens, clear bias, and personal stakes in the outcome
- **Two session modes** — Quick (2 rounds, single-pass) for gut checks; Deep (3-5 rounds, multi-agent) for high-stakes decisions
- **Genuine independence in Deep mode** — each persona runs as a separate Claude agent that literally cannot see what others are saying until the next round
- **Scope guard** — declines trivial topics and factual lookups where debate adds no value
- **Devil's advocate rule** — at least once per session, a persona must argue against their own bias when evidence demands it
- **Tension tracker** — running sidebar shows where personas agree, disagree, and what's unresolved
- **Context-aware handoffs** — no generic "what do you think?" prompts; offers specific pivots based on live tensions
- **Terminal-optimized output** — bold speaker labels, no blockquotes or italics (which render poorly in dark terminals)

## Usage

Trigger with natural language or the slash command:

```
brainstorm whether to build our own auth or use Clerk
help me decide between a monorepo and separate repos
let's think through our pricing model
/brainstorm should we hire for this or build in-house
```

### Mode Selection

The skill auto-selects based on topic complexity, or you can be explicit:

| Mode | Rounds | Architecture | Best For |
|------|--------|-------------|----------|
| **Quick** | 2 | Single-pass (all personas in one response) | Gut checks, quick tradeoffs, sanity checks |
| **Deep** | 3-5 | Multi-agent (each persona = separate agent) | Architecture decisions, strategy, high-stakes choices |

**Trigger Deep mode** by saying "deep", "thorough", "really think this through", or just tackling a topic with significant lasting consequences.

**Switch mid-session** — "let's go deeper on this" or "speed this up" to change modes on the fly.

## How It Works

### 1. Topic Clarification

If the request is vague, the skill asks 1-2 focused questions before assembling the panel. No panel is created until the problem space is clear.

### 2. Panel Assembly

Personas are designed with:
- **Name + role** — e.g., "Maya — Product Strategist"
- **Bias/philosophy** — what shapes their thinking (e.g., "ships fast and iterates")
- **Stakes** — what they're personally accountable for (e.g., "owns engagement metrics — a wrong platform bet means 6 months of declining DAU on her watch")

Stakes are what separate this from generic role-play. A persona who owns release cadence and gets burned by tech debt argues differently than one who just "prefers shipping fast."

**Example panel:**

```
Panel assembled for: "Should we build a mobile app or improve our web experience?"
Mode: Deep

- Maya — Product Strategist: Obsessed with user behavior data. Believes
  you go where the users already are. Stakes: Owns engagement metrics —
  a wrong platform bet means 6 months of declining DAU on her watch.

- Raj — Engineering Lead: Thinks about long-term maintainability. Prefers
  investing in a strong foundation over chasing platforms. Stakes: His team
  maintains whatever gets built — a hasty app means years of cross-platform
  debugging.

- Dana — Growth Lead: Focused on acquisition and reach. Judges every move
  by how it moves the needle on new users. Stakes: Quarterly growth targets —
  she needs a channel that converts, not a technical showpiece.
```

### 3. Debate Rounds

Each round follows a strict structure:

1. **Each persona speaks** (2-4 sentences, concise and opinionated)
2. **Friction is required** — at least one pushback per round; no rounds where everyone nods
3. **Tension tracker** — shows live state of disagreements
4. **Context-aware handoff** — offers specific pivots, not generic prompts

```
Maya: I've been looking at the usage data and 73% of sessions are mobile web...

Raj: Maya's numbers are compelling, but she's ignoring the maintenance cost.
     Two codebases means two bug surfaces...

Dana: You're both missing the bigger picture. Our competitors launched apps
      last quarter and app store visibility alone drove 40% of their signups...

Tensions: [native vs. PWA: unresolved] [timeline: Maya vs. Raj] [acquisition channel: Dana unopposed]
```

### 4. Devil's Advocate Moments

At least once per session, a persona argues against their own bias when evidence warrants it:

```
Maya: [against type] Look, I hate saying this, but Raj's numbers on
      maintenance cost actually support his case more than mine...
```

These moments are marked explicitly — they signal the debate is real, not theater.

### 5. Conclusion

When discussion converges (or after 3-4 productive rounds):

- **Clear winner** — panel recommendation with each member's rationale
- **Close call** — structured comparison table with champion, key argument, and main risk per option
- **Always includes** — final tension tracker (resolved + open) and a concrete next step

## Deep Mode: Multi-Agent Architecture

The key innovation. In Deep mode, each persona is spawned as a separate Claude agent, producing genuinely independent reasoning.

### Hybrid Round Strategy

**Round 1 — Parallel:** All persona agents run simultaneously. Each receives only the topic and their persona card. They cannot see each other — producing blind, independent opening positions.

**Rounds 2+ — Sequential:** Each agent runs one at a time, receiving the full conversation history. This creates real back-and-forth — Raj actually reads Maya's latest argument before responding.

The orchestrator (the main Claude instance) assembles responses, writes tension trackers, manages flow, and decides when to conclude.

### Why This Matters

In single-pass mode, all personas come from the same generation — the model inherently knows what "Maya" will say when writing "Raj." In multi-agent mode, each agent reasons in isolation. The disagreements are structurally genuine, not performatively constructed.

## Domain Adaptation

The skill adapts persona selection to the topic domain:

| Domain | Persona Style |
|--------|--------------|
| **Technical** | Genuine technical perspectives ("favors FP" vs. "pragmatic OOP") |
| **Creative** | Creative roles (art director, copywriter, audience advocate) |
| **Business** | Strategy roles with specific ownership areas |
| **Personal** | Values the user might hold ("the pragmatist", "the ambitious one") |

## Scope Guard

The skill **declines to assemble a panel** for:
- Trivially low-stakes topics with no real tradeoffs
- Pure factual lookups with a single correct answer
- Topics better served by a direct answer

It tells you why and just answers the question instead.

## File Structure

```
~/.claude/skills/brainstorm-party/
  SKILL.md      # Full skill definition (203 lines) — all rules, modes,
                # persona construction, agent prompts, edge cases
  README.md     # This file
```

## Installation

Copy the skill directory to your Claude Code skills folder:

```bash
cp -r brainstorm-party ~/.claude/skills/
```

Or clone from the repo:

```bash
git clone https://github.com/SnocmaG/brainstorm-party.git ~/.claude/skills/brainstorm-party
```

No configuration needed. The skill activates automatically when Claude Code detects brainstorming intent.

## Integration

brainstorm-party integrates with other workflows:

- **Jira** — tickets with the `needs-brainstorm` label trigger a brainstorm session before work begins. After the session, the ticket description is updated with the outcome and the label is removed.
- **Planning** — brainstorm results can feed directly into epic planning and ticket creation.
- **Mode switching** — start quick, go deep if the problem demands it, all within the same session.

## Requirements

- Claude Code CLI
- No external dependencies, APIs, or packages

## License

MIT
