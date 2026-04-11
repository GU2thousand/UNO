

## Bot Behavior

-   When the bot is affected by a **Wild Draw Four (+4)**, it will randomly choose,-   50% challenge 50% accept

----------

## Custom Cards Overview

Both custom cards share a key constraint:

> They can **only be triggered when the player has no legal playable card and is about to enter the draw phase**.

This means they are **conditional action cards**, not standard playable cards.

----------

## Custom Card 1: Group Draw

### Trigger Condition

-   The current player has **no legal playable card**

### Effect

-   The current player may play **Group Draw**
-   All **other players** each draw **1 card**
-   The **current player also draws 1 card**
-   Then proceed with the **normal draw rule**:
    -   If the drawn card is playable → may play it immediately
    -   Otherwise → end turn

----------

## Custom Card 2: Triple Peek

### Trigger Condition

-   The current player has **no legal playable card**

### Effect

-   The player looks at the **top 3 cards of the draw pile**
-   Selects **1 card** to add to their hand
-   If the selected card is playable → may play it immediately
-   The remaining **2 cards are placed at the bottom of the draw pile**
-   If the draw pile has fewer than 3 cards:
    -   **Reshuffle discard pile into draw pile first**

----------

## Completion Criteria

### Group Draw

-   Can only be used when the player has **no legal moves**
-   Cannot be played as a normal action card
-   Must correctly:
    -   Increase all other players’ hand size by 1
    -   Increase current player’s hand size by 1
    -   Continue with standard draw-phase logic

----------

### Triple Peek

-   Can only be used when the player has **no legal moves**
-   Only the **current player** can see the 3 cards
-   Other clients must **not see any information** about these cards
-   Remaining cards must be correctly returned to the **bottom of the deck**
-   Must handle **deck exhaustion + reshuffle correctly**

----------

## Privacy Requirement

> **Private information must remain private.**

Especially for **Triple Peek**:

-   Only the acting player receives the peek results
-   No leakage to other clients

----------

## Required Implementation Changes

### Game Layer

-   Add:
    -   `CardType.GROUP_DRAW`
    -   `CardType.TRIPLE_PEEK`
-   In `RuleEngine`:
    -   Add **precondition checks** (only usable when no legal moves)
-   Deck must support:
    -   Peek top N cards
    -   Select one card
    -   Return remaining cards to bottom
    -   Reshuffle discard pile into draw pile when needed

----------

### Server

-   Must support **private messaging**
-   For Triple Peek:
    -   Send card options **only to current player**
    -   Do NOT broadcast peek results

----------

### Client

-   For Triple Peek:
    -   Show a **selection dialog/UI** to the current player
    -   Allow choosing 1 card
    -   Send choice back to server

----------

## Message Protocol Additions

-   `TRIPLE_PEEK_OPTIONS`  
    → Server → Current player (private)
-   `TRIPLE_PEEK_CHOICE`  
    → Client → Server
-   `PRIVATE_UPDATE`  
    → Server → Specific client only

----------

## Acceptance Criteria

-   Group Draw:
    -   Only usable under correct condition
    -   Applies effect to all players correctly
-   Triple Peek:
    -   Only current player sees the 3 cards
    -   Other players see nothing about them
    -   Deck reshuffles correctly when insufficient cards

----------



Add this as a UI/UX requirement. Keep it crisp and testable:

----------

## UI Update: Playable Card Highlight

### Requirement

-   When indicating **playable cards in the current player’s hand**, the highlight color **must NOT be yellow**.

### Reason

-   Yellow cards visually blend with a yellow highlight, making the cue ambiguous.

### Expected Behavior

-   Use a **non-conflicting highlight color** (e.g., blue, green outline, or white glow).
-   The highlight must remain **clearly distinguishable across all card colors**:
    -   Red
    -   Blue
    -   Green
    -   Yellow

### Implementation Notes

-   Replace current highlight color (yellow) with a **high-contrast color**.
-   Prefer:
    -   Border highlight (e.g., 3–4px stroke), or
    -   Subtle glow/shadow instead of fill overlay
-   Ensure consistency across:
    -   Hover state
    -   Playable state
    -   Selected state (if applicable)

### Acceptance Criteria

-   Yellow cards are still clearly identifiable as playable
-   No color combination causes ambiguity
-   Highlight remains visible under different UI backgrounds

----------

If you want the blunt version: using yellow as a highlight in a game that already has yellow as a primary card color is a design mistake, not just a tweak. Fixing it now saves you from a pile of UI bugs later.


### Requirement

-   When indicating **playable cards in the current player’s hand**, the highlight color **must NOT be yellow**.

### Reason

-   Yellow cards visually blend with a yellow highlight, making the cue ambiguous.

### Expected Behavior

-   Use a **non-conflicting highlight color** (e.g., blue, green outline, or white glow).
-   The highlight must remain **clearly distinguishable across all card colors**:
    -   Red
    -   Blue
    -   Green
    -   Yellow

### Implementation Notes

-   Replace current highlight color (yellow) with a **high-contrast color**.
-   Prefer:
    -   Border highlight (e.g., 3–4px stroke), or
    -   Subtle glow/shadow instead of fill overlay
-   Ensure consistency across:
    -   Hover state
    -   Playable state
    -   Selected state (if applicable)

### Acceptance Criteria

-   Yellow cards are still clearly identifiable as playable
-   No color combination causes ambiguity
-   Highlight remains visible under different UI backgrounds

----------
