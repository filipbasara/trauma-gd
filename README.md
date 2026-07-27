![Trauma Logo](https://github.com/filipbasara/trauma-gd/blob/main/Trauma.png "Trauma Logo")


# Trauma - 2D Camera Shake Plugin for Godot

## Requirements
Godot 4.4 or newer. The plugin references its resources by `uid://`, which earlier
versions don't resolve.

## Installation
### From the Asset Library (in-editor)
1. Open the **AssetLib** tab and search for `Trauma`.
2. Click **Download**, then **Install**. Keep the default target so the files land
   in `res://addons/trauma/`.
3. Enable it under **Project > Project Settings > Plugins**.

### Manual
1. Download the [latest release](https://github.com/filipbasara/trauma-gd/releases)
   or clone the repo.
2. Copy the `addons/trauma/` folder into your project's `addons/` directory, so you
   end up with `res://addons/trauma/plugin.cfg`.
3. Reopen the project, then enable the plugin under
   **Project > Project Settings > Plugins**.

## Usage:
1. Add a `CameraShake` node as a child of your Camera2D.
2. Call `shake(profile)` with any ShakeProfile resource.
3. Or call `add_trauma(0..1)` for cumulative effects (e.g. per bullet hit).

> [!NOTE]
> `Camera2D.ignore_rotation` is on by default in Godot 4, and it makes the camera
> discard its own rotation. Turn it off if you want a profile's `rotation_intensity`
> to show up, otherwise you only get the offset shake. `CameraShake` pushes a
> warning the first time you shake a rotating profile on such a camera.

## Trauma system:
- `shake_amount = trauma ^ profile.trauma_exponent`
- trauma decays at `profile.decay_rate` per second.
- Traumas from multiple calls stack (capped at `1.0`).

## Example Setup
### Example.tscn
Attach to a Node2D that has a Camera2D child with a CameraShake child.
```gd
Scene tree:
  Node2D  ← Example.gd
  └── Camera2D
      └── CameraShake
```
### Example.gd
```gd
extends Node2D

@onready var shake: CameraShake = $Camera2D/CameraShake

# Presets loaded from files.
var preset_explosion: ShakeProfile = preload("res://addons/trauma/presets/explosion.tres")
var preset_earthquake: ShakeProfile = preload("res://addons/trauma/presets/earthquake.tres")
var preset_impact: ShakeProfile    = preload("res://addons/trauma/presets/impact.tres")
var preset_jitter: ShakeProfile    = preload("res://addons/trauma/presets/jitter.tres")
var preset_rumble: ShakeProfile    = preload("res://addons/trauma/presets/rumble.tres")

func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("ui_accept"):   # Space / Enter
		shake.shake(preset_explosion)
	elif event.is_action_pressed("ui_up"):
		shake.shake(preset_earthquake)
	elif event.is_action_pressed("ui_right"):
		shake.shake(preset_impact)
	elif event.is_action_pressed("ui_down"):
		shake.shake(preset_jitter)
	elif event.is_action_pressed("ui_left"):
		shake.shake(preset_rumble)
```

## Extending the API
### Extended.gd
```gd
extends Node2D

@onready var shake: CameraShake = $Camera2D/CameraShake

# You can also compose shakes or add trauma gradually, e.g. per bullet hit:
func _on_bullet_hit() -> void:
	shake.add_trauma(0.25)  # stacks up; 4 hits = full trauma

# Or build a profile in code at runtime:
func _custom_shake_example() -> void:
	var p := ShakeProfile.new()
	p.intensity = 50.0
	p.rotation_intensity = 3.0
	p.decay_rate = 4.0
	p.frequency = 30.0
	p.initial_trauma = 0.9
	shake.shake(p)
```
