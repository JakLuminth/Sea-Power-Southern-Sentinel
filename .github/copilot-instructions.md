# Copilot Instructions

## Project Guidelines
- Use `docs/sea-power-task-force-mode-guide.md` as the preferred repo-local reference summary for Task Force Mode authoring patterns derived from the referenced Sea Power guide.
- Use `docs/sea-power-mission-creation-guide.md` as the preferred repo-local reference summary for advanced mission authoring, trigger logic, briefing patterns, and direct `.ini` mission editing.
- Use `docs/sea-power-flight-deck-guide.md` as the preferred repo-local reference summary for flight deck operations and mission authoring patterns.
- For this Sea Power workspace, useful stock-reference locations are under:
  - `StreamingAssets/original/vessels` for vessels
  - `StreamingAssets/original/aircraft` for aircraft
  - `StreamingAssets/original/campaigns` for stock campaigns
  - `StreamingAssets/original/campaigns/<campaign>/missions` for campaign mission files
  - `StreamingAssets/original/missions` for standalone stock missions
- For advanced mission authoring, prefer these read-only stock references when syntax or behavior is uncertain:
  - `StreamingAssets/original/documentation/Mission Editor. Triggers and conditions.docx` for trigger and condition rules
  - `StreamingAssets/original/missions/Demo/MissionFileInformation.ini` for mission fields, actions, and examples
  - `StreamingAssets/original/missions/Warsaw Pact/Breakthrough.ini` for a complete authored mission example
  - `StreamingAssets/original/audio/voiceover/voice.ini` for voice-message keys and custom audio patterns
  - `StreamingAssets/original/language_en/ammunition_names.ini` for accountable ammunition category names
- Use `StreamingAssets/original/campaigns/pacific-strike-task-force/` as the complete stock Task Force Mode campaign reference, including `campaign.ini`, roster, commander settings, and mission files.
- Treat `StreamingAssets/original` as read-only stock game data; do not modify files there. Keep custom content under a direct-child mod folder below `StreamingAssets`, as described in the next rule.
- For packaged custom mods, use a mod folder directly under `StreamingAssets/<Your Mod Name>/` and enable it in the mod manager. Keep `StreamingAssets/user/missions` distinct as the default editor/user-mission location.
- For undocumented engine behavior, inspect the read-only compiled assemblies under `Managed/`, especially `Managed/Seapower-Scripts.dll` for mission, campaign, Task Force Mode, and briefing logic and `Managed/Noesis.NoesisGUI.dll` for XAML briefing UI behavior. Use assemblies from the current game build, never modify them, and verify inferred behavior against runtime logs or in-game testing.

## Authoring Rules
- Use the Mission Editor for initial mission layout, then edit advanced behavior directly in the mission `.ini`. Do not reopen and save a manually edited mission in the editor unless those direct edits can be recreated; editor saves may discard them. Test advanced behavior from the Scenarios menu.
- When syntax or behavior is uncertain, verify against the read-only stock references and the current game build. Treat community-guide behavior as patch-dependent and preserve each guide's source URL, author, update date, and verification caveats when maintaining the repo-local summaries.
- When adding triggers manually, update `[Mission] NumberOfTriggers` and keep objective completion, failure, and cancellation actions synchronized with trigger outcomes. Avoid assuming triggers firing exactly at mission start or under heavy time acceleration are reliable.
- Treat `Flight Deck Timings` as a global Options setting. Use mission `.ini` files for aircraft readiness and accountable ordnance; keep unit-level `GroundCrewCount` and `DeckParkSlots` in the carrier or airbase unit definition.
- For Sea Power mission and Task Force Mode files, do not use `Variant=*_variants` or `*_varients` shorthand tokens. Use stock-compatible explicit `VariantReference=VariantN` values, and keep roster entries explicit as `Variant1,Variant2,...`.
- Treat `campaign.ini` `MissionFile` paths as relative to the mod/package root, not relative to the directory containing `campaign.ini`.
- Store a left-pane briefing asset for each campaign-referenced combat mission in the adjacent `<MissionName>_briefing/BriefingText_en.xml` path when briefing text is required.
- Edit existing briefing assets in place rather than attempting to recreate files that already exist.
- Preserve briefing runtime tokens such as `{TaskForceName}`, `{FlagshipName}`, and `{SelectedUnitName}`.
- Keep briefing objective lines synchronized with the mission's `Objective_*` entries.
- After briefing changes, parse every campaign-referenced briefing XML file and verify that no referenced asset is missing and no custom briefing asset is unreferenced.
- For Sea Power mission authoring, treat vessel movement as waypoint-driven: every vessel that should move must have explicit Waypoints; do not attribute movement or course changes to automatic task-force AI without runtime evidence.

## Task Force Mode Guidelines
- Keep campaign-wide and per-mission Task Force Mode rules—including economy, generation type, flags, threat profiles, rewards, and availability—in `campaign.ini`.
- Use mission `.ini` files for authored placement and unit-slot behavior. In generated missions, use `Taskforce1Vessel1` as the anchor section and set `TaskForceModeAnchor=True` only there; use replacement slots, Air Tasking slots, and `JoinTaskForce=True` only when needed and follow the detailed repo-local guide for field restrictions.
- For Sea Power Task Force Mode, `TaskForceModeThreatProfile*` values are informational mission-resistance indicators only; they do not generate enemy threats at runtime. Actual enemy aircraft and units must be authored explicitly in the mission INI.
