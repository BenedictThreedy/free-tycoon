# ThreedyTycoon — LLM Design

How rounds 3–5 are generated and how the final verdict is written. This is the system that turns a fixed opening into an unpredictable product. Rules: [`GAME_DESIGN.md`](GAME_DESIGN.md). Data shapes: [`BUILD_SPEC.md`](BUILD_SPEC.md).

> **Principle:** the LLM **fills authored schemas**, it does not invent structure. Every generated `Question`/`Option`/`Bonus` is validated against JSON Schema; **any failure falls back to the authored pool**. The game is never blocked on the model.

---

## 1. What is generated, and when

| Round | Question | Options | Boni | Verdict |
|-------|----------|---------|------|---------|
| 1–2 | authored | authored | authored | — |
| 3–5 | **generated** | **generated** | **generated** | — |
| end | — | — | — | **generated** |

Generation is **server-side only** (key safety + validation). Pre-generate round *N+1* during round *N*'s VOTE/BONI phases to hide latency.

**Model routing:**
- Rounds 3–5 content → `claude-sonnet-4-6` (fast, cheap, structured).
- Final verdict → `claude-opus-4-8` (the showpiece; worth the quality).

---

## 2. The context object (what we feed the generator)

A compact, deterministic snapshot of the run so far. **No free-form table chatter** — only structured run state, so generations are reproducible and on-rails.

```jsonc
{
  "runSeed": "abc123",                  // for reproducibility/debugging
  "roundToGenerate": 3,
  "roles": [ /* Role[] — ids, strengths, weaknesses */ ],
  "seats": { "DEV": {"isCpu": false}, ... },
  "product": {
    "axes": { "techQuality": 6, "marketFit": -2, "morale": 3, "innovation": 4, "hype": 1 },
    "activeBoni": [ {"name": "Crunch Time", "optionBias": "techUp"} ],
    "decisionLog": [
      { "round": 1, "question": "Which language?", "winner": {"role":"PM","option":{"label":"HTML"}} },
      { "round": 2, "question": "Who's the customer?", "winner": {"role":"CEO","option":{"label":"forklift fleets"}} }
    ]
  },
  "guardrails": { /* see §4 */ }
}
```

The **decisionLog is the soul of the run** — it's the thread the generator pulls on. If round 1 chose "HTML" and round 2 chose "forklift fleets," round 3's generated question should *know* it's building an HTML app for forklift fleets and escalate that absurdity coherently.

---

## 3. Output schemas (the model must return these)

Use **structured output / tool-use** so the model is forced into the shape; validate server-side before accepting.

### 3a. Generated round content
```jsonc
{
  "question": {
    "id": "q-r3-...",
    "pairing": ["SALES", "DEV"],           // must be two distinct role ids from `roles`
    "prompt": "string, <= 140 chars",
    "optionsByRole": {
      "SALES": [ /* 3 Options */ ],
      "DEV":   [ /* 3 Options */ ]
    }
  },
  "boni": [ /* exactly N (default 3) Bonus objects */ ]
}
```

### 3b. Option object constraints
```jsonc
{
  "id": "string",
  "label": "string, <= 40 chars",
  "flavor": "string, <= 100 chars",
  "tier": 1,                                // 1|2|3 — must match role strength (see §4)
  "effects": { "techQuality": 2, "morale": -1 }   // each value in [-3, 3]
}
```

### 3c. Verdict (end of run)
```jsonc
{
  "headline": "string, <= 90 chars — fake tech-press headline",
  "review": "string, 2–4 sentences — narrates what the team built from decisionLog",
  "grade": "S|A|B|C|D|F",                   // must equal server-computed bucket
  "tags": ["string", ...]                   // 2–4 punchy tags, e.g. ['#disrupting', '#cobol']
}
```

> Note: the **score is computed by the server**, not the model. The model receives the computed `grade` and *narrates* it. The model never decides the number — prevents score hallucination.

---

## 4. Guardrails

Layered defense so a generation can't break the game or the tone.

**Structural (hard, server-enforced):**
- Output must validate against the JSON Schema above. Invalid → reject → fall back to authored pool.
- `pairing` must be two distinct ids present in `roles`.
- `optionsByRole` keys must equal the `pairing` roles; each gets exactly 3 options.
- Exactly N boni (default 3).
- All `effects` values clamped to `[-3, 3]`; `tier` ∈ {1,2,3}.
- Reject duplicate option labels within a question.

**Stat-coherence (soft, prompted + checked):**
- A role's options should bias toward its `strengths` and away from its `weaknesses` (strong role → higher avg tier and more positive effects on its strong axes). Server can re-rank/clamp tiers post-hoc if the model drifts.

**Tone & safety (prompted):**
- System prompt fixes the register: *workplace-comedy, satirical, SFW, no real companies/people, no slurs, light absurdity that escalates coherently from the decisionLog.*
- Keep it about **a team building a software product, each role pulling toward its own priorities** — comedy comes from the clash of perspectives and the surprising directions the product takes, not from anyone being bad at their job. No random non-sequiturs.

**Variance budget (prompted):**
- Round 3 ≈ "things get weird." Round 4 ≈ "things get out of hand." Round 5 ≈ "fully unhinged but still recognizably *this* product." Escalate, don't reset.

**Fallback chain (never block the game):**
```
generate → validate → (fail) → retry once with stricter prompt → (fail) → authored fallback pool
```
The authored fallback pool is the same content type as rounds 1–2 — always present, always valid.

---

## 5. Prompt sketches

### Round generator (system)
> You are the scenario generator for *ThreedyTycoon*, a satirical roguelike about a software startup where five roles each pull the product toward their own priorities. Given the run so far, generate the next round's encounter. The encounter is a question between two roles; each role gets 3 options flavored by its strengths and weaknesses. **Strong roles get smart, high-tier options on their strong axes; weak roles get plausible-but-worse options.** Escalate coherently from the decision log — the product so far is `<one-line summary>`. Stay SFW, satirical, no real companies or people. Return only the structured object.

### Round generator (user)
> `<context object from §2>`
> Generate round `<n>`. Variance level: `<weird | out-of-hand | unhinged>`.

### Verdict generator (system)
> You are a satirical tech-press critic reviewing the product a startup just shipped. You are given the full decision log and a pre-computed grade. Write a punchy headline and a 2–4 sentence review that *narrates what they actually built*, true to the decisions. Match the grade's energy. SFW, no real companies/people. Return only the structured object — do not change the grade.

### Verdict generator (user)
> `decisionLog: <...>`  `axes: <...>`  `grade: <server-computed>`

---

## 6. Why this design

- **Authored floor, generated ceiling:** the same schemas back both fixed and generated content, so the MVP ships without the model and the generator slots in behind it (BUILD_SPEC Phase 2).
- **Server-authoritative score:** the model narrates, never scores — kills the most common LLM-game failure (made-up numbers).
- **decisionLog as the through-line:** coherent escalation instead of random chaos is what makes round 5 *feel* like the payoff of *your* run, not a random generator.
- **Latency hidden:** pre-generate ahead during vote/boni phases.
