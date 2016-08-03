# sar\_game\_command\_msgs

A ROS package containing custom ROS messages for communication with SAR games.

## GameCommand

The GameCommand message is used to issue commands to SAR games. This message
includes the following fields and a set of constants for these fields:

- `game`: A constant listing which game should pay attention the command.
  TODO: This field is still under experiment. The available games are:
    - STORYTELLING (0)
    - TODO: more to add

- `command`: Specifies the command for the receiving game. This command could
  be any of the following:
    - START (0): Upon receiving this command, the game starts to play.
    - CONTINUE (1): This command is expected after a `PAUSE` command. Upon
      receiving this command, the game resumes to play.
    - PAUSE (2): This command puts the game on hold.
    - END (3): Upon receiving this command, the game exits gracefully (e.g.,
      continue to play until the next reasonable exit point). Do not exit the
      game abruptly.

- `level`: Include when sending a `GameCommand.START` message. Specifies the
  level the game should start at.

## GameState

The GameState message sends the state of the publishing game. This message
includes the following fields and a set of constants for these fields:

- `game`: The same as the one in GameCommand described above.

- `state`: Set to one of the following constants (use cases listed below):
    - START (0)
    - IN\_PROGRESS (1)
    - PAUSED (2)
    - USER\_TIMEOUT (3)
    - END (4)

- `performance`: Include when you send a `GameState.END` message. Should
  contain the player's sucess rate for the game's primary target skill (e.g.,
  ordering/sequencing, perspective taking, emotional understanding) as a
  decimal percentage (i.e., the number of successes or correct answers divided
  by the number of opportunities for success).

### When to publish a GameState message

`GameState.START`: when receiving a GameCommand.START

`GameState.IN_PROGRESS`: when receiving a GameCommand.START and
GameCommand.CONTINUE

`GameState.PAUSED`: when receiving a GameCommand.PAUSE

`GameState.TIMEOUT`: not receiving responses from the user

`GameState.END`: after wrapping up the game
