# Brief overview of the game's Lua files
Note that this list isn't comprehensive, but it should be enough to cover files a modder would commonly be dealing with.
## Main logic
- `movement.lua` - handles movement logic with move sources (`you`, `move`, `shift`, etc) and movements that are a direct effect of move sources (`push`, `pull`, `swap`)
- `rules.lua` - main parsing logic is handled here
- `conditions.lua` - handles all condition texts (`on`, `near`, `lonely`, `facing` etc)
- `blocks.lua` - contains what the source code calls "blocks", which are various functions that handle properties/effects that do not have to do with movement or collision-based effects
- `features.lua` - has functions for looking up objects that have a certain property
- `update.lua` - contains `doupdate()` and a few other functions. `doupdate()` is used to consolidate changes to the game state as a turn progresses.
- `convert.lua` - handles any text effects that involves transforming different objects
- `letterunits.lua` - handles parsing letter text blocks
- `vision.lua` - functions for handling the visual effects of the `3D` propoerty
- `mapcursor.lua` - handles `select` logic 

## Constants/Definitions
- `constants.lua` - contains indexes that are used in getting certain properties of objects. (See [Units](references/units.md))
- `values.lua` - contains various constant values used in the game logic
- `modsupport.lua` - defines all the avaliable mod hooks
- `Editor/editor_objectlist.lua` - while this is an editor-related file, it contains definitions of all the objects in the game. (The object list in `values.lua` seems to be a "legacy" version before the editor was implemented)

## State management
- `load.lua` - functions used for initialization of global variables and other stuff (yes, this game uses global variables). You can use this file to see which global variables are defined.
- `clears.lua` - handles clearing global variables
- `undo.lua` - functions that handle the undo system
- `map.lua` - handles overworld logic and levelpack completion data

## Helper functions
- `tools.lua` - contains various helper functions for dealing with the game logic
- `syntax.lua` - contains more helper functions. (I don't really see what's the difference between `syntax.lua` and `tools.lua`, but meh.)

## Cosmetic effects
- `effects.lua` - functions that handle particles and special effects
- `colours.lua` - functions for getting/setting various colors
- `dynamictiling.lua` - handles the "Tiled" animation style that allows walls to update its sprite based on nearby walls

## Misc
- `ending.lua` - does the "end" and "done" ending cutscenes
- `debug.lua` - functions for printing various debug information. (Note: you might have to replace the `MF_alert()`'s if you want to use these functions since `MF_alert()` only works when running in MMF)
- `menu.lua` - functions for various GUI elements
- `changes.lua` - seems to handle updating the level data when modifying it in the level editor