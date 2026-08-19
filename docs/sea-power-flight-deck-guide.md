# Sea Power – Flight Deck / Airbase Mission Authoring Guide

> Source: https://steamcommunity.com/app/1286220/discussions/3/769678769228297148/
> Source author: Ian
> Published: October 31, 2025; last edited November 1, 2025
>
> This is a patch-dependent community reference. The source post states that
> these mission-authoring controls require direct `.ini` editing until they are
> exposed in the Mission Editor. Verify behavior against the current game build
> when exact engine behavior matters. The author-confirmed follow-up replies
> through November 28, 2025 are also reflected below.

---

## Overview

Two mechanics apply to flight decks on both **carriers and airbases**:
1. **Aircraft Prep/Cooldown Times** — delay before launch and after recovery
2. **Airbase Ammunition / Ordnance Stores** — finite advanced weapon stocks

The global **Flight Deck Timings** setting in the game Options controls whether
prep and cooldown times are enabled and whether they use the full or reduced
duration. Mission `.ini` files configure initial aircraft readiness and
accountable ordnance stores for individual airbases or carriers.

---

## Aircraft Prep and Cooldown Times

Controlled by the **Flight Deck Timings** option:
- **Realistic** — full prep/cooldown times
- **Casual** — 50% reduced times
- **Arcade** — feature disabled (old behavior)

**Prep time** = delay before aircraft can launch.  
**Cooldown time** = refuel/service delay after recovery.

Times vary by loadout. Check `StreamingAssets/original/aircraft/<type>.ini` or mouse over a loadout in the Flight Deck UI to see "Preparation Time".

### Notes
- Ships with ASW helos always start with one helo ready to launch immediately
  with the ASW loadout equipped.
- This change affects **all missions** with carriers or airbases where the designer expected instant launch.
- Also affects AI carriers or airbases assigned to CivilianRoutes — aircraft must
  be prepped before the CivRoute fires, or the route will begin preparation and
  the launch may be delayed.

---

## FlightDeck_ReadyUpTasks Syntax

Add these keys to the airbase or carrier section of the mission `.ini`:

```ini
FlightDeck_ReadyUpTasks=3
FlightDeck_ReadyUpTask1=usn_a-6e,Squadron3,AntiShipHeavy,4,1200
FlightDeck_ReadyUpTask2=usn_f-14a,Squadron9,AirToAirLongRange,2,600
FlightDeck_ReadyUpTask3=usn_ea-6b,Squadron6,SEAD,1,800
```

**Per-task format:**
```
FlightDeck_ReadyUpTaskN=<aircraft_type>,<squadron>,<loadout>,<count>,<prep_time_seconds>
```

### Rules
- Aircraft type, squadron, loadout, and count must **match** what the airbase actually has.
- Aircraft **must have enough ordnance** to be loaded (see Accountable Ammunition below).

---

## Airbase Ammunition / Ordnance Stores

Aircraft cannot launch if the airbase is out of ammo for that loadout.  
**No partial loadouts** — if a loadout needs 4 Harpoons and only 2 are available, it cannot launch.

### Advanced Weaponry Categories (limited by default)

Advanced weaponry is tracked at the category level rather than separately for
each weapon. Only these categories are tracked as "accountable":

| Category | Weapons |
|---|---|
| `AirTorpedo` | All air-dropped torpedoes (Mk 46, UMGT-1, etc.) |
| `Harpoon` | AGM-84A, AGM-84D |
| `Phoenix` | AIM-54A |
| `SovietAdvancedASM` | AS-4 Kitchen, AS-6 Kingfish |
| `Ovod` | AS-13 Kingbolt |

> Categories are defined in `StreamingAssets\original\language_xx\ammunition_names.ini`

---

## Ordnance Configuration Syntax

The source post documents manual mission-file provisioning for advanced
weaponry only. Standard ammunition remains governed by the game's normal
airbase or carrier stores.

```ini
FlightDeck_AmmoCapacity=300000/300000
FlightDeck_NumberOfAccountableAmmunitionCategories=3
FlightDeck_AccountableAmmunitionCategory_1=AirTorpedo,16/100
FlightDeck_AccountableAmmunitionCategory_2=Harpoon,20/100
FlightDeck_AccountableAmmunitionCategory_3=Phoenix,16/100
```

