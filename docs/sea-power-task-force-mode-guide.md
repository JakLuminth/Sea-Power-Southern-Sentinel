# Sea Power Task Force Mode Guide Notes

Source reference: https://steamcommunity.com/sharedfiles/filedetails/?id=3756769210

Source author: Ian

Source status: updated July 31, 2025. This is a patch-dependent community
reference; the guide is based on the current Task Force Mode implementation and
the stock Pacific Strike campaign, but exact behavior can change with game
updates. The source author also corrected the mod-root path in the comments:
mod folders should be direct children of `StreamingAssets`, not nested under
`StreamingAssets/user/` when using the mod manager.

This file is a Copilot-friendly working summary of the referenced Sea Power Task Force Mode guide for use in this workspace. It captures the practical campaign-authoring instructions in repo-local markdown so it can be used consistently during editing.

## Scope and Intent

- Use files under `StreamingAssets/original/...` as read-only examples.
- Build custom mod work under `StreamingAssets/<Your Mod Name>/...` and enable
  the mod in the mod manager. The default editor/user mission location may still
  be `StreamingAssets/user/missions`; do not confuse that location with the
  current mod-manager layout for campaigns.
- A Task Force Mode campaign is a persistent-force campaign where the player spends points, buys units, carries survivors forward, and manages repairs/rearming across the mission timeline.

## Recommended Campaign Layout

A Task Force Mode campaign typically uses:
- `campaign.ini`
- `commander_settings.ini`
- `player_task_force_roster.ini`
- `unit_roster_descriptions_<language>.ini` (optional but recommended — provides the per-unit descriptions shown in the Unit Catalog tab of `campaign_rules_<language>.xml`)
- `missions/*.ini`
- optional `art/`, ribbons, medals, and event XML assets

The guide explicitly describes these campaign components:
- `campaign.ini` defines timeline, mission list, Task Force Mode rules, and rewards
- `player_task_force_roster.ini` defines what the player is allowed to buy
- `commander_settings.ini` defines commander nations, names, ranks, discounts, and awards
- `missions/*.ini` contains the actual scenario files launched by the campaign

> **Maintenance rule:** `unit_roster_descriptions_<language>.ini` must be kept in sync with `player_task_force_roster.ini`. Whenever a unit is added to or removed from any `[Allowed*]` section of the roster file, add or remove the corresponding entry (keyed by the same unit ID) in every `unit_roster_descriptions_*.ini` that the campaign ships. There is no global fallback description database — the only source the engine has is the campaign-local file. What the engine renders for a missing key (blank, raw key name, or nothing) is unconfirmed from stock data alone and should be verified in-game if it matters.

## Simplest Working Task Force Mode Pattern

For a straightforward campaign:
1. Create a mod folder directly under `StreamingAssets/<Your Mod Name>/`.
2. Add a normal `campaign.ini`.
3. Add `[TaskForceMode] Enabled=True` to `campaign.ini`.
4. Add `commander_settings.ini`.
5. Add `player_task_force_roster.ini`.
6. Use `Generated` for normal task-force missions.
7. In each generated mission file, define `Taskforce1Vessel1` and use it as the only section with `TaskForceModeAnchor=True`.
8. Configure rearm, repair, rewards, roster restrictions, and mission metadata in each `[MissionN]` block.

## Practical Gotchas and Validation Checklist

Use this checklist before testing a new or heavily-edited Task Force Mode campaign.

- **`[File]` section is required in `campaign.ini`**
  - Include:
    - `[File]`
    - `Base=campaigns/<campaign-folder>/campaign.ini`

- **Combat mission entries should use `Type=Mission`**
  - `Type=CombatMission` is not the stock Task Force Mode pattern.

- **Do not mix linear commander setup with Task Force Mode commander setup**
  - For pure Task Force Mode campaigns using `CommanderSettingsFile`, avoid adding `[PersistentData]` and `[CommanderName]` in `campaign.ini`.
  - Task Force Mode manages commander/task-force setup through its own flow.
  - Also remove any linear commander-name event from the mission chain (for example `00_commander_name.xml`); leaving it connected can still prompt linear name input or break progression.

