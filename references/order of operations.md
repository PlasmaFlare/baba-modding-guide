# Major Function Call Order
The old baba wiki page "Advanced Rulebook" removed the exact function call order whenever the game executes a turn. So I've decided one day to just make a bunch of print statements on some of the major functions and see which gets called first.

These are the functions in which I added `print()`'s for:

## Major Functions
#### **In syntax.lua**
- `command()` - gets called whenever the game receives a player input
#### **In movement.lua**
- `movecommand()` - the main function for handling all movement logic
#### **In convert.lua**
- `conversion()` - the main function for handling all transformation logic
- `dolevelconversions()` - handles level transformations
#### **In rules.lua**
- `code()` - updates the set of rules parsed from the text blocks, if needed (see [Overview of parsing](parsing.md))
#### **In update.lua**
- `doupdate()` - apply simultaneous changes to units, and other game state
#### **In clears.lua**
- `smallclear()` - one of the few functions that clears a few globals. (The other clear functions weren't called at all during turn execution)
#### **In blocks.lua**
- `block()` - contains code for a lot of properties that destroy/create/update units based on overlap, as well as other smaller effects like `play`.
- `moveblock()` - oddly enough, it contains code for properties that doesn't do actual movement like in `movecommand()`. Instead, it updates direction and/or position of objects. (I honestly wonder if this function used have the main movement logic.)
- `levelblock()` - contains code for all properties covered in `block()`, but applied to the overall level *and also empty tiles*
- `fallblock()` - contains code for handling `fall` logic
- `effectblock()` - handles visual properties. Mainly color properties and `hide`
- `findplayer()` - seems to handle the special logic that causes the music to go silent if there's no `you` objects on the level
- `diceblock()` - seems to be a discarded function that Hempuli originally used when handling `often` and `seldom` rng.

#### **Other functions**
- `MF_update()` - A MMF function that calls a lot of other major functions after `movecommand()`.

---
## Call Order
**Note**: indents mean that the parent function called the child function. Ex: `command()` calls `movecommand()` while executing.
### When executing a turn
- `command()`
  - `movecommand()`
    - `doupdate()` x 1-8 depending on how many different types of movement gets processed.
    - `doupdate()`
    - `code()`
    - `conversion()`
    - `doupdate()`
    - `code()`
    - `moveblock()`
      - `doupdate()` x 2
  - `MF_update()`
    - `doupdate()`
    - `smallclear()`
    - `code()` 
    - `fallblock()` 
    - `doupdate()` 
    - `code()` 
    - `conversion()` 
    - `fallblock()` 
    - `doupdate()` 
    - `code()` 
    - `levelblock()` 
    - `block()` 
    - `dolevelconversions()` 
    - `doupdate()` 
    - `smallclear()`
    - `code()`
    - `findplayer()`
    - `effectblock()`
    - `diceblock()`


### On undo
- `undo()`
- `doupdate()`
- `smallclear()`
- `code()`
- `doupdate()`
- `code()`
- `doupdate()`
- `smallclear()`
- `code()`
- `findplayer()`
- `effectblock()`
- `diceblock()`