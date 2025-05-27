# API Plan for a Fantasy Text-Based Game (Kids' Fantasy RPG API)

## Project Overview

This API will manage the game state, characters, locations, interactions with NPCs (Non-Player Characters), and items. It will be the core of the game, providing data to the front-end (initially text-based, later HTML/graphical). The goal is to build a solid API foundation consistent with design patterns, with future expansions for a graphical interface and an animated character in mind.

---

## Why This Project is Great for Learning API Patterns

* **Rich Resources and Relationships:** An RPG requires many interconnected entities like players, locations, NPCs, items, and dialogues. This offers ample opportunity to apply resource relationship patterns.
* **Complex Operations and States:** The game will involve dynamic operations such as character movement, environmental interactions (conversations, item collection), and managing player inventory and overall game state (save/load).
* **API Versioning:** You can plan for `v1` to cover basic text-based functionalities and then `v2` for UI extensions (e.g., adding location coordinates, attributes for graphics).
* **Authorization and Authentication:** Each child can have their own character and unique game save, introducing the need for managing authentication and authorization.

---

## Proposed API Resources and Endpoints

Below are suggested RESTful resources and endpoints you can consider when designing your API.

### 1. Player

* `GET /api/v1/player/{player_id}`: Retrieves information about the player (current location, inventory, health, etc.).
* `POST /api/v1/players`: Creates a new player (initial location, basic inventory, name).
    * **Body:** `{"name": "Hero", "starting_location_id": "forest_clearing"}`
* `PATCH /api/v1/player/{player_id}`: Partially updates the player's state (e.g., after collecting an item, changing health).
    * **Body:** `{"current_location_id": "new_location_id"}` or `{"inventory_add_item_id": "sword_of_light"}`

### 2. Locations

* `GET /api/v1/location/{location_id}`: Retrieves details of a specific location (description, available exits, NPCs in the location, items to collect). This will be a key endpoint after each movement.
* `GET /api/v1/location/{location_id}/exits`: Retrieves information about possible exits from the location (e.g., `"north": {"target_location_id": "forest_path", "description": "A path to the north"}`).
* `GET /api/v1/location/{location_id}/npcs`: Lists NPCs present in a given location.
* `GET /api/v1/location/{location_id}/items`: Lists items available for collection in a given location.

### 3. Movement

* `POST /api/v1/player/{player_id}/move`: Changes the player's location.
    * **Body:** `{"direction": "north"}`
    * **Response:** Updated player object, new location with its description, NPCs, and items.
    * The API should verify if movement in the specified direction is possible from the player's current location.

### 4. Items

* `GET /api/v1/item/{item_id}`: Retrieves details of an item (name, description, weight, collectibility, value).
* `POST /api/v1/player/{player_id}/items/collect`: The player collects an item from the current location.
    * **Body:** `{"item_id": "magic_key"}`
    * The API should check if the item is in the player's location and if it's collectible.
* `POST /api/v1/player/{player_id}/items/drop`: The player drops an item in the current location.
    * **Body:** `{"item_id": "old_sword"}`
* `GET /api/v1/player/{player_id}/inventory`: Retrieves a list of items in the player's inventory.

### 5. Non-Player Characters (NPCs)

* `GET /api/v1/npc/{npc_id}`: Retrieves details of an NPC (name, description, role).
* `POST /api/v1/player/{player_id}/talk/{npc_id}`: Initiates or continues a dialogue with an NPC.
    * **Body:** May contain `{"choice": "option_A_id"}` if dialogues have branching paths.
    * **Response:** The next dialogue line from the NPC and/or new dialogue options for the player.
* `GET /api/v1/npc/{npc_id}/dialogues`: Retrieves the dialogue structure for a given NPC (can be used for visualization or preparing UI options).

### 6. Save/Load Game

* `POST /api/v1/game/save/{player_id}`: Saves the player's current game state (location, inventory, dialogue state, etc.).
* `GET /api/v1/game/load/{player_id}`: Loads the game state for a player.

---

## Recommended Technologies for Python Implementation

* **Framework:** **FastAPI**
    * **Pros:** Modern, very fast, asynchronous, based on Python type hints. Automatically generates OpenAPI documentation (Swagger UI/ReDoc), which is invaluable for API design and testing.
* **Database:** **SQLite** (for a start)
    * **Pros:** Lightweight, easy to configure (database in a single file), ideal for rapid prototyping and learning.
* **ORM (Object-Relational Mapper):** **SQLAlchemy**
    * **Pros:** Powerful and flexible ORM, framework-agnostic. It will allow you to map Python objects (e.g., `Player`, `Location`, `Item`) to database tables, abstracting SQL query complexity. Using **Alembic** for database migrations would be a good addition.
* **Data Validation:** **Pydantic** (integrated with FastAPI)
    * **Pros:** Enables easy definition of input and output data models (e.g., `PlayerCreate`, `ItemResponse`), ensuring data validation and serialization.

---

## Example Database Schema

* **`players`:** `id`, `name`, `current_location_id`, `health`, `experience`, `inventory_json` (or a separate `player_items` table).
* **`locations`:** `id`, `name`, `description`.
* **`location_exits`:** `location_id`, `direction` (e.g., 'north', 'south'), `target_location_id`.
* **`npcs`:** `id`, `name`, `description`, `location_id`.
* **`items`:** `id`, `name`, `description`, `is_collectible`.
* **`location_items`:** `location_id`, `item_id`, `quantity`.
* **`player_items`:** `player_id`, `item_id`, `quantity`.
* **`dialogue_nodes`:** `id`, `npc_id`, `text`, `option_1_text`, `option_1_target_node_id`, `option_2_text`, `option_2_target_node_id` (or a more complex structure for dialogue trees).

---

## Future Extensions (after the core API)

* **Combat System:** Adding `Monster`, `Weapon`, `Armor` entities and endpoints for initiating and managing combat.
* **Quest System:** Managing quests, their status, objectives, and rewards.
* **Character Attributes:** Expanding with strength, dexterity, intelligence, and their impact on interactions.
* **Magic/Skills:** Implementing a system for abilities and spells.
* **Visualization:** The API will provide all necessary data for the front-end to render locations, characters, and items, facilitating the transition to a graphical version.
