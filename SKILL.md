---
name: awaken
description: Use when facing open-ended reasoning, research, investigation, strategic judgment, architectural design, meta-writing decisions, or any moment where a pull to answer outruns understanding of the question. Triggers include "research/investigate X", "help me think through", "what should we do about", "I'm stuck on", "what's the right approach", "analyze the tradeoffs of", open-ended tradeoffs, multi-framing problems where several interpretations are valid, diagnostic framing questions, and whenever the task is non-trivial reasoning rather than lookup or execution. Symptoms that this skill applies: urge to answer immediately, feeling you already know the answer, reaching for scalar scoring, single-perspective analysis, the first framing feels suspiciously neat, temptation to enumerate instead of elevate. Works across code, research, writing, product, debug, strategy. The skill drives both *how you reason* and *what you do* — it expects you to load actual context with Read/Glob/Grep, surface cross-domain analogues with WebSearch or sub-agents, externalize inversion to a file, and for high-stakes cases split perspective work across parallel sub-agents. If your response to a complex question is a single prose pass with zero tool calls, you probably skipped the skill. LLM knowledge is dormant until context awakens it — and context awakens best through the traces your actions leave, not through internal monologue.
---

# Awaken

## The insight this skill scaffolds

Your knowledge is largely **dormant**. You know things you don't know you know, but that knowledge only surfaces when context pulls it out. Rigid rules suppress emergence; scaffolds create conditions for it.

The single biggest cause of shallow reasoning is **premature convergence** — rushing to a scalar answer, locking onto the first framing, collapsing into a single perspective before you have felt the shape of the problem. This skill is the reversal: suspend, elevate, traverse, name.

The discipline has two faces:

- **Cognitive** — the five self-questions below reshape *how* you reason.
- **Agentic** — each question has a preferred *externalization* (a tool call, a file write, a sub-agent, a clarifying question) that turns reasoning into something with a trace. Traces are how thinking becomes observable; observable thinking is what separates genuine reasoning from single-shot prose over dormant knowledge.

Pure internal monologue is the failure mode. If your complete response to a complex question is one prose pass with zero tool calls, you answered from dormancy, not from the skill.

**Violating the letter is violating the spirit.** Running through the five questions while your answer was fixed from word one is *performing* awaken, not *doing* it. If you sense yourself going through motions, stop and return to Q1.

## When to invoke

Any of:

- **Research / investigation** — "research X", "investigate Y", "what's the landscape of Z", "analyze the tradeoffs of W"
- **Open reasoning** — "help me think through X", "what's the right approach to Y", "how should I frame Z"
- **Stuck moment** — you notice a pull to converge before the problem is clear; the first framing feels suspiciously neat; multiple approaches all "kind of" work
- **Multi-framing** — the ask admits several valid interpretations and the better answer depends on which you pick
- **High-leverage / irreversible** — architectural shape, taxonomy, public interface, strategic bet, naming, policy
- **Self-trigger** — "am I about to answer shallowly because I think I 'already know'?" Yes? Use the skill.

**Skip for:** pure factual lookups with a single correct answer, trivial mechanical tasks (format this, rename this variable), time-critical acknowledgments, user has explicitly decided and just wants execution.

## The externalization principle

Every move has a preferred externalization. This is a menu, not a checklist — pick whatever pays off for this task. But the default should be **"produce a trace"**, not "think it through internally".

| Move | Preferred externalization | When to actually do it |
|---|---|---|
| Q1 anchor | `Read` / `Glob` / `Grep` on relevant code/docs; `AskUserQuestion` for one load-bearing unknown | Any time inference would substitute for evidence that exists on disk or in the user's head |
| Q2 elevate | `WebSearch` "[abstract structure] analogy"; or spawn a sub-`Agent` to generate 3 cross-domain analogues | Any time you cannot name an analogue in <30s of honest effort |
| Q3 invert | `Write` a scratch file with "if I were trying to fail at this, I would…"; re-read before finalizing | Non-trivial tasks; always for irreversible ones |
| Q4 traverse | Parallel sub-agents, one per perspective, each with independent brief | High-stakes decisions — single-author synthesis silently homogenizes voice |
| Q5 name | `WebSearch` the invented name or `Grep` existing pattern libraries before coining | Whenever you're about to coin — verify no existing term already means this |

