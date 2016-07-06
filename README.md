# sar_game_command_msgs

A ROS package containing custom ROS messages for communication with SAR games.

# GameCommand

The GameCommand message is used to issue commands to SAR games. This message includes two fields, `game` and `command`, and a set of constants for these two fields.

The `game` field describes which game to take the commands.
TODO: This field is still under experiment.

The constants for this field are listed below.

- STORYTELLING (0)

TODO: more to add


The `command` field specifies the command that the receiving game should do. A list of constants for this field is summarized below.

- START (0)

Upon receiving this command, the game starts to play.

- CONTINUE (1)

This command is expected after a `PAUSE` command. Upon receiving this command, the game resumes to play.

- PAUSE (2)

This command puts the game on hold.

- END (3)

Upon receiving this command, the game exits gracefully (e.g., continue to play until the next reasonable exit point). Do not exit the game abruptly.

# GameState

The GameState message sends the state of the publishing game. This message includes two fields, `game` and `state`, and a set of constants for these two fields.
 
The game field is the same as the one in GameCommand described above.

The state field takes the following constants.

- START (0)
- IN_PROGRESS (1)
- PAUSED (2)
- USER_TIMEOUT (3)
- END (4)

# When to publish a game state

`GameState.START`: when receiving a GameCommand.START

`GameState.IN_PROGRESS`: when receiving a GameCommand.START and GameCommand.CONTINUE

`GameState.PAUSED`: when receiving a GameCommand.PAUSE

`GameState.TIMEOUT`: not receiving responses from the user

`GameState.END`: after wrapping up the game