- **Use explicit roster entry values in `player_task_force_roster.ini`**
  - Ships/submarines should use explicit variant IDs like `Variant1,Variant2,...`.
  - Aircraft/helicopters should use explicit squadron IDs like `Squadron1,Squadron2,...`.
  - Avoid shorthand tokens such as `*_variants` or `*_squadrons` in roster value fields.
  - If you want automatic capability-based pricing, omit `|point_cost` overrides entirely (for example: `wp_rkr_kirov=Variant1` instead of `wp_rkr_kirov=Variant1|55`).

- **Path style in campaign files**
  - Use `campaigns/<campaign-folder>/...` for `Base`, `MissionFile`, `AssetsPath_*`, and `FilePath_*`, relative to the mod root.
  - Do not prepend `user/` in these references.

- **Loadout progression uses mission unlock fields**
  - `LockedShipLoadoutVariants` defines the locked loadout pool.
  - Unlock entries during progression with `TaskForceModeLoadoutsToUnlock=<LoadoutName>` in mission blocks.
  - `InitialUnlockedLoadouts` in difficulty sections controls what is available at campaign start per difficulty.

- **Recommended mission UI metadata fields**
  - For each combat mission, include:
    - `MissionSequenceName_<lang>`
    - `MapShortName_<lang>`
    - `MissionIntro_<lang>`
  - Add `MissionResupplyRules_<lang>` on missions that grant rearm/repair access.

- **Commander name pool structure should match stock pattern**
  - In `[CommanderSettings]`: `CommanderNamePool<Nation>=Names_<Nation>`
  - Then define `[Names_<Nation>]` with `Names=` and `Surnames=` lists.

## Task Force Builder and Campaign Economy

Task Force Builder is where the player spends campaign points to build and maintain the persistent force.

Key ideas:
- The player buys ships, submarines, aircraft, and helicopters with campaign points.
- Surviving units carry forward between missions.
- Remaining points are saved for repairs, rearming, and future purchases.
- More capable units cost more points, forcing tradeoffs.

### Unit Point Costs

- Base-game units already have point costs in their own unit `.ini` files.
- A modded unit that should appear in Builder needs a `[TaskForce]` section with `TaskForceCost=...`.

Example concept from the guide:
- `[TaskForce] TaskForceCost=27`
- Optional loadout costs can also exist, such as `LoadoutCost_Late=10`.

### Campaign-Specific Price Overrides

A campaign can override unit prices in `player_task_force_roster.ini` without editing the unit file.

Basic roster patterns described by the guide:
- Ships/submarines: `unit_type=variants|point_cost`
- Aircraft/helicopters: `unit_type=squadrons|point_cost`

If `|point_cost` is omitted in the roster, Builder uses the unit's default `TaskForceCost`.

### Loadout Point Costs

- Ships and submarines can have loadout-specific campaign costs.
- These can be defined in unit files or overridden in the roster file.
- Aircraft loadouts are described as being included with the aircraft purchase rather than purchased separately.

## Campaign-Wide Task Force Rules

These fields belong in `[TaskForceMode]` and define campaign-wide behavior.

Important guide-described fields:
- `Enabled` - turns Task Force Mode on for the campaign
- `TaskForceRequireFlagship` - losing the flagship can fail a mission
- `DefaultTaskForceName` - starting name for the player's force
- `TaskForceNameOptions` - optional random task force names
- `CommanderSettingsFile` - usually `commander_settings.ini`
- `RosterFile` - usually `player_task_force_roster.ini`
- `TaskForceDifficultyPresets` - list of difficulty preset IDs
- `DefaultTaskForceDifficultyPreset` - default selected difficulty
- `StartingPoints` - fallback starting points if presets are not used
- `PointCap` - fallback point cap if presets are not used
- `ShipIncludesAirwing` - whether ships come with their normal airwing
- `PurchaseLoadouts` - whether some ship/submarine loadouts cost points
- `BasicShipLoadoutVariants` - loadouts available at campaign start
- `LockedShipLoadoutVariants` - loadouts that exist but start locked
- `CSARPointModifier` - survivors required per campaign point
- `CrewSkillInitial` - starting skill for new units
- `CrewSkillThresholds` - optional progression thresholds such as veteran/elite promotion
- `UnitDecommissionPointReturnModifier` - refund fraction on decommission
- `UnitDismissPointReturnModifier` - refund fraction on dismiss
- `DamageToAllowRepair` - repairable damage levels
- `DamageToDisallowRepair` - non-repairable damage levels
- `RepairPointsCost` - repair cost by damage level
- `CurrentTaskForce` - optional seeded starting force; not recommended for first-time authoring unless specifically needed

