# Changelog

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
- A lunar month: spring tides at full and new moon, neaps between, high water a little later each day (one month per season with Seasonality).
- A tide line under the minimap.
- Rain, seasons (with Seasonality) and spring snowmelt.
- Optional with Seasonality's winter freeze: fast water stays open (`[Stage] IceFreeSpeed`, off by default).
- ValheimRAFT vehicles drift with the current.
- Foam flecks on the water, synthesised water sounds, mist, driftwood and fish.
- Server settings sent to every player; weather swell replayed for late joiners.
- Flow map and seed atlas in the browser (`straumr atlas`), with a tide view (T): currents flooding and ebbing, the water rising and falling, a tide gauge.
