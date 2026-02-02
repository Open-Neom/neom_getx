# neom_getx vs getx — First Commit Comparison

## Overview

`neom_getx` is a hard fork of the original GetX (v5.0.0-rc) package. This first version applies a strategic pruning to remove modules that fall outside the core mission of the package.

The original GetX tried to be an all-in-one framework — HTTP client, animations, string validators, widget extensions, and more. While convenient for quick prototyping, this approach introduces maintenance burden, increases binary size, and makes it harder for new developers to understand what the package actually does.

`neom_getx` narrows the focus to four pillars:

    SM — State Management
    DI — Dependency Injection
    N  — Navigation
    TR — Translations

GetX is already good and easy to adopt for these four concerns. By removing everything else, the package becomes easier for future developers to understand and easier to maintain long-term.

---

## By the Numbers

| Metric | getx (original) | neom_getx (v1) | Delta |
|---|---|---|---|
| Dart files in `lib/` | 134 | 124 | **-10 files** |
| Lines of code in `lib/` | 20,615 | 19,679 | **-936 LOC** |
| Top-level modules | 9 | 8 | -1 module (`get_animations`) |
| `get_utils/` files | 21 | 15 | **-6 files** |

### Breakdown of Removed Lines

| Source | Files | LOC |
|---|---|---|
| `get_animations/` (full removal) | 4 | ~736 |
| `get_utils/` extensions (partial removal) | 6 | ~200 |
| **Total removed** | **10** | **~936** |

---

## What Was Removed

### `get_animations/` — Fully removed (4 files, ~736 LOC)

Widget animation extensions (`.fadeIn()`, `.bounce()`, `.spin()`, etc.) and the `GetAnimatedBuilder`. These were introduced in GetX 5.0.0-rc2 but are not used in any of the 21+ neom_modules or any of the apps (Cyberneom, EMXI, Gigmeout). Animation concerns should live in a dedicated package (e.g., `neom_animation`) rather than inside a state management library.

### `get_utils/` — Partially removed (6 files, ~200 LOC)

| Removed file | Reason |
|---|---|
| `duration_extensions.dart` | Duration delay helpers — not used across the ecosystem |
| `int_extensions.dart` | Int delay/duration helpers — not used |
| `iterable_extensions.dart` | Iterable chunking/mapping — not used |
| `num_extensions.dart` | Num delay/duration helpers — not used |
| `widget_extensions.dart` | Widget padding/margin shortcuts — not used |
| `widgets/optimized_listview.dart` | Custom ListView optimization — not used |

### What Was Kept in `get_utils/`

The following files remain because they are actively used across the ecosystem:

| File | Reason |
|---|---|
| `internacionalization.dart` | `.tr`, `.trArgs`, `.trPlural`, `LocalesIntl` — used across 80+ translation references |
| `string_extensions.dart` | String utilities actively used in modules |
| `context_extensions.dart` | Context shortcuts actively used in modules |
| `double_extensions.dart` | Double utilities actively used in modules |
| `dynamic_extensions.dart` | Dynamic type extensions actively used in modules |
| `event_loop_extensions.dart` | Event loop scheduling — dependency of translations |
| `equality/equality.dart` | Deep equality — used internally |
| `get_utils/get_utils.dart` | `GetUtils` static class — used by platform detection |
| `queue/get_queue.dart` | Internal queue — used internally |
| `platform/` (4 files) | Platform detection — used by `get_navigation` routing |

---

## What Was Kept (Unchanged)

| Module | Pillar | Purpose |
|---|---|---|
| `get_state_manager/` | **SM** | Reactive state (`GetxController`, `GetBuilder`, `Obx`, `.obs`, Workers) |
| `get_rx/` | **SM** | Reactive extensions (`Rx`, `RxList`, `RxMap`, `RxBool`, streams) |
| `get_instance/` | **DI** | Dependency injection (`Get.put`, `Get.find`, `Get.lazyPut`) |
| `get_navigation/` | **N** | Routing (`GetPage`, `Get.toNamed`, middleware, `GetMaterialApp`) |
| `get_utils/` (partial) | **TR** | Translations (`.tr`, `Translations` class, i18n) |
| `get_core/` | Core | Core interfaces (`GetInterface`, logging, lifecycle) |
| `get_common/` | Core | Shared reset utilities |

### Note on `get_connect/`

`get_connect/` (HTTP client, WebSocket, GraphQL) is **not used** within the neom ecosystem, but it was intentionally kept for backward compatibility. Other developers relying on `GetConnect` for HTTP communication should not be broken by this fork. It is a candidate for extraction into a separate package in a future version.

---

## Fixes Applied

| Fix | File | Description |
|---|---|---|
| Removed `GetUtils` dependency | `platform_web.dart` | Replaced `GetUtils.hasMatch(...)` with inline `RegExp(...).hasMatch()` to eliminate dependency on the deleted utility class |
| Updated barrel exports | `get_utils.dart`, `export.dart` | Removed exports of deleted files to prevent compilation errors |
| Removed main barrel export | `get.dart` | Removed `export 'get_animations/index.dart'` |

