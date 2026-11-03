# Straumr

**Living water for Valheim.** Rivers flow downstream, rapids run white, the sea has currents and tides, and whirlpools turn off the ends of tidal straits. Swimmers, boats and floating things all ride it.

*Straumr* is Old Norse for "current". This is a **beta** (0.1): it works, it's been played on, and it's still being tuned. Feedback welcome.

## New in 0.1.2

- **Truer river flow:** far fewer spots where water pours out of a point or vanishes into one (see *How it was checked*); lakes now drain through their real way out; the long way round a loop, or a lake's second way out, drifts along instead of standing dead.
- **Ripples that move with the current:** a thin sheet on the water carries ripples and sun glints along with the current. New, and still being tuned (see *The water's surface* below).
- **Foam** fades where it meets a bank or a rock instead of ending on a hard line.
- **Rocks** bend the current by the part of them that's in the water, not by their whole size: a big rock leaning on a bank no longer stills the river across all of it.
- **Wind** drifts lakes and broad slow reaches too, not only the sea, and a storm pushes harder than a breeze (`[Currents] StormDrift`).
- **Storm surge:** a storm raises the sea a little on top of the tide, up to 0.3 m in a full storm (`[Currents] StormSurge`).
- **Waves carry floating things** the way they run, turning in toward the beach over the shallows, so with the wind onshore driftwood, wreckage and felled logs wash up (`[Currents] WaveDrift`).
- **Felled logs float** and drift with the current and the waves: timber can be floated down a river (`[Currents] LogDrift`).
- **Wading:** the current pushes you downstream as you wade, more the deeper and faster it runs (`[Currents] Wading`).
- **Undertow:** where a river runs fast and smooth through a deep, narrow slot, it now and then pulls swimmers under for a moment (`[Currents] Undertow`).
- **Swimming effort,** optional and off by default: `[Currents] SwimWithCurrent` and `SwimAgainstCurrent` make swimming with or against the current cost less or more stamina.
- **A boat left to drift** in a river slowly swings round to lie along the current (`[Currents] BoatYaw`).
- **A boat left empty** (not moored) rides as if at anchor: it stops where it was left, swings a couple of metres either way, and in a river turns to lie along the current; board it and it's free (`[Currents] AnchorEmptyBoats`).
- **Steering in a bend is easier:** the current's turn on a boat is halved while someone is at the helm.
- **Boat speed** under the minimap while you're aboard: through the water, over the ground, and what the current adds or takes (`[Hud] BoatSpeed`).
- **Fixed:** in a blow, the current dropped out on every wave crest more than 30 cm above the still sea (Straumr took the crest for another mod's water). Swimmers, boats, floating things and foam lost the current there, and the API's `StraumrApi.Velocity` read zero.
- **River sound** starts at slower water and grows fuller the deeper the river, so a big river sounds like one.
- `straumr here` names the nearest rocks, logs, posts or walls bending the current where you stand.

## What it does

**Rivers**
- Every river flows from source to mouth, faster where it narrows, slow where it widens into a lake, and slower near the banks than mid-stream. Creeks run about 1 m/s and great rivers about 1.3 m/s (you swim about 2).
- Water slides along the banks and round islands instead of running into them, and carries on out past a river's mouth before it fades.
- Where several rivers share the same water (Riverheim often runs a few through one channel, sometimes in opposite directions), that water gets one way down: its shortest way to the sea. Rivers keep the direction Riverheim drew unless the other way is less than half as long, so they rarely run back toward the hills.
- A river that ends in a lagoon or bay runs out through it to the sea.
- **White water** where fast water runs shallow, worked out from the Froude number (how fast the water runs for how deep it is), the way real rapids form. White water shoves swimmers about, now and then pulls them under for a moment, and wears them out.
- **Undertow:** where a river runs fast through a slot, a channel deep for its width, its surface stays smooth while the water below now and then pulls a swimmer under for half a second or so, with at least 15 s between pulls in the worst slot (40 s in a mild one). Only true slots: from three times as wide as deep, 2 m deep and 1 m/s, in full at one and a half times as wide as deep, 3 m deep and 2 m/s. It fades out as the water turns white, which handles swimmers its own way. It costs no stamina and never takes anyone more than 1.5 m below where they swim.
- **Wading:** walking in a current over your ankles, the water pushes you downstream, more the deeper it is and with the speed squared. At a usual river's 1.2 m/s your feet hold up to about 0.6 m deep, and a metre deep carries you about 0.2 m/s; in a 2 m/s run, half a metre deep carries you 0.2 m/s and three quarters of a metre 0.6 m/s. Walking upstream is slower going. It never takes more than half your slowest walking pace, so you can always walk out.
- Swimming costs the usual stamina whichever way you swim, as in real water: the current changes where a stroke takes you, not what it costs. `SwimWithCurrent` and `SwimAgainstCurrent` can change that (0.85 makes going with a river a little easier, 1.25 fighting it a little harder).
- Rocks, logs, posts and walls that break the surface bend the current, with slack water and a small back-eddy behind them; the water stalls against their upstream face. A rock counts only as big as the part of it the water meets at the surface, so a big rock leaning on a bank turns the water round its toe, not across its whole bulk. A stone just under the surface doesn't stop it. A wall or dam across a channel stops the flow, and the water above it lies still.
- Bends push surface water to the outside bank, shallow rivers alternate pools and riffles, and side rivers meet in a swirling seam.
- **Weather and seasons:** rain swells the creeks first and the big rivers later; a storm swells them more. With Seasonality, spring snowmelt runs high and winter runs low (or freezes, if Seasonality freezes the water).
- Dig a channel below the water line between two waters and the current runs through it.

**The sea**
- Wind-driven currents run along the coasts, through the sounds and between islands; shallow swamps and marshes stay mostly calm, apart from the wind's drift on their broader pools. Rips run out from open beaches, and longshore drift runs along them.
- **Wind** drifts the surface of the sea, and of lakes and broad slow reaches too: from about 8 m off the shore, fully from about 60 m out. Running channels don't take it. A storm pushes harder than a breeze.
- **Waves** carry floating things slowly the way they run: downwind on open water, turning in toward the beach over the shallows, so with the wind onshore driftwood, wreckage and felled logs wash up. In a full gale that's about 0.05 m/s far out, up to 0.3 m/s over about 10 m of water and 0.07 to 0.1 m/s in the last metre or two; a quarter of that at half wind. Small boats with someone aboard take a share of it, a longship none. In a strong wind the shallows turn to surf, which pushes swimmers the way the waves run. None in a river's channel.
- **Tides:** the sea rises and falls up to 0.6 m either side of its usual level twice a game day (0.8 m at spring tides). Bays behind narrow mouths rise less and later, and their mouths race in and out (about 1.5 m/s, 2 m/s at spring tides). Up river mouths the flood pushes against the river and the ebb speeds it out. Slack water comes at high and low tide, a little later each day, and the moon sets the size: spring tides at full and new moon, gentle neaps between. With Seasonality, each season is one lunar month. A storm raises the sea a little more on top (up to 0.3 m in a full storm), as far up rivers as the tide reaches.
- What Valheim checks against the water follows the tide: swimming, boats, floating things, building damage under water, placing water pieces, fires going out, and creatures deciding where the water is.
- **Whirlpools** spin up off the ends of tidal straits near the peak of each tide. Mind the eye.
- A line under the minimap tells you the tide near any coast: *"Flood tide · full · high water in 6 min · +0.1 m"*.

**The water's surface** (new in 0.1.2, still being tuned)
- Ripples and sun glints on the water move with the current. Straumr lays a thin see-through sheet on the water round you, drawn with its own shader: it rises and falls with the game's waves and the tide, and the current carries its ripples along. It only adds glints and the sky's light on the ripples; it paints nothing white. Still water doesn't show it, a slow river (from 0.05 m/s) shows it faintly and water running 0.5 m/s or more shows it in full; on open water in a wind it fades to half, leaving most of the sea to the game's own waves; and it fades out by about 60 m from you.
- The shader is built into `Straumr.dll` for each platform: Windows (which also covers Proton), Linux and Mac.
- It switches itself off where it can't run (no shader for your platform, or a graphics card that can't run it), when NorgerWater is installed, while Seasonality has the water frozen, on Rivers and Lakes' water, and toward the Deep North, where the game calms its big waves.
- `straumr overlay` in the console turns it off and on in your game, to compare, and says whether it's on, and why if it couldn't start at all. Its settings are under `[Surface]`.

**Life on the water**
- **Foam** shows which way and how fast the water goes: a few scraps on all water (still water too), much more where it runs, shears and meets, in rafts, lace and strands drawn out by the current, and white water with streaks and spray. It fades where it meets a bank or a rock instead of ending on a hard line (`[Flecks] SoftEdges`), and `[Flecks] Lift` sets how high it floats. Rivers murmur, fuller the deeper they run, and rapids roar (sounds made from the physics of bubbles, with no recordings), and mist lies on the water at dawn.
- Driftwood washes up in bends, and fish hold in the pools.
- **Felled logs float**, about two thirds under, and drift with the current and the waves: rolled into a river, a log is carried off downstream, pivoting round an end that grounds and lining up with the stream. Logs don't jam or change the water.
- Boats feel the current while someone's aboard but always keep some headway, and a boat stopped against a bank or dock stays moored.
- A boat left to drift in a river (someone aboard, no one at the helm, sail furled and oars shipped, not moored) slowly swings round to lie along the current, bow or stern downstream, whichever is nearer: across a 1 m/s current it comes round in about half a minute to a minute, quicker in faster water. It fades out on open water.
- A line under the minimap while you're aboard a boat gives its speed through the water and over the ground, and what the current does along its heading: *"Boat 2.4 m/s through water"*, *"3.1 m/s over ground"*, *"current +0.7 (helping), running NE"*, one under the other. The current there is the share of it that carries boats (`[Currents] Boats`), so the numbers match what the boat does. Valheim's own boats only, so far.
- Ruins and other stonework the world placed don't crumble where a steep world leaves their edge hanging (`[World] StandingRuins`).

## Works with

Straumr runs on its own, on Valheim's own worlds, and detects these automatically when they're installed. None is required.

- **[Riverheim](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/):** worlds built around big river networks, which Straumr sets flowing.
- **[Rivers and Lakes](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/):** streams, lakes and waterfalls above sea level. Works side by side with Straumr today: Straumr leaves its water to it. With its next update, its streams carry on as jets into still water and its bigger lakes drift with the wind.
- **[Seasonality](https://thunderstore.io/c/valheim/p/RustyMods/Seasonality/):** seasons, snowmelt and freezing.
- **[ValheimRAFT](https://thunderstore.io/c/valheim/p/zolantris/ValheimRAFT/):** its vehicles ride the current too.

**NorgerWater:** when it's installed, Straumr's moving surface switches itself off and leaves the water's surface to it. Straumr hasn't been tested with it in game yet.

## For mod makers

Straumr has a public API for other mods (water wheels, mills, fishing, sailing AI, current charts): it reports the water's physical properties and leaves the gameplay to you. `StraumrApi.Sample(position)` gives the current, depth, tide and more in one call. Soft-depend on `wackah.straumr` and your mod works with or without Straumr installed. The full guide, with examples and its limits, is **`API.md`** in the download.

## Multiplayer

Install Straumr on the server and on every player's game. The server's physics settings are sent to everyone who joins, and players get their own back when they leave: all of `[Currents]` and `[Stage]`, the white-water thresholds (`WhiteFroude`, `WhiteFroudeFull`, `WhiteMinSpeed`) and `[World] StandingRuins`. How the water looks and sounds stays each player's own choice. Tides run off the server's clock, so everyone has the same tide.

## The flow map and seed atlas

- `straumr atlas` in the console (F5) opens a page from your own game at `http://localhost:8765`.
  - Type any seed: your game builds that world in memory (nothing is saved) and shows its rivers, sea currents, tides and whirlpools.
  - It starts whole-world, and fills in real detail as you zoom. Press **D** for a debug view of why the water does what it does at any spot, and **T** for the tide.
  - It shows the whole map, so don't type a seed you want to keep a surprise (or list it in `[Atlas] HiddenSeeds`).
  - Only that page can read it (a key it gets from the game), and it closes a few seconds after you close the page.
- `straumr map` writes the world you're in to a file the same page can open.

## Console (F5)

| Command | What it does |
|---|---|
| `straumr here` | the water at your feet, each part of the current (river, tide, sea, rocks...), what nearby is bending it, and its way to the sea |
| `straumr report` | writes a file describing the water where you stand, including what's standing in it, for bug reports |
| `straumr selftest` | checks Straumr is working right where you stand (each check PASS, WAIT, SKIP, NOTE for a known limit, or FAIL) |
| `straumr waterlevel` | the sea level here: the usual level, the tide, and the water's actual surface |
| `straumr perf` | how hard Straumr is working (turn on `[Diagnostics] PerfCounters`) |
| `straumr overlay` | turns the moving surface off and on in your game, to compare, and says whether it's running |
| `straumr atlas` | the seed atlas in your browser (works from the main menu too) |
| `straumr tide` | how the tide is running and when it turns* |
| `straumr tide flood` | hold the tide (`flood`, `ebb`, `high`, `low`, `spring`, `neap`) in your game; `straumr tide auto` lets go* |
| `straumr races` | the strongest tidal races and whirlpools within 4 km* |
| `straumr map` | write this world's flow map to a file* |
| `straumr waterstep` | a test: steps the water up zone by zone near you; `straumr waterstep off` undoes it* |

\* needs `devcommands` first, or admin rights on a server.

## Settings

Everything is in `BepInEx/config/wackah.straumr.cfg`, and most settings apply within a couple of seconds. **Shared** settings come from the server when you join one (yours apply again when you leave); the rest are each player's own. Settings marked *server* only do anything on the server.

**`[Currents]`** (shared): the water's physics.

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | true | All of Straumr's currents. |
| `Speed` | 1.2 | The typical river's current (m/s). Creeks and great rivers scale from it. |
| `MaxSpeed` | 3 | No river runs faster than this (m/s). |
| `VanillaSpeed` | 0.7 | Worlds without Riverheim: Valheim's own rivers, times the usual. |
| `OceanSpeed` | 0.8 | The typical open-sea current (m/s). 0 = none. |
| `Tides` | 1.5 | How fast the tide races through narrow sea channels (m/s). 0 = no tidal streams. |
| `TideRange` | 0.6 | How far the sea rises and falls either side of its usual level (m). 0 = no tide. |
| `StormSurge` | 0.3 | How far a full storm raises the sea on top of the tide (m), growing with the wind squared. 0 = off. |
| `SpringNeap` | 0.35 | How much bigger spring tides are, and neaps smaller (0 = every day alike). |
| `TidalMonth` | 16 | Without Seasonality: game days from one full moon to the next. |
| `Whirlpools` | 1 | How hard whirlpools spin (0 = none). |
| `Rapids` | 1 | How hard white water handles swimmers (0 = not at all, up to 2). |
| `Undertow` | 1 | How hard a river's slot pulls swimmers under now and then: above 1 harder, not more often (0 = never, up to 2). |
| `Rips`, `Longshore` | 0.8, 0.5 | Rip currents off open beaches, and drift along them. |
| `WindDrift` | 0.5 | Surface drift downwind at full wind (m/s), on roomy slack water: the sea, a lake's middle, a broad slow reach. |
| `StormDrift` | 0.3 | Extra drift in a storm (m/s at full wind), on the same water, growing with the wind squared: a breeze adds almost nothing. |
| `WaveDrift` | 1 | How strongly the waves carry floating things, small boats a little, and swimmers in the surf (1 = the drift Valheim's own waves would carry; 0 = none, up to 3). |
| `Players`, `Mobs`, `Items`, `Boats` | 1, 1, 1, 0.6 | How strongly the current carries each (0 = not at all). |
| `Wading` | 1 | How hard the current pushes players wading in it, never more than half your slowest walking pace (0 = off, up to 2). |
| `SwimWithCurrent`, `SwimAgainstCurrent` | 1, 1 | Swimming with or against the current costs this times the usual stamina, in full swimming straight along a current as fast as you swim (1 = the same; 0 to 3). |
| `LogDrift` | true | Felled logs float and drift with the current and the waves (off = logs as Valheim has them). |
| `BoatMinSpeed` | 3 | A boat under sail or oar always keeps this much headway (m/s). |
| `BoatTurn` | 0.35 | How much differing currents turn a boat (0 = never; half that while someone steers). |
| `BoatYaw` | 1 | How readily a boat drifting free swings round to lie along the current (0 = never, up to 3). |
| `AnchorEmptyBoats` | true | A boat left empty (not moored) holds where it was left, swinging a couple of metres, and in a river turns along the current; boarding frees it. |
| `MooringReach` | 2.5 | A stopped boat this close to a bank, dock or rock stays moored (m). |
| `Obstacles`, `Bends`, `Riffles`, `Seams`, `Boils`, `Windrows`, `Inflows` | true | The individual effects: rocks bending the water, bend flow, pools and riffles, junction seams, boils, foam lines in the wind, Rivers and Lakes stream jets. |
| `CreaturesFollowTide` | true | Creatures find the water's edge where the tide has it now. |

**`[Stage]`** (shared): rivers rising and falling.

| Setting | Default | What it does |
|---|---|---|
| `Seasons` | true | With Seasonality, the rivers follow the seasons. |
| `Spring`, `Summer`, `Fall`, `Winter` | 1.4, 1, 0.8, 0.7 | How fast rivers run mid-season, times usual. |
| `Snowmelt` | 0.35 | In spring, how much faster snow-fed rivers run at the afternoon peak than at dawn. |
| `Weather` | true | Rain swells the rivers: creeks at once, big rivers later and longer. |
| `RainSwell`, `StormSwell` | 0.3, 0.6 | How much faster after a good rain, and after a storm. |
| `IceFreeSpeed` | 0 | With Seasonality's freeze: water at least this fast (m/s) stays open. 0 = off. |

**`[Flecks]`** (your own; the three `White...` thresholds are shared): the foam.

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | true | Foam on the water. |
| `Rate` | 240 | How much foam (how often the water round you is looked at). |
| `MaxFlecks` | 1500 | Most flecks at once. |
| `Radius` | 60 | Foam within this distance of the camera (m). |
| `Size` | 0.4 | Typical fleck size (m). |
| `Lifetime` | 5 | How long a fleck lasts (s): longer drifts and gathers, shorter reads as strokes of current. |
| `CalmSpeed` | 0.3 | Foam thickens from this speed up (m/s). While the moving surface shows, slow water (below twice this) and slow open water get half the foam their speed would make. |
| `Opacity`, `Brightness` | 0.6, 2 | How solid and how bright the foam looks. |
| `Rafts` | 1.5 | How many flecks gather in each raft (0 = single flecks). |
| `WhiteWater` | true | Fast water looks white (only the look). |
| `Spray` | 0.3 | How often spray kicks up off the fastest white water. |
| `SoftEdges` | true | Foam fades where it meets a bank or a rock instead of ending on a hard line (Straumr's own foam shader, only where the game already draws the scene's depth). |
| `Lift` | 0.04 | How far the foam floats above the water's surface (m). Raise it a little if waves hide the foam. |
| `WhiteFroude`, `WhiteFroudeFull`, `WhiteMinSpeed` | 0.45, 0.9, 0.8 | When water turns white: from this Froude number, fully white at this one, never below this speed (m/s). |

**`[Surface]`** (your own): the moving surface.

| Setting | Default | What it does |
|---|---|---|
| `FlowOverlay` | true | The water's ripples and glints move with the current. Off by itself where it can't run, and under NorgerWater. |
| `Ripples` | 0.55 | How steep the moving ripples are (0 = flat, 1 = choppy). |
| `Glints` | 0.5 | How bright the sun's glints on them are (0 = none). |
| `SkyLight` | 0.4 | How much the ripples show the sky's light (0 = none). |
| `RippleSize` | 5 | How big the ripple pattern is before it repeats (m). |
| `FlowPeriod` | 1.8 | How long each copy of the ripples is carried before it fades and restarts (s): longer stretches them more on fast water, shorter pulses more. |
| `Lift` | 0.04 | How far the sheet sits above the game's water (m), so the two don't flicker through each other. |
| `SoftEdges` | true | Fade the sheet where it meets banks and rocks (only where the game already draws the scene's depth; turn it off if the sheet vanishes). |

**`[Hud]`** (your own): lines under the minimap.

| Setting | Default | What it does |
|---|---|---|
| `TideClock` | true | While you're by tidal water: how the tide runs and when it turns. |
| `BoatSpeed` | true | While you're aboard a boat: its speed through the water and over the ground, and how much the current adds to it or takes from it along its heading. |

**Your own, the rest of the look:** `[Sound]` `Enabled`, `Volume`; `[Mist]` `Enabled`, `Amount`.

**What the world gets** (*server*): `[Driftwood]` `Enabled`, `EveryMinutes`, `Prefab`; `[Fish]` `Enabled`, `EveryMinutes`, `PerSpot`, `Kinds`; `[Rocks]` the rocks set at swallow holes and river sources.

**Admin and tools:** `[World]` `StandingRuins` (shared) and `NewWorldSeed` (every new world this game creates gets this seed; made for dedicated servers, which can't be given one); `[Atlas]` `Port`, `HiddenSeeds`; `[Diagnostics]` `CurrentProbe`, `PerfCounters`, `LocationCheck`, `WorldGenCommand` (testing tools, off by default).

Upgrading from 0.1.0: if your foam settings are still 0.1.0's untouched defaults, they move to the new ones by themselves; anything you changed stays as you set it.

## Known limits (beta)

- Valheim's own water surface doesn't move with the current. Straumr's moving surface lays ripples and glints on it within about 60 m of you, mostly on running water; beyond that, on slack water, and wherever the moving surface is off, the foam and the things floating on it show the flow.
- In a wide bay or sea arm with several river lines far apart, the current can swirl oddly between them, or drift across toward a river a couple of hundred metres off instead of along the one on the map.
- Where two ways to the sea through the same water are nearly equal, the water splits between them and lies slack where they part (a faint spring there is now rare).
- A lake no river runs into lies still apart from the wind, and so can broad water far from any river.
- Damming a river: ground raised across it with the hoe stills the water above it, but the river below keeps running as before (Straumr doesn't yet know how much water a river carries), and the water doesn't rise behind it. Built walls and stones bend the water like rocks and leave calm water behind them, but don't dam it.
- A few things Valheim keeps at its fixed sea level don't move with the tide: mist, embers, the beach's shading, and where the world placed things.
- The sea is one flat sheet per 64 m zone, raised as a whole, so neighbouring zones can differ by a few centimetres.
- Rivers and Lakes' own streams and lakes move as that mod draws them. Straumr adds its stream jets and the wind's drift on its lakes once Rivers and Lakes shares its water with other mods (its next update).
- ValheimRAFT's vehicles ride the current, but the boat speed line, the current turning a boat (`BoatTurn`, `BoatYaw`), the anchor for empty boats and the waves' drift on boats cover Valheim's own boats only so far.
- Worlds much bigger than Valheim's standard size aren't supported yet.

## Roadmap

Plans, not promises; the next steps depend on what players find.

- **Next:** rivers that carry the water their land drains, so side rivers add up and a river that splits shares its water by what each branch can carry.
- **Later:** stones just under the surface showing a riffle, river levels rising, not only running faster, after storms and in spring, fish that hold in seams and behind rocks.
- **Further on:** water that remembers what happened: lakes filling and draining, a log jam backing a river up and then bursting.
- **Not planned:** new models or textures (water wheels, mills, nets). Those belong to other mods, built on the API.

## How it was checked

0.1.2 was measured against 0.1.1 on one Riverheim world, in 214 squares of 256 m along its rivers, lakes and junctions (about 370,000 wet 4 m cells), and on Valheim's own world for the same seed, with the same scripts on the same night (Valheim's own world in brackets):
- springs (water pouring out of one point with nothing feeding it): 182 to 71 (121 to 59), none of them at lakes any more
- spots where the current meets itself head-on: 187 to 116 (573 to 353)
- spots where the current turns right round from one 4 m cell to the next: 1229 to 887 (2784 to 2054)
- drains (water running in from every side with nowhere to go): 19 to 18 (17 to 6)
- bank water with the current running mostly into the land: 1.1% to 0.9%
- water faster than 2.2 m/s: 0.8% to 0.6%; river speeds by size unchanged (creeks about 1.0 m/s, great rivers about 1.3)
- more water now lies still where nothing feeds it: river water under 0.05 m/s 11% to 15% (36% to 39%), mostly broad water off to the side of the rivers and the slack where water parts, which only moved before because of a flaw in the old calculation

## Problems or ideas?

Message me on [Discord](https://discord.com/users/220007226121781248). `straumr report` writes everything I need to reproduce what you saw; please send that file and say which other mods you run.

## Thanks

Inspired by the river worlds of Riverheim and the streams and waterfalls of Rivers and Lakes. Go give them a look:

- **Riverheim:** [Thunderstore](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/) · [Discord](https://discord.gg/ES3J7mu6kW)
- **Rivers and Lakes:** [Thunderstore](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/) · [Discord](https://discord.gg/vuHnDRqfjn)

Built with help from Claude, Anthropic's AI.
