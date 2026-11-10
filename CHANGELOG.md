## 0.1.4

A hotfix for hitches when hosting big, heavily built worlds. Nothing about the water changes.

- **Fewer hitches on big bases.** When you host (or play single-player, or run a server) with a large base, the server's check for dams, cofferdams, kept pools and log barriers read every piece in a zone the first time it looked there, which could freeze the game for a fraction of a second when you walked or teleported into a big base. It now reads ahead in spare moments, at most 1 ms a frame, so the check finds the zone already read. On a lab base of about 3,500 pieces, the hitch on walking in fell from about 46 ms to about 10 ms, and the log drive's barrier checks from up to 38 ms to under 2 ms.
- The very first check after a world loads can still take a few tens of milliseconds while its code warms up.
- A player on 0.1.3 joining a 0.1.4 server is told the versions differ; the water is the same, but updating is recommended.

## 0.1.3

**Standalone: BepInEx only.** Straumr still needs only BepInEx. Riverheim, Rivers and Lakes and Worldhalla are read by reflection when they're there, and never required.

0.1.2 let you see the current and feel it; 0.1.3 gives the rivers height, lets players change the water, and works with Riverheim 1.3. Rivers stand above the sea, climb gently toward their sources, and rise and fall with the seasons and the rain. Dams, cofferdams and kept pools are new, experimental and **on by default** (turn each off in the config if you don't want it); floods are new and off by default. The currents were checked against 0.1.2 on the same Riverheim world (see *How it was checked* in the README).

**Multiplayer\*:** tested on a dedicated server with lab clients and the join, config-sync and restart tests; not yet tested with a group of players.

**Riverheim:** 1.2.x and 1.3.x both work (1.3.x is read through Worldhalla). 0.1.2 doesn't work with Riverheim 1.3. Worlds made with Riverheim_Config's settings need Riverheim_Config 3.1.1 or later on Riverheim 1.3.x, or their terrain changes. Old Riverheim 1.2 worlds convert one way the first time 1.3.x saves them, so back up the `.fwl` and `.db` first.

**Upgrading with dams on by default:** walls you already built across a river become dams, and rings you already closed start to drain, as soon as Straumr 0.1.3 loads a world. A dam standing with no record is taken as long closed, its pool full. To keep the world as it was, set `[Dams] Enabled`, `[Cofferdams] Enabled` and `[Pools] Enabled` to false (on a server, in the server's config). A dedicated server also needs `[Currents] RiverLevels` on, which is the default.

### Highlights

- **Rivers stand above the sea.** Each river's surface climbs toward its source (up to 1.34 m in an ordinary season on the test world), stands lower where the banks can't hold it, and slopes smoothly across the 64 m squares of water, with no steps.
- **Seasons, rain and snowmelt move the levels,** not only the speed: higher in spring (with Seasonality), spreading over flood plains, snow-fed rivers rising on spring afternoons, rain raising them a little. The rain is Valheim's lowland weather rebuilt from the server's clock, the same for every player.
- **Dams** (`[Dams] Enabled`, on by default): a wall across a river raises a pool that fills on the clock and spills over its lowest part; the river below carries only what comes over; doors in the wall are sluice gates. The pond is drawn as a body of water of its own.
- **Breaches and surges.** Break a dam's wall and the pool drains through the gap, its water running down the river as a front you can watch. A hole broken low in the wall pulls swimmers toward it.
- **Cofferdams** (`[Cofferdams] Enabled`, on by default): wall off a patch of water, and it drains to a bare bed; break the ring and it comes back, or, under a breaking dam, fills from that dam's pond at a pace you can watch.
- **Kept pools** (`[Pools] Enabled`, on by default): a basin you dig and cut off from the river keeps its water.
- **Floods** (`[Floods] Enabled`, off by default): rain raises each river's level by its own size's rain, with backwater where a high river meets a smaller one.
- **Built water is the server's and survives restarts:** the server decides each dam, cofferdam and kept pool, every game follows it, and the server keeps them with the world.
- **Riverheim 1.3 works,** through Worldhalla, with Straumr still needing only BepInEx.
- **Chutes, carried water and eddies:** white, faster water where a river runs down into a stretch its banks hold lower; rivers that know how much water they carry and share it at splits; eddies behind bars, islands and points.
- **Whirlpools that take hold, and a dry Deep North:** whirlpools draw swimmers to the eye and the sea dips into a funnel there; the tide fades out over the Deep North's ice so water no longer stands on it.
- **Breakwaters and log drives:** calmer waves behind what you build; logs and floating items carried down rivers by the server until a wall, dam or screen catches them.
- **Strid:** the river "Undertow" is renamed so it isn't mistaken for the sea mod; your setting carries over.

### Everything

**Rivers and levels**
- Rivers stand above the sea. Running water needs a slope to push it, so each river's surface climbs gently toward its source, worked out from the speed the river shows. On the Riverheim world it was checked on, the banks let rivers stand up to 1.34 m above the sea in an ordinary season.
- The banks decide how high: where a 64 m square's water can't rise to the river's full height without spilling over land away from the river or leaving ponds in hollows, it stands lower there, and the river runs down into it at up to 3 cm a metre.
- No steps between squares: the water slopes smoothly from one to the next. Swimming, boats and floating things meet the same surface you see.
- With Seasonality the rivers stand higher in spring and lower in winter. How high goes with the speed squared, so mid-spring's 1.4 times the speed stands about twice as high where the banks allow. In spring the rivers may spread over more of the low ground beside them, and the flood plains dry out again as the season turns. In spring, snow-fed rivers rise in the afternoons as the melt peaks. In Seasonality's freeze the rivers, ponds and kept pools settle to the sea's level.
- Rain follows the lowlands' weather (Meadows, Black Forest, Plains, Mistlands) as Valheim draws it for each weather spell, worked out again from the server's clock: the same rivers for every player wherever they stand, and for anyone joining late. Swamp rain doesn't count (it always rains there), nor does snow. A creek fills in about two minutes and drains in about eight; a great river's flood comes about ten minutes later, fills over some 25 minutes and drains over about 45. Rain raises the levels a little too, following the great rivers.
- Rivers know how much water they carry: from the land draining into each river on Riverheim worlds, from the length of channel above on Valheim's own. A river sounds as full as the water it carries, so a big river sounds big.
- A loop's far side or a lake's second way out runs at its share of the water, from how easily each way runs, instead of a flat 30% of the river's speed (`[Currents] Splits`). About 0.3% of the water runs differently. On Riverheim's rivers, where the water parts both ways (a divide), it drifts at 60% of the river's speed instead of 30%.
- Chutes: where a river runs down into a stretch its banks hold lower, it looks and sounds white and runs faster along the river, up to the typical river's speed at the steepest (`[Currents] Chutes`; 0 = only the look and the sound).
- Eddies behind the ground: behind a sand bar, an island or a point the water curls back into the lee while it stands out of the water, fading as the tide, the river and the waves cover it (`[Currents] LeeEddies`, `LeeEddyCover`).
- The current at a bend below a dam follows the channel instead of running into the bank.
- Driftwood washes up just above each river's spring high water: a strandline that shows how high the water can come. Branches left from before at spots that no longer exist are taken away where nobody sees.
- Rivers and Lakes: where one of its streams comes down a fall, the jet starts at the foot of the fall, and the plunge drives it out faster (up to half again for a fall of 10 m or more). Streams pouring into a raised river meet it at the river's level. Where a stream runs into Straumr's water at the sea's level, Straumr's water is the water there. A pond drawn over Rivers and Lakes' water covers it. As before, the jets need the Rivers and Lakes update that shares its water with other mods.

**Water bodies and dams** (experimental, on by default)
- Pieces you build across a river from bank to bank, 10 cm or more over its ordinary level, dam it: the water backs up behind them over the ground joined to it, at the river's own flow (an hour's filling takes an in-game hour), until it spills over the dam's lowest part or finds a way round it. Below, the river carries only what comes over, and runs lower while the pool fills.
- A pool stands up to `[Dams] Ceiling` over the sea (60 m by default); Valheim's own water ends 10 m over the sea, and above that Straumr's own water volumes carry it.
- A dam's pond is drawn as a body of water of its own: one wet footprint on the ground's own shoreline and the wall, one continuous surface, with the game's water look (`[Pools] OwnSurface`). It follows the pieces you built round it as well as the wall, so a hand-built wall no longer leaves a hole in the pond or an edge hanging in the air. Where it pours over the wall or through a breach, the water hangs off the pond's edge down the wall (`[Dams] SpillSheets`). A hole flush with the ground behind its wall is drawn as a chute rather than a hanging pour. Water over the crest falls as far as the water or ground below it, however tall the drop (a fixed length used to leave tall drops hanging short of what's below: 12 m of fall over dry ground, 30 m over water), and is drawn again once the ground below it loads, so it no longer falls through a river whose ground hadn't loaded. The back of a long wall seen past the end of its face no longer cuts the pond short or draws water past the wall. `[Dams] SpillRamp` draws the pool's level down a 2-4 m ramp below the crest instead of a step.
- Doors in a dam's wall are sluice gates (up to 8 a dam): a weir over the sill and an orifice under the lintel, both ways, opening and shutting over 5 in-game minutes. A mod can set one with `SetGateOpening`.
- Two walls found together become two dams, and two dams close together no longer confuse which one is upstream. Two dams in a row: the water between them is a cofferdam while nothing flows into it, and fills as the lower dam's pond once the upper pours into it. Where an upper dam's breach opens straight into the lower pond's water, the two are one water held by the lower wall.
- A pond open to other water (the river above it round the wall's end, another channel of a braided river, or the river above through a broken upper wall) can't stand above that water by more than its flow needs to get through the opening; where the water can go round, the dam lets it go round at the river's own level. (Found in play on a sea-level delta, where ponds stood 1-3 m over the river beside them; lab replays of those spots now go round at the river's level.)
- Two dams in a row where the upper dam's water goes round it and comes back into the river below the lower dam: the lower pond no longer takes the whole river (in play it filled the short reach between to its crest in 2 s and stood 1.2 m over it).
- Breaking the upper of two walls opens the water between them to the river above: the lower pond falls to that water instead of standing as if the upper wall were whole.
- A wall built across a broken dam's water closes it again.
- A new dam's pond is worked out on the server's background threads (a quarter of its cores at most), so a big pond's first flood, about a second for a 1 km stretch, doesn't stall the game.
- A wall in a river's tidal reach is a dam when its crest stands above the highest tide and a full storm's surge there; otherwise the tide tops it and it isn't judged.
- A pond handed to the pool below makes and loses no water, and no longer jumps (lab routine scene: the lower pond had jumped 3.7 m; it now moves no more than half a metre at a time).
- `straumr dam` lists the dams near you; `straumr dam show` marks where each one within 1 km would spill, with a beam, a ring, a label and a map pin, including the way round.

**Breaches and drawdown**
- Break a wall and the pool drains through the gap as over a weir, fast at first and slowing as the head falls, its water running down the river as a surge: a front at real speed, the river standing higher behind it. A pool below takes the surge into its own pool. Overtopping never breaks a dam.
- A pour never runs faster than water can at its depth (critical flow), so a breach doesn't drain a pond faster than it could.
- Widths are measured along the wall: a breach, a hole or a gap in a ring on a diagonal wall counts its true length (a 6 m cut on a 45-degree wall now passes 6.36 m, not 4.50 m), and stone taken down beside a break widens it (one lab case: a 9 m breach became 26.2 m for 28.5 m of lowered stone).
- Crest runs are measured along the wall too: diagonal dams used to spill about 1.3 to 1.4 times too easily.
- A pond that spills round the end of its wall stands the weir's head over the col's real width, not the width of all the water behind it, so a pond built higher collects higher.
- A hole in the wall with pieces still standing over it, or an open gate under its lintel, pulls swimmers within one and a half times its height toward it, and through if it is a metre or more each way (`[Dams] DragUnder`).
- The lower dam's water now reaches the line of an upper wall that's gone, so no wall of water is left standing after you delete a wall.
- When the corner of a wall is broken while a ring below it fills, the pond no longer holds back most of its outflow, so it falls at its openings' rate; a reshaped wall carries its water, not its old level, so no water is made in the new hollow. A rebuilt corner's pond is drawn. (Lab corner scene: pieces taken off, rebuilt, volume kept.)

**Cofferdams and kept pools** (experimental, on by default)
- A patch of river, lake or sea walled off on every side by pieces standing 10 cm above the highest water outside (with the highest spring tide and a full storm's surge where the tide reaches) drains at `[Cofferdams] DrainRate`, 0.5 m a real minute, laying the bed bare; break the ring and it comes back through the gap. Ground or a bank may close the ring too. Rings at most 128 m across.
- `[Cofferdams] WaterMask` hides the game's water over a drained cofferdam, so from past 128 m the pit shows its bed instead of flat water.
- A ring breaches only the dam whose wall broke, and a cofferdam takes the pool between two dams by its cells. A cofferdam breached from several sides fills from all round, with no jet from its middle.
- Kept pools: water you cut off from the river with ground alone keeps its own water instead of following the river. It keeps its water, not its level: dig it deeper or wider and it stands lower, throw earth in and it rises. A gap dug in its rim lets it fill or drain to the water outside; a door in its rim is a gate; two kept pools joined by digging come level.

**The timing curve**
- A dry cofferdam below a dam that breaks fills from that dam's pond through the opening, by the head across it. Dams run on Valheim's day (48 times faster than the water's own time with a 30-minute day), which would make a short fill over before you saw it. Now a short fill plays near its own length and a long one is sped up only as much as it needs, never past the clock's own 48 times: 5 s of the water's own time plays in about 5 s, 30 s in about 24 s, 1 minute in about 36 s, 5 minutes in about 68 s, 20 minutes in about 96 s, 80 minutes in about 2 minutes. This pacing applies to that fill, and to a ring refilled by its own river when no dam above pours into it (that used to come back at the clock's 48 times: a lab ring of about 1,100 m³ fed 27 m³/s refilled in about a second, and now takes about 29 s); ponds, spills and surges keep the clock's pace and the surge's own speed.
- When the ring is full, the pace eases up to the clock's over 10 seconds and the two dams come level.

**Surges and floods**
- Floods (`[Floods] Enabled`, off by default): rain raises each river's level by its own size's rain: creeks within an in-game hour of a storm, great rivers about a day later. A high river holds the rivers running into it up (backwater). Stretches whose low banks hold the river down spill over them in a flood (`FloodRoom`, `FloodPonds`). The levels step every in-game hour (75 real seconds). Without floods, the levels follow only the great rivers' rain.
- A server-wide test rain: an admin's `straumr rain` on a server sends every game a rain, so every game's rivers rise, it ends at the same moment everywhere, and late joiners get it.

**Restarts kept**
- The server keeps dams and cofferdams in `BepInEx/config/Straumr-water/<world>.dams.txt` (kept pools in `<world>.txt`), written when the world saves, and a dam's record also between saves, at most every half minute while it changes. After a restart a pond resumes filling, spilling or draining where it was, and a ring resumes draining, or filling from a breach, where it was. The clock doesn't advance while the server is off, so nothing finishes during downtime.
- A dam standing with no record there (a world from before, or `[Dams]` just turned on) is taken as long closed, its pool full.
- Restart tests on the lab server: a restart in the middle of a filling pond, a spill, a drained cofferdam, a ring of two dams and a breach left no differences in any of the five scenes, and was run again at each later build.
- A file from an older build is still read, and an older build reads what it knows and passes over the new parts.

**Network**
- The server's settings now include `[Floods]`, `[Dams]`, `[Pools]`, `[Cofferdams]` and `[LogDrive]` (see *Changed settings*), and the records of its dams, cofferdams, kept pools and log-drive barriers.
- The settings package is built in tagged, length-prefixed blocks: a game reads the blocks it knows and skips the rest, so a damaged or newer block costs only itself. Bodies over 8 KB are compressed. Packages go only to players through the game's own handshake, and wait while a player's send queue is full; one over 400 KB goes without the water block rather than stall the connection. Nothing is sent to a connection the server has turned away.
- A 0.1.3 game answers the first package with a hello. The server tells a player whose game has an older Straumr, or none, on screen that their water will differ. A player is told when the server has no Straumr, or an older one.
- A shared setting the server doesn't send is held at its default, not the player's own. A 0.1.2 server's `Undertow` is taken as `Strid`.
- Late joiners and players coming back after being far away get the same pond, surge and levels.

**Riverheim 1.3 and Worldhalla**
- Riverheim 1.2.x and 1.3.x both work. Riverheim 1.3 is a Worldhalla plugin; Straumr reaches its world through Worldhalla's public API by reflection, with no reference to Worldhalla. Straumr still needs only BepInEx.
- Every call into Riverheim now fails soft: the first failure is said once in the log, and from then on Straumr uses Valheim's own rivers instead of throwing. (0.1.2 read Riverheim 1.2's internals, which 1.3 removed, so its rivers break on Riverheim 1.3.)
- Checked on a lab server with Riverheim 1.3.1 and Worldhalla 1.0.0: a new world and a converted 1.2.1 world both read all their rivers (325) with their flow. `straumr selftest` has a new line that says how Riverheim was read.
- The seed atlas binds a Riverheim seed through Worldhalla on 1.3. (Not yet checked live.)
- Old 1.2 worlds keep their terrain, but the upgrade is one way: the first save in 1.3.x writes Worldhalla's format, which Riverheim 1.2 can't read. A player's game asks first, and a dedicated server just does it. Back up the `.fwl` and `.db` first. Riverheim's own notes cover it too.
- Worlds made with Riverheim_Config's settings need Riverheim_Config 3.1.1 or later on Riverheim 1.3.x, or their terrain changes (in the lab test, 325 rivers instead of 657).

**Whirlpools and tide**
- A turning whirlpool draws swimmers in gently at the rim of its core and faster than you can swim within 0.4 of it, carrying you into the eye, where it pulls you under for a second every 4 s and costs stamina until after 6 s it lets go (`[Currents] WhirlpoolGrip`). Boats and floating things aren't gripped.
- The sea dips into a funnel at the eye, deeper as the spin is faster; you swim, float and sail in the dip (`[Currents] WhirlpoolFunnel`).
- Over the Deep North's ice the tide and the storm surge fade to nothing (back to the full tide over about a kilometre), and tidal streams and whirlpools fade there too, so water no longer stands on the game's ice.
- Breakwaters: walls and other built pieces standing in the water, and ground raised above it, calm the waves in their lee (`[Currents] Breakwaters`, and `NaturalShelter` for the land as the world made it). Boats ride the same calmer waves that are drawn.

**Building and the kit**
- Log drives: a wall or dam across a river catches what floats down against it; still water backed up against it drifts toward it. Pieces with gaps at the waterline (a lattice, a table's legs, an open frame) pass water and catch what is thicker than their gaps, and never close a cofferdam or make a dam (`[LogDrive] Screens`). On the server, logs and floating items in a river with nobody near are carried on down it until a barrier catches them (`ServerDrive`, `Items`); caught things count as inside a base (`CaughtIsBase`).
- Test scenes for dams, breaches, holes, gates, kept pools and logs, built and cleared with `straumr kit` (needs `devcommands`); walls follow the hammer's 22.5-degree steps.

**Performance**
- The rivers' levels are planned on a worker thread.
- A dam's record is stamped about half a second after the last change to a piece in it, instead of every two to four seconds.
- Straumr's own per-frame work was trimmed: the foam's current grid and drops are batched, the moving surface re-bends only the zones that changed, the sea's sheets skip bends that change nothing, and the Rivers and Lakes seam is planned one at a time and kept when nothing changed.
- The server's look at its dams, rings and pools took about 4 ms every 2 seconds in the lab with two dams standing.

**Diagnostics, console and the lab**
- New: `straumr weather`, `flood`, `dam`, `dam show`/`off`, `gate`, `pools`, `cofferdam`, `work`, `drift`, `drive`, `seam`, `rain`, `ownwater`, `mask`, `kit`, `scenedump`, and the closed-form tests `damtest`, `pooltest`, `spilltest`, `drivetest`, `seamtest`.
- `straumr here` adds the river's level and state (how full it runs and why, what it carries, its next four hours, who has the upper hand at a mouth, and which river the water finally joins) and, near a Rivers and Lakes stream mouth, how far the stream falls. `waterlevel` shows the tide and the river's rise separately and whether they agree. `selftest` checks the river outlook and the levels' own consistency; it gains live checks of the sea's sheets, the foam's grid, the current and the ground, and of how Riverheim was read. `perf` names the longest game frame a minute and the slow parts, and `report` adds the settings package's size.
- Atlas: layers for River levels, Fords, Water power and Built water, and a Drift tool; point at a spot for its levels.
- `straumr scenedump` writes the world round you to a file for a bug report. The rest of the author's lab (scripted and fuzzed scenes, replays of recorded play, restart and join tests) is how each build was checked; players don't need it.

**API** (see `API.md`; `ApiVersion` stays 1)
- `FlowSample` gains `Discharge`, `RiverRise`, `RiverRiseUsual`, `RiverRiseHigh`, `FloodPlain`, `RiverStage`, `SurfaceFall`, `Flooded` and `FloodDepth`. `SurfaceHeight`, `WaterLevel` and `Depth` include a river's rise, and dams' pools and surges.
- New members: `Outlook` (and `RiverOutlook`), `LevelsChanged`, `DriftPath`, `WaveShelter`, `WaveFactor`, `TestRain`, `StartTestRain`, `StopTestRain`, `TestRainChanged`, `Cofferdam`, `CofferdamChanged`, `Dam`, `Dams`, `DamChanged`, `Head`, `HydraulicPower`, `Work`, `BestDrop`, `SetGateOpening`.
- Where the Deep North's ice lies, `TideHeight`, `TidalStream` and `FloodStream` fade to zero.

**Fixes**
- Water standing on the Deep North's ice (the tide put it there).
- The river "Undertow" no longer shares a name with the sea mod's (renamed Strid).
- A dam's pond, cofferdam and kept pool no longer show seams, stacked surfaces or edges hanging in the air in the cases found in testing: dams in series (including past an upper wall's stub end), a breach right after a wall is built, hand-built walls, and a corner broken and rebuilt.
- The sheet under a pond beside its wall no longer zigzags through the pond.
- Rain used to follow the weather in your own sky; it now follows the lowlands' weather rebuilt from the server's clock, the same for every player.
- Riverheim 1.3: Straumr no longer throws where Riverheim 1.2's internal names are gone.

### Known issues in 0.1.3

- **The overflow over a low point beside a dam isn't drawn yet:** where a pond spills round its wall's end, the water spills but the surface ends in a short edge hanging in the air.
- **Two dams built very close together (about 10 m) with a tiny ring between them** can draw water past a wall's end.
- **Occasional one-frame waterfall glitches right after a wall changes,** and a big pond's surface can show a seam for a few seconds while it is redrawn; both fix themselves. A surge follows the river's own line down, not into side channels, round islands or into bays beside it, and can show the edges of the 64 m tiles the water is drawn in; its front can look as if it stalls in places and then catches up.
- **Very big dams can still look odd in places:** a huge dam's pond that spills over a low saddle into a neighbouring channel is drawn over that channel at its own height instead of pouring into it; thick, stepped or uneven walls may not draw their overflow correctly across the top; a wall several pieces thick (four stone walls side by side, in play) draws its pour hanging over the top and can draw the surface short of its back; the last few metres of a long wall can show no waterfall. Breaking the dam puts it back to normal. Skipping time with a console command jumps the water's clock too, so ponds can change in one step.
- **A pond doesn't back the river up above it yet.** Where a dam's water is open to the river above it, the river above doesn't rise and spread with the pond: the dam lets the water go round at the river's level instead. A dam across one branch of a river that splits round an island still takes the whole river's flow, and the current just above a bypassed wall still runs into the wall instead of turning into the way round.
- Dam behaviour, overflow and rendering will be a main focus of the next patch.
- The latest dam fixes were checked in the lab and played by one player on a dedicated server. Roofs, fences and docks in a pond haven't been checked in play.
- Multiplayer has been tested on a dedicated server with lab clients, not with a group of players.

The README has the full list.

### Changed settings

- **Renamed:** `[Currents] Undertow` is now `[Currents] Strid`. Your setting carries over once, and is shared like the rest of `[Currents]`. A 0.1.2 server's `Undertow` is read as `Strid` by a 0.1.3 game. Old configs keep working.
- **New, shared from the server** (default): `[Currents] RiverLevels` (true), `RiverLevelScale` (1), `RiverFlooding` (0.5), `SpringFlooding` (4), `Splits` (true), `Chutes` (1), `LeeEddies` (1), `LeeEddyCover` (0.4), `Breakwaters` (1), `NaturalShelter` (false), `WhirlpoolGrip` (1), `WhirlpoolFunnel` (3); `[Dams]` `Enabled` (true), `Ceiling` (60), `DragUnder` (true), `SpillRamp` (false: the server's value is now everyone's); `[Cofferdams]` `Enabled` (true), `DrainRate` (0.5); `[Pools]` `Enabled` (true); `[Floods]` `Enabled` (false), `FloodScale` (1), `FloodRoom` (8), `FloodPonds` (256); `[LogDrive]` `Screens` (true), `ScreenPieces` and `WallPieces` (empty), `DamApproach` (true), `ServerDrive` (true), `Items` (true), `CaughtIsBase` (true), `ThroughUnexplored` (false).
- **New, each player's own:** `[Dams] SpillSheets` (true), `[Pools] OwnSurface` (true), `[Cofferdams] WaterMask` (true).
- **Removed:** none. **Defaults changed:** none; every 0.1.2 setting keeps its default. (`[Dams] Enabled`, `[Pools] Enabled` and `[Cofferdams] Enabled` are new in 0.1.3 and default to true; `[Floods] Enabled` defaults to false.) Riverheim 1.3 support, the timing curve and the dam fixes added no settings.

## 0.1.2

0.1.1 made the current trustworthy; 0.1.2 lets you see it and feel it. Checked against 0.1.1 on a Riverheim world and on Valheim's own world (see *How it was checked* in the README).

**The water's surface**
- Ripples and sun glints move with the current: a thin sheet with its own shader that rides the game's waves and the tide (`[Surface]`). It shows from 0.05 m/s, in full from 0.5, and fades to half on windy open water. It steps aside under NorgerWater, on frozen water, on Rivers and Lakes' water and toward the Deep North. `straumr overlay` turns it off and on.
- Foam fades where it meets a bank or a rock instead of ending on a hard line (`[Flecks] SoftEdges`). Where the ripples show, slow water carries less foam; still water keeps all of its.
- River sound starts at slower water and grows fuller the deeper the river.

**Rivers and lakes**
- Far fewer springs and drains. On the Riverheim world: springs 182 to 71 (none left at lakes), spots where the current meets itself head-on 187 to 116, reversals 1,229 to 887.
- Lakes drain through their real way out, and a lake's outlets no longer pour water back into it.
- The long way round a loop, or a lake's second way out, drifts along instead of standing dead.
- Rocks bend the water by the part of them that's in the water, measured at the waterline, not by their whole size. A rock whose shape the game carries twice is counted once.
- Water nothing feeds lies still where it used to move only because of a flaw in the old calculation (river water under 0.05 m/s: 11% to 15%).

**Weather**
- Wind drifts lakes and broad slow reaches too, not only the sea, and a storm pushes harder than a breeze (`[Currents] StormDrift`).
- Storm surge: a storm raises the sea up to 0.3 m on top of the tide (`[Currents] StormSurge`).
- Fixed: the current dropped out on every wave crest more than 30 cm above the still sea.

**In the water**
- Wading: the current pushes you downstream as you wade (`[Currents] Wading`).
- Undertow: fast water through a deep, narrow slot now and then pulls swimmers under for a moment (`[Currents] Undertow`).
- Swimming effort with or against the current, optional and off by default (`SwimWithCurrent`, `SwimAgainstCurrent`).
- Waves carry floating things the way they run, in toward the beach over the shallows, and in a gale the surf pushes swimmers (`[Currents] WaveDrift`).
- Felled logs float and drift with the current and the waves (`[Currents] LogDrift`).

**Boats**
- A boat drifting free in a river swings round to lie along the current (`[Currents] BoatYaw`), and the current's turn on a boat is halved while someone steers.
- A boat left empty rides as if at anchor (`[Currents] AnchorEmptyBoats`). Jumping off into a river no longer lets the current carry the boat away from you.
- Boat speed under the minimap while you're aboard: through the water, over the ground, and what the current adds or takes (`[Hud] BoatSpeed`).

**For mod makers**
- `WaterLevel` and `Sample`'s `SurfaceHeight` give the water as it stands, tide and storm surge easing in; `TideHeight` is where it's heading.
- `Sample` and `Velocity` agree on another mod's water, and another mod's lakes never run white.

**Diagnostics**
- `straumr here` names the rocks bending the current where you stand, and says when a stretch of river carries no source's water (a divide, or draining one way).
- `straumr selftest` checks the current's consistency near you (PASS, or a NOTE where rivers meet across wide water).

## 0.1.1

Most of this release makes the water more correct, then measures it: a whole Riverheim world was checked before and after (see the README).

**Rivers**
- Where several rivers share the same water (Riverheim often runs a few through one channel, sometimes in opposite directions), that water gets one way down: its shortest way to the sea. The rivers in it agree instead of pouring sideways into each other, and the drains and springs along their lines are gone. A river keeps the way Riverheim drew it unless the other way to the sea is less than half as long.
- Where several of Riverheim's river lines run through the same water, the one nearest the sea owns it, so currents no longer come out in stripes running opposite ways.
- River ends that reach the sea only through a narrow or shallow passage count as reaching it (they used to count as closed lakes, and drew water toward them).
- A river ending in a lagoon or bay runs out through it, instead of the lagoon draining back into the river.
- Lakes no longer have pits where a river's lake and channel stretches alternated.
- A river's distance to the sea is worked out properly through lakes and confluences, so lakes no longer run backwards from their outlet.
- Creeks no longer outrun great rivers: each river's speed comes from the water it usually holds, not from Riverheim's designed channel.
- Water slides along banks and round islands instead of running into them, and narrows speed the water up instead of stalling it.
- The current carries on past a river's mouth instead of stopping (or turning back) at it.
- Riffles only in shallow rivers; seams follow the river round bends; a river's lines no longer hold the water still across a lake.
- A wall or dam across a channel stops the flow; the water above it lies still.
- Rivers and Lakes' water is left to it: Straumr's river currents no longer push in its lakes and streams.

**The sea and tides**
- Narrow mouths on big bays and inner seas race as they should (before, any sea wider than ~700 m counted as open ocean).
- Currents flow between islands.
- The tide's stream, the tide line and whirlpools follow the local tide's timing (later behind straits).
- Water levels ease to a new tide instead of jumping (on login, after a settings change, at a freeze).
- Longshore drift no longer adds up several times on ragged coasts; windrows and boils hold steady over long sessions.
- The tide line shows near any coast, not only by tidal races.
- Creatures (their water checks, path-finding, spawn heights) follow the tide: `[Currents] CreaturesFollowTide`.
- Shallow swamps and marshes no longer count as open sea (the sea's currents there were far too strong).

**Things in the water**
- Rowing against a current is no longer faster than in still water.
- Rivers and Lakes stream jets stop at the far bank and get carried off by a river they pour into, sized by the stream.
- Swallow holes only pull the water in their own pool, never through a bank, and never faster than you can swim.
- Rocks, walls and docks: walls block the water, the wake fits the object, and debris gathers in a small back-eddy behind them.
- Only what breaks the surface bends the current: a stone just under it no longer stops the water dead. A creature standing in the water is never taken for a rock.
- White water pulls swimmers under for a moment, then lets them be for a while.
- Moored boats: shallow water no longer counts as a bank.
- Driftwood and fish stay out of rocks and banks, and stay with the world they belong to.

**Foam, sound and mist**
- Foam redesigned: a few scraps on all water (still water too), much more where the water runs, shears and meets; nine kinds of foam (packed bubbles, lace, strands drawn out by the current); lit by the scene so it doesn't glow. New `[Flecks]` settings: `CalmSpeed`, `Opacity`, `Brightness`, `Lifetime`, `Rafts`, `Spray`.
- Water sounds are placed at the water (not at your height), stop in menus, and have a smoother rumble.
- Mist no longer shrinks as you walk into it.

**New**
- A public API for other mods (`Straumr.Api`, see `API.md`): the water's current, depth, tide, white water and actual surface height at any point, river paths, power density, tide and flood events.
- Console: `straumr here` (the current split into its parts, and its way to the sea), `straumr report` (a file for bug reports, naming what stands in the water nearby), `straumr selftest` (PASS/FAIL checks where you stand), `straumr waterlevel`, `straumr perf`, `straumr waterstep` (a test).
- Atlas: a debug view (D) showing why the water does what it does at any spot. In the whole-world view, rivers carry as much foam as the sea, follow the water, and no longer loop round inside lakes. The atlas now only answers its own page and closes a few seconds after the page does.
- `[World] StandingRuins` makes visible an existing feature: world stonework doesn't crumble where a steep world leaves its edge hanging.

**Changed defaults**
- `[Diagnostics] LocationCheck` is now off (it's a tool for screening new worlds).
- `[Flecks]`: `Radius` 45 to 60, `Rate` 60 to 240 (it now counts how often the water is looked at, not flecks made), `MaxFlecks` 500 to 1500, `Size` 0.25 to 0.4. A config still holding all four of 0.1.0's defaults moves to the new ones by itself; anything you changed stays.

## 0.1.0: first beta

- Rivers that flow, from Valheim's own rivers and streams, or from Riverheim's river networks when it's installed.
- Froude-number white water, rapids that tumble swimmers, obstacles with wakes and eddies, bend flow, pools and riffles, confluence seams, boils.
- Sea currents, wind drift, rips and longshore drift.
- Tides that rise and fall, with tidal races at bay mouths, whirlpools off the straits, and the tide pushing up river mouths on the flood.
- A lunar month: spring tides at full and new moon, neaps between, high water a little later each day (one month per season with a seasons mod).
- A tide line under the minimap.
- Rain, seasons (with Seasonality) and spring snowmelt.
- Optional with Seasonality's winter freeze: fast water stays open (`[Stage] IceFreeSpeed`, off by default).
- ValheimRAFT vehicles drift with the current.
- Foam flecks on the water, synthesised water sounds, mist, driftwood and fish.
- Server settings sent to every player; weather swell replayed for late joiners.
- Flow map and seed atlas in the browser (`straumr atlas`), with a tide view (T): currents flooding and ebbing, the water rising and falling, a tide gauge.
