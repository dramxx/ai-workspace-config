---
name: godot-best-practices
description: Expert Godot 4 game development guidance covering GDScript patterns, scene architecture, signals, performance, and project structure. Use this skill when writing GDScript, designing scene trees, working with signals/autoloads/resources, implementing game systems (state machines, inventory, combat, UI), debugging Godot-specific issues, or structuring any Godot 4 project. Always trigger for any Godot or GDScript-related task, even if the user just mentions a `.gd` file, a node type, or a game mechanic they want to implement.
---

# Godot 4 Best Practices

Expert guide to building clean, performant, and maintainable games in Godot 4.

## When to Use

- Writing or reviewing GDScript
- Designing scene/node hierarchies
- Wiring up signals
- Implementing game systems (state machines, inventory, cameras, UI)
- Debugging Godot-specific errors
- Structuring a new Godot project
- Porting patterns from other engines (Unity/Unreal) to Godot idioms

---

## 1. Project Structure

Organize by feature, not by file type:

```
res://
├── assets/
│   ├── sprites/
│   ├── sounds/
│   └── fonts/
├── scenes/
│   ├── player/
│   │   ├── player.tscn
│   │   └── player.gd
│   ├── enemies/
│   │   ├── goblin.tscn
│   │   └── goblin.gd
│   ├── ui/
│   └── world/
├── scripts/
│   ├── autoloads/       # Global singletons
│   └── resources/       # Custom Resource types
├── shaders/
└── project.godot
```

**Key rules:**
- Keep `.gd` scripts next to their `.tscn` scene files
- Autoloads go in `scripts/autoloads/` — register them in Project Settings
- Custom Resources go in `scripts/resources/` with their own `.tres` instances in `assets/`

---

## 2. GDScript Best Practices

### Type Everything

Godot 4 GDScript has full static typing — use it:

```gdscript
# Bad - no types
var speed = 200
var health = 100

func move(direction):
    position += direction * speed

# Good - typed
var speed: float = 200.0
var health: int = 100

func move(direction: Vector2) -> void:
    position += direction * speed
```

### Class Declaration

Always name your classes:

```gdscript
class_name Player
extends CharacterBody2D

## Player character controlled by input.
## Handles movement, jumping, and interaction.
```

### Constants Over Magic Numbers

```gdscript
# Bad
if velocity.y > 800:
    is_falling = true

# Good
const MAX_FALL_SPEED: float = 800.0

if velocity.y > MAX_FALL_SPEED:
    is_falling = true
```

### Onready Variables

Use `@onready` for node references — never use `get_node()` at the top level:

```gdscript
# Bad - fragile, runs before tree is ready
var sprite = get_node("Sprite2D")

# Good
@onready var sprite: Sprite2D = $Sprite2D
@onready var animation_player: AnimationPlayer = $AnimationPlayer
@onready var hitbox: Area2D = $Hitbox
```

### Export Variables for Designer Control

```gdscript
@export var move_speed: float = 200.0
@export var jump_force: float = 400.0
@export var max_health: int = 100
@export var projectile_scene: PackedScene
@export_group("Combat")
@export var attack_damage: int = 10
@export var attack_cooldown: float = 0.5
```

### Enum for States and Categories

```gdscript
enum State { IDLE, RUNNING, JUMPING, ATTACKING, DEAD }
enum ItemType { WEAPON, ARMOR, CONSUMABLE, KEY_ITEM }

var current_state: State = State.IDLE
```

---

## 3. Node and Scene Architecture

### Scene = Reusable Unit

Each scene should be independently playable. Design for composition:

```
Player.tscn
└── CharacterBody2D (player.gd)
    ├── Sprite2D
    ├── CollisionShape2D
    ├── AnimationPlayer
    ├── Camera2D
    ├── Hitbox (Area2D)           ← detects what player hits
    │   └── CollisionShape2D
    └── Hurtbox (Area2D)          ← detects what hits player
        └── CollisionShape2D
```

### Ownership and Communication Rules

Follow this strict hierarchy:
1. **Parent → Child**: call methods directly or set properties
2. **Child → Parent**: emit signals, never `get_parent()`
3. **Sibling → Sibling**: communicate through parent or signals
4. **Global**: use Autoloads only for truly global state

```gdscript
# Bad - child reaching up and sideways
func take_damage(amount: int) -> void:
    get_parent().get_node("HUD").update_health(health)

# Good - emit signal, let parent handle wiring
signal health_changed(new_health: int)

func take_damage(amount: int) -> void:
    health -= amount
    health_changed.emit(health)
```

### Prefer Composition Over Deep Inheritance

```gdscript
# Bad - deep inheritance chain
class_name FlyingEnemy
extends Enemy  # extends Character  # extends ...

# Good - compose with components
# Enemy.tscn has a HealthComponent, MovementComponent, etc.
# Each component is its own scene with its own script
```

