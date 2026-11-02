# Straumr

**Living water for Valheim.** Rivers flow downstream, rapids run white, the sea has currents and tides, and whirlpools turn off the ends of tidal straits. Swimmers, boats and floating things all ride it.

*Straumr* is Old Norse for "current". This is a **beta** (0.1): it works, it's been played on, and it's still being tuned. Feedback welcome.

## What it does

**Rivers**
- Every river flows from source to mouth, faster where it narrows, slow where it widens into a lake, and slower near the banks than mid-stream. Creeks run about 1 m/s and great rivers about 1.3 m/s (you swim about 2).
- Water slides along the banks and round islands instead of running into them, and carries on out past a river's mouth before it fades.
- Where several rivers share the same water (Riverheim often runs a few through one channel, sometimes in opposite directions), that water gets one way down: its shortest way to the sea. Rivers keep the direction Riverheim drew unless the other way is less than half as long, so they rarely run back toward the hills.
- A river that ends in a lagoon or bay runs out through it to the sea.
- **White water** where fast water runs shallow, worked out from the Froude number, the way real rapids form. White water shoves swimmers about, now and then pulls them under for a moment, and wears them out.
- Rocks, logs, posts and walls that break the surface bend the current, with slack water and a small back-eddy behind them; the water stalls against their upstream face. A stone just under the surface doesn't stop it. A wall or dam across a channel stops the flow, and the water above it lies still.
- Bends push surface water to the outside bank, shallow rivers alternate pools and riffles, and side rivers meet in a swirling seam.
- **Weather and seasons:** rain swells the creeks first and the big rivers later; a storm swells them more. With Seasonality, spring snowmelt runs high and winter runs low (or freezes, if Seasonality freezes the water).
- Dig a channel below the water line between two waters and the current runs through it.

**The sea**
- Wind-driven currents run along the coasts, through the sounds and between islands; shallow swamps and marshes stay mostly calm. Wind drifts the surface, rips run out from open beaches, and longshore drift runs along them.
- **Tides:** the sea rises and falls up to 0.6 m either side of its usual level twice a game day (0.8 m at spring tides). Bays behind narrow mouths rise less and later, and their mouths race in and out (about 1.5 m/s, 2 m/s at spring tides). Up river mouths the flood pushes against the river and the ebb speeds it out. Slack water comes at high and low tide, a little later each day, and the moon sets the size: spring tides at full and new moon, gentle neaps between. With Seasonality, each season is one lunar month.
- What Valheim checks against the water follows the tide: swimming, boats, floating things, building damage under water, placing water pieces, fires going out, and creatures deciding where the water is.
- **Whirlpools** spin up off the ends of tidal straits near the peak of each tide. Mind the eye.
- A line under the minimap tells you the tide near any coast: *"Flood tide · full · high water in 6 min · +0.1 m"*.

**Life on the water**
- **Foam** shows which way and how fast the water goes: a few scraps on all water (still water too), much more where it runs, shears and meets, in rafts, lace and strands drawn out by the current, and white water with streaks and spray. Rivers murmur, rapids roar (sounds made from the physics of bubbles, with no recordings), and mist lies on the water at dawn.
- Driftwood washes up in bends, and fish hold in the pools.
- Boats feel the current while someone's aboard but always keep some headway, and a boat stopped against a bank or dock stays moored.
- Ruins and other stonework the world placed don't crumble where a steep world leaves their edge hanging (`[World] StandingRuins`).

## Works with

Straumr runs on its own, on Valheim's own worlds, and detects these automatically when they're installed. None is required.

- **[Riverheim](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/):** worlds built around big river networks, which Straumr sets flowing.
- **[Rivers and Lakes](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/):** streams, lakes and waterfalls above sea level. Works side by side with Straumr today: Straumr leaves its water to it. With its next update, its streams carry on as jets into still water and its bigger lakes drift with the wind.
- **Seasonality:** seasons, snowmelt and freezing.
- **ValheimRAFT:** its vehicles ride the current too.

## For mod makers

Straumr has a public API for other mods: water wheels, mills, fishing, sailing AI, current charts. It reports physical properties (velocity, depth, tide, power density) and leaves the gameplay to you. Everything is in `Straumr.Api` inside `Straumr.dll`, with no reflection needed. The full guide, with examples (a water wheel, fishing spots, a boat following a route, a current chart), is in **`API.md`**, which ships with the mod.

