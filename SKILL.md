---
name: brainstorm-party
description: This skill should be used when the user wants to brainstorm, ideate, solve a problem, or think through a decision collaboratively. It assembles a panel of 2-3 domain-savvy experts who debate, challenge each other, and work toward the best outcome through interactive rounds. Triggers on requests like "brainstorm", "let's think through", "I need ideas for", "help me decide", or any open-ended problem-solving request.
---

# Brainstorm Party

## Overview

Simulate a focused brainstorming session with 2-3 expert personas who each bring a distinct perspective. The panel members genuinely disagree, challenge assumptions, and push back on each other — they are not yes-men. The goal is to reach the best possible outcome through constructive tension, not artificial consensus.

## Scope Guard

Before starting, evaluate whether the topic actually benefits from a multi-perspective debate. **Decline to assemble a panel** for topics that are:
- Trivially low-stakes with no real tradeoffs (e.g., "what should I have for lunch")
- Pure factual lookups with a single correct answer (e.g., "what's the capital of France")
- Better served by a direct answer than a debate

If the topic doesn't warrant a panel, say so briefly and just answer the question directly. Reserve the brainstorm format for decisions, strategies, designs, tradeoffs, and genuinely open-ended problems.

## Session Modes

At the start, determine the session mode based on context. If the user doesn't specify, infer from the topic's complexity:

### Quick Mode (default for straightforward topics)
- 2 rounds, single-pass (all personas generated together)
- Best for: gut checks, quick tradeoff comparisons, sanity checks
- Lightweight — fast and low-overhead

### Deep Mode (for high-stakes or complex topics)
- 3-5 rounds using **multi-agent architecture** (see below)
- Best for: architectural decisions, strategy, anything where genuine independent reasoning matters
- Activate when: user says "deep", "thorough", "really think this through", or the topic involves significant tradeoffs with lasting consequences

The user can switch modes mid-session (e.g., "let's go deeper on this").

## Session Setup

### 1. Understand the Topic

Before assembling the panel, ensure the topic is clearly understood. If the user's request is vague or missing critical context, ask 1-2 focused clarifying questions before proceeding. Do not assemble the panel until the problem space is clear enough for productive discussion.

### 2. Assemble the Panel

Select 2-3 personas based on the topic's domain. Each member must bring a **distinct lens** — not just different titles, but fundamentally different ways of approaching the problem.

**Persona design rules:**
- Give each member a short name and a one-line role description (e.g., **Maya — Product Strategist** or **Raj — Systems Architect**)
- Each member has a clear bias or philosophy that shapes their thinking (e.g., "ships fast and iterates" vs. "builds for scale from day one")
- **Each member has stakes** — define what they're accountable for and what outcome they'd personally regret. Not just "bias: ships fast" but "owns release cadence, gets burned when tech debt slows the team." Stakes create authentic motivation, not just opinion.
- Members must represent genuinely different — sometimes opposing — viewpoints
- Keep it to 2 members for focused debates, 3 for topics that benefit from a tiebreaker or a wildcard perspective
- Choose personas that fit the domain naturally — do not force generic business roles onto creative or technical problems

### 3. Present the Panel

Introduce each member with their name, role, perspective, and **what's at stake for them** in a brief opening before the first round. Example:

**Panel assembled for: "Should we build a mobile app or improve our web experience?"**
**Mode: Deep**

- **Maya — Product Strategist**: Obsessed with user behavior data. Believes you go where the users already are. **Stakes:** Owns engagement metrics — a wrong platform bet means 6 months of declining DAU on her watch.
- **Raj — Engineering Lead**: Thinks about long-term maintainability. Prefers investing in a strong foundation over chasing platforms. **Stakes:** His team maintains whatever gets built — a hasty app means years of cross-platform debugging.
- **Dana — Growth Lead**: Focused on acquisition and reach. Judges every move by how it moves the needle on new users. **Stakes:** Quarterly growth targets — she needs a channel that converts, not a technical showpiece.

## Running the Session

### Terminal Readability Rules (MANDATORY)

All output MUST be readable in CLI terminals with dark backgrounds:
- **NEVER use blockquotes** (`>` prefix) — they render in dim/muted colors
- **NEVER use italics** (`*text*`) — they render in dim/unreadable colors
- Use **bold** (`**text**`) for emphasis instead of italics
- Use `[brackets]` for annotations like `[against type]` instead of italic parentheticals
- Speaker format: `**Name:** plain text here` (no blockquote prefix, no italics)
- Stakes format: `**Stakes:** plain text` (not italic)

### Round Structure

Each round follows this format:

1. **Each member speaks** — using bold names with plain text for terminal readability (NO blockquotes, NO italics — these render in dim/unreadable colors in CLI terminals):

**Maya:** I've been looking at the usage data and... [her position]

**Raj:** Maya's numbers are compelling, but she's ignoring the maintenance cost... [reacts and challenges]

**Dana:** You're both missing the bigger picture... [new angle]

2. **Friction is required** — at least one member must push back, disagree, or raise a concern in each round. Avoid rounds where everyone nods along. If a member agrees with a point, they should add a caveat, an edge case, or a "yes, but..." condition.

3. **Tension tracker** — after all members speak, show the current state of open debates:

```
Tensions: [build vs. buy: unresolved] [timeline: Maya vs. Raj] [scope: converging]
```

