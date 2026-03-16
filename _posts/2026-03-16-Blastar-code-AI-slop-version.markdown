---

layout: post
title: "Blastar Clone – Architecture and Gameplay Analysis"
date: 2026-03-14 00:32:20 +0300
description: Technical breakdown of the gameplay architecture used in my Blastar clone made in Godot.
img: blastar.png
----------------

# Blastar Clone – Architecture and Gameplay Analysis

This post explains the internal architecture and gameplay systems used in my **Blastar clone** written in **Godot using GDScript**.

Blastar is notable because it was **Elon Musk’s first game**, originally written in **BASIC in 1984**.
My goal was to recreate the core mechanics while keeping the implementation clean and modular.

The entire gameplay logic is handled inside the main script:

```
game.gd
```

---

# Game Architecture

The main script acts as the **central game controller**.

It manages:

* game states
* player spawning
* enemy spawning
* scoring
* UI updates
* special mechanics (Status Beam)

The script is attached to the main scene node.

```python
extends Node2D
```

---

# Game State Machine

The game uses a **simple state machine** to control the flow.

```python
enum S { TITLE, INSTRUCTIONS, PLAYING, GAME_OVER }
```

Game states:

| State        | Description      |
| ------------ | ---------------- |
| TITLE        | Title screen     |
| INSTRUCTIONS | Mission briefing |
| PLAYING      | Main gameplay    |
| GAME_OVER    | End screen       |

State transitions look like this:

```
TITLE
  ↓
INSTRUCTIONS
  ↓
PLAYING
  ↓
GAME OVER
  ↓
TITLE
```

This system keeps the logic organized and avoids mixing gameplay code with menu logic.

---

# Scene References

The game dynamically spawns several scenes during runtime.

```python
@export var bullet_scene: PackedScene
@export var freighter_scene: PackedScene
@export var status_beam_scene: PackedScene
```

These scenes are assigned in the **Godot Inspector**.

| Scene             | Purpose           |
| ----------------- | ----------------- |
| bullet_scene      | Player rocket     |
| freighter_scene   | Enemy ship        |
| status_beam_scene | Enemy beam attack |

Using `PackedScene` allows objects to be instantiated dynamically.

---

# Core Runtime Variables

The game tracks several key variables.

```python
var state = S.TITLE
var score = 0
var ships = 5
var freighter: Node = null
var beam: Node = null
var p: CharacterBody2D
var fspd = 180.0
```

Important variables:

| Variable  | Description          |
| --------- | -------------------- |
| state     | Current game state   |
| score     | Player score         |
| ships     | Remaining lives      |
| freighter | Active enemy         |
| beam      | Active status beam   |
| p         | Player reference     |
| fspd      | Enemy movement speed |

---

# UI System

UI labels are loaded once the scene is ready.

```python
@onready var sl: Label = $UI/ScoreLabel
@onready var hl: Label = $UI/ShipsLabel
@onready var ct: Label = $UI/CenterText
@onready var st: Label = $UI/SubText
@onready var bt: Label = $UI/StatusBeamText
```

These labels display:

* score
* remaining ships
* mission text
* title screen messages
* beam warnings

HUD updates are handled by a dedicated function:

```python
func _hud():
	sl.text = "SCORE " + str(score)
	hl.text = "SHIPS " + str(ships)
```

---

# Player Activation System

The player can be enabled or disabled depending on the game state.

```python
func _set_player(on: bool, pos := Vector2.ZERO):
	p.visible = on
	p.set_process(on)
	p.set_physics_process(on)

	if on:
		p.position = pos
		p.in_status_beam = false
```

When enabled:

* player becomes visible
* movement logic runs
* physics updates start
* player position is set

When disabled, the player is effectively paused.

---

# Enemy Spawning System

Enemies are created dynamically using scene instancing.

```python
func _spawn():
	if freighter and is_instance_valid(freighter):
		freighter.queue_free()

	freighter = freighter_scene.instantiate()
	freighter.position = Vector2(50, 40 + randi_range(0, 180))
	freighter.freighter_destroyed.connect(_on_kill)

	add_child(freighter)
```

Spawn behavior:

1. Remove previous freighter
2. Instantiate a new one
3. Randomize vertical position
4. Connect destruction signal
5. Add enemy to scene

---

# Enemy Movement System

The freighter constantly moves horizontally.

```python
freighter.position.x += fspd * delta
```

Once it reaches the right side of the screen:

```python
if freighter.position.x > 1100:
	freighter.position.x = 50
```

It wraps around to the left side, creating continuous movement.

---

# Status Beam Mechanic

The **Status Beam** is the main enemy attack.

It triggers when the enemy aligns with the player horizontally.

```python
if abs(freighter.position.x - p.position.x) < 5:
	_fire_beam()
```

When activated:

1. Player bullets are cleared
2. Player movement becomes restricted
3. Beam object is spawned
4. Signals determine the outcome

```python
func _fire_beam():
	if beam and is_instance_valid(beam):
		return

	bt.visible = true
	clean_bullets()

	p.beam_anchor_x = freighter.position.x
	p.in_status_beam = true
```

The beam scene emits signals:

* `beam_missed`
* `beam_hit_player`

These signals determine whether the player survives or loses a ship.

---

# Shooting System

The player fires rockets using the `shoot_rocket()` function.

```python
func shoot_rocket():
	if state != S.PLAYING:
		return
	
	clean_bullets()
	
	var r = bullet_scene.instantiate()
	r.global_position = p.global_position + Vector2(0, -25)

	add_child(r)
```

Rocket spawning behavior:

* appears slightly above the player
* only one rocket can exist at a time
* previous rockets are cleared before spawning

---

# Scoring System

Destroying a freighter awards **80 points**.

```python
func _on_kill():
	score += 80
	_hud()
```

After a short delay, a new enemy is spawned.

---

# Player Death and Respawn

If the player is hit by the beam:

```
ships -= 1
```

If ships remain:

* player respawns after 1 second
* a new enemy appears

If ships reach zero, the game transitions to the **Game Over** screen.

---

# Gameplay Loop

The main gameplay loop is controlled inside `_process()`.

```
Title Screen
     ↓
Instructions
     ↓
Gameplay
     ↓
Game Over
     ↓
Restart
```

This structure separates **menu logic from gameplay logic**, making the code easier to maintain.

---

# Conclusion

This Blastar clone demonstrates several core game development concepts:

* state machine architecture
* dynamic scene instancing
* signal-based event handling
* modular gameplay systems

Even though the game is simple, it shows how a small project can still benefit from **clean architecture and structured gameplay logic**.

Future improvements could include:

* multiple enemies
* increasing difficulty
* power-ups
* sound effects and particle systems
* score leaderboard