- **Setting up:** reference `Straumr.dll`, add `[BepInDependency("wackah.straumr", BepInDependency.DependencyFlags.SoftDependency)]`, and only touch the API when Straumr is installed (`Chainloader.PluginInfos.ContainsKey("wackah.straumr")`). Your mod then works with or without it.
- **The one call:** `StraumrApi.Sample(position)` gives the current, depth, Froude number, white water, tide height and rate, the water's actual surface height, how open the water is, what kind of water it is, where its current comes from, and how much to trust its direction (`Confidence`).
- **Helpers:** `Velocity`, `KineticPowerDensity` (W/m², for wheels), `WaterLevel` (the water's real surface, tide in: use it instead of Valheim's fixed sea level when placing things at the water's edge), `TidePhase`, `MinutesToTideTurn`, `SeaCurrent`, `FloodStream`, `DownstreamDirection`, `DownstreamPath`, `RiverMouth`, `SampleAlong`.
- **Events:** `TideTurned`, `SpringNeapChanged`, `FloodChanged`, `FreezeChanged`.
- **Versioning:** `StraumrApi.ApiVersion` is 1. It only goes up for a change that would break a mod built against it; new members don't raise it.
- **Limits:** the water is worked out on each player's game around them (within about 160 m), not on a dedicated server, and Rivers and Lakes' streams aren't sampled. `API.md` lists these honestly.

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
| `straumr here` | the water at your feet, each part of the current (river, tide, sea, rocks...), and its way to the sea |
| `straumr report` | writes a file describing the water where you stand, including what's standing in it, for bug reports |
| `straumr selftest` | checks Straumr is working right where you stand (PASS/FAIL per check) |
| `straumr waterlevel` | the sea level here: the usual level, the tide, and the water's actual surface |
| `straumr perf` | how hard Straumr is working (turn on `[Diagnostics] PerfCounters`) |
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
| `SpringNeap` | 0.35 | How much bigger spring tides are, and neaps smaller (0 = every day alike). |
| `TidalMonth` | 16 | Without Seasonality: game days from one full moon to the next. |
| `Whirlpools` | 1 | How hard whirlpools spin (0 = none). |
| `Rapids` | 1 | How hard white water handles swimmers (0 = not at all, up to 2). |
| `Rips`, `Longshore` | 0.8, 0.5 | Rip currents off open beaches, and drift along them. |
| `WindDrift` | 0.5 | Surface drift downwind at full wind on open water (m/s). |
| `Players`, `Mobs`, `Items`, `Boats` | 1, 1, 1, 0.6 | How strongly the current carries each (0 = not at all). |
| `BoatMinSpeed` | 3 | A boat under sail or oar always keeps this much headway (m/s). |
| `BoatTurn` | 0.35 | How much differing currents turn a boat (0 = never). |
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
| `CalmSpeed` | 0.3 | Foam thickens from this speed up (m/s). |
| `Opacity`, `Brightness` | 0.6, 2 | How solid and how bright the foam looks. |
| `Rafts` | 1.5 | How many flecks gather in each raft (0 = single flecks). |
| `WhiteWater` | true | Fast water looks white (only the look). |
| `Spray` | 0.3 | How often spray kicks up off the fastest white water. |
| `WhiteFroude`, `WhiteFroudeFull`, `WhiteMinSpeed` | 0.45, 0.9, 0.8 | When water turns white: from this Froude number, fully white at this one, never below this speed (m/s). |

**Your own, the rest of the look:** `[Sound]` `Enabled`, `Volume`; `[Mist]` `Enabled`, `Amount`; `[Hud]` `TideClock` (the tide line under the minimap).

**What the world gets** (*server*): `[Driftwood]` `Enabled`, `EveryMinutes`, `Prefab`; `[Fish]` `Enabled`, `EveryMinutes`, `PerSpot`, `Kinds`; `[Rocks]` the rocks set at river sinks and springs.

**Admin and tools:** `[World]` `StandingRuins` (shared) and `NewWorldSeed` (*server*: the seed for new worlds, since the dedicated server can't be given one); `[Atlas]` `Port`, `HiddenSeeds`; `[Diagnostics]` `CurrentProbe`, `PerfCounters`, `LocationCheck`, `WorldGenCommand` (testing tools, off by default).

Upgrading from 0.1.0: if your foam settings are still 0.1.0's untouched defaults, they move to the new ones by themselves; anything you changed stays as you set it.

## Known limits (beta)

- Valheim's own water surface doesn't move with the current: the foam and the things floating on it show the flow.
- In a wide bay or sea arm with several river lines far apart, the current can swirl oddly between them.
- Where two ways to the sea through the same water are nearly equal, the water splits between them, with a faint spring where they part.
- A few things Valheim keeps at its fixed sea level don't move with the tide: mist, embers, the beach's shading, and where the world placed things.
- The sea is one flat sheet per 64 m zone, raised as a whole, so neighbouring zones can differ by a few centimetres.
- Rivers and Lakes' own streams and lakes move as that mod draws them; Straumr only adds the jets and wind drift described above.
- Worlds much bigger than Valheim's standard size aren't supported yet.

## Roadmap

Plans, not promises; the next steps depend on what players find.

- **Next (0.1.2):** rivers you can see flowing (the water's own surface moving downstream, with the game's flowing-water look, the foam becoming the accent); wind drift on lakes; storms at sea that push harder, and waves that carry things toward the shore; rivers that carry the water their land drains, so side rivers add up and a river that splits shares its water by what each branch can carry.
- **Later:** swimming harder against the current, fast water pushing waders about, boats swinging into the current, logs riding a river down, stones just under the surface showing a riffle, rivers running higher after storms and in spring, storm surge at sea, fish that hold where the flow suits them.
- **0.2:** water that remembers what happened: lakes filling and draining, a log jam backing a river up and then bursting.
- **Not planned:** new models or textures (water wheels, mills, nets). Those belong to other mods, built on the API.

## How it was checked

0.1.1 was measured over a whole Riverheim world (214 areas of 256 m along its rivers, lakes and junctions), against 0.1.0:
- bank water with the current running mostly into the land: 6.2% to 1.1%
- spots where the current meets itself head-on: 1456 to 201
- spots where the current turns right round from one cell to the next: 5323 to 1245
- drains (water running in from every side with nowhere to go): 170 to 27
- water faster than 2.2 m/s: 6.1% to 0.8%
- creeks (rivers under 10 m wide) 2.0 to 1.0 m/s, great rivers (over 80 m wide) 0.9 to 1.3 m/s: creeks used to outrun great rivers

## Problems or ideas?

Message me on [Discord](https://discord.com/users/220007226121781248). `straumr report` writes everything I need to reproduce what you saw; please send that file and say which other mods you run.

## Thanks

Inspired by the river worlds of Riverheim and the streams and waterfalls of Rivers and Lakes. Go give them a look:

- **Riverheim:** [Thunderstore](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/) · [Discord](https://discord.gg/ES3J7mu6kW)
- **Rivers and Lakes:** [Thunderstore](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/) · [Discord](https://discord.gg/vuHnDRqfjn)

Built with help from Claude, Anthropic's AI.
