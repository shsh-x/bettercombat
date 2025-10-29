# Mod Documentation

## Monster Health

This section details the refactored implementation for modifying monster health. The new logic significantly improves performance and maintainability by eliminating redundant code blocks and dynamically accessing game data. This approach should serve as a template for refactoring other parts of the mod.

### Core Principles

The new system is built on three core features of Content Patcher:

1.  **`DynamicTokens` for Lists**: A single token, `{{MonsterList}}`, now holds a comma-separated list of all monsters affected by this feature. This creates a single, easily editable source of truth, eliminating the need to repeat the monster list in every patch.

2.  **`DynamicTokens` for Multipliers**: Instead of having a separate, large patch for each configuration option, we now define `DynamicToken`s (like `{{PercentageHealthMultiplier}}` and `{{LuckyHealthMultiplier}}`) that store a calculated multiplier based on the player's selection in `config.json`.

3.  **`ForEach` Patches**: The `Changes` block now uses the `ForEach` action to iterate over the `{{MonsterList}}` token. This allows us to apply a single, generic patch to every monster in the list, rather than having a unique, hardcoded patch for each one.

### Logic Implementation

The entire monster health modification is now handled by two main `ForEach` blocks in `content.jsonc`, controlled by `When` conditions.

#### 1. Percentage-Based Health (`HP +/-%`)

This logic applies when the user selects any option containing "HP" in the config.

