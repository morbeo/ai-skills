# Troubleshooting Reference

Common Godot errors and how to fix them. Organized by symptom.

---

## Table of Contents

1. [Null Reference / Node Not Found](#null-reference--node-not-found)
2. [Character Doesn't Move](#character-doesnt-move)
3. [Collisions Not Working](#collisions-not-working)
4. [Signals Not Firing](#signals-not-firing)
5. [Scene / Resource Won't Load](#scene--resource-wont-load)
6. [Input Not Detected](#input-not-detected)
7. [Animation Issues](#animation-issues)
8. [UI Problems](#ui-problems)
9. [Physics Behaving Strangely](#physics-behaving-strangely)
10. [Performance Issues](#performance-issues)

---

## Null Reference / Node Not Found

### "Invalid get index 'X' (on base: 'null instance')"

**Cause:** You're trying to access a node that doesn't exist or isn't ready yet.

**Fix 1: Use @onready**
```gdscript
# ❌ Wrong — runs before the scene tree is built
var sprite = $Sprite2D

# ✅ Right — waits until _ready()
@onready var sprite = $Sprite2D
```

**Fix 2: Check the node path**
- Open the scene in the editor
- Click the node you want to reference
- The path shown in the Scene dock is relative to the scene root
- `$Sprite2D` = direct child, `$Body/Sprite2D` = nested child

**Fix 3: Use scene-unique names**
- Right-click a node → "Access as Unique Name"
- Reference it anywhere in the scene with `%NodeName`
```gdscript
@onready var health_bar = %HealthBar  # Works regardless of where it is in the tree
```

**Fix 4: Node might be freed**
```gdscript
if is_instance_valid(target):
    target.take_damage(10)
```

### "Node not found: 'X'"

**Common causes:**
- Typo in the node name (case-sensitive!)
- Node was renamed in the editor but not in code
- Node hasn't been added to the scene yet (timing issue)
- You're in an instanced scene and the path is different

---

## Character Doesn't Move

### CharacterBody2D / CharacterBody3D won't move

**Fix 1: Call move_and_slide()**
```gdscript
# ❌ Setting velocity alone does nothing
velocity = Vector2(100, 0)

# ✅ Must call move_and_slide() after setting velocity
velocity = Vector2(100, 0)
move_and_slide()
```

**Fix 2: Use _physics_process, not _process**
```gdscript
# ❌ _process is for visuals, not physics
func _process(delta):
    velocity.x = 100
    move_and_slide()

# ✅ _physics_process runs at a fixed rate for physics
func _physics_process(delta):
    velocity.x = 100
    move_and_slide()
```

**Fix 3: Check Input Map**
- Go to Project → Project Settings → Input Map
- Make sure your action names match exactly (case-sensitive)
- Make sure keys are actually bound

**Fix 4: Multiply by delta for framerate independence**
```gdscript
# For non-velocity movement (e.g., position-based)
position.x += speed * delta
```

### RigidBody2D/3D won't move as expected

- Don't set `position` directly — use `apply_force()` or `apply_impulse()`
- Or set `linear_velocity` in `_integrate_forces()`
- Check that "Freeze" is not enabled in the inspector

---

## Collisions Not Working

### Bodies pass through each other

**Fix 1: Check collision layers and masks**
- Layer = "I exist on these layers"
- Mask = "I collide with things on these layers"
- Both bodies need compatible layer/mask settings
- Example: Player on layer 1, Enemy on layer 2. Player mask must include 2.

**Fix 2: Both need CollisionShape2D/3D**
- A body without a collision shape won't collide with anything
- Make sure the shape isn't disabled (`disabled = false`)

**Fix 3: Check body types**
- `StaticBody` ↔ `CharacterBody`: ✅ collides
- `StaticBody` ↔ `StaticBody`: ❌ neither moves, no collision response
- `Area2D` doesn't block movement — it only detects overlap

### Area2D signals not firing

**Fix 1: Enable monitoring**
```
Area2D → Inspector → Monitoring = true (for detecting others)
Area2D → Inspector → Monitorable = true (for being detected by others)
```

**Fix 2: Connect the right signal**
- `body_entered` → detects CharacterBody, RigidBody, StaticBody
- `area_entered` → detects other Area2D nodes

**Fix 3: Check collision layers/masks** (same as above)

---

## Signals Not Firing

### Connected signal doesn't call the method

**Fix 1: Verify connection exists**
```gdscript
# Check in _ready()
print($Button.pressed.get_connections())
```

**Fix 2: Check method signature**
```gdscript
# Signal: health_changed(current: float, max: float)

# ❌ Wrong — parameter count doesn't match
func _on_health_changed():
    pass

# ✅ Right — matches signal parameters
func _on_health_changed(current: float, max_value: float):
    pass
```

**Fix 3: Connect in code if editor connection isn't working**
```gdscript
func _ready():
    $Button.pressed.connect(_on_button_pressed)
    $Area2D.body_entered.connect(_on_body_entered)
```

**Fix 4: Signal might be emitted before connection**
```gdscript
# If a child emits in its _ready(), the parent might not have connected yet.
# Fix: use call_deferred or connect in the child's _ready
```

---

## Scene / Resource Won't Load

### "Failed to load resource: res://..."

**Fix 1: Check the path**
- Paths are case-sensitive on Linux/macOS
- Make sure the file exists in the FileSystem dock
- Use the exact path shown in the FileSystem dock

**Fix 2: Circular dependency**
- Scene A preloads Scene B, and Scene B preloads Scene A
- Fix: Use `load()` (runtime) instead of `preload()` (compile-time) for one of them
```gdscript
# Instead of:
var scene_b = preload("res://scene_b.tscn")  # Circular!

# Use:
var scene_b = load("res://scene_b.tscn")  # Loaded at runtime, avoids cycle
```

**Fix 3: Reimport**
- Delete the `.godot/imported/` folder (or specific cached files)
- Reopen the project — Godot will reimport everything

### .tres file won't load

- Check for GDScript syntax in the .tres file (`var`, `preload()`, etc.)
- See § File Formats in the main SKILL.md
- Ensure all `ext_resource` IDs are declared before use
- Check `load_steps` matches the actual number of resources

---

## Input Not Detected

### Input.is_action_pressed() always returns false

**Fix 1: Check action name**
```gdscript
# Must match exactly what's in Project Settings → Input Map
Input.is_action_pressed("move_left")  # Not "Move_Left" or "moveLeft"
```

**Fix 2: is_action_pressed vs is_action_just_pressed**
```gdscript
# is_action_pressed → true every frame the key is held
# is_action_just_pressed → true only the first frame

# For movement: use is_action_pressed (continuous)
# For jump/attack: use is_action_just_pressed (one-shot)
```

**Fix 3: Input eaten by UI**
- If a Control node (UI) has focus, it may consume input events
- Use `_unhandled_input()` instead of `_input()` for gameplay input
- This ensures UI gets first chance at the input

**Fix 4: Process mode**
- If the game is paused, input won't reach paused nodes
- Set `process_mode = PROCESS_MODE_ALWAYS` on nodes that need input while paused

---

## Animation Issues

### AnimationPlayer doesn't play

```gdscript
# Make sure the animation name matches exactly
$AnimationPlayer.play("run")  # Not "Run" or "running"

# List available animations for debugging:
print($AnimationPlayer.get_animation_list())
```

### AnimatedSprite2D doesn't animate

```gdscript
# Must call play() — it doesn't auto-play by default
$AnimatedSprite2D.play("walk")

# Or set Autoplay in the inspector for the default animation
```

### Animation plays but nothing moves

- Check that the animation tracks target the correct node paths
- If you renamed or moved nodes, the tracks may point to old paths
- Open the animation and verify the track paths in the bottom panel

---

## UI Problems

### Control node not visible

- Check `visible` property
- Check `modulate.a` (alpha) — might be transparent
- Check z-index or draw order
- Make sure it's inside a CanvasLayer if it should be on top of the game

### UI not scaling with window

- Set **Project → Project Settings → Display → Window → Stretch → Mode** to `canvas_items`
- Set **Aspect** to `keep` or `expand`
- Use anchors and containers (VBoxContainer, HBoxContainer) for responsive layout

### Button click not working

- Make sure no other Control is overlapping and consuming the click
- Check `mouse_filter` on parent containers:
  - `Stop` = consumes mouse events
  - `Pass` = handles events but lets them through
  - `Ignore` = transparent to mouse

---

## Physics Behaving Strangely

### Character jitters or vibrates

- Don't set `position` directly on physics bodies — let the physics engine handle it
- For CharacterBody: only modify `velocity`, then call `move_and_slide()`
- For camera following a physics body: enable camera smoothing or use `_physics_process`

### Character slides on slopes

```gdscript
# For CharacterBody2D/3D:
# Set in inspector or code:
floor_stop_on_slope = true
floor_max_angle = deg_to_rad(45)  # Max walkable slope angle
```

### Objects fall through the floor

- Make sure the floor has a CollisionShape
- For fast-moving objects, enable **Continuous CD** on the RigidBody
- Or increase physics ticks: Project → Physics → Common → Physics Ticks Per Second

---

## Performance Issues

### Game runs slowly

**Check 1: Are you creating nodes every frame?**
```gdscript
# ❌ Creating a new node every frame
func _process(delta):
    var label = Label.new()  # BAD — creates garbage every frame

# ✅ Create once, update as needed
@onready var label = $Label
func _process(delta):
    label.text = str(score)
```

**Check 2: Use the Profiler**
- Run the game → Debugger panel → Profiler tab
- Look for functions taking the most time

**Check 3: Object pooling**
- If spawning/destroying many objects (bullets, particles), use object pooling
- See `design-patterns.md § Object Pooling`

**Check 4: Reduce draw calls**
- Use `CanvasGroup` to batch 2D draw calls
- Use fewer unique materials in 3D
- Enable occlusion culling for 3D scenes
