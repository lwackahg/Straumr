# Straumr API

Straumr works out how the water moves on Riverheim (and Valheim) worlds (Riverheim 1.2.x and 1.3.x; 1.3 is read through Worldhalla, and Straumr needs only BepInEx): river currents, the sea's gyres, tides,
tidal races, whirlpools, rips, white water, floods and the winter freeze. It ships no models or textures (only its own shaders, built into the dll); this API is
how other mods (water wheels, mills, fishing, sailing AI) build on it, with no reflection.

**Design rule:** Straumr exposes physical properties (velocity, depth, power density, tide height and rate), not
bespoke mechanics (how fast your wheel turns, whether a harbour is safe, where the fish are). Those are your mod's to
decide from the physics.

Everything is in `Straumr.Api` in `Straumr.dll`: the static class `StraumrApi`, the struct `FlowSample` and a few
enums. `StraumrApi.Sample(position)` is the one primitive; the other helpers are built from the same look.

## New in 0.1.3

New members (each says "since 0.1.3" where it's described below): `FlowSample.Discharge`, `RiverRise`, `RiverRiseUsual`,
`RiverRiseHigh`, `FloodPlain`, `RiverStage`, `SurfaceFall`, `Flooded`, `FloodDepth`; `Outlook` and `RiverOutlook`;
`LevelsChanged`; `DriftPath`; `WaveShelter`, `WaveFactor`; `TestRain`, `StartTestRain`, `StopTestRain`,
`TestRainChanged`; `Cofferdam`, `CofferdamArgs`, `CofferdamChanged`; `Dam`, `Dams`, `DamArgs`, `DamChanged`; `Head`,
`HydraulicPower`, `Work`, `BestDrop`, `SetGateOpening`, `WorkArgs`, `DropArgs`, `WorkKind`.

Changed, with the same `ApiVersion` (no member was removed or changed its type):
- `SurfaceHeight`, `WaterLevel(pos)` and `Depth` include a river's rise above the sea (`[Currents] RiverLevels`, on by
  default), and with dams, kept pools and cofferdams on, their water. A mod that placed things at the water's edge by
  the sea's level should read `WaterLevel(pos)` (it was already the tide's level, not Valheim's fixed 30).
- `Velocity` includes lee eddies and chutes; `Kind`, `Source` and `Confidence` are as before.
- `TideHeight`, `TidalStream`, `FloodStream` and the tide's push up river mouths fade to zero where the game lays the
  Deep North's sea ice.
- With `[Floods]` on, `Outlook`'s `CrestLevel`, `LowLevel`, `MinutesToOverBanks` and `MinutesBackInBanks` include the
  flood room, and `LevelsChanged` fires at most once an in-game hour step (see "Floods" below).
- `Sample` inside a dam's pool, a cofferdam or a kept pool answers as that water (`Kind == Lake`, `Source == Local`).

## Setting up

```csharp
[BepInPlugin("you.watermill", "Watermill", "1.0.0")]
[BepInDependency("wackah.straumr", BepInDependency.DependencyFlags.SoftDependency)]
public class Watermill : BaseUnityPlugin
{
    void Awake()
    {
        // Touch Straumr's types only inside a class of your own, called only when Straumr is there,
        // so your mod still loads without it.
        if (BepInEx.Bootstrap.Chainloader.PluginInfos.ContainsKey("wackah.straumr"))
            StraumrBridge.Hook();
    }
}

static class StraumrBridge
{
    public static void Hook()
    {
        if (StraumrApi.InstalledApiVersion != 1) return;   // ApiVersion is a const: it's compiled into your mod
        StraumrApi.TideTurned += t => Debug.Log(t.HighWater ? "high water" : "low water");
    }
}
```

**Versioning.** `StraumrApi.ApiVersion` (a const) is raised only when a change would break a mod built against it (a
member removed, or its meaning changed); new members don't raise it. `InstalledApiVersion` reads the installed one at
run time; `Version` is the mod's own version. Members added after 0.1.2 (`Discharge`, `RiverRise`, `RiverRiseUsual`,
`RiverRiseHigh`, `FloodPlain`, `RiverStage`, `SurfaceFall`, `Outlook` and its `RiverOutlook`, `LevelsChanged`, `DriftPath`, `WaveShelter`, `WaveFactor`, `TestRain`, `StartTestRain`, `StopTestRain`, `TestRainChanged`, `Cofferdam`, `CofferdamArgs`, `CofferdamChanged`, `Dam`, `Dams`, `DamArgs`, `DamChanged`,
`RiverOutlook.FlowShare`, `Flooded`, `FloodDepth`, `RiverOutlook.FloodStage`, `Head`, `HydraulicPower`, `Work`, `BestDrop`,
`SetGateOpening`, `WorkArgs`, `DropArgs` and `WorkKind`) say "Since" where they're described: before using one, check `Version` and keep the code that touches it in a class of its own (as the bridge above),
so a mod built against a newer Straumr still loads against an older one.

