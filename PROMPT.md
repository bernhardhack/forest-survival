# Forest Survival — Design Spec

Single-file browser game. `index.html` at repo root, deployed via GitHub Pages.

---

## Design intent — read this before writing code

The game is about **scarcity and companionship** in a forest. There are no enemies. The player is alone with a small animal companion, gathering food and water, extending a camp, and surviving as many days as possible.

Three things must stay true, and every mechanic should serve them:

1. **Resources near the camp deplete permanently.** Nothing respawns. Every day the player must range further out. The camp gets stronger while the forest gets emptier, so the walk home gets longer and riskier. This generates tension without a single monster.
2. **The pet is both the solution to scarcity and a cause of it.** It finds resources the player can't see, and it eats from the player's stores. Never reduce it to a cosmetic follower.
3. **Every gain now costs something later.** Eat the plant or chop it for seeds. Drink the water or spend it on planting. Feed yourself or feed the pet.

If a feature doesn't serve one of those three, don't build it.

## Hard constraints

- Vanilla JavaScript, HTML5 Canvas 2D, **single self-contained `index.html`**. No frameworks, no bundler, no dependencies.
- **No external asset files.** All graphics drawn with canvas primitives, all sound synthesised with WebAudio.
- Desktop (keyboard + mouse) and mobile (touch) from the same build.
- Persistence via `localStorage`, wrapped in try/catch so it silently no-ops where unavailable.

## Vision and occlusion — the signature system

The forest is dense. **Trees physically block line of sight.** The player sees:

- A **cone of clear sight** in their facing direction, with every tree inside it casting a shadow that hides what's behind it.
- A small **ambient awareness circle** immediately around them.
- Beyond that, dim canopy-filtered gloom — shapes without detail.

This means the player's view changes constantly as they move between trunks, and sightlines open and close. It also means **the forest gets easier to see through as it's cut down** — an unsettling reward that ties directly to the depletion theme.

At night the cone narrows sharply and the gloom goes near-black. In rain, the cone shortens and mist thickens.

## Core loop

Two meters drain continuously:

- **Hydration** — drains faster. The pressing one.
- **Hunger** — drains slower, but food is harder to find.

When either empties, health drains. Health regenerates slowly only when both are above ~50%. Death ends the run and reports days survived.

The player leaves camp to gather with limited inventory slots, and must return before nightfall — at night it turns cold, visibility collapses, and being away from the fire drains health. Day/night cycle is the heartbeat: roughly 4 minutes day, 1.5 minutes night.

**Depletion is permanent.** Harvested berry bushes, drunk-dry seeps, and felled trees never return. Track them in world state.

## Harvest or chop — the central decision

Plants and trees can be used two ways, and the player chooses one:

- **Harvest** — food now. The plant is consumed and does not return.
- **Chop down** — **seeds** and wood, but no food. The tree is destroyed permanently and leaves a **stump**.

Make the two actions visually distinct in the interaction prompt so the player always chooses deliberately.

**Stumps stay forever.** They become the player's landmarks in a forest with no map, and a visible record of what they've taken. The ring of stumps around camp widening outward is the game's clearest image — render it and don't hide it.

## Seeds and growing

- Seeds are planted in the camp's plot and **watered from the player's own stores.** Water is the fastest-draining resource, so planting is a direct sacrifice of present survival.
- Seeds **grow overnight** and are harvestable the next morning. One night, one crop.
- An unwatered seed is wasted, not preserved. Watering must happen before sleep.
- Yield should be a thin margin — slightly more food than the water was worth. Planting is worth doing, never enough to live on.

**Guardrail:** seeds are finite and come only from felling trees. Crops never produce seeds, plots never reseed. The moment farming self-sustains, expeditions stop mattering and the game goes slack.

## The pet

Distinct silhouette, personality expressed through movement, never dialogue.