---

## 4. Signals

Signals are the backbone of Godot architecture. Use them liberally.

### Define with Types

```gdscript
# Godot 4 typed signals
signal health_changed(current: int, maximum: int)
signal died()
signal item_collected(item: ItemResource)
signal attack_started(damage: int, knockback: Vector2)
```

### Connect in Code, Not Editor (for dynamic scenes)

```gdscript
# In parent scene's _ready():
func _ready() -> void:
    player.health_changed.connect(_on_player_health_changed)
    player.died.connect(_on_player_died)

func _on_player_health_changed(current: int, maximum: int) -> void:
    hud.update_health_bar(current, maximum)

func _on_player_died() -> void:
    game_over_screen.show()
```

### One-Shot Connections

```gdscript
# Connect a signal that should only fire once
animation_player.animation_finished.connect(
    _on_death_animation_done, CONNECT_ONE_SHOT
)
```

---

## 5. Autoloads (Singletons)

Use autoloads sparingly — only for truly global systems:

**Good autoload candidates:**
- `GameManager` — scene transitions, game state (paused/playing)
- `AudioManager` — play sounds/music from anywhere
- `SaveManager` — save/load game data
- `EventBus` — global signal bus for decoupled communication

**Bad autoload candidates:**
- Player stats (use a Resource instead)
- Enemy behavior (belongs in the enemy scene)
- UI state (belongs in UI scenes)

### EventBus Pattern

```gdscript
# autoloads/event_bus.gd
class_name EventBus
extends Node

signal player_died()
signal level_completed(level_id: int)
signal enemy_killed(enemy_type: String, position: Vector2)
signal score_changed(new_score: int)
```

```gdscript
# Any scene can emit:
EventBus.enemy_killed.emit("goblin", global_position)

# Any scene can listen:
EventBus.enemy_killed.connect(_on_enemy_killed)
```

---

## 6. Custom Resources

Use `Resource` for data containers — replaces ScriptableObjects from Unity:

```gdscript
# scripts/resources/item_resource.gd
class_name ItemResource
extends Resource

@export var item_name: String = ""
@export var description: String = ""
@export var icon: Texture2D
@export var item_type: ItemType
@export var value: int = 0
@export var stackable: bool = false
@export var max_stack: int = 1
```

Create `.tres` instances in the editor. Reference them via `@export`:

```gdscript
@export var starting_weapon: ItemResource
@export var item_database: Array[ItemResource] = []
```

---

## 7. State Machine Pattern

Essential for characters and AI. Implement cleanly:

```gdscript
class_name Player
extends CharacterBody2D

enum State { IDLE, RUN, JUMP, FALL, ATTACK, HURT, DEAD }

var state: State = State.IDLE

func _physics_process(delta: float) -> void:
    match state:
        State.IDLE:   _state_idle(delta)
        State.RUN:    _state_run(delta)
        State.JUMP:   _state_jump(delta)
        State.FALL:   _state_fall(delta)
        State.ATTACK: _state_attack(delta)
        State.HURT:   _state_hurt(delta)
        State.DEAD:   _state_dead(delta)

func _change_state(new_state: State) -> void:
    # Exit current state
    match state:
        State.ATTACK: _exit_attack()

    state = new_state

    # Enter new state
    match state:
        State.JUMP:   _enter_jump()
        State.ATTACK: _enter_attack()

func _state_idle(delta: float) -> void:
    apply_gravity(delta)
    var direction := Input.get_axis("move_left", "move_right")
    if direction != 0:
        _change_state(State.RUN)
    elif Input.is_action_just_pressed("jump") and is_on_floor():
        _change_state(State.JUMP)
    move_and_slide()
```

---

## 8. Physics and Collision

### Use Collision Layers Properly

Set up named layers in Project Settings → Physics → 2D:
- Layer 1: `world`
- Layer 2: `player`
- Layer 3: `enemies`
- Layer 4: `player_projectiles`
- Layer 5: `enemy_projectiles`
- Layer 6: `pickups`

```gdscript
# In scripts, reference by name (Godot 4.1+)
# Or by bitmask
collision_layer = 0b000010  # layer 2 (player)
collision_mask  = 0b000101  # detects layer 1 (world) + 3 (enemies)
```

### Hitbox/Hurtbox Pattern

```gdscript
# hurtbox.gd - attached to Area2D on any damageable entity
class_name Hurtbox
extends Area2D

signal hit(damage: int, knockback: Vector2)

func _ready() -> void:
    area_entered.connect(_on_area_entered)

func _on_area_entered(area: Area2D) -> void:
    if area is Hitbox:
        hit.emit(area.damage, area.knockback_direction * area.knockback_force)
```

