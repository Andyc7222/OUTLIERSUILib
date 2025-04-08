# OutliersUILib

## Overview

OutliersUILib is a Lua library for creating interactive, keyboard/gamepad-navigable user interfaces within Roblox exploits. It leverages the `Drawing` library for rendering, providing a structured way to build menus with tabs, columns, and various interactive elements. It includes robust configuration management and a strong emphasis on cleanup via its `unload` function.

## Features

*   **Tabbed Interface:** Organize UI elements into distinct tabs.
*   **Column Layout:** Arrange items within tabs using columns.
*   **Keyboard/Gamepad Navigation:** Full control using directional keys, Enter, and Backspace/RCtrl.
*   **Diverse Item Types:** Buttons, Toggles, Sliders (incl. "Infinite" mode), Select (Dropdown), MultiDropdown, PlayerList (auto-updating), PlayerListMulti, Color Pickers (with Rainbow mode), Keybinds, Textboxes, and Separators.
*   **State Management:** Uses a global `flags` table (`getgenv().flags`) for easy access and modification of item states.
*   **Item Actions:** Define behavior for Toggles (RenderStepped/Heartbeat functions, property loops, metamethod hooks).
*   **Configuration System:** Save and load UI states (item values, colors, layout, flags) to JSON files. Built-in configuration tab for management.
*   **Notification System:** Display temporary messages on screen.
*   **Keybind List:** Optionally display currently active keybinds.
*   **Enhanced Load/Unload:** `init()` automatically cleans up previous instances before loading. `unload()` thoroughly removes all Drawings, disconnects events, restores hooks, and clears state to prevent memory leaks and errors.
*   **Customizable:** Modify colors, text settings, and layout dimensions.

## Dependencies

*   A functional `Drawing` library.
*   Exploit environment providing:
    *   File system functions (`readfile`, `writefile`, `delfile`, `isfolder`, `makefolder`, `listfiles`) for the configuration system.
    *   `HttpService` (for JSON operations and GUIDs).
    *   `hookmetamethod` (for certain Item Actions).
    *   Standard Roblox services (`UserInputService`, `RunService`, `Players`).

## Setup

1.  **Load the Library:** Obtain the script content and load it into your environment.
    ```lua
    -- Example: Replace with your actual loading method
    local OutliersUILib = loadstring(game:HttpGet("URL_TO_YOUR_RAW_OutliersUILib.lua"))()
    ```
2.  **Initialize:** Call the `init()` function. This creates the default "Configuration" tab and draws the initial UI.
    ```lua
    OutliersUILib:init()
    ```
3.  **Add Content:** Use the API functions (`addTab`, `addColumn`, `addItem`) to build your UI structure and elements.

## Core Concepts

### `flags` Table (`getgenv().flags`)

This global table is central to the library's state management.
*   When you create or modify a UI item that has a `flag` property defined, its current value (e.g., toggle state, slider number, selected option) is stored in `getgenv().flags[flagName]`.
*   Item `action`s and callbacks often rely on reading values from this table.
*   This allows different parts of your script (or even other scripts) to react to UI changes easily by checking `getgenv().flags`.

### Navigation

The UI is controlled via keyboard or gamepad:

*   **Up/Down:** Navigate vertically between items/options. Move between selection levels (Tabs -> Columns -> Items).
*   **Left/Right:** Navigate horizontally between tabs/columns. Modify item values (Sliders, Keybind modes, ColorPicker values). *Holding accelerates changes.*
*   **Enter/KeypadFive:** Select/Confirm. Enter a level, activate buttons, open dropdowns/color pickers/keybind listeners/textboxes, toggle items.
*   **Backspace/KeypadSeven/RightControl:** Go Back. Close popups (dropdowns, etc.), move up a selection level (Items -> Columns -> Tabs).

### Load/Unload Lifecycle

