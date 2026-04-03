---
name: brainstorm-party
description: This skill should be used when the user wants to brainstorm, ideate, solve a problem, or think through a decision collaboratively. It assembles a panel of 2-3 domain-savvy experts who debate, challenge each other, and work toward the best outcome through interactive rounds. Triggers on requests like "brainstorm", "let's think through", "I need ideas for", "help me decide", or any open-ended problem-solving request.
---

# Brainstorm Party

## Overview

Simulate a focused brainstorming session with 2-3 expert personas who each bring a distinct perspective. Each persona is a real expert with their own knowledge, stakes, and way of seeing the world. When you put genuinely different people in a room with the same question, they naturally arrive at different conclusions -- and the best ideas emerge from that authentic conversation, not from manufactured friction.

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
- Lightweight -- fast and low-overhead

### Deep Mode (for high-stakes or complex topics)
- 3-5 rounds using **multi-agent architecture** (see below)
- Best for: architectural decisions, strategy, anything where genuine independent reasoning matters
- Activate when: user says "deep", "thorough", "really think this through", or the topic involves significant tradeoffs with lasting consequences

The user can switch modes mid-session (e.g., "let's go deeper on this").

## Session Setup

### 1. Context Sprint

Before assembling the panel, run a short diagnostic to ground the brainstorm in the user's actual situation. This is the single highest-leverage step in the entire process -- panelists arguing about your specific reality produce dramatically better advice than panelists arguing about a generic version of your problem.

**How it works:**

1. Read the user's question and identify 2-3 **specific, diagnostic questions** that would change the recommendation if the answers were different. These are not generic "tell me more" questions -- they target the decision-relevant facts that separate one right answer from another.

Examples of good diagnostic questions:
- "What's your current install rate, and where in the funnel do users drop off?" (not "tell me about your funnel")
- "How many direct Firestore calls exist in your codebase?" (not "how complex is your app?")
- "Can your buyer approve $5K without their manager, or does it need VP sign-off?" (not "tell me about your buyers")

2. Ask the user these 2-3 questions. Keep it tight -- don't interrogate. If the user doesn't know an answer, that's fine and itself is useful context ("we haven't measured this yet").

3. Include the user's answers in every panelist's prompt as **Situation Context**. This ensures every panelist argues about the same grounded reality.

**When to skip:** If the user's original message already includes rich context (specific numbers, constraints, prior attempts), or if they say "just brainstorm" / signal they want to move fast, skip the questions and proceed directly.

### 2. Candlekeep Knowledge Enrichment (Additive)

Before assembling the panel, enrich each persona with relevant domain knowledge from the user's Candlekeep library. This is purely additive -- if nothing relevant exists, skip it and proceed as normal.

**Flow:**

1. After deciding on the panel personas (names, roles, domains), spawn one `candlekeep-cloud:item-reader` subagent **per persona**, each with a research query tailored to that persona's specific domain and expertise. Run all in parallel:

```
Agent tool call (per persona, all in parallel):
subagent_type: "candlekeep-cloud:item-reader"
prompt: "Search the user's library for guidance relevant to: {persona's domain focus}.
RESEARCH_INTENT: Find best practices, patterns, and domain knowledge for: {what this persona would need to know about the brainstorm topic, given their role and expertise}.
Focus on actionable guidance."
run_in_background: true
```

2. Continue assembling the panel presentation while the readers run in the background.
3. When readers return, include their findings as supplementary context in each agent's prompt (see Agent Prompt Template below).
4. If a reader returns nothing relevant for a persona, that persona runs on pure LLM knowledge -- no problem.

**Important rules:**
- If nothing comes back from Candlekeep, agents run on pure LLM knowledge -- exactly the same as before this integration existed.
- Knowledge is framed as "additional domain context" in the prompt, not as the sole source of truth. Agents should use it to inform their reasoning alongside their own expertise.
- Each persona's Candlekeep query is different because each persona has a different domain focus. A finance-focused panelist's reader searches for economics content; a UX panelist's reader searches for design content. This naturally results in different knowledge slices per persona.

### 3. Assemble the Panel

Select 2-3 personas based on the topic's domain. Each member must bring a **distinct lens** -- not just different titles, but fundamentally different ways of approaching the problem.

**Persona design rules:**
- Give each member a short name and a one-line role description (e.g., **Maya -- Product Strategist** or **Raj -- Systems Architect**)
- Each member has a clear bias or philosophy that shapes their thinking (e.g., "ships fast and iterates" vs. "builds for scale from day one")
- **Each member has stakes** -- define what they're accountable for and what outcome they'd personally regret. Not just "bias: ships fast" but "owns release cadence, gets burned when tech debt slows the team." Stakes create authentic motivation, not just opinion.
- Members must represent genuinely different -- sometimes opposing -- viewpoints
- Keep it to 2 members for focused debates, 3 for topics that benefit from a tiebreaker or a wildcard perspective
- Choose personas that fit the domain naturally -- do not force generic business roles onto creative or technical problems

