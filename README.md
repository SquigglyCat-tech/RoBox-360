# RoBox Dashboard

Unofficial Metro-style console dashboard for Roblox. Original RoBox
branding and assets throughout — not Xbox, not the Xbox OS port, no Microsoft
assets. Font face: Selawik (Segoe substitute) with Gotham fallback.

## Milestone 1 — foundation and interactive dashboard shell

Implemented:

- **Rojo-compatible source layout** (`default.project.json`): client in
  `StarterPlayerScripts`, server in `ServerScriptService`, shared modules in
  `ReplicatedStorage`.
- **Theme configuration** (`src/shared/Theme.shared.lua`): original RoBox
  palette, Selawik typography with Gotham fallback, tile geometry, restrained
  tween-based animations (no loops).
- **Reusable Tile component** (`src/client/Tile.lua`): border, icon
  glyph, title, subtitle, tweened selection state, hover and click input.
- **Navigation controller** (`src/client/Navigation.lua`): central
  state — sections, per-section selection index, current section, Guide open
  state with previous-selection preservation. Pure state + BindableEvents,
  no UI knowledge.
- **Input handling** (`src/client/Input.lua`): keyboard
  (arrows/WASD/Enter/Space/Esc/G), gamepad (DPad/thumbstick with deadzone +
  repeat throttling, ButtonA/ButtonB/ButtonStart). Mouse handled by tiles.
  One code path for activation regardless of input device.
- **Home screen** (`src/client/HomeScreen.lua`): RoBox wordmark and one
  tile per section (Games, Apps, Settings, Profile).
- **Section screens** (`src/client/SectionScreen.lua`): functional
  Games, Apps, Settings and Profile destinations with back navigation.
- **Detail stubs** (`src/client/DetailScreen.lua`): honestly report
  "not part of milestone 1" instead of fake functional buttons.
- **Working Settings demo:** Settings > Audio opens a volume pane whose
  changes are sent to the server, validated against `SettingsSchema`, and
  persisted to DataStore. Falls back to local defaults if the server or
  DataStore is unavailable.
- **Guide overlay** (`src/client/Guide.lua`): opens/closes with G
  (keyboard) or Start (gamepad), remembers the previous selection and
  restores it on close, menu-driven navigation with Resume and section
  shortcuts.
- **Server-side persistence** (`src/server/SettingsPersistence.server.lua`):
  RemoteFunction `RoBoxSettings`, schema validation on every write, DataStore
  with safe failure handling.
- **Simulated data flagged** (`src/shared/SimulatedData.shared.lua`):
  `SIMULATED_DATA = true`, clearly marked placeholder content.

## Repository layout

```
default.project.json   Rojo project file
README.md
docs/ASSETS.md         asset sources and licenses
src/
  client/              -> StarterPlayer.StarterPlayerScripts.RoBox.Client
    init.client.lua    LocalScript entry point
    Dashboard.lua      shell, screen stack, input wiring (ModuleScript)
    Navigation.lua     central navigation state (ModuleScript)
    Input.lua          keyboard + gamepad mapping (ModuleScript)
    Tile.lua           reusable tile component (ModuleScript)
    Guide.lua          Guide overlay (ModuleScript)
    HomeScreen.lua     home screen (ModuleScript)
    SectionScreen.lua  section shell (Games, Apps, Settings, Profile) (ModuleScript)
    DetailScreen.lua   not-implemented detail stub (ModuleScript)
    SettingsClient.lua settings cache + remote calls (ModuleScript)
  shared/              -> ReplicatedStorage.RoBox.Shared
    Theme.shared.lua   theme config
    Constants.shared.lua constants and input maps
    SettingsSchema.shared.lua defaults, schema, validation
    SimulatedData.shared.lua clearly flagged simulated data
  server/              -> ServerScriptService.RoBox.Server
    init.server.lua    Script entry point
    SettingsPersistence.lua RemoteFunction + DataStore wrapper
```

## Setup

1. Install Rojo 7.x (https://github.com/rojo-rbx/rojo/releases).
   This project was validated against Rojo 7.7.1.
2. Create a place in Roblox Studio, open it.
3. Enable **API Services** (Home > Game Settings > Security > Enable Studio
   Access to API Services) so DataStore persistence works in Play Solo.
4. Sync the source tree into Studio:

   ```
   rojo serve default.project.json
   ```

   then connect Rojo in Studio, or build a package and insert it:

   ```
   rojo build default.project.json -o RoBoxDashboard.rbxm
   ```

   and insert `RoBoxDashboard.rbxm` into Workspace/StarterPlayer etc. as the
   project tree describes.
5. Press Play.

## Controls

| Action | Keyboard | Gamepad | Mouse |
|---|---|---|---|
| Move selection | Arrows or WASD | DPad / left stick | Hover |
| Activate tile | Enter / Space | Button A | Click |
| Back | Esc | Button B | Back tile |
| Toggle Guide | G | Button Start | Guide items |
| Guide move | Arrows / WASD | DPad / left stick | Click item |
| Guide select | Enter / Space | Button A | Click item |

## Testing performed

- **Project file validation:** `rojo sourcemap default.project.json` parses
  and maps every source path to the expected service (see output below).
- **Luau review:** all modules use `--!strict`, explicit types, and no
  implicit globals. No runtime Luau execution is possible outside Roblox
  Studio (TweenService, GuiObjects, DataStoreService, Players, RemoteFunction
  all require the Roblox engine), so engine behavior was not executed here.

## What cannot be validated outside Roblox Studio

- Runtime input flow (keyboard/gamepad/mouse event delivery).
- Animation timing and tween completion.
- DataStore reads/writes (depends on API Services being enabled).
- ScreenGui layering and ZIndex behavior.
- Gamepad thumbstick deadzone feel.

## Studio validation checklist

1. Play Solo. Home screen shows "RoBox" wordmark + four section tiles.
2. Arrows/WASD and gamepad DPad move the highlight; wrap and row boundaries
   behave; Enter/Space or Button A opens the selected section.
3. Games / Apps / Settings / Profile each open their section screen with a
   working Back tile; Guide (G / Start) opens over any screen and closes
   without losing the selection.
4. Guide menu shortcuts jump to sections; Resume closes the Guide.
5. Settings > Audio: change volume with +/-; the value persists. Leave and
   rejoin the place — the saved volume returns (requires API Services).
6. With API Services disabled, volume changes still work locally and Output
   shows a warning, not an error.
7. No polling: nothing updates without user input; no unbounded animation
   loops.
