---
layout: post
title: "Blastar clone - code analyse of my game"
date: 2026-03-14 00:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: blastar.png # Add image post (optional)
---
# game.gd - main mechanics
{% highlight python %}
extends Node2D

enum S { TITLE, INSTRUCTIONS, PLAYING, GAME_OVER }
{% endhighlight %}
extends Node2D means that this code is assgined to Node2D type of object in Godot scene
enum S is creating state machines for changing state of game

{% highlight python %}
@export var bullet_scene: PackedScene
@export var freighter_scene: PackedScene
@export var status_beam_scene: PackedScene
{% endhighlight %}
@export means that we will need to assign these variables in Godot inspector
PackedScene means that variable is Scene and we can't assign anythin other

{% highlight python %}
var state = S.TITLE
var score = 0
var ships = 5
var freighter: Node = null
var beam: Node = null
var p: CharacterBody2D
var fspd = 180.0
{% endhighlight %}
Here we are creating basic variables as current state, score, ships. We are also initate beam, freighter, p (player), fspd(freighter speed).

{% highlight python %}
@onready var sl: Label = $UI/ScoreLabel
@onready var hl: Label = $UI/ShipsLabel
@onready var ct: Label = $UI/CenterText
@onready var st: Label = $UI/SubText
@onready var bt: Label = $UI/StatusBeamText
{% endhighlight %}
@onready means that these variables will be assigned when scene will be loaded.
Here we are assigning UI text objects for score, ships, status beam, and other text.

{% highlight python %}
func _set_player(on: bool, pos := Vector2.ZERO):
	p.visible = on
	p.set_process(on)
	p.set_physics_process(on)

	if on:
		p.position = pos
		p.in_status_beam = false
{% endhighlight %}
Here we're creating function _set_player which is getting 2 arguments as bool and Vector2.
Bool is true or false variable and Vector2 is positioning variable.
p.visible = on means that visibility of player will have value of bool 'on'.
p.set_process(on) means that everything in function process related to player will be turned on or off based on value of bool 'on'.
Same with p.set_physics_process(on).
if on is true the position of player will get value of pos and status beam will be false.

{% highlight python %}
func _hud():
	sl.text = "SCORE " + str(score)
	hl.text = "SHIPS " + str(ships)
{% endhighlight %}
Function _hud() is updating score and ships value in UI.

{% highlight python %}
func _clear_beam():
	if beam and is_instance_valid(beam):
		beam.queue_free()
		beam = null
{% endhighlight %}
Function _clear_beam() is removing beam from the scene by beam.queue_free() and nulling beam variable if it is existing (is_instance_valid)
and beam is not null.

{% highlight python %}
func clean_bullets():
	for c in get_children():
		if c.is_in_group("bullets"):
			c.queue_free()
{% endhighlight %}
Function clean_bullets() is iterating through children of scene and removing every object which is in "bullets" group.

{% highlight python %}
func _cleanup():
	if freighter and is_instance_valid(freighter):
		freighter.queue_free()
		freighter = null

	_clear_beam()

	clean_bullets()
{% endhighlight %}
_cleanup() is removing freighter and nulling its variable if it is existing and variable is not null.
It's also initate _clear_beam() abd clean_bullets() functions.

{% highlight python %}
func _end_beam():
	bt.visible = false
	p.in_status_beam = false
	_clear_beam()
{% endhighlight %}
_end_beam() is making Status Beam text invisible and setting status beam to false.
Also it initiate _clear_beam() function.

{% highlight python %}
func _on_miss():
	_end_beam()
{% endhighlight %}
It seems useless but it's make code more clean.

{% highlight python %}
func _on_hit():
	_end_beam()
	_lose()
{% endhighlight %}
This function has same functionality(readability and merging functions) as previous one.

{% highlight python %}
func _on_kill():
	score += 80
	_hud()
	_end_beam()
	freighter = null

	await get_tree().create_timer(0.5).timeout

	if state == S.PLAYING:
		_spawn()
{% endhighlight %}
_on_kill() is function initated by killing enemy. It's increasing score by 80, updating HUD and making sure that status beam is false.
Then nulling enemy variable and wait 0.5s to spawn next enemy if state is still PLAYING.

