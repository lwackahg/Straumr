# Straumr API

Straumr works out how the water moves on Riverheim (and Valheim) worlds: river currents, the sea's gyres, tides,
tidal races, whirlpools, rips, white water, floods and the winter freeze. It ships no models or textures (only its own shaders, built into the dll); this API is
how other mods (water wheels, mills, fishing, sailing AI) build on it, with no reflection.

**Design rule:** Straumr exposes physical properties (velocity, depth, power density, tide height and rate), not
bespoke mechanics (how fast your wheel turns, whether a harbour is safe, where the fish are). Those are your mod's to
decide from the physics.

Everything is in `Straumr.Api` in `Straumr.dll`: the static class `StraumrApi`, the struct `FlowSample` and a few
enums. `StraumrApi.Sample(position)` is the one primitive; the other helpers are built from the same look.

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
run time; `Version` is the mod's own version.

**Threads.** Main thread only (Unity's `Update`, `FixedUpdate`, coroutines). Nothing throws: with no world loaded,
before the field is built, or away from water Straumr knows, you get "no water" (`InWater == false`, zeros,
`Kind == None`).

**Between worlds.** At the main menu and while a world loads, everything answers "no water" and `Ready` is false
until the new world's field is built; nothing of the last world is kept to answer for the next.

## What a sample holds

| Field | Unit | Meaning |
|---|---|---|
| `InWater` | | Straumr knows the water here (see Limits for where it does) |
| `Frozen` | | iced over by Seasonality's freeze: `Velocity` and the tide fields are zero |
| `Velocity`, `Speed` | m/s | the depth-averaged current, horizontal (y = 0): exactly what carries swimmers, bent round rocks and docks. Swimmers ride at ~1.1x it, floating items 1.18x, boats 0.6x (Straumr's `[Currents] Boats`) |
| `Depth` | m | water depth |
| `Froude` | | `Speed / sqrt(9.81 * max(Depth, 0.5))`; above ~1 the flow is supercritical |
| `Whiteness` | 0..1 | how white the water runs: the same measure that tumbles swimmers (Froude-based, or a whirlpool's eye) |
| `TideHeight` | m | where the tide and any storm surge are taking the water here, above the sea's usual level (negative below); the water eases there at about 1 cm/s, so in a rising storm `SurfaceHeight` lags it |
| `TideRate` | m/s | how fast the tide is raising the water (+ rising). Tiny: a few mm/s at most. The tide's own rate: a storm's surge isn't in it |
| `SurfaceHeight` | m | the still water's surface here, absolute (no waves): the sea's level as it stands now, tide and any storm surge in, or another mod's lake's level |
| `TidalStream` | m/s | the sea's tidal stream here, + flood, - ebb, as over deep water (shallows drag it back) |
| `Openness` | 0..1 | 0 in a river's channel, 1 on wide open water |
| `Kind` | | `None, River, Lake, Sea, TidalRace, Rapids, Rip, Eddy, Whirlpool` |
| `Source` | | where the current comes from: `None, RiverChannel, Lake, OpenSea, Tidal, OtherWater, Fallback, Local` |
| `Confidence` | 0..1 | how representative the flow's *direction* is here (below) |

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
float   lvl = StraumrApi.WaterLevel(pos);               // absolute still sea-level surface here, tide and storm surge in (not Valheim's fixed 30)
float   ph  = StraumrApi.TidePhase(pos);                // -1 full ebb .. +1 full flood, later behind straits
float   min = StraumrApi.MinutesToTideTurn(pos);        // real minutes to the next slack here
Vector3 fl  = StraumrApi.FloodStream(pos);              // flood-tide stream at an average peak (ebb = -fl); anywhere once the sea is solved
Vector3 sea = StraumrApi.SeaCurrent(pos);               // the open sea's steady current, no tide/drift/surf; anywhere once solved
bool    sp  = StraumrApi.SpringTides;                   // also NeapTides, TideStrength (1 = average)
bool    ice = StraumrApi.IsFrozen(pos);
Vector3 dn  = StraumrApi.DownstreamDirection(pos);      // unit, the way the water runs along the river's line; zero off a river
Vector3 up  = StraumrApi.UpstreamDirection(pos);
int     n   = StraumrApi.SampleAlong(a, b, 25f, samples); // even samples, both ends, at most 1025
bool    ok  = StraumrApi.DownstreamPath(pos, 500, points);  // river points (~8 m apart) down the water to a lake or the sea
Vector3? m  = StraumrApi.RiverMouth(pos);               // where that path ends
```

**Which way is downstream.** Riverheim draws each river from source to mouth, but often runs several rivers through
the same water, some of them the other way. Straumr gives each body of water one way down (its shortest way to the
sea, keeping the way a river is drawn unless the other way is less than half as long), so `DownstreamDirection` and
`DownstreamPath` follow the water, which on a few stretches runs against the line as drawn.

**The water's level.** Straumr raises and lowers the sea (and everything at sea level, rivers and lakes included)
with the tide, so the water's surface is often not Valheim's fixed 30 m. Anything placed or spawned at the water's
edge should go by `WaterLevel(pos)`: the still-water surface there in metres, absolute, as the water stands now (tide
and any storm surge included; it eases toward them at about a centimetre a second), waves left out. It's 30 (Valheim's
level) with no world, with tides and surge off (`[Currents] TideRange = 0` and `StormSurge = 0`), in a freeze, before
the sea is solved and on a dedicated server. Rivers and Lakes' water above the sea isn't covered: where `Sample(pos).Source` is
`OtherWater`, that lake's level is in `Sample(pos).SurfaceHeight`; its streams aren't sampled at all.

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

Fish hold where slack water meets fast: eddies behind rocks and bends, and the seam between two currents.

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
// Unsubscribe in OnDestroy.
```

- Raised on the main thread, at most once a second, only for large-scale changes; never per position.
- `TideTurned` is the open sea's turn. Behind fast straits the tide turns later: use `MinutesToTideTurn(pos)` there.
- `FloodChanged` follows a mid-sized (about 30 m wide) river fed by mountain snow: `Rising` once it runs 10% over the
  season's usual and climbing, `Peak` when the rise levels off, `Falling` as it drains, `Normal` under 5%. A stage
  must hold 20 s before it's told. Creeks rise and fall within minutes, great rivers over half an hour and longer
  (`Creeks`, `GreatRivers` in the args). In spring the snowmelt pulses daily (lowest at dawn, peaking late afternoon),
  so snow-fed rivers flood a little every spring day.
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
