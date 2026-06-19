# AI Heroes Milan — Workshop Plan & Build Context

> Drop this at the repo root as `CLAUDE.md` (auto-loaded by Claude Code at session start),
> or keep it in `docs/` and `@docs/workshop-plan.md` from CLAUDE.md.
> Doc kept in English to match the repo — switch to FR if you prefer.

## Workshop

**"Debunk Your ViewModel Performance in the New AI Era"** — 90 min, hands-on.

Built on the droidcon Berlin 2025 workshop (this repo): reuses the Now in Android sample
(Hilt → Koin) + Kotzilla Platform + Koin IntelliJ plugin. Adds an MCP lab as the net-new,
differentiating piece.

**Thesis:** the DI graph (software architecture) is the missing observability layer that
turns an AI agent from *code generator* into *diagnostic partner*.

## What changed vs Berlin

- Reframed specifically around **ViewModel** performance (title promises scope leaks,
  lifecycle mismanagement, resolution overhead, memory patterns).
- 5 flat fix-exercises → **3-lab arc** (kept close to Berlin's hands-on flow): Lab 1 spot & fix the crash → Lab 2 instrument the subtler issues with Koin + Kotzilla → Lab 3 AI (MCP). No standalone "static analysis" lab — Kotzilla is live from the first launch.
- Curated to VM-centric exercises; `SyncWorker` + startup moved to take-home bonus.
- **New exercise**: a real ViewModel scope leak (delivers the missing "scope/lifecycle" promise).
- **New Lab 3**: AI-in-the-loop via the Kotzilla MCP server.

## Run sheet (90 min — hard cap)

Setup is the #1 live-failure risk → pushed entirely to **pre-event prerequisites**.
In-room intro is *verify + frame only*. No live signup.

| Time      | Block                          | Content |
|-----------|--------------------------------|---------|
| 0–10      | **Intro + frame** (10 min)    | Quick setup check (everyone has `kotzilla.json` + plugin + emulator running). ~5 min mobile archi / Koin DI recap. The core argument: **why architecture (the DI graph) is the lever for AI-assisted diagnosis.** |
| 10–25     | **Lab 1 — Spot & fix the crash** (15 min) | Berlin's opener, kept faithful (trimmed from 20): run the app, play with the **Interests crash**, analyze it in the Kotzilla console (stacktrace on `InterestsViewModel`), fix it. Run → reproduce → analyze → fix. |
| 25–60     | **Lab 2 — Setup with Koin + Kotzilla** (35 min) | Five detect-then-fix exercises, quantified in the console: 2.1 slow screen / ANR (`MainActivityViewModel`); 2.2 slow startup (`NiaApplication`); 2.3 background-thread (`SyncWorker`); 2.4 single/factory memory churn; 2.5 the **DI scope leak**. 2.2/2.3 are quick; time goes to 2.1, 2.4, 2.5. |
| 60–80     | **Lab 3 — AI in the loop (MCP)** (20 min) | Assistant connected to the Kotzilla MCP server. Run on `main` (all bugs present) so Kotzilla detects everything, then **explore the MCP tools** to ask for remediation grounded in the DI graph + runtime; apply + verify. Payoff: code generator → diagnostic partner. |
| 80–90     | **Wrap-up + Q&A** (10 min)    | Patterns to take home, the 🎁 **bonus** (`@Monitor` tracing), Kotzilla/Koin pointers. |

## Exercise mapping (Berlin → new arc)

| Berlin exercise | Maps to | VM-themed? |
|---|---|---|
| 1. Crash `InterestsViewModel` | Lab 2 warm-up (proves console works) | ✅ |
| 2. Main thread `MainActivityViewModel` | Lab 2 (resolution overhead) | ✅ |
| 5. Single vs Factory (`dataKoinModule`, `daosModule`, `flavoredNetworkModule`) | Lab 2 (memory patterns, memory graph) | ⚠️ data layer, kept |
| **NEW. ViewModel scope leak** | Lab 2 + **Lab 3 hero case** | ✅ |
| 3. `SyncWorker` background | Take-home **bonus** (off-theme) | ❌ |
| 4. `NiaApplication` slow startup | Take-home **bonus** (off-theme) | ❌ |

## NEW exercise — ViewModel scope leak (to build)

Goal: demonstrate the lifecycle mismanagement / scope leak the abstract promises.

**Inject (primary):** in a feature ViewModel's `init {}`, collect a long-lived flow on
`GlobalScope` instead of `viewModelScope` — the coroutine survives the ViewModel.

```kotlin
// BUG: collection outlives the ViewModel → leak
init {
    GlobalScope.launch {
        someRepository.longLivedStream().collect { /* update state */ }
    }
}
```

Pick a feature VM **not already used** by another exercise (e.g. `ForYouViewModel` or
`BookmarksViewModel`; avoid `InterestsViewModel` — owned by the crash).

**Symptom:** navigate in/out of the screen repeatedly → collectors accumulate, ViewModel
instances stay retained.

**Fix:**

```kotlin
init {
    viewModelScope.launch {
        someRepository.longLivedStream().collect { /* update state */ }
    }
}
```

**Kotzilla surface:** memory graph should show ViewModel instances that aren't released
across navigation (count climbs) — same view used by the single/factory exercise, so it's
visually consistent.
**⚠️ VERIFY when building:** confirm exactly how the retained instances render in the
console, and tune the flow so the leak is unmistakable in a single session.

**Alternative (Koin-native)** if the coroutine version is fiddly: open a Koin scope tied to
the screen and never close it → retained scoped instances.

## Lab 3 — MCP flow (open exploration)

- **Server:** https://mcp.kotzilla.io/mcp
- **Client:** Claude Code or Cursor (MCP-capable), connected + tested *at home*.

**Approach (decided):** not a scripted round1/round2. Attendees run the app on **`main` (all bugs
present)** so Kotzilla detects the full issue set in their session, then **explore the MCP tools**
themselves to ask for **remediation** — pick an issue, have the agent diagnose it via the DI graph +
runtime, apply the fix, re-run, confirm in console. README ships starter prompts; the no-MCP vs
with-MCP contrast is offered as an *optional* aside, not the main flow.

**Payoff line:** architectural + runtime context = the difference between guessing and diagnosing.

**TODO before the room (nice-to-have):** connect to the live server and note the actual tool names so the
starter prompts in the README point at real tools (exploration works regardless).

## Pre-event prerequisites (email to attendees)

Published list **+** the critical additions (do **NOT** do these live):

- Kotzilla free account; register the app (Name `Now in Android`, package
  `com.google.samples.apps.nowinandroid.demo.debug`, type `Android Compose`); download
  `kotzilla.json` → drop in `app/`.
- Koin IntelliJ plugin installed.
- Repo cloned + **one successful Gradle sync/build at home** (NiA is heavy — never build live).
- Android Studio + emulator or device ready.
- MCP-capable AI assistant (Claude Code or Cursor) installed and connected to
  https://mcp.kotzilla.io/mcp, tested.
- *(existing)* Kotlin (intermediate), Android/KMP familiarity, ViewModel/Lifecycle basics,
  a DI framework used in at least one project.

## Build state (as of 2026-06-19)

**Key discovery:** `workshop/` is a *clean Koin-**annotations** migration* of NiA, NOT the Berlin
codebase. Two consequences:
- DI uses `@KoinViewModel` / `@Single` / `@Factory` / `@Module` / `@ComponentScan` (no manual
  `org.koin.dsl.module { }`). Berlin's `dataKoinModule`/`daosModule`/`flavoredNetworkModule` names
  don't exist; equivalents are `DataKoinModule` (`@ComponentScan`), `DaosKoinModule` (`@Single` funcs),
  `FlavoredNetworkKoinModule` (`@ComponentScan`). Component-scanned classes use `javax.inject.Singleton`;
  module functions use `org.koin.core.annotation.Single`.
- **All Berlin bugs were absent** → re-injected into the annotation codebase (below).

**Bugs injected (Lab 1 + Lab 2):**

| # | File | Bug | Fix |
|---|---|---|---|
| 1 | `feature/interests/.../InterestsViewModel.kt` | `init { makeItCrash() }` → `error(...)` | remove `init`/`makeItCrash` |
| 2 | `app/.../MainActivityViewModel.kt` | `init { onPostInit() }` → `blockingThreadIssue(2500)` → **ANR / slow screen on `MainActivity`** (start from the screen symptom, drill to the VM) | remove blocking call |
| 3 | Data layer (see below) | full data layer flipped to factory | back to `@Singleton` |
| 4 | `app/.../di/JankStatsRegistry.kt` (plain class) + `@Singleton` provider in `app/.../di/JankStatsKoinModule.kt` + `app/.../MainActivity.kt` | DI **scope leak**: app-level singleton retains `@ActivityScope` `JankStats` → Activity scope can't release across recreation | remove the `@Singleton jankStatsRegistry()` provider + the `register()` call |

**Bug #3 (single/factory) — full data-layer scope (matches Berlin):**
- `core/database/.../di/DaosKoinModule.kt`: 5 DAO providers `@Single` → `@Factory`.
- `core/data/.../repository/*`: 6 repos `@Singleton` (javax) → `@Factory` (Koin) —
  `OfflineFirstUserDataRepository`, `OfflineFirstNewsRepository`, `OfflineFirstTopicsRepository`,
  `DefaultSearchContentsRepository`, `DefaultRecentSearchRepository`, `CompositeUserNewsResourceRepository`.
- `core/network/.../demo/DemoNiaNetworkDataSource.kt`: `@Singleton` (javax) → `@Factory` (Koin).
- Fix = revert all to **`@Singleton`** (preferred — Koin's `single` definition in the standard DI naming,
  alias of `@Single`; `javax.inject.Singleton` also works). Use cases stay `@Factory` (normal, not the bug).
  DataStore stays single, so multiple repo instances share one store → safe, just memory churn.

**README Lab 2 order (5 exercises):** 2.1 slow screen / ANR (`MainActivityViewModel`), **2.2 slow startup**
(`NiaApplication`), **2.3 background-thread** (`SyncWorker`), **2.4 single/factory** (Memory Graph), **2.5
DI scope leak ⭐** (Memory Graph). Then a separate **🎁 Bonus** = the tracing capability. (2.4 + 2.5
grouped as the two Memory-Graph exercises.)

- `app/.../NiaApplication.kt` (§2.2): `blockingThreadIssue(6_500)` in `onCreate` → slow-startup issue.
- `sync/work/.../SyncWorker.kt` (§2.3): `blockingThreadIssue(1500)` at start of `doWork` → background-thread issue.
- `feature/foryou/.../ForYouViewModel.kt` (🎁 **Bonus = tracing capability**, try-it, not a bug):
  `@Monitor` + its import are in the file, **commented out** — attendees uncomment to enable. `@Monitor`
  (`org.koin.core.annotation.Monitor`) is a Koin Compiler Plugin feature that wraps the class's public
  functions with Kotzilla trace calls at compile time (verified in `koin-compiler-plugin`). Additive.

Helper recreated: `core/data/.../model/Issue.kt` (`Issues.blockingThreadIssue`, etc.) — also reusable
for the SyncWorker / startup **bonus** exercises.

**Decisions locked — scope leak = DI scope, not coroutine:** the abstract's "scope leak" means a **Koin
DI scope** leak (the app already has an Activity-scope archetype), not a coroutine `GlobalScope` leak.
Implemented as the **singleton-retains-Activity-scope** variant: `JankStatsRegistry` is declared
`@Singleton` (via a provider in `JankStatsKoinModule`) and holds the `@ActivityScope` `JankStats` that
`MainActivity` registers in `onCreate` → retained across Activity recreation (rotation). Fix = drop the
singleton provider + registration. This also delivers the abstract's *lifecycle mismanagement* promise,
and is the **Lab 3 MCP hero case**.

The earlier `TopicViewModel` main-thread block was **reverted** — it duplicated `MainActivityViewModel`
(resolution overhead) and added a redundant step. Net-new is now exactly: the DI scope leak + Lab 3 MCP,
keeping the workshop a reuse of the Berlin set plus MCP as the headline (per scope decision).

> ✅ Content gap resolved: crash, main-thread/resolution, single/factory memory, **and** the DI scope
> leak / lifecycle exercise now cover the abstract's promises.

**⚠️ Not build-verified here** (NiA too heavy to build in this env). Needs a Gradle sync + run on the
emulator to confirm: KSP picks up `@Factory`, the crash/block fire, and Kotzilla surfaces all three.

## Open TODOs

- [x] ~~Decide which feature VM hosts the leak.~~ → `BookmarksViewModel`.
- [x] ~~Re-inject Berlin core bugs into the annotation codebase~~ (crash, main-thread, single/factory).
- [ ] Build + run once on emulator: verify all **6** injected bugs (4 core + 2 bonus) fire and render in the Kotzilla console.
- [ ] Run spotless (`./gradlew --init-script gradle/init.gradle.kts spotlessApply`) — the data-layer edits may need import reordering.
- [ ] Decide the scope-leak / lifecycle promise: add a dedicated exercise, or reframe the abstract (see content gap above).
- [x] ~~Lab 3 design~~ → open MCP-tool exploration on `main` (all bugs present); README has starter prompts.
- [x] ~~Note real MCP tool names + refine README prompts~~ → done from official docs
      (`doc.kotzilla.io/discover/mcpServer`). Tools: `get_issues`, `get_issue_context`,
      `get_screen_performance`, `get_performance_guidance`, `generate_report` (+ setup tools `list_apps`,
      `create_app`, `guide_sdk_installation`, `generate_app_config`, `get_platform_activity`). README Lab 3
      + prereq #6 updated with the `mcp-remote` config and doc-aligned prompts.
- [x] ~~`app/kotzilla.json` keys~~ → **Arnaud owns this** (template vs shared key, secret handling). README
      says "paste your key into the included `app/kotzilla.json`".
- [x] ~~Trim README into a workshop guide reflecting the 3-lab arc~~ → new `README.md` (3-lab arc + bonus section); old migration doc preserved at `docs/KOIN_COMPILER_PLUGIN.md`.
- [ ] Draft the pre-event email with the expanded prerequisites.