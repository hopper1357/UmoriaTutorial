# How to Create a Roguelike Game in C++

This tutorial will guide you through the process of creating a simple roguelike game in C++, inspired by the classic game Umoria. We will cover the basic components of a roguelike game, including the game loop, player character, dungeon, and monsters.

This tutorial is based on the [Umoria](https://github.com/dungeons-of-moria/umoria) codebase. We will be analyzing its structure and implementing a simplified version of it.

## Getting Started

This section will guide you through setting up the development environment for our roguelike game.

### Prerequisites

Before we start, you need to have the following tools installed on your system:

*   **A C++ compiler:** GCC or Clang are good choices.
*   **CMake:** A tool for managing the build process.
*   **ncurses:** A library for creating text-based user interfaces.

You can install these on a Debian-based Linux distribution (like Ubuntu) with the following command:

```bash
sudo apt-get update && sudo apt-get install build-essential cmake libncurses5-dev
```

### Project Structure

Our project will have a simple structure:

```
.
├── CMakeLists.txt
└── src
    ├── main.cpp
    ├── game.h
    ├── game.cpp
    ├── player.h
    └── player.cpp
```

*   `CMakeLists.txt`: This file will contain the build instructions for CMake.
*   `src/`: This directory will contain our source code.

### Setting up the Build System

Now, let's create the `CMakeLists.txt` file. This file tells CMake how to build our project.

```cmake
cmake_minimum_required(VERSION 3.10)
project(Roguelike)

# Find the ncurses library
find_package(Curses REQUIRED)

# Add the source files
add_executable(roguelike src/main.cpp src/game.cpp src/player.cpp src/dungeon.cpp src/monster.cpp src/item.cpp src/messagelog.cpp)

# Link the ncurses library
target_link_libraries(roguelike ${CURSES_LIBRARIES})

# Set the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

This `CMakeLists.txt` file does the following:
1.  Sets the minimum required version of CMake and defines our project name.
2.  Finds the `ncurses` library, which is required for our project.
3.  Creates an executable named `roguelike` from our source files.
4.  Links the `ncurses` library to our executable.
5.  Sets the C++ standard to C++17.

## The Core Game Loop

The heart of our roguelike game is the game loop. This loop is responsible for initializing the game, processing user input, updating the game state, and rendering the game world.

### The `main` function

Our `main` function will be the entry point of our game. It will be responsible for parsing command-line arguments and starting the game. Here is a simplified `main.cpp`:

```cpp
// src/main.cpp
#include "game.h"

int main(int argc, char *argv[]) {
    // Initialize the game
    Game game;

    // Start the game loop
    game.loop();

    return 0;
}
```

### The `Game` class

The `Game` class will manage the game's state and contain the main game loop. Let's start by defining the `Game` class in `src/game.h`:

```cpp
// src/game.h
#pragma once

class Game {
public:
    Game();
    void loop();

private:
    bool is_running;
};
```

And here's the implementation in `src/game.cpp`:

```cpp
// src/game.cpp
#include "game.h"
#include <ncurses.h>

Game::Game() : is_running(true) {
    // Initialize ncurses
    initscr();
    cbreak();
    noecho();
    keypad(stdscr, TRUE);
}

void Game::loop() {
    while (is_running) {
        // Get user input
        int ch = getch();

        // Process input
        switch (ch) {
            case 'q':
                is_running = false;
                break;
        }

        // Update game state

        // Render the game
        clear();
        mvprintw(0, 0, "Hello, Roguelike!");
        refresh();
    }

    // Clean up ncurses
    endwin();
}
```

This simple game loop does the following:
1.  **Initializes ncurses:** `initscr()` initializes the screen, `cbreak()` disables line buffering, `noecho()` prevents user input from being displayed, and `keypad(stdscr, TRUE)` enables function keys.
2.  **Loops:** The `while (is_running)` loop continues as long as the game is running.
3.  **Gets user input:** `getch()` waits for the user to press a key.
4.  **Processes input:** A `switch` statement handles the user's input. In this case, pressing 'q' will exit the game.
5.  **Renders the game:** `clear()` clears the screen, `mvprintw()` prints text at a specific location, and `refresh()` updates the screen.
6.  **Cleans up:** `endwin()` restores the terminal to its normal state.

## The Player Character

Now that we have a basic game loop, let's add a player character to our game. The player will be represented by a `@` symbol and will be able to move around the screen.

### The `Player` class

First, let's define a `Player` class in `src/player.h`:

```cpp
// src/player.h
#pragma once

class Player {
public:
    Player(int x, int y);

    void move(int dx, int dy);
    void draw();

private:
    int x, y;
};
```

And the implementation in `src/player.cpp`:

```cpp
// src/player.cpp
#include "player.h"
#include <ncurses.h>

Player::Player(int x, int y) : x(x), y(y) {}

void Player::move(int dx, int dy) {
    x += dx;
    y += dy;
}

void Player::draw() {
    mvaddch(y, x, '@');
}
```

This `Player` class has a constructor to set the initial position, a `move` method to update the position, and a `draw` method to draw the player on the screen.

### Integrating the Player into the Game

Now, let's add the player to our `Game` class. First, include the `player.h` header in `src/game.h` and add a `Player` member variable:

```cpp
// src/game.h
#pragma once
#include "player.h"

class Game {
public:
    Game();
    void loop();

private:
    bool is_running;
    Player player;
};
```

Next, update the `Game` constructor in `src/game.cpp` to initialize the player at the center of the screen:

```cpp
// src/game.cpp
#include "game.h"
#include <ncurses.h>

Game::Game() : is_running(true), player(10, 10) { // Example starting position
    // ... (ncurses initialization)
}
```

Finally, let's update the game loop to handle player movement and draw the player:

```cpp
// src/game.cpp
void Game::loop() {
    while (is_running) {
        // Get user input
        int ch = getch();

        // Process input
        switch (ch) {
            case 'q':
                is_running = false;
                break;
            case KEY_UP:
                player.move(0, -1);
                break;
            case KEY_DOWN:
                player.move(0, 1);
                break;
            case KEY_LEFT:
                player.move(-1, 0);
                break;
            case KEY_RIGHT:
                player.move(1, 0);
                break;
        }

        // Update game state
        // (The player's state is updated in the input processing)

        // Render the game
        clear();
        player.draw();
        refresh();
    }

    // Clean up ncurses
    endwin();
}
```

Now, when you run the game, you should see a `@` symbol that you can move around the screen with the arrow keys.

## The Dungeon

A roguelike game wouldn't be complete without a dungeon to explore. In this section, we'll create a simple dungeon for our player to move around in.

### Representing the Dungeon

We'll represent our dungeon as a 2D grid of tiles. Each tile can be either a wall or a floor. Let's start by defining a `Tile` enum and a `Dungeon` class in a new `src/dungeon.h` file:

```cpp
// src/dungeon.h
#pragma once
#include <vector>

enum class Tile {
    Wall,
    Floor
};

class Dungeon {
public:
    Dungeon(int width, int height);

    void draw();
    bool is_wall(int x, int y);

private:
    int width, height;
    std::vector<std::vector<Tile>> map;
};
```

And the implementation in `src/dungeon.cpp`:

```cpp
// src/dungeon.cpp
#include "dungeon.h"
#include <ncurses.h>

Dungeon::Dungeon(int width, int height) : width(width), height(height) {
    // Initialize the map with walls
    map.resize(height, std::vector<Tile>(width, Tile::Wall));

    // Create a simple room
    for (int y = 5; y < 15; ++y) {
        for (int x = 10; x < 30; ++x) {
            map[y][x] = Tile::Floor;
        }
    }
}

void Dungeon::draw() {
    for (int y = 0; y < height; ++y) {
        for (int x = 0; x < width; ++x) {
            if (map[y][x] == Tile::Wall) {
                mvaddch(y, x, '#');
            } else {
                mvaddch(y, x, '.');
            }
        }
    }
}

bool Dungeon::is_wall(int x, int y) {
    return map[y][x] == Tile::Wall;
}
```

This `Dungeon` class creates a simple room surrounded by walls. The `is_wall` method will be used to check for collisions.

### Integrating the Dungeon into the Game

Now, let's add the dungeon to our `Game` class. First, include `dungeon.h` in `src/game.h` and add a `Dungeon` member:

```cpp
// src/game.h
#pragma once
#include "player.h"
#include "dungeon.h"

class Game {
public:
    Game();
    void loop();

private:
    bool is_running;
    Player player;
    Dungeon dungeon;
};
```

Update the `Game` constructor to initialize the dungeon:

```cpp
// src/game.cpp
Game::Game() : is_running(true), player(10, 10), dungeon(40, 20) {
    // ...
}
```

Now, update the game loop to draw the dungeon and check for collisions before moving the player. We'll need to modify the `Player` class to get the player's next position.

In `src/player.h`, add `get_x()` and `get_y()` methods:
```cpp
// src/player.h
// ...
    int get_x() const { return x; }
    int get_y() const { return y; }
// ...
```

Then, in the `Game::loop()` in `src/game.cpp`:
```cpp
// src/game.cpp
void Game::loop() {
    while (is_running) {
        int ch = getch();

        int dx = 0, dy = 0;
        switch (ch) {
            case 'q': is_running = false; break;
            case KEY_UP: dy = -1; break;
            case KEY_DOWN: dy = 1; break;
            case KEY_LEFT: dx = -1; break;
            case KEY_RIGHT: dx = 1; break;
        }

        if (!dungeon.is_wall(player.get_x() + dx, player.get_y() + dy)) {
            player.move(dx, dy);
        }

        clear();
        dungeon.draw();
        player.draw();
        refresh();
    }
    // ...
}
```

Now you have a simple dungeon with a room, and the player can move around inside it, but not through the walls.

## Monsters

What's a dungeon without monsters? In this section, we'll add some simple monsters to our game that will move around randomly.

### The `Monster` class

Let's define a `Monster` class in a new `src/monster.h` file:

```cpp
// src/monster.h
#pragma once

class Monster {
public:
    Monster(int x, int y, char symbol);

    void move_randomly();
    void draw();

private:
    int x, y;
    char symbol;
};
```

And the implementation in `src/monster.cpp`:

```cpp
// src/monster.cpp
#include "monster.h"
#include <ncurses.h>
#include <cstdlib>

Monster::Monster(int x, int y, char symbol) : x(x), y(y), symbol(symbol) {}

void Monster::move_randomly() {
    int dx = std::rand() % 3 - 1; // -1, 0, or 1
    int dy = std::rand() % 3 - 1;
    x += dx;
    y += dy;
}

void Monster::draw() {
    mvaddch(y, x, symbol);
}
```

This `Monster` class is similar to our `Player` class, but it has a `move_randomly` method for its AI.

### Integrating Monsters into the Game

Now, let's add monsters to our `Game` class. We'll store them in a `std::vector`.

First, include `monster.h` and `<vector>` in `src/game.h`:

```cpp
// src/game.h
#pragma once
#include "player.h"
#include "dungeon.h"
#include "monster.h"
#include <vector>

class Game {
public:
    Game();
    void loop();

private:
    bool is_running;
    Player player;
    Dungeon dungeon;
    std::vector<Monster> monsters;
};
```

In the `Game` constructor in `src/game.cpp`, let's spawn a few monsters:

```cpp
// src/game.cpp
Game::Game() : is_running(true), player(10, 10), dungeon(40, 20) {
    // ...
    monsters.emplace_back(15, 12, 'o'); // an orc
    monsters.emplace_back(20, 8, 'g');  // a goblin
}
```

Finally, let's update the game loop to move and draw the monsters. We also need to add collision detection for monsters.

In `src/dungeon.h`, add a forward declaration for `Monster`:
```cpp
// src/dungeon.h
class Monster;
```

In `src/dungeon.cpp`, check for monster positions as well:
```cpp
// src/dungeon.cpp
bool Dungeon::is_wall(int x, int y) {
    // Also need to check for monster collision in a real game
    return map[y][x] == Tile::Wall;
}
```

In `src/game.cpp` `Game::loop()`:

```cpp
// src/game.cpp
void Game::loop() {
    while (is_running) {
        // ... (player movement)

        // Update game state
        for (auto& monster : monsters) {
            monster.move_randomly();
            // Add collision detection for monsters here
        }

        // Render the game
        clear();
        dungeon.draw();
        player.draw();
        for (auto& monster : monsters) {
            monster.draw();
        }
        refresh();
    }
    // ...
}
```
With these changes, you'll see monsters moving randomly around the dungeon. This is a basic foundation that you can build upon with more complex AI and combat mechanics.

## A Basic Combat System

Now that we have a player and monsters, let's make them fight! We'll implement a simple, turn-based combat system.

### Health and Attacks

First, let's add health points (HP) and an attack method to both our `Player` and `Monster` classes.

In `src/player.h`:
```cpp
// src/player.h
class Player {
public:
    // ...
    void attack(class Monster& target);
    int get_hp() const { return hp; }
    void take_damage(int damage);
private:
    int hp = 100;
    int attack_power = 10;
    // ...
};
```

In `src/monster.h`:
```cpp
// src/monster.h
class Monster {
public:
    // ...
    void attack(class Player& target);
    int get_hp() const { return hp; }
    void take_damage(int damage);
private:
    int hp = 20;
    int attack_power = 5;
    // ...
};
```
*Note: We use a forward declaration `class Monster&` and `class Player&` to avoid circular dependencies between the header files.*

Now, let's implement these methods.

In `src/player.cpp`:
```cpp
// src/player.cpp
#include "monster.h" // Include monster.h here

// ...
void Player::attack(Monster& target) {
    target.take_damage(attack_power);
}

void Player::take_damage(int damage) {
    hp -= damage;
}
```

In `src/monster.cpp`:
```cpp
// src/monster.cpp
#include "player.h" // Include player.h here

// ...
void Monster::attack(Player& target) {
    target.take_damage(attack_power);
}

void Monster::take_damage(int damage) {
    hp -= damage;
}
```

### Turn-based Gameplay

We need to modify our game loop to be turn-based. The player takes an action, and then each monster takes an action.

In `src/game.cpp`, we'll change the game loop to only process one key press at a time and then let the monsters have their turn.

```cpp
// src/game.cpp

// In Game::loop()
void Game::loop() {
    while (is_running) {
        // Render the game state before waiting for input
        clear();
        dungeon.draw();
        player.draw();
        for (auto& monster : monsters) {
            if (monster.get_hp() > 0) {
                monster.draw();
            }
        }
        refresh();

        // Get user input
        int ch = getch();

        // Player's turn
        bool player_turn_ended = false;
        switch (ch) {
            case 'q': is_running = false; break;
            case KEY_UP:
                // Check for monster to attack
                // ...
                player.move(0, -1);
                player_turn_ended = true;
                break;
            // ... other movement keys
        }

        // Monsters' turn
        if (player_turn_ended) {
            for (auto& monster : monsters) {
                if (monster.get_hp() > 0) {
                    // Simple AI: if next to player, attack, otherwise move randomly.
                    // This logic needs to be implemented.
                    monster.move_randomly();
                }
            }
        }

        // Remove dead monsters
        monsters.erase(std::remove_if(monsters.begin(), monsters.end(),
            [](const Monster& m) {
                return m.get_hp() <= 0;
            }), monsters.end());

        if (player.get_hp() <= 0) {
            is_running = false;
            // Game over message
        }
    }
    // ...
}
```
This is a more structured game loop. To make attacking work, you'd have to check if the player's move would land on a monster, and if so, call `player.attack()` instead of `player.move()`. Similarly, monsters would check if they are adjacent to the player before attacking.

This is a starting point for a combat system. You can expand it by adding more complex AI, different attack types, and a more robust turn-handling system.

## Items and Inventory

A roguelike is more fun with loot! Let's add items that the player can pick up and use.

### The `Item` class

First, let's define a simple `Item` class in a new `src/item.h` file.

```cpp
// src/item.h
#pragma once
#include <string>

class Item {
public:
    Item(std::string name, char symbol);

    void draw(int x, int y);
    const std::string& get_name() const;

private:
    std::string name;
    char symbol;
};
```

And its implementation in `src/item.cpp`:
```cpp
// src/item.cpp
#include "item.h"
#include <ncurses.h>

Item::Item(std::string name, char symbol) : name(name), symbol(symbol) {}

void Item::draw(int x, int y) {
    mvaddch(y, x, symbol);
}

const std::string& Item::get_name() const {
    return name;
}
```

### Items in the Dungeon

We need a way to place items on the dungeon floor. We'll modify the `Dungeon` class to hold a collection of items.

In `src/dungeon.h`, we'll store items and their locations.
```cpp
// src/dungeon.h
#include <vector>
#include <map>
#include "item.h"

// ... in Dungeon class
private:
    std::map<std::pair<int, int>, Item> items;
```
We can then add a function to place items in the dungeon and modify the `draw` function to show them.

### Player Inventory

Let's give our player an inventory.

In `src/player.h`:
```cpp
// src/player.h
#include <vector>
#include "item.h"

class Player {
    // ...
private:
    std::vector<Item> inventory;
};
```

### Picking up Items

To pick up an item, the player needs a "get" action. This would be triggered by a key press in the game loop. The logic would be:
1. Check if there's an item at the player's current location.
2. If so, add it to the player's inventory.
3. Remove the item from the dungeon floor.

Here's a simplified version of what you could add to `Game::loop()` in `src/game.cpp`:
```cpp
// in Game::loop()
case 'g': // 'g' for get
    // Logic to find an item at player's (x, y) in the dungeon's item list
    // If an item is found:
    //   player.add_to_inventory(found_item);
    //   dungeon.remove_item(player_x, player_y);
    player_turn_ended = true;
    break;
```

This section provides a conceptual overview. A full implementation would require more detailed handling of data structures and game state, but this is a good starting point for adding a rich item and inventory system to your game.

## UI Enhancements

A good user interface is crucial for a roguelike. Let's add a status bar to show player information and a message log to display game events.

### The Status Bar

The status bar will display the player's current HP. We can dedicate the top line of the screen for this.

Let's modify our `Game::loop()` to include a call to a new `draw_status_bar()` method.

In `src/game.h`:
```cpp
// src/game.h
class Game {
    // ...
private:
    void draw_status_bar();
    // ...
};
```

In `src/game.cpp`:
```cpp
// src/game.cpp
void Game::draw_status_bar() {
    mvprintw(0, 0, "HP: %d / 100", player.get_hp());
}

void Game::loop() {
    while (is_running) {
        // ...
        clear();
        draw_status_bar();
        dungeon.draw();
        // ...
        refresh();
    }
    // ...
}
```
Now, the player's HP will be displayed at the top of the screen and will update whenever the screen is refreshed.

### The Message Log

The message log will display messages about what's happening in the game, like "You hit the orc." or "The goblin misses you." We can display the last few messages at the bottom of the screen.

Let's create a simple `MessageLog` class.

In a new `src/messagelog.h`:
```cpp
// src/messagelog.h
#pragma once
#include <vector>
#include <string>

class MessageLog {
public:
    void add(const std::string& message);
    void draw();
private:
    std::vector<std::string> messages;
};
```
And `src/messagelog.cpp`:
```cpp
// src/messagelog.cpp
#include "messagelog.h"
#include <ncurses.h>

void MessageLog::add(const std::string& message) {
    messages.push_back(message);
    if (messages.size() > 5) { // Keep only the last 5 messages
        messages.erase(messages.begin());
    }
}

void MessageLog::draw() {
    int y = LINES - messages.size();
    for (const auto& msg : messages) {
        mvprintw(y++, 0, msg.c_str());
    }
}
```
Now, integrate this into the `Game` class. Add a `MessageLog` member to `Game` and call its `draw()` method in the game loop. You can add messages to the log from your combat functions.

For example, in `Player::attack`:
```cpp
// src/player.cpp
void Player::attack(Monster& target) {
    target.take_damage(attack_power);
    // game.message_log.add("You attack the monster!");
}
```
*Note: You would need to pass a reference to the `Game` object or the `MessageLog` to the `attack` method to make this work.*

With these UI enhancements, the game becomes much more informative and engaging for the player.

## Putting It All Together

Now that we have all the basic components of our roguelike game, let's put it all together and compile it.

### Compiling and Running the Game

To compile the game, you'll need to create the necessary source files (`main.cpp`, `game.h`, `game.cpp`, `player.h`, `player.cpp`, `dungeon.h`, `dungeon.cpp`, `monster.h`, `monster.cpp`) with the content we've discussed.

Then, you can use CMake to build the project. From the root of your project directory, run the following commands:

```bash
mkdir build
cd build
cmake ..
make
```

This will create an executable named `roguelike` in the `build` directory. You can run it with:

```bash
./roguelike
```

### Next Steps

This tutorial has covered the very basics of creating a roguelike game. Here are some ideas for how you can expand on it:

*   **Combat:** Implement a combat system so the player can fight monsters.
*   **Items:** Add items like potions, scrolls, and weapons.
*   **Dungeon Generation:** Create more complex and interesting dungeon layouts using procedural generation algorithms.
*   **More Monsters:** Add different types of monsters with unique behaviors.
*   **User Interface:** Improve the UI with a status bar, message log, and inventory screen.

By studying the Umoria codebase, you can find inspiration and examples for how to implement these features and many more. Good luck, and have fun creating your own roguelike adventure!
