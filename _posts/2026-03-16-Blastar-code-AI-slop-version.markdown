---
layout: post
title: "Blastar clone - code analysis (improved by AI)"
date: 2026-03-14 00:32:20 +0300
description: Code analysis of the main mechanics implemented in my Blastar clone written in Godot.
img: blastar.png
---

# game.gd - main mechanics

```python
extends Node2D

enum S { TITLE, INSTRUCTIONS, PLAYING, GAME_OVER }
```

`extends Node2D` means that this script is assigned to a **Node2D object in the Godot scene**.

`enum S` creates a **state machine** used for changing the state of the game.

---

```python
@export var bullet_scene: PackedScene
@export var freighter_scene: PackedScene
@export var status_beam_scene: PackedScene
```

`@export` means that these variables will need to be assigned in the **Godot Inspector**.

`PackedScene` means that the variable stores a **scene resource**, so we cannot assign anything other than a scene.

---

```python
var state = S.TITLE
var score = 0
var ships = 5
var freighter: Node = null
var beam: Node = null
var p: CharacterBody2D
var fspd = 180.0
```

Here we create the basic variables such as the current state, score, and ships (lives).

We also initialize references for:

- `beam`
- `freighter`
- `p` (player)
- `fspd` (freighter speed)

---

```python
@onready var sl: Label = $UI/ScoreLabel
@onready var hl: Label = $UI/ShipsLabel
@onready var ct: Label = $UI/CenterText
@onready var st: Label = $UI/SubText
@onready var bt: Label = $UI/StatusBeamText
```

`@onready` means that these variables will be assigned **when the scene is loaded**.

Here we assign UI text objects used for displaying:

- score
- ships
- status beam text
- center text
- subtext

---

```python
func _set_player(on: bool, pos := Vector2.ZERO):
	p.visible = on
	p.set_process(on)
	p.set_physics_process(on)

	if on:
		p.position = pos
		p.in_status_beam = false
```

Here we create the `_set_player` function, which takes two arguments:

- `on` – a boolean value
- `pos` – a `Vector2` position

`p.visible = on` means that the player's visibility will match the value of `on`.

`p.set_process(on)` enables or disables the `_process` logic of the player.

`p.set_physics_process(on)` enables or disables the physics logic of the player.

If `on` is `true`, the player's position is set to `pos`, and the `in_status_beam` flag is reset.

---

```python
func _hud():
	sl.text = "SCORE " + str(score)
	hl.text = "SHIPS " + str(ships)
```

The `_hud()` function updates the score and ships values in the UI.

---

```python
func _clear_beam():
	if beam and is_instance_valid(beam):
		beam.queue_free()
		beam = null
```

The `_clear_beam()` function removes the beam from the scene using `beam.queue_free()` and sets the `beam` variable to `null` if the beam exists and is valid.

---

```python
func clean_bullets():
	for c in get_children():
		if c.is_in_group("bullets"):
			c.queue_free()
```

The `clean_bullets()` function iterates through all children of the scene and removes every object that belongs to the `"bullets"` group.

---

```python
func _cleanup():
	if freighter and is_instance_valid(freighter):
		freighter.queue_free()
		freighter = null

	_clear_beam()

	clean_bullets()
```

`_cleanup()` removes the freighter and clears its reference if it exists.

It also calls `_clear_beam()` and `clean_bullets()` to remove the beam and all bullets.

---

```python
func _end_beam():
	bt.visible = false
	p.in_status_beam = false
	_clear_beam()
```

`_end_beam()` hides the Status Beam text, disables the status beam state for the player, and clears the beam object.

---

```python
func _on_miss():
	_end_beam()
```

This function may seem unnecessary, but it makes the code cleaner and easier to read.

---

```python
func _on_hit():
	_end_beam()
	_lose()
```

This function has a similar role as the previous one.  
It improves readability and separates logic for different events.

---

```python
func _on_kill():
	score += 80
	_hud()
	_end_beam()
	freighter = null

	await get_tree().create_timer(0.5).timeout

	if state == S.PLAYING:
		_spawn()
```

`_on_kill()` is called when the enemy is destroyed.

It increases the score by 80, updates the HUD, and ensures that the status beam is disabled.

Then it clears the enemy reference and waits 0.5 seconds before spawning a new enemy if the game state is still `PLAYING`.

---

```python
func _spawn():
	if freighter and is_instance_valid(freighter):
		freighter.queue_free()

	freighter = freighter_scene.instantiate()
	freighter.position = Vector2(50, 40 + randi_range(0, 180))
	freighter.freighter_destroyed.connect(_on_kill)

	add_child(freighter)
```

If a freighter already exists in the scene, it is removed.

Then a new freighter scene is instantiated.

Its position is set to:

- `x = 50`
- `y = 40 + random value between 0 and 180`

The `freighter_destroyed` signal is connected to `_on_kill`.

Finally, the enemy is spawned by adding it to the scene.

---

