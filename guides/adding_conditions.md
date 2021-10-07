# Implementing a condition
This assumes you added your custom condition through [this guide](custom_word_start.md). I also recommend looking through [the rule specification](../references/rules.md) as reference.

Unlike the guide on adding properties, this guide will feel a bit more specific since everything related to conditions is handled in one function: `testcond()`. This is a function in `conditions.lua` that gets called very often whenever the game wants to evaluate a set of conditions (based on the conditional texts seen in game) on a single object. It simply returns `true` or `false` if the unit fulfils the set of conditions. 

Note that as of 11/14/21, there isn't a mod hook for this function. But Hempuli plans to refactor `testcond` to "be less messy" sometime in the future. I'm guessing he plans to reduce the amount of repeated code and (possibly) provide a mod hook to implement your own conditions. But I don't know the details yet. For now, you need to overwrite this function in your own lua file.

## Example of adding custom prefix
Let's say that I want to implement a prefix called "maybe", where its similar to "often" and "seldom" but it only has a 50% chance. First, I would [add the text block to the editor](adding_obj_to_editor.md). Then, I would look for how "seldom" is implemented. This can be found in `testcond()` with the below code snippet. The relevant pieces of code I marked with comments:
<details>
<summary>Code Block for "seldom" implementation. Click to expand.</summary>

```lua
elseif (condtype == "seldom") then --- (A)
    valid = true
    
    if (condstatus[tostring(conds)] == nil) then
        condstatus[tostring(conds)] = {}
    end
    
    --- (B)
    local rnd = fixedrandom(1,6)
    --- end (B)
    
    local d = condstatus[tostring(conds)]
    local id = "seldom" .. "_" .. tostring(i)
    
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
    
    --- (C)
    if (rnd ~= 1) then
        if (orhandling == false) then
            result = false
            break
        end
    elseif orhandling then
        orresult = true
    end
    --- end (C)

elseif (condtype == "not seldom") then -- (D)
    valid = true
    
    if (condstatus[tostring(conds)] == nil) then
        condstatus[tostring(conds)] = {}
    end
    
    local rnd = fixedrandom(1,6)
    
    local d = condstatus[tostring(conds)]
    local id = "seldom" .. "_" .. tostring(i)
    
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
    
    if (rnd == 1) then
        if (orhandling == false) then
            result = false
            break
        end
    elseif orhandling then
        orresult = true
    end
```
</details>

(A) and (D) check for the condition type being "seldom" or "not seldom". This is part of a giant chain of if-else blocks that covers every single condition in the game. The names of each condition are taken directly from the text unit of the condition, with the "not" variation prepended before the name. So if I name a new object "text_maybe" and set its text type to 7 (See [Text Types](../references/editor_obj_settings.md#text-type)), I can append to the if-else chain this code to implement "maybe":

```lua
elseif (condtype == "maybe") then
    -- Some code that we'll fill in later
elseif (condtype == "not maybe") then
    -- Some code that we'll fill in later
end
```

Since "maybe" is similar in functionality to "seldom", I can copy most of the code for seldom. We see in (B) that `rnd` is a random number between 1-6, and in (C) it checks if `rnd` is not 1 before setting `result = false` (Ignore `orhandling` for now). Making `result = false` makes the entirety of `testcond()` return `false`. So we want to change our copy of (B) to represent a 50% probability:
```lua
local rnd = fixedrandom(1,2)
```

The full code that we append to the if-else chain is this:
<details>
<summary>Click to expand</summary>

```lua
elseif (condtype == "maybe") then
    valid = true
    
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
    
    --- (C)
    if (rnd ~= 1) then
        if (orhandling == false) then
            result = false
            break
        end
    elseif orhandling then
        orresult = true
    end
    --- end (C)

elseif (condtype == "not maybe") then -- (D)
    valid = true
    
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
    
    if (rnd == 1) then
        if (orhandling == false) then
            result = false
            break
        end
    elseif orhandling then
        orresult = true
    end
```
</details>

---

## Another sample
Here's another sample code from the game that implements "without". I added some commentary about the implementation so that you can get an idea about how an actual condition is implemented. A good exercise is to look through how other conditions are implemented and identify what code is common and what code is specific to the condition.
<details>
<summary>Click to expand</summary>

```lua
elseif (condtype == "without") then
    valid = true
    local allfound = 0
    local alreadyfound = {}
    local unitcount = {}
    
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
                result = false
                break
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
                    
                    for c,d in pairs(unitlists) do
                        -- Since we are handling "not Y", go through every object type that does not match the parameter and do similar checks from above to test the "without" condition.
                        if (c ~= pname) and (#unitlists[c] > 0) and (c ~= "text") then
                            for e,f in ipairs(d) do
                                if (f ~= unitid) and (alreadyfound[f] == nil) then
                                    alreadyfound[f] = 1
                                    foundunits = foundunits + 1
                                    
                                    if (foundunits >= unitcount[b]) then
                                        break
                                    end
                                end
                            end
                        end
                        
                        if (foundunits >= unitcount[b]) then
                            break
                        end
                    end
                    
                    if (foundunits < unitcount[b]) and (alreadyfound[bcode] == nil) then
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
        result = false
    end

    -- If all parameters are satisfied, then the "without" condition is satisfied. Otherwise, set result = false, making testcond() return false
    if (allfound ~= #params) then
        if (orhandling == false) then
            result = false
            break
        end
    elseif orhandling then
        orresult = true
    end
```
</details>