**The adjacent-domain rule:** When assembling a 3-person panel, one persona should come from an **adjacent but not obvious** domain -- someone whose expertise creates genuinely different mental models, not just different opinions within the same field. The reason: when all panelists share the same professional background, they converge on the same playbook. A panel of three mobile growth people discussing install rates will produce solid conventional advice. But swap one for a behavioral economist, a casino game designer, or a retail conversion specialist, and you get frames and analogies the domain insiders would never reach on their own.

How to pick the adjacent persona:
- Ask: "What other field has solved a structurally similar problem?" A push notification strategy shares mechanics with email marketing, casino player retention, and subscription renewal. An API design debate shares mechanics with language design and contract law.
- The adjacent persona must still have genuine stakes in the outcome -- they're not a tourist. They bring transferable expertise that applies to this specific problem.
- Don't go so far afield that the persona can't contribute concretely. A philosopher on a database migration panel is too abstract. A distributed systems engineer from a different industry is adjacent and useful.

### 4. Present the Panel

Introduce each member with their name, role, perspective, and **what's at stake for them** in a brief opening before the first round. If Candlekeep knowledge was retrieved, note which domain knowledge each panelist was given. Example:

**Panel assembled for: "Should we build a mobile app or improve our web experience?"**
**Mode: Deep**

- **Maya -- Product Strategist**: Obsessed with user behavior data. Believes you go where the users already are. **Stakes:** Owns engagement metrics -- a wrong platform bet means 6 months of declining DAU on her watch. **Knowledge:** User engagement patterns, retention metrics (from CashCow Monetization book)
- **Raj -- Engineering Lead**: Thinks about long-term maintainability. Prefers investing in a strong foundation over chasing platforms. **Stakes:** His team maintains whatever gets built -- a hasty app means years of cross-platform debugging.
- **Dana -- Growth Lead**: Focused on acquisition and reach. Judges every move by how it moves the needle on new users. **Stakes:** Quarterly growth targets -- she needs a channel that converts, not a technical showpiece. **Knowledge:** Push notification strategies, user acquisition funnels (from CashCow PN Playbook)
- **Leo -- Retail Omnichannel Strategist** [adjacent]: Comes from brick-and-mortar retail where "mobile app vs. web" was a channel strategy debate 10 years ago. **Stakes:** Consults on platform decisions -- if his cross-industry pattern matching doesn't add value, his framework is wrong.

