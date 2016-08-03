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
  be any of the following (use cases described below):
    - START (0)
    - CONTINUE (1)
    - PAUSE (2)
    - END (3)
    - WAIT\_FOR\_RESPONSE (4)
    - SKIP\_RESPONSE (5)

- `level`: Include when sending a `GameCommand.START` message. Specifies the
  level the game should start at.

### Expected game behavior

Upone receiving each command, the game should...

`GameCommand.START`: Start to play.

`GameCommand.CONTINUE`: Resume play. This command is expected to be send after
a `GameCommand.PAUSE` command.

`GameCommand.PAUSE`: Pause game play until a `GameState.CONTINUE` message is
received.

`GameCommand.END`: Exits gameplay gracefully. That is, continue to play until
the next reasonable exit point. Do not exit the game abruptly.

`GameCommand.WAIT_FOR_RESPONSE`: Wait for a player response before continuing.
If the game receives a player response, it can continue as normal; if it does
not, it should send a `GameState.USER_TIMEOUT` message to indicate that no
response was received.

`GameCommand.SKIP\_RESPONSE`: Resume play, optionally taking additional action
to reflect the fact that an expected user response was not received.

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

A game should publish `GameCommand` messages at the following times:

`GameState.START`: When a `GameCommand.START` message has been received and the
game is starting.

`GameState.IN_PROGRESS`: When a `GameCommand.START`, `GameCommand.CONTINUE`, or
`GameCommand.SKIP\_RESPONSE` message has been received, to indicate that the
game is continuing play now.

`GameState.PAUSED`: When a `GameCommand.PAUSE` message has been received and
the game is paused.

`GameState.USER_TIMEOUT`: If a response was expected from the user, but no
response was received within a reasonable amount of time. The game should PAUSE
after it sends this message to wait for instructions regarding whether it
should wait for a response again or skip the response and move on.

`GameState.END`: After wrapping up the game, when gameplay has ended.
