# Bonfire — planning document

Working title: **Bonfire** (was Campfire). Domain: **bonfire.observer** (Cloudflare, registered 2026-09-11 — do not buy again).
Repo: `nova-centauri/1280-hackyard-2-campfire`.
Status: Yard #2 **final primary**. Planning in this repo. DNS/placeholder via VPS-01. Product build not started from this vault.

BreezeVibe is no longer the weekend ship target. VibeShare stays backup/parked.

---

## 1. One sentence

A tab on your computer is a campfire. You start it, feed it, and try not to let it die — including overnight.

That’s the whole pitch. If you can’t say it in one breath, we overbuilt.

## 2. Why this one

BreezeVibe got muddy because it was two products (windows + house twin + HVAC). VibeShare is social and messy (strangers, video, vibe vs booze).

This is one object and one verb: **tend the fire**.

It’s a screensaver that can fail. It’s an idle game with physics. It’s a little live-time RPG with no quest log. People already understand campfires.

## 3. Core loop

1. Open the tab → cold pit, stone ring, night or day.
2. Build a startable pile (tinder → kindling → a few sticks).
3. Ignite (lighter / match / bow-drill). Fail if the pile is wrong or wet.
4. Feed it. Different fuel behaves differently.
5. It wants to die. Clock runs at **~2×**.
6. Score = how long it lived. Optional: wake up to coals still alive.

No menus. No HUD. The fire *is* the interface.

## 4. Time

- Simulation clock ≈ **2×** wall clock so a neglectful tab dies faster than a real fire.
- Tab visible: sim runs.
- Tab backgrounded: still runs (this is the overnight game), maybe at the same 2× or a slightly kinder 1× — pick one and stick to it. Default: **2× always**, even in background, so overnight is a real commitment (build a coal bed or it’s ash by morning).
- Pause only if the browser throttles so hard we have to; don’t add a pause button.

## 5. Fire model (simple math, not CFD)

Keep a tiny state machine + a few numbers. Not a fluid sim.

**State:** `unlit | smoldering | flaming | dying | dead`

**Quantities (floats):**

| Name | Meaning |
|---|---|
| `heat` | energy in the pit (0–1+) |
| `fuel` | burnable mass currently in the pile |
| `moisture` | water in the pile / incoming fuel |
| `oxygen` | smother vs draft (too much junk stacked = starve) |
| `smoke` | visibility / “this is a bad fire” |
| `coals` | leftover heat that can restart in the morning |

Each tick (~100–250ms):

```
heat    += burn(fuel, oxygen) - loss(weather, moisture) - time
fuel    -= consume(heat)
moisture-= boil_off(heat) ; incoming wet wood adds moisture
coals   += ember_from(heat) - decay
smoke   += treated_lumber + wet_wood + starve
```

Ignition succeeds if `fuel_structure` is valid (tinder present, not soaked) and ignition method meets a threshold:

- lighter: easy
- match: medium, wind/rain hurts
- bow-drill: slow, skill/patience, fails in wet

**Don’t** solve Navier–Stokes. Fake convection with upward particles and a heat shimmer.

## 6. Fuel & objects

Click / drag onto the pit. Each item is a prefab with mass, moisture, burn rate, smoke, hazard.

| Thing | What happens |
|---|---|
| Paper / leaves | lights easy, gone fast |
| Dry sticks / kindling | the real start |
| Dry split wood | heat + coals, the default good move |
| Wet wood | steals heat, hisses, smoke, may kill a small fire |
| Fat log | slow, needs an established bed |
| Pressure-treated lumber | ugly smoke, greenish, “buddy brought this” |
| Spray-can | dumb fun: jet / pop / maybe wreck the pit. Once. Don’t moralize; physics is the joke. |
| Water / rain | opposite of fuel |
| Too much too fast | smother (`oxygen` crash) |

Start kit on the ground around the pit so there’s no inventory UI: a little pile of paper, twigs, a few splits, a lighter. Other junk (wet wood, treated, can) is off to the side as temptation.

## 7. Starting the fire (first session)

Cold open. Dug pit, ring of stones, no flame.

The player has to:

1. Put tinder in.
2. Add kindling in a shape that can breathe (we can be generous: “enough sticks” not a perfect tipi tutorial).
3. Choose an ignition tool from the ground.
4. Hold / repeat until it catches (bow-drill = longer hold).

If they throw a log on a match, it dies. Teach by dying, not by a tooltip.

## 8. Weather & ground

- Clear, wind, rain. Rain is the main antagonist.
- Grass, dirt pit, wet stones when raining.
- Wind makes ignition harder and flames lean. Cheap.

No live weather API required (same lesson as BreezeVibe: canned climate is enough). Optional later: local rain from zip.

## 9. Overnight / live-time

The fantasy: leave the tab open, build a coal bed, go to bed, morning it’s still got glow.

Rules:

- A flaming fire left alone at 2× will be dead in a few real hours unless it’s all coals + a big log.
- “Ready in the morning” = `coals` above a threshold at local morning, not a quest flag.
- If they close the laptop, we persist state + wall-clock and simulate the gap on next load (don’t cheat a paused fire).

That’s the RPG. No XP. No levels. Duration + a morning coal check.

## 10. No interface

