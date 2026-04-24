# TAU Agent Guide

This guide is for agents working in this repository.
Read this file before implementing or changing Tau-based UI/menu logic.

Primary reference docs:

- https://mc-tau-ui.github.io/docs/getting-started/basics-of-tau
- https://mc-tau-ui.github.io/docs/getting-started/render
- https://mc-tau-ui.github.io/docs/getting-started/dynamic
- https://mc-tau-ui.github.io/docs/getting-started/primitive
- https://mc-tau-ui.github.io/docs/builtin-comps/how-to-use

## Scope and priority

- Prefer Tau for new UI and container menu work.
- Avoid ad-hoc vanilla `AbstractContainerMenu` + `AbstractContainerScreen` pairs if Tau can cover the use case.
- Keep behavior and style consistent with existing systems in `base/`, `survive/`, and `hud/`.

## Core mental model (from Tau docs)

- `UIComponent` is the core building block. Everything displayed in Tau is composed from nested `UIComponent`s.
- A `UIComponent` is conceptually like a stateless widget: it builds UI from input layout/theme.
- Complex UI should be composed from small reusable components to keep code maintainable.
- Many built-ins use a `Builder` pattern. If a builder exists, prefer it for optional arguments.

## Rendering modes

Tau supports two rendering modes:

1. `ScreenUIRenderer`
   - Renders a Tau UI as a Minecraft screen.
   - Can render standard translucent screen background.
   - Base docs say this renderer pauses by default.
2. `HudUIRenderer`
   - Renders directly in HUD/overlay events.

Project conventions:

- Use `ExtendedUIRender` (`base/.../ExtendedUIRender.kt`) for Tau screen rendering in this repo.
- Use `HudUIRenderer` pattern for HUD layers (see `hud/src/main/kotlin/...`).

## Dynamic and primitive components

- Use `DynamicUIComponent` when UI state must rebuild over time or on events.
- Call `rebuild()` when state changes and a rebuild is needed.
- In this repo's Tau usage, `rebuild()` must be triggered from the dynamic chain rooted at the currently rendered root
  component. If a nested dynamic child does not propagate correctly, promote dynamic state/tick handling to the root
  screen component (or ensure every parent on the path is dynamic-aware) so rebuild reliably takes effect.
- Do not mutate internal engine fields of `DynamicUIComponent`.
- Use `PrimitiveUIComponent` only for lower-level custom rendering/input integration when built-ins are insufficient.

## Built-in component usage rules

General reading rule (from docs): check each component's size behaviour (min/max/fixed) and parameters before composing.

Useful components for this repo:

- `Stack`: layer components in declared order.
- `Container`: single-child wrapper, optional background color, optional size behaviour.
- `Sized`: constrain child to fixed/percentage size.
- `Align` / `Center` / `Positioned`: explicit alignment and placement.
- `Row` / `Column` / `ListView`: horizontal/vertical/scrollable composition.
- `Texture`: draw textured regions (`texture`, `textureSize`, `uv`, `uvSize`, `size`).
- `Button`, `Text`, `TextField`, `WidgetWrapper`: interaction and text primitives.
- `Render` / `Renderable`: custom render callback integration.

## Theme guidance

- Tau supports custom themes via `Theme` interface.
- Default behavior is Minecraft-like theme.
- In this repo, keep default theme unless task explicitly requests custom theme behavior.

## Recommended container menu pattern (Tau)

1. Register menu with `TauMenuHolder`.
2. Implement `UIMenu` for layout and slot logic.
3. Register client menu screen with `TauMenuHelper.registerMenuScreen(...)`.
4. Open menu from server with `TauMenuHolder.openMenu(serverPlayer, pos)`.

### 1) Menu registration

Use a `DeferredRegister<MenuType<*>>` and `TauMenuHolder`:

```kotlin
val BOIL_POT_MENU = TauMenuHolder(registry, ::BoilPotUI, "boil_pot", FeatureFlags.DEFAULT_FLAGS)
```

### 2) UIMenu implementation

Implement `com.github.wintersteve25.tau.menu.UIMenu` with these key methods:

- `build(layout, theme, menu)`: build Tau component tree.
- `getSize()`: menu size, often `176x166` for vanilla-like containers.
- `getTitle()`: translatable title component.
- `getSlots(menu)`: return slot handlers (custom + `PlayerInventoryHandler`).
- `quickMoveStack(menu, player, index)`: explicit shift-click behavior.

Optional methods:

- `stillValid`, `tick`, `addDataSlots`, `getTheme`, `shouldRenderBackground`.

### 3) Screen registration

In `RegisterMenuScreensEvent`:

```kotlin
TauMenuHelper.registerMenuScreen(event, ModMenus.BOIL_POT_MENU)
```

### 4) Opening menu

From block interaction (server side only):

```kotlin
val serverPlayer = player as? ServerPlayer ?: return InteractionResult.PASS
ModMenus.BOIL_POT_MENU.openMenu(serverPlayer, pos)
```

## Slot/layout notes for Tau containers

- Tau menu slot handlers apply position offsets internally; verify visual alignment in-game.
- For fixed rows (example: 6 slots), use `Row` + `ItemSlot` and place with `Positioned(x, y)`.
- For player inventory area, use `PlayerInventory(Variable(true))` and include `PlayerInventoryHandler` in `getSlots`.
- Use `Texture.Builder` with explicit UV/size values for predictable background rendering.

## Behavior rules for gameplay-safe menus

- Keep server-authoritative behavior; do not rely on client-only checks for permissions/rules.
- For take-only containers:
  - enforce in inventory handler (`isItemValid=false`, `insertItem` returns input stack), and
  - mirror with `quickMoveStack` (allow container -> player only).
- Keep null/lookup access defensive (`getOrNull`, empty stack early return, fallback handlers).

## Non-container Tau screens in this repo

- For non-menu screens, use `ExtendedUIRender` + Tau components.
- If back navigation is needed, use `UIContext` + `ContextWrapperUI`.

## Current project references

- Screen wrapper: `base/src/main/kotlin/dev/deepslate/fallacy/base/client/screen/component/ExtendedUIRender.kt`
- Context wrapper: `base/src/main/kotlin/dev/deepslate/fallacy/base/client/screen/component/ContextWrapperUI.kt`
- UI context: `base/src/main/kotlin/dev/deepslate/fallacy/base/client/screen/UIContext.kt`
- Survive UI example: `survive/src/main/kotlin/dev/deepslate/fallacy/survive/client/screen/DietUI.kt`
- Tau menu example: `survive/src/main/kotlin/dev/deepslate/fallacy/survive/menu/BoilPotUI.kt`

## Verification checklist

- Compile with repo preference (Windows environment from WSL/OpenCode):
  - `powershell.exe ./gradlew :<module>:compileKotlin`
- In-game verify:
  - visual alignment,
  - scroll/input behavior,
  - shift-click logic,
  - insertion/extraction restrictions,
  - open/close flow,
  - no client/server desync symptoms.
