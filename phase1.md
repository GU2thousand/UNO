**Phase 1: Minimum Viable Loop**

## Goal

Get this working:

Two clients can connect to the same server, enter the same room, start a game, and see on the UI:  
their own hand and the top card of the discard pile.

At this stage, do **not** touch complex rules, do **not** touch action cards, and do **not** use a database.

## Minimum required capabilities

At a minimum, the system should support:

-   The server can start and listen on a port
-   The client can enter a username and connect to the server
-   One player can create a room, and another player can join it
-   When the room reaches the required number of players, the game can start
-   The server deals the initial hand to each player
-   The client can display:
    -   the player’s own hand
    -   the top card of the discard pile
    -   whose turn it is
    -   the list of players in the room
-   At minimum, clients can manually refresh and see a consistent shared state, instead of each client showing a different version

## What will be used in this phase

### GUI

-   `JFrame`
-   `JPanel`
-   `JButton`
-   `JLabel`
-   `SwingUtilities.invokeLater`

### Networking

-   `Socket`
-   `ServerSocket`
-   `ObjectInputStream`
-   `ObjectOutputStream`

### Concurrency

-   On the server side, one `ClientHandler implements Runnable` per client
-   `ExecutorService` is optional, but recommended

### Data model

-   `Card`
-   `Deck`
-   `PlayerState`
-   `GameSnapshot`

### Message protocol

At minimum, you need these message types:

-   `CONNECT`
-   `CREATE_ROOM`
-   `JOIN_ROOM`
-   `ROOM_UPDATE`
-   `START_GAME`
-   `GAME_STATE`
-   `ERROR`

## What exactly needs to be implemented in this phase

### Server

-   `ServerMain`: listens on a port
-   `ClientHandler`: receives messages from clients
-   `RoomManager`: manages rooms
-   `GameRoom`: stores players and the game instance inside a room
-   Initial dealing logic

### Client

-   `LobbyFrame`: connect, create room, join room
-   `GameFrame`: display hand cards and table state
-   `CardView`: render cards

### Game

-   `Deck`: create and shuffle the deck
-   `Card`: card data structure
-   `GameState`: snapshot of the current table state

## Acceptance criteria for this phase

The demo should be able to do the following:

-   Open two clients
-   Client A creates a room
-   Client B joins
-   Click Start
-   Both clients can see their own hand
-   Both clients can see the same discard pile top card
