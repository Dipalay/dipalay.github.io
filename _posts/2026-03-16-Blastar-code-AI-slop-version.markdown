---
layout: post
title: "Blastar clone – code analysis of my game"
date: 2026-03-14 00:32:20 +0300
description: Analysis of the main mechanics implemented in my Blastar clone written in Godot.
img: blastar.png
---

# game.gd – main mechanics

## Script base and game states

{% highlight python %}
extends Node2D

enum S { TITLE, INSTRUCTIONS, PLAYING, GAME_OVER }
{% endhighlight %}

`extends Node2D` means that this script is attached to a **Node2D object in the Godot scene**.

`enum S` defines a **state machine** used to control the game flow.

The game can be in four states:

| State        | Description          |
| ------------ | -------------------- |
| TITLE        | Title screen         |
| INSTRUCTIONS | Mission instructions |
| PLAYING      | Main gameplay        |
| GAME_OVER    | End screen           |

---

# Scene references

{% highlight python %}
@export var bullet_scene: PackedScene
@export var freighter_scene: PackedScene
@export var status_beam_scene: PackedScene
{% endhighlight %}

`@export` makes variables visible in the **Godot Inspector**, so scenes can be assigned in the editor.

`PackedScene` means the variable stores a **scene resource**, which can later be instantiated during gameplay.

These scenes represent:

* `bullet_scene` – player's rocket
* `freighter_scene` – enemy ship
* `status_beam_scene` – enemy beam attack

---

# Core game variables

{% highlight python %}
var state = S.TITLE
var score = 0
var ships = 5
var freighter: Node = null
var beam: Node = null
var p: CharacterBody2D
var fspd = 180.0
{% endhighlight %}

These variables store the main game data.

| Variable    | Description                         |
| ----------- | ----------------------------------- |
| `state`     | current game state                  |
| `score`     | player score                        |
| `ships`     | number of remaining lives           |
| `freighter` | reference to the enemy ship         |
| `beam`      | reference to the active status beam |
| `p`         | reference to the player             |
| `fspd`      | freighter movement speed            |

---

# UI references

{% highlight python %}
@onready var sl: Label = $UI/ScoreLabel
@onready var hl: Label = $UI/ShipsLabel
@onready var ct: Label = $UI/CenterText
@onready var st: Label = $UI/SubText
@onready var bt: Label = $UI/StatusBeamText
{% endhighlight %}

`@onready` means the variables are assigned **after the scene is loaded**.

These variables reference UI elements used to display:

* player score
* remaining ships
* center screen text
* instruction text
* status beam warning

---

# Player activation

{% highlight python %}
func _set_player(on: bool, pos := Vector2.ZERO):
p.visible = on
p.set_process(on)
p.set_physics_process(on)

```
if on:
	p.position = pos
	p.in_status_beam = false
```

{% endhighlight %}

This function enables or disables the player.

If `on` is `true`:

* the player becomes visible
* process and physics updates are enabled
* the player is moved to the given position
* the `in_status_beam` flag is reset

If `on` is `false`, the player becomes inactive.

---

# HUD update

{% highlight python %}
func _hud():
sl.text = "SCORE " + str(score)
hl.text = "SHIPS " + str(ships)
{% endhighlight %}

The `_hud()` function updates the **HUD labels** with the current score and number of ships.

---

# Removing the beam

{% highlight python %}
func _clear_beam():
if beam and is_instance_valid(beam):
beam.queue_free()
beam = null
{% endhighlight %}

This function safely removes the **status beam** from the scene and clears its reference.

---

# Cleaning bullets

{% highlight python %}
func clean_bullets():
for c in get_children():
if c.is_in_group("bullets"):
c.queue_free()
{% endhighlight %}

`clean_bullets()` iterates through the children of the scene and removes all nodes belonging to the `"bullets"` group.

---

# Cleaning gameplay objects

{% highlight python %}
func _cleanup():
if freighter and is_instance_valid(freighter):
freighter.queue_free()
freighter = null

```
_clear_beam()
clean_bullets()
```

{% endhighlight %}

This function removes active gameplay objects:

* the enemy freighter
* the status beam
* any remaining bullets

It is typically used when switching game states.

---

# Ending the beam

{% highlight python %}
func _end_beam():
bt.visible = false
p.in_status_beam = false
_clear_beam()
{% endhighlight %}

This function ends the **status beam event**.

It hides the warning text, resets the player beam state, and removes the beam.

---

# Beam result events

{% highlight python %}
func _on_miss():
_end_beam()
{% endhighlight %}

This function is called when the beam **misses the player**.

Its main purpose is to keep the code organized and readable.

---

{% highlight python %}
func _on_hit():
_end_beam()
_lose()
{% endhighlight %}

This function is called when the beam **hits the player**.

The beam ends and the player loses a ship.

