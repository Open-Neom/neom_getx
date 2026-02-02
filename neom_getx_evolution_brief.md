# 🛡️ Technical Brief: Evolution of neom_getx within the Open Neom Ecosystem

## 1. Context & Vision
* **Stakeholder:** Serzen (Founder & Architect of the SRZNVERSO).
* **Core Objective:** Establish `neom_getx` as the sovereign, stable, and high-performance standard for state management, replacing the inactive original GetX repository.
* **Ecosystem Scale:** 21+ modular libraries (neom_libraries) supporting complex apps like **Cyberneom** (Neuroscience/Health), **EMXI**, and **Gigmeout**.

## 2. Current Implementation Strategy
* **Temporary State:** Currently utilizing `dependency_overrides` in root applications to point `get` to the local path of `neom_getx`.
* **Architecture:** Transitioning to a Monorepo structure where shared logic resides in `neom_modules/` and entry-points in `apps/`.
* **Immediate Fixes:** Initial patches applied to resolve Null Safety inconsistencies in Web/Chrome (e.g., `webClientId` null errors).

## 3. Engineering Roadmap for Local AI

### Phase A: Global Migration (The Hard Fork)
* **Goal:** Complete renaming from `package:get` to `package:neom_getx`.
* **Action:** Perform a recursive refactor of all `.dart` imports across 21 modules.
* **Stability:** Update all `pubspec.yaml` files to transition from unstable `-dev` versions to a controlled `neom_getx: 1.0.0`.

### Phase B: Strategic Pruning ("The Poda")
* **Retention Policy:** Keep only **State Management (Rx)**, **Dependency Injection (DI)**, **Navigation**, and **Translations**.
* **Elimination Policy:** Remove generic utilities, string validations, and UI extensions that overlap with `neom_commons`.
* **Result:** A lighter, faster core focused on the high-performance requirements of neuroscience research (Cyberneom).

### Phase C: Path to the Standard
* **Performance:** Ensure `neom_getx` is the most efficient choice for Flutter Web, iOS, and Android.
* **Maintenance:** Monitor Flutter SDK updates (3.19.0+) to ensure zero downtime or breaking changes within the ecosystem.

## 4. Specific Prompt for Local AI / Agent
> "Analyze the provided `neom_modules` directory. Your primary mission is to automate the package renaming process. Following that, conduct a static analysis of `neom_getx` to identify dead code or redundant utilities not utilized by the 21 existing modules. Propose a surgical deletion plan to optimize the framework while ensuring the stability of its reactive core."