# ThreedyTycoon — Competitive Mechanics Analysis

Research into the **dopamine loop of modern roguelikes** and the **multiplayer mechanics of Jackbox party games**, to generate concrete design impulses for ThreedyTycoon. Multi-source, adversarially verified (25 claims, 23 confirmed, 2 refuted). Sources cited inline; full source list at the bottom.

> **TL;DR.** The two genres' loops mostly *reinforce*. Roguelikes give you "one more run" through a fixed **draft → play → resolve → reward** loop, an **escalating per-round bar**, **synergy-stacking builds** where any micro-decision can swing the run, and meta-progression best understood as *growth in the player's own mastery*. Jackbox makes **the act of voting itself the payoff** — players judge each other's creative outputs, assets get **remixed across players**, and an escalating vote-off produces the **shareable comedic reveal**. The genres *conflict* on one axis: roguelike mastery wants a **legible** system, while Jackbox comedy and an LLM judge thrive on **surprise**. ThreedyTycoon's LLM judge is the highest-leverage, highest-risk component — and research shows **rubric design is load-bearing**: a decomposed, weighted rubric lifts judge accuracy +17.7 pts, while a naive "vibes" prompt makes the judge *worse than no rubric at all*.

---

## ⚠️ Title correction: "Scritchy Scratchy" is the wrong game

You referenced **"Scritchy Scratchy."** It's a real game (Steam app 3948120, Lunch Money / Funday Games, Mar 2026) — **but it's an incremental scratch-card game (Casual/Simulation), not a roguelike or party game.** It has nothing analytically useful for ThreedyTycoon.

