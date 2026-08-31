# ReaScript API

Requires ReaSmoothPlayhead v0.2.6 or newer.

A script can put the playhead on one of the presets saved in the settings
window and hand the look back when it is done.

What a script applies is laid **over** the user's settings and never written
into them: it does not survive a REAPER restart, and the settings window goes
on showing and saving the user's own style throughout. While a preset is
applied the settings window says so and offers a "Clear applied preset" button,
so the user can take the look back at any time.

## SmoothPlayhead_ApplyPreset

```
boolean SmoothPlayhead_ApplyPreset(string name)
```

Show a saved preset until `SmoothPlayhead_ClearAppliedPreset`. Returns `false` if
there is no preset by that name.

```lua
reaper.SmoothPlayhead_ApplyPreset("Loop")
```

## SmoothPlayhead_ClearAppliedPreset

```
void SmoothPlayhead_ClearAppliedPreset()
```

Go back to the user's own settings.

```lua
reaper.SmoothPlayhead_ClearAppliedPreset()
```

## SmoothPlayhead_PresetExists

```
boolean SmoothPlayhead_PresetExists(string name)
```

True if a preset by this name exists.

```lua
if not reaper.SmoothPlayhead_PresetExists("Loop") then
  reaper.ShowMessageBox("Save a preset named \"Loop\" first.", "Loop color", 0)
  return
end
```

## SmoothPlayhead_CountPresets

```
integer SmoothPlayhead_CountPresets()
```

How many presets the user has saved.

## SmoothPlayhead_GetPresetName

```
boolean retval, string name = SmoothPlayhead_GetPresetName(integer index)
```

Preset name at this index, counting from 0.

```lua
for i = 0, reaper.SmoothPlayhead_CountPresets() - 1 do
  local _, name = reaper.SmoothPlayhead_GetPresetName(i)
  reaper.ShowConsoleMsg(name .. "\n")
end
```

## Example: a different color while looping

Needs a preset named `Loop` saved in the playhead settings. `atexit` matters —
without it the preset stays applied until REAPER restarts or the user clears it.

```lua
if not reaper.APIExists("SmoothPlayhead_ApplyPreset") then
  reaper.ShowMessageBox("Needs ReaSmoothPlayhead v0.2.6 or newer.", "Loop color", 0)
  return
end
if not reaper.SmoothPlayhead_PresetExists("Loop") then
  reaper.ShowMessageBox("Save a preset named \"Loop\" first.", "Loop color", 0)
  return
end

local applied

function tick()
  local looping = reaper.GetSetRepeatEx(0, -1) == 1
  if looping ~= applied then
    applied = looping
    if looping then
      reaper.SmoothPlayhead_ApplyPreset("Loop")
    else
      reaper.SmoothPlayhead_ClearAppliedPreset()
    end
  end
  reaper.defer(tick)
end

reaper.atexit(reaper.SmoothPlayhead_ClearAppliedPreset)
tick()
```