**Threads.** Main thread only (Unity's `Update`, `FixedUpdate`, coroutines). Nothing throws: with no world loaded,
before the field is built, or away from water Straumr knows, you get "no water" (`InWater == false`, zeros,
`Kind == None`).

**Between worlds.** At the main menu and while a world loads, everything answers "no water" and `Ready` is false
until the new world's field is built; nothing of the last world is kept to answer for the next.

## What a sample holds

| Field | Unit | Meaning |
|---|---|---|
| `InWater` | | Straumr knows the water here (see Limits for where it does) |
| `Frozen` | | iced over by a seasons mod's freeze: `Velocity` and the tide fields are zero |
| `Velocity`, `Speed` | m/s | the depth-averaged current, horizontal (y = 0): exactly what carries swimmers, bent round rocks and docks and (since 0.1.3) curling into the lee of bars, islands and points, as far as the tide, the river's level and the waves uncover them. Swimmers ride at ~1.1x it, floating items 1.18x, boats 0.6x (Straumr's `[Currents] Boats`) |
| `Depth` | m | water depth, all of it: since 0.1.3 with a river's rise above the sea, and on ground a raised river has spread over |
| `Froude` | | `Speed / sqrt(9.81 * max(Depth, 0.5))`; above ~1 the flow is supercritical |
| `Whiteness` | 0..1 | how white the water runs: the same measure that tumbles swimmers (Froude-based, or a whirlpool's eye) |
| `TideHeight` | m | where the tide and any storm surge are taking the water here, above the sea's usual level (negative below); the water eases there at about 1 cm/s, so in a rising storm `SurfaceHeight` lags it. None where the game lays the Deep North's sea ice (it lies at the sea's usual level), fading out over the kilometre before it; `TideRate` with it |
| `TideRate` | m/s | how fast the tide is raising the water (+ rising). Tiny: a few mm/s at most. The tide's own rate: a storm's surge isn't in it |
| `SurfaceHeight` | m | the still water's surface here, absolute (no waves): the sea's level as it stands now, tide and any storm surge in and (since 0.1.3) a river's rise, or another mod's lake's level |
| `TidalStream` | m/s | the sea's tidal stream here, + flood, - ebb, as over deep water (shallows drag it back). Like `TideHeight`, none where the game lays the Deep North's sea ice, fading out over the kilometre before it; so do the tide's push up river mouths (in `Velocity`), `FloodStream` and the whirlpools there |
| `Openness` | 0..1 | 0 in a river's channel, 1 on wide open water |
| `Kind` | | `None, River, Lake, Sea, TidalRace, Rapids, Rip, Eddy, Whirlpool` |
| `Source` | | where the current comes from: `None, RiverChannel, Lake, OpenSea, Tidal, OtherWater, Fallback, Local` |
| `Confidence` | 0..1 | how representative the flow's *direction* is here (below) |
| `Discharge` | m³/s | how much water the nearest river carries now: its usual (Straumr's estimate: the land draining into it on Riverheim's worlds, the channel above it on Valheim's own, scaled to the speeds the rivers show) times `RiverStage`: the nearest river line's within 60 m, in the water or on its banks (as `RiverStage`); 0 with no line within 60 m, on straits and frozen. Since 0.1.3; since 0.1.3 below a dam also times the share of the river's water arriving there (`RiverOutlook.FlowShare`): where a Rivers and Lakes stream's water counts below a dam, a little under the water arriving, since the share is of the river's water with the stream's |
| `RiverRise` | m | how far the river's surface stands above the sea here now, besides the tide (in `SurfaceHeight`): rivers climb gently toward their sources, higher in spring; the surface slopes smoothly, it doesn't step at zone edges. 0 on the sea, in a freeze, with RiverLevels off. Since 0.1.3 |
| `RiverRiseUsual` | m | the river's level here in an ordinary season, as far as the banks let it (above the sea's usual level, besides the tide). Filled on dry ground too, like the next three. Since 0.1.3 |
| `RiverRiseHigh` | m | the most it may stand here, at the spring high water: how far up a bank the water can come. Since 0.1.3 |
| `FloodPlain` | | the ground here is flood plain: above the river's ordinary level, at or under its spring high water (`RiverRiseUsual` < ground <= `RiverRiseHigh`; the world's own ground, not players' digging). A fact of the place whatever the weather: whether it's under water now is `InWater`. Since 0.1.3 |
| `RiverStage` | | how full the nearest river runs now, times its usual for its size: season, weather and spring snowmelt (1 as usual, 1.4 = 40% faster and higher, 0 frozen; 0 with no river within 60 m). Since 0.1.3 |
| `SurfaceFall` | m/m | which way and how steeply the river's surface falls here (horizontal, pointing downhill; its length is the slope). About zero along most rivers, which climb gently; up to 0.03 where a river runs down into a stretch its banks hold lower (a chute); where it falls 0.01 or more along the current, the river runs faster there (by the square root of that fall) and white. Where the levels' zones meet beside a river it can tilt across the current: that part sends nothing anywhere. Zero on the sea, frozen, with RiverLevels off. Since 0.1.3 |
| `Flooded` | | flood water: the ground here (the world's own) lies above the river's ordinary level (`RiverRiseUsual`) and under the water as it stands now (`SurfaceHeight`, the tide in it). Rain floods ground with Straumr's `[Floods]` on; the spring melt and high tides do in any case. False on a river's own bed, on dry ground and frozen. Since 0.1.3 |
| `FloodDepth` | m | the flood water's depth over that ground (0 where not `Flooded`). Since 0.1.3 |

**Kind** is the first of these that holds: `None` (not in water); then, unless frozen, `Whirlpool` (in a turning
whirlpool's eye, or within its core plus 10 m), `Rapids` (Whiteness >= 0.25), `Rip` (a rip current >= 0.3 m/s),
`TidalRace` (|TidalStream| >= 0.5 m/s), `Eddy` (in a river's channel the water runs back upstream at >= 0.1 m/s, and
it isn't the flood tide pushing up a mouth); then `River` (Openness < 0.5 with a river line within 200 m), `Lake`
(another mod's lake, or open water the sea's tide barely reaches), else `Sea`.

**Confidence** is a contract of fixed values:

| Value | When |
|---|---|
| 1.0 | `RiverChannel` (a solved river line within 200 m), `OpenSea` (the solved sea), `Tidal` |
| 0.8 | `Local`: a stream's inflow jet or a swallow hole's pull is half the current or more. Modelled on purpose, but it changes within a few metres |
| 0.7 | `Lake`: open water the flow is filled in across, between the river lines |
| 0.5 | `OtherWater`: another mod's lake, where Straumr only adds the jets pouring in and the wind's drift |
| 0.3 | `Fallback`: channel-like water over 200 m from every river line, or open water before or without the sea's solve |
| 0 | `None`: no water, or water Straumr knows with its currents switched off in its config |

Each source has one value, so `Confidence` and `Source` say the same thing two ways: use `Source` to decide what kind
of answer you got, `Confidence` to weigh it. `None` covers two cases, told apart by `InWater`: no water (`InWater`
false), and Straumr's currents switched off (`[Currents] Enabled = false`: `InWater` true, `Velocity` and
`TidalStream` zero, `TideHeight` as usual). Frozen water keeps its source's value.

## Helpers

```csharp
Vector3 v   = StraumrApi.Velocity(pos);                 // Sample(pos).Velocity, cheaper
float   wm2 = StraumrApi.KineticPowerDensity(pos);      // 0.5 * 1000 * v^3, W/m2
float   h   = StraumrApi.TideHeight(pos);               // m, water or not (works on the shore too)
float   lvl = StraumrApi.WaterLevel(pos);               // absolute still surface here: tide, storm surge and (0.1.3) a river's rise in (not Valheim's fixed 30)
float   ph  = StraumrApi.TidePhase(pos);                // -1 full ebb .. +1 full flood, later behind straits
float   min = StraumrApi.MinutesToTideTurn(pos);        // real minutes to the next slack here
Vector3 fl  = StraumrApi.FloodStream(pos);              // flood-tide stream at an average peak (ebb = -fl); anywhere once the sea is solved; none on the Deep North's ice
Vector3 sea = StraumrApi.SeaCurrent(pos);               // the open sea's steady current, no tide/drift/surf; anywhere once solved
bool    sp  = StraumrApi.SpringTides;                   // also NeapTides, TideStrength (1 = average)
bool    ice = StraumrApi.IsFrozen(pos);
float   lee = StraumrApi.WaveShelter(pos);              // 0..1, the share of the waves' height a lee takes off here now (since 0.1.3)
float   wf  = StraumrApi.WaveFactor(pos);               // 0..1, Valheim's waves here over the open sea's at this wind, shelter in (since 0.1.3)
Vector3 dn  = StraumrApi.DownstreamDirection(pos);      // unit, the way the water runs along the river's line; zero off a river
Vector3 up  = StraumrApi.UpstreamDirection(pos);
int     n   = StraumrApi.SampleAlong(a, b, 25f, samples); // even samples, both ends, at most 1025
bool    ok  = StraumrApi.DownstreamPath(pos, 500, points);  // river points (~8 m apart) down the water to a lake or the sea
Vector3? m  = StraumrApi.RiverMouth(pos);               // where that path ends
int     k   = StraumrApi.DriftPath(pos, 120f, path, 1.18f); // where a floating item goes in two minutes, a point a second (since 0.1.3)
```

**Which way is downstream.** Riverheim draws each river from source to mouth, but often runs several rivers through
the same water, some of them the other way. Straumr gives each body of water one way down (its shortest way to the
sea, keeping the way a river is drawn unless the other way is less than half as long), so `DownstreamDirection` and
`DownstreamPath` follow the water, which on a few stretches runs against the line as drawn.

**The water's level.** Straumr raises and lowers the sea (and everything at sea level, rivers and lakes included)
with the tide, so the water's surface is often not Valheim's fixed 30 m. Anything placed or spawned at the water's
edge should go by `WaterLevel(pos)`: the still-water surface there in metres, absolute, as the water stands now (tide
and any storm surge included, and since 0.1.3 a river's rise: rivers stand above the sea, climbing toward their sources;
it eases toward them), waves left out. It's 30 (Valheim's
level) with no world, with tides and surge off (`[Currents] TideRange = 0` and `StormSurge = 0`), in a freeze, before
the sea is solved and on a dedicated server. Rivers and Lakes' water above the sea isn't covered: where `Sample(pos).Source` is
`OtherWater`, that lake's level is in `Sample(pos).SurfaceHeight`; its streams aren't sampled at all.

**Waves and shelter (since 0.1.3).** Valheim's waves grow with the water's depth, which the game reads only at the
four corners of each 64 m zone (how far the ground there lies under 30 m, over 10 m, at most 1) and blends between them; the wind sets how
big they get (the biggest of its ten wave trains stands 8 m from trough to crest at full wind, times that factor).
Straumr's breakwaters lower those corner values in the lee of what players build (`[Currents] Breakwaters`, and with
`NaturalShelter` the land too), for what floats and for what's drawn alike. `WaveFactor(pos)` is the game's factor
as it stands now, shelter in; `WaveShelter(pos)` is the share of the height shelter takes off (0 with breakwaters off,
easing back to 0 over up to 20 s once they're turned off). Both are 64 m coarse, as the game's waves are, ease in and
out over up to 20 s when the wind turns or something is built, and answer only where the zone's water is loaded round the local player (0 elsewhere and on a dedicated
server). They leave out the game's fading of the big waves toward the Deep North. Under NorgerWater the sea is drawn
by Crest: `WaveShelter` is 0 and `WaveFactor` is Valheim's own value, which Crest may not follow.

## A river's outlook (since 0.1.3)

```csharp
RiverOutlook o = StraumrApi.Outlook(pos);   // once a second at most: it looks four hours ahead
if (o.River)
    Log($"{o.Cause}: x{o.Stage:0.00} now (season x{o.Season:0.00}, rain +{o.Rain:0.00}, snowmelt +{o.Snowmelt:0.00}); " +
        $"fullest x{o.CrestStage:0.00} in {o.MinutesToCrest:0} min, level up to +{o.CrestLevel:0.00} m in {o.MinutesToCrestLevel:0} min");
if (o.River && o.FloodPlain && o.MinutesToOverBanks >= 0f)   // a field on the flood plain: when it goes under, and comes out (-1: not within 4 h)
    Log($"{o.RiverType}: this ground under water in {o.MinutesToOverBanks:0} min, out again in {o.MinutesBackInBanks:0} min");
```

- One answer to "how high, and why": the river's levels (`LevelUsual`, `LevelSpring` as the banks allow, `LevelNow` as
  drawn), how full it runs (`Stage` = `Season` x (1 + `Rain`) x (1 + `Snowmelt`)), what it carries now and as usual
  (`Discharge` = `DischargeUsual` x `Stage` x `FlowShare`: `FlowShare`, 0..3, is the share of the river's
  water arriving here after any dam above, 1 with none), the
  surface's fall (`Slope`), the tide's height, whether the ground is
  flood plain (`LevelUsual` < ground <= `LevelSpring`, whatever the water does now), and a `Cause` in words ("as usual",
  "rain", "snowmelt", "rain and snowmelt", "a high season", "a low season", "frozen"; the tide moves the level and a mouth's current, `Tide` and `Estuary`, not how full the river runs).
- The next four hours: `CrestStage` and `MinutesToCrest`, `CrestLevel` and `MinutesToCrestLevel`, `MinutesBackToUsual`.
  Valheim draws its weather from the clock, so the coming rains are as certain as the past ones; the outlook is the same
  rain, season and snowmelt run forward on the same clock, the same in every game. The levels change once a weather
  spell (five minutes into it) and fade in over half a minute; with Straumr's `[Floods]` on, they step
  every in-game hour (75 real s) instead, each step fading in over the next 75 s, and the outlook looks at every
  fourth step (every five real minutes). `MinutesToCrestLevel` is when that fade is done.
  `CrestLevel` is the level the banks allow, the target the surface is drawn toward: the drawn height (`LevelNow`) can
  be lower where the surface slopes toward lower water. "Back to usual" is the season's own run (`Stage` within 2% of
  `Season` then). The season's factor moves with the calendar as in the rivers themselves; the outlook can't see past a
  season's turn (the weather changes then) nor a winter freeze coming. A rain's effect on the levels shows only where
  this game's weather is the server's.
- The lows and the banks, from the same forecast: `LowStage` and `MinutesToLow` (the lowest it runs over the four
  hours, and when: the snowmelt's low at dawn, a season easing off), `LowLevel` and `MinutesToLowLevel` (the lowest its
  level stands, as the banks allow, like `CrestLevel`), and `MinutesToOverBanks` and `MinutesBackInBanks`: when it next
  stands over its banks here and when it's back within them (0: now; -1: not within the four hours). Over its banks is
  above its ordinary level (`LevelUsual`, the top of its banks, where the flood plain begins); where the ground you
  sample is `FloodPlain`, it's above that ground, so the pair says when that ground goes under water and comes out
  again. All of them are the levels the banks allow (the drawn surface can stand lower where it slopes toward lower
  water), besides the tide. Where the drawn water stands more than 5 cm under that now (held by the water it runs out
  of or into: a stretch upstream whose banks hold the river down, or lower water), the pair is -1: a higher target
  needn't lift it.
- What kind of river, the numbers first: `Width` (its usual width here, m), `Catchment` (the land draining through
  here, km²: Riverheim's own catchment, its land weighted by its own rainfall; 0 on Valheim's own rivers and on
  straits. The world is small, so its rivers drain far less land than real rivers their size, and `DischargeUsual` is
  scaled to the speeds they show, not to this), `SnowShare` (0..1: how much of its water mountain snow feeds, 1 for a
  river rising in or under the mountains, half that for the river it runs into, and so on down; spring's daily melt
  pulse goes by it), `LakeShare` (0..1 of `DischargeUsual`: how much of it has come through a lake on its way),
  `TidalReach` (how far up its line from its mouth the tide reaches with at least 5% of the open sea's range, m; 0
  before the sea is solved) and `FromMouth` (how far up its line this point lies, m: it's in the tidal reach where
  `FromMouth <= TidalReach`). A river's mouth here is where it meets the sea, a lake or the river it joins.
  `RiverType` says it in words, as `straumr here` does: "a snow-fed great river, tidal in its last 3.2 km". It's for
  showing players; to decide anything, use the numbers with thresholds of your own. Straumr's:

  | Words | When | Where the line comes from |
  |---|---|---|
  | creek, small river, river, great river | `Width` under 11 m, to 35 m, to 104 m, beyond | Straumr's four flood sizes (half-widths 3, 10, 30 and 90 m, each flooding on its own clock): the lines are their geometric middles, so a river called great floods as a great river does. No standard names rivers by width |
  | snow-fed | `SnowShare` >= 0.5 | mountain snow at least half its water: where the melt rules a river's flow |
  | lake-fed | `LakeShare` >= 0.5 | most of its water has come out of a lake |
  | tidal in its last ... | `TidalReach` > 0 | at least 5% of the sea's range: NOAA's head of tide is where the mean range falls under 0.2 ft (6 cm), 5% of Straumr's default 1.2 m. The same 5% `Kind`'s Eddy rule takes for the flood up a mouth |
  | a lake on ..., the open water ... ends in | the point is out across a lake on the river's way, or in the open water it ends in | Straumr's lakes: a stretch of a river's line across water at least 40 m wide with about four times its usual section (not other mods' lakes) |
  | a strait between seas | Riverheim's channel between two seas | no river's water of its own: the tide drives it |

  Braided and spring-fed rivers aren't named: Riverheim's rivers never part (each piece of ground drains one way), and
  Straumr knows no groundwater. Rivers ending in a lake and handing their water on through it aren't in the next
  river's `LakeShare`, as they aren't in its `DischargeUsual`.
