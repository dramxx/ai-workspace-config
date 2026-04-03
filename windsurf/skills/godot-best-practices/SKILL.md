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

## Project Structure

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

## GDScript Patterns

### 1. Strong Typing
Always specify types for better performance and clarity:

```gdscript
# Bad
func process_damage(amount):
    health -= amount

# Good
func process_damage(amount: int) -> void:
    health -= amount
```

### 2. Use @onready
Defer node references until the node is ready:

```gdscript
class_name Player
extends CharacterBody2D

@onready var sprite: Sprite2D = $Sprite2D
@onready var collision: CollisionShape2D = $CollisionShape2D
@onready var state_machine: StateMachine = $StateMachine

func _ready() -> void:
    # Nodes are now guaranteed to exist
    sprite.modulate = Color.WHITE
```

### 3. Signals Best Practices

**Define signals clearly:**

```gdscript
class_name HealthComponent
extends Node

signal health_changed(current: int, maximum: int)
signal health_depleted
signal damage_taken(amount: int, source: Node)

func take_damage(amount: int, source: Node) -> void:
    health -= amount
    damage_taken.emit(amount, source)
    health_changed.emit(health, max_health)
    
    if health <= 0:
        health_depleted.emit()
```

**Connect signals in _ready():**

```gdscript
func _ready() -> void:
    health_component.health_changed.connect(_on_health_changed)
    health_component.health_depleted.connect(_on_health_depleted)

func _on_health_changed(current: int, maximum: int) -> void:
    health_bar.value = float(current) / maximum

func _on_health_depleted() -> void:
    state_machine.transition_to("death")
```

### 4. State Machines

Use a dedicated state machine component:

```gdscript
class_name StateMachine
extends Node

@export var initial_state: State

var current_state: State
var states: Dictionary = {}

func _ready() -> void:
    for child in get_children():
        if child is State:
            states[child.name.to_lower()] = child
            child.transition_requested.connect(_on_transition_requested)
    
    if initial_state:
        transition_to(initial_state.name)

func transition_to(state_name: String) -> void:
    var new_state: State = states[state_name.to_lower()]
    if not new_state:
        return
    
    if current_state:
        current_state.exit()
    
    current_state = new_state
    current_state.enter()

func _on_transition_requested(state_name: String) -> void:
    transition_to(state_name)
```

## Scene Architecture

### 1. Composition Over Inheritance
Build complex behavior from components:

```gdscript
# Player scene structure
Player (CharacterBody2D)
├── CollisionShape2D
├── Sprite2D
├── HealthComponent
├── MovementComponent
├── AnimationPlayer
└── StateMachine
    ├── IdleState
    ├── WalkState
    └── JumpState
```

### 2. Use Resources for Data
Separate data from behavior:

```gdscript
class_name WeaponData
extends Resource

@export var damage: int
@export var range: float
@export var fire_rate: float
@export var bullet_scene: PackedScene

class_name Weapon
extends Node2D

@export var weapon_data: WeaponData

func fire() -> void:
    var bullet: Bullet = weapon_data.bullet_scene.instantiate()
    bullet.damage = weapon_data.damage
    add_child(bullet)
```

### 3. Autoloads for Global State
Use autoloads sparingly for truly global data:

```gdscript
# scripts/autoloads/game_manager.gd
extends Node

signal game_paused
signal game_resumed
signal player_died

var current_level: int = 0
var score: int = 0
var is_paused: bool = false

func pause_game() -> void:
    is_paused = true
    get_tree().paused = true
    game_paused.emit()

func resume_game() -> void:
    is_paused = false
    get_tree().paused = false
    game_resumed.emit()
```

## Performance Optimization

### 1. Use _process() vs _physics_process()
```gdscript
# _process() for frame-based updates (animations, UI)
func _process(delta: float) -> void:
    animation_player.advance(delta)

# _physics_process() for physics-based updates
func _physics_process(delta: float) -> void:
    move_and_slide()
```

### 2. Object Pooling
Reuse objects instead of creating/destroying:

