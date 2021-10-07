# Rules
A rule is a representation of the various sentences that you can make in Baba Is You. The game's code has a few different names for rules. Namely "sentence", "rule, "option" and "feature". For the purposes of this guide, I'll only use "rule". 

One key variable that is essential to getting rule data is a global variable called `featureindex`. This is a table that gets repopulated with rules when reparsing happens. It serves as the main lookup table for searching rules formed by the text blocks in game, giving a base point to implement the various properties and other interactions seen in game.

Most of the time, you would interface with `featureindex` using functions defined in `features.lua`. [Click here to get rundown of common functions](#featureslua-functions).



## Rule format
 
Below is a rule stored for **"keke on baba and not near me is you"**:
```lua
{
    -- rule[1] = Basic rule
    {"keke", "is", "you"}, 

    -- rule[2] = Conditions
    {                      
        { "on" ,      {"baba"} },
        { "not near", {"me"}   },
    },

    -- rule[3] = unitids
    {<unitid of "keke">, <unitid of "is">, <unitid of "you">, <unitid of "on">, <unitid of "baba"> ... },

    -- rule[4] = tags
    {}
}
```

A breakdown of each of the components:
- `rule[1]` = The rule without any conditionals, which I call the "basic rule". This is in the format `{target, verb, property}`.
  - `target` and `property` can have "not " (with the space) prepended. `{"not baba", is, "not blue"}`
- `rule[2]` = List of conditionals. For each condition `cond`:
  - `cond[1]` = The name of the condition, called the "condition type". 
    - Can have "not " (with the space) prepended ("baba **not facing** fence is blue")
  - `cond[2]` = List of parameters for the condition. 
    - This is present in infix conditions ("on","near","above" etc) but not present in prefix conditions ("lonely", "idle", "seldom" etc)
    - A parameter can have "not " prepended. ("baba facing **not fence** is blue")
- `rule[3]` = List of unitids for all text units involved in the original formation of the rule.
- `rule[4]` = List of tags. Tags are strings only used within the game's code to arbitrarily mark rules for implementing certain words. Some tags I've seen:
  - `base` - indicates that this is a baserule ("text is push", "level is stop")
  - `mimic` - indicates that this is a rule generated from "mimic" copying another rule.


## Featureindex format
`featureindex` mainly stores lists of rules grouped by the content of the basic rule (`rule[1]`). A rule can be stored in multiple lists within `featureindex`. Here's an example of what featureindex can look like.
```
featureindex = {
    "keke" = [List of rules with "keke" as either a target or property]
    "push" = [List of rules with "push" as a property],
    "is" = [List of rules with "is" as a verb]
}
```
(For a closer look at how `featureindex` is populated look at the function `addoption()` in `rules.lua`)

# `features.lua` Functions
(Note: This doesn't cover all functions, but some of the more common functions used for getting rule data)

(Note2: I omitted a few parameters in some of the functions since they are not used in common scenarios)

### `findfeature(target, verb, property)`
```lua
local result = findfeature("keke", "is", "push")
local result2 = findfeature(nil, "is", "pink")
```
Returns a list of each rule in `featureindex` that fits the basic rule. 
- Each item in the list is formated as `{basic rule, conditions}`. 
- Setting `target` to `nil` acts as a wildcard.
---

### `hasfeature(target, verb, property, unitid, x, y)`
```lua
local unit_has_feature = hasfeature("baba", "is", "you", unitid)
```
Returns true or false of whether or not the unit has the basic rule fulfilled. 
- Setting `target` to `nil` acts as a wildcard.
- Conditions get evaluated, meaning that all unitids returned satisfy any additional conditions stored for the basic rule. 
- Normally `x` and `y` can be omitted. But if `target` is "empty", `x` and `y` have to be filled in and `unitid` needs to be set to 2.
---

### `findfeatureat(target, verb, property, x, y)`
```lua
local list_of_unitids = findfeatureat(nil, "is", "pink", 4, 5)
```
Returns a list of unitids at (x,y) in the level grid that fulfill the basic rule. 
- Setting `target` to `nil` acts as a wildcard. 
- Conditions get evaluated like hasfeature(). 
---

### `hasfeature_count(rule1,rule2,rule3,unitid,x,y)`
```lua
local unit_has_feature = hasfeature("baba", "is", "you", unitid)
```
Similar to hasfeature(), but returns a count of how many rules in `featureindex` have their conditions satisfied. This can be used to stack rules.
- Setting `target` to `nil` acts as a wildcard.
- Conditions get evaluated like hasfeature(). 
- Normally `x` and `y` can be omitted. But if `target` is "empty", `x` and `y` have to be filled in and `unitid` needs to be set to 2.

---
### `findallfeature(rule1,rule2,rule3,ignore_empty_)`
```lua
local all_unitids_with_rule, all_empties_with_rule = findallfeature(nil, "is", "you")
```
Uses `findfeature()` to get all unitids/empties with the specified base rule. `ignore_empty_` can be set to a boolean to choose whether to return all empties with the rule. If it is omitted, it is set to false.
- If there are empties, `all_unitids_with_rule` will contain 2's to indicate empty. `all_empties_with_rule` would be formatted as:
    ```lua
    all_empties_with_rule = {
        -- Results from one rule. Each tileid indicates a position of the empty that fulfills the rule
        {
            <tileid> = 0,
            <tileid> = 0,
            <tileid> = 0,
        },

        -- Results from another rule
        {
            <tileid> = 0,
            <tileid> = 0,
            <tileid> = 0,
        },

        -- a tileid encodes a coordinate pair into one number
        -- tileid = x + y * roomsizex
    }
    ```