*   **`init()`:** Sets up the UI. **Crucially, it calls `unload()` first if the library was already loaded**, ensuring a clean state before rebuilding.
*   **`unload()`:** **Essential for cleanup.** Removes all Drawings, disconnects all event listeners, restores any hooked metamethods, resets looped property modifications, clears internal state, and resets the `flags` table. **Always call `unload()` when your script finishes or the UI is no longer needed.** Failure to do so can lead to visual artifacts, errors, and memory leaks.

---

## API Reference

### UI Structure

*   **`OutliersUILib:addTab(name)`**
    *   Adds a new tab.
    *   `name` (string): Tab label.
    *   *Returns:* `tab` object (table).

*   **`OutliersUILib:addColumn(tabObject, name)`**
    *   Adds a column to a tab.
    *   `tabObject` (table): The object returned by `addTab`.
    *   `name` (string): Column header label.
    *   *Returns:* `column` object (table).

*   **`OutliersUILib:addItem(columnObject, itemTable)`**
    *   Adds an item to a column.
    *   `columnObject` (table): The object returned by `addColumn`.
    *   `itemTable` (table): Defines the item (see [UI Item Types](#ui-item-types)).
    *   *Returns:* The `itemTable` object.

### UI Management

*   **`OutliersUILib:setVisibility(visible, duration)`**
    *   Shows/hides the UI with an optional fade.
    *   `visible` (boolean): `true` to show, `false` to hide.
    *   `duration` (number, optional): Fade duration in seconds (default: 0.3).

*   **`OutliersUILib:unload()`**
    *   **Critical function.** Completely removes the UI and cleans up all resources. Call this when finished.

### Notifications

*   **`OutliersUILib:notify(optionsOrString)`**
    *   Displays a notification.
    *   `optionsOrString` (table | string): Notification content or a table with options (`title`, `content`, `duration`, `width`, `height`, `textColor`, `titleColor`, `bgColor`, `outlineColor`).
    *   *Returns:* Notification `id` (string).

*   **`OutliersUILib:closeNotification(id)`**
    *   Closes a specific notification by ID.
    *   `id` (string): The ID returned by `notify`.

*   **`OutliersUILib:closeAllNotifications()`**
    *   Closes all active notifications.

*   **`OutliersUILib:updateNotificationSettings(settingsTable)`**
    *   Updates default notification appearance/behavior.
    *   `settingsTable` (table): Options (`maxNotifications`, `lifetime`, `height`, `spacing`, `width`, `animationSpeed`).

### Configuration Management

*(Requires file system permissions)* Files are stored in the `OutliersConfigs` folder.

*   **`OutliersUILib:getConfigList()`**
    *   Gets a list of saved config names (without `.json`).
    *   *Returns:* `table` (array of strings).

*   **`OutliersUILib:saveConfig(name)`**
    *   Saves the current UI state (item values, flags, colors, text settings) to `OutliersConfigs/<name>.json`.
    *   `name` (string): Config file name.
    *   *Returns:* `success` (boolean), `message` (string).

*   **`OutliersUILib:loadConfig(name)`**
    *   Loads state from `OutliersConfigs/<name>.json`. Updates UI elements, flags, settings, and re-initializes active toggle actions.
    *   `name` (string): Config file name to load.
    *   *Returns:* `success` (boolean), `message` (string).

*   **`OutliersUILib:deleteConfig(name)`**
    *   Deletes `OutliersConfigs/<name>.json`.
    *   `name` (string): Config file name to delete.
    *   *Returns:* `success` (boolean), `message` (string).

---

## UI Item Types

All items added via `addItem` use a table with these common properties:

*   `name` (string): Display label for the item.
*   `type` (string): The type identifier (see below).
*   `flag` (string, optional): Key in `getgenv().flags` to store this item's state.
*   `callback` (function, optional): Called when the item's value changes or is activated. Arguments vary by type.

**Specific Types:**

*   ### `Separator`
    *   `type = "Separator"`
    *   `name` (string, optional): Text displayed between hyphens (`- Name -`).

*   ### `Toggle`
    *   `type = "Toggle"`
    *   `value` (string): Initial state: `"On"` or `"Off"`.
    *   `flag` (string): Stores `true` for "On", `false` for "Off".
    *   `callback(newValue)`: Called with `"On"` or `"Off"`.
    *   `action` (table, optional): Defines behavior while "On" (see [Item Actions](#item-actions)).

*   ### `Button`
    *   `type = "Button"`
    *   `callback()`: Called when pressed (Enter).

*   ### `Slider`
    *   `type = "Slider"`
    *   `value` (number | string): Initial value. Can be `"Inf"` if `mode = "Inf"`.
    *   `min` (number, default=0): Minimum value.
    *   `max` (number, default=1000): Maximum value.
    *   `increment` (number, default=1): Step value. Supports decimals.
    *   `mode` (string, optional): Set to `"Inf"` for infinity (`math.huge`).
    *   `flag` (string): Stores the current numerical value (or `math.huge`).
    *   `callback(newValue)`: Called with the new number.

*   ### `Select`
    *   `type = "Select"`
    *   `value` (string): The initially selected option.
    *   `options` (table): Array of available string choices.
    *   `shortDisplay` (boolean, optional): If `true`, show only first 3 chars in main UI.
    *   `flag` (string): Stores the selected string.
    *   `callback(newValue)`: Called with the selected string.

*   ### `MultiDropdown`
    *   `type = "MultiDropdown"`
    *   `values` (table): Array of initially selected option strings.
    *   `options` (table): Array of all available string choices.
    *   `flag` (string): Stores the table of selected strings.
    *   `callback(newValuesTable)`: Called with the updated table of selections.

*   ### `PlayerList`
    *   `type = "PlayerList"`
    *   Like `Select`, but `options` are automatically populated/updated with player names.
    *   `value` (string): Selected player name. Auto-updates if player leaves.
    *   `flag` (string): Stores the selected player name.
    *   `callback(newValue)`: Called with the selected player name.

*   ### `PlayerListMulti`
    *   `type = "PlayerListMulti"`
    *   Like `MultiDropdown`, but `options` are auto-updated player names. Selections are pruned if players leave.
    *   `values` (table): Array of selected player names.
    *   `flag` (string): Stores the table of selected player names.
    *   `callback(newValuesTable)`: Called with the updated table.

*   ### `ColorPicker`
    *   `type = "ColorPicker"`
    *   `value` (Color3): Initial color.
    *   `rainbow` (boolean, default=false): Enable rainbow cycling.
    *   `rainbowSpeed` (number, default=5): Rainbow speed (1-20).
    *   `flag` (string): Stores the current `Color3` value.
    *   `callback(newValue)`: Called with the updated `Color3`.

*   ### `Keybind`
    *   `type = "Keybind"`
    *   `key` (KeyCode | UserInputType, optional): Initial `Enum`.
    *   `keyName` (string, optional): String name ("F", "MouseButton1", "None"). Used for display/saving.
    *   `mode` (string): Activation mode (e.g., "Toggle", "Hold", "Always On", "None"). Must be in `modes`.
    *   `modes` (table): Array of allowed mode strings.
    *   `flag` (string): Stores `true`/`false` based on key state and mode.
    *   `callback(keyName, mode)`: Called when key or mode changes.

*   ### `Textbox`
    *   `type = "Textbox"`
    *   `value` (string): Initial text content. Basic input (A-Z, backspace, enter, esc).
    *   `flag` (string): Stores the current text content.
    *   `callback(newValue)`: Called when Enter is pressed.

---

## Item Actions (`action` property for `Toggle`)

Defines code executed while a `Toggle` item is `"On"`. Cleaned up automatically by `unload()`.

*   **Common `action` Property:**
    *   `type` (string): The action type identifier.

*   **Action Types:**

    *   **`RenderSteppedFunc` / `HeartbeatFunc`**
        *   `type = "RenderSteppedFunc"` or `"HeartbeatFunc"`
        *   `func` (function): Function to run every frame (only if toggle flag is true).

    *   **`loopModify`**
        *   Continuously sets an object's property based on another flag's value (e.g., linking a speed toggle to a speed slider).
        *   `type = "loopModify"`
        *   `object` (Instance): The instance to modify.
        *   `property` (string): Property name (e.g., `"WalkSpeed"`).
        *   `flag` (string): Flag name (usually from a Slider) providing the value.
        *   `flagMax` (number, optional): If set, property = `flagMax - flags[flag]`.
        *   `keybindFlag` (string, optional): Only modify if `flags[keybindFlag]` is true.
        *   `storeOldValue` (boolean, optional): If true, original value is restored on toggle off/unload.

    *   **`hookIndex`**
        *   Hooks `__index` metamethod.
        *   `type = "hookIndex"`
        *   `object` (Instance | table, default=game): Object to hook.
        *   `callback(target, key)`: Called on property access when toggle is "On".
            *   Return `true, value` to override the lookup.
            *   Return `false` or nothing to proceed normally.

    *   **`hookNamecall`**
        *   Hooks `__namecall` metamethod.
        *   `type = "hookNamecall"`
        *   `object` (Instance | table, default=game): Object to hook.
        *   `callback(methodName, argsTable)`: Called on method call when toggle is "On".
            *   Return `true, newArgsTable` to proceed with modified arguments.
            *   Return `"R", returnValue` to intercept and return `returnValue`.
            *   Return `false` or nothing to proceed normally.

    *   **`multiple`**
        *   Groups multiple actions under one toggle.
        *   `type = "multiple"`
        *   `actions` (table): An array of individual action tables.

---

## Customization

Default appearance can be modified *before* calling `init()` by changing the `UIColorSettings` and `UITextSettings` tables near the top of the library script, or the layout variables (`startX`, `startY`, `baseColumnWidth`, etc.).

Alternatively, use the built-in "Configuration" tab *after* initialization to adjust these settings live and save them to a config file.

---

## Example Usage

```lua
-- Assuming OutliersUILib is loaded
local OutliersUILib = ...

-- Initialize UI (includes Config tab)
OutliersUILib:init()

-- Add a custom tab
local myTab = OutliersUILib:addTab("My Features")

-- Add a column
local mainCol = OutliersUILib:addColumn(myTab, "Utilities")

-- Add a toggle linked to a slider for walkspeed
OutliersUILib:addItem(mainCol, {
    type = "Toggle",
    name = "Enable Speed",
    value = "Off",
    flag = "SpeedEnabled",
    action = {
        type = "loopModify",
        object = game:GetService("Players").LocalPlayer.Character and game:GetService("Players").LocalPlayer.Character:FindFirstChildOfClass("Humanoid"), -- Basic check
        property = "WalkSpeed",
        flag = "SpeedValue", -- Linked slider flag
        storeOldValue = true -- Restore speed on disable/unload
    },
    callback = function(val)
        if val == "Off" and game:GetService("Players").LocalPlayer.Character and game:GetService("Players").LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
           -- Manually reset if turned off via UI click and storeOldValue didn't run yet
           game:GetService("Players").LocalPlayer.Character.Humanoid.WalkSpeed = 16
        end
        print("Speed Enabled:", val)
    end
})

OutliersUILib:addItem(mainCol, {
    type = "Slider",
    name = "Speed Amount",
    value = 50,
    min = 16,
    max = 200,
    flag = "SpeedValue" -- Flag used by the toggle's action
})

-- Add a button for notification
OutliersUILib:addItem(mainCol, {
    type = "Button",
    name = "Show Message",
    callback = function()
        OutliersUILib:notify({ title = "Info", content = "Button was pressed!" })
    end
})

-- Add a keybind
OutliersUILib:addItem(mainCol, {
    type = "Keybind",
    name = "Action Key",
    keyName = "Q",
    key = Enum.KeyCode.Q,
    mode = "Hold",
    modes = {"Hold", "Toggle", "None"},
    flag = "ActionKeyHeld"
})

-- IMPORTANT: Remember to unload when finished!
-- For example, in your script's cleanup routine:
-- OutliersUILib:unload()