-   **Trigger**: `"When": { "MonstersHealth|contains=HP": true }`
-   **Action**: A `ForEach` patch iterates through each monster name in `{{MonsterList}}`.
-   **Multiplier**: The `{{PercentageHealthMultiplier}}` token is set based on the config choice (e.g., `HP -90%` sets the multiplier to `0.10`).
-   **Calculation**: For each monster, the new health is calculated with the following expression, which dynamically fetches the monster's base health from the game data (`Data/Monsters`) at index 0:

    ```
    "Value": "{{Round: {{Query: {{Data/Monsters:{{PatchValue}}:0}} * {{PercentageHealthMultiplier}} }} }}"
    ```
    *(Note: `{{PatchValue}}` is the special token within a `ForEach` block that represents the current item in the list, i.e., the monster's name.)*

#### 2. Lucky Health

This logic applies when the user selects any "Lucky Health" option.

-   **Trigger**: `"When": { "MonstersHealth|contains=Lucky": true }`
-   **Action**: A second `ForEach` patch iterates through the `{{MonsterList}}`.
-   **Multiplier**: The `{{LuckyHealthMultiplier}}` token is set based on both the config choice (`Easy`, `Easier`, `Easiest`) and the player's `{{DailyLuck}}`.
-   **Calculation**: The calculation is similar, but uses the luck-based multiplier:

    ```
    "Value": "{{Round: {{Query: {{Data/Monsters:{{PatchValue}}:0}} * {{LuckyHealthMultiplier}} }} }}"
    ```

The specific logic for the `LuckyHealthMultiplier` is as follows:

-   **Lucky Health (Easy)**
    -   `> 0.07` Luck: **-25%** HP (0.75x multiplier)
    -   `< -0.07` Luck: **+25%** HP (1.25x multiplier)

-   **Lucky Health (Easier)**
    -   `> 0.07` Luck: **-50%** HP (0.50x multiplier)
    -   `> 0.02` to `<= 0.07` Luck: **-25%** HP (0.75x multiplier)
    -   `< -0.02` to `>= -0.07` Luck: **+5%** HP (1.05x multiplier)
    -   `< -0.07` Luck: **+15%** HP (1.15x multiplier)

-   **Lucky Health (Easiest)**
    -   `> 0.07` Luck: **-80%** HP (0.20x multiplier)
    -   `> 0.02` to `<= 0.07` Luck: **-50%** HP (0.50x multiplier)
    -   `< 0.02` Luck: **No change** (1.0x multiplier)

## Damage Received

This section details the implementation for modifying the damage players receive from monsters. It follows the same core principles as the Monster Health section.

### Logic Implementation

Monster damage modification is handled by two `ForEach` blocks, which are controlled by the `DamageReceived` configuration setting.

#### 1. Percentage-Based Damage (`DMG +/-%`)

This logic applies when the user selects any option containing "DMG" in the config.

-   **Trigger**: `"When": { "DamageReceived|contains=DMG": true }`
-   **Action**: A `ForEach` patch iterates through each monster name in `{{MonsterList}}`.
-   **Multiplier**: The `{{DamageMultiplier}}` token is set based on the config choice (e.g., `DMG -90%` sets the multiplier to `0.10`).
-   **Calculation**: For each monster, the new damage is calculated by fetching the base damage from game data at index 1:

    ```
    "Value": "{{Round: {{Query: {{Data/Monsters:{{PatchValue}}:1}} * {{DamageMultiplier}} }} }}"
    ```

#### 2. Lucky Damage

This logic applies when the user selects any "Lucky Damage" option. It provides a risk/reward system where bad luck increases damage taken, and good luck decreases it.

-   **Trigger**: `"When": { "DamageReceived|contains=Lucky": true }`
-   **Action**: A second `ForEach` patch iterates through the `{{MonsterList}}`.
-   **Multiplier**: The `{{LuckyDamageMultiplier}}` token is set based on both the config choice (`Easy`, `Easier`, `Easiest`) and the player's `{{DailyLuck}}`.
-   **Calculation**: The calculation is similar, but uses the luck-based damage multiplier:

    ```
    "Value": "{{Round: {{Query: {{Data/Monsters:{{PatchValue}}:1}} * {{LuckyDamageMultiplier}} }} }}"
    ```

The specific logic for the `LuckyDamageMultiplier` is as follows:

-   **Lucky Damage (Easy)**
    -   `> 0.07` Luck: **-25%** Damage Taken (0.75x multiplier)
    -   `> 0.02` to `<= 0.07` Luck: **-15%** Damage Taken (0.85x multiplier)
    -   `< -0.02` to `>= -0.07` Luck: **+15%** Damage Taken (1.15x multiplier)
    -   `< -0.07` Luck: **+25%** Damage Taken (1.25x multiplier)

-   **Lucky Damage (Easier)**
    -   `> 0.07` Luck: **-50%** Damage Taken (0.50x multiplier)
    -   `> 0.02` to `<= 0.07` Luck: **-25%** Damage Taken (0.75x multiplier)
    -   `< -0.02` to `>= -0.07` Luck: **+5%** Damage Taken (1.05x multiplier)
    -   `< -0.07` Luck: **+15%** Damage Taken (1.15x multiplier)

-   **Lucky Damage (Easiest)**
    -   `> 0.07` Luck: **-80%** Damage Taken (0.20x multiplier)
    -   `> 0.02` to `<= 0.07` Luck: **-50%** Damage Taken (0.50x multiplier)
    -   `< 0.02` Luck: **No change** (1.0x multiplier)

### How to Adapt This Logic for Other Features

This component-based approach can be used to refactor other parts of the mod:

1.  **Isolate Repeated Data**: Identify a list that is repeated in multiple patches (e.g., a list of weapons, items, or other game objects).
2.  **Create a List Token**: In the `DynamicTokens` section, create a token that holds this data as a single, comma-separated string.
    ```json
    { "Name": "MyObjectList", "Value": "ObjectA,ObjectB,ObjectC" }
    ```
3.  **Create a Multiplier/Value Token**: For the values that change based on `config.json` (like damage, price, or speed), create a dynamic token that holds this value. Use `When` conditions to set it based on the player's selection.
    ```json
    { "Name": "MyValueMultiplier", "Value": "1.5", "When": { "MyConfig": "OptionA" } },
    { "Name": "MyValueMultiplier", "Value": "2.0", "When": { "MyConfig": "OptionB" } }
    ```
4.  **Build a `ForEach` Patch**: In the `Changes` array, create a `ForEach` block that iterates over your list token.
5.  **Create a Generic Sub-Patch**: Inside the `ForEach`, create a single `Patch` that performs the modification. Use `{{PatchValue}}` to refer to the current item from the list. If possible, fetch base stats dynamically from the game data (e.g., `{{Data/Weapons:{{PatchValue}}:2}}` to get a weapon's damage).

This pattern will make the mod significantly more streamlined and easier to update in the future.

## Monster Speed

This section covers the implementation for modifying monster speed. It follows the same dynamic and optimized principles used for the health and damage features.

### Logic Implementation

Monster speed modification is handled by a single `ForEach` block, controlled by the `MonstersSpeed` configuration setting.

-   **Trigger**: `"When": { "MonstersSpeed|contains=Speed": true }`
-   **Action**: A `ForEach` patch iterates through each monster name in the dynamic `{{MonsterList}}`.
-   **Multiplier**: The `{{SpeedMultiplier}}` token is set based on the config choice (e.g., `Speed +50%` sets the multiplier to `1.50`).
-   **Filtering**: The patch includes a `When` condition to ensure it only applies to actual monsters by checking that index 12 in `Data/Monsters` is `true`. This prevents non-monster entities from being affected.
-   **Calculation**: For each monster, the new speed is calculated by fetching the base speed from game data at index 10:

    ```
    "Value": "{{Round: {{Query: {{Data/Monsters:{{PatchValue}}:10}} * {{SpeedMultiplier}} }} }}"
    ```

The available options provide the following adjustments:

-   **Speed +/-25%** (0.75x or 1.25x multiplier)
-   **Speed +/-50%** (0.50x or 1.50x multiplier)
-   **Speed +/-75%** (0.25x or 1.75x multiplier)

## Monster Detection Radius

This section details the refactored and bug-fixed implementation for modifying monster detection radius (their field of view). The original implementation had several bugs, including modifying the wrong game data and inconsistent monster lists. The new logic is reliable and split into two distinct categories for better control.

### Logic Implementation

The feature is controlled by two separate options in the `config.json` file: `GroundMonsterDetection` and `FlyingMonsterDetection`. This required implementing three separate `ForEach` patches to handle the logic correctly.

-   **Data Index**: All patches correctly modify index `9` in `Data/Monsters` for the detection radius.
-   **Multiplier Tokens**: The `{{GroundDetectionMultiplier}}` and `{{FlyingDetectionMultiplier}}` tokens are set based on the config choices, ranging from `0.0` (-100%) to `2.0` (+100%).

#### 1. Ground Monsters

-   **Config Option**: `GroundMonsterDetection`
-   **Target**: Applies to any monster that is **not** a glider (index 4 is `false`) and is **not** named "Ghost".
-   **Action**: A `ForEach` patch iterates through the `{{MonsterList}}` and applies the `{{GroundDetectionMultiplier}}` to all monsters matching the criteria.

#### 2. Flying Monsters

-   **Config Option**: `FlyingMonsterDetection`
-   **Target**: Applies to two groups of monsters:
    1.  All monsters that **are** gliders (index 4 is `true`).
    2.  The **Ghost** monster specifically, which is treated as a flying monster as per the game's internal logic, regardless of its glider flag.
-   **Action**: Two separate `ForEach` patches are used to apply the `{{FlyingDetectionMultiplier}}` to both of these groups, ensuring they are all controlled by the single `FlyingMonsterDetection` setting.

## Experience Points

This section covers the implementation for modifying the experience points (XP) gained from defeating monsters. It follows the same dynamic and optimized principles as the other refactored features.

### Logic Implementation

Monster XP modification is handled by a single `ForEach` block, controlled by the `ExperiencePoints` configuration setting.

-   **Trigger**: `"When": { "ExperiencePoints|not=Default": true }`
-   **Action**: A `ForEach` patch iterates through each monster name in the dynamic `{{MonsterList}}`.
-   **Multiplier**: The `{{ExperienceMultiplier}}` token is set based on the config choice (e.g., `XP +50%` sets the multiplier to `1.50`).
-   **Filtering**: The patch includes a `When` condition to ensure it only applies to actual monsters by checking that index 12 in `Data/Monsters` is `true`.
-   **Calculation**: For each monster, the new XP is calculated by fetching the base XP from game data at index 13:

    ```
    "Value": "{{Round: {{Query: {{Data/Monsters:{{PatchValue}}:13}} * {{ExperienceMultiplier}} }} }}"
    ```

The available options provide the following adjustments:

-   **Increase XP**: +25%, +50%, +100%, +150%, +200%
-   **Decrease XP**: -25%, -50%, -75%

## Monster Loot Drop Rate

This section details the unique, hybrid implementation for modifying monster loot drop rates. Due to the complexity of the loot data string (e.g., `itemID1 probability1 itemID2 probability2...`), a simple multiplier could not be used without corrupting the item IDs.

### Logic Implementation

The solution involves pre-generating a unique formula for each monster, which is then selected dynamically.

-   **Config Option**: `LootDropRate`
-   **Multiplier Token**: A standard `{{LootMultiplier}}` token is created based on the user's selection (e.g., `Rate +50%` sets the multiplier to `1.50`).
-   **Generated Tokens**: The core of this feature is a large set of dynamic tokens, one for each monster (e.g., `NewLootString_Green_Slime`). Each of these tokens contains the monster's specific loot string, but with the probability values replaced by a query that multiplies the base probability by the `{{LootMultiplier}}`.
    ```json
    {
      "Name": "NewLootString_Green_Slime",
      "Value": "766 {{Query: 0.75 * {{LootMultiplier}} }} 766 {{Query: 0.05 * {{LootMultiplier}} }} ..."
    }
    ```
    *(Note: To prevent errors, each calculation is automatically capped at a maximum of 0.99, so no drop rate can exceed 99%)*
-   **Final Patch**: A single, clean `ForEach` patch iterates through the `{{MonsterList}}`. It uses token composition to dynamically build the name of the token it needs to use for the current monster.
    ```
    "Value": "{{NewLootString_{{PatchValue | replace: ' ', '_'}}}}"
    ```
    This fetches the correct, pre-calculated formula string for each monster and applies it to the loot data at index 6.

This hybrid approach, while making the `content.jsonc` file larger, is the only reliable method to handle such a complex data structure within Content Patcher and ensures the feature is both powerful and easy for the end-user to control.

### Shared Loot

This option, when enabled (`true`), replaces the normal, specific loot for each monster with a different, pre-defined loot table that has very low drop probabilities. This is useful for a gameplay experience where monster-specific loot is much rarer.

**Interaction with other settings:**
- This feature **works with** the main `LootDropRate` multiplier. The multiplier will be applied to the probabilities of the shared loot table items.
- This feature **works with** the `PrismaticShardChance` setting. A chance to drop a Prismatic Shard will be added on top of the shared loot table.

### Prismatic Shard Chance

This option gives every monster in the game a small, configurable chance to drop a Prismatic Shard. This chance is added independently to whatever other loot the monster drops.

**Interaction with other settings:**
- The chance is a flat value and is **not** affected by the `LootDropRate` multiplier.
- It works with both normal loot and `Shared Loot`.

## Lucky Adventuring Buffs

This feature was ported directly from the previous version of the mod and did not require refactoring, as its implementation was already efficient.

### Logic Implementation

-   **Config Option**: `LuckyAdventuringBuffs` (true/false).
-   **Action**: When set to `true`, two `EditData` patches are activated:
    1.  The first patch adds a new buff definition to `Data/Buffs`. This buff, named `CombatMadeEasy_BlessingOfCombat`, has randomized effects for stats like Mining Level, Luck, Max Stamina, Speed, Defense, and Attack. The values are chosen from a pre-defined list using the `{{Random}}` token.
    2.  The second patch adds an entry to `Data/TriggerActions`. This trigger fires at the start of each day and has a 20% chance (`RANDOM 0.20`) to apply the aforementioned buff to the player.

## Weapon Modifications

This section covers the refactored implementation for modifying weapon critical hit chance and critical power. The new system replaces hundreds of lines of hardcoded values with two clean, dynamic `ForEach` patches.

### Core Principles

The implementation follows the same principles as other refactored systems:

1.  **`WeaponList` Token**: A `{{WeaponList}}` dynamic token holds a list of all weapon IDs from `Data/Weapons`.
2.  **Value Tokens**: `{{CritChanceValue}}` holds the decimal value to add to the base crit chance (e.g., `0.05` for `+5%`), and `{{CritPowerMultiplier}}` holds the multiplication factor (e.g., `4` for `x4`). These are set based on the user's config selection.
3.  **`ForEach` Patches**: Two `ForEach` patches iterate over the `{{WeaponList}}` to apply changes dynamically.

### Logic Implementation

Unlike monster data which uses numeric indices, weapon data in `Data/Weapons` uses named fields. The new implementation correctly targets these fields.

#### 1. Critical Hit Chance

-   **Config Option**: `WeaponCritChance`
-   **Action**: A `ForEach` patch iterates through `{{WeaponList}}`.
-   **Target Field**: The patch correctly targets the `CritChance` field for each weapon.
-   **Calculation**: The new crit chance is calculated by adding the `{{CritChanceValue}}` to the weapon's base value.

    ```json
    "TargetField": [ "{{PatchValue}}", "CritChance" ],
    "Value": "{{Query: {{Data/Weapons:{{PatchValue}}:CritChance}} + {{CritChanceValue}} }}"
    ```

#### 2. Critical Hit Power

-   **Config Option**: `WeaponCritPower`
-   **Action**: A second `ForEach` patch iterates through `{{WeaponList}}`.
-   **Target Field**: The patch correctly targets the `CritMultiplier` field.
-   **Calculation**: The new crit power is calculated by multiplying the weapon's base value by the `{{CritPowerMultiplier}}`.

    ```json
    "TargetField": [ "{{PatchValue}}", "CritMultiplier" ],
    "Value": "{{Query: {{Data/Weapons:{{PatchValue}}:CritMultiplier}} * {{CritPowerMultiplier}} }}"
    ```
