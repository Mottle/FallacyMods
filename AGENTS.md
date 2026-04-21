# AGENTS.md

## Repo Shape (Do Not Guess)
- Root is a Gradle multi-project build (`settings.gradle.kts`) with modules: `base`, `thermal`, `survive`, `hud`.
- `base/`, `thermal/`, and `survive/` are git submodules (`.gitmodules`) and each also has its own standalone Gradle setup.
- `hud/` is part of this repo (not a submodule).
- Module dependency graph (from module `build.gradle.kts`):
  - `base`: shared foundation.
  - `thermal`: depends on `:base`.
  - `hud`: depends on `:base`.
  - `survive`: depends on `:base` and `:thermal`; `:hud` is `compileOnly`.

## Subproject AGENTS Contract
- Module-level agent docs now exist at:
  - `base/AGENTS.md`
  - `thermal/AGENTS.md`
  - `survive/AGENTS.md`
  - `hud/AGENTS.md`
- When a task touches a specific module, read that module's `AGENTS.md` first, then apply root rules.
- For module-specific details, module `AGENTS.md` takes precedence; for cross-module/repo-wide constraints, this root `AGENTS.md` takes precedence.
- If module architecture, entrypoints, registration flow, or build conventions change, update the corresponding module `AGENTS.md` in the same change.

## Toolchain and Build Facts
- Java toolchain is 21 and Kotlin JVM target is 21 in all modules.
- Gradle wrapper is `8.13` (`gradle/wrapper/gradle-wrapper.properties`).
- All mod modules use `net.neoforged.moddev` `2.0.113`.
- All modules resolve local jars from `../lib` via `flatDir`, so root `lib/` can affect compilation.
- Tau usage guide for agents: `lib/TAU_AGENT_GUIDE.md`.
- If a task needs Tau UI/menu work, read `lib/TAU_AGENT_GUIDE.md` before making changes.

## Commands That Usually Matter
- Command execution preference: because the real dev environment is Windows, prefer running Gradle via `powershell.exe ./gradlew ...` from WSL/OpenCode.
- Build one module from root (preferred for focused verification):
  - `powershell.exe ./gradlew :base:build`
  - `powershell.exe ./gradlew :thermal:build`
  - `powershell.exe ./gradlew :survive:build`
  - `powershell.exe ./gradlew :hud:build`
- Fast compile check after edits: `powershell.exe ./gradlew :<module>:compileKotlin`
- There are no `src/test` directories currently; do not assume tests provide coverage.
- NeoForge run configs are declared per module (`client`, `server`, `gameTestServer`, `data`) and mapped to generated Gradle run tasks.

## Generated Files and Datagen
- Mod metadata is templated: edit `*/src/main/templates/META-INF/neoforge.mods.toml` plus module `gradle.properties`, not `build/generated/...`.
- `src/generated/resources` is included in main resources for modules; treat it as generated source-of-truth output for datagen.
- For thermal datapack changes, update/regenerate `thermal/src/generated/resources/...`, not runtime save data under `run/`.

## Real Entrypoints
- Main mod objects:
  - `base/src/main/kotlin/dev/deepslate/fallacy/base/TheMod.kt`
  - `thermal/src/main/kotlin/dev/deepslate/fallacy/thermal/TheMod.kt`
  - `survive/src/main/kotlin/dev/deepslate/fallacy/survive/TheMod.kt`
  - `hud/src/main/kotlin/dev/deepslate/fallacy/hud/TheMod.kt`
- Event wiring is mostly `@EventBusSubscriber`-based (commands, registries, GUI hooks); check subscriber objects before adding manual init logic.

## Git Gotcha (Easy to Miss)
- Root `git status` only shows submodule pointer changes for `base`, `thermal`, `survive`.
- Inspect actual file diffs with module-local commands, e.g. `git -C base status`, `git -C thermal status`, `git -C survive status`.