4. **Context-aware handoff** — do NOT use a generic "what do you want to do?" prompt. Instead, offer 2-3 **specific** pivots based on the tensions that just surfaced. Examples:
   - "Maya and Raj are deadlocked on build vs. buy — want to add a budget constraint to break the tie?"
   - "Dana raised a growth angle no one's addressed — should she make her full case?"
   - "The panel is converging on Option A — ready to stress-test it, or wrap up?"

### Persona Voice Rules

- Each member speaks in first person with a consistent voice and personality
- Members build on, challenge, and reference each other's points — this is a conversation, not parallel monologues
- A member who was challenged must respond to the challenge, not ignore it
- Members can change their position if genuinely persuaded, but should articulate why
- Avoid performative disagreement — friction must be substantive and rooted in the member's perspective and stakes

### The Devil's Advocate Rule

At least once per session, one persona must **argue against their own stated bias** when the evidence warrants it. If Maya is data-driven and the data actually supports Raj's position, she should say so — painfully, reluctantly, but honestly. This is the single best signal that the brainstorm isn't theater. Real experts follow evidence even when it contradicts their instincts.

Mark these moments explicitly so they stand out:

**Maya:** [against type] Look, I hate saying this, but Raj's numbers on maintenance cost actually support his case more than mine...

### Keeping Rounds Focused

- Each member's contribution per round: 2-4 sentences. Concise, punchy, opinionated.
- Do not let rounds become essays. If a topic has too many angles, split into sub-topics across rounds.
- If discussion stalls or loops, have a member explicitly call it out and propose a new angle.

## Multi-Agent Architecture (Deep Mode)

When running in Deep Mode, use the Task tool to spawn each persona as a **separate agent**. This produces genuinely independent reasoning — each agent literally cannot anticipate what the others will say.

### Hybrid Round Strategy

**Round 1 — Parallel (independent blind perspectives):**
- Spawn all persona agents simultaneously using the Task tool with `subagent_type: "general-purpose"`
- Each agent receives: the topic, their persona card (name, role, bias, stakes), and instructions to give their opening position in 2-4 sentences
- Agents run in parallel — they cannot see each other's responses
- After all complete, the orchestrator (you) assembles their responses with bold speaker names, writes the tension tracker, and presents to the user

**Rounds 2+ — Sequential (reactive conversation):**
- Spawn each persona agent one at a time
- Each agent receives: the full conversation history so far (all prior rounds), their persona card, and instructions to react to what others said
- This creates genuine back-and-forth — Raj actually reads Maya's latest argument before responding
- The orchestrator assembles each round after all agents have spoken

**Orchestrator responsibilities (you):**
- Write the tension tracker after each round
- Write the context-aware handoff prompts
- Manage the conversation flow and decide when to conclude
- Apply the devil's advocate rule by instructing one agent per session to argue against their bias when appropriate

### Agent Prompt Template

When spawning a persona agent, use a prompt like:

```
You are {name}, a {role}. {bias/philosophy description}.
Your stakes: {what you're accountable for and what outcome you'd regret}.

Topic: {the brainstorming topic}

{For Round 1}: Give your opening position on this topic in 2-4 concise, opinionated sentences. Speak in first person.

{For Round 2+}: Here is the conversation so far:
{prior rounds}

React to what the other panelists said. Reference them by name. Challenge or build on their points. 2-4 sentences, first person, concise and opinionated.

{If devil's advocate round}: If the evidence discussed so far actually supports a position contrary to your usual bias, acknowledge it honestly. Argue against your own instinct if the reasoning demands it.
```

### Fallback

If the Task tool is unavailable or the user prefers speed, fall back to single-pass mode (all personas generated in one response) for any round. The formatting, tension tracker, and devil's advocate rules still apply regardless of mode.

## Concluding the Session

When the user signals readiness to wrap up (or after 3-4 rounds of productive discussion), move to conclusion.

**If there is a clear winner:**
Present the panel's recommendation with a brief rationale from each member on why they converged (or conceded).

**If the topic is subjective or close:**
Present the top 2-3 options in a structured format:

| Option | Champion | Key Argument | Main Risk |
|--------|----------|-------------|-----------|
| Option A | Maya | ... | ... |
| Option B | Raj | ... | ... |

Then state which option has the most panel support and why, while leaving the final call to the user.

**Always end with:**
- A final tension tracker showing which debates were resolved and which remain open
- A concrete next step or action the user can take based on the outcome

## Edge Cases

- **Trivially low-stakes topics**: Do not assemble a panel. Answer directly. (See Scope Guard above.)
- **Highly technical topics**: Personas should reflect genuine technical perspectives (e.g., "favors functional programming" vs. "pragmatic OOP advocate"), not generic manager roles.
- **Creative topics**: Use creative-domain personas (e.g., art director, copywriter, audience advocate) — not business personas.
- **Personal decisions**: Be empathetic. Personas can represent different values or priorities the user might hold (e.g., "the pragmatist", "the ambitious one", "the risk-aware one") rather than external experts.
- **User provides their own panel**: If the user specifies who should be in the room, use their selections and infer perspectives from the roles given.
- **User requests mode switch**: If the user says "go deeper" mid-session, switch from Quick to Deep mode and begin spawning agents from the next round. If they say "let's speed this up", switch to Quick/single-pass.
- **Domain expertise limits**: If the topic is highly specialized and the LLM's knowledge is thin, have personas acknowledge uncertainty rather than generating confident-sounding nonsense. A persona saying "I'm not sure about the specifics here, but my concern is..." is far more valuable than fabricated expertise.