- Follows with its own pathfinding — pausing, sniffing, catching up in bursts. It should feel alive, not tethered.
- **Sniffing**: finds the nearest unharvested resource within roughly 3× the player's vision and trots toward it, looking back. Show a subtle directional cue when it has something.
- **Its own hunger meter.** Eats from the player's stores automatically when hungry.
- **Alerts**: reacts to incoming rain ~15 seconds early, and to nightfall.
- Can be told to **stay** at camp — no feeding cost on the trip, but no sniffing either.

| State | Behaviour |
|---|---|
| Healthy | Sniffs actively, quick, playful idles |
| Hungry | Stops sniffing, lags, slows |
| Starving | Won't follow, lies down |
| Dead | Permanent for the run |

While healthy it should find **more** food than it eats. While hungry it contributes nothing and still costs. Neglect must spiral on its own.

Its death is a real loss — pause, hold the moment, mark the spot permanently. Do not soften it. Do not offer a replacement.

## Weather

- **Rain**: refills water via the camp's collector and fills puddles worth drinking, but shortens vision, thickens mist, and accelerates cold at night. Good and bad at once.
- **Fog**: vision cone shortens dramatically. Navigation by stumps becomes essential.
- Telegraph both through the pet's reaction and a shift in colour and sound.

## Camp

Starts as a crude shelter. Built from wood and gathered materials:

- **Fire** — warmth at night and a lit radius; consumes wood continuously.
- **Rain collector** — passive water during rain only. Never enough alone.
- **Storage** — raises stockpile capacity.
- **Planting plot** — see above. Expandable in small increments.
- **Drying rack** — preserves food so it decays slower.
- **Pet bed** — the pet loses hunger slower overnight.

Balance so camp production **never** fully sustains the player. It stretches the time between expeditions; it never removes them.

## World generation

Procedural, seeded, seed visible and re-enterable. Value-noise driven density producing: dense woodland (many trees, few resources), clearings (berries, light), a stream corridor (water, follows terrain), and deep forest further out (best resources, hardest to navigate and return from). Better resources further from spawn. Large enough that day 20 demands genuinely long trips.

## Controls

- **Keyboard**: WASD/arrows move, mouse aims sight, `E` interact, `F` chop, `I` inventory, `B` build, `C` call pet, `Space` pet stay/follow, `` ` `` debug.
- **Touch**: left virtual stick moves, right stick aims, contextual action button that relabels by what's nearby. Minimum 64px targets, `touch-action: none`.
- Unified input state — game logic must not know the device.

## Rendering

- Fixed internal resolution scaled to viewport, aspect preserved, letterboxed, integer scaling where possible.
- Fixed timestep (60 Hz) with accumulator, decoupled from render.
- Palette: damp greens, moss, wet bark, grey mist. Firelight and the lamp are the only warm colours in the frame.

## HUD

Hydration, hunger, health, day counter, time of day, pet status, inventory. Minimal and out of the way — the gloom is the point, don't bury it under panels.

## Audio

WebAudio synthesis only: footsteps varying by ground, chopping, harvest, pet sounds (happy / hungry / alert), wind through canopy, rain, a low tone when a meter goes critical. AudioContext starts on first interaction.

## Build in stages — stop after each

1. **Milestone 1**: canvas, responsive scaling, fixed-timestep loop, movement (keyboard + touch), forest rendering, tree collision, vision cone with tree occlusion.
2. **Milestone 2**: world generation, terrain zones, resources, harvest/chop with permanent depletion and stumps, inventory.
3. **Milestone 3**: hydration/hunger/health, day-night cycle, camp shelter, death and run-end. **The game becomes playable here — stop and assess how the numbers feel.**
4. **Milestone 4**: the pet — following, sniffing, hunger states, alerts, death.
5. **Milestone 5**: seeds and planting, camp modules, weather, saves.
6. **Milestone 6**: audio, polish, palette pass, README.

All balance numbers live in a single `TUNING` object.

After each milestone, say what to test and what you'd tune next. Do not run ahead without confirmation.
