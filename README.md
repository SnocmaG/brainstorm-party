# brainstorm-party

A Claude Code skill that turns decisions and open-ended problems into structured multi-perspective debates. It assembles 2-3 expert personas with explicit biases and stakes, grounds the discussion in your actual situation, and synthesizes a recommendation after authentic debate.

Pure prompt -- no code, no dependencies, no API keys. Drop it in your skills directory and go.

## Why This Exists

LLMs default to giving you one confident answer. That's great for factual questions, but terrible for decisions with real tradeoffs. brainstorm-party fixes this by simulating the experience of having smart people in a room who genuinely see the problem differently -- so you get the tension, the edge cases, and the "yeah but what about..." before you commit.

## Features

- **Context sprint** -- asks 2-3 sharp diagnostic questions before assembling the panel, grounding the entire brainstorm in your specific situation (not a generic version of your problem)
- **Candlekeep integration** -- enriches each persona with relevant knowledge from your Candlekeep library, routed per persona's domain. Purely additive -- works fine without it.
- **Expert panel assembly** -- 2-3 personas selected for the domain, each with a distinct lens, clear bias, and personal stakes in the outcome
- **Adjacent-domain persona** -- one panelist from a structurally similar but different field, bringing cross-domain frames and analogies the domain insiders wouldn't reach
- **Two session modes** -- Quick (2 rounds, single-pass) for gut checks; Deep (3-5 rounds, multi-agent) for high-stakes decisions
- **Genuine independence in Deep mode** -- each persona runs as a separate Claude agent that literally cannot see what others are saying until the next round
- **Organic debate** -- no forced disagreement or devil's advocate rules. Different expertise + different knowledge + different stakes = natural divergence
- **Assumptions to validate** -- every conclusion names the 2-3 biggest assumptions the recommendation depends on and how to check them
- **Scope guard** -- declines trivial topics and factual lookups where debate adds no value
- **Tension tracker** -- running sidebar shows where personas agree, disagree, and what's unresolved
- **Context-aware handoffs** -- no generic "what do you think?" prompts; offers specific pivots based on live tensions
- **Terminal-optimized output** -- bold speaker labels, no blockquotes or italics (which render poorly in dark terminals)

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

**Switch mid-session** -- "let's go deeper on this" or "speed this up" to change modes on the fly.

## How It Works

### 1. Context Sprint

The skill asks 2-3 specific diagnostic questions before assembling the panel. These aren't generic "tell me more" questions -- they target the facts that would change the recommendation if the answers were different.

Examples:
- "What's your current install rate, and where in the funnel do users drop off?"
- "Can your buyer approve $5K without their manager, or does it need VP sign-off?"
- "How many direct Firestore calls exist in your codebase?"

If your original message already includes rich context, or you signal you want to move fast, the sprint is skipped.

### 2. Candlekeep Enrichment

If you have a Candlekeep library, the skill searches it for domain knowledge relevant to each persona's expertise. Different personas get different knowledge slices -- a finance panelist gets economics chapters, a UX panelist gets design chapters.

If no relevant books exist, the skill proceeds normally on pure LLM knowledge. Candlekeep is a bonus, not a requirement.

### 3. Panel Assembly

Personas are designed with:
- **Name + role** -- e.g., "Maya -- Product Strategist"
- **Bias/philosophy** -- what shapes their thinking (e.g., "ships fast and iterates")
- **Stakes** -- what they're personally accountable for (e.g., "owns engagement metrics -- a wrong platform bet means 6 months of declining DAU on her watch")
- **Adjacent domain** -- one panelist from a different field that's solved a structurally similar problem

Stakes are what separate this from generic role-play. A persona who owns release cadence and gets burned by tech debt argues differently than one who just "prefers shipping fast."

### 4. Debate Rounds

Each round follows a structure:

1. **Each persona speaks** (2-4 sentences, concise and opinionated)
2. **Authentic reactions** -- panelists agree, partially agree, or push back from genuine conviction. No forced disagreement.
3. **Tension tracker** -- shows live state of debates
4. **Context-aware handoff** -- offers specific pivots, not generic prompts

```
Maya: I've been looking at the usage data and 73% of sessions are mobile web...

Raj: Maya's numbers are compelling, but she's ignoring the maintenance cost.
     Two codebases means two bug surfaces...

Dana: You're both missing the bigger picture. Our competitors launched apps
      last quarter and app store visibility alone drove 40% of their signups...

Tensions: [native vs. PWA: unresolved] [timeline: Maya vs. Raj] [acquisition channel: Dana unopposed]
```

### 5. Conclusion

When discussion converges (or after 3-4 productive rounds):

- **Clear winner** -- panel recommendation with each member's rationale
- **Close call** -- structured comparison table with champion, key argument, and main risk per option
- **Key assumptions to validate** -- the 2-3 biggest assumptions the recommendation depends on, with specific ways to check them
- **Final tension tracker** -- resolved + open debates
- **Concrete next step** -- what to do first

## Deep Mode: Multi-Agent Architecture

The key innovation. In Deep mode, each persona is spawned as a separate Claude agent (always Opus), producing genuinely independent reasoning.

### Hybrid Round Strategy

**Round 1 -- Parallel:** All persona agents run simultaneously. Each receives the topic, their persona card, situation context, and Candlekeep knowledge (if any). They cannot see each other -- producing blind, independent opening positions.

**Rounds 2+ -- Sequential:** Each agent runs one at a time, receiving the full conversation history. This creates real back-and-forth -- Raj actually reads Maya's latest argument before responding.

The orchestrator (the main Claude instance) assembles responses, writes tension trackers, manages flow, and decides when to conclude.

### Why This Matters

In single-pass mode, all personas come from the same generation -- the model inherently knows what "Maya" will say when writing "Raj." In multi-agent mode, each agent reasons in isolation. The disagreements are structurally genuine, not performatively constructed.

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
  SKILL.md      # Full skill definition -- all rules, modes,
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

## Requirements

- Claude Code CLI
- Candlekeep CLI (`ck`) -- optional, for knowledge enrichment
- No other external dependencies, APIs, or packages

## License

MIT
