# Implementing a condition
This assumes you added your custom condition through [this guide](custom_word_start.md). I also recommend looking through [the rule specification](../references/rules.md) as reference.

Everything related to conditions is handled in the file `conditions.lua`. `testcond()` is a function that gets called very often whenever the game wants to evaluate a set of conditions (based on the conditional texts seen in game) on a single object. You rarely have to modify this function, however; all conditions in the game have their own functions inside of a table called `condlist` that get called by `testcond()`. You can add individual conditions to this table without having to copy the entire `conditions.lua` file.

Each function simply returns `true` or `false` if the unit fulfils one individual condition, and `testcond()` returns whether the unit fulfils all conditions in the entire set.

## Base form of a condition function
<details>
<summary>Adding a condition function to <code>condlist</code>. Click to expand.</summary>

```lua
-- add "mycondition" to condlist
condlist["mycondition"] = function(params,checkedconds,checkedconds_,cdata)
    -- Read necessary information about the unit off of cdata, see full list below
    local unitid,x,y = cdata.unitid,cdata.x,cdata.y

    -- Calculate whether the unit fulfils this condition, checking each name in the list "params" if this is an infix condition
    local success = false
    
    -- Return whether the unit has fulfilled the condition
    return (success == true),checkedconds
end
```
</details>

For conditions that have been inverted by a "not" text, `testcond()` will automatically negate the condition function's output. You can manually check if the condition is inverted inside its function if different behavior is needed, but keep in mind that the result will be the opposite of whatever you return.

<details>
<summary>List of all <code>cdata</code> entries and how to read them. Click to expand.</summary>

```lua
local name      = cdata.name
local unitid    = cdata.unitid         
local x,y       = cdata.x,cdata.y
local dir       = cdata.dir
local limit     = cdata.limit           -- This is to be passed to `testcond()` if your condition function checks other conditions. If it exceeds 80, an infinite loop is triggered.
local extras    = cdata.extras
local surrounds = cdata.surrounds       -- This contains data about objects that surround the current level on the outer map. See implementation of "near".
local subtype   = cdata.subtype         -- This is used by "powered[abc]" conditions, where "[abc]" can be any string that matches a unique condition text (you must modify testcond() to support this)
local conds     = tostring(cdata.conds) -- The list of all conditions in the rule that this condition is part of.
local i         = cdata.i               -- The index of this condition into the aforementioned list of conditions.
local notcond   = cdata.notcond         -- Whether this condition has been inverted by a "not" text.
local debugname = cdata.debugname       -- This is not used by any base game conditions. It's set to the name of the rule's first condition.

-- You can read multiple entries in a single line like this
local name,unitid,x,y = cdata.name,cdata.unitid,cdata.x,cdata.y
```
</details>

---

## Example of adding custom prefix
Let's say that I want to implement a prefix called "maybe", where its similar to "often" and "seldom" but it only has a 50% chance. First, I would [add the text block to the editor](adding_obj_to_editor.md). Then, I would look for how "seldom" is implemented. This can be found in the `seldom` function inside `condlist` with the below code snippet. The relevant pieces of code I marked with comments:
<details>
<summary>Code Block for "seldom" implementation. Click to expand.</summary>

```lua
seldom = function(params,checkedconds,checkedconds_,cdata)
    local unitid,x,y,conds,i = cdata.unitid,cdata.x,cdata.y,tostring(cdata.conds),cdata.i
    
    if (condstatus[tostring(conds)] == nil) then
        condstatus[tostring(conds)] = {}
    end
    
    -- (A)
    local rnd = fixedrandom(1,6)
    -- end (A)
    
    local d = condstatus[tostring(conds)]
    -- (B)
    local id = "seldom" .. "_" .. tostring(i)
    -- end (B)
    
    if (unitid ~= 2) then
        id = id .. "_" .. tostring(unitid)
    else
        id = id .. "_" .. tostring(unitid) .. tostring(x) .. tostring(y)
    end
    
    if (d[id] ~= nil) then
        rnd = d[id]
    else
        d[id] = rnd
    end
    
    -- (C)
    return (rnd == 1),checkedconds
    -- end (C)
end,
```
</details>

Since "maybe" is similar in functionality to "seldom", I can copy most of the code for seldom into the body of our condition's own function. We see in (A) that `rnd` is a random number between 1-6, and in (C) the unit fulfils the condition if `rnd` is equal to 1 (a 1 in 6 chance). So we want to change our copy of (A) to represent a 50% probability:
```lua
local rnd = fixedrandom(1,2)
```
Note that in (B) the condition's name is used to cache the status of this condition for the unit. All that we need to do here is change the name "seldom" to our condition's name "maybe".

The full code that we add to `condlist` is this:
<details>
<summary>Click to expand</summary>

```lua
condlist["maybe"] = function(params,checkedconds,checkedconds_,cdata)
    local unitid,x,y,conds,i = cdata.unitid,cdata.x,cdata.y,tostring(cdata.conds),cdata.i
    
    if (condstatus[tostring(conds)] == nil) then
        condstatus[tostring(conds)] = {}
    end
    
    local rnd = fixedrandom(1,2)
    
    local d = condstatus[tostring(conds)]
    local id = "maybe" .. "_" .. tostring(i)
    
    if (unitid ~= 2) then
        id = id .. "_" .. tostring(unitid)
    else
        id = id .. "_" .. tostring(unitid) .. tostring(x) .. tostring(y)
    end
    
    if (d[id] ~= nil) then
        rnd = d[id]
    else
        d[id] = rnd
    end
    
    return (rnd == 1),checkedconds
end
```
</details>

