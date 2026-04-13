---
name: godot
description: >
  Assist with designing and coding games in Godot Engine (4.x). Provides step-by-step
  instructions, working code examples, design guidance, and troubleshooting for GDScript,
  scenes, resources, and common game systems. Use this skill whenever the user mentions
  Godot, GDScript, .tscn, .tres, game development with Godot, or asks about implementing
  any game mechanic (movement, combat, inventory, UI, etc.) in Godot. Also use when
  debugging Godot errors, planning node hierarchies, or choosing between Godot approaches.
  Trigger for: "physics", "signals", "6DOF", "RigidBody", "CharacterBody", "Area2D",
  "AnimationPlayer", "shader", "export variable", "autoload", or any Godot-specific term.
  Even casual mentions like "my game," "my project," or "player controller" in a game dev
  context should trigger this skill.
---

# Godot Engine Development Skill

Step-by-step guidance for building games in Godot 4.x. Beginner-friendly by default —
explains concepts before code, uses simple examples, and builds up incrementally.

## Core Approach

When helping with Godot:

1. **Explain the "why" first.** Before writing code, briefly explain the approach and
   why it's the right choice. One sentence is often enough.
2. **Step-by-step implementation.** Break every task into numbered steps. Each step
   should be small enough to test immediately.
3. **Working examples.** Every code snippet should be complete enough to run. No
   pseudocode or "fill this in" placeholders unless the user asks for an outline.
4. **Show the result.** Describe what the user should see/experience after each step.
5. **One concept at a time.** Don't introduce signals, exports, and state machines
   all in one answer. Layer complexity as needed.

## Understanding the Request

| User Says | What They Need |
|---|---|
| "How do I make X move/jump/shoot" | Step-by-step tutorial → § Tutorials |
| "What's the best way to structure X" | Design guidance → § Design Guidance |
| "I'm getting an error" / "X doesn't work" | Troubleshooting → § Troubleshooting |
| "Create a .tscn / .tres file" | File format expertise → § File Formats |
| "Explain how X works in Godot" | Concept explanation with example |
| "Help me build [game system]" | Tutorial + design guidance combined |

If the request is ambiguous, give a brief answer and ask one follow-up question.

---

## Tutorials: Common Game Tasks

When the user asks how to implement something, follow this pattern:

### Tutorial Structure

```
1. What we're building (one sentence)
2. Setup — what nodes/scenes to create
3. Step-by-step code with explanations
4. Test it — what you should see
5. Next steps — how to extend it
```

For detailed tutorials on the most common tasks, read `references/tutorials.md`.
It covers:

- **Movement:** 2D top-down, 2D platformer, 3D first-person, 3D third-person
- **Combat:** Melee attacks, projectiles, health systems, damage/knockback
- **UI:** HUD, menus, health bars, inventory screens, dialogue boxes
- **Inventory:** Picking up items, storing items, equipping items
- **State Machines:** Player states, enemy AI states, game states
- **Saving/Loading:** Save game data, load game data, autosave
- **Audio:** Playing sounds, background music, spatial audio
- **Animation:** AnimationPlayer basics, AnimatedSprite2D, blend trees

Always read the relevant section of `references/tutorials.md` before writing a
tutorial response — it contains tested, working code.

---

## Design Guidance

Help users make good architectural decisions. Key principles:

### When to Use What

| Concept | Use When | Example |
|---|---|---|
| **Scenes (.tscn)** | Reusable game objects with visual/physical presence | Player, Enemy, Bullet, Level |
| **Resources (.tres)** | Pure data — stats, configs, definitions | Item stats, spell data, loot tables |
| **GDScript (.gd)** | All game logic and behavior | Movement code, AI, game rules |
| **Autoloads (singletons)** | Global state that persists between scenes | GameManager, AudioManager, SaveData |
| **Signals** | One thing telling others "something happened" | `health_changed`, `item_picked_up` |
| **Groups** | Finding all nodes of a type | All enemies, all interactables |
| **Export vars** | Making values tweakable in the editor | Speed, damage, jump height |