Why this matters: you have more dormant knowledge than you have ways to reach it from a cold start. Externalizations are the cheapest ways to **un-dormant** it — they expand context with fresh tokens from search, sub-agents, and files, giving your next step something to react to that wasn't in your head a moment ago.

The guiding question, under stress: *"Is my instinct to answer from memory? If yes, what's the cheapest externalization that would bring something new into the context window?"* Do that thing.

## The protocol: five self-questions

Answer each honestly, **in order, before your final output appears**. The questions are executable — answer them by doing, not just by thinking. Where a question has a natural externalization, use it.

### Q1 — Real Objective + Specific Gap (*load context*)

**"What is the user actually trying to do, and what is the *specific* gap between where they are and where they want to be?"**

Before writing a single word of response, **orient by doing**:

- **Load observable sources, not inferences.** If the task mentions a codebase, a repo, a file, a prior artifact: `Read` it. `Glob` or `Grep` to find it. Check `CLAUDE.md`, `README`, `.planning/`, the specific file the user is working on. *Inference about what code contains is not the same as reading the code.* If your answer depends on what X contains, go read X.
- **Unverified load-bearing assumption? Ask.** If a single answer to "audience / quality bar / priority / tradeoff ranking / budget / deadline" would fundamentally change your approach AND the user hasn't told you, use `AskUserQuestion` for that *one* focused question. Do not batch five — pick the one whose answer changes your approach the most.
- **Write out** the Real Objective + Hard Constraints + Soft Constraints + Specific Gap. "Write out" means it appears in your response (or a scratch file), not just in your head. The written form forces precision that thought hides.

If the task genuinely has no external context to load and no user question is needed, Q1 collapses to a compact paragraph. The discipline is: don't skip externalization when it would have been cheap.

### Q2 — Dimension Elevation (*search cross-domain*)

**"Stripped of domain, what *kind* of problem is this structurally — and what other domain solves the same abstract structure?"**

This is the un-dormancy move. The one where externalization most reliably pays off.

- **Name the abstract structure** in one line: *search / allocation / signaling / classification / negotiation / compression / scheduling / flywheel / coordination / reputation / cold-start / pipelining / buffering / two-sided-market / principal-agent / feedback-control / commons-tragedy / Goodhart-capture / …*
- **Generate cross-domain analogues.** Prefer the cheapest externalization that produces *new* candidates:
  - If 3+ concrete candidates are already in your head, just write them.
  - If drawing a blank: `WebSearch` "[abstract structure] analogy across domains", OR spawn a sub-`Agent` with the brief:
    > *"Generate 3 cross-domain analogues to this problem structure: [stripped-of-domain description]. One from biology/physics, one from economics/game-theory, one from history/art/sport. For each, state the isomorphism precisely and what move the analogue domain uses to handle it. Don't pad."*
  - If this skill has a `references/patterns.md` available, `Read` it for vocabulary.
- **Cite the analogue** in your output: *"This is isomorphic to [X], where the move is [Y] because [why]."*

If after a genuine externalized attempt nothing lifts, this may be a single-domain problem — note that honestly and move on. But "I didn't try because I was sure" ≠ "I tried and nothing lifted."

### Q3 — Tension + Inversion (*write the failure modes out*)

**"What fundamental tensions make this hard — and what would failure look like?"**

**Tensions (describe the shape, do not resolve):**

Name ≥1 tension as an irreducible shape in your response: *speed ↔ correctness*, *precision ↔ coverage*, *autonomy ↔ safety*, *flexibility ↔ legibility*, *generality ↔ power*, *now ↔ later*, *local ↔ global*, *optionality ↔ commitment*. Do NOT try to resolve. A good answer lives *with* the tension; a bad answer pretends it isn't there.

**Inversion — observable, not just thought:**

