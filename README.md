Prefabricator for UE4 / UE5
=====================

Create Prefabs in Unreal Engine

Website: https://prefabricator.dev

---
See UE57_FIX.md for crash fix details.

## Unofficial UE 5.7 Fix

This build includes a fix for a crash in PrefabRandomizer when used with streamed levels
(e.g. Dungeon Architect / Snap Grid Flow).

### Root cause
Randomization was executed too early during BeginPlay, before all actors were fully initialized.

### Fix
Randomize() is deferred by one tick using:

SetTimerForNextTick()

This ensures safe execution after level streaming is complete.

### Result
- No crash
- Works with Snap Grid Flow modules
- No Blueprint Delay required

---