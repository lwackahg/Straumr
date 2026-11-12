# Straumr

**Living water for Valheim.** Rivers run white, the sea rises and falls with the tide, and you can dam a river and watch the pond fill.

<!-- screenshot: the user picks -->

Straumr runs on Valheim's own worlds, and shows best on Riverheim's river worlds. **It requires only BepInEx:** no framework, no library and no config mod. This is a **beta** (0.1): it works, and it is still being tuned.

> **Want more?** The full guide (every feature, setting, command and known issue) is on GitHub: [GUIDE.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md). The API for mod makers is there too: [API.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/API.md). Questions and bug reports are welcome on the [Straumr Discord](https://discord.gg/bU9AD8hb8M).

## New in 0.1.5

- **After a crash, ponds come back as at the last world save.**
- **Built water is drawn closer to how it behaves:** water over a wall's own top, falls that join up, spills beside a dam, and a pond that is full when you join.
- **A ring's breach no longer leaves a pond rising over it.** A ring is a closed loop of walls between two dams.
- **Expand World Size:** any world size and "Stretch world" work in the lab. Default worlds are unchanged.
- **The atlas no longer changes the ground** of an Expand World Size world you play.

Every version's changes are in [CHANGELOG.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/CHANGELOG.md).

## What it does

**Rivers.** Every river flows from its source to its mouth, faster where it narrows and slower near the banks. Creeks run about 1 m/s and great rivers about 1.3 m/s. You swim about 2. Water slides round islands and rocks instead of running into them. Where fast water runs shallow it turns white, and it shoves swimmers, dunks them now and then, and wears them out. In a slot (the Strid) the surface stays smooth while the water below can pull a swimmer under for half a second. Wade into a current and it pushes you downstream. You can always walk out.

**River levels.** Rivers stand above the sea and climb gently toward their sources, up to 1.34 m above the sea in an ordinary season on the world it was checked on. With Seasonality they rise in spring, may spread over flood plains, and fall in winter. Rain raises them a little. Driftwood washes up just above the spring high water. Everyone on a server sees the same water.

**Dams and ponds** (experimental). Build a wall across a river, bank to bank, and a pond backs up behind it. It fills on the game clock until it spills over the lowest part of the wall, or round its end. Break the wall and the pond falls through the gap. The released water runs down the river as a front you can watch come. A door in the wall is a sluice gate. A hole broken low in the wall pulls swimmers toward it. Where nobody is near, the server carries logs and floating things on down a river until a wall or dam catches them (lab-tested, not yet played). Not every wall counts as a dam: see the limits below.

**Cofferdams** (experimental). Wall off a patch of river, lake or sea on every side, standing above the water. The water inside drains and lays the bed bare, so you can build on it. Break the ring and the water comes back through the gap. Break a dam above a dry cofferdam and the ring fills from that dam's pond, over a time you can watch.

**Kept pools** (experimental). Dig a basin, let river water in through a channel, then fill the channel in again. The basin keeps its water. Dig it deeper and the same water stands lower.

**Floods** (experimental, off by default). Rain raises the rivers: creeks within an in-game hour of a storm's start, great rivers about a day after the rain began. A high river holds back the rivers that run into it.

**The sea and tides.** Wind-driven currents run along the coasts, through the sounds and between islands. Rips run out from open beaches. The tide raises and lowers the sea up to 0.6 m either side of its usual level (0.8 m at spring tides), twice a game day, and the moon sets its size. Bays behind narrow mouths rise later, and their mouths race in and out. Swimming, boats, floating things and fires going out all follow the tide. With the wind onshore, waves carry driftwood and logs up onto beaches. Breakwaters calm the waves behind them across a harbour-sized basin.

**Whirlpools.** Whirlpools spin up off the ends of tidal straits near the peak of each tide. At the rim of a whirlpool's core the pull draws you in gently. Closer in, it is faster than you can swim, and you are carried into the eye. The eye pulls you under for a second every 4 s and costs stamina, until after 6 s it lets go and you can swim clear. Boats are not gripped. The sea dips into a funnel at the eye.

**Life on the water.** Foam shows which way and how fast the water goes. Rivers murmur, rapids roar (the sounds are made from the physics of bubbles, with no recordings), and mist lies on the water at dawn. Felled logs float and drift with the current. A boat left to drift in a river, with nobody at the helm, swings round to lie along it. A line under the minimap tells you the tide and your boat's speed.

**The atlas and the API.** `straumr atlas` opens a page that shows the rivers, currents and tides of any seed. Other mods can read the water through a public API.

## The builder's limits

- The server holds 16 dams, 64 kept pools and 64 cofferdams. A cofferdam can be at most 128 m across.
- A dam's wall must cross the river from bank to bank, 10 cm or more over its ordinary level. Raised ground alone is not a dam.
- A pond is judged up to about 2 km from its dam. Ponds longer than about 500 m were checked only in a closed-form test, not in the lab or in play.
- A pond or kept pool stands up to 60 m over the sea by default.
- Straumr knows the water within about 160 m of each player.
- More limits are in the [guide](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md#limits).

## Install

- Install Straumr on the server **and** on every player's game.
- A game with another Straumr version gets a notice on screen. It is not refused. A live 0.1.4 game joining a 0.1.5 server was not tried.
- Tested on a dedicated server with lab clients. **Not yet tested with a group of players.**
- Settings are in `BepInEx/config/wackah.straumr.cfg`. All of them are in the [guide](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md#settings).

## Riverheim

**Riverheim 1.2.x and 1.3.x both work.**

| | Riverheim 1.2.x | Riverheim 1.3.x |
|---|---|---|
| **Straumr 0.1.3 and later** | works | works |
| Straumr 0.1.2 | works | rivers break: update Straumr |

- **Back up before you move a world to Riverheim 1.3.x.** The first save converts a 1.2 world one way. Copy the world's `.fwl` and `.db` files first.
- **A world made with Riverheim_Config's settings** needs Riverheim_Config 3.1.1 or later on Riverheim 1.3.x. If not, its terrain changes.

## Read before you start

- **Dams, cofferdams and kept pools are experimental and ON by default.** On load, walls you built across a river become dams, and closed rings start to drain. To keep a world as it is, set `[Dams] Enabled`, `[Cofferdams] Enabled` and `[Pools] Enabled` to false. On a server, use the server's config. **Floods are OFF by default** (`[Floods] Enabled`).
- **The water can hurt a player.** Whirlpools pull swimmers under and cost stamina. The Strid can pull a swimmer under for a moment. White water shoves and dunks swimmers. A hole low in a dam's wall pulls swimmers through it. Wading in a current pushes you. To turn them off, set these to 0: `[Currents] Whirlpools`, `Strid`, `Rapids` and `Wading`. For the dam hole, set `[Dams] DragUnder` to false. On a server, these are the server's settings.
- **Works with:** Riverheim, Rivers and Lakes, Seasonality, ValheimRAFT and Expand World Size. Details are in the [guide](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md#works-with).
- **Mods that clash or change behaviour:**
  - **NorgerWater:** Straumr switches off its moving surface, dams, kept pools and breakwaters. Not tested in game.
  - **Undertow** and **Fires Valheim Galaxies:** running Straumr with either is not recommended. Both also push the sea currents, so boats and swimmers would be carried twice.
  - **shudnal's Seasons:** both mods run but do not work together.
- **If you remove Straumr:** your terrain and your pieces are untouched, and ponds and drained rings are plain Valheim water again. Objects the server placed stay (driftwood, fish, rocks at springs and sinks). Its files stay in `BepInEx/config`. This is from the code; a removal test in game was not run.

## Commands you need most

Type `straumr` commands in the console (F5).
- `straumr here` shows the water at your feet and what shapes it.
- `straumr dam show` marks where each dam near you spills.
- `straumr atlas` opens the seed atlas in your browser.
- `straumr report` writes a file about the water where you stand. Send it with a bug report.
- `straumr scenedump` (needs `devcommands`) writes the world round you to a file I can look at offline. Use it for anything wrong with the water you can see.

The full list is in the [guide](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md#commands).

## Known issues

Tiers, from worst to least: **Breaking** (a crash, a broken save, or water plainly wrong in ordinary play; a release does not ship with one), **Noticeable** (you will see it in normal play, and it stays), **Situational** (needs a particular build or timing), **Minor** (centimetres or seconds, and rare).

No known Breaking issues.

- **Noticeable: Springs and drains.** You see the current and foam pour out of, or into, one spot mid-river. It happens in some places on Riverheim worlds. A test of 214 squares of a Riverheim 1.2.1 world found 88 such spots (70 out, 18 in). Riverheim 1.3.1 gave 25 and 0. It does not settle. A fix is planned.
- **Noticeable (with Seasonality): a pond in a freeze.** In Seasonality's freeze, the rivers, dam ponds and kept pools are drawn at the sea's level, so a raised pond shows its bed dry until the thaw. A dam's pond also gets no river inflow all winter, so it drains toward its crest and refills in the thaw (from the code, not measured in play). A kept pool is drawn the same way, but nothing drains it (from the code). A frozen pond is not ice, a wall shows no falls, and logs lying on the ice can still be pushed about. It ends in the thaw. Planned.
- **Situational (while you change a dam's wall):** its pond and falls are drawn again. For about a second (up to 3 s) the pond can show as a flat sheet with straight sides, and a fall can jump or vanish. It settles when you stop.

10 more Situational and 4 Minor issues are in the [guide](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md#known-issues).

Not built yet (the ones players try first):
- **Dirt alone is not a dam.** A dam needs built pieces across the river.
- **No backwater above a pond.** A pond does not raise the river above it.
- **Dug cuts:** damming a river does not send it down a dug cut unless the cut ends lower than the dam's crest.

## Want more?

- **The guide:** [GUIDE.md on GitHub](https://github.com/lwackahg/Straumr/blob/v0.1.5/GUIDE.md) has every feature, limit, setting, command and known issue.
- **The API:** [API.md on GitHub](https://github.com/lwackahg/Straumr/blob/v0.1.5/API.md).
- **Questions, ideas and bug reports:** the [Straumr Discord](https://discord.gg/bU9AD8hb8M). Send the file from `straumr report`, and say which other mods you run.
- **Every version's changes:** [CHANGELOG.md](https://github.com/lwackahg/Straumr/blob/v0.1.5/CHANGELOG.md).

## Thanks

Inspired by the river worlds of Riverheim and the streams and waterfalls of Rivers and Lakes. Go give them a look:

- **Riverheim:** [Thunderstore](https://thunderstore.io/c/valheim/p/Gurebu/Riverheim/) · [Discord](https://discord.gg/ES3J7mu6kW)
- **Rivers and Lakes:** [Thunderstore](https://thunderstore.io/c/valheim/p/Voxheim/RiversAndLakes/) · [Discord](https://discord.gg/vuHnDRqfjn)

Built with help from Claude, Anthropic's AI.
