# BLACKSITE — Tactical Roguelike

Single-file browser roguelike (`index.html`, ~1,600 lines: vanilla JS + CSS + inline SVG, no build step, no dependencies). Owner is a non-programmer game designer: explain changes in plain language, keep everything in the single file, and never break the save format without migrating it.

## Vision
Modern-military turn-based roguelike. Slay-the-Spire map structure, XCOM flavor. Replayability comes from: (1) branching dungeon paths, (2) build variety via classes/talents/gear, (3) permanent meta-progression (scrap → Armory unlocks). Depth over volume — a tight loop people replay, not sprawling content.

## Core systems (do not break these invariants)
- **Classes:** Marksman (crit), Mechanic (turrets/area control), Assault (burst/aggression, Armory unlock), Medic (sustain/lifesteal, Armory unlock). Each starts with ONLY its unique infinite-ammo pistol plus its signature skill (AIM / TURRET / ADRENALINE / TRIAGE) already active.
- **Skills:** active abilities in `SKILLS` with AP costs and cooldowns that PERSIST ACROSS FIGHTS (`P.cds`, tick down only during combat turns — deliberate anti-spam design, don't reset them per combat). Generic skills (Grenade, Spray, Flashbang, Camo, Field Patch, Mine) are drafted via post-fight Field Requisition offers — never granted automatically. Skill damage/healing scales with `RUN.tier` and is amplified by talent-tree nodes. Crowd control is capped: elites/bosses have `big=true` and are only dazed by Flashbang (next attack −50%), never fully stunned.
- **Progression split:** after each non-boss fight the reward modal offers ONE pick of 3 cards (new skill or small perk, `buildOffer`); level-ups grant 1 talent point (`P.tp`) spent in the talent tree (`TREE`, 3 branches + class SPEC OPS branch from Armory packs). Deeper tree rows need 2 points/tier spent in that branch. Tree ranks live in `P.tree`, read via `tr(id)`.
- **Bottom nav:** CHARACTER / INVENTORY / TALENTS panels available on map AND in combat (overlay, `openPanel`). Loadout is LOCKED during combat — `C.ammo` is keyed by item uid at combat start, so equipping mid-fight would break mags.
- **Ammo:** pistols infinite; all other weapons have per-combat mags (`mag` field). This is a core tension — never give non-pistol weapons infinite ammo.
- **Equipment slots:** pistol (LOCKED — only pistol-type weapons), w1, w2, ammo, helmet, chest, legs, boots, special. Gear grants base stats via `applyGear(item, ±1)` (symmetric on equip/unequip — keep it symmetric).
- **Rarity:** grey/green/blue/purple/orange (`RARITIES`), multiplies gear stats (`sm`) and weapon dmg (`wm`). Elites drop green+, bosses blue+.
- **XP per kill** (never per damage dealt — exploitable). Fast early levels: 12/18/26/36 then `16+15·level`.
- **Two in-run currencies:** gold (shop items, trainer stats, gunsmith mods) vs talent points (tree ranks). Keep these decision spaces separate.
- **Scrap** is meta currency, banked instantly (survives death), spent in Armory. Meta unlocks favor VARIETY (classes, talent packs, weapons) with only light stat pads — avoid the Rogue Legacy trap of balancing around meta stats.
- **Map:** 18 layers, `genMap()`; elites only layer ≥6; SVG connector lines between layers; first dungeon always Rust Yards, then choose between the other two at tier+1 after each boss.
- **Enemy scaling:** by dungeon tier AND map depth (`scaleE(base, tier, layer)`); bosses exempt from depth scaling (fixed endpoints).
- **Persistence:** dual-mode in `loadMeta/saveMeta` — `window.storage` (Claude artifact) → `localStorage` (standalone/GitHub Pages) → memory. Key `blacksite-meta`. META holds scrap, unlocks, runs, bestTier, kills, found (codex discovery).

## Code conventions
- Everything in `index.html`. Data tables at top of script (WEAPONS, GEART, RARITIES, CLASSES, SKILLS, TREE/SPEC_BRANCH, PERKS, ENEMIES, ELITES, DUNGEONS, ARMORY, CONSUMABLES).
- `wstat(item)` resolves weapon stats (base × rarity × gunsmith mods + P.mods.mag) — always attack via items, not raw WEAPONS entries.
- All sprites are inline SVG (`SPRITES`, `buildAvatar()` paper-doll that reflects equipped gear, `TURRET_SVG`, `WICONS`/`GICONS` small icons, `WART` detailed per-weapon side profiles tinted by rarity via currentColor, `DUNGEON_BG` per-dungeon battle backdrops set in `startCombat`). High contrast against the dark scene — an earlier turret was invisible because it was drawn in near-background colors. Backdrops must stay near-black silhouettes so figures read on top.
- Combat FX: `floatAt` damage numbers, `retrig(el, cls)` for animations, `_fx`/`_lunge` flags replayed after `renderCombat()` rebuilds DOM. Enemy turn is async/sequenced (`C.busy` locks input).
- Mobile-first: owner plays on iPhone. Big touch targets, portrait layout, battle scene top / actions bottom.

## Balance philosophy (hard-won through playtests)
Owner repeatedly found the player too strong. Enemies should take 3–4 turns to kill early. When adding power sources, compensate elsewhere. Camo+Flashbang chaining felt too safe and was nerfed three ways at once (2 AP each, cross-fight cooldowns, elite/boss stun immunity) — revisit if they now feel dead. Known watch items: Mechanic historically stronger than Marksman (turrets = free damage); boss difficulty after the 18-layer extension is unvalidated.

## Roadmap (owner's stated intentions, not yet built)
- Special abilities/affixes on gear (rarity variations beyond stat multipliers)
- More weapon models per icon type
- Possible rest-site/checkpoint node if 18-layer death loss feels brutal
- Eventually: app packaging (PWA/Capacitor)

## Testing
No test suite. After changes: syntax-check the script block, then the owner playtests on iPhone via GitHub Pages. Prefer small, reviewable diffs — the owner reviews from a phone.
