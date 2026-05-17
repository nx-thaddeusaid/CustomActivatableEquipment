# CustomActivatableEquipment — Performance Profiling

## Context

The activation state machine runs inside the BattleTech game process. Its hot paths are all in-game callbacks — impossible to isolate to a benchmark project without game DLLs. Profiling requires `dotnet-trace` against a live game session.

---

## dotnet-trace Profiling

### Prerequisites

```bash
dotnet tool install -g dotnet-trace
```

### Step 1 — Start a combat mission

Load a mission with mechs that carry activatable equipment (ECM, active probes, MASC, etc.). The more activatable components per mech, the more useful the trace.

### Step 2 — Find the BattleTech process

```bash
# Linux (via Steam/Proton or native)
PID=$(pgrep -f "BattleTech")

# Or: look for the Unity player
PID=$(pgrep -f "BattleTech.x86_64")
```

### Step 3 — Capture a profile during active combat

Start the capture, then cycle through 2–3 full rounds of combat including activating/deactivating equipment:

```bash
dotnet-trace collect --process-id $PID \
  --duration 00:01:00 \
  --profile cpu-sampling \
  --output cae-combat.nettrace
```

### Step 4 — Analyse

Open in SpeedScope or PerfView. Filter to `CustomActivatableEquipment` frames.

**Key methods to find:**

| Namespace/Class | Method | Why it matters |
|---|---|---|
| `CustomActivatableEquipment` | `ActivationStateChanged` | Called on every component state toggle |
| `ComponentRefInjector` | `InjectComponentRef` | Runs at mech loading time |
| `*Patch` classes | `Prefix`/`Postfix` | Harmony patches fire on every target method call |

---

## What to look for

- **Activation/deactivation calls:** If `ActivationStateChanged` appears high in the flame graph during combat, the state machine itself has a hot inner loop. Look for LINQ chains or repeated dictionary lookups.
- **Patch overhead:** Harmony patches intercept game methods. If any `Prefix`/`Postfix` appears for methods called every frame (e.g., `AbstractActor.OnActivationEnd`), that's a frame-rate risk.
- **Component iteration:** Search for `GetComponents` or `foreach` over component lists — these scale with mech component count.

---

## Targeted micro-benchmark (if you extract logic)

If a pure-logic helper (no game types) can be extracted, add it to a `CustomActivatableEquipment.Benchmarks` project using the same source-include pattern as `ModTek.Benchmarks`:

```xml
<Compile Include="../ActivatableEquipment/SomeHelper.cs" />
```

Then run with BenchmarkDotNet. See `../../ModTek/ModTek.Benchmarks/` for the pattern.
