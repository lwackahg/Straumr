# Straumr guide

The reference for Straumr. The front page is the [README](https://github.com/lwackahg/Straumr/blob/v0.1.5/README.md). The API for mod makers is [API.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/API.md). Questions and bug reports: the [Straumr Discord](https://discord.gg/bU9AD8hb8M).

Contents:
- [Features](#features)
- [Works with](#works-with), [Multiplayer](#multiplayer), [The seed atlas](#the-seed-atlas)
- [Limits](#limits)
- [Settings](#settings)
- [Commands](#commands)
- [Known issues](#known-issues)
- [Roadmap](#roadmap)

## Features

### What it does

**Rivers**
- Every river flows from source to mouth, faster where it narrows, slow where it widens into a lake, and slower near the banks than mid-stream. Creeks run about 1 m/s and great rivers about 1.3 m/s (you swim about 2).
- Water slides along the banks and round islands instead of running into them, and carries on out past a river's mouth before it fades.
- Where several rivers share the same water (Riverheim often runs a few through one channel, sometimes in opposite directions), that water gets one way down: its shortest way to the sea. Rivers keep the direction Riverheim drew unless the other way is less than half as long, so they rarely run back toward the hills.
- A river that ends in a lagoon or bay runs out through it to the sea.
- **Rivers carry their water**. Straumr estimates how much water each river carries: from the land draining into it on Riverheim worlds, from the length of channel above it on Valheim's own. A river sounds as full as the water it carries. Where the water has two ways down (a loop, or a lake with a second way out), the second way carries its share, worked out from how easily each runs (`[Currents] Splits`).
- **Chutes**. Where a river runs down into a stretch its banks hold lower, the slope (up to 3 cm a metre) shows as white water, in the foam and the sound, and the river runs faster along it, up to the typical river's speed (`[Currents] Chutes`; 0 = only the look and the sound).
- **White water** where fast water runs shallow  (worked out from the Froude number, the way real rapids form). White water shoves swimmers about, now and then pulls them under for a moment, and wears them out.
- **Strid** (called Undertow in 0.1.2): where a river runs fast through a slot, a channel deep for its width, its surface stays smooth while the water below now and then pulls a swimmer under for half a second or so, with at least 15 s between pulls in the worst slot (40 s in a mild one). It fades out as the water turns white, which handles swimmers its own way. It costs no stamina and, at the default setting, never takes anyone more than 1.5 m below where they swim. The Strid is the River Wharfe's slot, which it is modelled on.
- **Wading:** walking in a current over your ankles, the water pushes you downstream, more the deeper it is and with the speed squared. Walking upstream is slower going. It never takes more than half your slowest walking pace, so you can always walk out.
- Swimming costs the usual stamina whichever way you swim, as in real water: the current changes where a stroke takes you, not what it costs. `SwimWithCurrent` and `SwimAgainstCurrent` can change that (0.85 makes going with a river a little easier, 1.25 fighting it a little harder).
- Rocks, logs, posts and walls that break the surface bend the current, with slack water and a small back-eddy behind them; the water stalls against their upstream face. A rock counts only as big as the part of it the water meets at the surface. A stone just under the surface doesn't stop it. A wall across a channel stops the flow, and the water above it lies still.
- **Eddies behind the ground**: where a sand bar, an island or a point sticks out into a river, the water past its end curls back into its lee, and over a bar that is out of the water or barely covered the river splits round it instead of running over. It goes by how much water covers the ground right then (the tide, the river's level, the waves), so it comes and goes with them (`[Currents] LeeEddies`).
- Bends push surface water to the outside bank, shallow rivers alternate pools and riffles, and side rivers meet in a swirling seam.
- **Weather and seasons:** rain swells the creeks first and the big rivers later; a thunderstorm swells them more. The rain is the lowlands' (Meadows, Black Forest, Plains, Mistlands) as Valheim draws it for each weather spell, worked out again from the server's clock, so it is the same for every player wherever they stand. With Seasonality, spring snowmelt runs high and winter runs low (or freezes, if Seasonality freezes the water). The rivers' levels follow all of it.
- Dig a channel below the water line between two waters and the current runs through it, in your game.

**River levels**
- Rivers stand above the sea. Running water needs a slope to push it, so each river's surface climbs gently from the sea toward its source, worked out from the speed the river shows: steeper where it runs fast and narrow. On the Riverheim world it was checked on, the banks let rivers stand up to 1.34 m above the sea in an ordinary season (`[Currents] RiverLevels`; `RiverLevelScale` makes them higher or lower).
- The banks decide how high. Where a 64 m square's water can't rise to the river's full height without spilling over land away from the river or leaving ponds in hollows, it stands lower there, and the river runs down into it at up to 3 cm a metre (see *Chutes*).
- The water slopes smoothly from one square to the next, with no steps at their edges. Swimming, boats and floating things meet the same surface you see.
- With Seasonality the rivers stand higher in spring and lower in winter: how high goes with how fast they run, squared, so mid-spring's 1.4 times the speed stands about twice as high where the banks allow. In spring they may also spread over more of the low ground beside them, the flood plains (`[Currents] SpringFlooding`; `RiverFlooding` says how far they may spread at all), which dry out again as the season turns; rivers fed by mountain snow rise in the afternoons. In Seasonality's freeze they behave differently (see Known issues).
- Rain raises them a little, following the great rivers, which swell later and less than the creeks. With `[Floods]` on it raises them by each river's own size's rain (see *Floods*).
- The levels change at most once a weather spell (five minutes into it), and a new level fades in over half a minute. Every player's game works the same surface out from the world, the shared settings and the server's clock.
- Driftwood washes up just above each river's spring high water: a strandline that shows how high the water can come.
- Valheim's own worlds barely show it: their banks are low and their rivers slow.

**Dams and ponds** (experimental, on by default: `[Dams] Enabled`; the server's setting is everyone's)
- Pieces you build (with any ground you raise beside them) that stand across a river from bank to bank, 10 cm or more over its ordinary level, dam it. The water backs up behind the wall as a level pool over everything joined to it below that level, at the river's own flow, until it spills over the lowest part of the wall or finds a way round.
- The pool fills on the in-game clock: an hour's filling takes an in-game hour (75 real seconds with Valheim's 30-minute day). Below the dam the river carries only what comes over or through the wall, and runs lower while the pool fills.
- **Breaches and surges.** Break a wall and the pool falls through the gap as over a weir, fast at first and slowing as the head falls. The released water runs down the river as a front at real speed (the river's own current plus a long wave's speed in its depth: about 6.6 m/s on a river 3 m deep running 1.2 m/s), so you can watch it come. The river stands higher behind it. A pool downstream takes the surge into its own pool. Overtopping never breaks a dam.
- **How wide and how fast.** A breach, a hole or a gap is measured along the wall, so a wall built at an angle counts its true length, and stone taken down beside a break widens it. The crest is measured the same way, so a diagonal dam doesn't spill too easily. A pour never runs faster than water can at its depth, and a pond that spills round a wall's end over a narrow col stands the weir's head over the col's real width.
- **Gates.** A door standing in a dam's wall is a sluice gate (up to 8 a dam). Opened, it passes water over its sill, and under its lintel through an opening, both ways. A gate opens or shuts over 5 in-game minutes; the door itself swings at once. A bottom door is a low outlet, a top door a weir.
- **Holes.** A hole broken low in the wall with pieces still standing over it, or an open gate under its lintel, draws swimmers within one and a half times its height toward it, harder the more water it passes, and takes them through if it is a metre or more each way (out below the dam, on its jet); a smaller one holds them against it (`[Dams] DragUnder`).
- **Two dams in a row.** The water between them is a cofferdam while nothing flows into it. Once the upper dam's pond pours into it, it fills as the lower dam's pond. If the upper dam breaks into a dry ring, the ring fills over the timing curve (see Cofferdams) and then the two come level. Where the dams are so close that the upper breach opens straight into the lower pond's water, the two become one water held by the lower wall. Water is neither made nor lost at the hand-over. Where the upper dam's water goes round it and comes back into the river below the lower dam, the lower pond doesn't get the whole river. Break the upper wall and the lower pond falls to the open water above it.
- **Water it's open to.** A pond open to other water (the river above it round the wall's end, another channel of a braided river, or the river above through a broken upper wall) can't stand above that water by more than its flow needs to get through the opening. Where the water can go round, the dam lets it go round at the river's own level.
- **The look.** A dam's pond is drawn as a body of water of its own with the game's water look, its edge on the ground's own shoreline and the wall (`[Pools] OwnSurface`, each player's own). Where it pours over the wall or through a breach, the water hangs off the pond's edge down the wall (`[Dams] SpillSheets`). `[Dams] SpillRamp` draws the pool's level over the crest and down a 2-4 m ramp below it instead of a step in the wall (off by default).
- **Sleeping and time skips move the dams' clock too.** After a night's sleep, a filling or draining pond has moved on by the night's time, as if you had watched. This is correct, not a bug. A console time skip does the same.
- Dams are written to `BepInEx/config/Straumr-water/<world>.dams.txt` by the server with the world's save, and only then. A restart resumes a filling pool, a spill or a breach where the world was saved: after a clean stop that is where it was, and after a crash or a kill it is the last world save. A dam standing with no record there (a world from before, or `[Dams]` just turned on) is taken as long closed, its pool full.

**Cofferdams** (experimental, on by default: `[Cofferdams] Enabled`)
- A patch of river, lake or sea walled off on every side by pieces standing at least 10 cm above the highest water outside (the river's level, plus the highest spring tide and a full storm's surge where the tide reaches), with no gap, is a cofferdam. Ground or a bank may close the ring too.
- Its water falls at `[Cofferdams] DrainRate` (0.5 m a real minute) until the bed lies bare. Break the ring and the water fills back in through the gap as over a weir. An enclosure built before the setting is turned on drains too.
- `[Cofferdams] WaterMask` (each player's own) hides the game's water over a drained cofferdam, so from past 128 m you see the bed inside the walls instead of flat water.
- **Filling from a dam above: the timing curve.** A dry cofferdam below a dam that breaks fills from that dam's pond through the opening, by the head across it. That can take seconds or hours in the water's own time, and the dams' clock runs 48 times faster than the water's own time (with Valheim's 30-minute day), so a short fill would be over before you saw it. A fill plays at its own pace when it is short and is sped up only as far as it needs to be when it is long, never past the clock's own 48 times. A fill that would take 5 s of the water's own time plays in about 5 s; 1 minute in about 36 s; 80 minutes in about 2 minutes. A ring refilled by its own river, with no dam above pouring into it, is paced the same way. Only these fills are paced this way: ponds, spills and surges keep the clock's pace and the surge's own speed.
- Saved with the world, like dams.

**Kept pools** (experimental, on by default: `[Pools] Enabled`)
- Water you cut off from the river with ground alone keeps its own level instead of following the river: dig a basin, let the water in through a channel (from a dam's pool too), fill the channel in again, and the basin keeps the water it had, through floods and after the dam is gone. It keeps its water, not its level: dig it deeper or wider and the same water stands lower, throw earth in and it rises and spills over its rim.
- Only water the server saw joined to the water outside and then cut off is kept: a pond it never saw joined stays as it is. A gap dug in the rim lets it fill or drain to the water outside; what it lets into a river runs down that river as a dam's release does; a door standing in its rim is a gate. Two kept pools joined by digging come level.
- The server keeps its pools in `BepInEx/config/Straumr-water/<world>.txt`, written when the world saves.

**Floods** (experimental, off by default: `[Floods] Enabled`)
- Rain raises each river's level by its own size's rain, as it already speeds its current. Creeks rise within an in-game hour of a storm's start; great rivers peak about a day after the rain began and fall back over days. A high river holds the rivers running into it up at their mouths. A stretch whose low banks hold the river down in ordinary weather spills over them in a flood (`[Floods] FloodRoom`, `FloodPonds`). With floods off, the levels follow only the great rivers' rain.
- The levels step every in-game hour (75 real seconds), each step fading in over the next.

**The sea**
- Wind-driven currents run along the coasts, through the sounds and between islands; shallow swamps and marshes stay mostly calm, apart from the wind's drift on their broader pools. Rips run out from open beaches, and longshore drift runs along them.
- **Wind** drifts the surface of the sea, and of lakes and broad slow reaches too: from about 8 m off the shore, fully from about 60 m out. Running channels don't take it. A storm pushes harder than a breeze.
- **Waves** carry floating things slowly the way they run: downwind on open water, turning in toward the beach over the shallows, so with the wind onshore driftwood, wreckage and felled logs wash up. In a full gale the drift is about 0.05 m/s far out and up to 0.3 m/s over about 10 m of water. Small boats with someone aboard take a share of it, a longship none. In a strong wind the shallows turn to surf, which pushes swimmers the way the waves run. None in a river's channel.
- **Breakwaters**: walls and other built pieces standing in the water, and ground raised above it, keep out the waves coming from upwind. The calm is most just behind them, less further back, none past 128 m. Valheim shapes its waves only at the corners of each 64 m zone, so the calm shows across a harbour-sized basin, not behind one short wall. Boats ride the same calmer waves that are drawn (`[Currents] Breakwaters`; `NaturalShelter` lets the land as the world made it calm the waves in its lee too, off by default).
- **Tides:** the sea rises and falls up to 0.6 m either side of its usual level twice a game day (0.8 m at spring tides). Bays behind narrow mouths rise less and later, and their mouths race in and out (about 1.5 m/s, 2 m/s at spring tides). Up river mouths the flood pushes against the river and the ebb speeds it out. Slack water comes at high and low tide, a little later each day, and the moon sets the size: spring tides at full and new moon, gentle neaps between. A storm raises the sea a little more on top (up to 0.3 m in a full storm), as far up rivers as the tide reaches.
- **The Deep North's ice**: the game lays its sea ice at a fixed height, so the tide had put water on top of it. Over the ice the tide and the storm surge now fade to nothing, back to the full tide over about a kilometre, and tidal streams and whirlpools fade there the same way.
- What Valheim checks against the water follows the tide: swimming, boats, floating things, building damage under water, placing water pieces, fires going out, and creatures deciding where the water is.
- **Whirlpools** spin up off the ends of tidal straits near the peak of each tide. At the rim of a turning whirlpool's core the pull draws you in gently (about 1.2 m/s, slower than you swim, so you can swim away) and harder toward the eye; within 0.4 of the core (4 to 12 m from the middle) it is faster than you can swim, so once that close you're carried into the eye. There it pulls you under for a second every 4 s and costs 3 stamina a second, until after 6 s it lets go and you can swim clear. In full while it turns at 1 m/s or more, near the peak of the tide. Boats and floating things aren't gripped (`[Currents] WhirlpoolGrip`). The sea also dips into a funnel at the eye, which you swim, float and sail in (`[Currents] WhirlpoolFunnel`).
- A line under the minimap tells you the tide near any coast: *"Flood tide · full · high water in 6 min · +0.1 m"*.

**The water's surface** (still being tuned)
- Ripples and sun glints on the water move with the current. Straumr lays a thin see-through sheet on the water round you, with its own shader. It rises and falls with the game's waves and the tide, and the current carries its ripples along. It only adds glints and the sky's light on the ripples. Still water doesn't show it, running water (from 0.5 m/s) shows it in full, and it fades out by about 60 m from you.
- The shader is built into `Straumr.dll` for each platform: Windows (which also covers Proton), Linux and Mac.
- It switches itself off where it can't run (no shader for your platform, or a graphics card that can't run it), when NorgerWater is installed, while Seasonality has the water frozen, on Rivers and Lakes' water, and toward the Deep North, where the game calms its big waves.
- `straumr overlay` in the console turns it off and on in your game, to compare, and says whether it's on, and why if it couldn't start at all. Its settings are under `[Surface]`.

**Life on the water**
- **Foam** shows which way and how fast the water goes: a few scraps on all water (still water too), much more where it runs, shears and meets, in rafts, lace and strands drawn out by the current, and white water with streaks and spray. It fades where it meets a bank or a rock instead of ending on a hard line (`[Flecks] SoftEdges`), and `[Flecks] Lift` sets how high it floats. Rivers murmur, fuller the more water they carry, and rapids roar (sounds made from the physics of bubbles, with no recordings), and mist lies on the water at dawn.
- Driftwood washes up in bends and just above the spring high water, and fish hold in the pools.
- **Felled logs float**, about two thirds under, and drift with the current and the waves: rolled into a river, a log is carried off downstream, pivoting round an end that grounds and lining up with the stream.
- **Log drives** (lab-tested, not yet played). A wall or dam across a river catches what floats down against it. Still water backed up against it drifts toward it (at 5 cm/s at least) so things reach it instead of stopping short (`[LogDrive] DamApproach`). Pieces with gaps at the waterline (a lattice, a table's legs, an open frame) pass water and catch what is thicker than their gaps (`[LogDrive] Screens`). On the server, logs and floating items in a river with nobody near are carried on down it until a barrier catches them or the sea, a lake or another mod's water takes them; a player coming near takes them over where they are (`[LogDrive] ServerDrive`, `Items`). A thing caught within 40 m upstream of a barrier counts as inside a base, so Valheim doesn't remove a dropped item after an hour (`CaughtIsBase`).
- Boats feel the current while someone's aboard but always keep some headway, and a boat stopped against a bank or dock stays moored.
- A boat left to drift in a river (someone aboard, no one at the helm, sail furled and oars shipped, not moored) slowly swings round to lie along the current, bow or stern downstream, whichever is nearer: across a 1 m/s current it comes round in about half a minute to a minute, quicker in faster water. It fades out on open water.
- A boat left empty (not moored) rides as if at anchor: it stops where it was left, swings a couple of metres either way, and in a river turns to lie along the current; board it and it's free (`[Currents] AnchorEmptyBoats`).
- A line under the minimap while you're aboard a boat gives its speed through the water and over the ground, and what the current does along its heading: *"Boat 2.4 m/s through water"*, *"3.1 m/s over ground"*, *"current +0.7 (helping), running NE"*. The current there is the share of it that carries boats (`[Currents] Boats`). Valheim's own boats only, so far.
- Ruins and other stonework the world placed don't crumble where a steep world leaves their edge hanging (`[World] StandingRuins`).

### Works with

Straumr runs on its own, on Valheim's own worlds, and detects these automatically when they're installed. None is required.

- **[Riverheim](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/):** worlds built around big river networks, which Straumr sets flowing. **1.2.x and 1.3.x both work.** Riverheim 1.3 is a [Worldhalla](https://thunderstore.io/c/valheim/p/Gurebu/Worldhalla/) plugin and needs it; Straumr finds Riverheim's rivers through Worldhalla's public API by reflection, so Straumr itself still needs only BepInEx.If Riverheim's insides change again, Straumr says so once in its log and carries on with Valheim's own rivers rather than failing. Two warnings, both from how Riverheim and Worldhalla work, not from Straumr:
  - **Old 1.2 worlds convert one way.** A world made by Riverheim 1.2 loads in 1.3.x with the same terrain (checked: the generation code gives identical heights, biomes and rivers for 10 seed and preset pairs), but the first save writes Worldhalla's format, which Riverheim 1.2 can't read. A player's game asks first; a dedicated server just does it. Back up the `.fwl` and `.db` files before you move a world. Riverheim's own notes cover this too.
  - **Worlds made with Riverheim_Config's settings need Riverheim_Config 3.1.1 or later** on Riverheim 1.3.x, or their terrain changes.
- **[Rivers and Lakes](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/):** streams, lakes and waterfalls above sea level. Works side by side with Straumr: Straumr leaves its water to it. Its streams carry on as jets into still water, and its bigger lakes drift with the wind, once Rivers and Lakes shares its water with other mods. Where one of its streams comes down a fall, the jet starts at the foot of the fall, and the plunge drives it out faster (up to half again for a fall of 10 m or more). Streams that pour into a raised river meet it at the river's level. Where a stream runs into Straumr's water at the sea's level, Straumr's water is the water there.
- **[Seasonality](https://thunderstore.io/c/valheim/p/RustyMods/Seasonality/):** seasons, snowmelt and freezing. With it the rivers run faster and stand higher in spring, with flood plains, and run slower and stand lower in winter.
- **[ValheimRAFT](https://thunderstore.io/c/valheim/p/zolantris/ValheimRAFT/):** its vehicles ride the current too.
- **Expand World Size:** worlds of any size. Straumr reads its live world radius, the edge's width and the stretch by name when it's installed, never requiring it, and solves the sea's currents out to the world's own edge. Checked with Expand World Size 1.45 on a lab server on worlds with a 21 km radius (22 km edge), a 7 km radius (7.5 km edge) and the default; the sea solve takes about 10.7 to 13.6 s on a 22 km edge, in the background at load. Riverheim's map stays the size Riverheim made it, so a bigger radius adds sea, not land. "Stretch world" works too, and the atlas no longer undoes it in the world you play; the limits are under [Known issues](#known-issues).

**NorgerWater:** when it's installed, Straumr's moving surface switches itself off and leaves the water's surface to it, the rivers stay at the sea's level (it draws the sea at one level everywhere), and dams, kept pools, own-surface ponds and breakwaters switch off. Straumr hasn't been tested with it in game yet.

### Similar mods

Running Straumr with **[Undertow](https://thunderstore.io/c/valheim/p/RavenIronStudios/Undertow/)** (Raven Iron) is not recommended. Both mods give the sea currents and push boats, swimmers and floating things along them, so at sea you would be carried twice, by two sets of currents that don't know about each other.

Running Straumr with **[Fires Valheim Galaxies](https://thunderstore.io/c/valheim/p/incompletionists/FiresValheimGalaxies/)** (incompletionists) is not recommended. Both mods raise and lower the sea with a tide, run the rivers' mouths and push boats and swimmers along currents, so the tide would be moved by both at once and anything afloat carried twice.

### For mod makers

Straumr has a public API for other mods (water wheels, mills, fishing, sailing AI, current charts): it reports the water's physical properties and leaves the gameplay to you. `StraumrApi.Sample(position)` gives the current, depth, tide and more in one call; also a river's rise above the sea, how full it runs, how much water it carries and whether the ground is flood plain or under flood water. `StraumrApi.Outlook(position)` gives a river's next four hours, `StraumrApi.LevelsChanged` says when the river surface round the player changes, and dams, cofferdams, gates, water power (`Work`, `BestDrop`, `HydraulicPower`) and drift paths have members of their own. Soft-depend on `wackah.straumr` and your mod works with or without Straumr installed. `ApiVersion` stays 1. The full guide, with examples and its limits, is [API.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/API.md).

### Multiplayer

**Tested on a dedicated server with lab clients, with the join, config-sync and restart tests; not yet tested with a group of players.** Nobody has yet had two or more real players in the same pond, joined mid-fill or watched the same surge from two places.

Install Straumr on the server and on every player's game. The server's physics settings are sent to everyone who joins, and players get their own back when they leave: all of `[Currents]`, `[Stage]`, `[Floods]`, `[Dams]` (except `SpillSheets`), `[Pools]` (except `OwnSurface`), `[Cofferdams]` (except `WaterMask`) and `[LogDrive]`, the white-water thresholds (`WhiteFroude`, `WhiteFroudeFull`, `WhiteMinSpeed`) and `[World] StandingRuins`. How the water looks and sounds stays each player's own choice. A shared setting the server doesn't send is held at its default while you're on that server.

Tides run off the server's clock, so everyone has the same tide. The rivers' levels are worked out the same way on every player's game, from the world, the shared settings and the server's clock (its season and its weather), so everyone sees the same water. Rain moves the levels only while a player's game draws the same weather as the server's: `straumr weather` says whether it does. Where it doesn't (a weather mod or setting that differs), rain still moves that player's currents, but not the water's height.

Dams, cofferdams and kept pools are decided by the server: it finds them, and sends each game a record with the settings; each game works the water out from that record and the server's clock, so a boat, a swimmer and a dropped item sit at the same water in every game. A game that joins late, or comes back after being far away, gets the same water. The server's own water stays at the sea's level, as with the tide: only players' games raise the rivers. On a dedicated server, dams and the server's log drive need `[Currents] RiverLevels` on.

Players who join with an older Straumr (0.1.2), or none, are told on screen which Straumr the server runs and that their water will differ, and a player whose Straumr is another version than the server's is told both versions. 0.1.5 needs no new setting and the world's files keep their format. A player whose server has no Straumr, or an older one, is told too. An unknown or damaged part of the settings package costs only itself: the rest is read.

Mixed versions: neither side is refused, and each is told the versions differ. A live 0.1.4 game joining a 0.1.5 server was not tried.

### The seed atlas

- `straumr atlas` in the console (F5) opens a page from your own game at `http://localhost:8765`.
  - Type any seed: your game builds that world in memory (nothing is saved) and shows its rivers, sea currents, tides and whirlpools.
  - It starts whole-world, and fills in real detail as you zoom. Press **D** for a debug view of why the water does what it does at any spot, and **T** for the tide.
  - Layers (all in the panel): rivers, sea currents, tidal races, whirlpools, white water, **River levels** (off by default: where raised rivers stand over land, and the spring flood plain), **Fords** (off: where a river can be waded across), **Water power** (off: the strongest water per area, for water wheels) and **Built water** (dams and their pools, kept pools, cofferdams and the river below each dam).
  - **Drift** (key **R**): click the water and it draws the path something floating takes from there over 5, 15 or 60 minutes.
  - It shows the whole map, so don't type a seed you want to keep a surprise (or list it in `[Atlas] HiddenSeeds`).
  - On an Expand World Size world the page reads the world's edge and draws the stretched rivers; a seed you type is built with the played world's size (see [Known issues](#known-issues)).
  - Only that page can read it (a key it gets from the game), and it closes a few seconds after you close the page.
- `straumr map` writes the world you're in to a file the same page can open.
- On a dedicated server the atlas is started from the config file by its admin, and its `@here` seed is the server's own world.

## Limits

The builder's hard limits and the rules behind them. Each number is from the code.

### Building
- **Dams:** the server holds 16. At most 8 of a dam's doors are taken as gates (lowest sills first); a door past the eighth counts as wall. An open door that is not in a dam's own wall (a side channel, a mill race, a way round) also counts as wall to the dam's pool. Leave a gap, not an open door, where the water should go round.
- **A dam's wall** is built pieces across the river from bank to bank, 10 cm or more over its ordinary level. Raised ground beside them can complete it, but ground you raise with the hoe alone is not found as a dam. Walls and stones that do not dam a river bend the water like rocks and leave calm water behind them.
- **Not dams:** a wall in a river's tidal reach that the highest tide and a storm's surge can top, and a dam the water goes round at the river's own level. Overtopping never breaks a dam. A wall has to be broken.
- **Reach:** a pond is judged up to about 2 km from its dam. Ponds longer than about 500 m were checked only in a closed-form test, not in the lab or in play.
- **Ceiling:** a pond or kept pool may stand up to 60 m over the sea by default (`[Dams] Ceiling`, up to 100 m). Valheim's own water ends 10 m over the sea. Above that, Straumr's own water carries it.
- **Cofferdams:** the server holds 64. One can be at most 128 m across. Its walls must stand at least 10 cm above the highest water outside, with no gap.
- **Kept pools:** the server holds 64. A kept pool is only made from a cut the server saw.

### Rivers and levels
- River levels change at most once a weather spell (five minutes into it), and a new level fades in over half a minute. The seasons move them in small steps. Arriving in a world, the levels come some seconds after the currents.
- Rain raises the levels only as much as it swells the great rivers (all of it with `[Floods]` on), and only while your game draws the server's weather (`straumr weather` says). Otherwise it moves only the currents in your game.
- Where a square's banks hold the river lower, the river runs down into it at 3 cm a metre. In spring that can be a drop of over a metre in about 50 m.
- The server's own water stays at the sea's level, as with the tide. Only players' games raise the rivers.
- Valheim's own worlds barely show the levels: their banks are low and their rivers slow.
- In a wide bay or sea arm with several river lines far apart, the current can swirl oddly between them, or drift across toward a river a couple of hundred metres off instead of along the one on the map.
- Where two ways to the sea through the same water are nearly equal, the water splits between them and lies slack where they part.
- A lake no river runs into lies still apart from the wind, and so can broad water far from any river.
- If a Riverheim update changes how its rivers are built, Straumr can't read their water. It works it out from each river's length instead, and says so in `BepInEx/config/Straumr-stream.txt`.

### Sea and surface
- The tide raises the sea one 64 m square at a time, so neighbouring squares can differ by a few centimetres. The rivers' rise slopes smoothly across the squares. Waves behind breakwaters are as coarse as Valheim's own, 64 m.
- A few things Valheim keeps at its fixed sea level do not move with the tide or the rivers' rise: mist, embers, the beach's shading, and where the world placed things.
- Valheim's own water surface does not move with the current. Straumr's moving surface lays ripples and glints on it within about 60 m of you, mostly on running water. Beyond that, on slack water, and wherever the moving surface is off, the foam and the things floating on it show the flow.
- Straumr knows the water within about 160 m of each player (further where they have been lately). A boat 1 km from every player is out of its sight. The server runs no water of its own except the tide's clock, the log drive and what it decides about dams, cofferdams and pools.

### Other mods
- Rivers and Lakes' own streams and lakes move as that mod draws them. Straumr adds its stream jets and the wind's drift on its lakes once Rivers and Lakes shares its water with other mods.
- ValheimRAFT's vehicles ride the current, but the boat speed line, the current turning a boat (`BoatTurn`, `BoatYaw`), the anchor for empty boats and the waves' drift on boats cover Valheim's own boats only so far.
- Log drives have been checked in the lab, not yet in play.
- With Expand World Size, the sea solve at load takes about 10.7 to 13.6 s on a 22 km edge (1.5 s on a default world), and the atlas's whole-world build about 30 s instead of 9 s, with about 30 MB more memory while the field is made. The sea's currents at full flood are weaker on a big world.

## Settings

Everything is in `BepInEx/config/wackah.straumr.cfg`, and most settings apply within a couple of seconds. **Shared** settings come from the server when you join one (yours apply again when you leave); the rest are each player's own. Settings marked *server* only do anything on the server. 84 settings are shared and 48 are each player's own.

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
| `TidalMonth` | 16 | Game days from one full moon to the next (a seasons mod can set it instead). |
| `Whirlpools` | 1 | How hard whirlpools spin (0 = none). |
| `WhirlpoolGrip` | 1 | How hard a turning whirlpool takes hold of swimmers (0 = the old gentle drift and the eye's tumble only; up to 2). |
| `WhirlpoolFunnel` | 3 | How deep a whirlpool's surface sinks toward its eye, times what its spin would hold (1 = physical; 0 = flat; up to 6; at most 4 m). |
| `Rapids` | 1 | How hard white water handles swimmers (0 = not at all, up to 2). |
| `Strid` | 1 | Was `Undertow`. How hard a river's slot pulls swimmers under now and then: above 1 harder, not more often (0 = never, up to 2). |
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
| `RiverLevels` | true | Rivers stand above the sea, climbing gently toward their sources, higher in spring and lower in winter. Off: all water at the sea's level, as before 0.1.3. |
| `RiverLevelScale` | 1 | How high the rivers stand, times what their currents need (0.5 = half as high). |
| `RiverFlooding` | 0.5 | How far raised rivers may spread over low ground beside them: each 64 m square's water may grow by up to this share of what it already holds (0.15: banks get wet, flat meadows mostly stay dry). Where a river would spread further, it stands lower there. |
| `SpringFlooding` | 4 | In spring (with Seasonality), how many times `RiverFlooding`'s spread the rivers may take (1 = the same all year). |
| `Splits` | true | A loop's far side or a lake's second way out carries its share of the water (off: a drift at a third of the river's speed, as before 0.1.3). |
| `Chutes` | 1 | Where a river runs steeply down into a stretch its banks hold lower, how much faster it runs too (0 = only the look and the sound). |
| `LeeEddies` | 1 | Eddies curling back into the lee of sand bars, islands and points (0 = none, up to 2). |
| `LeeEddyCover` | 0.4 | How deep the water over a bar's top must be before the river runs straight over it and its eddy is gone (m). |
| `Breakwaters` | 1 | Calmer waves in the lee of what players build (0 = off, the game's own waves). Not under NorgerWater. |
| `NaturalShelter` | false | With `Breakwaters`: the land as the world made it calms the waves in its lee too. It changes the waves near land everywhere, so it's off by default. |

**`[Stage]`** (shared): rivers rising and falling.

| Setting | Default | What it does |
|---|---|---|
| `Seasons` | true | With Seasonality, the rivers follow the seasons. |
| `Spring`, `Summer`, `Fall`, `Winter` | 1.4, 1, 0.8, 0.7 | How fast rivers run mid-season, times usual. How high they stand goes with this squared, as far as the banks allow. |
| `Snowmelt` | 0.35 | In spring, how much faster snow-fed rivers run at the afternoon peak than at dawn; they rise with it. |
| `Weather` | true | Rain swells the rivers: creeks at once, big rivers later and longer, and the great rivers' levels a little. The rain is the lowlands' as Valheim draws it, the same for every player. |
| `RainSwell`, `StormSwell` | 0.3, 0.6 | How much faster after a good rain, and after a thunderstorm (light rain less). |
| `IceFreeSpeed` | 0 | With Seasonality's freeze: a 64 m zone where a river runs at least this fast (m/s) stays open. 0 = everything freezes. |

**`[Dams]`** (shared, except `SpillSheets`): dams and their ponds. Experimental.

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | true | Dams: walls across a river hold a pool back. Experimental, on by default. On a dedicated server it needs `[Currents] RiverLevels` on (the default). |
| `Ceiling` | 60 | The highest a dam's pond or a kept pool may stand over the sea (m; 9.5 to 100). Above 10 m Straumr's own water volumes carry it, in each player's game; an older Straumr can't swim, sail or float in that part. |
| `DragUnder` | true | A hole broken low in a dam's wall, or an open gate under its lintel, draws swimmers toward it and through. |
| `SpillRamp` | false | Draw a spilling pool's level over its dam's crest and down a 2-4 m ramp below it, instead of a step in the wall. The server's value is everyone's, because it changes the level a swimmer or boat feels. Off by default. |
| `SpillSheets` | true | Your own. Where a dam's pond pours over its wall or through a breach, draw the water hanging down the wall as moving water. |

**`[Cofferdams]`** (shared, except `WaterMask`). Experimental.

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | true | Walled-off patches of water drain and lay the bed bare; break the ring and they refill. Experimental, on by default. |
| `DrainRate` | 0.5 | How fast a closed cofferdam's water falls (m a real minute). |
| `WaterMask` | true | Your own. Hide the game's water over a drained cofferdam so the pit seen from past 128 m shows its bed. Off under NorgerWater. |

**`[Pools]`** (shared, except `OwnSurface`). Experimental.

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | true | Water cut off from the river with ground alone keeps its own water. Experimental, on by default. |
| `OwnSurface` | true | Your own. A kept pool, and a dam's pond, drawn as a body of water of its own at its level instead of bending the river's sheet up to it. Off under NorgerWater. |

**`[Floods]`** (shared). Experimental.

| Setting | Default | What it does |
|---|---|---|
| `Enabled` | false | Rain floods the rivers' levels, each river by its own size's rain. Off: the levels follow only the great rivers' rain. Experimental, off by default. |
| `FloodScale` | 1 | How much of the rain's swell the rivers' levels take (1 = as much as their currents; 2 = twice; 0 = none). The currents don't change. |
| `FloodRoom` | 8 | In a flood, how many times `RiverFlooding`'s spread the rivers may take over low banks and flood plains. |
| `FloodPonds` | 256 | In a flood, how much water may stand in hollows apart from the river in each 64 m square (m²; 16 m² outside floods). |

**`[LogDrive]`** (shared): logs and floating things at barriers.

| Setting | Default | What it does |
|---|---|---|
| `Screens` | true | Built pieces with gaps at the waterline pass water and catch what floats down. Off: every piece in the water walls it. Changes rings and dams already built of such pieces. |
| `ScreenPieces`, `WallPieces` | empty | Piece prefab names (comma-separated) to take as screens, or as walls, whatever their colliders say. |
| `DamApproach` | true | Still water backed up against a wall or a dam drifts toward it at 5 cm/s at least. Never felt by a swimmer or a boat. |
| `ServerDrive` | true | *Server.* Logs and floating items in a river with nobody near are carried on down it until a barrier catches them. |
| `Items` | true | *Server.* Floating items are carried too, not only logs. |
| `CaughtIsBase` | true | An item lying caught at a barrier counts as inside a base. |
| `ThroughUnexplored` | false | For testing on a lab server: carry things on into zones nobody has explored. |

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
| `WhiteWater` | true | Fast water, and rivers running steeply down into lower water, look white (only the look). |
| `Spray` | 0.3 | How often spray kicks up off the fastest white water. |
| `SoftEdges` | true | Foam fades where it meets a bank or a rock instead of ending on a hard line. |
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
| `FlowPeriod` | 1.8 | How long each copy of the ripples is carried before it fades and restarts (s). |
| `Lift` | 0.04 | How far the sheet sits above the game's water (m). |
| `SoftEdges` | true | Fade the sheet where it meets banks and rocks (turn it off if the sheet vanishes). |

**`[Hud]`** (your own): lines under the minimap.

| Setting | Default | What it does |
|---|---|---|
| `TideClock` | true | While you're by tidal water: how the tide runs and when it turns. |
| `BoatSpeed` | true | While you're aboard a boat: its speed through the water and over the ground, and how much the current adds to it or takes from it along its heading. |

**Your own, the rest of the look:** `[Sound]` `Enabled`, `Volume`; `[Mist]` `Enabled`, `Amount`.

**What the world gets** (*server*): `[Driftwood]` `Enabled`, `EveryMinutes`, `Prefab`; `[Fish]` `Enabled`, `EveryMinutes`, `PerSpot`, `Kinds`; `[Rocks]` the rocks set at swallow holes and river sources.

**Admin and tools:** `[World]` `StandingRuins` (shared) and `NewWorldSeed` (every new world this game creates gets this seed; made for dedicated servers, which can't be given one); `[Atlas]` `Port`, `HiddenSeeds`; `[Diagnostics]` `CurrentProbe`, `PerfCounters`, `LocationCheck`, `WorldGenCommand` (testing tools, off by default).

## Commands

Type these in the console (F5). A command marked * needs `devcommands` first, or admin rights on a server.

| Command | What it does |
|---|---|
| `straumr here` | the water at your feet, each part of the current (river, tide, sea, rocks...), what nearby is bending it, its way to the sea, and the river's level and state: how full it runs and why, how much water it carries, the next four hours |
| `straumr report` | writes a file describing the water where you stand, including what's standing in it and the settings package's size, for bug reports |
| `straumr selftest` | checks Straumr is working right where you stand (each check PASS, WAIT, SKIP, NOTE for a known limit, or FAIL) |
| `straumr waterlevel` | the water level here: the sea's usual level, the tide and the river's rise, the water's actual surface, and whether they all agree |
| `straumr weather` | the rain the rivers follow: this weather spell's in the lowlands, your sky against it, how swollen the rivers are, whether your weather is the server's, and the next rains |
| `straumr flood` | the levels' hourly step, the flood here and ahead, and what holds the water here |
| `straumr dam` | the dams near you, their pools, what the river below carries, and each one's gates; `straumr dam show` marks where each dam within 1 km spills, in your game; `straumr dam off` clears the marks |
| `straumr pools`, `straumr cofferdam` | the kept pools and the cofferdams near you and their state |
| `straumr gate 0.5` | needs `devcommands`: the door you look at, set as a sluice gate that far open (0 to 1; `off` = as the door stands) |
| `straumr work` | the water power within 10 m and the best drop within 30 m, as another mod would read it from the API |
| `straumr drift 5` | where something floating dropped here ends up over 5 minutes (up to 10), as the water runs now |
| `straumr drive` | the barriers across the rivers near you that catch logs |
| `straumr seam` | where Rivers and Lakes' streams run into Straumr's water (`on`, `off`, `show`) |
| `straumr perf` | how hard Straumr is working (turn on `[Diagnostics] PerfCounters` for the lookups too) |
| `straumr overlay` | turns the moving surface off and on in your game, to compare, and says whether it's running |
| `straumr atlas` | the seed atlas in your browser (works from the main menu too) |
| `straumr tide` | how the tide is running and when it turns* |
| `straumr tide flood` | hold the tide (`flood`, `ebb`, `high`, `low`, `spring`, `neap`) in your game; `straumr tide auto` lets go* |
| `straumr races` | the strongest tidal races and whirlpools within 4 km* |
| `straumr rain thunder` | a test rain on the rivers (`light`, `rain`, `thunder`; `off` ends it): on a server for everyone, and the server checks its admin list* |
| `straumr map` | write this world's flow map to a file* |
| `straumr waterstep` | a test: steps the water up zone by zone near you; `straumr waterstep off` undoes it* |
| `straumr ownwater`, `straumr mask` | compare built water drawn as bodies of their own, and the water mask over drained cofferdams, against the old look, in your game* |
| `straumr kit`, `straumr scenedump`, `straumr damtest`, `pooltest`, `spilltest`, `drivetest`, `seamtest`, `screens`, `sheettest` | testing tools for building test scenes, dumping the world round you for a bug report, and checking Straumr's own rules; mostly for the author and for reports* |

## Known issues

0.1.5 fixed many of the drawing faults that 0.1.4 listed (see [CHANGELOG.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/CHANGELOG.md)). This is a beta. Straumr was checked in a lab and played by one player on a dedicated server, not yet by a group of players.

**Tiers**, from worst to least:
- **Breaking:** a crash, a broken save, or water plainly wrong in ordinary play. A release does not ship with one.
- **Noticeable:** wrong where you will see it in normal play, and it stays.
- **Situational:** needs a particular build or timing (the line says which). It settles, or it is small.
- **Minor:** centimetres or seconds, and rare.
- **Not built yet** is its own list at the end. Those are limits, not faults.

Each line says what you see, how you get there, whether it settles, and what to do where there is something to do. A ring is a closed loop of walls in a river. A line that needs another mod starts with its name. Drawing faults change only what you see, not the water's levels or flows.

### Breaking

None is open.

### Noticeable

- **Springs and drains.** You see the current and foam pour out of, or into, one spot mid-river. It happens in some places on Riverheim worlds. A test of 214 squares of a Riverheim 1.2.1 world found 88 such spots (70 out, 18 in). Riverheim 1.3.1 gave 25 and 0. It does not settle. A fix is planned.
- **With Seasonality: a pond in a freeze.** In Seasonality's freeze, the rivers, dam ponds and kept pools are drawn at the sea's level, so a raised pond shows its bed dry until the thaw. A dam's pond also gets no river inflow all winter, so it drains toward its crest and refills in the thaw (from the code, not measured in play). A kept pool is drawn the same way, but nothing drains it (from the code). A frozen pond is not ice, a wall shows no falls, and logs lying on the ice can still be pushed about. It ends in the thaw. Planned.

### Situational

- **Falls tear at joins (stepped wall).** You see a slit between two falls, or a dry edge by a pillar. It happens where a neighbour drops past a step's side, beside pillars under about 1 m wide, and where runs meet at one point. It stays.
- **Falls have no sides (any wall).** You see the thin sheet's edge from beside a wall's end or a step. Past a wall's end the fall can hang as a free curve. It stays. Closing the sides is planned.
- **While you change a dam's wall, its pond and falls are drawn again (while editing).** For about a second (up to 3 s) the pond can show as a flat sheet with straight sides, and a fall can jump or vanish. It settles when you stop. The redraw is also in 0.1.4.
- **A stutter (big dam).** You see a short stutter after a big change to a big dam's wall, with the pond in view. It was seen in play on the release build. It passes after a moment. It is being looked into.
- **Water at a wall's ends (after a breach or rebuild).** A pour can start 3.6 to 4.5 m from the wall's top, and loose sheets can hang in the breach. At a diagonal wall end, a fall's top can stand in the air, up to about 1 m, once nearly 2 m (also in 0.1.4). It is not checked if it settles.
- **A hole under a wall piece draws no water.** You see no water at a hole that has wall above it and the pool above that, and the pond stands lower than it should. It stays. It was also in 0.1.4. It will be fixed with holes through a wall's thickness.
- **A spill over a low saddle (huge dam).** The spill can be drawn up to about 3 m above the channel beside it, instead of pouring down. It was not measured again after the new spill drawing.
- **The pond ends early (flying fast over a big pond).** A big pond is not drawn out to the sight limit, and its pieces can arrive late. The pieces arrive by themselves. This moves up first if players report it.
- **The level is wrong for 10 to 20 s (two walls about 28 m apart).** You see the water between two walls about 2 m too high (0.13 to 2.32 m in tests), then it is right. To see it: build two walls about 28 m apart across a river, let the upper one spill, then break the lower one. It settles by itself, so wait. After such a hand-over the stored water can also differ from a fresh look at the same walls by up to 0.29 m (lab only). It was also in 0.1.4. A fix is planned.
- **Some distances do not scale (Expand World Size, "Stretch world" 2).** A few distances are fixed in metres, so at stretch 2 they cover less of the land. It was tested only with Expand World Size 1.45, in the lab, not in play. It stays. It is left for later.
- **Riverheim's map is cut off (Expand World Size, radius not matching the stretch).** Riverheim's map keeps its own size, times the stretch. On a 7 km radius world in the lab, 45 rivers had no way out (0 on a full-size world). On a bigger radius, the extra is sea. It stays.

### Minor

- **Small drawing glitches.** A dark strip can show between runs off a wide wall top (it may be gone). The water shared between falls off a stepped wall's side is not tuned. Two dams about 10 m apart can draw a small sliver of water past a wall's end. The lab flags a few places at rings and cofferdams, as since 0.1.3. All are rare.
- **About 3% low after a restart (ring).** You see the pond above a breached ring come back about 3% low (756 to 921 m³ of about 28,800; 2.5 to 3.1 cm). To see it: breach a ring so the breach joins two ponds, then restart the server (a clean stop or a crash). It is not checked if it settles. It was also in 0.1.4. A fix is planned.
- **The last 8 cm (ring removed soon after a breach).** You remove a ring within a minute of its breach. Its pond drains in about 10 s, then the last 8 cm drops at once. It ends with that drop. In 0.1.4 the whole pond vanished at once. A fix is planned.
- **A join or leave while a seed world builds (Expand World Size, atlas).** Do not join or leave a world while an atlas seed world builds. A seed world takes the played world's size, not Valheim's.

### Not built yet

Limits, not faults.
- **No backwater above a pond.** A pond does not raise the river above it. Where a dam's water is open to the river above, the water goes round at the river's own level. Planned.
- **Dirt and dug cuts.** A block of dirt alone is not a dam. A dug low area beside a pond fills as part of that pond and empties when the dam goes. A dam sends a river down a dug cut only if the cut ends lower than the dam's crest. If not, the river, the cut and the low area become one pond that spills over the dam. Not tested: a dug low area with its own exit back to the river. Planned.
- **Dead-end hollows.** A hollow with no lower exit is not a basin of its own. Planned.
- **Ponds have no ripples or glints.** The river's surface has them. A later version may add them.
- **Surges follow the river's line.** A surge does not spread into side channels, round islands or into bays. Planned.
- **With Expand World Size: tides are untuned on resized worlds.** A bigger radius makes tidal streams about twice as fast over the same land. At "Stretch world" 2, tidal races mostly vanish. Left for later.
- **With shudnal's Seasons** (not tried in game): both mods run but do not work together. They keep separate season clocks, and both set the water's height in winter. Straumr's currents do not stop on Seasons' ice, and its ponds do not freeze. Support is planned.

### Not checked yet

- two or more real players in the same pond at once (no group of players has played it);
- Straumr with NorgerWater in game;
- the log drive in play;
- roofs, fences and docks standing in a pond.

## Roadmap

Plans, not promises; the next steps depend on what players find.

- **Next:** making built water and dug channels part of one water model, so ponds, dams and the water backed up behind them follow the ground players change; then the map showing the rivers, lakes and dams players make.
- **Later:** stones just under the surface showing a riffle, fish that hold in seams and behind rocks, ice that freezes where the water stands, culverts and other small water works, a flood that travels down a river after rain.
- **Further on:** water that remembers what happened: lakes filling and draining, a log jam backing a river up and then bursting; a breach's surge worked out as a true travelling wave (today it is a front that runs down the river).
- **Not planned:** new models or textures (water wheels, mills, nets). Those belong to other mods, built on the API.
