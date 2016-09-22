# sar\_game\_command\_msgs

A ROS package containing custom ROS messages for communication with SAR games.

## GameCommand

The GameCommand message is used to issue commands to SAR games. This message
includes the following fields and a set of constants for these fields:

- `game`: A constant listing which game should pay attention the command.
  TODO: This field is still under experiment. The available games are:
    - STORYTELLING (0)
    - ROCKET_BARRIER (1)
    - GALACTIC_TRAVELER (2)
    - SPACESHIP_TIDYUP (3)
    - ALIEN_CODES (4)
    - HOUSE_PERSPECTIVE_TAKING (5)
    - TRAIN_SEQUENCING (6)

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

`GameCommand.SKIP_RESPONSE`: Resume play, optionally taking additional action
to reflect the fact that an expected user response was not received.

## GameState

The GameState messages are sent by games to indicate what's happening in the
game. This message includes the following fields and a set of constants for
these fields:

- `game`: The same as for `GameCommand`, described above.

- `state`: Set to one of the following constants (use cases listed below):
    - START (0)
    - IN\_PROGRESS (1)
    - PAUSED (2)
    - USER\_TIMEOUT (3)
    - END (4)
    - READY (5)

- `performance`: Include when you send a `GameState.END` message. Should
  contain the player's sucess rate(s) for the game's primary target skill (e.g.,
  ordering/sequencing, perspective taking, emotional understanding) as a
  decimal percentage (i.e., the number of successes or correct answers divided
  by the number of opportunities for success). The perfomance field should be
  a JSON string with names of performance metrics mapped to their associated
  values.

### Performance metrics

#### Metrics reported by each game

The possible performance metrics that each game can report are as follows:

- Rocket-Barrier Game:
    - `child-explainer-location-accuracy` : In the version of the game where
      the child explains the rocket for the parent to build, the proportion of
      pieces of the parent's constructed rocket that are correct in both piece
      type and position.
    - `child-explainer-piece-choice-accuracy` : In the version of the game
      where the child explains the rocket for the parent to build, the
      proportion of pieces on the rocket that are correct in piece type, but
      not necessarily piece position.
    - `child-builder-location-accuracy` : In the version of the game where
      the parent explains the rocket for the child to build, the proportion of
      pieces of the child's constructed rocket that are correct in both piece
      type and position.
    - `child-builder-piece-choice-accuracy` : In the version of the game
      where the parent explains the rocket for the child to build, the
      proportion of pieces on the child's constructed rocket that are correct
      in piece type, but not necessarily piece position.

- Storytelling Game:
    - `child-emotion-question-accuracy` : The percentage of emotion questions
      that the child answered correctly this session. These multiple-choice
      questions ask the child to identify which emotions were felt by
      characters in the stories.
    - `child-tom-question-accuracy` : The percentage of theory of mind (ToM)
      questions that the child answered correctly this session. These
      multiple-choice questions ask the child to identify what emotion a
      character in a story might feel next.
    - `child-order-question-accuracy` : The percentage of order questions that
      the child answered correctly this session. These multiple-choice
      questions ask the child to identify in which scene of the story different
      events happened, such as what happened first or when a particular
      character felt sad.

#### Examples

The following is an example of the performance field for the Rocket-Barrier
game:

> {  
>    "child-explainer-location-accuracy" : 0.8,  
>    "child-explainer-piece-choice-accuracy" : 0.6  
> }  

The following is an example of the performance field for the Storytelling game.
Note that depending on the child's level in the Storytelling game, not all
types of questions will be asked. The performance metric for a question type
will only be reported if that type of question was asked that session. Thus,
sometimes only the `child-emotion-question-accuracy` will be reported,
sometimes both the `child-emotion-question-accuracy` and
`child-order-question-accuracy` will be reported, and sometimes all three will
be reported.

> {  
>   "child-emotion-question-accuracy" : 0.7,  
>   "child-tom-question-accuracy" : 0.0,  
>   "child-order-question-accuracy" : 0.5  
> }  

The following empty json string should be sent in the performance field if the game 
ends and there is no performance information to report (rather than sending nothing 
in this field).

> {
> }


### When to publish a GameState message

A game should publish `GameState` messages at the following times:

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

`GameState.READY`: When a game is up and running and is ready to receive a 
`GameCommand.START` message (when the game is ready to be started and played).   