**Format:**
```
FlightDeck_AmmoCapacity=<current>/<max>
FlightDeck_AccountableAmmunitionCategory_N=<Category>,<available>/<max_magazine>
```

### Custom Air Group Note
If using `CustomAirGroup=True`, check **Auto Generate Ordnance** in the Mission
Editor (or set the stores manually). Auto-generation replaces the default
stores with stores generated for the custom air group.  
> Advanced weaponry loadouts will only receive **one loadout's worth** of ordnance when auto-generated.

## Unit-Level Flight Deck Capacity

`GroundCrewCount` and `DeckParkSlots` are properties of the carrier or airbase
unit definition. They must be supplied or corrected in the unit `.ini`; the
source author states that they cannot be changed through the Mission Editor or
mission `.ini`.

Modded units that omit these values may fall back to insufficient defaults and
can disrupt older missions.

## Ammunition Recovery

Unused aircraft ammunition is returned to the airbase or carrier pool after the
aircraft completes its return to base. Only ammunition that was not fired is
refunded.

---

## Source Example (Carrier with Custom Air Group)

> **Ammunition value caveat:** The source post uses
> `FlightDeck_AccountableAmmunitionCategory_2=Harpoon,120/100` while describing
> the format as `<available>/<max_magazine>` and stating that 120 Harpoons are
> stored. This is internally inconsistent. The value is retained below for
> source traceability, but it must be verified against the current game build
> before being copied into a mission.

```ini
[Taskforce1Vessel1]
Type=usn_cvn_nimitz
VariantReference=Variant2
StationRole=Core
IsValuableUnit=True
CrewSkill=Seasoned
RelativePositionInNM=-326.12,0,-219.51
Telegraph=2
Heading=52
Waypoints=-18.2874,0,21.41593
CustomAirGroup=True
usn_a-6e=Squadron4,12
usn_a-7e=Squadron5,8
usn_e-2c=Squadron4,1
usn_ea-6b=Squadron3,2
usn_f-14a=Squadron11,6|Squadron12,8
usn_s-3a=Squadron4,8
usn_sh-3h=Squadron1,2
FlightDeck_ReadyUpTasks=7
FlightDeck_ReadyUpTask1=usn_f-14a,Squadron12,AirToAirLongRange,2,0
FlightDeck_ReadyUpTask2=usn_ea-6b,Squadron3,EW,1,0
FlightDeck_ReadyUpTask3=usn_e-2c,Squadron4,AEW,1,157
FlightDeck_ReadyUpTask4=usn_f-14a,Squadron11,AirToAirIntercept,2,300
FlightDeck_ReadyUpTask5=usn_s-3a,Squadron4,AntiShip,2,900
FlightDeck_ReadyUpTask6=usn_a-6e,Squadron4,AntiShipHeavy,12,1800
FlightDeck_ReadyUpTask7=usn_a-7e,Squadron5,SEAD,8,1800
FlightDeck_AmmoCapacity=300000/300000
FlightDeck_NumberOfAccountableAmmunitionCategories=3
FlightDeck_AccountableAmmunitionCategory_1=AirTorpedo,16/100
FlightDeck_AccountableAmmunitionCategory_2=Harpoon,120/100
FlightDeck_AccountableAmmunitionCategory_3=Phoenix,72/100
```

**Result at mission start:**
- 2× F-14A CAP — ready to launch (0 sec)
- 1× EA-6B — ready to launch (0 sec)
- 1× E-2C — ready very soon (157 sec / ~2.5 min)
- 2× F-14A intercept — Alert 5 (300 sec / 5 min)
- 2× S-3A anti-ship — Alert 15 (900 sec / 15 min)
- 12× A-6E + 8× A-7E strike — Alert 30 (1800 sec / 30 min)
- 1× SH-3H ASW — always ready (ASW helo rule)
- Stores: 120 Harpoons (30 sorties), 72 Phoenix (18 sorties)
