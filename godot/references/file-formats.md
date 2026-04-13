# File Formats Reference

Deep dive into Godot's text-based file formats. Read this when creating or editing
.tscn/.tres files programmatically, or debugging "failed to load" errors.

---

## Overview

| Extension | Purpose | Syntax |
|---|---|---|
| `.gd` | GDScript code | Full programming language |
| `.tscn` | Scene files (node trees) | Serialization format |
| `.tres` | Resource files (data) | Serialization format |

**The key insight:** `.gd` uses GDScript. `.tscn`/`.tres` use a completely different
serialization syntax. Mixing them up is the #1 source of file-loading errors.

---

## GDScript (.gd)

Standard programming language. No special rules beyond normal GDScript:

```gdscript
extends CharacterBody2D
class_name Player

var health: int = 100
const MAX_SPEED = 300.0
@export var damage: int = 10
@onready var sprite = $Sprite2D

func _ready():
    var texture = preload("res://icon.png")
    print("Ready!")
```

---

## Scene Files (.tscn)

Describe a tree of nodes. Structure:

```
[gd_scene load_steps=3 format=3]          ← Header

[ext_resource type="Script" path="res://player.gd" id="1_abc"]     ← External resources
[ext_resource type="Texture2D" path="res://icon.png" id="2_def"]

[sub_resource type="RectangleShape2D" id="SubResource_1"]          ← Inline resources
size = Vector2(32, 64)

[node name="Player" type="CharacterBody2D"]                        ← Root node
script = ExtResource("1_abc")
speed = 200.0

[node name="Sprite" type="Sprite2D" parent="."]                   ← Child nodes
texture = ExtResource("2_def")

[node name="Collision" type="CollisionShape2D" parent="."]
shape = SubResource("SubResource_1")
```

### Rules

1. **`load_steps`** = total number of ext_resource + sub_resource + 1 (for the scene itself)
2. **`ext_resource`** declarations must come before any node that uses them
3. **IDs are strings** — `id="1_abc"` — referenced as `ExtResource("1_abc")`
4. **Parent paths:** `parent="."` = child of root, `parent="Player"` = child of Player
5. **No GDScript keywords:** No `var`, `func`, `preload()`, `const`
6. **Property values** use Godot's serialization format:
   - Vectors: `Vector2(10, 20)`, `Vector3(1, 2, 3)`
   - Colors: `Color(1, 0, 0, 1)`
   - Arrays: `Array[int]([1, 2, 3])` — must be typed
   - Booleans: `true`, `false`

### Common Mistakes in .tscn

```
# ❌ WRONG — using preload
script = preload("res://player.gd")

# ✅ RIGHT — using ExtResource
script = ExtResource("1_abc")
```

```
# ❌ WRONG — using var keyword
var speed = 200.0

# ✅ RIGHT — just property = value
speed = 200.0
```

```
# ❌ WRONG — untyped array
items = [1, 2, 3]

# ✅ RIGHT — typed array
items = Array[int]([1, 2, 3])
```

---

## Resource Files (.tres)

Store data — items, configurations, custom resources. Same syntax rules as .tscn
but describe a single resource instead of a node tree.

```
[gd_resource type="Resource" script_class="ItemData" load_steps=2 format=3]

[ext_resource type="Script" path="res://item_data.gd" id="1"]

[resource]
script = ExtResource("1")
name = "Iron Sword"
damage = 15
description = "A sturdy blade."
stackable = false
```

### Rules

1. **Header:** `[gd_resource type="..." ...]` — not `[gd_scene ...]`
2. **`script_class`** (optional) helps Godot know the custom type
3. **`[resource]`** section contains the actual property values
4. **Same syntax restrictions** as .tscn — no GDScript keywords
5. **Nested resources** use `[sub_resource]` just like in .tscn

### Example: Resource with Sub-Resources

```
[gd_resource type="Resource" script_class="SpellData" load_steps=4 format=3]

[ext_resource type="Script" path="res://spell_data.gd" id="1"]
[ext_resource type="Script" path="res://spell_effect.gd" id="2"]
[ext_resource type="Texture2D" path="res://icons/fireball.png" id="3"]

[sub_resource type="Resource" id="Effect_1"]
script = ExtResource("2")
effect_type = 0
magnitude = 25.0
duration = 3.0

[resource]
script = ExtResource("1")
spell_name = "Fireball"
icon = ExtResource("3")
mana_cost = 15
effects = Array[Resource]([SubResource("Effect_1")])
```

---

## Editing .tscn/.tres Safely

### Safe Edits (low risk)

- Changing property values on existing nodes/resources
- Adding new properties (they'll use defaults if unknown)
- Changing `ext_resource` paths (if the new file is compatible)

### Risky Edits (validate after)

- Adding/removing `ext_resource` or `sub_resource` entries (update `load_steps`!)
- Reordering nodes (parent references may break)
- Changing node types

### Avoid if Possible

- Complex scene restructuring — use the Godot editor instead
- Editing inherited scenes programmatically
- Modifying animation tracks in .tscn (very fragile format)

### Validation Checklist

After editing a .tscn or .tres file, verify:

1. `load_steps` matches actual count of ext_resource + sub_resource + 1
2. All `ExtResource("id")` references have matching `[ext_resource ... id="id"]`
3. All `SubResource("id")` references have matching `[sub_resource ... id="id"]`
4. No GDScript syntax (`var`, `func`, `preload()`, `const`)
5. Arrays are typed: `Array[Type]([...])`
6. File paths in ext_resource are correct and exist
7. Parent paths in nodes are valid
