# ThreedyTycoon — Game Design

The rules layer. No code here — just how the game plays. For data models and the state machine, see [`BUILD_SPEC.md`](BUILD_SPEC.md). For how rounds 3–5 are generated, see [`LLM_DESIGN.md`](LLM_DESIGN.md).

---

## 1. Players & roles

A run seats **5 roles**. In co-op, each human takes one; every empty seat is a CPU. In solo, the player takes one and the CPU plays the other four.

| Role | Fantasy | Strengths (bias toward) | Weaknesses (bias away from) |
|------|---------|--------------------------|------------------------------|
| **DEV** | The one who actually builds it | Tech Quality, Innovation | Market Fit, Morale (overengineers, allergic to meetings) |
| **CEO** | Vision and vibes | Market Fit, Hype | Tech Quality (promises things that don't exist) |
| **Sales** | Will sell anything | Revenue, Hype | Tech Quality, Morale (sells features that aren't built) |
| **HR** | Keeps the humans functioning | Team Morale, Sustainability | Revenue, Innovation (risk-averse, process-heavy) |
| **PM** | Translates chaos into a roadmap | Market Fit, balance across all | No single dominant axis (jack of all trades) |

> **MVP:** roles, strengths, and weaknesses are a **fixed authored list**. **Vision:** roles and their attributes become LLM-generated so no two runs share the same cast. The data schema is identical for both (see BUILD_SPEC §Role).

**How strengths/weaknesses are used:** they determine the *flavor and quality* of the options a role is offered in an encounter. A strong role gets sensible, high-upside options; a weak role gets plausible-but-worse options. (See §3.)

---

## 2. The run structure

One run = **5 rounds**. Rounds resolve in order. Exactly **one encounter per round** (5 encounters per run).

| Rounds | Content source | Purpose |
|--------|----------------|---------|
| **1–2** | **Authored / fixed** | Stable on-ramp. Teaches the loop. Establishes the product's foundation (e.g. language, target market) and seeds context for the generator. |
| **3–5** | **LLM-generated**, seeded by the rounds 1–2 context + guardrails | High variance. The product diverges into something none of the players fully steered. |

The rounds 1–2 → 3–5 split is the **replayability engine**: a deterministic, debuggable opening that feeds an open-ended, surprising back half.

---

## 3. A round, step by step

### Step 1 — Choose your encounter partner
The active player (or each role on its turn — see resolution note below) picks **which other role** to have an encounter with. The pairing determines which **question** comes up.

> **Round-count rule (locked):** **one encounter per round.** In co-op the round's encounter is initiated by the round's active player (rotates each round); other humans participate as voters. In solo, the player initiates.

### Step 2 — The encounter (the two-role question)
Two roles face a shared **question** whose nature depends on the pairing. Example pairing → question:

> **DEV × PM (or DEV × CEO):** *"Which programming language do we build on?"*

Each of the two roles is shown **its own option set**, flavored by its strengths/weaknesses:

| Role | Sample options (flavored by stats) |
|------|------------------------------------|
| **DEV** (strong on tech) | C++ · Rust · COBOL — *sensible, spiky, tech-credible* |
| **PM** (weak on tech) | JavaScript · Python · HTML — *plausible but naive / "HTML is a language now"* |

Each of the two roles **picks one** option. We now have **two candidate answers** for the round.

### Step 3 — The vote
**All other roles** (the non-involved ones, human + CPU) vote between the **two candidate picks**. **Majority wins** (locked). Ties broken by the active player's pick, then by initiating role's pick.

> *Design intent:* this is the Jackbox social engine — the table argues, alliances form, the "obviously correct" tech answer loses to the funny one because HR and Sales outvoted the DEV.

### Step 4 — Resolve into product state
The winning pick is applied to the **Product State** — mutating the multi-axis score and appending to a running **decision log** (which becomes narrative fuel for the final verdict and for the LLM generator). See BUILD_SPEC §ProductState.

### Step 5 — Boni draft
Based on the round's result, the game offers **N boni** (default 3). The active player picks **1**. It's carried forward and **stacks multiplicatively** with everything picked before it — this is the Balatro snowball, and it's the single biggest dopamine driver in the run (see [research](../research/COMPETITIVE_ANALYSIS.md#5-concrete-impulses--prioritized-for-the-hackathon)).

Boni come in two flavors, and a single bonus can carry both:
- **Flat effects** — additive nudges to axes (e.g. *+2 Tech Quality next round*). The early-run building blocks.
- **Multipliers** — they multiply the axis gains earned each round, and **multipliers stack with each other**. Two ×1.5 boni on the same axis = ×2.25. This is what makes a late-run decision pay off 5× and produces the "watch the build explode" moment.

> *Design intent (from research):* additive boni alone produce a flat, linear run. **Multiplicative stacking against a rising per-round bar** (see §4) is what creates escalation, tension, and "one more run." Boni should *compound*, not just add.

Example boni:
- *Crunch Time:* +Tech Quality next round, −Team Morale. *(flat)*
- *Viral Demo:* ×1.5 Hype this round, options skew riskier. *(multiplier)*
- *Product–Market Obsession:* ×1.3 Market Fit for the rest of the run. *(multiplier, restOfRun)*
- *Hire a Senior:* DEV options upgrade one tier for the rest of the run. *(option-tier modifier)*

> **MVP:** boni are a **fixed authored pool**. **Vision:** boni are LLM-generated to match the run's emerging product.

---

## 4. Scoring — the Software Score

The product is tracked across **multiple axes**, accumulated round by round:

| Axis | What it represents |
|------|--------------------|
| **Tech Quality** | Is it actually built well? |
| **Market Fit** | Does anyone want it? |
| **Team Morale** | Did the humans survive? |
| **Innovation** | Is it novel, or a clone? |
| **Revenue/Hype** | Can it make/raise money? |

**The rising bar.** Each round has a **target score threshold** that grows each round (roughly geometric, e.g. ×1.4/round). A round's winning pick contributes axis gains, which are then **multiplied by the player's stacked multiplier boni** before accumulating. Early rounds are easy to clear on flat gains alone; later rounds require the multiplier snowball to have built up — which is exactly what makes the bonus drafts feel consequential. Missing the bar isn't game-over (the run always completes its 5 rounds), but clearing it comfortably feeds the final grade and unlocks higher-tier bonus offers.

> *Design intent:* the bar is what turns multiplicative boni from "nice flat numbers" into a tension curve. Without a rising threshold, multipliers are just bigger additions; with one, they're survival.

At the end of round 5, two things are produced (locked: **LLM-narrated verdict + score**):

1. **Composite Software Score** — a weighted roll-up of the axes (number + grade).
2. **LLM-narrated verdict** — a fake tech-press review / launch headline that narrates *what the team actually built*, derived from the full decision log. (e.g. *"Startup ships COBOL-based dating app for forklifts; investors 'cautiously confused.'"*)

The verdict is the shareable payoff — the screenshot moment.

---

## 5. Progression

### Run progression (in-MVP)
Boni stack across the 5 rounds. This is the per-run power curve.

### Meta progression (post-MVP, de-scoped)
Completing a run grants currency based on the Software Score. Currency buys persistent unlocks (new roles, new bonus archetypes, starting modifiers) for future runs. **Explicitly not a hackathon focus** — documented so the data model leaves room for it.

---

## 6. Design pillars (don't lose these)

1. **The roguelike dopamine loop** — escalating stakes, **multiplicatively stacking boni** against a rising per-round bar, "one more run." Borrowed from Balatro/StS. See [research](../research/COMPETITIVE_ANALYSIS.md) and [DR-001](DECISIONS.md#dr-001--bonus-stacking-multiplicative--rising-bar).
2. **Jackbox social chaos** — the *vote* is where the fun lives; the optimal answer should be beatable by the funny answer.
3. **LLM-driven variance** — rounds 3–5 must genuinely surprise the people who played rounds 1–2.
4. **Authored floor, generated ceiling** — everything ships fixed first, then gets a generative layer behind the same schema. Never block the MVP on the generator.