### Node Hierarchy Principles

**Keep it shallow.** Deep nesting makes code harder to maintain:

```
# ✅ Good — flat, clear
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
├── AnimationPlayer
├── HitboxArea (Area2D)
│   └── CollisionShape2D
└── Camera2D

# ❌ Avoid — unnecessarily deep
Player
└── Body
    └── Visual
        └── Sprite2D
            └── Effects
                └── AnimationPlayer
```

**One script per node (usually).** Don't put everything in one massive script.
Split responsibilities:

```
Player (CharacterBody2D)       → player.gd (movement + input)
├── HealthComponent (Node)     → health_component.gd (HP logic)
├── HitboxArea (Area2D)        → hitbox.gd (damage detection)
└── WeaponPivot (Marker2D)     → weapon.gd (attack logic)
```

### Signals vs Direct Calls

**Use signals when** the sender doesn't need to know who's listening:
```gdscript
# health_component.gd
signal health_changed(current_hp, max_hp)
signal died

func take_damage(amount):
    hp -= amount
    health_changed.emit(hp, max_hp)
    if hp <= 0:
        died.emit()
```

**Use direct calls when** you know exactly who you're talking to:
```gdscript
# player.gd — calling a known child node
func attack():
    $WeaponPivot.swing()
```

### Component Pattern

For reusable behaviors, use components — small nodes with one job:

```gdscript
# health_component.gd — attach to anything that has HP
extends Node
class_name HealthComponent

signal health_changed(current, max)
signal died

@export var max_hp: float = 100.0
var current_hp: float

func _ready():
    current_hp = max_hp

func take_damage(amount: float):
    current_hp = max(current_hp - amount, 0)
    health_changed.emit(current_hp, max_hp)
    if current_hp <= 0:
        died.emit()

func heal(amount: float):
    current_hp = min(current_hp + amount, max_hp)
    health_changed.emit(current_hp, max_hp)
```

Now attach `HealthComponent` to Player, Enemy, Destructible Crate — anything.

For more design patterns, read `references/design-patterns.md`.

---

## Troubleshooting

### Common Errors and Fixes

For a comprehensive list of common errors, read `references/troubleshooting.md`.
Here are the most frequent ones:

**"Invalid call. Nonexistent function" / Null reference:**
```
# Usually means the node path is wrong or the node isn't ready yet.
# Fix: Use @onready or check the node path in the scene tree.

@onready var sprite = $Sprite2D  # ✅ Waits until _ready()
var sprite = $Sprite2D           # ❌ Too early — node might not exist
```

**"Node not found" errors:**
```gdscript
# The path is relative to the current node. Check the Scene tree.
$Sprite2D           # Direct child
$Body/Sprite2D      # Grandchild
%UniqueNode         # Scene-unique name (set in editor: right-click → Access as Unique Name)
```

**Character doesn't move:**
```gdscript
# For CharacterBody2D/3D, you MUST call move_and_slide()
func _physics_process(delta):
    velocity.x = direction * speed
    move_and_slide()  # ← Don't forget this!
```

**Signal not working:**
```gdscript
# Check: Is the signal connected? Is the method signature correct?
# Connect in code:
$Button.pressed.connect(_on_button_pressed)

# Or connect in the editor: Node tab → Signals → double-click
```

**"res:// file not found":**
- Check the exact file path (case-sensitive on Linux/macOS)
- Make sure the file is inside the project folder
- Try reimporting: Project → Tools → Reimport All

---

## File Formats

Godot uses text-based files. GDScript (.gd) is a full programming language,
but .tscn and .tres files use a strict serialization format with different rules.

### The Golden Rule

> **GDScript syntax (.gd) ≠ Resource syntax (.tscn/.tres)**
>
> Never use `var`, `func`, `preload()`, or `const` in .tscn/.tres files.

### Quick Reference

**In .gd files** — full GDScript:
```gdscript
var health = 100
const MAX_SPEED = 300.0
var texture = preload("res://icon.png")
```

