# Capture the Flag

*Worked as Gameplay Programmer and Pixel Artist*

11th of August 2025 until 17th of August 2025.

Capture the flag is an unreleased solo project. It is a LAN multiplayer game made in Godot. The players play in two teams, red and blue. Red has to kill the blue team while the blue team needs to get to the flags and then return to their base.

It was quite a challenge to work with multiplayer as I had never done so before.I did learn how server and clients work and how to host a dedicated server in Godot.

![Capture the Flag lobby and gameplay](CaptureTheFlag.gif)

# What I did
* I made a dedicated server to use for LAN multiplayer
* I made functions that transferred data between server and clients
* I made sure that the transferred data was done so safely to avoid cheating

# Code snippet

Here is a code snippet of the script that handles the lobby, such as when a player joins and leaves.

<details>
<summary>Show me!</summary>

```swift
...

func _ready() -> void:
	if not OS.has_feature("dedicated_server"):
		return
	
	print("Starting dedicated server.")
	_create_server()


func _create_server() -> void:
	peer = ENetMultiplayerPeer.new()
	var result := peer.create_server(PORT, MAX_PLAYERS)
	if result != OK: 
		print("Server creation failed.")
		return
	
	multiplayer.multiplayer_peer = peer
	
	multiplayer.peer_connected.connect(_on_peer_connected)
	multiplayer.peer_disconnected.connect(_on_peer_disconnected)

	print("Server created successfully.")


func _on_peer_connected(id: int) -> void:
	print("PID " + str(id) + " connected.")
	
	set_default_name(id)
	
	var main_menu: MainMenu = level_spawner.get_node("MainMenu")
	if main_menu == null: return
	
	for peer_id in multiplayer.get_peers():
		if id == peer_id:
			continue
		main_menu.add_player_box.rpc_id(id, peer_id, player_teams.get(peer_id), player_names.get(peer_id))
	
	player_teams.set(id, Enums.Team.Undecided)
	main_menu.add_player_box.rpc(id, player_teams.get(id), player_names.get(id))
	
	disable_ready_button.rpc()


func _on_peer_disconnected(id: int) -> void:
	print("PID " + str(id) + " disconnected.")
	
	var main_menu: MainMenu = level_spawner.get_node("MainMenu")
	if main_menu:
		main_menu.remove_player_box.rpc(id, player_teams.get(id))
		
	player_names.erase(id)
	player_teams.erase(id)


@rpc("any_peer")
func set_player_name(player_name: String) -> void:
	var id: int = multiplayer.get_remote_sender_id()
	player_names.set(id, player_name)
	var team: Enums.Team = player_teams.get(id)
	set_player_box_label.rpc(id, player_names.get(id), team)


func set_default_name(id: int) -> void:
	player_names.set(id, "Player" + str(player_index))


@rpc("call_local")
func set_player_box_label(id: int, player_name: String, team: Enums.Team) -> void:
	var container := get_team_container(team)
	
	if not container.get_node(str(id)):
		printerr("PlayerBox was not found in their team.")
		return
	
	var player_box: PlayerBox = container.get_node(str(id))
	player_box.label.text = player_name


func get_team_container(team: Enums.Team) -> VBoxContainer:
	var container: VBoxContainer

	match team:
		Enums.Team.Undecided:
			container = get_tree().get_first_node_in_group("UndecidedContainer")
		Enums.Team.Red:
			container = get_tree().get_first_node_in_group("TeamRedContainer")
		Enums.Team.Blue:
			container = get_tree().get_first_node_in_group("TeamBlueContainer")
		_:
			printerr("Team was not found.")
			return

	return container

...
```

</details>