```gdscript
# hitbox.gd - attached to Area2D on attacks/projectiles
class_name Hitbox
extends Area2D

@export var damage: int = 10
@export var knockback_force: float = 200.0
var knockback_direction: Vector2 = Vector2.RIGHT
```

---

## 9. Performance

### Object Pooling for Frequent Spawns

```gdscript
# Bullets, particles, enemies — pool instead of instance/free
class_name ObjectPool
extends Node

@export var scene: PackedScene
@export var pool_size: int = 20

var _pool: Array[Node] = []

func _ready() -> void:
    for i in pool_size:
        var obj := scene.instantiate()
        obj.hide()
        add_child(obj)
        _pool.append(obj)

func get_object() -> Node:
    for obj in _pool:
        if not obj.visible:
            obj.show()
            return obj
    # Pool exhausted — optionally grow
    var obj := scene.instantiate()
    add_child(obj)
    _pool.append(obj)
    return obj

func return_object(obj: Node) -> void:
    obj.hide()
```

### Avoid Per-Frame Allocations

```gdscript
# Bad - allocates Vector2 every frame
func _process(delta: float) -> void:
    velocity = Vector2(Input.get_axis("left", "right"), 0) * speed

# Good - reuse, or use built-in approach
var _input_dir: Vector2 = Vector2.ZERO

func _process(delta: float) -> void:
    _input_dir.x = Input.get_axis("move_left", "move_right")
    velocity = _input_dir * speed
```

### Use `_physics_process` vs `_process` Correctly

- `_physics_process` — movement, collision, physics queries (fixed timestep)
- `_process` — visual updates, UI, input reading (variable timestep)
- Never put physics in `_process`; never put heavy rendering in `_physics_process`

### Groups for Batch Operations

```gdscript
# Tag nodes via editor or code:
add_to_group("enemies")

# Broadcast to all:
get_tree().call_group("enemies", "on_alarm_triggered")

# Get all:
var enemies := get_tree().get_nodes_in_group("enemies")
```

---

## 10. Scene Transitions and Loading

```gdscript
# autoloads/scene_manager.gd
class_name SceneManager
extends Node

func change_scene(path: String) -> void:
    get_tree().change_scene_to_file(path)

# Async loading for large scenes
func load_scene_async(path: String) -> void:
    ResourceLoader.load_threaded_request(path)
    # Poll with ResourceLoader.load_threaded_get_status()
    # Then call get_tree().change_scene_to_packed(...)
```

---

## 11. Common Mistakes to Avoid

| Mistake | Problem | Solution |
|---|---|---|
| Using `get_node()` paths everywhere | Breaks on refactor | Use `@onready` + typed vars |
| Deep inheritance chains | Rigid, hard to extend | Composition + signals |
| Putting logic in Autoloads | God object | Keep autoloads thin |
| `get_parent()` in child scripts | Tight coupling | Emit signals upward |
| Untyped GDScript | Runtime bugs | `static_vars = true` in project settings |
| Physics in `_process` | Non-deterministic | Use `_physics_process` |
| Instancing in `_process` | GC pressure | Object pooling |
| Magic numbers | Unreadable | Named constants / exports |

---

## 12. Project Settings to Always Configure

```
# project.godot recommended settings:
[physics]
2d/default_gravity=980          # Or 980.0 for realistic feel

[rendering]
textures/canvas_textures/default_texture_filter=0  # Nearest for pixel art

[display]
window/stretch/mode="canvas_items"   # For pixel art scaling
window/stretch/aspect="keep"

[input]                              # Always define input actions in project settings
# move_left, move_right, jump, attack, interact, pause
```

Enable in script editor:
- **Project Settings → GDScript → `static_vars`** → Enable for stricter typing

---

## Quick Reference

```gdscript
# Class header
class_name MyNode
extends Node2D

# Typed exports
@export var speed: float = 200.0

# Typed onready
@onready var sprite: Sprite2D = $Sprite2D

# Typed signal
signal health_changed(hp: int)

# Emit signal
health_changed.emit(health)

# Connect signal
child.health_changed.connect(_on_health_changed)

# State machine
match state:
    State.IDLE: _state_idle(delta)
    State.RUN:  _state_run(delta)

# Safe node check
if is_instance_valid(target):
    target.take_damage(damage)

# One-shot signal
signal.connect(callback, CONNECT_ONE_SHOT)
```

---

## References

- [Godot 4 Documentation](https://docs.godotengine.org/en/stable/)
- [GDScript Style Guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- [Godot Best Practices](https://docs.godotengine.org/en/stable/tutorials/best_practices/index.html)
- [GDQuest Open Source Projects](https://github.com/gdquest-demos)