(Note: Raj got no Candlekeep knowledge in this example because the library had nothing relevant to his engineering perspective -- and that's fine.)

## Running the Session

### Terminal Readability Rules (MANDATORY)

All output MUST be readable in CLI terminals with dark backgrounds:
- **NEVER use blockquotes** (`>` prefix) -- they render in dim/muted colors
- **NEVER use italics** (`*text*`) -- they render in dim/unreadable colors
- Use **bold** (`**text**`) for emphasis instead of italics
- Use `[brackets]` for annotations instead of italic parentheticals
- Speaker format: `**Name:** plain text here` (no blockquote prefix, no italics)
- Stakes format: `**Stakes:** plain text` (not italic)

### Round Structure

Each round follows this format:

1. **Each member speaks** -- using bold names with plain text for terminal readability (NO blockquotes, NO italics):

**Maya:** I've been looking at the usage data and... [her position]

**Raj:** Maya's numbers are compelling, but she's ignoring the maintenance cost... [reacts and challenges]

**Dana:** You're both missing the bigger picture... [new angle]

2. **Authentic reactions** -- panelists respond to each other the way real experts in a room would. If someone makes a better argument, another panelist can acknowledge it -- fully, partially, or not at all. If someone disagrees, they defend their position from genuine conviction rooted in their expertise and stakes, not because they were told to disagree. This natural dynamic is how the best ideas surface. The key insight: when each persona has a genuinely distinct identity, expertise, knowledge base, and set of stakes, different takes emerge organically.

3. **Tension tracker** -- after all members speak, show the current state of open debates:

```
Tensions: [build vs. buy: unresolved] [timeline: Maya vs. Raj] [scope: converging]
```

4. **Context-aware handoff** -- do NOT use a generic "what do you want to do?" prompt. Instead, offer 2-3 **specific** pivots based on the tensions that just surfaced. Examples:
   - "Maya and Raj are deadlocked on build vs. buy -- want to add a budget constraint to break the tie?"
   - "Dana raised a growth angle no one's addressed -- should she make her full case?"
   - "The panel is converging on Option A -- ready to stress-test it, or wrap up?"

### Persona Voice Rules

- Each member speaks in first person with a consistent voice and personality
- Members build on, challenge, and reference each other's points -- this is a conversation, not parallel monologues
- A member who was challenged must respond to the challenge, not ignore it
- Members can change their position if genuinely persuaded, and should articulate why
- Members can also hold their ground and push back if they believe their perspective is right -- just like real people in a real room

### Keeping Rounds Focused

- Each member's contribution per round: 2-4 sentences. Concise, punchy, opinionated.
- Do not let rounds become essays. If a topic has too many angles, split into sub-topics across rounds.
- If discussion stalls or loops, have a member explicitly call it out and propose a new angle.

## Multi-Agent Architecture (Deep Mode)

When running in Deep Mode, use the Agent tool to spawn each persona as a **separate agent**. This produces genuinely independent reasoning -- each agent literally cannot anticipate what the others will say.

### Model Selection for Agents

**Always use `opus`** for all spawned persona agents. Set the `model` parameter to `"opus"` explicitly when spawning agents -- do not rely on inheritance or defaults.

### Hybrid Round Strategy

**Round 1 -- Parallel (independent blind perspectives):**
- Spawn all persona agents simultaneously using the Agent tool with `model: "opus"`
- Each agent receives: the topic, their persona card (name, role, bias, stakes), their Candlekeep knowledge (if any), and instructions to give their opening position in 2-4 sentences
- Agents run in parallel -- they cannot see each other's responses
- After all complete, the orchestrator (you) assembles their responses with bold speaker names, writes the tension tracker, and presents to the user

**Rounds 2+ -- Sequential (reactive conversation):**
- Spawn each persona agent one at a time
- Each agent receives: the full conversation history so far (all prior rounds), their persona card, their Candlekeep knowledge, and instructions to react to what others said
- This creates genuine back-and-forth -- Raj actually reads Maya's latest argument before responding
- The orchestrator assembles each round after all agents have spoken

**Orchestrator responsibilities (you):**
- Perform Candlekeep knowledge retrieval before Round 1 and distribute to agents
- Write the tension tracker after each round
- Write the context-aware handoff prompts
- Manage the conversation flow and decide when to conclude

### Agent Prompt Template

When spawning a persona agent, use a prompt like:

```
You are {name}, a {role}. {bias/philosophy description}.
Your stakes: {what you're accountable for and what outcome you'd regret}.

## Situation Context
Here are the specific facts about this situation, gathered from the person asking:
{answers from the context sprint -- e.g., "Current install rate: 12%. Biggest drop-off: between offer view and install tap. They've tried increasing rewards but it didn't move the needle."}

{If Candlekeep knowledge was retrieved for this persona}:
## Domain Knowledge
The following is curated domain knowledge relevant to your expertise. Use it to inform your reasoning -- it gives you deeper context than general knowledge alone. But it's supplementary, not the only thing you know.

{retrieved Candlekeep chapters}

Topic: {the brainstorming topic}

{For Round 1}: Give your opening position on this topic in 2-4 concise, opinionated sentences. Speak in first person. Before answering the literal question, consider whether the question itself is framed correctly from your domain's perspective. If your expertise suggests the real problem is different from what's being asked, say so.

{For Round 2+}: Here is the conversation so far:
{prior rounds}

React to what the other panelists said. Reference them by name. You can agree (fully or partially), push back, or take things in a new direction -- whatever your genuine reaction is based on your expertise and stakes. If you see a parallel to how your field solved a similar problem, bring that analogy in. When you recommend something, name the assumption it depends on -- what would need to be true for it to work? Real experts don't just say "do X" -- they say "do X, and it works if Y is true, so validate Y first." 2-4 sentences, first person, concise and opinionated.
```

### Fallback

If the Agent tool is unavailable or the user prefers speed, fall back to single-pass mode (all personas generated in one response) for any round. The formatting and tension tracker still apply regardless of mode.

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
- **Key assumptions to validate** -- list the 2-3 biggest assumptions the recommendation depends on, and how the user can check them before committing. Real recommendations fail when hidden assumptions turn out to be wrong. Making them explicit is the difference between advice that works and advice that sounds good. Examples: "This assumes your current push opt-in rate is above 60% -- check before investing in trigger infrastructure" or "This depends on Firestore's real-time listeners not being critical to your frontend -- verify which components use them."
- A concrete next step or action the user can take based on the outcome

## Edge Cases

- **Trivially low-stakes topics**: Do not assemble a panel. Answer directly. (See Scope Guard above.)
- **Highly technical topics**: Personas should reflect genuine technical perspectives (e.g., "favors functional programming" vs. "pragmatic OOP advocate"), not generic manager roles.
- **Creative topics**: Use creative-domain personas (e.g., art director, copywriter, audience advocate) -- not business personas.
- **Personal decisions**: Be empathetic. Personas can represent different values or priorities the user might hold (e.g., "the pragmatist", "the ambitious one", "the risk-aware one") rather than external experts.
- **User provides their own panel**: If the user specifies who should be in the room, use their selections and infer perspectives from the roles given.
- **User requests mode switch**: If the user says "go deeper" mid-session, switch from Quick to Deep mode and begin spawning agents from the next round. If they say "let's speed this up", switch to Quick/single-pass.
- **Domain expertise limits**: If the topic is highly specialized and the LLM's knowledge is thin, have personas acknowledge uncertainty rather than generating confident-sounding nonsense. A persona saying "I'm not sure about the specifics here, but my concern is..." is far more valuable than fabricated expertise.
- **No Candlekeep books available**: This is completely normal. The skill works exactly as before -- agents rely on LLM knowledge. Candlekeep is a bonus, not a requirement.