Hard rule: **no chrome**.

No buttons that say START FIRE. No fuel inventory panel. No settings cog on day one.

Allowed:

- The world (pit, tools on the ground, sky)
- Cursor / click / hold / drag
- Maybe a faint time-alive that appears only after death (gravestone: “4h 12m”)

If we need a mute, it’s a rock you flip, not a menu.

## 11. Visual quality bar (Three.js)

This *is* the demo. A brown cone with orange sprites fails.

Weekend-quality, not Pixar:

- **Fire:** volume-ish cards or mesh + noise, real glow (bloom), heat distortion, embers
- **Smoke:** particles, wet/treated = thicker/dirtier
- **Coals:** pulsed glow after flame dies
- **Grass:** instanced blades or good textured clumps, wet-darken in rain
- **Rain:** streaks + hits on stones/pit, steam when it hits hot stuff
- **Night:** the fire is the light. That’s the postcard.

Camera: one locked-ish beauty angle, slight orbit allowed. Not a free-fly cluttering the “no UI” rule.

## 12. Audio (generated, not a looped MP3)

Procedural or granular, tied to state:

- crackle / pop when dry wood hits a heat threshold
- hiss / steam when wet or rain hits coals
- low roar when flaming is fat
- treated lumber: nastier hiss
- spray-can: one ugly event
- silence when dead (that silence should hurt)

Start muted-until-gesture because browsers.

## 13. Weekend MVP vs later

**Ship this weekend if this becomes the stream:**

- One scene, one pit, Three.js
- Start puzzle + 2× clock + dry/wet wood + one treated + one spray-can
- Rain as a weather toggle in the *world* (clouds roll in), not a menu
- Persist coals across refresh
- Bloom fire + smoke + basic grass + crackle
- Death card with duration

**Cut:**

- Multiplayer “our fire”
- Account / cloud save beyond localStorage
- Real combustion chemistry
- Photo-real logs, scanned campsite
- Mobile app
- Zip-code live weather
- Bow-drill IK animation (a hold-to-spin is enough)

**Longer build:** seasons, animals, neighbors, smell jokes, share-a-link to a still-burning fire.

## 14. Demo beat (60–90s)

1. Cold pit. Hands add paper + twigs. Match. It almost dies.
2. Dry splits. Bloom, crackle, night looks good.
3. Wet log: hiss, smoke, dip.
4. Treated scrap: dirty smoke (laugh).
5. Spray-can: one bad idea.
6. Rain. Steam. Feed a fat log. Walk away. Come back — coals or ash.

If the room doesn’t want to stare at the flame, the visual bar failed.

## 15. Risks

| Risk | Why | Mitigation |
|---|---|---|
| Looks like a student particle demo | fire is the product | spend the weekend on look, not features |
| “Game with no UI” still needs onboarding | people won’t know to pick up sticks | objects are obvious on the ground |
| 2× + background tabs | browsers throttle timers | `visibility` + timestamp catch-up |
| Spray-can / treated lumber | jokes vs “don’t do this IRL” | no tutorial voice; consequences in-sim |
| Scope creep into survival craft | Steve can feel it | one pit, no hunger, no trees to chop |

## 16. Stack (when we build)

- Web, Three.js, one repo
- No backend for MVP (localStorage)
- When it ships: `bonfire.observer` on VPS-1, same webhook-pull pattern as BreezeVibe

## 17. Commanding prompt (paste when Steve says go)

```
Build a no-UI campfire you tend in a browser tab.

Product: a dug pit + stone ring. Player starts a fire (tinder, kindling, sticks) then keeps it alive. Clock runs at 2× real time, including when the tab is in the background (simulate the gap on return). Goal: longest fire, including overnight coals.

Look is the product. Three.js. Good fire (glow, bloom, embers, heat shimmer), smoke, rain, grass, wet stones. Night lit by the fire. Generated audio: crackle, pop, hiss, steam. No HUD, no menus, no inventory panel. Tools and fuel sit on the ground. Click/hold/drag only.

Fuel changes the fire: dry wood, wet wood (hiss/smoke), pressure-treated (dirty smoke), spray-can (one dumb event). Too much fuel smothers. Rain is the antagonist.

Ignition: lighter (easy), matches (medium), bow-drill (hold-to-spin, hard when wet).

Simple math: heat, fuel, moisture, oxygen, smoke, coals. States: unlit / smoldering / flaming / dying / dead. Not CFD.

Persist state + wall clock so overnight/refresh is honest.

Weekend cuts: no accounts, no mobile app, no live weather API, no multiplayer, no survival crafting.

Demo: cold start → catch → good night fire → wet + treated + can → rain → coals or ash. Death shows duration only.

Success = someone stares at it and then tries not to let it die.
```

## 18. Yard vs longer

**Yard project** if we keep one pit and spend the hours on fire/rain/audio.

**Longer build** if we add a forest, crafting, friends, or a real combustion thesis.

## 19. Lane

| Stream | Repo | Role |
|---|---|---|
| Bonfire | this repo + bonfire.observer | Yard #2 **final primary** |
| BreezeVibe | `1280-hackyard-2` / breezevibe.site | live, no longer the weekend ship |
| VibeShare | `1280-hackyard-2-vibeshare` | backup, parked |
