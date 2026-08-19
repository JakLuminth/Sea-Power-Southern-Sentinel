# Sea Power Copilot Instructions

Use these instructions when working with Sea Power missions, campaigns, briefings, and mod content in this repository.

## Reference Sources

- Prefer these repository guides over external sources:
  - `docs/sea-power-task-force-mode-guide.md` for Task Force Mode authoring patterns
  - `docs/sea-power-mission-creation-guide.md` for advanced mission authoring, trigger logic, briefing patterns, and direct `.ini` editing
  - `docs/sea-power-flight-deck-guide.md` for flight deck operations and related mission authoring patterns
- Use these stock game-data locations for reference:
  - `StreamingAssets/original/vessels` for vessels
  - `StreamingAssets/original/aircraft` for aircraft
  - `StreamingAssets/original/campaigns` for stock campaigns
  - `StreamingAssets/original/campaigns/<campaign>/missions` for campaign mission files
  - `StreamingAssets/original/missions` for standalone stock missions
- For advanced mission authoring, consult these read-only stock references when syntax or behavior is uncertain:
  - `StreamingAssets/original/documentation/Mission Editor. Triggers and conditions.docx` for trigger and condition rules
  - `StreamingAssets/original/missions/Demo/MissionFileInformation.ini` for mission fields, actions, and examples
  - `StreamingAssets/original/missions/Warsaw Pact/Breakthrough.ini` for a complete authored mission example
  - `StreamingAssets/original/audio/voiceover/voice.ini` for voice-message keys and custom audio patterns
  - `StreamingAssets/original/language_en/ammunition_names.ini` for accountable ammunition category names
- Use `StreamingAssets/original/campaigns/pacific-strike-task-force/` as the complete stock Task Force Mode campaign reference, including its `campaign.ini`, roster, commander settings, and mission files.
- For undocumented engine behavior, inspect the read-only compiled assemblies from the current game build under `Managed/`:
  - `Managed/Seapower-Scripts.dll` for mission, campaign, Task Force Mode, and briefing logic
  - `Managed/Noesis.NoesisGUI.dll` for XAML briefing UI behavior
- Never modify compiled assemblies. Verify inferred behavior against runtime logs or in-game testing.

## Content Layout

- Treat `StreamingAssets/original` as read-only stock game data. Never modify files there.
- Place packaged custom mods directly under `StreamingAssets/<Your Mod Name>/` and enable them in the mod manager.
- Keep `StreamingAssets/user/missions` as the default location for editor-created and user missions; do not treat it as a packaged mod folder.
- In mod development repositories, keep Workshop and runtime content exclusively under `mod/`.
- Point the direct-child `StreamingAssets/<Your Mod Name>/` symlink to `mod/`.
- Keep `.git`, `.github`, `docs`, and other development-only content outside `mod/` so folder-based Workshop uploads contain only deployable files.

## Authoring Rules

- Use the Mission Editor for initial mission layout, then edit advanced behavior directly in the mission `.ini`. Do not reopen and save a manually edited mission in the editor unless those direct edits can be recreated; editor saves may discard them. Test advanced behavior from the Scenarios menu.
- When syntax or behavior is uncertain, verify it against the read-only stock references and the current game build.
- Treat community-guide behavior as patch-dependent. Preserve each guide's source URL, author, update date, and verification caveats when maintaining the repository summaries.
- When adding triggers manually, update `[Mission] NumberOfTriggers` and keep objective completion, failure, and cancellation actions synchronized with trigger outcomes. Avoid assuming triggers firing exactly at mission start or under heavy time acceleration are reliable.
- Treat `Flight Deck Timings` as a global Options setting. Use mission `.ini` files for aircraft readiness and accountable ordnance; keep unit-level `GroundCrewCount` and `DeckParkSlots` in the carrier or airbase unit definition.
- For Sea Power mission and Task Force Mode files, do not use `Variant=*_variants` or `*_varients` shorthand tokens. Use stock-compatible explicit `VariantReference=VariantN` values, and keep roster entries explicit as `Variant1,Variant2,...`.
- Treat `campaign.ini` `MissionFile` paths as relative to the mod/package root, not relative to the directory containing `campaign.ini`.
- Treat vessel movement as waypoint-driven. Every vessel that should move must have explicit waypoints; do not attribute movement or course changes to automatic task-force AI without runtime evidence.

## Briefing Rules

- Store a left-pane briefing asset for each campaign-referenced combat mission in the adjacent `<MissionName>_briefing/BriefingText_en.xml` path when briefing text is required.
- Edit existing briefing assets in place rather than attempting to recreate files that already exist.
- Preserve briefing runtime tokens such as `{TaskForceName}`, `{FlagshipName}`, and `{SelectedUnitName}`.
- Keep briefing objective lines synchronized with the mission's `Objective_*` entries.
- After briefing changes, parse every campaign-referenced briefing XML file and verify that no referenced asset is missing and no custom briefing asset is unreferenced.

## Task Force Mode Rules

- Keep campaign-wide and per-mission Task Force Mode rules—including economy, generation type, flags, threat profiles, rewards, and availability—in `campaign.ini`.
- Use mission `.ini` files for authored placement and unit-slot behavior. In generated missions, use `Taskforce1Vessel1` as the anchor section and set `TaskForceModeAnchor=True` only there; use replacement slots, Air Tasking slots, and `JoinTaskForce=True` only when needed and follow the detailed repo-local guide for field restrictions.
- Treat `TaskForceModeThreatProfile*` values as informational mission-resistance indicators only; they do not generate enemy threats at runtime. Author actual enemy aircraft and units explicitly in the mission `.ini`.