```gdscript
class_name BulletPool
extends Node

@export var bullet_scene: PackedScene
@export var pool_size: int = 50

var available_bullets: Array[Bullet] = []
var active_bullets: Array[Bullet] = []

func _ready() -> void:
    for i in pool_size:
        var bullet: Bullet = bullet_scene.instantiate()
        add_child(bullet)
        bullet.visible = false
        available_bullets.append(bullet)

func get_bullet() -> Bullet:
    var bullet: Bullet
    if available_bullets.size() > 0:
        bullet = available_bullets.pop_back()
    else:
        bullet = bullet_scene.instantiate()
        add_child(bullet)
    
    bullet.visible = true
    active_bullets.append(bullet)
    return bullet

func return_bullet(bullet: Bullet) -> void:
    bullet.visible = false
    active_bullets.erase(bullet)
    available_bullets.append(bullet)
```

### 3. Use Groups for Organization
```gdscript
# In enemy scene
func _ready() -> void:
    add_to_group("enemies")
    add_to_group("damageable")

# Find all enemies
var enemies: Array[Node] = get_tree().get_nodes_in_group("enemies")

# Deal damage to all damageable objects
for node in get_tree().get_nodes_in_group("damageable"):
    if node.has_method("take_damage"):
        node.take_damage(10)
```

## Input Handling

### 1. Use Input Map
Define actions in Project Settings → Input Map:

```gdscript
func _process(delta: float) -> void:
    if Input.is_action_just_pressed("jump"):
        jump()
    
    var direction: Vector2 = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    velocity = direction * speed
```

### 2. Buffer Inputs
For better responsiveness:

```gdscript
class_name InputBuffer
extends Node

var jump_buffer: bool = false
var jump_buffer_timer: float = 0.0

func _process(delta: float) -> void:
    if Input.is_action_just_pressed("jump"):
        jump_buffer = true
        jump_buffer_timer = 0.1
    
    if jump_buffer_timer > 0:
        jump_buffer_timer -= delta
        if jump_buffer_timer <= 0:
            jump_buffer = false

func consume_jump() -> bool:
    if jump_buffer:
        jump_buffer = false
        jump_buffer_timer = 0
        return true
    return false
```

## Common Patterns

### 1. Singleton Pattern (Autoload)
```gdscript
# scripts/autoloads/audio_manager.gd
extends Node

var audio_player: AudioStreamPlayer2D

func _ready() -> void:
    audio_player = AudioStreamPlayer2D.new()
    add_child(audio_player)

func play_sound(sound: AudioStream) -> void:
    audio_player.stream = sound
    audio_player.play()
```

### 2. Observer Pattern (Signals)
```gdscript
class_name AchievementSystem
extends Node

signal achievement_unlocked(name: String, description: String)

func check_achievements() -> void:
    if GameManager.score >= 1000:
        achievement_unlocked.emit("High Scorer", "Score 1000 points")
```

### 3. Factory Pattern
```gdscript
class_name EnemyFactory
extends Node

@export var enemy_scenes: Array[PackedScene]

func create_enemy(enemy_type: String, position: Vector2) -> Enemy:
    for scene in enemy_scenes:
        var enemy: Enemy = scene.instantiate()
        if enemy.enemy_type == enemy_type:
            enemy.global_position = position
            return enemy
    return null
```

## Debugging Tips

### 1. Use assert() for Development
```gdscript
func _ready() -> void:
    assert(sprite != null, "Sprite2D node not found!")
    assert(health_component != null, "HealthComponent not found!")
```

### 2. Debug Draw
```gdscript
func _draw() -> void:
    if Engine.is_editor_hint():
        # Draw attack range in editor
        draw_circle(Vector2.ZERO, attack_range, Color.RED, false, 2.0)
```

### 3. Print with Context
```gdscript
func take_damage(amount: int) -> void:
    print("[%s] Taking %d damage, health: %d/%d" % [name, amount, health, max_health])
    health -= amount
```

## Best Practices Summary

1. **Strong typing** for performance and clarity
2. **@onready** for safe node references
3. **Components** over deep inheritance
4. **Resources** for data separation
5. **Signals** for loose coupling
6. **State machines** for complex behavior
7. **Object pooling** for performance
8. **Groups** for organization
9. **Input map** for configurable controls
10. **Autoloads** sparingly for true globals

These patterns will help you build maintainable, performant Godot games that scale well as your project grows.