---

## Tests

The `test/` directory contains test suites for `animations/` and `utils/` that reference removed modules. These tests have been commented out for now. This is intentional — we want to evaluate whether those tests should be adapted for retained functionality or simply removed entirely in a future cleanup pass.

Test suites for the core modules (`instance/`, `internationalization/`, `navigation/`, `rx/`, `state_manager/`) remain intact and unmodified.

---

## Roadmap

### Phase 1 — Package Rename (Current)

- Migrate all 566 `.dart` files across the ecosystem from `import 'package:get/get.dart'` to `import 'package:neom_getx/neom_getx.dart'`
- Remove the `dependency_overrides` hack — publish `neom_getx: 1.0.0` so apps can depend on it directly
- Evaluate remaining tests: decide whether `test/animations/` and `test/utils/` should be updated or removed

### Phase 2 — Full Pruning of `get_utils/`

- Move the `.tr` extension and `Translations` class into `get_navigation/` (where the `Translations` abstract class already lives)
- Migrate remaining extension usages (`string_extensions`, `context_extensions`, `dynamic_extensions`, `double_extensions`) into `neom_commons` or `neom_utilities`
- Remove `get_utils/` entirely, making `neom_getx` purely **SM + DI + N + TR**

### Phase 3 — Extract `get_connect/`

- If no active external projects depend on it, remove it
- If demand exists, maintain it until it is extracted into a separate `neom_connect` package 
- This removes the last non-core module from `neom_getx`

### Phase 4 — SM + DI + N + TR — Future Possibilities

Once `neom_getx` contains only the four pillars, the following become achievable:

#### State Management (SM)
- **Reactive performance profiling**: Built-in tools to measure `.obs` rebuild frequency and identify unnecessary rebuilds across Open Neom's screens
- **Scoped reactive state**: Introduce scoped `Rx` containers that automatically dispose when a navigation scope ends, reducing memory leaks in long-running sessions
- **Stream interop**: First-class bridges between `Rx` observables and Dart `Stream`/`StreamController` for integration with platform channels and native APIs

#### Dependency Injection (DI)
- **Module-aware DI**: DI scopes that align with `neom_modules/` boundaries — when a module is loaded, its dependencies are registered; when unloaded, they are cleaned up automatically
- **Lazy module loading**: Combined with Flutter's deferred imports, enable modules to load their DI bindings only when first navigated to
- **Testing DI overrides**: Simplified mock injection for unit testing across the 21+ module ecosystem

#### Navigation (N)
- **VR/XR/AR spatial routing**: Extend the existing `GetPage` and middleware system to support spatial navigation patterns for `neom_vr` and future XR modules — scene transitions, spatial anchors, and 3D route stacks
- **Deep link standardization**: Unified deep linking across Cyberneom, EMXI, and Gigmeout with automatic route registration from module definitions
- **Route analytics**: Built-in navigation event tracking that feeds into `neom_analytics` without requiring manual instrumentation
- **Nested navigation improvements**: Better support for bottom navigation bars and tab-based flows common in the neom apps

#### Translations (TR)
- **Dynamic translation loading**: Load translation keys per module on demand rather than bundling all translations at startup
- **Translation key validation**: Build-time checks to ensure all `.tr` keys have corresponding entries in all supported locales
- **RTL layout integration**: Deeper integration between locale changes and RTL/LTR layout switching

#### Cross-Pillar
- **neom_getx DevTools extension**: A Flutter DevTools plugin showing real-time state tree, DI registry, active routes, and current locale — all in one panel
- **Code generation**: Optional `build_runner` integration to generate type-safe route definitions and DI registrations from annotations
- **Flutter Web optimization**: Targeted performance improvements for web builds, critical for Open Neom's browser-based platform

---

## Benefits Summary

| Benefit | Impact |
|---|---|
| **Clearer purpose** | Developers immediately understand what the package does: SM + DI + N + TR |
| **Easier onboarding** | New contributors don't need to understand HTTP clients or animation systems |
| **Reduced maintenance** | 10 fewer files and 936 fewer lines to maintain, test, and debug |
| **Smaller binary** | Less code compiled into final apps, especially relevant for Flutter Web |
| **Better separation of concerns** | Animations belong in `neom_animation`, HTTP in `neom_connect` — each package has one job |
| **Sovereign control** | No dependency on the inactive original GetX repository — full control over patches, evolution, and Flutter SDK compatibility |
| **Foundation for specialization** | A lean core enables VR/XR routing, module-aware DI, and reactive profiling without carrying dead weight |

---

## Philosophy

GetX was built as a "do everything" framework. `neom_getx` is built as a "do the right things" framework.

**SINT** — State Manager, Dependency Injection, Navigation, Translations. Nothing more, nothing less.