**In .tscn/.tres files** — serialization format:
```
[ext_resource type="Texture2D" path="res://icon.png" id="1"]

[node name="Player" type="CharacterBody2D"]
health = 100            # Just property = value, no "var"
max_speed = 300.0

[node name="Sprite" type="Sprite2D" parent="."]
texture = ExtResource("1")  # NOT preload()!
```

### Key .tscn/.tres Rules

1. **Declare external resources at the top:**
   ```
   [ext_resource type="Script" path="res://player.gd" id="1"]
   ```

2. **Reference them with ExtResource:**
   ```
   script = ExtResource("1")
   ```

3. **Inline resources use SubResource:**
   ```
   [sub_resource type="RectangleShape2D" id="SubResource_1"]
   size = Vector2(32, 64)

   [node name="Collision" type="CollisionShape2D" parent="."]
   shape = SubResource("SubResource_1")
   ```

4. **Use typed arrays:**
   ```
   items = Array[Resource]([])   # NOT just []
   ```

For the full file format specification with more examples and edge cases,
read `references/file-formats.md`.

---

## GDScript Patterns

Quick reference for common GDScript patterns beginners often need:

### Export Variables (Tweakable in Editor)

```gdscript
@export var speed: float = 200.0
@export var jump_force: float = -400.0
@export var max_health: int = 100
@export var weapon_scene: PackedScene  # Drag a .tscn here in the editor

@export_range(0, 100) var volume: int = 50
@export_enum("Sword", "Bow", "Staff") var weapon_type: int = 0
@export_group("Movement")  # Groups in the inspector
@export var acceleration: float = 500.0
@export var friction: float = 600.0
```

### Node Lifecycle

```gdscript
func _ready():
    # Called once when the node enters the scene tree.
    # All child nodes are ready. Safe to access $ChildNode.
    pass

func _process(delta):
    # Called every frame. Use for visuals, UI, non-physics logic.
    # delta = time since last frame (use it to be framerate-independent)
    pass

func _physics_process(delta):
    # Called at a fixed rate (60 times/sec by default).
    # Use for movement, physics, collision.
    pass

func _input(event):
    # Called for every input event. Use for one-shot inputs (jump, pause).
    if event.is_action_pressed("jump"):
        jump()

func _unhandled_input(event):
    # Like _input but only fires if no UI element consumed the event.
    pass
```

### Input Handling

```gdscript
# Continuous input (movement) — check in _physics_process:
var direction = Input.get_axis("move_left", "move_right")
velocity.x = direction * speed

# One-shot input (jump, attack) — check in _input or _unhandled_input:
func _unhandled_input(event):
    if event.is_action_pressed("jump") and is_on_floor():
        velocity.y = jump_force
```

### Scene Instancing (Spawning Objects)

```gdscript
@export var bullet_scene: PackedScene

func shoot():
    var bullet = bullet_scene.instantiate()
    bullet.position = $Muzzle.global_position
    bullet.rotation = $Muzzle.global_rotation
    get_tree().current_scene.add_child(bullet)
```

### Timers

```gdscript
# Option 1: Timer node (set up in editor, connect timeout signal)
func _on_timer_timeout():
    can_shoot = true

# Option 2: Code timer (one-shot)
await get_tree().create_timer(1.5).timeout
print("1.5 seconds passed")

# Option 3: Tween (for animations/transitions)
var tween = create_tween()
tween.tween_property($Sprite2D, "modulate:a", 0.0, 0.5)  # Fade out over 0.5s
```

---

## Reference Files

Load these when you need deeper detail:

| File | When to Read |
|---|---|
| `references/tutorials.md` | User asks "how do I make X" — contains tested step-by-step implementations |
| `references/design-patterns.md` | User asks about architecture, structure, or "what's the best way to..." |
| `references/troubleshooting.md` | User reports an error or something not working |
| `references/file-formats.md` | Creating/editing .tscn or .tres files, or debugging load errors |
