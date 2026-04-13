# Design Patterns Reference

Proven architectural patterns for Godot projects. Read this when the user asks
"what's the best way to..." or needs help structuring a game system.

---

## Table of Contents

1. [Component Pattern](#component-pattern)
2. [Observer Pattern (Signals)](#observer-pattern-signals)
3. [State Machine Pattern](#state-machine-pattern)
4. [Resource-Based Data](#resource-based-data)
5. [Autoload Singletons](#autoload-singletons)
6. [Scene Composition](#scene-composition)
7. [Event Bus](#event-bus)
8. [Object Pooling](#object-pooling)

---

## Component Pattern

**Use when:** Multiple entities share some behaviors but not all.

Instead of deep inheritance (Enemy → FlyingEnemy → FlyingShootingEnemy), attach
small, reusable components:

```
Enemy (CharacterBody2D)
├── HealthComponent (Node)      → Any entity can have health
├── HitboxComponent (Area2D)    → Any entity can be hit
├── DropLootComponent (Node)    → Any entity can drop loot
└── PatrolComponent (Node)      → Only some enemies patrol
```

**Component template:**
```gdscript
# Generic component pattern
extends Node
class_name MyComponent

signal something_happened

@export var enabled: bool = true

func do_thing():
    if not enabled:
        return
    # Component logic here
    something_happened.emit()
```

**Checking for components:**
```gdscript
# Option 1: has_node (check if component exists)
if target.has_node("HealthComponent"):
    target.get_node("HealthComponent").take_damage(10)

# Option 2: has_method (duck typing)
if target.has_method("take_damage"):
    target.take_damage(10)

# Option 3: Groups (for finding all entities with a trait)
for enemy in get_tree().get_nodes_in_group("damageable"):
    enemy.get_node("HealthComponent").take_damage(10)
```

---

## Observer Pattern (Signals)

**Use when:** Systems need to react to events without tight coupling.

**Key rules:**
- The emitter doesn't know (or care) who's listening
- Multiple listeners can connect to the same signal
- Connect in `_ready()` or when nodes are added to the scene

**Common signal patterns:**

```gdscript
# Pattern 1: Child emits, parent connects
# In parent _ready():
$HealthComponent.died.connect(_on_player_died)

# Pattern 2: Autoload emits, anyone connects (Event Bus — see below)
GameEvents.level_completed.connect(_on_level_completed)

# Pattern 3: Connect with extra arguments using callables
$Button.pressed.connect(_on_item_selected.bind(item_data))
```

**When NOT to use signals:**
- When you need a return value (use direct method call)
- When there's exactly one known recipient (direct call is simpler)
- For high-frequency per-frame communication (signals have overhead)

---

## State Machine Pattern

**Use when:** An entity has distinct behavioral modes (idle, running, attacking,
stunned, dead) and the logic for each mode is different.

See `tutorials.md § Simple State Machine` for the full implementation.

**When to use a state machine vs simple if/else:**

```gdscript
# ✅ Simple enough for if/else (2-3 states, minimal logic per state)
if is_on_floor():
    # ground behavior
else:
    # air behavior

# ✅ Use a state machine when:
# - 4+ distinct states
# - Each state has significant logic
# - States have enter/exit behavior
# - You need to prevent invalid transitions
```

**Advanced: Hierarchical State Machine**

For complex characters, states can have sub-states:

```
StateMachine
├── Grounded
│   ├── Idle
│   ├── Run
│   └── Crouch
├── Airborne
│   ├── Jump
│   ├── Fall
│   └── Glide
└── Combat
    ├── Attack
    ├── Block
    └── Dodge
```

The parent state handles shared logic (e.g., Grounded checks for jump input),
child states handle specifics.

---

## Resource-Based Data

**Use when:** You have data that should be editable in the Godot inspector and
shared across instances (items, abilities, enemy types, dialogue).

**Step 1: Define the resource class**
```gdscript
# weapon_data.gd
extends Resource
class_name WeaponData

@export var name: String
@export var damage: int
@export var attack_speed: float
@export var icon: Texture2D
@export var weapon_range: float = 1.5
@export_multiline var description: String
```

**Step 2: Create instances in the editor**

Right-click in FileSystem → New Resource → WeaponData. Fill in fields, save as
`iron_sword.tres`, `fire_staff.tres`, etc.

**Step 3: Use in scripts**
```gdscript
@export var weapon: WeaponData  # Drag .tres file here in inspector

func attack():
    deal_damage(weapon.damage)
    play_animation(weapon.attack_speed)
```

**Benefits:**
- Designers can tweak values without touching code
- Resources are shared by reference (memory efficient)
- Easy to create variants (duplicate a .tres and change values)
- Version-control friendly (text-based .tres files)

**Caution:** Resources are shared by default. If you modify a resource at runtime,
ALL references to it change. To get an independent copy:
```gdscript
var my_copy = weapon.duplicate()
my_copy.damage = 999  # Only affects this copy
```

---

## Autoload Singletons

**Use when:** You need global state or systems that persist across scene changes.

Set up in **Project → Project Settings → Autoload**.

**Common autoloads:**

```gdscript
# game_manager.gd — Overall game state
extends Node

var score: int = 0
var current_level: int = 1
var player_data: Dictionary = {}

func reset():
    score = 0
    current_level = 1
```

```gdscript
# audio_manager.gd — Persistent audio
extends Node

@onready var music_player = $MusicPlayer
@onready var sfx_player = $SFXPlayer

func play_music(stream: AudioStream):
    if music_player.stream != stream:
        music_player.stream = stream
        music_player.play()

func play_sfx(stream: AudioStream):
    sfx_player.stream = stream
    sfx_player.play()
```

**Guidelines:**
- Keep autoloads minimal — only truly global state
- Don't put gameplay logic in autoloads (use scene scripts)
- Autoloads are always at the root of the scene tree
- Access anywhere: `GameManager.score += 100`

---

## Scene Composition

**Use when:** Building reusable game objects from smaller pieces.

**Principle:** Each `.tscn` is a self-contained unit that works on its own.

```
# enemy.tscn — a complete enemy
Enemy (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
├── HealthComponent.tscn (instanced)    ← Reusable scene
├── HitboxComponent.tscn (instanced)    ← Reusable scene
├── NavigationAgent2D
└── StateMachine
    ├── PatrolState
    └── ChaseState
```

**Instancing scenes at runtime:**
```gdscript
var enemy_scene = preload("res://scenes/enemy.tscn")

func spawn_enemy(pos: Vector2):
    var enemy = enemy_scene.instantiate()
    enemy.position = pos
    add_child(enemy)
```

**Scene inheritance** (less common, but useful):
- Create a base scene (e.g., `base_enemy.tscn`)
- Right-click → New Inherited Scene
- Override specific parts (different sprite, different stats)

---

## Event Bus

**Use when:** Distant parts of the game need to communicate without references
to each other (e.g., an enemy dying updates the UI score counter).

**Setup:** Create an autoload:

```gdscript
# game_events.gd — Autoload
extends Node

signal enemy_killed(enemy_type: String, position: Vector2)
signal item_collected(item: Resource)
signal player_health_changed(current: float, max: float)
signal level_completed(level_number: int)
signal dialogue_started(dialogue_id: String)
signal dialogue_ended
```

**Emitting:**
```gdscript
# In enemy.gd
func die():
    GameEvents.enemy_killed.emit(enemy_type, global_position)
    queue_free()
```

**Listening:**
```gdscript
# In score_ui.gd (completely separate from enemy code)
func _ready():
    GameEvents.enemy_killed.connect(_on_enemy_killed)

func _on_enemy_killed(type, pos):
    score += get_score_for(type)
    update_display()
```

**Guidelines:**
- Define all events in one place (easy to see what's available)
- Use typed signal parameters
- Don't overuse — direct signals are better for parent-child communication

---

## Object Pooling

**Use when:** You spawn and destroy many objects frequently (bullets, particles,
coins) and want to avoid garbage collection hitches.

```gdscript
# object_pool.gd
extends Node
class_name ObjectPool

@export var scene: PackedScene
@export var pool_size: int = 20

var pool: Array[Node] = []

func _ready():
    for i in pool_size:
        var obj = scene.instantiate()
        obj.set_process(false)
        obj.visible = false
        add_child(obj)
        pool.append(obj)

func get_object() -> Node:
    for obj in pool:
        if not obj.visible:
            obj.visible = true
            obj.set_process(true)
            return obj
    # Pool exhausted — optionally grow it
    return null

func return_object(obj: Node):
    obj.visible = false
    obj.set_process(false)
    # Reset state as needed
```

**Usage:**
```gdscript
@onready var bullet_pool = $BulletPool

func shoot():
    var bullet = bullet_pool.get_object()
    if bullet:
        bullet.global_position = $Muzzle.global_position
        bullet.global_rotation = $Muzzle.global_rotation
        bullet.activate()  # Custom method to reset and start moving
```

Use pooling when you notice frame drops during heavy spawning. For most games,
regular `instantiate()`/`queue_free()` is fine.
