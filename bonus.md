# Bonus Features Tutorial

Welcome to the bonus features tutorial! In this guide, we'll explore how to add several advanced and interesting mechanics to our roguelike game, building upon the concepts from the beginner, intermediate, and advanced tutorials.

## 1. Weight-Based Inventory System

Let's make our inventory system more realistic and challenging by adding item weights and a carrying capacity for the player.

### Item Weight

First, we need to give our items a `weight` property. We can add this to our `Item` class.

In `src/item.h`:
```cpp
// src/item.h
class Item {
public:
    Item(std::string name, char symbol, int weight);
    int get_weight() const;
    // ...
private:
    int weight;
    // ...
};
```
And in `src/item.cpp`:
```cpp
// src/item.cpp
Item::Item(std::string name, char symbol, int weight) : name(name), symbol(symbol), weight(weight) {}

int Item::get_weight() const {
    return weight;
}
```

Now, when you create items, you can give them a weight, for example: `Item("Health Potion", '!', 1)`.

### Player Carrying Capacity

The player's ability to carry items should depend on their strength. Let's add this to the `Player` class.

In `src/player.h`:
```cpp
// src/player.h
class Player {
public:
    // ...
    int get_strength() const;
    int get_current_weight() const;
    int get_max_weight() const;
    bool can_carry(const Item& item) const;
    void add_to_inventory(const Item& item);
private:
    int strength = 10;
    int current_weight = 0;
    // ...
};
```
The `get_max_weight()` could simply be `strength * 10`. The `can_carry()` method would check if adding a new item's weight would exceed the `max_weight`.

The `add_to_inventory()` method would then look like this:
```cpp
// src/player.cpp
void Player::add_to_inventory(const Item& item) {
    if (can_carry(item)) {
        inventory.push_back(item);
        current_weight += item.get_weight();
    } else {
        // game.message_log.add("You can't carry any more!");
    }
}
```
When picking up items, you would now use this new method, which prevents the player from becoming over-encumbered. This adds a new layer of strategy, as the player must decide which items are valuable enough to carry.

## 2. Potion-Mixing Crafting System

Let's add a simple crafting system to our game. We'll allow the player to mix two potions to create a new one.

### Potion Recipes

We need a way to define the outcomes of mixing different potions. A `std::map` is a good way to represent recipes.

```cpp
// In your Game class or a dedicated CraftingSystem class
std::map<std::pair<std::string, std::string>, std::string> potion_recipes;

// Example recipes
potion_recipes[{"Health Potion", "Mana Potion"}] = "Rejuvenation Potion";
potion_recipes[{"Mana Potion", "Health Potion"}] = "Rejuvenation Potion"; // Order shouldn't matter
```
You would initialize these recipes when the game starts.

### The "Mix" Action

The player needs a "mix" command. This would typically:
1.  Prompt the player to select the first potion from their inventory.
2.  Prompt for the second potion.
3.  Check if a recipe exists for the selected pair.

Here's a conceptual look at the logic in your `Game::loop()`:
```cpp
// in Game::loop()
case 'm': // 'm' for mix
    // 1. Show inventory and get first potion choice
    // 2. Show inventory and get second potion choice
    // 3. Check for a recipe
    // if (potion_recipes.count({potion1.get_name(), potion2.get_name()})) {
    //     std::string new_potion_name = potion_recipes.at({potion1.get_name(), potion2.get_name()});
    //     // 4. Remove the two old potions from inventory
    //     // 5. Add the new potion to inventory
    //     // game.message_log.add("You created a " + new_potion_name + "!");
    // } else {
    //     // game.message_log.add("Nothing happens.");
    // }
    player_turn_ended = true;
    break;
```

This system adds a fun, experimental layer to the game. Players can discover new recipes, and you can create powerful, unique potions that can only be obtained through crafting.

## 3. Elemental Weapons

Let's add more depth to our combat system with elemental weapons and monster resistances.

### Defining Elements and Resistances

First, we need an `enum` for our damage types.
```cpp
// In a new file, e.g., src/types.h
enum class Element {
    Physical,
    Fire,
    Frost
};
```

Now, let's give our weapons an elemental property. We can add this to the `Item` class.
```cpp
// src/item.h
#include "types.h"

class Item {
    // ...
private:
    Element damage_type = Element::Physical;
    // ...
};
```
You can then create special weapons like a `new Item("Flaming Sword", '/', 5, Element::Fire)`.

