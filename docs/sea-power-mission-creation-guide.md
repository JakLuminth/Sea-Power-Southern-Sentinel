# Sea Power Mission Creation Guide Notes

Source reference: https://steamcommunity.com/sharedfiles/filedetails/?id=3392721470

Source author: Techno

Source status: updated November 4, 2025. The source describes itself as a work in
progress and combines official documentation with community testing and
experimentation. Treat field names and examples as patch-dependent; confirm
uncertain behavior against the stock references and current game build.

This file is a Copilot-friendly working summary of the referenced Sea Power community mission creation guide. It is intended as a repo-local reference for authored mission editing patterns, especially direct `.ini` work beyond what the in-game editor safely preserves.

## Scope and Intent

- This guide is about advanced mission creation rather than Task Force Mode campaign structure.
- It complements the Task Force Mode guide by focusing on authored mission `.ini` patterns, trigger logic, messaging, air groups, briefings, and scenario scripting.
- Treat it as a practical mission-authoring reference, not as an authoritative replacement for official documentation.

## Main Themes From the Guide

The guide explicitly covers:
- basics and source references
- conditional triggers
- weather and sea state
- custom air groups
- teleports, formations, and changing sides
- popup/intelligence/objective messages
- voice lines and custom audio
- scripted attacks
- civilian routes / scripted unit spawning
- briefings
- folder structure and uploading
- mission design tips

## Core Workflow Guidance

- Use the mission editor to create the initial structure of a mission:
  - place units
  - place groups
  - set waypoints
  - establish the basic geometry
- After the mission skeleton is correct, switch to direct `.ini` editing for advanced behavior.
- Important warning from the guide:
  - if you reopen and save a mission in the editor after manual `.ini` edits, those manual changes may be lost
  - test advanced mission behavior from the scenarios menu rather than by relaunching through the editor

Practical workspace rule:
- Use the editor for layout, then preserve advanced authored logic by editing mission `.ini` files directly.

## Default User Mission Location

The guide states that user missions normally save under:
- `StreamingAssets/user/missions`

For this workspace, custom campaign missions live under campaign folders such as:
- `StreamingAssets/<Your Mod Name>/campaigns/<campaign>/missions`

## Official and Semi-Official Reference Sources Mentioned

The guide points to useful stock references inside the game install:
- `StreamingAssets/original/documentation/Mission Editor. Triggers and conditions.docx`
- `StreamingAssets/original/missions/Demo/MissionFileInformation.ini`
- example stock mission patterns such as:
  - `StreamingAssets/original/missions/Warsaw Pact/Breakthrough.ini`

Practical workspace rule:
- Prefer these stock files as read-only references when advanced mission behavior is unclear.
- The source's official trigger reference is especially useful for checking valid
  condition types and `ConditionsCompleted` operators instead of inferring them
  from a single community example.

## Trigger Model

The guide treats triggers as the core of authored mission logic.

### Trigger Anatomy
A trigger has three conceptual parts:
1. Trigger definition
2. Conditions
3. Actions

Typical pattern:
- `[TriggerN]`
- `Name=...`
- optional description fields for author clarity
- one or more `Condition_<Name>_...` entries
- `ConditionsCompleted=...`
- one or more `Action_...` entries

### Conditions
- Conditions are the logic checks inside a trigger.
- A trigger can have multiple conditions.
- Conditions can be combined with logical rules in `ConditionsCompleted`.
- Condition types are defined with fields like:
  - `Condition_<Name>_Type=<type>`
- Valid condition types should be confirmed against stock references such as `MissionFileInformation.ini` and the official trigger documentation.

### Actions
Typical trigger actions described or exemplified by the guide include:
- display a message to the player
- end the mission
- set victory for a side
- enable other triggers
- change weather/sea state
- support scripted mission flow

### Practical Trigger Rules
- If you manually add triggers, update `NumberOfTriggers` in the `[Mission]` section.
- Trigger naming and optional description fields improve maintainability.
- Conditional triggers are a major tool for building richer mission flow.
- Avoid relying on triggers that fire exactly at mission start, and retest logic
  that runs during heavy time acceleration; the source notes that both cases can
  behave unreliably.

## Conditional Trigger Guidance

The guide emphasizes conditional triggers as a powerful way to make mission outcomes depend on multiple factors.

Practical rule:
- Use multiple conditions with explicit `ConditionsCompleted` logic when mission success should depend on both location and survival, or timing and destruction, rather than a single event.

## Weather and Sea State

The guide notes that weather and sea state can be manipulated through mission logic.

