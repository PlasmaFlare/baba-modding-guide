# Undo System
The undo system might seem daunting to work with at first. However, the way that it is actually implemented is fairly straightforward.

At its core, the game maintains a global table called `undobuffer`. This used as the main undo stack used for saving and restoring state between turns. `undobuffer` mainly gets manipulated in `undo.lua` (unsurprisingly) through three main functions:
- `newundo()` - adds a new undo entry to `undobuffer`
- `addundo()` - adds a "line" of undo data to the current undo entry (This will be explained in the next section).
- `undo()` - removes the last undo entry from the stack and applies all "lines" of undo data

(Note: I'm ignoring `undostate()` since its mostly updating a variable)

## Undo Entry Contents
The game's undo system works based on delta information. This means that each undo entry in `undobuffer` stores a list of all changes to the level state between turns. For example, if from Turn 1 to Turn 2:
- The Baba at (3,4) moves to (3,5)
- The Keke at (6,9) gets destroyed
- A Me gets created at (0,1)

The game will record all of the above information into the current undo entry, which can then be used to restore state if the player undos from Turn 2 to Turn 1.

### "Lines" of undo data

Each bullet from above is represented in what the game calls a "line" of undo data. Each line is a table who's contents depend on the type of change to the level's state between turns. `line[1]` is always a string identifying the type.

Below is a list of all possible line formats. Each field in a line can be accessed by using the corresponding index. So if `line[1] == "update"` and I wanted to get the new Y coordinate of the updated object, I would get it by `line[4]`.

```lua
-- Update
{
    [1] = "update",
    [2] = name,
    [3] = newx,
    [4] = newy,
    [5] = newdir,
    [6] = oldx,
    [7] = oldy,
    [8] = olddir,
    [9] = unit.values[ID],
    [10] = unitid
}

-- Create
{
    [1] = "create",
    [2] = name,
    [3] = unit.values[ID],
    [4] = baseingameid, -- (unused)
    [5] = source of creation, -- Can be "create", "convert", "emptyconvert" or "back"
    [6] = x,
    [7] = y,
    [8] = dir,
}

-- Remove
{
    [1] = "remove",
    [2] = name,
    [3] = x,
    [4] = y,
    [5] = dir,
    [6] = unit.values[ID],
    [7] = baseingameid (unused),
    [8] = unit.strings[U_LEVELFILE], -- (8-13 are related to leveldata if this object is a transformed level)
    [9] = unit.strings[U_LEVELNAME],
    [10] = unit.values[VISUALLEVEL],
    [11] = unit.values[COMPLETED],
    [12] = unit.values[VISUALSTYLE],
    [13] = unit.flags[MAPLEVEL],
    [14] = unit.strings[COLOUR],
    [15] = unit.strings[CLEARCOLOUR],
    [16] = unit.followed,
    [17] = unit.back_init,
    [18] = unit.originalname,
}

-- Backset (handling "Back")
{
    [1] = "backset",
    [2] = name,
    [3] = unit.values[ID],
    [4] = unit.back_init,
}

-- Done
{
    [1] = "done",
    [2] = name,
    [3] = x,
    [4] = y,
    [5] = dir,
    [6] = unit.values[ID],
    [7] = unitid,
    [8] = unit.values[FLOAT],
}

-- Float
{
    [1] = "float",
    [2] = name,
    [3] = unit.values[ID],
    [4] = prev float value,
    [5] = new float value,
}

-- LevelUpdate
{
    [1] = "levelupdate",
    [2] = old xoffset,
    [3] = old yoffset,
    [4] = new xoffset,
    [5] = new yoffset,
    [6] = old level dir,
    [7] = new level dir,
}

-- MapRotation
{
    [1] = "maprotation",
    [2] = old level rotation, -- (in degrees)
    [3] = new level rotation, -- (in degrees)
    [4] = new level dir,
}

-- MapDir
{
    [1] = "mapdir",
    [2] = old level dir,
    [3] = new level dir,
}

-- MapCursor
{
    [1] = "mapcursor",
    [2] = old unit.values[CURSOR_ONLEVEL], -- (the levelname of the level the cursor is on)
    [3] = old x,
    [4] = old y,
    [5] = old dir,
    [6] = new unit.values[CURSOR_ONLEVEL],
    [7] = new x,
    [8] = new y,
    [9] = new dir,
    [10] = unit.values[ID],
    
}

-- Colour (unused)
{
    [1] = "colour",
    [2] = unit.values[ID],
    [3] = color 1,
    [4] = color 2,
    [5] = unit.values[A] (Alpha value?),
}

-- Broken
{
    [1] = "broken",
    [2] = 0 or 1, -- (0 = not broken, 1 = broken)
    [3] = unit.values[ID],
    [4] = name,
}

-- Bonus
{
    [1] = "bonus",
    [2] = 0 or 1, -- (0 = bonus not collected, 1 = bonus collected)
}

-- Followed
{
    [1] = "followed",
    [2] = unit.values[ID],
    [3] = unit.followed,
    [4] = follow priority, -- (something specific with follow logic)
    [5] = name,
    [6] = unitid,
}

-- StartVision (3D related)
{
    [1] = "startvision",
    [2] = unitid of target object to take first person perspective of -- (alt, theres a weird "0.5" value as well, but idk what it does),
}

-- StopVision (3D related)
{
    [1] = "stopvision",
    [2] = unit.values[ID] of target object that had first person perspective -- (alt, theres a weird "0.5" value as well, but idk what it does),
    [3] = camera x,
    [4] = camera y,
    [5] = camera dir,
}

-- VisionTarget (3D related) (related to switching 3d perspectives)
{
    [1] = "visiontarget",
    [2] = unit.values[ID] of old target object
    [3] = unit.values[ID] of new target object
}

-- Holder
{
    [1] = "holder",
    [2] = unit.values[ID] of object that is holding something
    [3] = unit.holder,
    [4] = unit.values[ID] of object that is being holded
}
```