- `RiverId` is Straumr's index of the nearest line, for the running game only: never save it.
  `TrunkRiverId` is the river its water finally joins, through rivers and lakes (itself if none), `Mouth` where that
  river ends and `ReachesSea` whether the water gets to the sea there (else a closed lake): a watershed is every river
  with one `TrunkRiverId`. For anything kept between sessions (saved data, a dam), use `RiverAnchor` and `TrunkAnchor`
  instead: built from the world's seed and where the river rises (and where its trunk meets the sea or its lake), so
  the same river gets the same anchor every time the world loads. An anchor is a matching hint, not a guarantee: should
  a world generator update move a source a little, match by `Source` (where the river rises) instead.
- Where the tide reaches up a river: `RiverPush` and `TidePush` (m/s along the river's way down, + seaward) and
  `Estuary`, which has the upper hand ("the river pushes out against the flood tide", "the flood tide holds the river
  back", "the river and the flood tide in balance", "the ebb draws the river out"; null where the tide doesn't reach).
- `FloodStage` (since 0.1.3): `Stage` / `Season`, how far rain and spring's snowmelt run the river over its season's own
  (1: not at all; 0 frozen).
- Players' games only; main thread.

## Floods (since 0.1.3, experimental)

```csharp
var s = StraumrApi.Sample(pos);
if (s.Flooded)
    Log($"flood water {s.FloodDepth:0.00} m deep here: dry ground at the river's ordinary level");
var o = StraumrApi.Outlook(pos);
Log($"running x{o.FloodStage:0.00} over its season's own; highest +{o.CrestLevel:0.00} m in {o.MinutesToCrestLevel:0} min");
```

- Only with Straumr's `[Floods] Enabled` on (off by default; on a server, the server's setting): rain then
  raises each river's level by its own size's rain, as it already speeds its current. Creeks rise within an in-game
  hour of a storm's start and peak with the rain; great rivers start rising some ten in-game hours in, peak about a day
  after the rain began and are back within about five days (`straumr flood` gives the steps at a spot). A high river
  holds the rivers running into it up at their mouths (backwater), and a stretch whose low banks hold the river down in
  ordinary weather spills over them in a flood (its flood room, `[Floods] FloodRoom` and `FloodPonds`). Without it, the
  levels follow only the great rivers' rain, as in 0.1.3.
- `Flooded` and `FloodDepth` (on `FlowSample`): ground above the river's ordinary level (`RiverRiseUsual`) under the
  river's water as it stands now, and how deep. The tide is in the water's height, so tidal marsh at high water and
  spring's flood plains count as well as rain's floods; a dam's pool doesn't, the surge below a breached dam does. Where
  two rivers' water meets (a creek's mouth, a lake's edge) the surface can stand a few cm over the nearer one's ordinary
  level on an ordinary day.
- `FloodStage` (on `RiverOutlook`) is the river's flow over its season's own (the currents' `Stage` / `Season`), not its
  level: the levels follow `[Floods]` (its FloodScale, the hourly steps), so a level's rise lags the flow's.
- **Contract changes, no `ApiVersion` change:** with floods on, `CrestLevel`, `LowLevel`, `MinutesToOverBanks` and
  `MinutesBackInBanks` include the flood room and the backwater from the water downstream ("as the banks and the flood
  allow"); and the levels step every in-game hour, so `LevelsChanged` fires at most once a step (75 s) while the water
  moves, where it fires at most once a weather spell with floods off. `LevelNow` and `RiverRise` are the drawn water,
  as before.
- Every game works the same flood from the server's clock, its weather and the shared settings; nothing new is sent.
  A game whose weather isn't the server's (`straumr weather` says so) gets no rain floods in its levels, as for 0.1.3's
  rain. Under NorgerWater the rivers stay at the sea's level, floods too (their currents still run faster).
- Players' games only (the levels are worked out round the local player); main thread.

## A test rain (since 0.1.3)

```csharp
// On the server (a dedicated server, or a game hosting its own world): rain on every game's rivers, for an event.
if (StraumrApi.StartTestRain(1f))            // pulse: ~1 a rain over the lowlands, 0.4 light rain, ~2 a thunderstorm
    Log("the rivers are rising everywhere");
StraumrApi.TestRainChanged += r => Log(r.On ? $"test rain, pulse {r.Pulse:0.00}, {r.MinutesLeft:0} min left" : "test rain over");
bool on = StraumrApi.TestRain.On;            // any game, dedicated servers too
StraumrApi.StopTestRain();                   // server only: the rivers back to the real weather's at once
```

- The same rain as an admin's `straumr rain`: the rivers act as if it rained over the lowlands (Meadows, Black Forest,
  Plains, Mistlands). It brings no sky: Valheim's weather goes on as drawn, so an event mod starts its own weather
  besides. `Pulse` is a spell's pulse as `straumr weather` shows it: how much wetter than usual the lowlands are (about 1
  for a rain over all of them, 0.4 light rain, 2 a thunderstorm with the default settings, less their usual wetness).
  In its spells the rivers take the wetter of it and the real weather's: a test rain never dries a real storm.
- It falls from the last weather spell through this one (so the rivers are up at once; the levels follow at their next
  look: the next spell's, or with `[Floods]` on the next in-game hour's) and ends by itself when this spell ends (`MinutesLeft`), at the same moment in every game. The rivers then drain
  it as after a real rain: the creeks over a quarter of an hour, the great rivers over an hour or so. Asked for again
  while one falls, or within a spell of its end, it runs on through this spell from where it began, at the wetter of
  the two pulses (so the rivers never dip between the two); later, a new one starts. Asked for again unchanged, nothing
  is sent. `StopTestRain` takes it out of the rivers' memory at once.
- Three numbers, the spells it begins and ends and its pulse, go to every game with Straumr's synced settings, players joining late
  included; each game works out the rest from the server's clock, as it does for real rain. Players need a Straumr that
  has this (0.1.3); older ones keep the real weather's rivers. Where a game's weather isn't the server's (`straumr
  weather` says so), its currents follow the rain but its levels don't, as for real rain.
- `StartTestRain` and `StopTestRain` are server only: false on a player's game connected to a server, so a player's mod
  can't make rain for everyone (admins use `straumr rain`, which the server checks against its admin list). Also false
  with no world, with Straumr's `[Stage] Weather` off, or where Valheim's weather can't be rebuilt. A pulse is clamped
  to 3.
- Not saved: a server's restart ends it. An event that must outlast a restart starts it again.
- `FloodChanged` fires for a test rain's flood as for a real one: `TestRain.On` tells them apart. `TestRainChanged` is
  raised when one starts, is replaced or ends (by itself, or stopped), once a second at most, after the first minute in
  a world (as the other events). Unlike the river members above, these work on a dedicated server too. Main thread.

## Cofferdams (since 0.1.3, experimental)

```csharp
if (StraumrApi.Cofferdam(pos, out CofferdamArgs dam))       // is this point inside a walled-off patch of water?
    Log(dam.Sealed ? $"cofferdam {dam.Id}: water at {dam.Level:0.00}, dry in {dam.MinutesToDry:0.0} min"
                   : $"cofferdam {dam.Id}: breached ({dam.BreachWidth:0.0} m), full in {dam.MinutesToFull:0.0} min");
StraumrApi.CofferdamChanged += d => Log(d.Gone ? $"cofferdam {d.Id} full again" : d.Sealed ? $"cofferdam {d.Id} closed" : $"cofferdam {d.Id} breached");
```

- Only with Straumr's `[Cofferdams] Enabled` on (on by default in 0.1.3; on a server, the server's setting). A patch of
  river, lake or sea walled off on every side, by built pieces (and ground where they meet a bank or ground raised with
  the hoe) standing at least 10 cm above the highest water outside (the river's level this in-game hour, plus the
  highest spring tide and a full storm's surge as far as the tide reaches there), with no gap, is a cofferdam. Its water
  falls at `[Cofferdams] DrainRate` (0.5 m a real minute by default) until it stands half a metre under its lowest
  ground; broken, it fills back through the gap as over a weir (Q = 1.6 x width x head^1.5) until it's within 5 cm of
  the water outside.
- The server decides which rings are closed and stamps when, and every game works the water out from that and the
  server's clock: the same level everywhere, so a boat, a swimmer and a dropped item sit at the same water in every
  game. A game without the server's word (an older server's Straumr, before the settings arrive) sees no cofferdams.
