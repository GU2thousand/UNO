You must support the following:

-   Determine whether a given card can be played
    
-   Only the current player is allowed to take actions
    
-   After a card is played, update the top card of the discard pile
    
-   If no playable card is available, the player may draw one card
    
-   If the drawn card is playable, the player may either play it immediately or end their turn
    
-   Turn order must advance correctly
    
-   The game ends when a player's hand reaches 0 cards
    
-   If a non-current player attempts an action, the server must reject it
    

This phase is extremely important because your proposal explicitly states that the server is the authoritative source of truth and is responsible for legality checks and state transitions. This stage is where that statement becomes real.

What you will need in this phase

**Rules layer**

-   `RuleEngine`
    
-   `canPlay(Card card, GameState state)`
    
-   `playCard(...)`
    
-   `drawCard(...)`
    
-   `advanceTurn(...)`
    
-   `checkWinner(...)`
    

**Server**

-   `synchronized` or a room-level lock
    
-   Ensure that only one action is committed at a time
    

**Client**

-   Clicking a hand card sends `PLAY_CARD`
    
-   Clicking the draw button sends `DRAW_CARD`
    
-   Refresh the UI after receiving the updated game state
    

**Message protocol additions**

-   `PLAY_CARD`
    
-   `DRAW_CARD`
    
-   `INVALID_MOVE`
    
-   `TURN_UPDATE`
    
-   `GAME_OVER`
    

What must be implemented at the code level

**RuleEngine must handle:**

-   Color matching
    
-   Number matching
    
-   Type matching
    
-   Basic Wild card play validation
    
-   Updating the hand and deck after drawing
    

**GameRoom must handle:**

-   Receiving `PLAY_CARD`
    
-   Calling the rule engine to validate the move
    
-   Broadcasting the latest state if the move succeeds
    
-   Sending `INVALID_MOVE` only to the player who initiated the invalid action
