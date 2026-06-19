![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*hVKWuT24riZnx4VzDpQVJQ.png)

Debunk Your ViewModel Performance in the New AI Era
==================

> **AI Heroes Milan — 90 min hands-on workshop**
> [Kotzilla](https://kotzilla.io)

ViewModels are at the heart of modern Android and KMP apps, and they come with hidden performance
traps most teams discover too late: lifecycle mismanagement, scope leaks, bad memory patterns,
dependency-resolution overhead. These issues are hard to trace in production — and AI-assisted
development makes them worse: more code ships faster, abstractions multiply, and ViewModel problems
stay invisible until users are already impacted.

In this workshop we debug **real ViewModel performance scenarios** on a production-grade app, using
**Koin** and **Kotzilla** to detect issues at the component level — no manual instrumentation. Then
we bring an **AI agent** into the loop and show how DI-based observability turns it from a *code
generator* into a *diagnostic partner*.

**Thesis:** your DI graph (the software architecture) is the missing observability layer. It gives
an AI agent the architectural context it needs to diagnose, not just guess.

This app is a migrated version of Google's **[Now in Android](https://github.com/android/nowinandroid)**
where Dagger Hilt is replaced by **Koin** (Koin Compiler Plugin). It ships with intentionally
introduced performance bugs that you will detect and fix during the labs. The deep-dive on the
Hilt → Koin migration itself lives in [`docs/KOIN_COMPILER_PLUGIN.md`](docs/KOIN_COMPILER_PLUGIN.md).

---

## The toolbox

- **[Koin](https://insert-koin.io/)** — pragmatic, lightweight DI for Kotlin & KMP. Your DI graph is
  the architectural map we exploit all workshop long.
- **Koin Compiler Plugin** — gives you **compile-time safety** on the DI graph: missing or inconsistent
  definitions are caught at build time, not at runtime.
- **[Koin IntelliJ Plugin](https://plugins.jetbrains.com/plugin/10671-koin)** — **graphical navigation**
  of the Koin graph inside the IDE (jump between definitions, see scopes and dependencies), plus
  Kotzilla integration. Use it to *read* the architecture, not to find bugs for you.
- **[Kotzilla Platform](https://kotzilla.io/)** — runtime AI performance monitoring for Koin. It measures
  **every component** and its architecture behavior (resolution time, memory, scopes) **and screen
  rendering across Activity / Fragment / Compose** — surfacing crashes, **ANRs / slow screens**,
  main-thread/resolution issues, and a live memory graph. No manual instrumentation. *Quantify* what you
  felt.
- **[Kotzilla MCP server](https://mcp.kotzilla.io/mcp)** — exposes your DI graph + runtime data to an
  MCP-capable AI assistant. The bridge that makes AI diagnosis architecture-aware.

---

## ⚙️ Prerequisites

Setup is the #1 live-failure risk. **Please complete everything below at home** — we will not build
or sign up live (Now in Android is a 30-module app; first build is slow).

1. **Kotlin** intermediate level, plus basic **Android/KMP**, **ViewModel/Lifecycle**, and DI familiarity.
2. **Android Studio** installed with the **Android SDK** and a working **emulator or device** (the SDK can
   be pre-installed via Android Studio — make sure it's set up and an emulator boots).
3. **Clone this repo and run ONE successful Gradle sync + build at home** (`./gradlew :app:assembleDemoDebug`).
4. **Kotzilla account + app registration** (see *Setup* below) → paste your key into the
   `app/kotzilla.json` already included in the repo.
5. **Koin IntelliJ Plugin** installed in Android Studio.
6. **An MCP-capable AI assistant** (Claude Code, Cursor, Windsurf, or Android Studio) **connected to the
   Kotzilla MCP server** via the `mcp-remote` transport (endpoint `https://mcp.kotzilla.io/mcp`) and
   confirmed working — used in Lab 3. Config snippet + per-client paths:
   [doc.kotzilla.io/getstartedCustom/mcpSetup](https://doc.kotzilla.io/getstartedCustom/mcpSetup).

---

## Setup & Sign-up

1. [Sign up](https://console.kotzilla.io/signup) for a free Kotzilla account.
2. [Register your application](https://console.kotzilla.io/onboarding):
   - **Name:** `Now in Android`
   - **Package:** `com.google.samples.apps.nowinandroid.demo.debug`
   - **Type:** `Android Compose`
3. A **`kotzilla.json`** is already included in **`app/`**. In the onboarding, copy your generated key
   config and **paste it into `app/kotzilla.json`** (replacing the contents) — no need to move files
   around. The Koin & Kotzilla libraries are already wired into this project, so skip the remaining
   onboarding steps.

<img width="250" height="216" alt="kotzilla.json in the app folder" src="https://github.com/user-attachments/assets/ebc57a6a-63bb-40ce-9931-f126d4ad3bdd" />

4. Launch the app from Android Studio, then open the [Kotzilla console](https://console.kotzilla.io/)
   to see your first live session.

<img width="670" height="400" alt="First session in the Kotzilla console" src="https://github.com/user-attachments/assets/bc225d90-573a-4f92-81f6-cff0a4385758" />

---

## Run sheet (90 min)

| Duration | Block | What happens |
|----------|-------|--------------|
| 10 min | **Intro + frame** | Verify everyone's setup (`kotzilla.json` + plugin + emulator). ~5 min mobile-architecture / Koin DI recap. The core argument: **why the DI graph is the lever for AI-assisted diagnosis.** |
| 15 min | **Lab 1 — Spot & fix the crash** | Run the app, play with the **Interests crash**, analyze it in the Kotzilla console (stacktrace on `InterestsViewModel`), and fix it. First hands-on loop: run → reproduce → analyze → fix. |
| 35 min | **Lab 2 — Setup with Koin + Kotzilla** | The subtler issues, quantified in the console: slow screen / **ANR** on `MainActivity` → trace to its ViewModel → fix; slow startup (`NiaApplication`) → fix; background-thread blocking (`SyncWorker`) → fix; memory graph → single/factory churn → fix; the DI **scope leak** (retained scoped instances) → fix. |
| 20 min | **Lab 3 — AI in the loop (MCP)** | Connect the assistant to the Kotzilla MCP server and **explore the tools**: pull the issues Kotzilla detected on `main`, ask for **remediation** grounded in the DI graph + runtime, apply and verify. Payoff: code generator → diagnostic partner. |
| 10 min | **Wrap-up + Q&A** | Patterns to take home, the optional **bonus** exercises, Kotzilla/Koin pointers. |

---

> 🔐 **Solutions** for every exercise (including the bonus) live in [`SOLUTIONS.md`](SOLUTIONS.md). The
> whole point is to diagnose with Kotzilla + the Koin graph *first* — only open it to check your fix or if
> you're truly stuck.

## Lab 1 — Spot & fix the crash (run the app, play with the exception)

**Goal:** your first hands-on loop — *run → reproduce → analyze → fix*. Kotzilla is live from the first
launch, so the issue shows up in the console without any manual logging.

**Reproduce:** run the app and tap the **Interests** tab — it **crashes**. 💥 Try it a few times. (You'll
also notice the app takes a moment to appear on launch — that's a Lab 2 issue, park it for now.)

**Analyze:** open the [Kotzilla console](https://console.kotzilla.io/) and find the **Crash** issue
linked to `InterestsViewModel`, with its full stacktrace.
> Tip: the Koin IntelliJ plugin lets you jump straight to the `InterestsViewModel` definition and see
> what it resolves — handy for navigating from the issue to the code.

<img width="250" height="550" alt="Interests crash" src="https://github.com/user-attachments/assets/c1c3a666-0e60-4afd-a552-c5d48fbd8eeb" />

**Your task:** open `InterestsViewModel` (the stacktrace points right at it) and find what it does on
construction that throws. Remove it, rebuild, and reopen Interests — no crash, and the issue stops
showing in the console. ✅

---

## Lab 2 — Setup with Koin + Kotzilla

**Goal:** with Kotzilla live, turn the subtler symptoms into **measured, component-level issues** in the
[console](https://console.kotzilla.io/) — resolution time, the memory graph, retained scoped instances —
then fix each. Still no manual instrumentation.

### Reading the console — two ways in

Kotzilla gives you two complementary entry points, and Lab 2 uses both:

- **Slow Screens → Screen Details** — the *user-facing* view, on the **Dashboard**. The **Slow Screens**
  table lists each screen with P50/P95 render times, sessions, and **ANR** counts; click **Analysis** to
  open **Screen Details**, which shows the screen's **Top navigation patterns** (the user flows it appears
  in — Activities, Fragments, and Compose Navigation) and the **Top blocking UI components** (the Koin
  components resolved during rendering, ranked by resolution time and linked to their issues). Start here
  when the symptom is something a user *feels* — a frozen screen or an ANR (→ 2.1). *(ANR threshold = 2s.)*
- **Issues View** — the *component-level* view: **Critical Issues** (crashes, startup, main-thread and
  background-thread performance) and **All Issues** (adds architectural / memory issues). Each issue shows
  its description, impacted sessions, affected versions, status, and P50/P95 resolution times, and links
  straight to the offending component.

From either one, open a session's **Timeline View** — including the **Memory Graph** (components *in
memory*, *created*, and *open scopes*) — which you'll read in 2.4 and 2.5.

<img width="1961" height="798" alt="Kotzilla console — Screens/Flows and Issues" src="https://github.com/user-attachments/assets/9526b595-8909-48d8-98c2-292767d4d8a7" />

> 🏗️ **Beyond the console:** the **Kotzilla Gradle plugin** also prints a PASS/WARN/FAIL report at build time (`kotzillaBuildReport{Variant}`), so the same issues surface in your **CI/CD** — and can be made a hard build gate with `skipBuildReportFailure = false`.

## 2.1 — Slow screen & ANR (main-thread blocking)

**Why it matters:** a screen only renders as fast as its main thread. Blocking work there — whether in an
**Activity**, a **Fragment**, or a **Compose** screen — freezes the UI, and past a few seconds Android
raises an **ANR** (*Application Not Responding*). Kotzilla measures **screen rendering time and ANRs**
across all three, so you start from the user-visible symptom and drill down to the cause.

**Symptom:** the **MainActivity** screen hangs on launch — it blocks the main thread long enough (~2.5s)
to trigger an **ANR**.

**Detect:** on the **Dashboard**, find `MainActivity` in the **Slow Screens** table (high P95 + an **ANR**
count). Click **Analysis** → **Screen Details** → **Top blocking UI components**: the block is in
resolving **`MainActivityViewModel`** on the main thread (component resolution runs on the main thread by
default). Kotzilla walks you from the screen straight to the component behind it.

<img width="1961" height="798" alt="Main thread / ANR on MainActivity, traced to MainActivityViewModel" src="https://github.com/user-attachments/assets/fad7b881-21be-4a2a-b12f-1b3585bfbbe0" />


**Your task:** follow the trail from the ANR on the `MainActivity` screen to `MainActivityViewModel`, find
the blocking work it runs while being constructed, and remove it. Confirm the screen renders fast and the
ANR / resolution-time issue is gone in the console.

## 2.2 — Slow app startup (`NiaApplication`)

**What it is:** blocking work on the main thread during `Application.onCreate` delays the first usable
screen — a slow cold start that every user pays on launch.

**Detect:** the **Dashboard** shows elevated **Cold / Warm Startup** (P50/P95), and the **Issues View** has
a **Slow Startup** issue pointing at `NiaApplication`.

**Your task:** open `NiaApplication` (`app/.../NiaApplication.kt`), remove the blocking work in `onCreate`,
and confirm startup time drops. Tip: explore `trace { }` to measure your own startup spans.

<img width="400" height="400" alt="Manual tracing with the trace function" src="https://github.com/user-attachments/assets/61022e17-0566-4e0c-882b-7f2e9ed4ffcc" />

## 2.3 — Background-thread blocking (`SyncWorker`)

**What it is:** the data sync runs heavy synchronous work on its background (IO) thread. It doesn't freeze
the UI, but it ties up the worker far longer than it should — wasting resources and delaying sync.

**Detect:** in the **Issues View**, find the **Background Thread** issue on the sync worker
(`SyncWorker` / `androidx.work.ListenableWorker`).

**Your task:** open `SyncWorker` (`sync/work/.../SyncWorker.kt`), find the heavy synchronous work in
`doWork`, and remove it. Confirm the background-thread issue clears.

## 2.4 — Memory leak patterns: single vs factory

**What it is:** the data layer is declared with **`@Factory`**, so Koin **recreates** these components on
every resolution instead of keeping one in memory — wasting memory and CPU. Because repositories are
resolved by every ViewModel, the churn is significant.

**Detect:** open the latest session's **Timeline View → Memory Graph** and compare *created* vs *in
memory* counts for the data-layer components — the *created* count climbs.

<img width="731" height="304" alt="image" src="https://github.com/user-attachments/assets/15be03a6-0624-40e4-85cd-ac8821819f63" />


**Your task:** find the data-layer components (DAOs, repositories, network data source) declared with the
wrong lifetime and give them the right one, so a single instance is kept in memory instead of recreated
on every resolution. Re-run and compare the memory allocation + resolution times before/after. 🎉

> Hint: stateless use cases in `core/domain` being `@Factory` is *correct* — focus on the components
> that hold/serve data.

## 2.5 — DI scope leak / lifecycle mismanagement ⭐

**What it is:** a classic scope leak. `MainActivity` opens a Koin **Activity scope** (`activityScope()`),
and `JankStats` is an **`@ActivityScope`** instance that should die with the Activity. But something
longer-lived holds on to it — an application-level **`@Singleton`** keeps a reference, so it outlives the
Activity scope. Every Activity recreation then leaves its activity-scoped instance — and the Activity
window it tracks — retained forever. *A longer-lived scope is holding a shorter-lived one* (finding
*which* component is your task — use the DI graph).

**Symptom:** rotate the screen (or otherwise recreate the Activity) a few times. The Koin Activity scope
closes each time, but its instances are never released.

<img width="720" height="717" alt="image" src="https://github.com/user-attachments/assets/c871cad6-08a1-4edf-88b6-5951adba4ee8" />


**Detect:** in the **Memory Graph** (Timeline View), watch the **open scopes** / retained scoped instances
climb across recreations and never come back down — the scope was closed, but its objects are still alive.

**Your task:** find *what* is holding the activity-scoped instance beyond the scope's life. Use the DI
graph (who depends on / holds what) to locate the longer-lived component that keeps a reference to a
shorter-lived scoped one, and break that link. Re-run, rotate, and confirm the retained count no longer
climbs.

> This is a great case for Lab 3: a leak that only makes sense once you can see *both* the DI graph
> (who holds what, in which scope) *and* the runtime (which instances are retained).

---

## 🎁 Bonus — Function tracing with `@Monitor`

**Optional, and not a bug to fix** — this one shows off Kotzilla's *tracing capability*: turn on
function-level tracing with a single annotation.

`feature/foryou/.../ForYouViewModel.kt` ships with `@Monitor` (and its import) **commented out**.
Uncomment both. `@Monitor` is a **Koin Compiler Plugin** feature — at compile time it wraps every public
function of the class with Kotzilla trace calls, so you get per-function timings with *zero* manual
`trace { }`.

Re-run, interact with the **For You** screen (follow a topic, bookmark, scroll), and watch the traced
functions appear in Kotzilla. You can also annotate a single function instead of the whole class.
*(Requires the Kotzilla SDK on the classpath — already set up here.)*

---

## Lab 3 — AI in the loop (MCP)

**Goal:** connect your AI assistant to the **Kotzilla MCP server** and let it pull everything Kotzilla
detected — the DI graph, runtime instances, sessions, and issues — then ask it for **remediation**. The
point: with architectural + runtime context on tap, the assistant goes from *code generator* to
*diagnostic partner*.

**Setup:**
- Run the app **on the `main` branch (all bugs present)** and exercise it so Kotzilla detects the full
  set of issues in your session — the crash, the main-thread resolution, the factory memory churn, and
  the DI scope leak from Lab 2.
- Your MCP-capable assistant (Claude Code / Cursor / Windsurf / Android Studio) connected to the Kotzilla
  MCP server (do this **at home** — see prerequisites). The endpoint is `https://mcp.kotzilla.io/mcp`,
  added via the `mcp-remote` transport:

```json
{
  "mcpServers": {
    "kotzilla": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.kotzilla.io/mcp"]
    }
  }
}
```

  **Claude Code** — add it in one command instead of editing the JSON:

```bash
claude mcp add kotzilla -- npx -y mcp-remote https://mcp.kotzilla.io/mcp
```

  Append `-s user` to make it available across all your projects:
  `claude mcp add -s user kotzilla -- npx -y mcp-remote https://mcp.kotzilla.io/mcp`.
  On first use the server runs a browser auth flow with your Kotzilla account.

**The tools at your disposal.** The Kotzilla MCP server exposes (among others — the registration/setup
tools aren't needed here since the app is already configured):

| Tool | What it gives you |
|---|---|
| `get_issues` | the performance & architecture issues detected in your sessions, ranked by impact |
| `get_issue_context` | the deep dive: dependency resolution tree + timings, blocking components, stack traces, which thread |
| `get_screen_performance` | screen rendering times & ANR counts |
| `get_performance_guidance` | fix patterns / best practices for a given issue type |
| `generate_report` | a compact pass/warn/fail performance summary |

**Explore — drive your own investigation.** Natural language is enough; the assistant picks the right
tools. Prompts to get going (from the [Kotzilla MCP docs](https://doc.kotzilla.io/discover/mcpServer)):

- *"What are the main performance issues in my app?"* → ranked list via `get_issues`
- *"Why is it slow?"* / *"What's causing this issue?"* → `get_issue_context`: resolution tree, timings,
  the blocking/retained components, on which thread
- *"How can I fix this issue?"* → `get_performance_guidance` + the assistant edits your code
- *"Generate a report for my app"* → `generate_report`

Pick an issue (the DI scope leak is the richest), ask the agent to diagnose it through the MCP tools,
apply the remediation it proposes, re-run, and confirm in the console that it's gone.

**Payoff:** because the agent can query the DI graph + runtime through MCP, its remediation is specific
and evidence-grounded — it names the offending component and scope instead of guessing. Architectural +
runtime context is the difference between *guessing* and *diagnosing*.

> 💡 Optional contrast: ask the same "why is this leaking?" question **without** MCP first (paste only the
> code) — notice how generic the answer is — then again **with** MCP connected.

---

## Wrap-up — everything we saw

### The traps, and what each one taught

- **Crash on resolution** (Lab 1) — a throwing `init` surfaces as a component crash whose stacktrace
  points straight at the ViewModel. The DI container is where the failure becomes visible.
- **Main-thread blocking → ANR** (2.1) — heavy work during construction/resolution freezes the screen
  (Activity, Fragment, *or* Compose). **Component resolution runs on the main thread by default**, so keep
  `init`/construction cheap.
- **Slow app startup** (2.2) — blocking in `Application.onCreate` delays the first usable screen on every
  cold start.
- **Background-thread blocking** (2.3) — off-UI work still hurts: heavy `doWork` ties up the worker and
  delays sync.
- **`single` vs `factory`** (2.4) — the wrong DI scope means needless re-creation: memory + CPU churn
  (*creating* climbs vs *in memory*). Choose component lifetimes deliberately.
- **DI scope leak / lifecycle mismanagement** (2.5) — a longer-lived scope (a `@Singleton`) holding a
  shorter-lived one (`@ActivityScope`) retains instances forever. Let each scope own and release its own
  dependencies.
- **`@Monitor` tracing** (bonus) — one annotation gives per-function timings, generated at compile time —
  no manual `trace { }`.

### The tooling, layered

- **Koin Compiler Plugin** → compile-time safety on the graph, *and* the `@Monitor` tracing proxy.
- **Koin IDE plugin** → read & navigate the DI graph (scopes, dependencies) without running anything.
- **Kotzilla** → runtime truth with no manual instrumentation: components & resolution timing, **screens &
  ANRs** across Activity/Fragment/Compose, the **Memory Graph** (instances + open scopes). Console:
  *Slow Screens / Screen Details*, *Issues View*, *Timeline View*.
- **Kotzilla MCP** → feeds that DI graph + runtime data to your AI assistant.

### The thesis to take home

- **The DI graph is your observability layer.** Scoping and resolution timing are *architectural facts* —
  make them measurable, and component-level problems stop being invisible.
- **Feel → measure → diagnose.** Running the app surfaces the symptom, Kotzilla quantifies it at the
  component level, and AI + MCP closes the loop with architectural context — turning the assistant from a
  *code generator* into a *diagnostic partner*.

---

# License

**Now in Android** is distributed under the terms of the Apache License (Version 2.0). See the
[license](LICENSE) for more information. Original project readme: [`README.original.md`](README.original.md).