## Difficulty Presets

The guide describes difficulty presets as optional but recommended campaign-start variants.

They are defined using sections like:
- `[TaskForceModeDifficulty_Easy]`
- `[TaskForceModeDifficulty_Moderate]`
- `[TaskForceModeDifficulty_Difficult]`

Typical uses include varying:
- starting points
- point cap
- repair cost modifier
- unlocked loadouts
- refund modifiers
- starting crew skill

## Mission-Level Rules in `campaign.ini`

Each `[MissionN]` block can define mission-specific Task Force Mode behavior.

The guide highlights these categories of mission-level settings:
- which mission file launches
- whether the player's task force, airwing, or submarines are involved
- the visible threat profile shown to the player
- whether Builder is enabled before that mission
- mission rewards and completion points
- roster restrictions for that mission
- optional air tasking or airbase prep settings
- mission generation mode

Frequently used mission-level fields described by the guide include:
- `MissionFile`
- `TaskForceModeMissionGenerationType`
- `TaskForceModeIncludesTaskForce`
- `TaskForceModeIncludesAirwing`
- `TaskForceModeIncludesSubmarine`
- `TaskForceModeThreatProfileShip`
- `TaskForceModeThreatProfileAir`
- `TaskForceModeThreatProfileSub`
- `TaskForceModeThreatProfileLand`
- `TaskForceModeRearm`
- `TaskForceModeRepair`
- `TaskForceModeEnableTaskForceBuilder`
- `TaskForceModeAllowedRosterUnits`
- `TaskForceModeCompletionPoints`
- `TaskForceModeCompletionCapPoints`
- `TaskForceModeRibbonAwards`
- optional mission notes, builder situation text, mission intro text, and resupply rules

Additional current mission-level fields include:
- `TaskForceModeAirTaskingAvailable`
- `TaskForceModeAirTaskingFlight<N>`
- `TaskForceModeAirbasePrepAvailable`
- `TaskForceModeAirbasePrepReadySlots`
- `TaskForceModeAirbasePrepInProgressSlots`
- `TaskForceModeRequiredUnitType`
- `TaskForceModeMaxUnits`
- `TaskForceModeCompletionRewardedUnits`
- `TaskForceModeCommanderIncreaseRank`
- `TaskForceModeDebriefNoticeTitle_<lang>` and `TaskForceModeDebriefNoticeText_<lang>`
- `TaskForceModeFinalMission`
- `TaskForceModeServiceRecordOnboarding`

### Conditional Rearm

Rearm can be gated by campaign variables instead of being unconditionally
available:
- `TaskForceModeRearmByVariableAND=VariableName,Check|...`
- `TaskForceModeRearmByVariableOR=VariableName,Check|...`

The source lists checks such as `IsTrue`, `IsFalse`, `NumberGreaterThan`,
`NumberLessThan`, and `StringEqual`. Use the simple `TaskForceModeRearm=True`
pattern first, and verify variable-driven behavior against the current build.

### Threat Profile Format (important)

Use stock-format threat profile values in mission blocks:

- `TaskForceModeThreatProfileShip=True,<level>`
- `TaskForceModeThreatProfileAir=True,<level>`
- `TaskForceModeThreatProfileSub=True,<level>`
- `TaskForceModeThreatProfileLand=False` (or `True,<level>` if land threats are used)

Where `<level>` is an integer severity shown to the player in mission info (scale: 1-5 in stock campaigns).

Practical guidance:
- Prefer this `True,<level>` format for compatibility with current Task Force Mode behavior.
- Keep threat profile values aligned with actual mission composition so mission cards are accurate.
- If legacy text labels such as `Low/Medium/High` are used in older content, verify in-game before adopting them in new campaigns.

## Mission Generation Types

### Generated
Use `TaskForceModeMissionGenerationType=Generated` for normal task-force missions.

Meaning:
- Place one player vessel in the mission file.
- Mark `Taskforce1Vessel1` with `TaskForceModeAnchor=True`. The current source
  warns that the anchor must be set only on that section; if no anchor is marked,
  the generator uses `Taskforce1Vessel1`.
