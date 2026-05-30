# Free Tycoon — Build Spec

Implementation layer: tech stack, data models, state machine, and the phased hackathon plan. Rules live in [`GAME_DESIGN.md`](GAME_DESIGN.md); generation lives in [`LLM_DESIGN.md`](LLM_DESIGN.md).

---

## 1. Tech stack (browser-based, Jackbox topology)

The game runs **entirely in the browser** and uses the **Jackbox topology**: one **shared host screen** (the "stage") and per-player **phone/laptop controllers** that join with a **room code**.

```
                ┌─────────────────────────┐
                │   HOST SCREEN (stage)    │   ← TV / shared laptop / projector
                │   3D scene + round state │
                └────────────┬─────────────┘
                             │ WebSocket (authoritative room state)
                   ┌─────────┴──────────┐
              ┌────┴────┐          ┌─────┴────┐
              │ Room     │          │  LLM     │  ← rounds 3–5 generation
              │ Server   │──────────│  service │     (server-side only)
              └────┬─────┘          └──────────┘
        ┌──────────┼──────────┬──────────┐
   ┌────┴───┐ ┌────┴───┐ ┌────┴───┐  ...  (phones join via room code)
   │ Player │ │ Player │ │ Player │
   └────────┘ └────────┘ └────────┘
```

