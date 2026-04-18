# UE 5.7 PrefabRandomizer Crash Fix (Snap Grid Flow / Streaming Levels)

## Summary

A crash was identified when using `PrefabRandomizer` inside streamed levels,
especially with Dungeon Architect Snap Grid Flow (SGF) modules.

The issue has been fully isolated and resolved.

---

## Root Cause

`PrefabRandomizer` executes `Randomize()` too early during `BeginPlay`.

In streamed module levels (e.g. SGF), not all actors and components are fully initialized at that point.
This leads to undefined behavior and eventually a crash:

```text
Pure virtual not implemented ()
```

This is a classic Unreal lifecycle/timing issue — not a logic error in the randomizer itself.

---

## Fix
Randomize() is deferred by one tick using `SetTimerForNextTick()`

```cpp
#include "TimerManager.h"

void APrefabRandomizer::BeginPlay()
{
    Super::BeginPlay();

    if (bRandomizeOnBeginPlay)
    {
        const int32 Seed = FMath::Abs(SeedOffset + (int32)GetTypeHash(GetActorLocation()));

        GetWorldTimerManager().SetTimerForNextTick([this, Seed]()
        {
            if (IsValid(this))
            {
                Randomize(Seed);
            }
        });
    }
}
```
## Why This Works

`SetTimerForNextTick()` ensures execution happens after:

* Level streaming is completed
* Actors are fully initialized
* Components are properly registered

This is effectively equivalent to Delay(0.0) in Blueprint,
but implemented correctly and safely in C++.

## Result
* No crash anymore
* Works reliably with Snap Grid Flow modules
* No Blueprint Delay required
* No need for manual triggering
* Works with:
  * Explicit ActorsToRandomize
  * Empty list (randomizes entire level)
## Notes
* The issue only occurs in streamed/instanced levels (e.g. Dungeon Architect)
* Regular levels may not exhibit this problem
* This fix is minimal, safe, and does not change existing behavior
## Conclusion

The crash was caused by incorrect execution timing (engine lifecycle),
not by invalid logic in `PrefabRandomizer`.

## Optional

If needed, the exact patch can be provided as a pull request.