- At mission launch, the game replaces that anchor with the player's saved task force and places the rest of the force around it.

Practical rule:
- Generated missions should normally contain one authored player anchor vessel, not a fully slotted set of player replacement ships.

### Replaced
Use `TaskForceModeMissionGenerationType=Replaced` only when exact player-ship slot placement is needed.

Meaning:
- The player's surface ships fill specific authored player ship slots using
  `TaskForceModeReplacedUnitIndex=1`, `2`, `3`, and so on.
- Use this when exact positions, formations, identities, or trigger-sensitive slot placement matter.

Practical rule:
- Use `Taskforce1Vessel1` as the first slot and set `TaskForceModeAnchor=True`
  only on that section; make sure triggers still reference section names that
  exist after replacement.

### Placeholder Units

`TaskForceModePlaceholderUnit=True` marks a player vessel as an editor-visible
placeholder that should not remain as that exact ship when the generated mission
launches. Use this when a mission needs a visible slot while it is being authored.

### Blank / Empty
Leave `TaskForceModeMissionGenerationType` blank when the mission should launch as-authored without using the player's persistent task force.

Meaning:
- Best for detached submarine, detached air, or other special-case missions that should not pull in the saved task force.

Practical rule:
- Detached side missions should usually leave the field blank unless there is a specific reason to involve the persistent force.

## Air Tasking

Air Tasking creates pre-mission flight rows that fill authored aircraft or
helicopter slots. Enable it in the relevant `[MissionN]` section with:

```ini
TaskForceModeAirTaskingAvailable=True
TaskForceModeAirTaskingFlight1=CAP|CAP|Fighter|2|AirToAir/AirToAirLongRange
```

The current five-part row format is:
`RoleId|DisplayName|AllowedUnitRoles|SlotCount|AllowedLoadouts`.
The older four-part format omits `AllowedUnitRoles`. In the mission file, pair
each eligible slot with `TaskForceModeAirTaskingRole=<RoleId>` and
`TaskForceModeAirTaskingSlot=<slot number>`. Use unique `Flight<N>` keys; only
unassigned aircraft are eligible, and ship-assigned helicopters are not used for
Air Tasking. If fewer aircraft are available than requested, the available ones
are launched and unused slots are removed.

## Airbase Preparation

Enable pre-mission preparation for land-based aircraft with:

```ini
TaskForceModeAirbasePrepAvailable=True
TaskForceModeAirbasePrepReadySlots=2
TaskForceModeAirbasePrepInProgressSlots=4
```

The mission must contain a player land unit whose type includes `airbase` or
`airfield`; otherwise Airbase Prep has no suitable unit to operate on.

## JoinTaskForce

`JoinTaskForce=True` adds a hand-placed unit to the player's persistent force at
debrief if it survives. The current source supports normal vessel, submarine,
aircraft, and helicopter sections, and recommends adding a `CampaignTag` to the
unit. Joined units cost zero points and retain their skill and selected loadout.
Joined aircraft and helicopters become unassigned roster units; they are not
automatically assigned to a ship, airbase, or Air Tasking slot.

Do not use `JoinTaskForce` for land units, airbases, weapons, enemy units, or
custom non-unit sections. The mission must reach debrief for the join to be
processed.

## Generated Mission Authoring Pattern

For generated missions, the guide's working pattern is:
- `PlayerTaskforce=Taskforce1`
- one authored player vessel in the mission file as `Taskforce1Vessel1`
- `TaskForceModeAnchor=True` only on `Taskforce1Vessel1` when an explicit anchor is used
- the anchor's position, heading, speed, and waypoints define the generated task force's starting geometry

In plain terms:
- the mission file contributes the anchor ship's placement data
- the saved campaign task force is then built around that anchor at launch

## Replaced Mission Authoring Pattern

Use replaced missions only when the mission truly needs specific authored player ship slots.

This is the higher-control option and should be treated as an exception rather than the default campaign pattern.

## Detached Mission Pattern

For detached missions that should not use the persistent task force:
- leave `TaskForceModeMissionGenerationType` blank
- structure the mission file as a self-contained scenario
- use campaign timeline placement only to expose the mission as an event or optional branch

## Roster File Guidance

The roster file controls what the player can buy.

