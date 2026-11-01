# Straumr

**Living water for Valheim.** Rivers flow downstream, rapids run white, the sea has currents and tides, and whirlpools turn off the ends of tidal straits. Swimmers, boats and floating things all ride it.

*Straumr* is Old Norse for "current". This is a **beta** (0.1): it works, it's been played on, and it's still being tuned. Feedback welcome.

## What it does

**Rivers**
- Every river flows from source to mouth, faster where it narrows, lazy where it widens into a lake, and slower near the banks than mid-stream.
- **White water** where fast water runs shallow, worked out from the Froude number, the way real rapids form. White water shoves swimmers about, dunks them now and then, and wears them out.
- Rocks and logs in the stream bend the current, with swirling wakes behind them and eddies beside fast water.
- Bends push surface water to the outside bank. Pools and riffles alternate along the river, and side rivers meet in a swirling seam.
- **Weather and seasons:** rain swells the creeks first and the big rivers later. With Seasonality, spring snowmelt runs high and winter runs low (or freezes, if Seasonality freezes the water).
- Dig a ditch from a river and the water flows down it. Dam it and the flow stops.

**The sea**
- Gentle wind-driven currents run along the coasts and through the sounds. Wind drifts the surface, rips run out from open beaches, and longshore drift runs along them.
- **Tides:** the sea rises and falls about 0.6 m twice a game day. Bays behind narrow mouths rise less and later, and their mouths race in and out (up to 1.5 m/s). Up river mouths the flood pushes against the river and the ebb speeds it out. Slack water comes at high and low tide, a little later each day, and the moon sets the size: spring tides (bigger, faster races and whirlpools) at full and new moon, gentle neaps between. With Seasonality, each season is one lunar month.
- **Whirlpools** spin up off the ends of tidal straits near the peak of each tide. Mind the eye.
- A line under the minimap tells you the tide when you're near the sea: *"Flood tide · full · high water in 6 min · +0.1 m"*.

**Life on the water**
- Foam flecks show which way the water runs. Rivers murmur, rapids roar (sounds made from the physics of bubbles, with no recordings), and mist lies on the water at dawn.
- Driftwood washes up in bends, and fish hold in the pools.
- Boats feel the current but always keep headway, and a boat stopped by a bank or dock stays moored.

## Works with

Straumr runs on its own, on Valheim's own worlds, and detects these automatically when they're installed. None is required.

- **[Riverheim](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/):** worlds built around big river networks, which Straumr sets flowing.
- **[Rivers and Lakes](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/):** streams, lakes and waterfalls above sea level. Works side by side with Straumr today. Straumr's currents in Rivers and Lakes' own streams and lakes switch on with its next update.
- **Seasonality:** seasons, snowmelt and freezing.
- **ValheimRAFT:** its vehicles ride the current too.

## Multiplayer

Install Straumr on the server and on every player's game. The server's physics settings (currents, tides, seasons) are sent to everyone who joins. How the water looks and sounds stays each player's own choice. Tides and seasons run off the server's clock, so everyone sees the same water.

## The flow map and seed atlas

- `straumr atlas` in the console (F5) opens a page from your own game at `http://localhost:8765`.
  - Type any seed: your game builds that world in memory (nothing is saved) and shows its rivers, sea currents, tides and whirlpools.
  - It starts whole-world, and fills in real detail as you zoom.
  - Switch between the Riverheim world and Valheim's own world for the same seed, and scrub the time of day to watch the tides turn.
  - It shows the whole map, so don't type a seed you want to keep a surprise.
- `straumr map` writes the world you're in to a file the same page can open.

## Console (F5)

These need `devcommands` first, or admin rights on a server. `straumr atlas` works for anyone, from the main menu too.

| Command | What it does |
|---|---|
| `straumr here` | the water at your feet: speed, depth, Froude number, tide |
| `straumr tide` | how the tide is running and when it turns |
| `straumr tide flood` | hold the tide (`flood`, `ebb`, `high`, `low`) in your game; `straumr tide auto` lets go |
| `straumr races` | the strongest tidal races and whirlpools within 4 km |
| `straumr map` | write this world's flow map to a file |
| `straumr atlas` | the seed atlas in your browser |

## Settings

Everything is in `BepInEx/config/wackah.straumr.cfg`, and most settings apply within a couple of seconds:
- `[Currents]`: river speed, sea, tides, tide range, whirlpools, rapids, what gets pushed.
- `[Stage]`: seasons and weather.
- `[Flecks]`, `[Sound]` and `[Mist]`: the look and sound.
- `[Hud]`: the tide line.

## Known limits (beta)

- Valheim's own checks that assume a fixed sea level (what counts as underwater for building, the beach's shading) don't move with the tide.
- The sea is one flat sheet per 64 m zone, raised as a whole, so neighbouring zones can differ by a few centimetres.
- Worlds much bigger than Valheim's standard size aren't supported yet.

## Problems or ideas?

Message me on [Discord](https://discord.com/users/220007226121781248). Please send your `BepInEx/LogOutput.log` (Thunderstore Mod Manager / r2modman: Settings > Browse profile folder) and say which other mods you run.

## Thanks

Inspired by the river worlds of Riverheim and the streams and waterfalls of Rivers and Lakes. Go give them a look:

- **Riverheim:** [Thunderstore](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/) · [Discord](https://discord.gg/ES3J7mu6kW)
- **Rivers and Lakes:** [Thunderstore](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/) · [Discord](https://discord.gg/vuHnDRqfjn)

Built with help from Claude, Anthropic's AI.
