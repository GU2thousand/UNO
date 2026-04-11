## Phase 4: Bot



## Additional UI Requirements

### 1. Card Text Overflow

-   Some card labels (e.g., **Reverse**) exceed the card boundaries in the UI
-   Adjust font size dynamically to ensure text fits within the card

----------

### 2. Hand Layout Improvement

-   The current GUI displays cards in a single horizontal row
-   Update the hand area to support **multi-line wrapping layout**
-   Cards should automatically wrap to the next line when space is insufficient

### Objective

Add a bot.

The bot is a server-side player that automatically performs valid actions.

----------

### Definition of Completion

-   A room can include bot players
-   When it is the bot’s turn, no human interaction is required
-   The bot scans its hand in order
-   It plays the **first valid card** it finds
-   If no valid card exists:
    -   The bot draws one card
    -   If the drawn card is playable, it is played immediately
    -   Otherwise, the bot ends its turn

----------

### Design Constraints

The bot **must be implemented on the server side**, because:

-   The server enforces game rules
-   The server controls state transitions
-   The server handles broadcasting to clients

----------

### Required Components

#### Server

-   `BotPlayer`
-   `SimpleBotStrategy`
-   In `GameRoom` turn logic:
    -   Detect whether the current player is a bot

----------

#### Game Layer

-   Reuse `RuleEngine`
-   The bot must **not** have its own special play logic
-   All actions must go through the same pipeline:
    -   `playCard(...)`

----------

#### Concurrency

-   Add a small delay before bot actions (300–800 ms) to simulate human behavior
-   Can use:
    -   `ScheduledExecutorService`, or
    -   a simple `Thread.sleep(...)`

----------

### Suggested Implementation

Card  chooseFirstPlayableCard(List<Card> hand, GameState  state);

----------

### Acceptance Criteria

-   A human player can start a game with a bot
-   The bot acts automatically during its turn
-   The bot does not perform illegal actions
-   Both bot and human actions go through the same server-side validation pipeline

----------


