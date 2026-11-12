# Straumr

**Standalone: BepInEx only.** Straumr needs nothing but BepInEx: no framework, no library, no config mod. Riverheim, Rivers and Lakes and Worldhalla are picked up when they're installed, by reflection, and are never required.

**Living water for Valheim.** Rivers flow downstream and stand above the sea, rapids run white, the sea has currents and tides, and whirlpools turn off the ends of tidal straits. Swimmers, boats and floating things all ride it. New in 0.1.3: dams that hold a river back, cofferdams that drain a patch of water and kept pools that hold water you dug (experimental, **on by default**), and floods (experimental, off by default). New in 0.1.5: built water drawn much closer to how it behaves (the water over a wall's own top, falls and spills round dam ends, ponds full when you join), ponds that come back as at the last world save after a crash, and any world size with Expand World Size.

*Straumr* is Old Norse for "current". This is a **beta** (0.1): it works, it's been played on, and it's still being tuned. Feedback welcome on the [Straumr Discord](https://discord.gg/bU9AD8hb8M).

**Riverheim 1.2.x and 1.3.x both work.** Riverheim 1.3 is a Worldhalla plugin; Straumr reads it through Worldhalla's public API, still without needing Worldhalla itself. 0.1.2 reads Riverheim 1.2's insides, which 1.3 removed, so its rivers break on Riverheim 1.3: update Straumr when you update Riverheim. Two things to know before moving a world to 1.3.x (details under *Works with*):
- Old Riverheim 1.2 worlds convert one way the first time 1.3.x saves them. Back up the world's `.fwl` and `.db` files first.
- A world made with Riverheim_Config's settings needs Riverheim_Config 3.1.1 or later on Riverheim 1.3.x, or its terrain changes.

| | Riverheim 1.2.x | Riverheim 1.3.x |
|---|---|---|
| **Straumr 0.1.3 and later** | works | works |
| Straumr 0.1.2 | works | rivers break: update Straumr |

**Multiplayer\*:** install Straumr on the server and on every player's game. \*Tested on a dedicated server with lab clients and the join, config-sync and restart tests; **not yet tested with a group of players.**

## New in 0.1.5


0.1.5 redraws how built water looks, fixes how it comes back after a crash and what a ring's breach does to the pond above it, and reads worlds of any size with Expand World Size. The water's behaviour (levels, storage, flows) is the same as 0.1.4's apart from those fixes; what else changed is where Straumr draws water the server has already decided on. The drawing was played by one player on a dedicated server through successive builds, each fixing what the last one showed (see *Known issues* for what is still open).


- **After a crash, ponds come back as at the last world save.** The server now writes the dams' file (`BepInEx/config/Straumr-water/<world>.dams.txt`) only with the world's save, as it does the kept pools'. In 0.1.3 and 0.1.4 it was also written between saves, every 30 s while a dam changed; after a crash or a kill that file was newer than the world, was moved back onto the world's clock, and ran the minutes in between a second time, so a pond came back ahead of the world, with water made or lost. A clean stop was never affected. A pond's saved state is now the world's, like every other object: what happened after the last save is gone with it.

- **Water over a wall follows the wall's own top.** The top is drawn as a height surface: water runs across it as it would over ground, a sheet across a flat top at its critical depth, a drop at every step down, a hollow in the top filling and leaving by its lowest gap, and the fall to the water or ground below starting at the top's real downstream edge. A wall several pieces thick (four stone walls side by side), a stepped wall and an uneven top draw their overflow across the top instead of a waterfall hanging in the air over it, and the last metres of a long wall past what the dam's record covers are poured too. A level shelf below a wall carries the water across it to its edge and over. Water leaves by every edge it reaches, the sides too, not only the downstream one.

- **Falls that join up.** Stretches of one wall face in line meet, with no gap between them and no dry strip between falls; falls are joined across a wall's joins; a fall's foot is fanned where a column goes into the water a row later; a wall's end is followed on past a stub and poured where the dam's record holds no wall.

- **Falls that land on dry ground run on down it.** A fall that lands on a bank and never reaches the water carries its water on from where it landed, down the ground the steepest way to the water it reaches, a stream's width. None runs where water already stands within 3 m of the fall's foot, where it reaches none, or where the fall was under a metre, and a run never climbs a step of more than 0.5 m. Water running on what's under it is milky by its speed, at most 0.15 white.

- **Spills beside a dam are drawn.** Where a pond rises over a low point beside its wall or goes round the wall's end, the spill now runs over the land from the pond's edge to where it joins the river (or to a hollow's floor), at the flow the dam's record sends round at the pond's level. It is drawn from edges within 32 m of the way the record sends it, where the edge stands up to 1.5 m over the ground, and not at all while the pond stands under its level.

- **Fewer seams.** A big pond's pieces beside one that is worked out again are worked out and swapped in with it, so a seam no longer shows while you build at it (checked on a scripted pond: 0 of 32 border points unshared, 7 before). A pond piece built in the tick its zone loads no longer cuts a hole round the sea's level in the pond.

- **A pond is full when you join.** Built water is drawn at its level from the moment a game sees it. Before, a game joining a server drew a pond at the river's level for about half a minute to a minute, and then it rose (in play, 53 s and 8.9 m on a big pond). This changes only what is drawn.

- **Expand World Size: any world size.** Straumr no longer assumes a 10 km world. It reads Expand World Size's live radius, edge and stretch by name (nothing of it is needed), and the sea's currents are solved out to the world's own edge; before, nothing past 10.5 km was solved and everything past 10.4 km counted as open sea. On a default world nothing changes: the same field, byte for byte (see *How it was checked*). Limits are under *Known issues*.

- **The atlas's `@here` works on a dedicated server** (its admin starts it with `[Diagnostics] WorldGenCommand` = `atlas`; the key is written to `BepInEx/config/Straumr-atlas-key.txt` while it runs): the server's own world, for looking at the water of a live world.

- **Mod makers:** on a dedicated server a pond's lift is now part of `StraumrApi`'s `Sample`, river rise and river state (before, it counted as 0 there). The server's own river levels stay at the sea's level. `ApiVersion` stays 1.

- **A ring's breach no longer leaves a pond rising over it.** In 0.1.4, after a ring (a closed ring of walls between two dams) was breached, the pond above it could stay held together with the pool below, on out-of-date figures, and rise over the ring's crest for ever: in the lab a pond of about 4 ha drawn 3 m over the river, and the ring's inside about 1.1 m over its stone. Now the pool is judged again from the walls as they stand, finds the river going round the ring at its own level, and its water leaves round the ring at the river's level, at the water's own pace. The lab's water accounting balanced through it: 0 m³ made or lost. Also fixed, mostly: taking a ring away while its pond was held made the whole pond vanish at once in 0.1.4 (113,514 m³ in the lab's first case); now the pond drains round the gap in about 10 s and its last ~8 cm goes at once (see *Known issues*). On 0.1.4 the only thing found that clears the first fault was removing the ring, which deleted the pond's water. The one-edit lab scenario for it is `fuzzmut-20261110-3-MF`.

- **Expand World Size's "Stretch world" works.** Riverheim's rivers were read at their original size, so on a world stretched to 2 every river line lay at half its distance from the middle, and the currents were pulled toward lines on dry land, which showed as springs and drains in the water (a player's report). Now Straumr reads the rivers' lines and widths at the world's stretch. On that player's setup (radius 21 km, edge 22 km, stretch 2, water depth 0.75, Riverheim 1.3.1) in the lab: the share of river-line points on water went from 34% to 99.4%, and the rivers with no way out from 110 (of 325) to 0. The worst step between neighbouring river levels fell from 49 m to 32 m (at one river mouth). Counted over the same land, 6 springs and 0 drains against 25 and 0 on a default world (8 m cells). A default world is unchanged, byte for byte (see *How it was checked*). A dam built in the lab on that setup holds, spills and breaches. Limits are under *Known issues*.

- **The atlas no longer undoes Expand World Size in the world you play.** Opening the atlas for another seed, in a world with non-default Expand World Size settings, used to undo its changes for the world you were playing until a restart: all of the ground changed (265 m on average), and new zones saved their trees, rocks and locations on the wrong ground. The atlas's seed worlds are now built beside the played world with Expand World Size's patch left alone, and the live ground measured 0% changed (it was 100%). A seed world built while you play takes the played world's radius, edge and stretch. If Expand World Size's inside differs from the version Straumr was written against (1.45), the atlas builds no seed world while you play rather than risk yours. The atlas page now reads the world's edge, so a bigger world shows whole, with its stretched rivers, and the detail squares reach the edge.


## New in 0.1.3

- **Rivers stand above the sea.** Each river's surface climbs gently toward its source (up to 1.34 m above the sea in an ordinary season on the Riverheim world it was checked on), stands lower where the banks are too low to hold it, and slopes smoothly from one 64 m square of water to the next. Swimming, boats and floating things meet the surface you see. `[Currents] RiverLevels`.
- **Seasons, rain and snowmelt move the levels as well as the speed.** With Seasonality the rivers stand higher in spring and lower in winter and spread over flood plains in spring; rain raises them a little; rivers fed by mountain snow rise on spring afternoons. The rain is Valheim's own lowland weather, worked out again from the server's clock, so every player gets the same rivers. Driftwood now washes up just above the spring high water.
- **Dams** (experimental, on by default: `[Dams] Enabled`). A wall of pieces across a river, bank to bank, raises a pool behind it that fills on the clock and spills over its lowest part, or round its end if the ground is lower there. The river below carries only what comes over. Break the wall and the pool drains through the gap as a surge that runs down the river. A door in the wall is a sluice gate. A hole broken low in the wall pulls swimmers toward it. The pond is drawn as a body of water of its own, to the wall and to the pieces you built round it.
- **Cofferdams** (experimental, on by default: `[Cofferdams] Enabled`). Wall off a patch of river, lake or sea on every side, standing above the water, and the water inside drains and lays the bed bare to build on. Break the ring and the water comes back through the gap. Break a dam above a dry cofferdam and the ring fills from that dam's pond, over a time you can watch (short fills at about real pace, long ones sped up).
- **Kept pools** (experimental, on by default: `[Pools] Enabled`). Dig a basin, let river water in through a channel, fill the channel in again, and the basin keeps its water instead of following the river. Dig it deeper and the same water stands lower.
- **Floods** (experimental, off by default: `[Floods] Enabled`). Rain raises each river's level by the rain its size would see: creeks within an in-game hour of a storm, great rivers about a day later. A high river holds the rivers that run into it up.
- **Built water is the server's.** On a server, the server decides each dam, cofferdam and kept pool and every player's game follows it, so everyone sees the same water. The server keeps dams, cofferdams and kept pools with the world, so a restart picks them up as they were: a pond still filling, a spill, a breach, a ring still draining or filling. After a crash or a kill they come back as at the last world save, like everything else in the world.
- **Chutes and carried water.** Where a river runs steeply down into a stretch its banks hold lower, it looks and sounds white and runs faster. A big river sounds big, and a loop's far side or a lake's second way out carries its share of the water instead of a flat 30%.
- **Whirlpools** draw swimmers in toward the eye and the sea dips into a funnel there. **The Deep North's ice stays dry:** the tide fades out over it, so water no longer stands on the game's ice.
- **Riverheim 1.3.x works** (through Worldhalla, by reflection; see above and *Works with*). Straumr no longer throws if Riverheim's insides change: it says so once and carries on with Valheim's own rivers.
- **Calmer waves behind what you build** (`[Currents] Breakwaters`), and **eddies** that curl back into the lee of sand bars, islands and points (`[Currents] LeeEddies`).
- **Log drives.** Logs and floating items in a river with nobody near are carried on down it by the server until a wall, a dam or a fence of closed pieces catches them. Open frames and lattices let water through and catch what floats down against them.
- **Multiplayer hardening.** A game with an older Straumr, or none, is told on screen that its water will differ. A damaged or unknown part of the server's settings costs only itself.
- **The river "Undertow" is now "Strid"** (`[Currents] Strid`), so it isn't mistaken for the sea mod of that name. Your setting carries over.
- **For mod makers:** the API gains river levels, a river's outlook for the next four hours, floods, dams, cofferdams, water power and gates, drift paths and wave shelter (`API.md`). `ApiVersion` stays 1.
- **New console commands and atlas layers:** `straumr weather`, `flood`, `dam`, `dam show`, `gate`, `pools`, `cofferdam`, `work`, `drift`, `drive`, `seam`; atlas layers for river levels, fords, water power and built water.

## What it does

**Rivers**
- Every river flows from source to mouth, faster where it narrows, slow where it widens into a lake, and slower near the banks than mid-stream. Creeks run about 1 m/s and great rivers about 1.3 m/s (you swim about 2).
- Water slides along the banks and round islands instead of running into them, and carries on out past a river's mouth before it fades.
- Where several rivers share the same water (Riverheim often runs a few through one channel, sometimes in opposite directions), that water gets one way down: its shortest way to the sea. Rivers keep the direction Riverheim drew unless the other way is less than half as long, so they rarely run back toward the hills.
- A river that ends in a lagoon or bay runs out through it to the sea.
- **Rivers carry their water** (new in 0.1.3). Straumr estimates how much water each river carries: from the land draining into it on Riverheim worlds, from the length of channel above it on Valheim's own. A river sounds as full as the water it carries. Where the water has two ways down (a loop, or a lake with a second way out), the second way carries its share, worked out from how easily each runs (`[Currents] Splits`), instead of a flat 30% of the river's speed.
- **Chutes** (new in 0.1.3). Where a river runs down into a stretch its banks hold lower, the slope (up to 3 cm a metre) shows as white water, in the foam and the sound, and the river runs faster along it, up to the typical river's speed (`[Currents] Chutes`; 0 = only the look and the sound).
- **White water** where fast water runs shallow, worked out from the Froude number (how fast the water runs for how deep it is), the way real rapids form. White water shoves swimmers about, now and then pulls them under for a moment, and wears them out.
- **Strid** (called Undertow in 0.1.2): where a river runs fast through a slot, a channel deep for its width, its surface stays smooth while the water below now and then pulls a swimmer under for half a second or so, with at least 15 s between pulls in the worst slot (40 s in a mild one). Only true slots: from three times as wide as deep, 2 m deep and 1 m/s, in full at one and a half times as wide as deep, 3 m deep and 2 m/s. It fades out as the water turns white, which handles swimmers its own way. It costs no stamina and, at the default setting, never takes anyone more than 1.5 m below where they swim. The Strid is the River Wharfe's slot, which it is modelled on.
- **Wading:** walking in a current over your ankles, the water pushes you downstream, more the deeper it is and with the speed squared. At a usual river's 1.2 m/s your feet hold up to about 0.6 m deep, and a metre deep carries you about 0.2 m/s; in a 2 m/s run, half a metre deep carries you 0.2 m/s and three quarters of a metre 0.6 m/s. Walking upstream is slower going. It never takes more than half your slowest walking pace, so you can always walk out.
- Swimming costs the usual stamina whichever way you swim, as in real water: the current changes where a stroke takes you, not what it costs. `SwimWithCurrent` and `SwimAgainstCurrent` can change that (0.85 makes going with a river a little easier, 1.25 fighting it a little harder).
- Rocks, logs, posts and walls that break the surface bend the current, with slack water and a small back-eddy behind them; the water stalls against their upstream face. A rock counts only as big as the part of it the water meets at the surface. A stone just under the surface doesn't stop it. A wall across a channel stops the flow, and the water above it lies still.
- **Eddies behind the ground** (new in 0.1.3): where a sand bar, an island or a point sticks out into a river, the water past its end curls back into its lee, and over a bar that is out of the water or barely covered the river splits round it instead of running over. It goes by how much water covers the ground right then (the tide, the river's level, the waves), so it comes and goes with them (`[Currents] LeeEddies`).
- Bends push surface water to the outside bank, shallow rivers alternate pools and riffles, and side rivers meet in a swirling seam.
- **Weather and seasons:** rain swells the creeks first and the big rivers later; a thunderstorm swells them more. The rain is the lowlands' (Meadows, Black Forest, Plains, Mistlands) as Valheim draws it for each weather spell, worked out again from the server's clock, so it is the same for every player wherever they stand. With Seasonality, spring snowmelt runs high and winter runs low (or freezes, if Seasonality freezes the water). The rivers' levels follow all of it.
- Dig a channel below the water line between two waters and the current runs through it, in your game.

**River levels** (new in 0.1.3)
- Rivers stand above the sea. Running water needs a slope to push it, so each river's surface climbs gently from the sea toward its source, worked out from the speed the river shows: steeper where it runs fast and narrow. On the Riverheim world it was checked on, the banks let rivers stand up to 1.34 m above the sea in an ordinary season (`[Currents] RiverLevels`; `RiverLevelScale` makes them higher or lower).
- The banks decide how high. Where a 64 m square's water can't rise to the river's full height without spilling over land away from the river or leaving ponds in hollows, it stands lower there, and the river runs down into it at up to 3 cm a metre (see *Chutes*).
- The water slopes smoothly from one square to the next, with no steps at their edges. Swimming, boats and floating things meet the same surface you see.
- With Seasonality the rivers stand higher in spring and lower in winter: how high goes with how fast they run, squared, so mid-spring's 1.4 times the speed stands about twice as high where the banks allow. In spring they may also spread over more of the low ground beside them, the flood plains (`[Currents] SpringFlooding`; `RiverFlooding` says how far they may spread at all), which dry out again as the season turns; rivers fed by mountain snow rise in the afternoons. In Seasonality's freeze the rivers settle to the sea's level.
- Rain raises them a little, following the great rivers, which swell later and less than the creeks. With `[Floods]` on it raises them by each river's own size's rain (see *Floods*).
- The levels change at most once a weather spell (five minutes into it), and a new level fades in over half a minute. Every player's game works the same surface out from the world, the shared settings and the server's clock.
- Driftwood washes up just above each river's spring high water: a strandline that shows how high the water can come.
- Valheim's own worlds barely show it: their banks are low and their rivers slow.

**Dams and ponds** (new in 0.1.3, experimental, on by default: `[Dams] Enabled`; the server's setting is everyone's)
- Pieces you build (with any ground you raise beside them) that stand across a river from bank to bank, 10 cm or more over its ordinary level, dam it. Raised ground alone isn't found as a dam. The water backs up behind the wall as a level pool over everything joined to it below that level, at the river's own flow, until it spills over the lowest part of the wall or finds a way round.
- The pool fills on the in-game clock: an hour's filling takes an in-game hour (75 real seconds with Valheim's 30-minute day). Below the dam the river carries only what comes over or through the wall, and runs lower while the pool fills.
- A pool stands up to `[Dams] Ceiling` over the sea (60 m by default, up to 100). Above Valheim's own water level (10 m over the sea) Straumr's own water volumes carry it, in each player's game. A pool that would reach past 512 m from its dam before spilling, a wall in a river's tidal reach that the highest tide and a storm's surge can top, and a dam the water simply goes round at the river's own level aren't treated as dams.
- **Breaches and surges.** Break a wall and the pool falls through the gap as over a weir, fast at first and slowing as the head falls. The released water runs down the river as a front at real speed (the river's own current plus a long wave's speed in its depth: about 6.6 m/s on a river 3 m deep running 1.2 m/s), so you can watch it come. The river stands higher behind it. A pool downstream takes the surge into its own pool. Overtopping never breaks a dam.
- **How wide and how fast.** A breach, a hole or a gap in a ring is measured along the wall, so a wall built at an angle counts its true length, and stone taken down beside a break widens it. The crest is measured the same way, so a diagonal dam doesn't spill too easily. A pour never runs faster than water can at its depth. Where a pond spills round its wall's end over a narrow col, it stands the weir's head over the col's real width, not a made-up one.
- **Gates.** A door standing in a dam's wall is a sluice gate (up to 8 a dam). Opened, it passes water over its sill, and under its lintel through an opening, both ways. A gate opens or shuts over 5 in-game minutes; the door itself swings at once. A bottom door is a low outlet, a top door a weir.
- **Holes.** A hole broken low in the wall with pieces still standing over it, or an open gate under its lintel, draws swimmers within one and a half times its height toward it, harder the more water it passes, and takes them through if it is a metre or more each way (out below the dam, on its jet); a smaller one holds them against it (`[Dams] DragUnder`).
- **Two dams in a row.** The water between them is a cofferdam while nothing flows into it. Once the upper dam's pond pours into it, it fills as the lower dam's pond. If the upper dam breaks into a dry ring, the ring fills over the timing curve below and then the two come level. Where the dams are so close that the upper breach opens straight into the lower pond's water, the two become one water held by the lower wall. Water is neither made nor lost at the hand-over (checked in the lab: no jump over half a metre in the routine scene). Where the upper dam's water goes round it and comes back into the river below the lower dam, the lower pond doesn't get the whole river. Break the upper wall and the water between is open to the river above again: the lower pond falls to it instead of standing as if the upper wall were whole.
- **Water it's open to.** A pond open to other water (the river above it round the wall's end, another channel of a braided river, or the river above through a broken upper wall) can't stand above that water by more than its flow needs to get through the opening; where the water can go round, the dam lets it go round at the river's own level. (Found in play on a sea-level delta, where ponds stood 1-3 m over the river beside them; lab replays of those spots now go round at the river's level.)
- **The look.** A dam's pond is drawn as a body of water of its own with the game's water look, its edge on the ground's own shoreline and the wall (`[Pools] OwnSurface`, each player's own). Where it pours over the wall or through a breach, the water hangs off the pond's edge down the wall (`[Dams] SpillSheets`). `[Dams] SpillRamp` draws the pool's level over the crest and down a 2-4 m ramp below it instead of a step in the wall (off by default).
- The server holds at most 16 dams.
- Dams are written to `BepInEx/config/Straumr-water/<world>.dams.txt` by the server with the world's save, and only then. A restart resumes a filling pool, a spill or a breach where the world was saved: after a clean stop that is where it was, and after a crash or a kill it is the last world save. A dam standing with no record there (a world from before, or `[Dams]` just turned on) is taken as long closed, its pool full.
- `straumr dam` lists the dams near you; `straumr dam show` marks where each one within 1 km would spill (a beam, a ring and a label), including the way round if that is lower than the wall.

**Cofferdams** (new in 0.1.3, experimental, on by default: `[Cofferdams] Enabled`)
- A patch of river, lake or sea walled off on every side by pieces standing at least 10 cm above the highest water outside (the river's level, plus the highest spring tide and a full storm's surge where the tide reaches), with no gap, is a cofferdam. Ground or a bank may close the ring too. The ring can be at most 128 m across.
- Its water falls at `[Cofferdams] DrainRate` (0.5 m a real minute) until the bed lies bare. Break the ring and the water fills back in through the gap as over a weir. An enclosure built before the setting is turned on drains too.
- `[Cofferdams] WaterMask` (each player's own) hides the game's water over a drained cofferdam, so from past 128 m you see the bed inside the walls instead of flat water.
- **Filling from a dam above: the timing curve.** A dry cofferdam below a dam that breaks fills from that dam's pond through the opening, by the head across it. That can take seconds or hours in the water's own time, and the dams' clock runs 48 times faster than the water's own time (with Valheim's 30-minute day), so a short fill would be over before you saw it. A fill plays at its own pace when it is short and is sped up only as far as it needs to be when it is long, never past the clock's own 48 times. A fill that would take 5 s of the water's own time plays in about 5 s; 30 s plays in about 24 s; 1 minute in about 36 s; 5 minutes in about 68 s; 20 minutes in about 96 s; 80 minutes in about 2 minutes. A ring refilled by its own river, with no dam above pouring into it, is paced the same way (a lab ring of about 1,100 m³ fed 27 m³/s now takes about 29 s instead of about a second). Only these fills are paced this way: ponds, spills and surges keep the clock's pace and the surge's own speed.
- Saved with the world, like dams.

**Kept pools** (new in 0.1.3, experimental, on by default: `[Pools] Enabled`)
- Water you cut off from the river with ground alone keeps its own level instead of following the river: dig a basin, let the water in through a channel (from a dam's pool too), fill the channel in again, and the basin keeps the water it had, through floods and after the dam is gone. It keeps its water, not its level: dig it deeper or wider and the same water stands lower, throw earth in and it rises and spills over its rim.
- Only water the server saw joined to the water outside and then cut off is kept: a pond it never saw joined stays as it is. A gap dug in the rim lets it fill or drain to the water outside; what it lets into a river runs down that river as a dam's release does; a door standing in its rim is a gate. Two kept pools joined by digging come level.
- The server keeps up to 64 pools, in `BepInEx/config/Straumr-water/<world>.txt`, written when the world saves.

**Floods** (new in 0.1.3, experimental, off by default: `[Floods] Enabled`)
- Rain raises each river's level by its own size's rain, as it already speeds its current. Creeks rise within an in-game hour of a storm's start; great rivers peak about a day after the rain began and fall back over days. A high river holds the rivers running into it up at their mouths. A stretch whose low banks hold the river down in ordinary weather spills over them in a flood (`[Floods] FloodRoom`, `FloodPonds`). With floods off, the levels follow only the great rivers' rain.
- The levels step every in-game hour (75 real seconds), each step fading in over the next.

**The sea**
- Wind-driven currents run along the coasts, through the sounds and between islands; shallow swamps and marshes stay mostly calm, apart from the wind's drift on their broader pools. Rips run out from open beaches, and longshore drift runs along them.
- **Wind** drifts the surface of the sea, and of lakes and broad slow reaches too: from about 8 m off the shore, fully from about 60 m out. Running channels don't take it. A storm pushes harder than a breeze.
- **Waves** carry floating things slowly the way they run: downwind on open water, turning in toward the beach over the shallows, so with the wind onshore driftwood, wreckage and felled logs wash up. In a full gale that's about 0.05 m/s far out, up to 0.3 m/s over about 10 m of water and 0.07 to 0.1 m/s in the last metre or two; a quarter of that at half wind. Small boats with someone aboard take a share of it, a longship none. In a strong wind the shallows turn to surf, which pushes swimmers the way the waves run. None in a river's channel.
- **Breakwaters** (new in 0.1.3): walls and other built pieces standing in the water, and ground raised above it, keep out the waves coming from upwind. The calm is most just behind them, less further back, none past 128 m. Valheim shapes its waves only at the corners of each 64 m zone, so the calm shows across a harbour-sized basin, not behind one short wall. Boats ride the same calmer waves that are drawn (`[Currents] Breakwaters`; `NaturalShelter` lets the land as the world made it calm the waves in its lee too, off by default).
- **Tides:** the sea rises and falls up to 0.6 m either side of its usual level twice a game day (0.8 m at spring tides). Bays behind narrow mouths rise less and later, and their mouths race in and out (about 1.5 m/s, 2 m/s at spring tides). Up river mouths the flood pushes against the river and the ebb speeds it out. Slack water comes at high and low tide, a little later each day, and the moon sets the size: spring tides at full and new moon, gentle neaps between. A storm raises the sea a little more on top (up to 0.3 m in a full storm), as far up rivers as the tide reaches.
- **The Deep North's ice** (new in 0.1.3): the game lays its sea ice at a fixed height, so the tide had put water on top of it. Over the ice the tide and the storm surge now fade to nothing, back to the full tide over about a kilometre, and tidal streams and whirlpools fade there the same way.
- What Valheim checks against the water follows the tide: swimming, boats, floating things, building damage under water, placing water pieces, fires going out, and creatures deciding where the water is.
- **Whirlpools** spin up off the ends of tidal straits near the peak of each tide. New in 0.1.3: at the rim of a turning whirlpool's core the pull draws you in gently (about 1.2 m/s, slower than you swim, so you can swim away) and harder toward the eye; within 0.4 of the core (4 to 12 m from the middle) it is faster than you can swim, so once that close you're carried into the eye. There it pulls you under for a second every 4 s and costs 3 stamina a second, until after 6 s it lets go and you can swim clear. In full while it turns at 1 m/s or more, near the peak of the tide. Boats and floating things aren't gripped (`[Currents] WhirlpoolGrip`). The sea also dips into a funnel at the eye: at the default, about 1.1 m at the eye of a pool turning at 1.5 m/s and 2 m deep at 2 m/s, which you swim, float and sail in (`[Currents] WhirlpoolFunnel`; 1 is the depth the spin's physics holds, 0.37 m and 0.66 m).
- A line under the minimap tells you the tide near any coast: *"Flood tide · full · high water in 6 min · +0.1 m"*.

**The water's surface** (new in 0.1.2, still being tuned)
- Ripples and sun glints on the water move with the current. Straumr lays a thin see-through sheet on the water round you, drawn with its own shader: it rises and falls with the game's waves and the tide, and the current carries its ripples along. It only adds glints and the sky's light on the ripples; it paints nothing white. Still water doesn't show it, a slow river (from 0.05 m/s) shows it faintly and water running 0.5 m/s or more shows it in full; on open water in a wind it fades to half, leaving most of the sea to the game's own waves; and it fades out by about 60 m from you.
- The shader is built into `Straumr.dll` for each platform: Windows (which also covers Proton), Linux and Mac.
- It switches itself off where it can't run (no shader for your platform, or a graphics card that can't run it), when NorgerWater is installed, while Seasonality has the water frozen, on Rivers and Lakes' water, and toward the Deep North, where the game calms its big waves.
- `straumr overlay` in the console turns it off and on in your game, to compare, and says whether it's on, and why if it couldn't start at all. Its settings are under `[Surface]`.

**Life on the water**
- **Foam** shows which way and how fast the water goes: a few scraps on all water (still water too), much more where it runs, shears and meets, in rafts, lace and strands drawn out by the current, and white water with streaks and spray. It fades where it meets a bank or a rock instead of ending on a hard line (`[Flecks] SoftEdges`), and `[Flecks] Lift` sets how high it floats. Rivers murmur, fuller the more water they carry, and rapids roar (sounds made from the physics of bubbles, with no recordings), and mist lies on the water at dawn.
- Driftwood washes up in bends and just above the spring high water, and fish hold in the pools.
- **Felled logs float**, about two thirds under, and drift with the current and the waves: rolled into a river, a log is carried off downstream, pivoting round an end that grounds and lining up with the stream.
- **Log drives** (new in 0.1.3). A wall or dam across a river now catches what floats down against it. Still water backed up against it drifts toward it (at 5 cm/s at least) so things reach it instead of stopping short (`[LogDrive] DamApproach`). Pieces with gaps at the waterline, judged from their own colliders (a lattice, a table's legs, an open frame), pass water and catch what is thicker than their gaps; they never close a cofferdam, make a dam or act as a gate (`[LogDrive] Screens`). On the server, logs and floating items in a river with nobody near are carried on down it along the rivers' lines until a barrier across the river catches them or the sea, a lake or another mod's water takes them; a player coming near takes them over where they are (`[LogDrive] ServerDrive`, `Items`). A thing lying caught within 40 m upstream of a barrier counts as inside a base, so Valheim doesn't remove a dropped item after an hour (`CaughtIsBase`).
- Boats feel the current while someone's aboard but always keep some headway, and a boat stopped against a bank or dock stays moored.
- A boat left to drift in a river (someone aboard, no one at the helm, sail furled and oars shipped, not moored) slowly swings round to lie along the current, bow or stern downstream, whichever is nearer: across a 1 m/s current it comes round in about half a minute to a minute, quicker in faster water. It fades out on open water.
- A boat left empty (not moored) rides as if at anchor: it stops where it was left, swings a couple of metres either way, and in a river turns to lie along the current; board it and it's free (`[Currents] AnchorEmptyBoats`).
- A line under the minimap while you're aboard a boat gives its speed through the water and over the ground, and what the current does along its heading: *"Boat 2.4 m/s through water"*, *"3.1 m/s over ground"*, *"current +0.7 (helping), running NE"*. The current there is the share of it that carries boats (`[Currents] Boats`). Valheim's own boats only, so far.
- Ruins and other stonework the world placed don't crumble where a steep world leaves their edge hanging (`[World] StandingRuins`).

## Works with

Straumr runs on its own, on Valheim's own worlds, and detects these automatically when they're installed. None is required.

- **[Riverheim](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/):** worlds built around big river networks, which Straumr sets flowing. **1.2.x and 1.3.x both work.** Riverheim 1.3 is a [Worldhalla](https://thunderstore.io/c/valheim/p/Gurebu/Worldhalla/) plugin and needs it; Straumr finds Riverheim's rivers through Worldhalla's public API by reflection, so Straumr itself still needs only BepInEx. Checked on a lab server with Riverheim 1.3.1 and Worldhalla 1.0.0: a new world and a converted 1.2.1 world both read their rivers, with each river's flow (325 rivers, all carrying flow, on the preset the test used). Checked in a player's game too, joined to a dedicated server on a new 1.3.1 world: the game read Riverheim's rivers, its flows matching the server's. If Riverheim's insides change again, Straumr says so once in its log and carries on with Valheim's own rivers rather than failing. Two warnings, both from how Riverheim and Worldhalla work, not from Straumr:
  - **Old 1.2 worlds convert one way.** A world made by Riverheim 1.2 loads in 1.3.x with the same terrain (checked: the generation code gives identical heights, biomes and rivers for 10 seed and preset pairs), but the first save writes Worldhalla's format, which Riverheim 1.2 can't read. A player's game asks first; a dedicated server just does it. Back up the `.fwl` and `.db` files before you move a world. Riverheim's own notes cover this too.
  - **Worlds made with Riverheim_Config's settings need Riverheim_Config 3.1.1 or later** on Riverheim 1.3.x, or their terrain changes. On the lab's test world, 1.3.1 without it read 325 rivers where 1.2.1 with Riverheim_Config's settings read 657.
- **[Rivers and Lakes](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/):** streams, lakes and waterfalls above sea level. Works side by side with Straumr: Straumr leaves its water to it. Its streams carry on as jets into still water, and its bigger lakes drift with the wind, once Rivers and Lakes shares its water with other mods. Where one of its streams comes down a fall, the jet starts at the foot of the fall, and the plunge drives it out faster (up to half again for a fall of 10 m or more). Streams that pour into a raised river meet it at the river's level. Where a stream runs into Straumr's water at the sea's level, Straumr's water is the water there.
- **[Seasonality](https://thunderstore.io/c/valheim/p/RustyMods/Seasonality/):** seasons, snowmelt and freezing. With it the rivers run faster and stand higher in spring, with flood plains, and run slower and stand lower in winter.
- **[ValheimRAFT](https://thunderstore.io/c/valheim/p/zolantris/ValheimRAFT/):** its vehicles ride the current too.
- **Expand World Size:** worlds of any size. Straumr reads its live world radius, the edge's width and the stretch by name when it's installed, never requiring it, and solves the sea's currents out to the world's own edge. Checked with Expand World Size 1.45 on a lab server on worlds with a 21 km radius (22 km edge), a 7 km radius (7.5 km edge) and the default; the sea solve takes about 10.7 to 13.6 s on a 22 km edge, in the background at load (*How it was checked*). Riverheim's map stays the size Riverheim made it, so a bigger radius adds sea, not land. "Stretch world" works too, and the atlas no longer undoes it in the world you play; the limits are under *Known issues*.

**NorgerWater:** when it's installed, Straumr's moving surface switches itself off and leaves the water's surface to it, the rivers stay at the sea's level (it draws the sea at one level everywhere), and dams, kept pools, own-surface ponds and breakwaters switch off. Straumr hasn't been tested with it in game yet.

## Similar mods

**[Undertow](https://thunderstore.io/c/valheim/p/RavenIronStudios/Undertow/)** (Raven Iron) gives Valheim's sea its own currents and tides. It's a sea mod; Straumr is mostly about rivers, with a sea of its own beside them. As their pages describe them:

- **Both:** wind-driven sea currents that run along the coasts, race through straits and go slack between them; tidal streams; a storm surge; boats, swimmers and floating things carried by the current; foam on the water that shows the flow.
- **Only Straumr:** rivers that flow from source to mouth on Riverheim and Valheim's own worlds, standing above the sea, with white water, wading, slots and floating logs; a tide that raises and lowers the sea itself (swimming, boats and building damage follow it); whirlpools; dams, kept pools and cofferdams (experimental, on by default); the tide and boat lines under the minimap; a public API for other mods.
- **Only Undertow:** flotsam that gathers in slack water, nothing drawn on the screen by design, and storms from Ragnarok's Wrath.

**[Fires Valheim Galaxies](https://thunderstore.io/c/valheim/p/incompletionists/FiresValheimGalaxies/)** (incompletionists) replaces the moon with a real 3D one whose phases drive the sea's tides, plus ocean and river currents and storm whirlpools. As their pages describe them:

- **Both:** tides that follow the moon, bigger at full and new moon, with the sea itself rising and falling; rivers taking part of the tide, their mouths running out on the ebb and in on the flood; rivers pushing downstream and slow ocean currents, carrying ships, swimmers and creatures; whirlpools (Galaxies': in deep water during storms, grinding hulls; Straumr's: off the ends of tidal straits at the tide's peak); the tide held still under ice.
- **Only Straumr:** each river routed from its source to the sea, with white water, wading, slots and floating logs; rivers standing above the sea; a tide that runs less and later behind straits; dams, kept pools, cofferdams and the surge when a dam breaks (experimental, on by default); Rivers and Lakes side by side; a public API for other mods.
- **Only Galaxies:** the 3D moon with phases and blood moons, bodies orbiting the world that you can fly out to and stand on, and diving below the surface.

**Running Straumr with either is not recommended.**
- **With Undertow:** both mods give the sea currents and push boats, swimmers and floating things along them, so at sea you'd be carried twice, by two sets of currents that don't know about each other.
- **With Fires Valheim Galaxies:** both mods raise and lower the sea with a tide, run the rivers' mouths and push boats and swimmers along currents, so the tide would be moved by both at once and anything afloat carried twice.

## For mod makers

Straumr has a public API for other mods (water wheels, mills, fishing, sailing AI, current charts): it reports the water's physical properties and leaves the gameplay to you. `StraumrApi.Sample(position)` gives the current, depth, tide and more in one call; since 0.1.3 also a river's rise above the sea, how full it runs, how much water it carries and whether the ground is flood plain or under flood water. `StraumrApi.Outlook(position)` gives a river's next four hours, `StraumrApi.LevelsChanged` says when the river surface round the player changes, and dams, cofferdams, gates, water power (`Work`, `BestDrop`, `HydraulicPower`) and drift paths have members of their own. Soft-depend on `wackah.straumr` and your mod works with or without Straumr installed. `ApiVersion` stays 1. The full guide, with examples and its limits, is **`API.md`** in the download.

## Multiplayer

**Tested on a dedicated server with lab clients, with the join, config-sync and restart tests; not yet tested with a group of players.** Nobody has yet had two or more real players in the same pond, joined mid-fill or watched the same surge from two places.

Install Straumr on the server and on every player's game. The server's physics settings are sent to everyone who joins, and players get their own back when they leave: all of `[Currents]`, `[Stage]`, `[Floods]`, `[Dams]` (except `SpillSheets`), `[Pools]` (except `OwnSurface`), `[Cofferdams]` (except `WaterMask`) and `[LogDrive]`, the white-water thresholds (`WhiteFroude`, `WhiteFroudeFull`, `WhiteMinSpeed`) and `[World] StandingRuins`. How the water looks and sounds stays each player's own choice. A shared setting the server doesn't send is held at its default while you're on that server.

Tides run off the server's clock, so everyone has the same tide. The rivers' levels are worked out the same way on every player's game, from the world, the shared settings and the server's clock (its season and its weather), so everyone sees the same water. Rain moves the levels only while a player's game draws the same weather as the server's: `straumr weather` says whether it does. Where it doesn't (a weather mod or setting that differs), rain still moves that player's currents, but not the water's height.

Dams, cofferdams and kept pools are decided by the server: it finds them, and sends each game a record with the settings; each game works the water out from that record and the server's clock, so a boat, a swimmer and a dropped item sit at the same water in every game. A game that joins late, or comes back after being far away, gets the same water. The server's own water stays at the sea's level, as with the tide: only players' games raise the rivers. On a dedicated server, dams and the server's log drive need `[Currents] RiverLevels` on.

Players who join with an older Straumr (0.1.2), or none, are told on screen which Straumr the server runs and that their water will differ, and a player whose Straumr is another version than the server's is told both versions. 0.1.5 needs no new setting and the world's files keep their format. A player whose server has no Straumr, or an older one, is told too. An unknown or damaged part of the settings package costs only itself: the rest is read.

Mixed versions, checked at the release gate: neither direction is refused. A 0.1.5 server with a 0.1.4 game, and a 0.1.5 game on a 0.1.4 server, each tell the player on screen that the versions differ ("This server runs Straumr 0.1.4, your game 0.1.5: the water may differ from everyone else's.", and the same the other way round). The settings sync and every setting are unchanged since 0.1.4, the dams' file keeps its format (only its comment header changed), and a world saved on 0.1.5 loads on 0.1.4 unchanged (43 comparisons, 0 different). A newer settings package's later blocks are skipped whole, so an older reader reads nothing wrong. **A real 0.1.4 game joining a 0.1.5 server was not tried live;** this is read from the code and the saved files.

## The flow map and seed atlas

- `straumr atlas` in the console (F5) opens a page from your own game at `http://localhost:8765`.
  - Type any seed: your game builds that world in memory (nothing is saved) and shows its rivers, sea currents, tides and whirlpools.
  - It starts whole-world, and fills in real detail as you zoom. Press **D** for a debug view of why the water does what it does at any spot, and **T** for the tide.
  - Layers (all in the panel): rivers, sea currents, tidal races, whirlpools, white water; and new in 0.1.3, **River levels** (off by default: where raised rivers stand over land, and the spring flood plain), **Fords** (off: where a river can be waded across), **Water power** (off: the strongest water per area, for water wheels) and **Built water** (dams and their pools, kept pools, cofferdams and the river below each dam).
  - **Drift** (key **R**): click the water and it draws the path something floating takes from there over 5, 15 or 60 minutes.
  - It shows the whole map, so don't type a seed you want to keep a surprise (or list it in `[Atlas] HiddenSeeds`).
  - On an Expand World Size world the page reads the world's edge and draws the stretched rivers; a seed you type is built with the played world's size (see *Known issues*).
  - Only that page can read it (a key it gets from the game), and it closes a few seconds after you close the page.
- `straumr map` writes the world you're in to a file the same page can open.
- On a dedicated server the atlas is started from the config file by its admin, and its `@here` seed is the server's own world.

## Console (F5)

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

\* needs `devcommands` first, or admin rights on a server.

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
| `WhirlpoolGrip` | 1 | New. How hard a turning whirlpool takes hold of swimmers (0 = the old gentle drift and the eye's tumble only; up to 2). |
| `WhirlpoolFunnel` | 3 | New. How deep a whirlpool's surface sinks toward its eye, times what its spin would hold (1 = physical; 0 = flat; up to 6; at most 4 m). |
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
| `RiverLevels` | true | New. Rivers stand above the sea, climbing gently toward their sources, higher in spring and lower in winter. Off: all water at the sea's level, as before 0.1.3. |
| `RiverLevelScale` | 1 | New. How high the rivers stand, times what their currents need (0.5 = half as high). |
| `RiverFlooding` | 0.5 | New. How far raised rivers may spread over low ground beside them: each 64 m square's water may grow by up to this share of what it already holds (0.15: banks get wet, flat meadows mostly stay dry). Where a river would spread further, it stands lower there. |
| `SpringFlooding` | 4 | New. In spring (with Seasonality), how many times `RiverFlooding`'s spread the rivers may take (1 = the same all year). |
| `Splits` | true | New. A loop's far side or a lake's second way out carries its share of the water (off: a drift at a third of the river's speed, as before 0.1.3). |
| `Chutes` | 1 | New. Where a river runs steeply down into a stretch its banks hold lower, how much faster it runs too (0 = only the look and the sound). |
| `LeeEddies` | 1 | New. Eddies curling back into the lee of sand bars, islands and points (0 = none, up to 2). |
| `LeeEddyCover` | 0.4 | New. How deep the water over a bar's top must be before the river runs straight over it and its eddy is gone (m). |
| `Breakwaters` | 1 | New. Calmer waves in the lee of what players build (0 = off, the game's own waves). Not under NorgerWater. |
| `NaturalShelter` | false | New. With `Breakwaters`: the land as the world made it calms the waves in its lee too. It changes the waves near land everywhere, so it's off by default. |

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

Upgrading to 0.1.5 from 0.1.3 or 0.1.4: no setting was added, renamed, removed or given a new default. Upgrading from 0.1.2: `[Currents] Undertow` is now `[Currents] Strid` and your value carries over once. No other setting was renamed, removed or given a new default. Upgrading from 0.1.0: if your foam settings are still 0.1.0's untouched defaults, they move to the new ones by themselves; anything you changed stays as you set it.

## Known issues

0.1.5 fixed what 0.1.4 listed about how built water is drawn: the overflow over a low point beside a dam, thick, stepped and uneven walls, the last stretch of a long wall, and the seam while a big pond is redrawn (see *New in 0.1.5*). Each glitch that is still open in the drawing has its own line below, and the physics limits follow. It's a beta: expect some of them.

**Tiers.** Each issue that is wrong carries a tier, and they are sorted by it:
- **S** = a crash, a broken save, or water plainly wrong in ordinary play. None is open.
- **A** = visibly wrong for a long time in ordinary play.
- **B** = needs a particular build or timing, settles by itself, or is small.
- **C** = centimetres or rare; most players never see it.

Things not built yet are listed apart, under *Not built yet*; they are limits, not faults. A line that needs another mod starts by naming it. Items about how built water is drawn change only what is drawn: the water's levels and flows don't depend on them.

**Tier A: visibly wrong for a long time in ordinary play**
- **[A]** **Springs and drains in the currents.** In some places a river's current can seem to pour out of, or into, a spot mid-river: Straumr's reckoning of which way the water runs disagrees with itself there. A check of 214 squares of a Riverheim world found 88 such spots (70 outward, 18 inward). Re-measured on this build, on the same Riverheim 1.2.1 world and the same 214 squares of 256 m as *How it was checked* below: 70 springs (water pouring out of one point with nothing feeding it) and 18 drains (water running in from every side with nowhere to go), the same numbers as before. Of the springs 50 are at the top of a river, where its water really enters, 19 are at divides (where the water parts both ways) and 1 is in wide water; of the drains 9 are at divides and 9 at river mouths. Leaving the tops out, divides are 28 of the 38 left (74%). On Riverheim 1.3.1 (the lab's Preview01 world) it measured 25 springs and 0 drains. The planned fix works a divide's head out from the water each river carries.
- **[A]** **With Seasonality: in its freeze, a pond is drawn at the frozen river's level,** because in a freeze the rivers settle to the sea's level and the pond's own raise goes with them. The server's pond has no inflow in a freeze, so it drains toward its crest over the winter and refills in the thaw (read from the code; not measured in play). A frozen pond isn't ice at its own level, a wall shows no falls in a freeze, and logs lying on the ice can still be pushed about. Planned as a step of its own. Tier A only if you run Seasonality.

**Tier B: needs a particular build or timing, settles by itself, or is small**
- **[B]** **Falls on a stepped wall can still tear at their joins.** A column on a step's top whose neighbour drops past the step's side leaves a slit between them; joins don't resume below a pillar narrower than about 1 m; runs that meet at one point never join. In one scene dump from the player's game there were 5 torn joins; the player's screenshot shows slits and a dry edge by a pillar and a block.
- **[B]** **Water at a wall's ends after a breach or a rebuild.** A pour can start 3.6 to 4.5 m from any crest run, near the breach (seen at the two ends of one dam in the player's dumps), and loose sheets and stubs can hang in a breach.
- **[B]** **Falls have no sides.** A fall over a wall is drawn as a thin curved sheet with nothing closing its ends, so from beside a wall's end or a step you see the sheet's edge, and past a wall's end it hangs as a free curve. Closing the sides is planned.
- **[B]** **A fall at a stepped wall's end can switch back and forth between two drawings** (a plain sheet and a full fall), over and over while you watch. Seen in the release build's play; being looked into.
- **[B]** **A big change to a big dam's wall can stutter for a moment** (with its pond in view) while the dam is worked out again. Seen in the release build's play; being looked into.
- **[B]** **A fall's top can stand 0.64 m in the air** at a wall's diagonal end (a capture from the player's game); it is in 0.1.4 too. In one other capture a pour starts at the edge of a missing wall section, and in a few old captures the lab's checks flag one vertex of a pour.
- **[B]** **A big pond isn't drawn out to the sight limit yet,** and flying fast over one, its pieces can arrive late. (The first to move up if players report it.)
- **[B]** **A huge dam's spill over a low saddle into a neighbouring channel** was drawn over that channel at its own height, up to about 3 m above the water there, instead of pouring down into it. Spills beside a dam are now drawn from the dam's record, but this case wasn't measured again.
- **[B]** **An opening under a lintel counts as part of the crest.** The server counts a hole in a wall with wall above it as part of an open crest run, with no limit from the wall over it, and the drawing gives it nothing: in the player's play, a 1.1 m hole under a wall piece with the pool standing over it. The total water is right, the place wrong, and the pond stands lower than it should. It was in 0.1.4 too; to be fixed together with holes through a wall's thickness.
- **[B]** **With Expand World Size: "Stretch world" and distances in metres.** A few distances Straumr works with are fixed in metres and don't grow with the stretch (for example how far a river's water reaches from its line and how far a river counts as owning the water beside it), so at stretch 2 they cover less of the stretched land. They were measured in the lab and left for a later round. Tested with Expand World Size 1.45 only, lab-tested on the player's exact setup (not played on one yet).
- **[B]** **With Expand World Size: a radius that doesn't match the stretch.** Riverheim's map keeps the size Riverheim made it, scaled by the stretch, so a world whose radius doesn't match is cut off from it: on a 7 km radius world (7.5 km edge) 45 rivers have no way out against 0 on a full-size world in the lab, because Riverheim's map is cut at the edge; on a bigger radius the extra is sea, not land. Not investigated beyond that.
- **[B]** **Ring take and give-back at the wrong level.** A ring that takes the pool between two walls, or gives it back, can do it at a level that differs from the pool's, by 0.13 to 2.32 m (measured in the lab). The water is drawn wrong for about 10 to 20 s, then settles to the right level and stays. To see it: two walls about 28 m apart across a river (the water between them is the ring), the upper wall spilling, then the lower wall broken: for a moment the water between the walls stands about 2 m too high. 0.1.4 does the same (2.08 m off there, 2.15 m on 0.1.5, in the lab). Found by the mutation fuzz; planned for a later version.
- **[B]** **Ring: stored water over a different footprint.** A ring that takes the pool can leave the stored water over a different footprint than a fresh look at the same walls finds, so the two disagree: up to 0.29 m in the lab (the scenario `fuzzmut-20261110-1-RS` still fails on it). Not seen in play. It was in 0.1.4 too. Found by the mutation fuzz; planned for a later version.

**Tier C: centimetres or rare; most players never see it**
- **[C]** **A dark strip on the runs across a wide top.** Columns leaving a 4 m wide top start 3 m further out than the ones off a 1 m crest beside them, so their rows never pair and a dark strip shows between them. One attempt to close it didn't. It was seen in the player's play of an earlier build; the last round could not reproduce it in the dump it checked, so whether it still shows is not known.
- **[C]** **The water shared between the falls off the side of a stepped wall** (a player's screenshot) hasn't been tuned.
- **[C]** **Two dams built very close together (about 10 m), with a tiny ring of water between them, can draw water past a wall's end:** in the lab a sliver of surface under half a square metre, about 2.4 m past the wall's line.
- **[C]** **Rings and cofferdams, in the lab:** the lab's drawing checks still flag a few places at rings and cofferdams (a pour or a surface a little off a wall), as they have on every build since 0.1.3, 0.1.4 included. Not seen in play.
- **[C]** **Occasional one-frame waterfall glitches right after a wall changes.** For a frame or two after you break or reshape a wall (the lab's window was about 2.5 s), a pour can hang about half a metre off its place before the pond's record catches up.
- **[C]** **With Expand World Size: the atlas.** Joining or leaving a world while an atlas seed world is being built isn't guarded: wait for the atlas to finish first. A seed world takes the played world's radius, edge and stretch, so it shows that world's size, not Valheim's.
- **[C]** **Restart of a held pair after a ring's breach.** After a restart (a clean stop or a crash) while a ring's breach holds two ponds as one, the pond above can come back about 3% low: 756 to 921 m³ of about 28,800, 2.5 to 3.1 cm lower (the lab's scenario `fuzzmut-20261111-3-ML`). 0.1.4 does the same (918 and 921 m³ low there). It happens only while the breach holds the pair as one. This replaces the smaller restart residual (+25 to +80 m³) listed before. Found by the mutation fuzz; planned for a later version.
- **[C]** **A ring taken away soon after its breach.** A ring taken away within about a minute of its breach: its pond drains round the gap in about 10 s, then its last ~8 cm goes at once (the lab's ledger shows about 15,000 m³ of book-keeping difference in that last step). In 0.1.4 the whole pond vanished at once (62,749 to 67,434 m³ in the gate's runs). Found by the mutation fuzz; planned for a later version.

**Not built yet** (limits, not faults; each says when it is planned)
- **A pond doesn't back the river up above it yet.** Real water open to the river above a dam would raise that river as the pond rises, and spread upstream. Straumr doesn't route the river on its levels yet, so where a dam's water is open to the river above it (round the wall's end, or through a broken wall upstream), the dam lets the water go round at the river's own level instead of raising a pond over it. A dam across one branch of a river that splits round an island still takes the whole river's flow over its wall, and the current just above a bypassed wall still runs into the wall instead of turning into the way round. Planned for a later version.
- **Dug cuts, low areas and dirt.** Players will try these, so plainly: a block made only of raised ground (dirt) isn't seen as a dam; it needs built pieces across the river. A dug low area next to a pond fills as part of that pond, at the pond's level, and empties when the dam goes: it doesn't keep water of its own. Damming a river doesn't send it down a dug cut unless the cut leads to an exit lower than the dam's crest: otherwise the river, the cut and the low area become one pond that spills over the dam. (Lab, on the user-divert scene: a 4 m cut from the river through a ridge into a ringed low area, then a dam below it: the pond filled to the crest in about 150 s, 10.57 ha, with the low area in it; removing the dam drained all of it in about 20 s. A low area with a lower exit back to the river, and a cut dug below the river's level all the way, were not tried.) Planned for a later version.
- **A dead-end hollow beside a pond isn't a basin of its own.** The spill's route is drawn over the land to where it joins the river or to the hollow's floor, and no further: a hollow with no lower exit doesn't fill from its bottom as a basin and keep its own level when the spill stops. A basin of its own for such hollows is a later step.
- **Ponds have no ripples or glints of their own.** A pond drawn as a body of water of its own doesn't get the moving ripples and glints that the river's surface has within about 60 m; the river water beside it does. (Fading them at the pond's edge would itself leave a seam.) Not built yet; a later version may add them.
- **Surges follow the river's own line down.** A surge doesn't spread into side channels, round the far side of islands or into bays beside the river, and where it runs it can show the edges of the 64 m tiles the water is drawn in. A big surge's front can look as if it stalls and then catches up: the water under it moves on at the surge's pace (in play, a 10 ha pond broken open: the drawn front up to about 1 m off the water's own, as it passed). A later version is planned to follow them properly.
- **With Expand World Size: tides on resized worlds are untuned.** Straumr scales the tide to the strongest races over the whole sea, so on a bigger radius, over the same land, the tidal streams run about twice as fast (measured in the lab); and with "Stretch world" at 2 the tidal races mostly vanish, because the narrows that make them are measured in a fixed number of cells, not in metres. At full flood the share of the whole sea running faster than 0.2 m/s is 4 to 5% on the 21 km worlds against 16% on a default world, because there is far more open ocean. A tuning question, left for later.
- **With shudnal's Seasons** (a different mod from Seasonality; read from its source, not tried in game): Straumr reads only Seasonality's season and freeze. With Seasons both run but don't cooperate: each keeps its own season clock, so they won't match; both set the water's height in winter and overwrite each other; Straumr's currents don't stop on Seasons' ice; Straumr's ponds don't freeze. Support is planned.
- **Sleeping and time skips move the water's clock too.** A night's sleep jumps the water's clock by the night's time, stepped exactly as if you had stayed awake (checked in the code and by a lab test: a 675 s jump is 2,700 steps, the same bits as stepping them one at a time). So after sleeping, a pond that was filling or draining has moved on by the night's time, the same as if you had watched it. That is correct, not a bug. A console time skip (devcommands' time skips and the like) does the same over however much time it skips.

**Not checked yet**
- two or more real players in the same pond at once (no group of players has played it);
- Straumr with NorgerWater in game;
- the log drive in play;
- roofs, fences and docks standing in a pond.

**Played by one player.** The drawing rounds were checked in the lab on scripted scenes, on replays of captures from the player's own game (a corpus of 132 cases at the first round) and offline, and then played on a dedicated server by one player, build after build: the way round and the big pond's seam, the play fixes, a pond full on joining, and a round on falls at joins and ends. What the player found each time is either fixed above or listed here.

**The release build itself was played** (md5 e2d1adbbd137) on the same server before posting: dams loaded as they were, walls reshaped and broken with the big pond in view. What that play found is under *Known issues* (falls with no sides, a fall that switches between two drawings, the short stutter on a big wall change). A killed-server restart was checked in the lab, not in play. No Expand World Size world was played: lab-tested only.

**Dams, cofferdams and kept pools are new, experimental and on by default; floods are new and off by default.** They have been checked in a lab and played on one test server by one player, not yet by two players at once on a real server. Expect rough edges at the edges of the rules below. Because they're on, walls you already built across a river become dams, and rings you closed start to drain, when 0.1.3 loads the world: on a world you can't afford to see change, set `[Dams] Enabled`, `[Cofferdams] Enabled` and `[Pools] Enabled` to false first (a server's config, on a server).

**Other limits**
- Valheim's own water surface doesn't move with the current. Straumr's moving surface lays ripples and glints on it within about 60 m of you, mostly on running water; beyond that, on slack water, and wherever the moving surface is off, the foam and the things floating on it show the flow.
- In a wide bay or sea arm with several river lines far apart, the current can swirl oddly between them, or drift across toward a river a couple of hundred metres off instead of along the one on the map.
- Where two ways to the sea through the same water are nearly equal, the water splits between them and lies slack where they part.
- A lake no river runs into lies still apart from the wind, and so can broad water far from any river.
- Ground raised with the hoe across a river isn't found as a dam: a dam needs built pieces across it (raised ground beside them can complete it). Built walls and stones that don't dam a river bend the water like rocks and leave calm water behind them.
- A dam is judged from what the server sees. A pool that would reach past 512 m from its dam, a wall in a river's tidal reach that the highest tide and a storm's surge can top and a dam the water goes round at the river's own level aren't dams. Overtopping never breaks a dam; a wall has to be broken. The server holds 16 dams, 64 kept pools and 64 cofferdams.
- An open door that isn't in a dam's own wall (in a side channel, a mill race, a way round) still counts as wall to the dam's pool, and so do a dam's doors past its first 8. Leave a gap rather than an open door where the water should go round.
- A cofferdam can be at most 128 m across. A kept pool is only made from a cut the server saw.
- In Seasonality's freeze the rivers, dam ponds and kept pools settle to the sea's level, so raised ponds show dry beds until the thaw. Freezing them where they stand is planned.
- River levels change at most once a weather spell (five minutes into it), and a new level fades in over half a minute; the seasons move them in small steps. Arriving in a world, the levels come some seconds after the currents.
- Rain raises the levels only as much as it swells the great rivers (all of it with `[Floods]` on), and only while your game draws the server's weather (`straumr weather` says); otherwise it moves only the currents in your game.
- The server's own water stays at the sea's level, as with the tide: only players' games raise the rivers.
- Where a square's banks hold the river lower, the river runs down into it at 3 cm a metre: in spring that can be a drop of over a metre in about 50 m.
- Valheim's own worlds barely show the levels: their banks are low and their rivers slow.
- A few things Valheim keeps at its fixed sea level don't move with the tide or the rivers' rise: mist, embers, the beach's shading, and where the world placed things.
- The tide raises the sea one 64 m square at a time, so neighbouring squares can differ by a few centimetres. The rivers' rise doesn't step: it slopes smoothly across the squares. Waves behind breakwaters are as coarse as Valheim's own, 64 m.
- Log drives are new and have been checked in the lab, not yet in play.
- Straumr knows the water within about 160 m of each player (further where they have been lately); a boat 1 km away from every player is out of its sight, and the server runs no water of its own except the tide's clock, the log drive and what it decides about dams, cofferdams and pools.
- Rivers and Lakes' own streams and lakes move as that mod draws them. Straumr adds its stream jets and the wind's drift on its lakes once Rivers and Lakes shares its water with other mods.
- ValheimRAFT's vehicles ride the current, but the boat speed line, the current turning a boat (`BoatTurn`, `BoatYaw`), the anchor for empty boats and the waves' drift on boats cover Valheim's own boats only so far.
- If a Riverheim update changes how its rivers are built, Straumr can't read their water and works it out from each river's length instead, and says so in `BepInEx/config/Straumr-stream.txt`.
- With Expand World Size, the sea solve at load takes about 10.7 to 13.6 s on a 22 km edge (1.5 s on a default world) and the atlas's whole-world build about 30 s instead of 9 s, with about 30 MB more memory while the field is made; the sea's currents at full flood are weaker on a big world (*How it was checked*).

## Roadmap

Plans, not promises; the next steps depend on what players find.

- **Next:** making built water and dug channels part of one water model, so ponds, dams and the water backed up behind them follow the ground players change; then the map showing the rivers, lakes and dams players make.
- **Later:** stones just under the surface showing a riffle, fish that hold in seams and behind rocks, ice that freezes where the water stands, culverts and other small water works, a flood that travels down a river after rain.
- **Further on:** water that remembers what happened: lakes filling and draining, a log jam backing a river up and then bursting; a real wave where a dam breaks.
- **Not planned:** new models or textures (water wheels, mills, nets). Those belong to other mods, built on the API.

## How it was checked

**What was checked for 0.1.5.** The drawing and the crash fix were gated the way the earlier builds were, and then more.


- **The checkpoint gate on the drawing rounds** (build "r7a", 50b3c7de14b8, compared with the 0.1.4 build): of 6,128 methods, 5,818 are identical and 310 differ: 217 are tests, 85 are drawing and 8 are the pond drawn at its level on joining. Nothing in saving, reading back, syncing or the water's hydrology differs. Speed, with the lab server alone on the machine, three back-to-back pairs: the look's mean cost about 4.8 ms on both builds, 95th percentile about 10 ms, worst 43 to 55 ms on both. A 0.1.4 world resumed on the new build: 22 restart comparisons and 24 fields per dam differ nowhere, and its first frame is bit-identical. Restart 5 of 5 with no difference, the race test consistent, the late-join test passed three times. The scripted drawing families showed only the kinds *Known issues* lists.

- **The mutation fuzz** (new, lab only). It builds, breaks, opens, gates, pools and restarts at random in the lab, compares each record with the water's own accounting (none made, none lost) and with a fresh look at the same pieces, and shrinks a failure to the fewest edits. It was proved to bite: three bugs put in on purpose were each found (a level carried over a restart, a breach recorded without its checkpoint, a drained orphan), the second shrunk from 7 edits to 1. Run on the build as it was, it found two real faults, which is how the crash fix below was found (the second, a ring's stale record, is F2 above).

- **Saved-world tests.** A world saved on 0.1.4 loaded on the new build, in the lab's upgrade family: no failures, and 43 restart comparisons on each side with no difference. Two items to explain from it: a way-round record now carries its flow, and one breached dam's way round was kind 0 on 0.1.4 and 3 on the new build (outlet flow up to 4 m³/s, the levels 0.07 m apart). Only 7 frames matched by the clock inside the Flume; the standalone comparison matched 85.

- **The restart (crash) fix.** The lab's one-dam scenario (a dam filling, raised after the world's save, the server stopped as a crash would a while after that save) read the pond back ahead on the old build (+1.17 m; 2 of 2 runs) and held on the fix (5 of 5). Restart, race, late-join and the saved-world test were green on the main build and on the 0.1.4 source with this one fix applied (the summed method compare of that source with the fix: 19 entries, all the fix). The case that first showed it, two dams in series, had read back about +0.26 m on the lower dam (48,465 to 57,271 m³) and 4,711 m³ more in the upper (6 of 6 runs).

- **World size gives default worlds the same water, byte for byte.** On the lab's default Riverheim 1.2.1 world: the atlas for the seed and for the live world each give 214 of 214 detail squares byte-identical to the build without it (the world file differs only in its "made" time and the new "edge", 10,500), the server's status lines match, and 49 methods differ, every one a routed use of the extent or new code and none a drawing method. Repeated after rebasing on the crash fix, with the same hashes. The lab's scenario and restart families on that build: 13 runs, no exceptions, restart 0 of 88 comparisons different, only the known kinds. A kit dam holds, spills and breaches on a default world, a 21 km radius world (edge 22 km) and a 7 km radius world (edge 7.5 km), with no exceptions; on the 21 km world the sea's currents move in 57% of the sea between 10.5 and 16 km and in 63% between 16 and 21 km.

- **What the bigger world costs** (lab server alone, fresh worlds, two back-to-back passes within 1% of each other): the sea solve takes 1.5 s on a default world, 10.7 s on a 22 km edge with the land stretched and depth 0.75, and 13.5 to 13.6 s on a 22 km edge with the radius alone, 7 to 9 times as long, in the background on half the cores; the river field is ready first, and only the sea's own currents and tides wait on it. The atlas's whole-world build goes from about 9 s to about 30 s and its file from 4.9 to 15.2 MB. The sea's arrays take about 27 MB instead of 6 MB, and the map built while the field is made about 38 MB instead of 9 MB, transient. At full flood the share of the sea running faster than 0.2 m/s falls from 16% to 4 to 5% on the 21 km worlds, because the tide is normalised to the strongest races and there is more open ocean: weaker flood currents on a big world, a tuning question rather than a fault. The stretch round measured the same thing again: on a bigger radius alone, over the same land, the tidal streams run about twice as fast (the tide is scaled to the strongest races over all the sea, and the open ocean adds slow cells), and with the land stretched to 2 the tidal races mostly vanish (the round's summary figures, not a table).

- **Stretch and the atlas** (lab, on the build with the stretch fix, "st-c3"). On the player's setup, the share of river-line points on water 34% to 99.4%, rivers with no way out 110 to 0, worst level step 49 m to 32 m; a kit dam holds. The atlas's seed worlds left the live world's ground 0% changed (100% before); the atlas page reads the world's edge. Default worlds gave the same hashes as before (214 of 214 squares, byte for byte). The lab's scenario and restart families: 13 runs, no exceptions, restart 0 of 88 comparisons different, only the known kinds. The 6 springs and 0 drains over matching land against 25 and 0 on a default world are an 8 m check.

- **The ring's breach** (lab, on the build with the ring fix). The one-edit scenario `fuzzmut-20261110-3-MF` failed on the build before in 2 of 2 runs and passes on the fix; the water's own accounting balanced through the re-judge (0 m³ made or lost). Taking a ring away during a hold dropped the whole pond at once on the build before; it now drains round the gap and the last ~8 cm goes at once (`fuzzmut-f2-remove-held` is still red on both builds: -67,434 and -62,749 m³ before, -14,999 and -15,070 m³ now). The four ring and series items under *Known issues* are what remains.

**The release build itself** (md5 `e2d1adbbd137a51e78f71e5d1743fcd2`; every run below was on it, except the comparison runs on 0.1.4, 0.1.3 and the build before the ring fix). Nothing in the mod was changed for the gate.
- **Every method compared with the drawing-rounds build** (r7a): 5,990 identical and 405 different, each classified: 149 are lab test code, 147 are new code that nothing in the game calls (added only; nothing outside it refers to it), 53 are the world-size change, 15 the stretch change, 15 the ring fix, 11 the crash fix, 8 the version number, and 7 the compiler's renumbering of one lambda. No drawing code differs, so the drawing checks of the drawing rounds stand.
- **A default world gives the same water:** 214 of 214 squares byte-identical for the seed and for the live world, the world file's hash the same as the world-size build's, the server's status lines identical, no exceptions.
- **The lab's families:** self-tests and the settings sync 21 passed, 0 failed; the scenario and restart families only the known kinds, the same counts as on the stretch and ring-fix builds; restart 5 of 5, 0 of 88 comparisons different; the race test consistent three times; the late-join test passed three times; no exceptions. The mutation fuzz was red, all from known or older causes (the four ring and series items, one of them new to the suite and also in 0.1.4: the held pair's restart); it was capped at 3 of 20 cases.
- **Saved worlds:** a world saved on 0.1.3 and one saved on 0.1.4, each loaded on this build: no failures, 43 restart comparisons per side, 0 different (a way-round record now carries its flow; flows differ by at most 0.45 m³/s and levels by at most 0.008 m). A world saved on 0.1.5 and resumed on 0.1.4: no failures, 43 comparisons, 0 different.
- **Closed-form tests:** `damtest`, `spilltest`, `selftest` and `pooltest` report nothing failed (268 passing lines, none failing); `drivetest` 14 passed; the multiplayer tests (n1 to n9) passed; no exceptions.
- **Speed** (lab server alone, three back-to-back pairs against 0.1.4): the look's mean 5.29, 4.72 and 4.75 ms on 0.1.4 against 5.32, 7.51 and 4.68 ms; 95th percentile at most 52.7, 42.6 and 43.2 ms against 61.0, 42.9 and 43.8 ms. The 7.51 ms pair held a 311.7 ms frame that was two status-line writes taking 303 ms under a disk stall; 0.1.4 had a 663 ms one of the same kind in another pair. **The stall from writing the dams' file between saves is gone:** 0.1.4 showed server updates of 199.8 and 380.8 ms, the release none in three runs.
- **The player's own scenes** (a copy of the player's world: a big dam, a thick wall, two dams on the delta): the same verdicts as the drawing-rounds build (spill at 36.42 and steady 37.48 on the big dam), the replays exit 0, no exceptions. The pools were worked out in 5.5 to 9.5 s against 3.8 to 4.0 s before, with other lab runs going in parallel (not a clean measurement).
- **Mixed versions:** read from the code and the files, and checked with saved worlds; **a live 0.1.4 client joining a 0.1.5 server was not tried.** See *Multiplayer*.
- **Not run on this build:** the player's play (below). Expand World Size was lab-tested on the stretch build with the reported player's exact setup (radius 21 km, edge 22 km, stretch 2, depth 0.75); the final build adds only the ring-breach fix, and it was not re-run.

**The currents.** 0.1.3 was measured against 0.1.2 on the same Riverheim world, in the same 214 squares of 256 m along its rivers, lakes and junctions (about 369,000 wet 4 m cells), with the same scripts, with dams, pools and floods off, on an earlier test build of this release. The currents stayed as they were, and the check was repeated, with the same result, at several later builds:
- springs (water pouring out of one point with nothing feeding it): 71 to 70
- drains (water running in from every side with nowhere to go): 18 to 18
- spots where the current meets itself head-on: 116 to 118
- spots where the current turns right round from one 4 m cell to the next: 887 to 897
- bank water with the current running mostly into the land: 611 to 613 cells (0.9%)

**The river levels,** on the Riverheim world:
- of the 16,261 64 m squares where a river would stand 5 cm or more in spring (3,368 of them a lake's), the banks let 15,507 stand 5 cm or more in an ordinary season and 10,412 stand 25 cm or more, the highest 1.34 m; 868 squares' banks hold their river lower
- no steps: over 13 x 13 squares round a river its banks hold down in two places, neighbouring squares' water met to 0.0 mm on every shared edge, in an ordinary season and in spring; between raised neighbours the surface fell at most 3 cm a metre in 99% of places (the steeper rest is dry ground beside the water)
- driftwood: 1,798 of its 1,879 spots sit above a river's spring high water, which reaches up to 2.99 m above the sea

**Built water and multiplayer, on 0.1.3's release build.** A lab server builds, breaks, raises, rebuilds and restarts dams, cofferdams and kept pools on a script, replays recorded scenes of real play through checks on the drawn water, and runs a torture round of fuzzed scenes. The scripted rounds have all been run more than once as the dam code changed. On the release build, what failed in the scripted rounds was what *Known issues* lists, each failure the same kind as the builds before it and reproduced when run alone; the server's own consistency checks on its dam records failed nowhere. Closed-form self-tests (`straumr damtest`, `pooltest`, `spilltest`, `drivetest`, `selftest`) passed with no failure on the release build. A server restart in the middle of a filling pond, a spill, a drained cofferdam, a ring of two dams and a breach left no differences in any of those five scenes, on the release build. Two callers building, breaking, repairing and removing one dam seconds apart left one record on each wall and none after everything was removed. The settings package was written and read back: all 84 shared settings held and were given back on leaving, a 0.1.2 reader read the part it knows, late joiners at three moments got the settings, weather, test rain, dams, pools and cofferdams, and a refused connection got nothing. The server's look at its dams, rings and pools took about 4 ms every 2 seconds in the lab with two dams standing.

**Riverheim 1.3.** Checked on a lab server with Riverheim 1.3.1 and Worldhalla 1.0.0: a new world and a converted 1.2.1 world both read all their rivers with their flow, and the self-test passed. A player's game joining a Riverheim 1.3.1 server through Worldhalla read the same rivers as the server (its log: "flow: Riverheim's own ... largest 621", the server's figure) with the server's Riverheim_Config settings applied. Riverheim's generation code gives bit-for-bit the same heights, biomes and rivers in 1.3.1 as in 1.2.1 on the 10 seed and preset pairs tested, so a plain Riverheim world keeps its terrain. A world made with Riverheim_Config's settings did not: see *Works with*.

**Not checked yet:** two or more real players at once on a real server (the same pond at the same moment, joining mid-fill, a surge seen from two places), Straumr with NorgerWater, the log drive in play, and Riverheim 1.3 in the seed atlas.

## Problems or ideas?

Come to the [Straumr Discord](https://discord.gg/bU9AD8hb8M). `straumr report` writes everything I need to reproduce what you saw; please send that file and say which other mods you run. For anything wrong with the water you can see, `straumr scenedump` (with `devcommands`) writes the world round you to a file I can look at offline.

## Thanks

Inspired by the river worlds of Riverheim and the streams and waterfalls of Rivers and Lakes. Go give them a look:

- **Riverheim:** [Thunderstore](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/) · [Discord](https://discord.gg/ES3J7mu6kW)
- **Rivers and Lakes:** [Thunderstore](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/) · [Discord](https://discord.gg/vuHnDRqfjn)

Built with help from Claude, Anthropic's AI.
