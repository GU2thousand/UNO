## Required Card Types to Support

-   Skip
-   Reverse
-   Draw Two
-   Wild
-   Wild Draw Four

----------

## Rule Edge Cases

-   In a **2-player game**, **Reverse** does not simply change direction  
    → Instead, the same player immediately gets another turn
-   **Mixed stacking is allowed**  
    → Example: `+2` can be stacked with **Wild Draw Four**

----------

## Definition of Completion (Feature Scope)

### Skip

-   The next player is skipped
-   Turn advances by more than one position

### Reverse

-   In multiplayer: changes direction of play
-   In 2-player: current player takes another turn immediately

### Draw Two

-   The next player draws 2 cards

### Wild

-   The player must choose a color when played
-   The current active color is updated accordingly

### Wild Draw Four

-   The player must choose a color
-   The next player draws 4 cards

----------

## Required Components for This Phase

### Game Layer

-   Extend `CardType`
-   Implement:
    -   `applyCardEffect(...)`
-   Maintain:
    -   `direction`
    -   `pendingDrawCount`
    -   `currentColor`

----------

### Client

-   When playing **Wild / Wild Draw Four**:
    -   Show a color selection dialog
    -   Use:
        
        JOptionPane.showOptionDialog(...)
        

----------

### Server

The server must persist and control:

-   Current direction
-   Pending draw count
-   Current overridden color

----------

## Required Classes / Fields

### `GameState`

CardColor  currentColor;  
int  direction;  
int  pendingDraw;  
boolean  gameStarted;

### `RuleEngine`

resolveActionEffect(...)

----------

## Acceptance Criteria

At minimum, the following scenarios must work correctly:

-   Skip correctly skips the next player
-   Reverse correctly changes direction
-   Reverse in a 2-player game results in an immediate extra turn
-   Draw Two and Wild Draw Four function correctly (including stacking)
-   Wild correctly updates the current color after being played