The guide describes these sections:
- `[AllowedVessels]`
- `[AllowedSubmarines]`
- `[AllowedAircraft]`
- `[AllowedHelicopters]`
- optional `[LoadoutPrices]`

Practical rules:
- every allowed unit should have a valid campaign point cost either in the unit file or overridden in the roster file
- roster overrides are for campaign-specific balance, not mandatory for base-game units
- `TaskForceModeAllowedRosterUnits` in a mission can narrow availability, but it cannot make unavailable units appear if they are not in the roster

## Commander Settings Guidance

The guide describes `commander_settings.ini` as controlling:
- commander nations
- default commander names
- name pools
- rank lists
- same-nation discount
- navy names and emblems
- ribbons, medals, and citations

Important `CommanderSettings` ideas:
- `CommanderNations`
- `CommanderDefaultNation`
- `CommanderNameDefault<Nation>`
- `CommanderNamePool<Nation>`
- `CommanderStartingRankLevel`
- `SameNationUnitDiscount`
- `NavyName<Nation>`
- `NavyEmblem<Nation>`

### Officer Ranks

The guide describes rank entries as:
- `DisplayName,Abbreviation,Grade,RankLevel,ImagePath`

Rank level is what promotion logic uses.

## Ribbons, Medals, and Citations

The guide describes award setup using:
- `[TaskForceRibbons]`
- per-ribbon sections like `[Ribbon_<id>]`

Common ribbon-related fields listed by the guide include:
- `Type`
- `Precedence`
- `Name_en`
- `ImagePath`
- `MedalImagePath`
- `Devices`
- `StripeColors`
- `StripeWidths`
- `ReferenceSource`
- `SourceUrl`
- `CitationIssuingBody_en`
- `CitationIssuance_en`
- `CitationAwardName_en`
- `CitationAuthority_en`
- `CitationRecipient_en`
- `CitationDate_en`
- `CitationSignatureName_en`
- `CitationSignatureTitle_en`
- `CitationSealImagePath`
- `CitationText_en`
- nation-specific citation body fields like `CitationTextUS_en` or `CitationTextAustralia_en`

Useful citation tokens mentioned by the guide:
- `{CommanderRank}`
- `{CommanderName}`
- `{CommanderLastName}`
- `{TaskForceName}`
- `{Self}`

Mission-level award granting is done with `TaskForceModeRibbonAwards` in `campaign.ini`.

## First-Campaign Checklist From the Guide

Campaign folder:
- Put the mod folder directly under `StreamingAssets/<Your Mod Name>/...` and enable it in the mod manager.
- Do not edit `StreamingAssets/original/...`
- Create `campaigns/<your-campaign>/campaign.ini`
- Create `player_task_force_roster.ini`
- Create `commander_settings.ini`

Campaign rules:
- Add `[TaskForceMode] Enabled=True`
- Add starting points and point cap
- Add at least one difficulty preset
- Add a roster file reference
- Add a commander settings file reference

Roster:
- Add at least a few ships to `[AllowedVessels]`
- Ensure every allowed unit has a valid Task Force cost
- Add aircraft/helicopters if the campaign uses them

Each generated mission:
- Add `TaskForceModeMissionGenerationType=Generated`
- Add `TaskForceModeIncludesTaskForce=True`
- Decide whether airwing or submarines are included
- Add `TaskForceModeRearm` and `TaskForceModeRepair`
- Add `TaskForceModeEnableTaskForceBuilder`
- Add completion points and cap points
- Add ribbon awards if desired
- In the mission file, use `Taskforce1Vessel1` as the anchor section; set `TaskForceModeAnchor=True` only there when explicitly marking the anchor.

## Guidance For This Workspace

Use these defaults unless there is a strong mission-specific reason not to:
- Main task-force missions: `Generated`
- Detached submarine missions: blank generation type
- Detached air strike missions: blank generation type
- Use `Replaced` only when exact authored slot behavior is truly required
- Prefer `Taskforce1Vessel1` as the single player anchor section in generated mission INIs
- Treat mission-level vessel caps as optional scenario design tools, not automatic defaults
- Keep real-world geography aligned with the globe-based map when using named locations

## Why This Summary Exists

The original web guide is not in a format ideal for repo instructions. This markdown file captures the practical authoring instructions in a reusable form for ongoing Sea Power campaign work in this workspace.