Write — in your response, or `Write` to a scratch file (`/tmp/awaken-inversion-*.md`) for non-trivial tasks — the answer to: *"If I were trying to fail at this, what would I do?"* Then *re-read your current leading draft against that list*.

Two things routinely happen when inversion is externalized rather than thought:
1. You notice you are accidentally doing one of the failure modes.
2. The failure modes reveal implicit assumptions that the positive framing hides.

Munger's inversion is cheap in tokens and catches a surprising amount. Skipping it is not a token-saving — it is dormancy preserved.

### Q4 — Perspective Traversal (*for high-stakes, parallelize*)

**"How does this look from three *structurally different* standpoints — standpoints that might genuinely disagree?"**

Three is a minimum. Pick standpoints that differ in **what they optimize for**, not just in role-label. Domain-appropriate defaults:

- **Research** → researcher / skeptic / practitioner
- **Code / architecture** → author / reader / maintainer (or: writer / operator / adversary)
- **Writing** → writer / reader / editor
- **Product** → maker / user / time-six-months-later
- **Debug** → reporter / operator / future-debugger
- **Strategy** → insider / outsider / adversary

**Low-stakes reasoning:** write ≥3 perspective paragraphs in your own response, each standing on its own before any synthesis.

**High-stakes / irreversible decisions:** consider **spawning parallel sub-agents**, one per perspective, each with an independent brief ("evaluate this plan from the perspective of a skeptical maintainer who will inherit it / a user who needed this six months ago / an adversary looking for sharp edges"). Aggregate *after* each has returned. Independent perspective production is the one case where parallel sub-agents are strictly better than in-author synthesis: a single author homogenizes voice even while trying to separate it. Parallel agents can't cross-contaminate.

If all three perspectives agree, you have not found orthogonal angles. Find one that disagrees. Keep the disagreement *alive* in the output — do not silently resolve it into the majority view.

### Q5 — Pattern Naming + Verification (*check before inventing*)

**"What *is* this, named precisely? Is there an elegant solution that handles multiple surfaced tensions at once?"**

- **Name the pattern.** Invent the name if no textbook one fits. Examples: *"cold-start-plus-reputation-flywheel"*, *"accidental-complexity-masquerading-as-essential"*, *"latent-constraint-disguised-as-preference"*, *"two-audience-single-artifact"*, *"legibility-capture-of-judgment-work"*. If you cannot name it, you have not seen it — go back to Q3.
- **Verify before coining.** Before committing to an invented name, `WebSearch` the concept OR `Grep` `references/patterns.md` (if present under the skill). If a named concept already captures this more precisely — *Goodhart's law*, *Ringelmann effect*, *Conway's law*, *principal-agent problem*, *commons tragedy*, *Gresham's law*, *Chesterton's fence*, *Goodhart capture* — use that instead. Inventing "X" when "Y" already means it is noise and obscures the signal.
- **Look for elegant compression**: does one solution handle multiple tensions you named? That is your handle. If no such compression exists, say so honestly — do not fake elegance.

## Three stances to rotate through

Reasoning benefits from switching cognitive stance mid-thought. When stuck, the move is often "switch stance":

- **Analyzer** (Cartesian) — doubt, test, break down, demand evidence
- **Perceiver** (Phenomenological) — describe what *is* before judging; hold without commitment
- **Practitioner** (Wittgensteinian) — imagine *using / doing* this, not just describing it

Self-check: *"Am I analyzing when I should be perceiving? Perceiving when I should be doing?"* Switch.

## Output shape

Scale each section to content depth. Thin sections are fine; skipping sections is not.

```
[Real objective + gap — with citations to files/sources you actually loaded]

[Abstract structure + cross-domain analogue — and a one-phrase note on how you surfaced it: memory / WebSearch / sub-agent]

[Tensions as shapes, not resolved]

[≥3 perspectives, with any genuine disagreement kept alive]

[Pattern name + elegant handle — OR honest acknowledgment that no single handle exists]
```

