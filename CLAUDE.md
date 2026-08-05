# BLACKSITE — Tactical Roguelike

Single-file browser roguelike (`index.html`, ~1,600 lines: vanilla JS + CSS + inline SVG, no build step, no dependencies). Owner is a non-programmer game designer: explain changes in plain language, keep everything in the single file, and never break the save format without migrating it.

## Vision
Modern-military turn-based roguelike. Slay-the-Spire map structure, XCOM flavor. Replayability comes from: (1) branching dungeon paths, (2) build variety via classes/talents/gear, (3) permanent meta-progression (scrap → Armory unlocks). Depth over volume — a tight loop people replay, not sprawling content.

## Core systems (do not break these invariants)
- **Classes:** Marksman (crit), Mechanic (turrets/area control), Assault (burst/aggression, Armory unlock), Medic (sustain/lifesteal, Armory unlock). Each starts with ONLY its unique infinite-ammo pistol; class signature skill (AIM / TURRET / ADRENALINE / TRIAGE) is guaranteed to appear in the first level-up offer.
- **Ammo:** pistols infinite; all other weapons have per-combat mags (`mag` field). This is a core tension — never give non-pistol weapons infinite ammo.
- **Equipment slots:** pistol (LOCKED — only pistol-type weapons), w1, w2, ammo, helmet, chest, legs, boots, special. Gear grants base stats via `applyGear(item, ±1)` (symmetric on equip/unequip — keep it symmetric).
- **Rarity:** grey/green/blue/purple/orange (`RARITIES`), multiplies gear stats (`sm`) and weapon dmg (`wm`). Elites drop green+, bosses blue+.
- **XP per kill** (never per damage dealt — exploitable). Fast early levels: 12/18/26/36 then `16+15·level`.
- **Two in-run currencies:** gold (shop items, trainer stats, gunsmith mods) vs talents (level-up picks). Keep these decision spaces separate.
- **Scrap** is meta currency, banked instantly (survives death), spent in Armory. Meta unlocks favor VARIETY (classes, talent packs, weapons) with only light stat pads — avoid the Rogue Legacy trap of balancing around meta stats.
- **Map:** 18 layers, `genMap()`; elites only layer ≥6; SVG connector lines between layers; first dungeon always Rust Yards, then choose between the other two at tier+1 after each boss.
- **Enemy scaling:** by dungeon tier AND map depth (`scaleE(base, tier, layer)`); bosses exempt from depth scaling (fixed endpoints).
- **Persistence:** dual-mode in `loadMeta/saveMeta` — `window.storage` (Claude artifact) → `localStorage` (standalone/GitHub Pages) → memory. Key `blacksite-meta`. META holds scrap, unlocks, runs, bestTier, kills, found (codex discovery).

## Code conventions
- Everything in `index.html`. Data tables at top of script (WEAPONS, GEART, RARITIES, CLASSES, TALENTS, ENEMIES, ELITES, DUNGEONS, ARMORY, CONSUMABLES).
- `wstat(item)` resolves weapon stats (base × rarity × gunsmith mods + P.mods.mag) — always attack via items, not raw WEAPONS entries.
- All sprites are inline SVG (`SPRITES`, `buildAvatar()` paper-doll that reflects equipped gear, `TURRET_SVG`, `WICONS`/`GICONS`). High contrast against the dark scene — an earlier turret was invisible because it was drawn in near-background colors.
- Combat FX: `floatAt` damage numbers, `retrig(el, cls)` for animations, `_fx`/`_lunge` flags replayed after `renderCombat()` rebuilds DOM. Enemy turn is async/sequenced (`C.busy` locks input).
- Mobile-first: owner plays on iPhone. Big touch targets, portrait layout, battle scene top / actions bottom.

## Balance philosophy (hard-won through playtests)
Owner repeatedly found the player too strong. Enemies should take 3–4 turns to kill early. When adding power sources, compensate elsewhere. Known watch items: Mechanic historically stronger than Marksman (turrets = free damage); boss difficulty after the 18-layer extension is unvalidated.

## Roadmap (owner's stated intentions, not yet built)
- Special abilities/affixes on gear (rarity variations beyond stat multipliers)
- More weapon models per icon type
- Possible rest-site/checkpoint node if 18-layer death loss feels brutal
- Eventually: app packaging (PWA/Capacitor)

## Testing
No test suite. After changes: syntax-check the script block, then the owner playtests on iPhone via GitHub Pages. Prefer small, reviewable diffs — the owner reviews from a phone.
