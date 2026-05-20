# Technical Preferences

<!-- Populated by /setup-engine. Updated as the user makes decisions throughout development. -->
<!-- All agents reference this file for project-specific standards and conventions. -->

## Engine & Language

- **Engine**: Unreal Engine 5.7
- **Language**: C++ (primary), Blueprint (gameplay prototyping)
- **Rendering**: Lumen (global illumination), Nanite (virtualized geometry)
- **Physics**: Chaos Physics (default in UE5)

## Input & Platform

<!-- Written by /setup-engine. Read by /ux-design, /ux-review, /test-setup, /team-ui, and /dev-story -->
<!-- to scope interaction specs, test helpers, and implementation to the correct input methods. -->

- **Target Platforms**: PC (Steam / Epic)
- **Input Methods**: Keyboard/Mouse, Gamepad
- **Primary Input**: Keyboard/Mouse (mouse-aimed spells, precision advantage on PC)
- **Gamepad Support**: Partial (recommended — test but not required for prototype)
- **Touch Support**: None
- **Platform Notes**: No hover-only UI interactions. Gamepad support should not block prototype shipping but must be validated before vertical slice. All menus must be navigable without a mouse.

## Naming Conventions

- **Classes**: Prefixed PascalCase — `A` (Actor), `U` (UObject), `F` (struct), `I` (interface), `E` (enum). e.g. `APlayerWizard`, `UCorruptionComponent`, `FSpellData`
- **Variables**: PascalCase — `MoveSpeed`, `MaxCorruption`, `CurrentWave`
- **Booleans**: `b` prefix — `bIsInfected`, `bIsAlive`, `bCanCast`
- **Functions**: PascalCase — `TakeDamage()`, `GetCorruptionLevel()`, `PurgeCorruption()`
- **Files**: Match class name without prefix — `PlayerWizard.h`, `WaveSpawner.cpp`
- **Scenes/Levels**: PascalCase — `Arena_Main.umap`, `BP_ZombieSpawner.uasset`
- **Constants**: PascalCase (`static constexpr`) or `UPPER_SNAKE_CASE` for macros

## Performance Budgets

- **Target Framerate**: 60 fps
- **Frame Budget**: 16.6 ms
- **Draw Calls**: ≤ 1,000 (typical UE5 PC arena target)
- **Memory Ceiling**: To be established after prototype profiling

## Testing

- **Framework**: Unreal Automation Framework (`FAutomationTestBase`) for unit/integration; Gauntlet for larger system tests
- **Minimum Coverage**: Infection state machine, wave escalation logic, purge calculations
- **Required Tests**: Corruption state transitions, spell damage formulas, wave spawner counts, cleanse zone overlap logic

## Forbidden Patterns

<!-- Add patterns that should never appear in this project's codebase -->
- [None configured yet — add as architectural decisions are made]

## Allowed Libraries / Addons

<!-- Add approved third-party dependencies here — only add when actively integrating, never speculatively -->
- ECABridge (MCP plugin for AI-assisted development — `Plugins/ECABridge/`) — prototype tooling

## Architecture Decisions Log

<!-- Quick reference linking to full ADRs in docs/architecture/ -->
- [No ADRs yet — use /architecture-decision to create one]

## Engine Specialists

<!-- Written by /setup-engine when engine is configured. -->
<!-- Read by /code-review, /architecture-decision, /architecture-review, and team skills -->
<!-- to know which specialist to spawn for engine-specific validation. -->

- **Primary**: unreal-specialist
- **Language/Code Specialist**: ue-blueprint-specialist (Blueprint graphs, BP/C++ boundary design)
- **Shader Specialist**: unreal-specialist (no dedicated shader specialist — primary covers materials)
- **UI Specialist**: ue-umg-specialist (UMG widgets, CommonUI, input routing, widget styling)
- **Additional Specialists**: ue-gas-specialist (Gameplay Ability System — spells, effects, attributes), ue-replication-specialist (if multiplayer added later — property replication, RPCs)
- **Routing Notes**: Invoke primary for C++ architecture and broad engine decisions. Invoke Blueprint specialist for Blueprint graph architecture and BP/C++ boundary design. Invoke GAS specialist for all spell and ability code. Invoke UMG specialist for all UI implementation. Invoke replication specialist only if multiplayer systems are added.

### File Extension Routing

<!-- Skills use this table to select the right specialist per file type. -->
<!-- If a row says [TO BE CONFIGURED], fall back to Primary for that file type. -->

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| Game code (.cpp, .h files) | unreal-specialist |
| Shader / material files (.usf, .ush, Material assets) | unreal-specialist |
| UI / screen files (.umg, UMG Widget Blueprints) | ue-umg-specialist |
| Scene / prefab / level files (.umap, .uasset) | unreal-specialist |
| Native extension / plugin files (.uplugin, plugin modules) | unreal-specialist |
| Blueprint graphs (.uasset BP classes) | ue-blueprint-specialist |
| General architecture review | unreal-specialist |