---

# Destroying the freighter

{% highlight python %}
func _on_kill():
score += 80
_hud()
_end_beam()
freighter = null

```
await get_tree().create_timer(0.5).timeout

if state == S.PLAYING:
	_spawn()
```

{% endhighlight %}

This function runs when the enemy freighter is destroyed.

Steps performed:

1. Increase score by 80.
2. Update the HUD.
3. End any active beam.
4. Clear the freighter reference.
5. After a short delay, spawn a new enemy if the game is still running.

---

# Spawning the enemy

{% highlight python %}
func _spawn():
if freighter and is_instance_valid(freighter):
freighter.queue_free()

```
freighter = freighter_scene.instantiate()
freighter.position = Vector2(50, 40 + randi_range(0, 180))
freighter.freighter_destroyed.connect(_on_kill)

add_child(freighter)
```

{% endhighlight %}

This function creates a new enemy freighter.

Steps:

1. Remove any existing freighter.
2. Instantiate a new freighter scene.
3. Set its position to `x = 50` and a random `y` coordinate.
4. Connect the destruction signal.
5. Add the enemy to the scene.

---

# Title screen

{% highlight python %}
func show_title():
state = S.TITLE
score = 0
ships = 5

```
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

{% endhighlight %}

This function shows the **title screen** and resets the game.

---

# Instructions screen

{% highlight python %}
func show_instructions():
state = S.INSTRUCTIONS

```
ct.text = "MISSION"

st.text = "USE ARROW KEYS FOR CONTROL\nAND SPACE BAR TO SHOOT\n\nDESTROY ALIEN FREIGHTER\nCARRYING DEADLY HYDROGEN BOMBS\nAND STATUS BEAM MACHINES\n\nPRESS SPACE TO BEGIN"
```

{% endhighlight %}

This screen explains the controls and the objective of the game.

---

# Starting the game

{% highlight python %}
func start_game():
state = S.PLAYING
score = 0
ships = 5

```
ct.visible = false
st.visible = false
bt.visible = false

sl.visible = true
hl.visible = true

_hud()

_set_player(true, Vector2(576, 500))
_spawn()
```

{% endhighlight %}

This function starts a new game.

It resets the score and ships, enables the player, and spawns the first enemy.

---

# Game over screen

{% highlight python %}
func show_game_over():
state = S.GAME_OVER

```
_cleanup()
_set_player(false)

bt.visible = false
st.visible = false

ct.text = "BLASTAR\n\nFLEET DESTROYED\n\nFINAL SCORE: " + str(score) + "\n\nPRESS SPACE FOR ANOTHER GAME"
ct.visible = true
```

{% endhighlight %}

This function displays the **game over screen** and shows the final score.

---

# Status beam attack

{% highlight python %}
func _fire_beam():
if beam and is_instance_valid(beam):
return

```
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

{% endhighlight %}

This function activates the **status beam attack**.

Actions performed:

* show beam warning text
* remove player bullets
* align the player with the beam
* spawn the beam object
* connect hit and miss signals

---

# Shooting rockets

{% highlight python %}
func shoot_rocket():
if state != S.PLAYING:
return

```
clean_bullets()

var r = bullet_scene.instantiate()
r.global_position = p.global_position + Vector2(0, -25)

add_child(r)
```

{% endhighlight %}

This function spawns a rocket projectile slightly above the player.

Only one rocket exists at a time because previous bullets are removed before spawning a new one.

---

# Losing a ship

{% highlight python %}
func _lose():
ships -= 1
_hud()

```
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

{% endhighlight %}

This function handles player death.

If ships remain, the player respawns after 1 second and a new enemy is spawned.

If no ships remain, the game ends.

---

# Scene initialization

{% highlight python %}
func _ready():
p = $Player
_set_player(false)
show_title()
{% endhighlight %}

`_ready()` runs once after the scene loads.

It retrieves the player node and shows the title screen.

---

# Main update loop

{% highlight python %}
func _process(d):
var space = Input.is_action_just_pressed("ui_accept") or Input.is_action_just_pressed("ui_select")

```
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

{% endhighlight %}

`_process()` runs every frame and controls **state transitions** based on player input.

---

# Enemy movement and beam trigger

{% highlight python %}
func _update(d):
if state != S.PLAYING or not freighter or not is_instance_valid(freighter):
return

```
if beam and is_instance_valid(beam):
	return

if abs(freighter.position.x - p.position.x) < 5:
	_fire_beam()
	return

freighter.position.x += fspd * d

if freighter.position.x > 1100:
	freighter.position.x = 50
```

{% endhighlight %}

This function updates the enemy behavior.

If the freighter aligns with the player horizontally, the **status beam attack** is triggered.

Otherwise the enemy moves across the screen and reappears on the left side when reaching the right edge.