**The game you almost certainly meant is [Scratchcard Hero](https://store.steampowered.com/app/3622600/Scratchcard_Hero/)** (Steam app 3622600, dev *manadream*) — *"a roguelite deckbuilder where all your cards are lottery scratch-off tickets."* Its hook is that **the scratch/reveal is skill-based**, not pure chance: *"how you scratch it matters. Speed, precision and quick decision making are needed to win."* That's the relevant reference.

> ⚠️ **Caveat:** both scratch-card titles are unreleased/early — Scratchcard Hero's mechanics are *stated design intent*, not play-verified. Confirm before leaning on it. *(Verified: the claim that Scritchy Scratchy is the intended roguelike was refuted 0-3.)*

---

## 1. The roguelike dopamine loop — what actually makes it compelling

### Balatro — synergy-stacking against an exponential bar
Officially *"a poker-inspired roguelike deck builder all about creating powerful synergies and winning big"* ([Steam](https://store.steampowered.com/app/2379780/Balatro/)). Scoring is **Chips × Mult** — the hand's base value becomes Chips, multiplied by Mult, and **Jokers add to either side when their condition triggers** ([Wikipedia](https://en.wikipedia.org/wiki/Balatro)). Because it's multiplicative, stacked Jokers compound explosively against **per-Ante "blind" thresholds that grow exponentially**. The "one more run" feel is attributed to the fact that *"every pick-up, discard and joker can dramatically alter the course of your run."*

> **Impulse:** ThreedyTycoon's **boni should compound multiplicatively**, not just add flat points, against a **rising per-round bar**. The dopamine is in watching a build *snowball* — one early bonus making a later decision pay off 5×.

### Slay the Spire 2 — meta-progression as a challenge ladder
STS2's **Ascension** system is a 10-level ladder; each level unlocks by completing a run at the current top level, and modifiers **stack cumulatively** — resource scarcity → restrictions → economy/card-pool limits → enemy scaling, topping out at a **double boss** ([untapped.gg](https://sts2.untapped.gg/en/guides/ascensions-list-best-strategies), [wiki](https://slaythespire.wiki.gg/wiki/Slay_the_Spire_2:Ascension)). *(Note: STS2 has 10 Ascensions; the original had 20 — don't conflate. EA values may change.)*

> **Impulse:** if ThreedyTycoon ever adds meta-progression, the *strongest* version isn't "unlock more content" — it's an **opt-in difficulty ladder** that adds stacking constraints, giving repeat players a reason to return. For a party game played once, this matters less (see §4 conflict).

### 9 Kings — separate the decision from the resolution; make *placement* matter
Each "year" is a fixed loop: **draft cards → play until two remain → auto-battle → earn rewards** ([mejoress](https://www.mejoress.com/9-kings-definitive-guide/)). The active decision phase is cleanly separated from automated resolution, and buffs **stack multiplicatively** on a 3×3 grid where **adjacency/positioning is its own agency lever** (center beats corners). *(⚠️ 2-1 verified — grid specifics rest on secondary/community sources; treat exact values as illustrative. The "12-16 tight deck" tip was refuted 0-3.)*

> **Impulse:** ThreedyTycoon already separates decision (encounter/vote) from resolution (product mutation) — good. Consider whether **how decisions combine/are positioned** (order, pairing, which axis they hit) can become a second agency lever beyond just *which* option wins.

### The deepest lever: "metaprogression of the mind"
Per Cogmind dev Kyzrati, the strongest replayability driver is players **growing their own understanding** of how the systems interact — *"metaprogression of the mind"* — explicitly distinct from unlockable content, aiming at a *"highly-replayable base/core game."* And well-designed randomness should be **removable as a deciding factor by a hypothetical optimal player** — it tests mastery rather than overriding it ([Grid Sage Games](https://www.gridsagegames.com/blog/2025/08/designing-for-mastery-in-roguelikes-w-roguelike-radio/)).

> **Impulse + warning:** ThreedyTycoon's replayability should come from players *learning the role/encounter/axis interactions*, not just unlocks. **BUT** — an LLM judge is a *different, opaque class of randomness*. The "optimal player can read and mitigate the rules" precondition only holds **if the scoring rubric is legible enough for mastery to develop.** This directly motivates §3.

---

## 2. Jackbox — voting *is* the game

### The core primitive: peer judgment of creative output
Jackbox has built **17+ voting titles** and positions the vote as the climax: *"the power of the vote determines who wins… and who forever holds a grudge,"* the fun being *"choosing the best punchline or deciding whose invention should definitely not be patented"* ([Jackbox Voting](https://www.jackboxgames.com/game-type/voting)). Voting is a **proven, repeatable primitive**, not a one-off.

> **Impulse:** make **peer voting on role outputs the emotional climax**, not a side mechanic. ThreedyTycoon already does this (the majority vote in each encounter) — lean *harder* into it.

### The social engine: create → remix → vote-off → reveal
**Tee K.O.** is the template ([official](https://www.jackboxgames.com/games/tee-k-o), [Fandom](https://jackboxgames.fandom.com/wiki/Tee_K.O.)): players create under time pressure (draw images *and* write slogans — two sequential timed inputs), then **all assets are shuffled to other players at random** — you build the competitive artifact from *other people's* work. Winners emerge from an **escalating head-to-head gauntlet** (king-of-the-hill, winner stays on). Audiences are **active judges**, not spectators — e.g. Fibbage 3's audience votes in two phases (first on a decoy, then the real answer). *(⚠️ "gauntlet" is king-of-the-hill, not a seeded bracket — 2-1 on the "elimination" wording.)*

> **Impulse:** three concrete moves for ThreedyTycoon —
> 1. **Remix role outputs across players** before judging — the comedic gold in Jackbox is that the winning artifact is recombined from others' contributions. ThreedyTycoon's product is *already* a collective remix of 5 roles' decisions; surface that explicitly in the reveal.
> 2. **Stage the final verdict as an escalating reveal**, not a flat number drop — build tension toward the headline.
> 3. **Audience/spectator voting** is a proven extender if you want >5 participants.

---

## 3. The LLM judge — your highest-leverage, highest-risk component

ThreedyTycoon scores the product with an LLM. Research is blunt here: **rubric design decides whether the judge helps or hurts.** A recursively-decomposed, correlation-weighted rubric (RRD) **lifts GPT-4o judge accuracy from 55.6% → 73.3% (+17.7 pts)**, while a **naive LLM rubric *degrades* it to 42.9% — 13 points below using no rubric at all** ([Shen et al., arXiv 2602.05125](https://arxiv.org/html/2602.05125v1/)). *(⚠️ single model, single benchmark; directionally faithful but generalizing from "rubric" to "game scoring prompt.")*

> **Impulse — this validates and extends [`LLM_DESIGN.md`](../docs/LLM_DESIGN.md):**
> - **Do not ship a vibes-based scoring prompt.** Build the multi-axis score as a **decomposed, weighted rubric** (Tech Quality, Market Fit, Morale, Innovation, Hype). Our current design — *server computes the score, LLM only narrates* — is the safest possible version of this and should be kept.
> - **Make the rubric legible to players.** Per §1, mastery (the replayability driver) only develops against a readable system. Show players *which axes their decisions moved.* Opaque judging kills the roguelike loop.
> - The **narrated verdict** (our fake-tech-press headline) is exactly the right place to absorb the *subjective/surprise* energy that Jackbox thrives on — while the *number* stays rule-bound and learnable. This neatly resolves the genre conflict (below).

---

## 4. Where the loops reinforce vs conflict

| | Roguelike | Jackbox | ThreedyTycoon move |
|---|-----------|---------|------------------|
| **Reinforce** | escalating stakes, stacking builds, "one more run" | escalating vote-off, comedic climax | Per-round bar rises (roguelike) *and* each round's vote is the social beat (Jackbox). Same loop. |
| **Reinforce** | randomness-with-agency | random asset redistribution | LLM-generated rounds 3-5 = surprise; player votes = agency over it. |
| **⚠️ Conflict** | mastery wants a **legible** system | comedy/LLM wants **surprise & subjectivity** | **Split the two:** keep the *score* rule-bound and legible (masterable); push *surprise* into the LLM's **narration** and the generated rounds. |
| **⚠️ Conflict** | meta-progression rewards **repeat** play | party games are often played **once** | De-prioritize meta-progression (already our call). Put the replay value in **per-run variance** (rounds 3-5) + **mastery of role interactions**, not unlock grind. |

---

## 5. Concrete impulses — prioritized for the hackathon

**Adopt now (cheap, high-impact):**
1. **Multiplicative bonus stacking** against a **rising per-round bar** — the Balatro snowball is the single biggest dopamine driver. *(Currently our boni are additive — consider a multiplier tier.)*
2. **Escalating verdict reveal** — stage the final score like a Tee K.O. gauntlet, build to the headline. Don't drop a number.
3. **Show which axes moved** after each resolved encounter — legibility = masterability = replayability.
4. **Decomposed, weighted scoring rubric**; keep score server-computed, LLM narrates only. *(Already in LLM_DESIGN — research strongly validates it.)*

**Adapt / consider:**
5. **Remix role contributions** visibly in the reveal — the product is a collective remix; make that the joke.
6. **Audience voting** as an optional extender beyond 5 players (proven Jackbox primitive).
7. **Positioning/combination as a second agency lever** (9 Kings) — does the *order* or *pairing* of decisions matter, not just the winner?

**Change / drop:**
8. **Don't build meta-progression for the hackathon** — party games are played once; replay value is in per-run variance + mastery. *(Already our call — research confirms.)*
9. **Don't ship a "vibes" LLM scoring prompt** — verified to make judging *worse than nothing*.

---

## Open questions worth a design huddle

1. **Transparency vs surprise dial:** how much rubric you expose before mastery erodes the comedic randomness — and vice versa. (The core genre tension.)
2. **Vote target:** do players vote on each other's role decisions (Jackbox peer judgment, our current design) or only collaborate toward the LLM score? Does the LLM *replace or supplement* player voting as the climax?
3. **Meta-progression model:** pure "metaprogression of the mind" (no unlocks) vs a light persistent layer — and how that serves a play-once party audience.
4. **Shareable payoff:** how to make the LLM reveal feel like Tee K.O. gauntlet energy (narrate/roast the product) rather than an opaque verdict.

---

## Sources

**Primary:** [Balatro (Steam)](https://store.steampowered.com/app/2379780/Balatro/) · [Scratchcard Hero (Steam)](https://store.steampowered.com/app/3622600/Scratchcard_Hero/) · [Scritchy Scratchy (Steam)](https://store.steampowered.com/app/3948120/Scritchy_Scratchy/) · [Jackbox — Voting](https://www.jackboxgames.com/game-type/voting) · [Jackbox — audience play](https://www.jackboxgames.com/blog/how-audience-play-along-differs-in-each-jackbox-game) · [Tee K.O. (Fandom)](https://jackboxgames.fandom.com/wiki/Tee_K.O.) · [Grid Sage Games — Designing for Mastery](https://www.gridsagegames.com/blog/2025/08/designing-for-mastery-in-roguelikes-w-roguelike-radio/) · [Shen et al., Rethinking Rubric Generation (arXiv 2602.05125)](https://arxiv.org/html/2602.05125v1/)

**Secondary:** [Wikipedia — Balatro](https://en.wikipedia.org/wiki/Balatro) · [STS2 Ascension (untapped.gg)](https://sts2.untapped.gg/en/guides/ascensions-list-best-strategies) · [STS2 Ascension (wiki)](https://slaythespire.wiki.gg/wiki/Slay_the_Spire_2:Ascension) · [9 Kings guide (mejoress)](https://www.mejoress.com/9-kings-definitive-guide/)

**Method:** 6 search angles → 26 sources fetched → 116 claims → 25 verified by 3-vote adversarial check → 23 confirmed, 2 refuted. Refuted & excluded: "Scritchy Scratchy is the intended roguelike" (0-3); "9 Kings rewards 12-16 tight decks" (0-3).
