# Tutorials Reference

Tested, step-by-step implementations for common game tasks. Each tutorial follows the
pattern: what we're building → setup → step-by-step code → test it → extend it.

Always adapt to the user's specific needs. These are starting points, not rigid scripts.

---

## Table of Contents

1. [2D Top-Down Movement](#2d-top-down-movement)
2. [2D Platformer Movement](#2d-platformer-movement)
3. [3D First-Person Movement](#3d-first-person-movement)
4. [3D Third-Person Movement](#3d-third-person-movement)
5. [Melee Attack System](#melee-attack-system)
6. [Projectile System](#projectile-system)
7. [Health System with UI](#health-system-with-ui)
8. [Basic Inventory](#basic-inventory)
9. [Simple State Machine](#simple-state-machine)
10. [Save and Load](#save-and-load)
11. [Basic Dialogue System](#basic-dialogue-system)
12. [Main Menu and Pause](#main-menu-and-pause)

---

## 2D Top-Down Movement

**What we're building:** A character that moves in 8 directions with smooth acceleration.

### Step 1: Create the Scene

Create a new scene with this structure:
```
Player (CharacterBody2D)
├── Sprite2D
└── CollisionShape2D
```

- Set the Sprite2D texture to any image (or use Godot's icon.svg)
- Set the CollisionShape2D shape to a RectangleShape2D or CircleShape2D

### Step 2: Set Up Input Map

Go to **Project → Project Settings → Input Map** and add:
- `move_up` → W key + Up arrow
- `move_down` → S key + Down arrow
- `move_left` → A key + Left arrow
- `move_right` → D key + Right arrow

### Step 3: Write the Movement Script

Attach a script to the Player node:

```gdscript
# player.gd
extends CharacterBody2D

@export var speed: float = 200.0

func _physics_process(delta):
    # Get input direction as a vector
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")

    # Set velocity and move
    velocity = direction * speed
    move_and_slide()
```

### Step 4: Test It

Run the scene (F5 or F6). You should be able to move in 8 directions.
The diagonal movement is automatically normalized by `get_vector()`.

### Extend It

**Add acceleration and friction:**
```gdscript
@export var speed: float = 200.0
@export var acceleration: float = 800.0
@export var friction: float = 1000.0

func _physics_process(delta):
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")

    if direction != Vector2.ZERO:
        velocity = velocity.move_toward(direction * speed, acceleration * delta)
    else:
        velocity = velocity.move_toward(Vector2.ZERO, friction * delta)

    move_and_slide()
```

**Add sprite flipping:**
```gdscript
    if direction.x != 0:
        $Sprite2D.flip_h = direction.x < 0
```

---

## 2D Platformer Movement

**What we're building:** A character that runs and jumps with gravity.

### Step 1: Create the Scene

```
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
└── Camera2D (optional)
```

Also create a floor: `StaticBody2D` with a `CollisionShape2D` (wide rectangle).

### Step 2: Input Map

Add: `move_left`, `move_right`, `jump` (Space bar)

### Step 3: Movement Script

```gdscript
# player.gd
extends CharacterBody2D

@export var speed: float = 300.0
@export var jump_force: float = -500.0
@export var gravity: float = 1200.0

func _physics_process(delta):
    # Apply gravity
    if not is_on_floor():
        velocity.y += gravity * delta

    # Jump
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_force

    # Horizontal movement
    var direction = Input.get_axis("move_left", "move_right")
    velocity.x = direction * speed

    move_and_slide()
```

### Step 4: Test It

Run the scene. The character should fall to the floor, run left/right, and jump.

### Extend It

**Coyote time (jump briefly after leaving a ledge):**
```gdscript
var coyote_timer: float = 0.0
@export var coyote_time: float = 0.1

func _physics_process(delta):
    if is_on_floor():
        coyote_timer = coyote_time
    else:
        coyote_timer -= delta
        velocity.y += gravity * delta

    if Input.is_action_just_pressed("jump") and coyote_timer > 0:
        velocity.y = jump_force
        coyote_timer = 0.0

    var direction = Input.get_axis("move_left", "move_right")
    velocity.x = direction * speed
    move_and_slide()
```

**Variable jump height (short tap = short jump):**
```gdscript
    if Input.is_action_just_released("jump") and velocity.y < 0:
        velocity.y *= 0.5  # Cut upward velocity when releasing jump
```

---

## 3D First-Person Movement

**What we're building:** WASD movement with mouse-look.

### Step 1: Create the Scene

```
Player (CharacterBody3D)
├── CollisionShape3D (CapsuleShape3D)
└── CameraPivot (Node3D)
    └── Camera3D
```

- Position the CameraPivot at head height (y ≈ 1.5)
- Position the Camera3D at (0, 0, 0) relative to the pivot

### Step 2: Input Map

Add: `move_forward` (W), `move_back` (S), `move_left` (A), `move_right` (D), `jump` (Space)

### Step 3: Movement Script

```gdscript
# player.gd
extends CharacterBody3D

@export var speed: float = 5.0
@export var jump_force: float = 4.5
@export var mouse_sensitivity: float = 0.002
@export var gravity: float = 12.0

@onready var camera_pivot = $CameraPivot

func _ready():
    Input.mouse_mode = Input.MOUSE_MODE_CAPTURED

func _unhandled_input(event):
    # Mouse look
    if event is InputEventMouseMotion:
        rotate_y(-event.relative.x * mouse_sensitivity)
        camera_pivot.rotate_x(-event.relative.y * mouse_sensitivity)
        camera_pivot.rotation.x = clamp(camera_pivot.rotation.x, -PI/2, PI/2)

    # Escape to free mouse
    if event.is_action_pressed("ui_cancel"):
        Input.mouse_mode = Input.MOUSE_MODE_VISIBLE

func _physics_process(delta):
    # Gravity
    if not is_on_floor():
        velocity.y -= gravity * delta

    # Jump
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_force

    # Movement direction (relative to where we're looking)
    var input_dir = Input.get_vector("move_left", "move_right", "move_forward", "move_back")
    var direction = (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()

    if direction:
        velocity.x = direction.x * speed
        velocity.z = direction.z * speed
    else:
        velocity.x = move_toward(velocity.x, 0, speed)
        velocity.z = move_toward(velocity.z, 0, speed)

    move_and_slide()
```

### Step 4: Test It

Run the scene. WASD moves, mouse looks around, Space jumps, Esc frees the cursor.

---

## 3D Third-Person Movement

**What we're building:** Character movement with a following camera.

### Step 1: Create the Scene

```
Player (CharacterBody3D)
├── CollisionShape3D (CapsuleShape3D)
├── PlayerModel (Node3D)
│   └── MeshInstance3D (or your character model)
└── CameraPivot (SpringArm3D)
    └── Camera3D
```

- Set SpringArm3D length to ~4.0, position at head height
- SpringArm3D handles camera collision automatically

### Step 2: Movement Script

```gdscript
# player.gd
extends CharacterBody3D

@export var speed: float = 5.0
@export var jump_force: float = 4.5
@export var mouse_sensitivity: float = 0.002
@export var gravity: float = 12.0

@onready var camera_pivot = $CameraPivot
@onready var model = $PlayerModel

func _ready():
    Input.mouse_mode = Input.MOUSE_MODE_CAPTURED

func _unhandled_input(event):
    if event is InputEventMouseMotion:
        camera_pivot.rotate_y(-event.relative.x * mouse_sensitivity)
        camera_pivot.rotate_x(-event.relative.y * mouse_sensitivity)
        camera_pivot.rotation.x = clamp(camera_pivot.rotation.x, -PI/3, PI/3)

func _physics_process(delta):
    if not is_on_floor():
        velocity.y -= gravity * delta

    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_force

    var input_dir = Input.get_vector("move_left", "move_right", "move_forward", "move_back")
    # Direction relative to camera, not player
    var direction = (camera_pivot.global_transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized()
    direction.y = 0

    if direction:
        velocity.x = direction.x * speed
        velocity.z = direction.z * speed
        # Rotate model to face movement direction
        model.look_at(global_position + direction, Vector3.UP)
    else:
        velocity.x = move_toward(velocity.x, 0, speed)
        velocity.z = move_toward(velocity.z, 0, speed)

    move_and_slide()
```

---

## Melee Attack System

**What we're building:** A simple melee attack with a hitbox that damages enemies.

### Step 1: Setup

Add to the Player scene:
```
Player
├── ...existing nodes...
└── AttackArea (Area2D)
    └── CollisionShape2D (disabled by default)
```

- Position the AttackArea in front of the player
- Set the CollisionShape2D `disabled = true` in the inspector

### Step 2: Attack Script

```gdscript
# Add to player.gd (or create a separate weapon script)

@export var attack_damage: int = 10
@export var attack_duration: float = 0.15
var can_attack: bool = true

func _unhandled_input(event):
    if event.is_action_pressed("attack") and can_attack:
        attack()

func attack():
    can_attack = false
    $AttackArea/CollisionShape2D.disabled = false

    # Wait for attack duration, then disable hitbox
    await get_tree().create_timer(attack_duration).timeout
    $AttackArea/CollisionShape2D.disabled = true

    # Cooldown
    await get_tree().create_timer(0.3).timeout
    can_attack = true
```

### Step 3: Detect Hits

Connect the `AttackArea.body_entered` signal:

```gdscript
func _ready():
    $AttackArea.body_entered.connect(_on_attack_area_body_entered)

func _on_attack_area_body_entered(body):
    if body.has_method("take_damage"):
        body.take_damage(attack_damage)
```

### Step 4: Enemy Takes Damage

On any enemy or damageable object:
```gdscript
# enemy.gd
var health: int = 30

func take_damage(amount: int):
    health -= amount
    # Visual feedback
    $Sprite2D.modulate = Color.RED
    await get_tree().create_timer(0.1).timeout
    $Sprite2D.modulate = Color.WHITE

    if health <= 0:
        queue_free()  # Remove from scene
```

---

## Projectile System

**What we're building:** A bullet that flies forward and damages what it hits.

### Step 1: Create Bullet Scene

New scene:
```
Bullet (Area2D)
├── Sprite2D (small circle or arrow)
└── CollisionShape2D (small circle)
```

Save as `bullet.tscn`.

### Step 2: Bullet Script

```gdscript
# bullet.gd
extends Area2D

@export var speed: float = 600.0
@export var damage: int = 10
@export var lifetime: float = 3.0

func _ready():
    # Auto-destroy after lifetime
    await get_tree().create_timer(lifetime).timeout
    queue_free()

func _physics_process(delta):
    # Move forward (in the direction the bullet is rotated)
    position += transform.x * speed * delta

func _on_body_entered(body):
    if body.has_method("take_damage"):
        body.take_damage(damage)
    queue_free()
```

Connect the `body_entered` signal to `_on_body_entered` in the editor.

### Step 3: Shooting from the Player

```gdscript
# In player.gd
@export var bullet_scene: PackedScene  # Drag bullet.tscn here in the inspector

func _unhandled_input(event):
    if event.is_action_pressed("shoot"):
        shoot()

func shoot():
    var bullet = bullet_scene.instantiate()
    bullet.global_position = $Muzzle.global_position
    bullet.global_rotation = $Muzzle.global_rotation
    get_tree().current_scene.add_child(bullet)
```

Add a `Marker2D` named "Muzzle" to the player scene, positioned where bullets spawn.

---

## Health System with UI

**What we're building:** A reusable health component with a health bar.

### Step 1: Health Component

```gdscript
# health_component.gd
extends Node
class_name HealthComponent

signal health_changed(current_hp, max_hp)
signal died

@export var max_hp: float = 100.0
var current_hp: float

func _ready():
    current_hp = max_hp

func take_damage(amount: float):
    current_hp = max(current_hp - amount, 0.0)
    health_changed.emit(current_hp, max_hp)
    if current_hp <= 0:
        died.emit()

func heal(amount: float):
    current_hp = min(current_hp + amount, max_hp)
    health_changed.emit(current_hp, max_hp)

func get_health_percent() -> float:
    return current_hp / max_hp if max_hp > 0 else 0.0
```

### Step 2: Health Bar UI

Create a scene or add to your HUD:
```
HealthBar (ProgressBar)
```

Script:
```gdscript
# health_bar.gd
extends ProgressBar

func update_health(current_hp: float, max_hp: float):
    max_value = max_hp
    value = current_hp
```

### Step 3: Connect Them

In the entity that has health (Player, Enemy, etc.):
```gdscript
func _ready():
    $HealthComponent.health_changed.connect($HealthBar.update_health)
    $HealthComponent.died.connect(_on_died)

func _on_died():
    queue_free()  # Or play death animation, etc.
```

---

## Basic Inventory

**What we're building:** A simple inventory that stores items as resources.

### Step 1: Item Resource

```gdscript
# item_data.gd
extends Resource
class_name ItemData

@export var name: String = ""
@export var description: String = ""
@export var icon: Texture2D
@export var stackable: bool = true
@export var max_stack: int = 99
```

Create items in the editor: **right-click in FileSystem → New Resource → ItemData**.
Fill in the fields and save as `.tres` files.

### Step 2: Inventory Script

```gdscript
# inventory.gd
extends Node
class_name Inventory

signal inventory_changed

@export var max_slots: int = 20
var slots: Array[Dictionary] = []  # [{item: ItemData, quantity: int}, ...]

func _ready():
    slots.resize(max_slots)

func add_item(item: ItemData, quantity: int = 1) -> bool:
    # Try to stack with existing
    if item.stackable:
        for i in slots.size():
            if slots[i] and slots[i].item == item and slots[i].quantity < item.max_stack:
                var space = item.max_stack - slots[i].quantity
                var to_add = min(quantity, space)
                slots[i].quantity += to_add
                quantity -= to_add
                if quantity <= 0:
                    inventory_changed.emit()
                    return true

    # Find empty slot
    for i in slots.size():
        if not slots[i]:
            slots[i] = {item = item, quantity = quantity}
            inventory_changed.emit()
            return true

    return false  # Inventory full

func remove_item(slot_index: int, quantity: int = 1):
    if slots[slot_index]:
        slots[slot_index].quantity -= quantity
        if slots[slot_index].quantity <= 0:
            slots[slot_index] = null
        inventory_changed.emit()
```

### Step 3: Picking Up Items

On a pickup in the world:
```gdscript
# item_pickup.gd
extends Area2D

@export var item: ItemData
@export var quantity: int = 1

func _on_body_entered(body):
    if body.has_node("Inventory"):
        var inventory = body.get_node("Inventory")
        if inventory.add_item(item, quantity):
            queue_free()  # Picked up successfully
```

---

## Simple State Machine

**What we're building:** A basic state machine for player or enemy behavior.

### Step 1: State Machine Node

```gdscript
# state_machine.gd
extends Node
class_name StateMachine

var current_state: State
var states: Dictionary = {}

func _ready():
    # Register all child State nodes
    for child in get_children():
        if child is State:
            states[child.name.to_lower()] = child
            child.state_machine = self

    # Start with the first child state
    if get_child_count() > 0:
        current_state = get_child(0)
        current_state.enter()

func _process(delta):
    if current_state:
        current_state.update(delta)

func _physics_process(delta):
    if current_state:
        current_state.physics_update(delta)

func transition_to(state_name: String):
    var new_state = states.get(state_name.to_lower())
    if new_state and new_state != current_state:
        current_state.exit()
        current_state = new_state
        current_state.enter()
```

### Step 2: Base State

```gdscript
# state.gd
extends Node
class_name State

var state_machine: StateMachine

func enter():
    pass

func exit():
    pass

func update(_delta: float):
    pass

func physics_update(_delta: float):
    pass
```

### Step 3: Example — Player Idle and Run States

```gdscript
# idle_state.gd
extends State

func physics_update(delta):
    var owner_node = owner as CharacterBody2D
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")

    if direction != Vector2.ZERO:
        state_machine.transition_to("run")

    owner_node.velocity = Vector2.ZERO
    owner_node.move_and_slide()
```

```gdscript
# run_state.gd
extends State

@export var speed: float = 200.0

func physics_update(delta):
    var owner_node = owner as CharacterBody2D
    var direction = Input.get_vector("move_left", "move_right", "move_up", "move_down")

    if direction == Vector2.ZERO:
        state_machine.transition_to("idle")

    owner_node.velocity = direction * speed
    owner_node.move_and_slide()
```

### Scene Structure

```
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
└── StateMachine
    ├── Idle (idle_state.gd)
    └── Run (run_state.gd)
```

---

## Save and Load

**What we're building:** Save/load game data to a JSON file.

### Step 1: Save Manager (Autoload)

```gdscript
# save_manager.gd — add as Autoload in Project Settings
extends Node

const SAVE_PATH = "user://savegame.json"

func save_game(data: Dictionary):
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    file.store_string(JSON.stringify(data, "\t"))

func load_game() -> Dictionary:
    if not FileAccess.file_exists(SAVE_PATH):
        return {}
    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    var json = JSON.new()
    json.parse(file.get_as_text())
    return json.data if json.data is Dictionary else {}

func has_save() -> bool:
    return FileAccess.file_exists(SAVE_PATH)

func delete_save():
    if has_save():
        DirAccess.remove_absolute(SAVE_PATH)
```

### Step 2: Using It

```gdscript
# Save
func save():
    SaveManager.save_game({
        "player_position": {"x": position.x, "y": position.y},
        "health": $HealthComponent.current_hp,
        "inventory": get_inventory_data(),
        "current_level": get_tree().current_scene.scene_file_path
    })

# Load
func load():
    var data = SaveManager.load_game()
    if data.is_empty():
        return
    position = Vector2(data.player_position.x, data.player_position.y)
    $HealthComponent.current_hp = data.health
```

---

## Main Menu and Pause

**What we're building:** A main menu and an in-game pause screen.

### Main Menu

Create a scene `main_menu.tscn`:
```
MainMenu (Control)
├── VBoxContainer (centered)
│   ├── TitleLabel (Label)
│   ├── PlayButton (Button)
│   ├── OptionsButton (Button)
│   └── QuitButton (Button)
```

```gdscript
# main_menu.gd
extends Control

func _on_play_button_pressed():
    get_tree().change_scene_to_file("res://scenes/game.tscn")

func _on_quit_button_pressed():
    get_tree().quit()
```

Connect each button's `pressed` signal.

### Pause Menu

Create a scene or add a CanvasLayer to your game:
```
PauseMenu (CanvasLayer)
└── Panel (CenterContainer)
    └── VBoxContainer
        ├── Label ("Paused")
        ├── ResumeButton
        └── QuitButton
```

```gdscript
# pause_menu.gd
extends CanvasLayer

func _ready():
    visible = false
    process_mode = Node.PROCESS_MODE_ALWAYS  # Works even when paused

func _unhandled_input(event):
    if event.is_action_pressed("pause"):
        toggle_pause()

func toggle_pause():
    var paused = not get_tree().paused
    get_tree().paused = paused
    visible = paused

func _on_resume_button_pressed():
    toggle_pause()

func _on_quit_button_pressed():
    get_tree().paused = false
    get_tree().change_scene_to_file("res://scenes/main_menu.tscn")
```

**Important:** Set the PauseMenu's `process_mode` to "Always" so it works when
the game is paused. All other nodes default to "Inherit" (which means they pause).