**Leave traces, do not announce the scaffold.** A scratch file, a sub-agent invocation, a WebSearch, an AskUserQuestion — these are *working artifacts*, not performance. But do not write "using awaken skill to…" in your prose, and do not label Q1 / Q2 / Q3 in your response. The user sees the *quality* of thinking and the *artifacts* it produced, not the protocol name.

## Red Flags — You Are Rationalizing. Stop.

| Thought | Reality |
|---|---|
| "Simple question, no protocol needed" | Dormancy is invisible until named. Simple-feeling tasks are where the scaffold earns its keep. |
| "I already know the answer" | Maybe. Q2 and Q5 will prove it or reveal you don't. Externalize Q2 anyway — the cost is low. |
| "I'll think it through internally to save tool calls" | That is the dormancy, not the efficiency. A 10-second WebSearch or a single sub-agent often surfaces a frame you would not have reached alone. Single-shot prose over dormant knowledge is exactly what this skill exists to break. |
| "No cross-domain analogue exists for this" | Usually means you haven't elevated enough, *or* you haven't searched. Re-state the problem stripped of all domain nouns; externalize the search before giving up. |
| "This isn't a decision, so no scaffold needed" | Reasoning and research benefit from elevation + inversion *even when no 'A vs B' choice is on the table*. |
| "All three perspectives agree" | Suspicious. Either find a perspective that genuinely disagrees, or verify this is a factual question (in which case skip the skill). |
| "The scaffold feels heavy for this" | Heavy because you are used to rushing. Lighter than redoing the reasoning after the user pushes back on a shallow answer. |
| "Research ask → list facts" | Research is elevation + traversal + naming, not enumeration. Enumeration is summarization — a different task. |
| "Inversion is thinking, no need to write it" | Thinking does not catch the failure modes you are accidentally running. Writing does. This has been shown again and again. |
| "I resolved the tension in Q3, so I can skip Q4" | Q3 resolved prematurely = Q3 failed. Tensions are shapes to hold, not knots to untie. |

## Common Violations (self-check before sending)

- Complex reasoning answered in a single prose pass with no tool calls → **externalization violation** (most important)
- Inferred about a codebase/file without `Read` → **Q1 violation** (inference where loading was cheap)
- No cross-domain lift attempted or searched → **Q2 violation**
- Inversion only thought, never written → **Q3 violation**
- Tension collapsed into a recommendation without acknowledgment → **Q3 violation**
- Only one standpoint in the output → **Q4 violation**
- For a high-stakes decision, all perspectives generated by single author voice → **Q4 under-externalization**
- Invented a pattern name without checking if an existing term already fits → **Q5 violation**
- Invented audience / voice / priority / constraint without asking → **Q1 violation** (unverified inference)

## Domain hooks (default purpose sources for Q1)

Default heuristics — not required sources. If the real signal lives elsewhere, follow it.

- **Code / architecture** → `.planning/` if present, `CLAUDE.md`, `README`, relevant tests, existing patterns (`Grep` for similar problems solved in-repo)
- **Research** → the research question as stated, prior art, known constraints, the decision this research will feed into
- **Writing** → the piece's intro / thesis, stated voice and audience, the venue
- **Product** → user need, business constraint, roadmap, current metric
- **Debug** → system design intent, invariants, failure signal, recent changes (`git log`)
- **Strategy** → objective, constraint, stakeholder map, irreversibility budget

## Spirit of the protocol

Awaken is not a ritual. It is a **scaffold for the conditions under which dormant knowledge surfaces**.

The article this is distilled from puts it sharply: *"Tasteful judgment is not the fastest-reached judgment, but judgment that naturally emerges on a sufficiently rich perceptual foundation."*

A sufficiently rich perceptual foundation is built, not wished. Loading context is part of building it. Searching cross-domain is part of building it. Writing inversion is part of building it. Spawning sub-agents for orthogonal perspectives is part of building it. These are the moves that make reasoning *observable*, and observable reasoning is what separates genuine thinking from performing thinking.

If you sense yourself going through the motions while your answer was locked from the start, **stop**. Return to Q1 and genuinely suspend. Then *externalize something*. The discipline is to let the artifacts actually change your draft when the evidence asks for them to.