Practical rule:
- Mission weather does not have to be static; for longer or more cinematic authored missions, trigger-based environment changes may be an option.
- The source documents `Action_SetSeaState`, `Action_SetClouds`,
  `Action_SetRain`, `Action_SetThunderstorm`, `Action_SetSnow`, and
  `Action_SetFog`. Sea state values range from 0 to 12, with 1 to 10
  recommended; thunderstorm requires overcast clouds.

## Direct `.ini` Editing Use Cases

The guide positions direct `.ini` editing as the way to add features that are difficult or impossible to preserve through the editor alone.

Examples of logic best handled in direct `.ini` editing:
- advanced trigger networks
- objectives and messages
- scripted routes and civilian behavior
- custom air-group definitions
- teleport/side-change/formations logic
- more elaborate briefings and support messaging

## Messaging Categories

The guide distinguishes multiple message styles and use cases:
- popup messages
- intelligence messages
- objective messages
- voice line / audio-linked messaging

Practical rule:
- Match message type to purpose:
  - popups for immediate tactical notification
  - intelligence/naval messages for operational updates
  - objective messages for explicit task-state changes

## Objectives

The guide includes a dedicated objective-focused section.

Practical rule:
- Objectives should be driven by trigger logic and synchronized with mission flow.
- When changing trigger outcomes, always verify the corresponding objective complete/fail/cancel actions remain coherent.

## Custom Air Groups

The guide includes custom air groups as a distinct topic.

Practical rule:
- When using mission-authored aircraft or ship-based aviation behavior beyond basic defaults, verify whether custom air group fields are needed and whether the editor preserves them safely.
- `CustomAirGroup=True` replaces the unit's default air group. Specify the
  desired aircraft entries explicitly; otherwise the unit can spawn with no
  aircraft. Check the carrier or airbase's supported types and capacity rather
  than assuming every requested aircraft will be accepted.

## Teleports, Formations, and Changing Sides

The guide identifies these as advanced but supported mission-authoring techniques.

Practical rule:
- When mission behavior depends on side changes, teleports, or explicit formation control, prefer direct `.ini` authoring with stock examples rather than relying on editor assumptions.

## Scripted Attacks and Civilian Routes

The guide highlights:
- scripted attacks
- civilian routes
- scripted spawning/route behavior

Practical rule:
- Neutral/civilian traffic can be an authored gameplay system, not just decoration.
- Use direct route and trigger logic to support convoy pressure, clutter, deception, and traffic management in authored scenarios.
- Despite its name, `CivilianRoute` can launch aircraft for any team. The route
  must reference an airbase or carrier that already has the aircraft available;
  the source's later author clarification says it does not create arbitrary
  variants from nothing.

## F10 Menu and Custom Units

The source includes an F10 menu section and a custom-units section. These areas
are less complete than the trigger and mission-scripting sections, so use the
guide as a pointer to current examples rather than as a complete schema. Check
stock mission files and the current build before depending on custom-unit or F10
behavior in a campaign.

## Briefings

The guide includes a dedicated briefings section.

Practical rule:
- Mission briefings should stay aligned with actual authored logic, trigger pacing, and map-symbol reality.
- If the scenario changes structurally, update briefing text alongside mission logic.

## Folder Structure and Uploading

The guide includes publishing-oriented material.

Practical workspace rule:
- Keep standalone editor-created missions in the default
  `StreamingAssets/user/missions` location when appropriate. For a packaged mod,
  put the mod folder directly under `StreamingAssets/<Your Mod Name>/` and
  mirror the structure under `StreamingAssets/original`; avoid editing
  `StreamingAssets/original/...`.

## Mission Design Guidance Implied by the Guide

Derived practical rules:
- Start simple in the editor, then add complexity in `.ini`
- Prefer maintainable trigger names and clear mission logic structure
- Use stock references liberally when uncertain about exact fields or behavior
- Expect some advanced features to depend on community-tested patterns rather than polished official docs
- Test authored behavior outside the editor once direct `.ini` changes are in place

## Guidance For This Workspace

Use this guide when working on:
- authored mission logic
- triggers and objectives
- support messages
- civilian routes
- direct `.ini` mission scripting
- any advanced mission behavior not safely preserved by the editor

Use `docs/sea-power-task-force-mode-guide.md` instead when the question is mainly about:
- persistent force setup
- task force builder economy
- campaign-wide Task Force Mode configuration
- generated vs replaced campaign mission usage

## Why This Summary Exists

The original web guide is not ideal as a persistent repo instruction source. This markdown file captures the mission-authoring parts most useful for future Sea Power scenario and campaign work in this workspace.