| Concern | Recommendation | Why |
|---------|----------------|-----|
| **Frontend framework** | **React + TypeScript + Vite** | Fast HMR for a hackathon; huge ecosystem; pairs with R3F. |
| **3D (polish phase)** | **react-three-fiber + drei** (Three.js) | 3D-in-React with minimal glue. *Phase 2 only.* |
| **MVP visuals (phase 1)** | **Tailwind CSS + Framer Motion** stylized 2.5D | Prove the loop with juicy 2D before touching WebGL. |
| **Multiplayer transport** | **WebSocket room server** — [**PartyKit**](https://partykit.io) or [**Colyseus**](https://colyseus.io), or a plain Node `ws` server | Authoritative room state + room codes. PartyKit is the fastest path; Colyseus gives you room/state primitives out of the box. |
| **State (client)** | **Zustand** | Tiny, no boilerplate, mirrors server snapshots well. |
| **LLM** | **server-side** call to Claude (`claude-sonnet-4-6` for speed/cost, `claude-opus-4-8` for the final verdict) | Never call the model from the client (key safety + validation). Use structured output. |
| **Hosting** | Vite static build (host+controller) + room server (PartyKit/Render/Fly) | Single repo, two deploy targets. |

**Two client surfaces, one codebase:**
- `/` → **controller** (phone): join room, see your role's options, tap to pick/vote.
- `/host` → **stage** (shared screen): the 3D/2.5D scene, current question, live vote tally, the verdict reveal.

The **server is authoritative** for room state and the *only* place that talks to the LLM. Controllers send intents (`pick`, `vote`, `chooseBonus`); the server validates and broadcasts new snapshots.

---

## 2. Data models

TypeScript-flavored. These schemas serve **both** the authored MVP and the generated vision — generation just fills the same shapes (validated against JSON Schema, see LLM_DESIGN).

```ts
type RoleId = 'DEV' | 'CEO' | 'SALES' | 'HR' | 'PM';           // MVP fixed; vision: generated ids
type Axis = 'techQuality' | 'marketFit' | 'morale' | 'innovation' | 'hype';

interface Role {
  id: RoleId;
  name: string;
  fantasy: string;
  strengths: Axis[];          // bias options toward these
  weaknesses: Axis[];         // bias options away from these
}

interface Option {
  id: string;
  label: string;              // "Rust", "Ship it Friday", ...
  flavor?: string;            // optional one-liner
  effects: Partial<Record<Axis, number>>;  // applied to ProductState on win
  tier: 1 | 2 | 3;            // quality tier; strong roles get higher-tier options
}

interface Question {
  id: string;
  pairing: [RoleId, RoleId];  // which two roles this encounter is between
  prompt: string;             // "Which programming language do we build on?"
  // each involved role gets its own option set, flavored by its stats:
  optionsByRole: Record<RoleId, Option[]>;
}

interface Bonus {
  id: string;
  name: string;               // "Crunch Time"
  description: string;
  // FLAT modifiers — additive nudges to axes (early-run building blocks):
  flat?: Partial<Record<Axis, number>>;
  // MULTIPLIERS — multiply axis GAINS earned per round; stack multiplicatively
  // with each other (two ×1.5 on the same axis => ×2.25). The snowball. See DR-001.
  mult?: Partial<Record<Axis, number>>;
  optionBias?: 'risky' | 'safe' | 'techUp';
  duration: 'nextRound' | 'restOfRun';
}

interface ProductState {
  axes: Record<Axis, number>;          // accumulated score per axis
  activeBoni: Bonus[];                 // stacked run modifiers
  decisionLog: DecisionLogEntry[];     // every resolved encounter — narrative fuel
}

interface DecisionLogEntry {
  round: number;
  question: string;
  candidates: { role: RoleId; option: Option }[];  // the two picks
  winner: { role: RoleId; option: Option };
  voteTally: Record<string /*optionId*/, number>;
}

interface RunState {
  runId: string;
  roundNumber: 1 | 2 | 3 | 4 | 5;
  phase: RoundPhase;
  roles: Role[];
  seats: Record<RoleId, { playerId: string | null; isCpu: boolean }>;
  activePlayerRole: RoleId;            // initiates this round's encounter
  currentQuestion: Question | null;
  candidates: { role: RoleId; option: Option }[];
  product: ProductState;
  bonusChoices: Bonus[];               // the N offered after resolve
}
```

---

## 3. Round state machine

```
                ┌──────────┐
   run start →  │  LOBBY   │  players join, pick roles, host starts
                └────┬─────┘
                     ▼
   each round:  ┌──────────┐
                │  CHOOSE  │  active role picks encounter partner → sets Question
                └────┬─────┘     (rounds 3–5: generate Question first, see LLM_DESIGN)
                     ▼
                ┌──────────┐
                │ ENCOUNTER│  the two roles each pick from their own options
                └────┬─────┘     → produces 2 candidates
                     ▼
                ┌──────────┐
                │   VOTE   │  all other roles vote; majority wins
                └────┬─────┘     (ties: active player's pick → initiator's pick)
                     ▼
                ┌──────────┐
                │ RESOLVE  │  apply winner to ProductState + append decisionLog
                └────┬─────┘
                     ▼
                ┌──────────┐
                │   BONI   │  offer N boni, active player picks 1
                └────┬─────┘
                     │  round < 5 → next round (CHOOSE)
                     ▼  round = 5
                ┌──────────┐
                │  VERDICT │  composite score + LLM-narrated review
                └──────────┘
```

`RoundPhase = 'LOBBY' | 'CHOOSE' | 'ENCOUNTER' | 'VOTE' | 'RESOLVE' | 'BONI' | 'VERDICT'`

**CPU behavior:** for any seat where `isCpu`, the server auto-acts on a timer — picks a stat-weighted option in ENCOUNTER, votes (weighted toward its strengths) in VOTE, picks a synergistic bonus in BONI. Keeps solo play flowing and fills empty co-op seats.

---

## 4. Scoring resolution

Per-round gain is computed, snowballed by stacked multipliers, then accumulated and checked against the **rising bar** (see DR-001):

```
// at RESOLVE, for the winning option:
for each axis a:
  flatBonus   = Σ activeBoni.flat[a]                  // additive
  multFactor  = Π activeBoni.mult[a]                  // multiplicative — STACKS
  roundGain[a] = (winningOption.effects[a] + flatBonus) * multFactor
  axes[a]     += roundGain[a]

roundScore   = Σ ( roundGain[a] * weight[a] )         // weights authored, sum to 1
barCleared   = roundScore >= bar(round)               // bar(r) ≈ base * 1.4^(r-1)

// at end of run:
compositeScore = Σ ( axes[a] * weight[a] )
grade          = bucket(compositeScore, barsCleared)   // S / A / B / C / D / F
verdict        = LLM(decisionLog, axes, grade)         // see LLM_DESIGN §Verdict
```

- **Flat** boni add inside the parens; **multipliers** stack (`Π`) and scale the whole gain — this is the Balatro snowball.
- The **bar grows each round** (geometric, ~×1.4). Missing it isn't game-over; it feeds the grade and gates higher-tier bonus offers.
- `optionBias` from active boni is passed into option generation/selection for the next round.

> See **DR-001** (Design Decisions) — multiplicative stacking + rising bar is the current bet, with a documented fallback if it feels off in playtest.

---

## 5. Phased hackathon plan

> **Risk flag:** full 3D + room-join + live LLM in one weekend is a lot. The phases below de-risk by proving the *loop* before the *graphics*. Ship Phase 1 end-to-end before touching Phase 2.

### Phase 0 — Skeleton (hours 0–3)
- Vite + React + TS + Tailwind. Room server (PartyKit) with room codes, join, role pick.
- `RunState` + state machine with **hardcoded** rounds 1–2 content. No LLM, no 3D.

### Phase 1 — The loop, fully playable (hours 3–14) ← **MVP target**
- All 5 rounds with **authored** content (rounds 3–5 use a fixed fallback pool — *not yet generated*).
- Full CHOOSE → ENCOUNTER → VOTE → RESOLVE → BONI cycle, CPU seats working.
- Composite score + a **templated** (non-LLM) verdict.
- Stylized **2.5D** UI with Framer Motion juice (vote tally animation, bonus draft). **This is a complete, demoable game.**

### Phase 2 — LLM generation (hours 14–22)
- Server-side Claude integration. Rounds 3–5 generate `Question`/`Option`/`Bonus` from rounds 1–2 context, validated against JSON Schema with **fallback to the authored pool** on any failure (see LLM_DESIGN §Guardrails).
- LLM-narrated final **verdict** (the shareable headline).

### Phase 3 — 3D polish (hours 22+, optional)
- Swap the 2.5D stage for an **r3f** scene: a little office that visibly mutates as decisions resolve. Pure polish — never on the MVP critical path.

### Explicitly out of scope for the hackathon
- Meta-progression / persistent unlocks (schema leaves room; no UI).
- Generated **roles/attributes** (vision; MVP roles are fixed).
- Accounts, persistence beyond a live room, matchmaking.

---

## 6. Open implementation questions (for Benedict + Dennis)

- Room server: PartyKit (fastest) vs Colyseus (more batteries) — decide at Phase 0.
- LLM latency in rounds 3–5: generate the next round's content *during* the current round's vote/boni phases to hide latency. (Pre-generate ahead.)
- Reconnect handling for dropped phones — nice-to-have, likely skip for hackathon.