---

## Another sample
Here's another sample code from the game that implements "without". I added some commentary about the implementation so that you can get an idea about how an actual condition is implemented. A good exercise is to look through how other conditions are implemented and identify what code is common and what code is specific to the condition.
<details>
<summary>Click to expand</summary>

```lua
without = function(params,checkedconds,checkedconds_,cdata)
    local allfound = 0
    local alreadyfound = {}
    local unitcount = {}
    
    local name,unitid,notcond = cdata.name,cdata.unitid,cdata.notcond
    
    -- "params" is a list of object names accompaning the infix condition (E.g. If "baba without keke is you", params = {"keke"})
    if (#params > 0) then
        -- Initializes "unitcount", which counts the frequency of each parameter of the same value
        for a,b in ipairs(params) do
            if (unitcount[b] == nil) then
                unitcount[b] = 0
            end
            
            unitcount[b] = unitcount[b] + 1
        end
        
        if (unitcount["level"] ~= nil) and (unitcount["level"] > 0) then
            unitcount["level"] = unitcount["level"] - 1
        end
            
        for a,b in ipairs(params) do
            local pname = b

            -- "pnot" is a flag set to true if we are handling "X without not Y is Z"
            local pnot = false
            if (string.sub(b, 1, 4) == "not ") then
                pnot = true
                pname = string.sub(b, 5)
            end
            
            -- "bcode" is an identifier for the parameter. Not sure why "a" isn't used instead since "a" is the index of the parameter.
            local bcode = b .. "_" .. tostring(a)
            
            -- For some reason "X without group is Y" isn't handled. I guess group is jank lol.  
            if (string.sub(pname, 1, 5) == "group") then
                return false,checkedconds
            end
            
            if ((b ~= "level") and (b ~= "empty")) or ((b == "level") and (unitcount["level"] > 0)) then
                -- Normal handling of "without"

                if (pnot == false) then
                    -- Handles "X without Y is Z"

                    if (alreadyfound[bcode] == nil) then

                        -- "unitlists" is a global variable. unitlists["baba"] gets a list of all baba units in the level
                        if (unitlists[b] == nil) or (#unitlists[b] == 0) and (alreadyfound[bcode] == nil) then
                            -- If there are no units for a parameter, mark this parameter as satisfied, meaning that the "without" condition for this parameter is satisfied
                            alreadyfound[bcode] = 1
                            allfound = allfound + 1
                        elseif (unitlists[b] ~= nil) and (#unitlists[b] > 0) then

                            -- In the case that there are units for a parameter, the parameter will still be satisfied only if the number of units for that parameter is less than
                            -- the frequency of the parameter. This means the rule "baba without keke and keke is blue" will still be satisfied if there is only one keke.
                            local found = false
                            
                            if (b ~= name) then
                                if (#unitlists[b] < unitcount[b]) then
                                    found = true
                                end
                            else
                                if (#unitlists[b] < unitcount[b] + 1) then
                                    found = true
                                end
                            end
                            
                            if found then
                                alreadyfound[bcode] = 1
                                allfound = allfound + 1
                            end
                        end
                    end
                else
                    -- Handles "X without not Y is Z"
                    local foundunits = 0
                    local targetcount = unitcount[b]
                    
                    for c,d in pairs(unitlists) do
                        -- Since we are handling "not Y", go through every object type that does not match the parameter and do similar checks from above to test the "without" condition.
                        if (c ~= pname) and (#unitlists[c] > 0) and (c ~= "text") and (string.sub(c, 1, 5) ~= "text_") then
                            for e,f in ipairs(d) do
                                if (f ~= unitid) and (alreadyfound[f] == nil) then
                                    alreadyfound[f] = 1
                                    foundunits = foundunits + 1
                                    
                                    if (foundunits >= targetcount) then
                                        break
                                    end
                                end
                            end
                        end
                        
                        if (foundunits >= targetcount) then
                            break
                        end
                    end
                    
                    if (foundunits < targetcount) and (alreadyfound[bcode] == nil) then
                        alreadyfound[bcode] = 1
                        allfound = allfound + 1
                    end
                end
            elseif (b == "empty") then
                -- Handles "X without empty is Y". Interesting that "X without not empty is Y" is also handled like this.
                local empties = findempty()
                
                if (name ~= "empty") then
                    if (#empties < unitcount[b]) and (alreadyfound[bcode] == nil) then
                        alreadyfound[bcode] = 1
                        allfound = allfound + 1
                    end
                else
                    if (#empties < unitcount[b] + 1) and (alreadyfound[bcode] == nil) then
                        alreadyfound[bcode] = 1
                        allfound = allfound + 1
                    end
                end
            elseif (b == "level") then
                -- Handles "X without level is Y". Though since there is always a level, "without" is automatically unsatisfied.
                -- Interesting that "X without not level is Y" is also handled like this.
                allfound = -99
                break
            end
        end
    else
        print("no parameters given!")
        return false,checkedconds
    end
    
    -- If more than zero parameters are satisfised, then the "not without" condition is NOT satisfied.
    -- (this result gets inverted by testcond(), so 0 or less means satisfied)
    if notcond then
        return (allfound > 0),checkedconds
    end
    
    -- If all parameters are satisfied, then the "without" condition is satisfied.
    return (allfound == #params),checkedconds
end,
```
</details>
