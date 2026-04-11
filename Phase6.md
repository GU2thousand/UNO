## Phase 6: Database

### Objective

Add **history tracking** and **statistics** to the project.

Only the following need to be implemented:

-   player profiles
-   completed match history
-   summary statistics

----------

### Definition of Completion

-   A match record is written when a game ends
-   Player profiles can be saved
-   The system can query:
    -   total games played
    -   total wins
    -   win rate
-   Optionally, the UI can display recent match results

----------

### Required Technologies for This Phase

#### Persistence

-   JDBC
-   SQLite

----------

### Suggested Table Schema

#### `players`

id INTEGER  PRIMARY  KEY AUTOINCREMENT  
username TEXT UNIQUE  NOT  NULL  
games_played INTEGER  DEFAULT  0  
wins INTEGER  DEFAULT  0

#### `matches`

id INTEGER  PRIMARY  KEY AUTOINCREMENT  
winner_name TEXT  
played_at DATETIME DEFAULT  CURRENT_TIMESTAMP  
player_count INTEGER

#### `match_players`

id INTEGER  PRIMARY  KEY AUTOINCREMENT  
match_id INTEGER  
username TEXT  
result TEXT

----------

### Required Java Classes

-   `DatabaseManager`
-   `PlayerRepository`
-   `MatchRepository`

----------

### Key Methods

createPlayerIfNotExists(username)  
recordMatchResult(...)  
getPlayerStats(username)

----------

### When Database Operations Should Be Triggered

-   When a player connects or registers
-   When a match ends

Do **not** write to the database after every card play.

----------

### Acceptance Criteria

-   After a match finishes, a corresponding record exists in the database
-   After restarting the program, match history and statistics are still available
-   The GUI can display a player’s win rate or a summary of recent match history