```python
func show_title():
	state = S.TITLE
	score = 0
	ships = 5

	_cleanup()
	_set_player(false)
	_hud()

	sl.visible = false
	hl.visible = false
	bt.visible = false

	ct.text = "BLASTAR"
	ct.visible = true

	st.text = "BY E.R.MUSK\n\nPRESS SPACE TO START"
	st.visible = true
```

`show_title()` sets the game state to `TITLE` and resets the score and ships.

It also runs `_cleanup()`, disables the player, and updates the HUD.

Score, ships, and status beam texts are hidden, while the title and subtitle are displayed.

---

```python
func show_instructions():
	state = S.INSTRUCTIONS

	ct.text = "MISSION"

	st.text = "USE ARROW KEYS FOR CONTROL\nAND SPACE BAR TO SHOOT\n\nDESTROY ALIEN FREIGHTER\nCARRYING DEADLY HYDROGEN BOMBS\nAND STATUS BEAM MACHINES\n\nPRESS SPACE TO BEGIN"
```

`show_instructions()` changes the state to `INSTRUCTIONS` and displays the mission text.

---

```python
func start_game():
	state = S.PLAYING
	score = 0
	ships = 5

	ct.visible = false
	st.visible = false
	bt.visible = false

	sl.visible = true
	hl.visible = true

	_hud()

	_set_player(true, Vector2(576, 500))
	_spawn()
```

`start_game()` changes the state to `PLAYING` and resets score and ships.

Center text, subtext, and status beam text are hidden, while score and ships labels are visible.

The HUD is updated, the player is placed at position `(576, 500)`, and the first enemy is spawned.

---

```python
func show_game_over():
	state = S.GAME_OVER

	_cleanup()
	_set_player(false)

	bt.visible = false
	st.visible = false

	ct.text = "BLASTAR\n\nFLEET DESTROYED\n\nFINAL SCORE: " + str(score) + "\n\nPRESS SPACE FOR ANOTHER GAME"
	ct.visible = true
```

`show_game_over()` sets the state to `GAME_OVER`, cleans up the scene, and disables the player.

It hides the subtext and status beam text, then displays the final score.

---

```python
func _fire_beam():
	if beam and is_instance_valid(beam):
		return

	bt.visible = true
	
	clean_bullets()

	p.beam_anchor_x = freighter.position.x
	p.in_status_beam = true

	beam = status_beam_scene.instantiate()
	beam.position = Vector2(freighter.position.x, freighter.position.y + 20)

	beam.beam_missed.connect(_on_miss)
	beam.beam_hit_player.connect(_on_hit)

	add_child(beam)
```

`_fire_beam()` activates the status beam attack.

It shows the status beam text, removes player bullets, and aligns the player with the enemy's X position.

Then the status beam scene is instantiated and positioned below the enemy.

Signals are connected for beam hit and beam miss events.

If a beam already exists, nothing happens.

---

```python
func shoot_rocket():
	if state != S.PLAYING:
		return
	
	clean_bullets()
	
	var r = bullet_scene.instantiate()
	r.global_position = p.global_position + Vector2(0, -25)

	add_child(r)
```

`shoot_rocket()` removes existing bullets and spawns a new rocket.

The rocket appears slightly above the player (`y - 25`) and is then added to the scene.

---

```python
func _lose():
	ships -= 1
	_hud()

	if ships <= 0:
		show_game_over()
		return

	if freighter and is_instance_valid(freighter):
		freighter.queue_free()
		freighter = null

	_set_player(false)

	await get_tree().create_timer(1.0).timeout

	if state == S.PLAYING:
		_set_player(true, Vector2(576, 500))
		_spawn()
```

`_lose()` decreases the number of ships and updates the HUD.

If there are no ships left, the game switches to the game over screen.

Otherwise the enemy and player are removed, and after one second the player respawns and a new enemy appears.

---

```python
func _ready():
	p = $Player
	_set_player(false)
	show_title()
```

`_ready()` runs once after the scene loads.

Here we assign the Player node to the variable `p`, disable the player, and show the title screen.

---

```python
func _process(d):
	var space = Input.is_action_just_pressed("ui_accept") or Input.is_action_just_pressed("ui_select")

	match state:
		S.TITLE:
			if space:
				show_instructions()

		S.INSTRUCTIONS:
			if space:
				start_game()

		S.PLAYING:
			_update(d)

		S.GAME_OVER:
			if space:
				show_title()
```

`_process()` runs every frame.

The variable `d` (delta) helps keep movement consistent regardless of frame rate.

The variable `space` becomes true if the player presses the space key.

The state machine controls transitions between:

- title
- instructions
- gameplay
- game over

---

```python
func _update(d):
	if state != S.PLAYING or not freighter or not is_instance_valid(freighter):
		return

	if beam and is_instance_valid(beam):
		return

	if abs(freighter.position.x - p.position.x) < 5:
		_fire_beam()
		return

	freighter.position.x += fspd * d

	if freighter.position.x > 1100:
		freighter.position.x = 50
```

This function updates the enemy behaviour.

If the horizontal distance between the player and the enemy is less than 5 pixels, the status beam attack starts.

Otherwise the enemy moves across the screen.

When the enemy reaches `x = 1100`, it returns to `x = 50` and repeats the movement.