Next, monsters need resistances. We can add a map of resistances to our `Monster` class.
```cpp
// src/monster.h
#include <map>
#include "types.h"

class Monster {
    // ...
private:
    std::map<Element, float> resistances; // float is a multiplier, e.g., 0.5 for 50% resistance
    // ...
};
```
In the `Monster` constructor, you could set up resistances, for example, a "Frost Wolf" might have `resistances[Element::Frost] = 0.5;` and `resistances[Element::Fire] = 2.0;` (a weakness).

### Updating Combat Calculations

Now, we need to update our `take_damage` methods to account for these new properties. The `Player::attack` method would need to pass the element type of the weapon.

The `Monster::take_damage` method would look like this:
```cpp
// src/monster.cpp
void Monster::take_damage(int damage, Element damage_type) {
    float multiplier = 1.0;
    if (resistances.count(damage_type)) {
        multiplier = resistances.at(damage_type);
    }
    hp -= static_cast<int>(damage * multiplier);
}
```

This system makes combat more tactical. Players need to pay attention to a monster's resistances and choose the right weapon for the job. It also opens the door for a wider variety of monsters and magical items.

## 4. Hunger System

A hunger system adds a survival element to the game, forcing the player to manage their resources carefully.

### The Hunger Clock

We'll add a `food` counter to our `Player` class. This counter will decrease with every turn the player takes.

In `src/player.h`:
```cpp
// src/player.h
class Player {
    // ...
public:
    void update_food(int amount);
    int get_food() const;
private:
    int food = 5000; // Start with a full stomach
    // ...
};
```
And in `src/player.cpp`:
```cpp
// src/player.cpp
void Player::update_food(int amount) {
    food += amount;
}
int Player::get_food() const {
    return food;
}
```

### Game Loop Integration

In our `Game::loop()`, after every player turn, we'll decrease the food counter.
```cpp
// in Game::loop()
        // ...
        // Monsters' turn
        if (player_turn_ended) {
            player.update_food(-1); // Decrease food every turn
            // ... (monster AI)
        }
```

You can then check the player's food level and apply different status effects.
```cpp
// in Game::loop()
// ...
// Check for hunger status
if (player.get_food() < 1000) {
    // game.message_log.add("You are hungry.");
}
if (player.get_food() < 200) {
    // game.message_log.add("You are weak with hunger.");
    // Apply penalties, like reduced damage or speed.
}
```

### Eating

To replenish hunger, the player needs to eat. This requires having "food" items and an "eat" command.

When the player uses the "eat" command on a food item:
1.  The item is removed from their inventory.
2.  The player's `food` counter is increased by the nutritional value of the food item.
3.  A message is displayed to the player.

This system encourages players to keep moving and exploring, as they can't afford to wait around forever.

## 5. Character Skills and Perks

A skill system allows players to customize their character's development, making leveling up more exciting.

### Skill Points and Perks

First, let's give the player `skill_points` that they gain when they level up.

In `src/player.h`:
```cpp
// src/player.h
class Player {
    // ...
public:
    void level_up();
private:
    int skill_points = 0;
    // ...
};
```
In the `level_up` method, you would increment `skill_points`.

Next, we need to define our perks. An `enum` for perk types and a `struct` for perk data is a good approach.
```cpp
// In a new file, e.g., src/perks.h
enum class PerkType {
    Toughness, // +10 max HP
    StrongArm, // +2 attack power
    Lockpicking
};

struct Perk {
    PerkType type;
    std::string name;
    std::string description;
};
```

### The Skill Screen

You'll need a new UI screen for spending skill points. This screen would:
1.  Be accessible via a key press (e.g., 's' for skills).
2.  Display the player's available skill points.
3.  List the available perks and their descriptions.
4.  Allow the player to spend a point to acquire a perk.

### Applying Perk Effects

When a player acquires a perk, you need to apply its effect. This can be done with a `switch` statement based on the `PerkType`.

```cpp
// In a method like Player::acquire_perk(PerkType perk_type)
void Player::acquire_perk(PerkType perk_type) {
    if (skill_points > 0) {
        skill_points--;
        switch (perk_type) {
            case PerkType::Toughness:
                max_hp += 10;
                hp += 10;
                break;
            case PerkType::StrongArm:
                attack_power += 2;
                break;
            case PerkType::Lockpicking:
                // Grant lockpicking ability
                break;
        }
    }
}
```

A skill system is a great way to add long-term replayability and strategic depth to your roguelike. Players can plan different "builds" for their characters, trying out new combinations of perks with each playthrough.
