# ThreedyTycoon — Vision Art Prompts

Prompts for **ChatGPT Image 2.0**. The plan: generate the **same anchor scene in 5 style directions** so we can compare looks and lock one. Only the *style* paragraph changes between prompts — the scene is held constant for a fair comparison.

**Tone (locked):** a passionate, energetic team *building a software product together*. Five strong personalities, each pulling toward what they care about — the comedy is in the clash of perspectives and the surprising places the product ends up, **not** in anyone being incompetent or the team being dysfunctional. Think "lively startup with big opinions," not "the office that can't ship."

---

## Shared scene (baked into every prompt)

> Five characters representing a software startup team, mid-collaboration and full of energy: a **Developer** (hoodie, headphones, sticker-covered laptop, in the zone), a **CEO** (blazer over a band tee, gesturing at a big idea), a **Salesperson** (sharp, confident, mid-pitch with a grin), an **HR** lead (warm, grounded, holding a "team morale" mug, keeping everyone together), and a **Product Manager** (sticky notes and a roadmap, connecting the dots between everyone). They're gathered around one desk with a glowing monitor, clearly building something together — collaborative and spirited. Title text **"THREEDYTYCOON"** integrated into the composition. Landscape 16:9.

---

## 1 — Balatro-style neon CRT roguelike
> Vision key art for a roguelike party game called **ThreedyTycoon**. [Shared scene.] Render it as moody retro-arcade card-game art: deep navy-black background, glowing neon accents in cyan, magenta and gold, glossy playing-card-style panels framing each character, subtle CRT scanlines and a soft vignette, chunky retro UI chrome. High contrast, dramatic rim lighting — "Balatro meets a software startup." Bold arcade title treatment for "THREEDYTYCOON." 16:9.

## 2 — Flat vector startup energy (Jackbox vibe)
> Vision key art for a multiplayer party game called **ThreedyTycoon**. [Shared scene.] Bright, bold flat-vector illustration with confident thick outlines, geometric shapes, expressive upbeat cartoon characters, punchy saturated color palette, clean modern UI accents. Playful, friendly, high-energy "Jackbox party game" feel. Big friendly title "THREEDYTYCOON." 16:9.

## 3 — Stylized 3D isometric tycoon office
> Vision key art for a 3D tycoon party game called **ThreedyTycoon**. [Shared scene, arranged in:] a charming stylized low-poly isometric software-startup office diorama — warm soft lighting, rounded friendly 3D shapes, "Two Point Studios / The Sims" tycoon-sim charm, lots of tiny detailed props (monitors, sticky notes, a foosball table, a thriving office plant). Cozy, characterful, lively. Stylized 3D title "THREEDYTYCOON." 16:9.

## 4 — Hand-drawn comic / editorial-cartoon
> Vision key art for a satirical roguelike party game called **ThreedyTycoon**. [Shared scene.] Hand-drawn comic / editorial-cartoon style: loose confident ink linework, expressive caricature faces full of personality, watercolor-and-marker shading, visible texture and energy, New Yorker-cartoon-meets-graphic-novel feel. Witty and warm — the joy and drama of building something. Hand-lettered title "THREEDYTYCOON." 16:9.

## 5 — Threedy brand-aligned (premium product look)
> Vision key art for a digital game called **ThreedyTycoon**, in the visual language of a premium modern software product. [Shared scene, stylized restrained:] dark-gray surfaces (never pure black), crisp clean composition, Nunito Sans typography, restrained and confident, benchmarked against Linear / Framer / Arc — minimal, polished, sophisticated, with one or two tasteful accent-color highlights. Looks like a real Threedy product, not a generic game. Sleek title "THREEDYTYCOON." 16:9.

---

## Workflow

1. Run all 5. Pick a winner (or a blend) → that becomes the locked style for the actual UI.
2. Round 2: regenerate the winning style across **3 scenes** to pressure-test it holds across the whole game:
   - **The voting moment** — host screen showing two candidate answers, players voting on phones, the table leaning in.
   - **The LLM verdict reveal** — the fake tech-press headline + Software Score; the shareable payoff.
   - **Host + phones setup** — the Jackbox topology (shared screen + everyone's phones as controllers).
3. If characters need to stay recognizable across scenes, generate **character reference sheets** for the 5 roles first and feed them back in.

## Tips for ChatGPT Image 2.0
- It handles long natural-language scene descriptions well — keep the scene specific and the style paragraph distinct.
- It renders in-image text reasonably; spell "THREEDYTYCOON" in caps and state where it goes.
- Generate one style at a time for best fidelity; ask for variations on the winner rather than all 5 in one image.