{% highlight python %}
func _spawn():
	if freighter and is_instance_valid(freighter):
		freighter.queue_free()

	freighter = freighter_scene.instantiate()
	freighter.position = Vector2(50, 40 + randi_range(0, 180))
	freighter.freighter_destroyed.connect(_on_kill)

	add_child(freighter)
{% endhighlight %}
If freighter is in scene and is not null then we remove it and then we instantioate scene of freighter,
set its position to x:50 and y:40+random number between 0 and 180.
Then it's initiate _on_kill function if freighter will send signal "freighter_destroyed".
And then we are spawning enemy by add_child(freighter).

{% highlight python %}
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
{% endhighlight %}
show_title() is assigning text to the text labels like center text(ct), subtext(st).
It's making score, ships and status beam text invisible and ct, st visible.
State of game is set to TITLE and score is 0 and ships are 5.
Also there are initated functions as cleanup, set_player and hud.

{% highlight python %}
func show_instructions():
	state = S.INSTRUCTIONS

	ct.text = "MISSION"

	st.text = "USE ARROW KEYS FOR CONTROL\nAND SPACE BAR TO SHOOT\n\nDESTROY ALIEN FREIGHTER\nCARRYING DEADLY HYDROGEN BOMBS\nAND STATUS BEAM MACHINES\n\nPRESS SPACE TO BEGIN"
{% endhighlight %}
show_instructions() is setting state to instructions and center text and subtext.

{% highlight python %}
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
{% endhighlight %}
start_game() is changing state to PLAYING, score to 0 and ships to 5. Center text, Subtext and status beam text are invisible and 
score label and ships label are visible. Then we update HUD and put the player into the scene at position(576,500). Then spawn enemy.

{% highlight python %}
func show_game_over():
	state = S.GAME_OVER

	_cleanup()
	_set_player(false)

	bt.visible = false
	st.visible = false

	ct.text = "BLASTAR\n\nFLEET DESTROYED\n\nFINAL SCORE: " + str(score) + "\n\nPRESS SPACE FOR ANOTHER GAME"
	ct.visible = true
{% endhighlight %}
show_game_over() is changing state to GAME_OVER then it's making _cleanup(), turn off player, making subtext and Status Beam Text invisible.
Then setting center text with score value and making it visible.

{% highlight python %}
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
{% endhighlight %}
_fire_beam() is making status beam text visible. It's removing player bullets from scene, setting player x.position to enemy x.position.
Status beam is set to true, then we instantiate status beam scene in game and set its position to freighter x and y+20(to spawn under enemy).
Then if beam will miss player we initiate _on_miss and if it will hit then _on_hit. After that we adding this to game scene to show it up.
If beam is already in progress (if beam and is instance valid) then nothing happens.

{% highlight python %}
func shoot_rocket():
	if state != S.PLAYING:
		return
	
	clean_bullets()
	
	var r = bullet_scene.instantiate()
	r.global_position = p.global_position + Vector2(0, -25)

	add_child(r)
{% endhighlight %}
shoot_rocket() is cleaning other bullets and then instantiate bullet scene, then bullet scene got position of player with y-25
(to spawn 25px over player). Then we add bullet to the game scene.

{% highlight python %}
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
{% endhighlight %}
_lose() is decreasing ships(lifes) and updating HUD. If there is no more ships we go to game over screen.
If not we are removing enemy and player and after 1s we are respawning player and spawning new enemy.

{% highlight python %}
func _ready():
	p = $Player
	_set_player(false)
	show_title()
{% endhighlight %}
_redy() is function that is initiated once after scene is loaded. Here we are assigning Player object to p variable.
Player is turned off and we are going to tittle screen.

{% highlight python %}
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
{% endhighlight %}
_process is function that is initiated every frame that our PC is generating. d is delta that will let us to make some values same
for everyone despite different framerate. var space is bool value which is true if we hit space (it's default key for "ui_accept" key).
At the beggining we are in TITLE, after hiting space we are going to instructions, after hiting space we are going to playing.
In PLAYING we are going to _update(d) and if we lose we are going to game over and if we hit space here we are going back to title.

{% highlight python %}
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
{% endhighlight %}
If distance of x between player and enemy is less than 5px then status beam is started. Then we are moving enemy by adding speed*delta 
to its current position. If enemy will reach x=1100 then it will respawn at x=50 to rerun over the screen all over again. If state is not
PLAYING or if status beam is active then nothing will happen.