- Inside one, `Sample` gives its own water: still (`Velocity` zero; a breach's inflow jet while it fills), `Kind ==
  Lake`, `Source == Local`, `Depth` and `SurfaceHeight` the lowered water's; where it's dry, `InWater == false`.
  `WaterLevel(pos)` is the lowered surface too. `Level` is under `Bed` once it's dry; `Outside` is the water outside
  as this game draws it.
- `Id` is the server's number; the server keeps its rings with the world (`BepInEx/config/Straumr-water/<world>.dams.txt`,
  written each time the world saves), so after a restart each comes back as it was, its number too, draining or refilling
  on from where it was. A ring the server first finds already closed with no record of it (a world from a build that
  didn't keep them, or `[Cofferdams]` just turned on) is taken as long drained (`MinutesSince` counts from then).
- A ring below a dam that breaks fills from that dam's pond through the opening, at a pace set when the fill begins: a short fill plays near its own length and a long one is sped up, never past the clock's own pace (48 times with Valheim's 1800 s day). `MinutesToFull` follows it, in clock minutes. When the ring is full its pool is the lower dam's pond.
- `CofferdamChanged` is raised when a ring closes (`Sealed`), breaks, closes again, or the water inside is level with the
  outside again (`Gone`), in every game, dedicated servers too, after the first minute in a world. Main thread.

## Dams (since 0.1.3, experimental)

```csharp
if (StraumrApi.Dam(pos, out DamArgs dam))          // in a dam's pool, or on the river it cuts below it?
    Log(dam.Below ? $"below dam {dam.Id}: the river carries {dam.Share:P0} of its water, standing {dam.Surge:+0.00;-0.00} m off its own level"
        : dam.Spilling ? $"dam {dam.Id}: pool at {dam.Level:0.00}, {dam.Outflow:0} of {dam.Inflow:0} m³/s leaving"
        : $"dam {dam.Id}: pool at {dam.Level:0.00}, rising {dam.RisePerHour:0.0} m an hour, spills in {dam.HoursToSpill * 60f:0} min");
StraumrApi.DamChanged += d => Log($"dam {d.Id}: {(d.Gone ? "gone" : d.Breached ? "breached" : d.Spilling ? "spilling" : "closed")}");
var dams = new List<DamArgs>(); StraumrApi.Dams(dams);   // every dam the server has sent
```

- Only with Straumr's `[Dams] Enabled` on (on by default; on a server, the server's setting; a dedicated
  server also needs `[Currents] RiverLevels` on) and not under NorgerWater. Built pieces (with any ground raised beside
  them: raised ground alone isn't found) that cut a river's water in two from bank to bank, standing at least 10 cm over
  its natural level (an ordinary season's, without the tide), are a dam. The water backs up behind it as a level pool over
  everything joined to it below that level, at the river's own flow, until it spills over the dam's lowest part, finds
  another way down round it, or reaches `[Dams] Ceiling` over the sea (60 m by default, up to 100; Valheim's own water ends 10 m over the sea, and above that Straumr's own water volumes carry the pool in each player's game: an older Straumr can't swim, sail or float in that part). The river below carries only
  what comes over or through the dam, and a change in that (the dam closing, a breach, a gate opening) runs down the
  river as a front, at the river's own current plus a long wave's speed in its depth (V + sqrt(g d): about 6.6 m/s on
  a river 3 m deep running 1.2 m/s, in real seconds, so it can be seen running): ahead of it the river runs as
  before, behind it with the dam's water. The river stands higher behind a front that brings more water (a breach's
  surge: the extra water over its width times the front's speed, B w) and lower behind one that brings less (a pool
  filling: never under the sea), within what its banks hold in spring (in a flood, with `[Floods]` on); where the
  river ends, the water it runs into holds its level, and once the front has had time to run there and back the river
  settles where that much water stands it (each step of its rise below there times the share of its water squared, less
  one). Rivers and Lakes streams pouring in below a dam count toward what arrives below them where they bring more
  than the river's own flow gains there. Below a dam the share (the current) is held to 0..3 times the river's water;
  the surge's level takes the water as it comes (a breach's real peak), within the banks. Where no dam takes water from
  a river or adds to it (none built, or each one sending down what comes in, to within 1 % of its river), the river
  below is as without dams, to the bit; below one whose water goes elsewhere for good (bypassed, or spilling round its
  end) the river stays lower for as long, at that settled level, as its current is slower.
- The server finds the dam and stamps its pool's course; every game works the pool out from that record and the
  server's clock, step by step on the same tables, so a boat, a swimmer and a dropped item sit at the same water in
  every game. A game without the server's word (an older server's Straumr, before the settings arrive) sees no dams.
- **Time** is Valheim's: hours are in-game hours (75 real seconds with Valheim's 1800 s day; sleeping moves them on).
  Durations are real ones mapped one to one onto in-game hours (a pool that would take 3 hours to fill takes 3 in-game
  hours).
- `DamArgs`: `Id` (the server's number this session, for people); `Key` (its lasting identity, from the world's seed,
  its river's anchor and where its first wall cut the river: the body of water it holds, which keeps this key when its
  wall is broken and a new one built across the same water; keep this, never `Id`); `RiverAnchor` (keep this, not `RiverId`) and
  `RiverId`; `Middle`; `Crest` (the lowest level it spills at, absolute m) and `CrestWidth` (m of crest within 10 cm of
  it); `Natural` (the river's own level at the dam, an ordinary season's); `Level` (the pool now, absolute); `Stored`
  (m³ over `Natural`; 0 below it); `Area` (m²); `Inflow` (m³/s: its river's own, and while a dam above sends a surge or holds water back, what reaches it from there), `Outflow` (m³/s); `Spilling`; `RisePerHour` (m an in-game hour, negative
  falling); `HoursToSpill` (-1 spilling or not rising); `WayRound`, `WayRoundLevel`, `WayRoundAt` (water the pool would
  pour into besides the river below); `Bypassed` (the water goes round at the river's own level, into other water: no
  pool, though the river below still loses its water; where it goes round into the river below itself, what was built
  doesn't cut the river at all, and nothing below changes); `Judged` (its pool is worked out: false while the server works it out, in a
  river's tidal reach, or for a pool reaching further than 512 m round the dam before it spills, which Straumr doesn't
  judge yet); `Breached`, `BreachWidth`; `Found` (standing when the server first looked with no record of it: taken as
  long closed, its pool full); `Gone`; and for the point asked, `Below`, `Share` (0..3 of the river's usual water arriving there, as
  the front from the dam has brought it: above 1 while a breach or an open gate lets the pool out; 1 in the pool) and
  `Surge` (m the river stands over its own level there, negative under it: every dam's together, their water summed at
  each point, as far as the banks allow and as much of it as the rivers' rise is drawn; 0 off the rivers below dams.
  Where another dam's pool stands there, the water is drawn at the higher of that pool and this).
- `Sample` in a pool: `SurfaceHeight`, `Depth` and `RiverRise` include it, flooded land is `InWater`, and `Kind ==
  Lake`. Below a dam, `Velocity` is the cut current and `Discharge` the water arriving (usual x stage x share), and
  `SurfaceHeight`, `Depth` and `RiverRise` include the surge. `WaterLevel(pos)` is the pool's (the surge's below).
  `Outlook`'s `LevelNow` includes the pool and the surge, and `FlowShare` is the share arriving.
- Since 0.1.5, on a dedicated server a pool's lift is counted in `Sample`'s `RiverRise` and in `Outlook`'s `LevelNow` (before, it counted as 0 there; the server's own river levels still stand at the sea's level). `ApiVersion` stays 1.
- A breach is a wall broken (a piece taken out): the pool falls through the gap as over a weir drowned by the water
  below, and settles where the gap alone passes the river. Overtopping never breaks a dam yet. A dam leaves
  (`Gone`) only once none of its walls stands and its pool has drained.
- A door in a dam's wall is a sluice gate (see "Water power and gates" below): the dam is found with every door shut,
  so opening one never breaches it; the pool falls through the doorway instead, and `Outflow`, `RisePerHour` and the
  river's `Share` below include what the gates pass.
- `DamChanged` is raised when a dam closes (or is found standing), its pool starts spilling, it's breached, closes
  again, a gate in it is set, or it goes, in every game, dedicated servers too, after the first minute in a world. Main
  thread.
- Saved with the world: the server writes its dams to `BepInEx/config/Straumr-water/<world>.dams.txt` each time the
  world saves (and between saves, at most every half minute while they change) and reads them when it loads, so a pool filling,
  spilling or draining through a breach goes on from where it was after a restart, and `Id` stays. A dam standing with
  no record there (a world from a build that didn't keep them, `[Dams]` just turned on) is found standing and taken as
  long closed (its pool at its steady level for the river's flow then).

## Kept pools (since 0.1.3, experimental)

No members of their own; what they change in the members above:

- Only with Straumr's `[Pools] Enabled` on (on by default; on a server, the server's setting) and not under
  NorgerWater. Water the players cut off from the river with ground alone (a basin dug beside it and its channel filled
  in again, a bay closed with raised ground within about 8 m of the ground raised) keeps the water it held when it was
  cut off (without the tide; a dam's pool's, where it was dug from one) instead of following the river: a volume, not a
  level, so digging its bed deeper lowers it, earth thrown in raises it, and over its rim it spills. Water it lets out
  into a river (through a gap or a door in its rim, a gate) is that river's: the current, the surge and `Discharge` below
  follow it, as below a dam; a river filling it loses that water. Only
  water the server saw joined to the water outside and then cut off: a pond it never saw joined stays as it is. Water
  coming up over its rim fills it and it's part of that water until it falls back under the rim; a gap dug in its rim
  lets it fill or drain to the water outside, and then it's that water again (or, where that's a dam's pool's, stays
  joined to the pool); two kept pools joined by digging come level. The filling and draining go at the storage clock's
  pace (an in-game hour's filling in an in-game hour). A cut that needs a built piece to close is a cofferdam, as before.
- Inside one, `Sample` gives its own water, as inside a cofferdam: still (`Velocity` zero; while it fills from the water
  outside, through a gap dug in its rim or over its rim, the inflow jet), `Kind == Lake`, `Source == Local`, `Depth`
  and `SurfaceHeight` its own; where it's dry, `InWater == false`. `WaterLevel(pos)` is its level. `Cofferdam(pos)` is
  false there, and no event is raised for a kept pool.
- `Outlook`'s `LevelNow` counts a pool held over the water round it as it counts a dam's pool; one held under it isn't
  counted (nor is a cofferdam's lowered water).
- Saved with the world: the server writes its kept pools to `BepInEx/config/Straumr-water/<world>.txt` each time the
  world saves and reads them when it loads, so they keep their levels through a restart.

## Water power and gates (since 0.1.3, experimental)

```csharp
if (StraumrApi.Work(pos, out WorkArgs w))                     // a dam's gate, spilling crest or breach within 10 m?
    Log($"{w.Kind} of dam {w.DamId}: {w.Flow:0.0} m³/s over {w.Head:0.00} m: {w.Power / 1000f:0} kW of water power");
if (StraumrApi.BestDrop(pos, 30f, out DropArgs d))            // within 30 m: where to take water in and give it back
    Log($"in at {d.Intake}, out at {d.Outlet}, {d.Head:0.00} m lower, {d.Flow:0.0} m³/s: up to {d.Power / 1000f:0} kW");
float head = StraumrApi.Head(upper, lower);                    // m, NaN unless both points are water Straumr knows
float watts = StraumrApi.HydraulicPower(flow, head);           // rho g Q H: 9.81 kW per m³/s falling a metre
StraumrApi.SetGateOpening(doorView, 0.5f);                     // a sluice half open, for Straumr (the door stays as Valheim has it)
```

- **Rates are per in-game second.** Every flow Straumr gives (`Flow`, `Inflow`, `Outflow`, `Discharge`) is m³ per
  in-game second, and `Power` is watts of in-game time (J per in-game second). In-game time runs 86400 / the day's
  length times real time: 48 with Valheim's 1800 s day (a mod changing the day's length changes it). So a gate passing
  17.5 m³/s over 2.7 m (464 kW in-game) moves 840 m³ and 22 MJ every real second. A machine of yours that counts real
  seconds (a smelter's timer) must convert. Hours are in-game hours (75 real seconds with Valheim's day).
- **Head is what there is to use.** Straumr's rivers stand only a little over the sea (a median 0.41 m on the Riverheim test world, 0.06 m on
  Valheim's own, as measured on an earlier test build), so a river's own drop near a point is centimetres: `BestDrop` down a stretch of river gives watts to
  a few kW. A dam holds metres: a river of 28.3 m³/s falling 2.7 m over a dam carries about 750 kW. What a machine
  takes is yours to decide (a wheel or turbine on a fall perhaps 60-90 %, an undershot wheel in a current far less:
  `KineticPowerDensity`).
- **Gates.** With `[Dams] Enabled` on, every door piece standing in a dam's wall (where it walls the river: without it,
  the cells' tops fall 10 cm or more) is a gate, at most 8 a dam, lowest first. The dam is found with every door shut,
  so an open sluice is still a dam, never a breach. Through an open gate, a weir over its sill (what the door stands on)
  while the pool stands under its lintel (the underside of what spans the doorway), and over the lintel the lesser of
  that and an orifice, Q = 0.6 x width x height x sqrt(2 g dh), dh down to the water below or the opening's middle,
  whichever is higher; both ways (back into the pool if the water below stands higher). A 2 x 2 m door with the pool
  2.7 m over the water below passes 17.5 m³/s. A gate opens and shuts over 5 in-game minutes (6.25 real s): Valheim's
  door swings at once, the water follows. Side-by-side or stacked doors are the partial settings (a bottom door is a
  low outlet, a top door a weir); `SetGateOpening` sets any opening from 0 to 1 (`WorkArgs.Open` is where it is now,
  `Target` where it's going). The server takes a door's new state or setting once two of its looks agree (2-4 s), so
  a lever or slider moving every frame doesn't re-send the dams every look. Where nothing stands on a door, its top is
  part of the dam's crest (the water over it spills as over the wall), and the gate is the doorway under that top.
  Barred doors and gates (the iron gate) pass no water while shut. A door taken away (smashed, or a wall built over
  it) leaves its gate shut at once, and the hole is a breach. The jet through an open gate runs at the opening's
  speed, shown at most at `[Currents] MaxSpeed` (3 m/s by default).
- **Every door is shut to the dams, not only those in a dam's wall** (for now): an open door in a side
  channel, a mill race or a way round a dam is wall to the dam's pool, and so are the doors past a dam's first 8. Leave
  a gap rather than an open door where the water should go round.
- `Work`: `Kind` (`Spillway`, `Gate`, `Breach`), `Key` (a gate's lasting key, from the world's seed and its door piece,
  which is saved with the world: one key a door as long as it stands; a spillway's or breach's is its dam's), `DamKey`,
  `DamId`, `Number` (a gate's along its dam: by sill, then by the door piece), `Middle`, `Upper`
  (the pool now), `Lower` (the water below, as this in-game hour's tables have it), `Head`, `Flow` (now), `Power`,
  `Open`, `Target`, `Width`, `Sill`, `Soffit` (+infinity: nothing over it). A shut gate is a work with `Flow` 0. A
  breach broken low with pieces still standing over it (a hole) is a `Breach` of its own, its `Soffit` the underside of
  what stands over it; `BreachWidth` counts it with the gap.
- `BestDrop(pos, reach)`: over works passing water (in 2 m above, out 2 m below) and stretches of river (between two of
  its points, the drawn levels' difference, the water passing both now: across a dam still filling, only what comes
  over it), the pair whose ends both lie within `reach` of `pos` with the most `Power`. `Kind` is `None` down a river.
- `Head(upper, lower)`: `Sample`'s `SurfaceHeight` at `upper` less at `lower` (a dam's pool, a river's rise and the
  tide in); NaN unless both points lie in water Straumr knows.
- `SetGateOpening(doorView, opening)` writes a float, `straumr.gate`, on the door piece's ZDO (saved with the piece),
  in 5 % steps; NaN or below 0 sets it to -1, and the door's own state counts again. It returns false for anything
  that isn't a door piece and inside a ward the local player may not use (Valheim's door checks the same). Your game
  takes the piece's ownership to write it (a mod writing every frame fights other players for it); the server takes
  the setting once two looks agree (2-4 s). A lever mod can just as well open and shut doors with Valheim's own door
  call: Straumr follows the door's state.
- Read only: none of these stores or sends anything. Main thread; `Work` is a few dozen lookups, `BestDrop` a few
  hundred: now and then, not every frame. Nothing throws. Straumr makes no pump, wheel or lever: those are your pieces.

## Drift, fords and water power

**Drift (since 0.1.3).** `DriftPath(from, seconds, into, carry)` follows the current from a point a second at a time,
each second's step taken from the current half way along it, and adds a point a second (the start first) at the water's
level. `carry` is how much of the current carries the thing: 1 for the water itself, 1.18 for a floating item as
Straumr carries them (`[Currents] Items` at 1), 1.1 for a swimmer, `[Currents] Boats` for a boat with someone aboard.
- The water is held as it runs at the moment you ask: the tide doesn't turn and the rivers don't rise or fall on the
  way. Over a minute or two that is what happens; for longer on tidal water, ask again as it goes.
- It ends where the water does: on land, where it lies still (under 1 cm/s), and where Straumr doesn't know the water
  (past about 160 m from the local player: see Limits). At most 600 s (601 points); returns how many points it added.
- Waves, wind drift on the item itself and anything it bumps into are left out: it's the current's own path.
- Main thread; two looks at the current per second of drift (about 1,200 for ten minutes): now and then, not every frame.

**Fords.** There is no wading call: swimming is the game's mechanic, and Straumr's push on waders is its own on top,
and both go by two numbers `Sample` already gives, `Depth` (all of the still water, a river's rise included) and
`Speed`. Valheim starts a character swimming once the water over its feet is deeper than its `m_swimDepth` less 0.4
(`Character.InLiquidSwimDepth`): read that from the player rather than assuming a number. The game measures that
against the surface with its waves, so `Depth` a little under the line can still lift a wader off. Straumr's own push
on waders (`[Currents] Wading`) starts where (`Depth` - 0.25) x `Speed`² passes 0.5 m³/s² and never takes more than
half the slowest walking pace: given for reference, a design choice that may change. The atlas's Fords layer goes by the
same numbers.

**Water power.** `KineticPowerDensity(pos)` (since 0.1.1) is ½ρv³ of `Velocity` there, in W/m²: what a wheel's blades
could at most take from each square metre facing the current. The atlas's Water power layer marks the strongest per
area from the rivers' mean speeds at usual flow; its close-up readout gives the 4 m cell's. Mid-channel runs faster than
a river's mean (and the power goes with the cube), so sample across the channel before you place a wheel. For water
falling a head (a dam, a gate), `HydraulicPower`, `Work` and `BestDrop` (since 0.1.3, above).

## Example: a water wheel

Power from the water, rotation from the rim speed. The wheel decides its own blade area and efficiency.

```csharp
// Twice a second is plenty: the water changes over seconds.
FlowSample s = StraumrApi.Sample(wheelHub.position + Vector3.down * radius);  // where the blades dip
if (!s.InWater || s.Frozen) { targetRpm = 0f; return; }

float bladeArea  = 1.2f;   // m2 of blade in the water
float efficiency = 0.25f;  // an undershot wheel takes 20-30% of what passes it (no machine takes over 59%)
float watts = 500f * s.Speed * s.Speed * s.Speed * bladeArea * efficiency;   // = KineticPowerDensity * area * share

// Only the current across the wheel's face turns it; the rim runs at about half that, less under load.
float along   = Mathf.Abs(Vector3.Dot(s.Velocity, wheelHub.forward));
float rimSpeed = 0.45f * along;
targetRpm = rimSpeed / (2f * Mathf.PI * radius) * 60f;
```

A 1.2 m/s river on 1.2 m2 of blade gives ~260 W at 25%; a 3 m/s rapid gives ~4 kW. The rivers' speed changes with
the season (by default x1.4 in mid-spring down to x0.8 in autumn and x0.7 in winter, with Seasonality), rain (up to 30%
faster, 60% after a storm) and, on snow-fed rivers in spring, the daily snowmelt (up to 35% at its late-afternoon
peak). They stack: over spring's own speed, a storm and the melt together run a snow-fed river about x2.2 (x3 its
summer speed) until `[Currents] MaxSpeed` caps it, and the power goes with the cube.

## Example: fishing spots (eddies and seams)

Fish hold where slack water meets fast: eddies behind rocks, bars, islands and points, and the seam between two
currents. A bar's eddy comes and goes with the tide and the river's level, so sample when you need it, don't cache it.

```csharp
var best = new List<Vector3>();
for (float dx = -40f; dx <= 40f; dx += 8f)
for (float dz = -40f; dz <= 40f; dz += 8f)
{
    Vector3 p = player.position + new Vector3(dx, 0f, dz);
    FlowSample s = StraumrApi.Sample(p);
    if (!s.InWater || s.Confidence < 0.5f) continue;
    if (s.Kind == WaterKind.Eddy) { best.Add(p); continue; }

    // A seam: the current changes sharply across the flow over a few metres.
    if (s.Speed < 0.2f) continue;
    Vector3 across = Vector3.Cross(Vector3.up, s.Velocity / s.Speed) * 3f;
    float shear = (StraumrApi.Velocity(p + across) - StraumrApi.Velocity(p - across)).magnitude / 6f;  // 1/s
    if (shear > 0.15f && s.Depth > 1.5f) best.Add(p);
}
```

Deep slow water beside a `River` (`Depth > 3`, `Speed < 0.4`) is another classic holding spot. Straumr places a few
fish there itself on servers that run it; your mod can weight its own spawns the same way.

## Example: a boat following a route

```csharp
var samples = new List<FlowSample>();
StraumrApi.SampleAlong(from, to, 25f, samples);
Vector3 dir = (to - from); dir.y = 0f; dir.Normalize();
float help = 0f; int known = 0;
foreach (FlowSample s in samples)
{
    if (!s.InWater) continue;              // land, or too far from the player for Straumr to know
    known++;
    help += Vector3.Dot(s.Velocity, dir);  // + carries the boat along, - holds it back
    if (s.Kind == WaterKind.Whirlpool || s.Kind == WaterKind.Rapids) { /* steer round, or slow down */ }
}
float meanAlong = known > 0 ? help / known : 0f;   // m/s
```

Straumr carries a boat only while someone is aboard, at 0.6x the current by default (`[Currents] Boats`), and a boat
under sail or oar keeps as headway against it the smaller of `[Currents] BoatMinSpeed` (3 m/s by default) and its own
speed through the water. Plan with `Velocity`, not with those settings: they are the server's and can change.

## Building a current chart (or anything map-wide)

Straumr has no map overlays, skills or chart sharing of its own (it depends on nothing but BepInEx); a chart mod can
build them on this API. The parts it needs:
- `Sample` near the player, as they sail: rivers and lakes are only solved round players, so record what was seen.
- `FloodStream` and `SeaCurrent` anywhere once the sea is solved: record a strait as "tidal, this axis, this strength
  at springs (x (1 + SpringNeap))", and work out flooding or ebbing live with `TidePhase` and `TideTurned`. Whether to
  chart water the player never sailed is the chart mod's choice; Straumr doesn't decide that.
- `DownstreamPath` and `RiverMouth` for the shape of a river network.

## Events

```csharp
StraumrApi.TideTurned        += t => Log(t.HighWater ? "high water" : "low water");   // every ~7.5 real minutes
StraumrApi.SpringNeapChanged += s => Log(s.Spring ? "spring tides" : s.Neap ? "neap tides" : "middling tides");
StraumrApi.FloodChanged      += f => Log($"rivers {f.Previous} -> {f.Stage} (+{f.Swell:P0})");
StraumrApi.FreezeChanged     += frozen => Log(frozen ? "the water froze" : "the thaw");
StraumrApi.LevelsChanged     += l => RefreshDockHeights();   // since 0.1.3: the river surface round you changed
StraumrApi.CofferdamChanged  += d => Log($"cofferdam {d.Id}: {(d.Gone ? "full again" : d.Sealed ? "closed" : "breached")}"); // since 0.1.3
StraumrApi.DamChanged        += d => Log($"dam {d.Id}: {(d.Gone ? "gone" : d.Breached ? "breached" : d.Spilling ? "spilling" : "closed")}");       // since 0.1.3 (a gate set, too)
// Unsubscribe in OnDestroy.
```

- Raised on the main thread, at most once a second, only for large-scale changes; never per position.
- `TideTurned` is the open sea's turn. Behind fast straits the tide turns later: use `MinutesToTideTurn(pos)` there.
- `FloodChanged` follows a mid-sized (about 30 m wide) river fed by mountain snow: `Rising` once it runs 10% over the
  season's usual and climbing, `Peak` when the rise levels off, `Falling` as it drains, `Normal` under 5%. A stage
  must hold 20 s before it's told. Creeks rise and fall within minutes, great rivers over half an hour and longer
  (`Creeks`, `GreatRivers` in the args). In spring the snowmelt pulses daily (lowest at dawn, peaking late afternoon),
  so snow-fed rivers flood a little every spring day.
- `LevelsChanged` (since 0.1.3): a new level step or new settings worked into the river surface round the local
  player, or the rivers finished rising or settling (RiverLevels on or off, a freeze). `Raised` says whether they stand
  above the sea now. Read `SurfaceHeight` / `RiverRise` again wherever you keep them. With `[Floods]` on,
  the levels step every in-game hour, so while the water moves (rain, a flood, spring's melt) it fires at most once
  every 75 real s; with floods off, at most once a weather spell, as before.
- Events start about a minute after a world loads; read the current state any time (`TidePhase`, `SpringTides`,
  `StraumrApi.Flood`, `IsFrozen`).

## Limits, honestly

- **Where Straumr knows the water.** The field is built on each player's game around the local player: within about
  160 m of them (further where they've been lately). A boat 1 km away gets `InWater == false`. A dedicated server runs
  no water at all: there, only the tide's clock works (`TidePhase`, `MinutesToTideTurn`, spring and neap tides,
  `TideTurned`), and as the open sea's tide everywhere, with no lag behind straits, since the server doesn't solve the
  sea. `FloodChanged` and `FreezeChanged` never fire there: the rivers' stage (floods, the freeze) is worked out on
  players' games.
- **Other mods' water.** Rivers and Lakes' streams are its own and aren't sampled; its lakes are, where Straumr moves
  them (a stream pouring in, wind drift on a big lake), with `Source == OtherWater` and `Kind == Lake`. Away from Straumr's own
  water its still or frozen lakes read as no water; over Straumr's water they read as water with no current. For those, `y` matters: the point must be within 3 m of the lake's surface.
- **It's a model.** Ideal flow, hydraulic geometry and a solved sea, tuned to feel right and to agree with real rivers'
  orders of magnitude, not a CFD solution. Expect the right direction and speed to within tens of percent;
  `Confidence` says where it's rougher.
- **Cost.** `Sample` is a few dozen lookups in tables Straumr has already built: made for sampling now and then or
  across an area, not for every vertex of a mesh each frame. To see what it costs on your machine, set Straumr's
  `[Diagnostics] PerfCounters = true` and type `straumr perf` in the console. `DownstreamPath` and `RiverMouth` walk a
  river's line: now and then, not every frame.
- **Units.** Metres, seconds, m/s, W/m2. Tide rates are tiny (mm/s); by default the open sea's tide rises and falls
  ±0.6 m on an average tide (Straumr's `TideRange`), up to ±0.8 m at spring tides, and a full storm adds up to 0.3 m on top
  (`StormSurge`).
